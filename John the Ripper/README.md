# John the Ripper

## 🎯 概要
**John the Ripper (JtR)** は、高速かつ柔軟なオープンソースのオフラインパスワードクラッキングツールです。Unixのパスワードハッシュ（DES, MD5, Blowfish等）、Windowsのハッシュ（LM, NTLM）、さらには暗号化されたアーカイブ（ZIP, RAR）やSSH鍵、Kerberosチケットなど、数百種類のハッシュ形式に対応しています。ペネトレーションテストや OSCP においては、ターゲットからダンプした認証情報を平文化（クラック）し、権限昇格やネットワーク内の横展開を行うための極めて重要なツールです。

---

## 🛠 代表的な用途
- **OS認証情報の奪取**: Linux の `/etc/shadow` や Windows の SAM データベースから抽出したハッシュの解析。
- **暗号化ファイルの復元**: パスワード保護された ZIP、RAR、PDF、SSH秘密鍵のパスフレーズ特定。
- **Active Directory 攻撃の完遂**: Kerberoasting や AS-REP Roasting で得たチケットのオフラインクラック。
- **OSCP での利用ポイント**: `hashdump` 等で得たハッシュを `rockyou.txt` を用いて解析し、次のホストへの侵入に必要なパスワードを特定する際に必須となります。

---

## 🔍 基本コマンド
```bash
# デフォルト設定での実行（single, wordlist, incremental の順で試行）
john hash.txt

# 指定したワードリストを使用して解析
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# 解析済みのパスワードを表示
john --show hash.txt

# ハッシュ形式を指定して実行（例：NTLM）
john --format=nt hash.txt
```

---

## 🧪 よく使う攻撃シナリオ
- **Linux権限昇格シナリオ**: 低権限で侵入後、不適切な権限の `/etc/shadow` を発見。`unshadow` で統合したファイルを攻撃端末へ持ち帰り、辞書攻撃で root パスワードを特定。
- **ADドメイン制覇シナリオ**: Impacket 等でサービスアカウントのチケット（TGS-REP）を収集。JtR で解析してサービスアカウントの平文パスワードを奪取し、横展開を開始。
- **秘密鍵の奪取シナリオ**: ターゲット上で見つけた `id_rsa` がパスフレーズ保護されている場合、`ssh2john` でハッシュ化して JtR で解析し、SSHアクセスを可能にする。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
```bash
# 過去のデータ漏洩リストから得たハッシュの形式を特定する
john --list=formats

# ターゲット組織特有のキーワードからカスタム単語リストを作成する（外部ツール出力を利用）
cat leashed_info.txt | john --stdout --rules > custom_wordlist.txt
```

---

### 🧭 Enumeration（詳細調査）
※ JtRはオフラインツールのため、通常このフェーズでは使用されませんが、脆弱なポリシーを確認するために使用されることがあります。
```bash
# 入手したハッシュがどのアルゴリズムかベンチマークで確認
john --test
```

---

### 🚪 Initial Access（初期侵入）
```bash
# Webアプリのソースコード露出で見つかった独自ハッシュを解析
john --format=raw-md5 discovered_hashes.txt --wordlist=dict.txt

# 公開ディレクトリから取得したバックアップのZIPパスワードを特定
zip2john backup.zip > zip.hash
john zip.hash
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# Linuxシステムからpasswdとshadowファイルを結合（解析準備）
unshadow /etc/passwd /etc/shadow > myhashes.txt

# WindowsのSAMおよびSYSTEMレジストリから抽出したハッシュの解析（NTLM形式）
john --format=nt sam_hashes.txt
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# rootユーザーのハッシュのみをピンポイントで攻撃
john --users=0 myhashes.txt

# 入手したSSH秘密鍵のパスフレーズを解析
ssh2john id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt

# sudoers設定に関連するユーザーのパスワードを優先解析
john --users=admin,svc_deploy myhashes.txt
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# Kerberoastingで得たハッシュを解析（krb5tgs形式）
john --format=krb5tgs krb_tgs_hashes.txt --wordlist=rockyou.txt

# AS-REP Roastingで得たハッシュを解析
john --format=krb5asrep asrep_hashes.txt --wordlist=rockyou.txt

# メモリダンプから得たNTLMv2ハッシュの解析
john --format=netntlmv2 msv_hashes.txt
```

---

### 🏢 Active Directory（AD）
```bash
# ドメインコントローラーからダンプしたNTDS.dit内の全ハッシュを解析
john --format=nt ntds_hashes.txt --wordlist=big_dict.txt

# 収集した全ドメインユーザーに対し、共通のルール変形を適用して攻撃
john --wordlist=dict.txt --rules=Jumbo ad_users.hash
```

---

## 🧰 便利オプション一覧
- `--format=<format>`：ハッシュのアルゴリズムを明示的に指定。
- `--wordlist=<file>`：辞書攻撃に使用する単語リストを指定。
- `--rules`：辞書の単語に数字や記号を自動付与するルール変形を有効化。
- `--incremental`：総当たり（ブルートフォース）モードを有効化。
- `--mask=?l?l?l?l`：特定のパターン（例：小文字4文字）を指定したマスク攻撃。
- `--show`：現在までに解析に成功した結果をハッシュと共に出力。
- `--session=<name>`：解析セッションに名前を付けて保存。
- `--restore=<name>`：中断したセッションを再開。
- `--fork=<N>`：複数のCPUコアを使用して並列処理（高速化）。

---

## 🚀 実践例

```bash
# 【1】標準的なLinux Shadow解析（unshadow処理を含む）
unshadow /etc/passwd /etc/shadow > combined.txt
john --wordlist=/usr/share/wordlists/rockyou.txt combined.txt

# 【2】マスク攻撃：特定の規則性（小文字6文字など）に基づいた総当たり
john --mask=?l?l?l?l?l?l hash.txt -min-len=6

# 【3】辞書攻撃 + ルール適用：単語の前後への数字付与などを自動実行
john --wordlist=passwords.txt --rules=All ntlm.hash

# 【4】SSH秘密鍵の解析：ssh2johnを利用した一連の流れ
/usr/share/john/ssh2john.py id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

---

## 🧩 他ツールとの組み合わせ
- **impacket** → `secretsdump.py` でダンプしたハッシュ（NTLM）をそのまま JtR の解析に渡します。
- **ffuf** → Webディレクトリ列挙で見つかった設定ファイルからハッシュを抽出し、JtR で平文化。
- **netexec** → JtR で解析成功した平文パスワードを `nxc smb` のスプレー攻撃に使用して、ネットワーク全体への権限を確認。
- **Responder** → ネットワーク上でキャプチャした NTLMv2 ハッシュを JtR に渡し、オンライン認証をバイパス。

---

## 📝 注意点（OSCP 試験での使い方）
- **リソース管理**: JtR は CPU を激しく消費します。試験環境の VM が重くなる場合は `--fork` 数を調整してください。
- **Potファイルの確認**: クラック済みのパスワードは `~/.john/john.pot` に保存されます。同じハッシュを再度攻撃しようとすると「No password hashes left to crack」と出るため、`--show` を忘れないようにしてください。
- **ハッシュ形式の不一致**: 自動認識が失敗する場合は、`hash-identifier` ツールや公式サイトの `Example Hashes` を参照して `--format` を手動指定してください。

---

## 🔗 参考リンク
- 公式：[Openwall - John the Ripper](https://www.openwall.com/john/)
- 公式 Wiki：[John the Ripper Documentation](https://openwall.info/wiki/john)
- ハッシュ例：[John the Ripper Format Examples](https://countuphacking.com/john-the-ripper-formats/)
