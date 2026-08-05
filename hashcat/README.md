# hashcat

## 🎯 概要
**hashcat**は、世界最速を自負するオープンソースのオフラインパスワードクラッキングツールです。CPUだけでなく、GPUの並列演算能力を最大限に活用することで、膨大な計算量を必要とするパスワードハッシュの解析を高速に実行します。ペネトレーションテストやOSCPにおいては、ターゲットから奪取したパスワードハッシュ（Windows NTLMハッシュ、Linux Shadowファイル、Kerberosチケット、データベースハッシュなど）を平文パスワードに戻すために使用されます。非常に多くのハッシュアルゴリズム（MD5、SHA-1、SHA-256、bcryptなど）に対応しており、柔軟な攻撃モードを備えているのが特徴です。

---

## 🛠 代表的な用途
- **奪取したハッシュの解析**: システムやデータベースからダンプした認証情報の平文特定。
- **パスワードポリシーの検証**: 解析の成功率や傾向から、組織のパスワード設定基準が脆弱でないかを評価。
- **AD環境の攻略**: KerberoastingやAS-REP Roastingで得たチケットハッシュのオフライン解析。
- **OSCP での利用ポイント**: 試験環境では、取得したハッシュを自分のローカルマシン（GPU環境推奨）に持ち帰り、`rockyou.txt`などの標準辞書やカスタムルールを用いて、次のホストへの侵入に必要なパスワードを特定する際に必須となります。

---

## 🔍 基本コマンド
```bash
# 特定のハッシュアルゴリズム（例：MD5 = 0）で辞書攻撃
hashcat -a 0 -m 0 hash.txt dict.txt

# 解析済みの結果を表示
hashcat -m 0 hash.txt --show

# 解析セッションの中断と復元（--restore）
hashcat -a 0 -m 1000 --session <名前> hash.txt dict.txt
hashcat --restore --session <名前>
```

---

## 🧪 よく使う攻撃シナリオ
- **辞書攻撃 + ルール変形**: 標準的な単語リストに対し、記号の付与や文字の置換（`a`を`4`にする等）を自動適用して解析効率を上げます。
- **マスク攻撃（総当たり）**: 「英字3文字 + 数字4文字」のように、ターゲットのパスワードポリシーに基づいたパターンを定義して効率的に総当たりを行います。
- **ハイブリッド攻撃**: 辞書にある単語の前後に対して、特定の文字パターンを組み合わせます。

---

## 📌 フェーズ別コマンド例  
Pentest‑Playbook の各フェーズで実際に使うコマンド例を記載します。  
書式は **「# 説明 → コマンド例」** の形式で統一します。

---

### 🔎 OSINT（外部情報収集）
```bash
# 過去の漏洩リスト（leaked password list）に含まれるハッシュが既知のアルゴリズムか確認
# ※hash-identifier等と併用し、hashcatに渡すモード番号を特定する
```

---

### 🔎 Reconnaissance（技術的偵察）
オフラインツールのため、このフェーズでは基本的に使用しません。

---

### 🧭 Enumeration（外部サービスの詳細調査）
オフラインツールのため、このフェーズでは基本的に使用しません。

---

### 🚪 Initial Access（初期侵入）
```bash
# Webアプリのソースコード露出やGitHub検索で見つけた設定ファイル内のハッシュを解析
hashcat -a 0 -m 0 discovered_hash.txt /usr/share/wordlists/rockyou.txt
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# 侵入後にLinuxの/etc/shadowをダンプした場合の解析（SHA-512 = 1800）
hashcat -a 0 -m 1800 shadow_hashes.txt rockyou.txt
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# WindowsのSAMデータベースからダンプした管理者ハッシュを解析（NTLM = 1000）
hashcat -a 0 -m 1000 sam_hashes.txt rockyou.txt

# バックアップファイル（.bak）内から見つかった独自ハッシュの解析
hashcat -a 3 -m 100 backup_hash.txt ?l?l?l?l?d?d?d?d
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# メモリからダンプしたパスワードハッシュを一括解析
hashcat -a 0 -m 1000 creds_dump.txt dict.txt -r /usr/share/hashcat/rules/best64.rule

# データベースのユーザーテーブルから取得したパスワードを解析（MySQL = 300）
hashcat -a 0 -m 300 database_dump.txt common_passes.txt
```

---

### 🏢 Active Directory（AD）
```bash
# Kerberoastingで得たTGS-REPハッシュの解析（krb5tgs = 13100）
hashcat -a 0 -m 13100 kerberoast_hashes.txt rockyou.txt

# AS-REP Roastingで得たハッシュの解析（krb5asrep = 18200）
hashcat -a 0 -m 18200 asrep_hashes.txt rockyou.txt

# ドメインコントローラーのNTDS.ditからダンプしたハッシュの解析
hashcat -a 0 -m 1000 ntds_hashes.txt big_dict.txt
```

---

## 🧰 便利オプション一覧
- `-a`：攻撃モードの指定（0: 辞書, 3: マスク, 6: ハイブリッド）。
- `-m`：ハッシュアルゴリズムの番号指定（0: MD5, 100: SHA1, 1000: NTLM, 1800: SHA512, 13100: Kerberos 5等）。
- `-r`：単語リストを変形させるルールファイルを指定。
- `-o`：解析成功したパスワードをファイルに出力。
- `-w`：パフォーマンスプロファイルの調整（1: 低負荷, 3: 高速, 4: 非常に高速/画面が固まる可能性あり）。
- `-1, -2, -3, -4`：カスタム文字セットの定義。
- `--show`：既にクラック済みの結果をハッシュと共に出力。
- `--force`：警告を無視して実行（仮想環境など）。

---

## 🚀 実践例

```bash
# 【1】標準的な辞書攻撃（NTLMハッシュをrockyouで解析）
hashcat -a 0 -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt

# 【2】ルールベース攻撃（辞書にある単語に数字や記号を付与して解析率アップ）
hashcat -a 0 -m 1000 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule

# 【3】マスク攻撃（総当たり：英大文字1つ、英小文字、数字などのパターン指定）
# 例：英小文字1字+英大文字1字+英小文字3字+数字4字（例：aAabc1234）
hashcat -a 3 -m 1000 hashes.txt ?l?u?l?l?l?d?d?d?d

# 【4】カスタム文字セットを使用したマスク攻撃
# 英小文字と記号のみの8文字をターゲットにする
hashcat -a 3 -m 1800 hash.txt -1 ?l?s ?1?1?1?1?1?1?1?1

# 【5】Kerberoastingで得たサービスアカウントのチケットを解析（OSCP頻出）
hashcat -a 0 -m 13100 krb_tgs_hashes.txt rockyou.txt
```

---

## 🧩 他ツールとの組み合わせ
- **impacket** → `secretsdump.py`や`GetUserSPNs.py`で取得したハッシュ（`hashcat`形式）をそのまま解析に渡します。
- **netexec** → `nxc smb <IP> --sam`で得たハッシュを解析し、得られたパスワードで再度`nxc`を実行して「Pwned!」なホストを探します。
- **Responder** → ResponderでキャプチャしたNTLMv2ハッシュ（モード 5600）を解析します。
- **CeWL** → ターゲットのWebサイトから収集した単語リストをカスタム辞書としてHashcatに渡します。

---

## 📝 注意点（OSCP 試験での使い方）
- **GPUの利用**: 試験環境（Kali VM）内ではGPUパススルーが設定されていない限り、GPU加速が使えず解析が非常に遅くなります。可能であればホストOS側のHashcat（GPU対応）で解析することを推奨します。
- **ハッシュ形式の整形**: `hashcat`が認識できる形式（例：`user:hash`や`hash`のみ）にテキストを加工する必要があります。
- **辞書の準備**: Kali Linuxの標準辞書`/usr/share/wordlists/rockyou.txt.gz`は、使用前に`gzip -d`で展開しておく必要があります。
- **モード番号の確認**: アルゴリズム番号を間違えると解析が始まらないため、公式サイトの例（Example Hashes）で自分のハッシュと形式が一致するか確認してください。

---

## 🔗 参考リンク
- 公式：[hashcat.net](https://hashcat.net/hashcat/)
- ハッシュ例：[Hashcat Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)
- ルール解説：[Hashcat Rule-based Attack](https://hashcat.net/wiki/doku.php?id=rule_based_attack)
