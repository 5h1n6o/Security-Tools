# enum4linux-ng

## 🎯 概要
**enum4linux-ng** は、Windows および Samba システムから情報を収集するための強力な列挙（Enumeration）ツールです。従来の Perl ベースの `enum4linux` を Python 3 で書き直した次世代版であり、より高速で、YAML や JSON 形式での出力にも対応しています。ペネトレーションテストや OSCP において、SMB（TCP 139/445）ポートが開いているターゲットに対して最初に使用すべき「決定版」のツールであり、共有フォルダ、ユーザー一覧、グループ情報、パスワードポリシーなどを自動で抽出します。

---

## 🛠 代表的な用途
- **共有（Shares）の列挙**: 匿名アクセス可能なフォルダや、機密情報が含まれるバックアップ共有を特定します。
- **ユーザー・グループの抽出**: 攻撃対象となる有効なユーザー名リストを作成し、後のブルートフォース攻撃に繋げます。
- **パスワードポリシーの確認**: パスワードの最小長やロックアウトのしきい値を把握し、安全な辞書攻撃の戦略を立てます。
- **OSCP での利用ポイント**: SMB サービスが稼働している場合、まず実行して「Null Session（匿名ログイン）」が可能かどうかを確認し、そこから得られる情報を元に攻略の足がかり（初期資格情報など）を探すのに必須です。

---

## 🔍 基本コマンド
```bash
# 全ての列挙項目を網羅的に実行（最も一般的）
enum4linux-ng -A 10.10.10.1

# 特定のユーザー名とパスワードを使用して情報を取得
enum4linux-ng -u "user" -p "password" -A 10.10.10.1

# 匿名セッション（Null Session）を使用してユーザーのみを列挙
enum4linux-ng -U 10.10.10.1

# 出力結果を JSON 形式で保存（後続のツール処理に便利）
enum4linux-ng -A 10.10.10.1 -oJ output.json
```

---

## 🧪 よく使う攻撃シナリオ
- **匿名アクセスによる機密情報奪取**: パスワードなしでアクセス可能な共有フォルダを特定し、設定ファイルやメモからパスワードを盗み出します。
- **パスワードポリシーに合わせた辞書攻撃**: システムが要求するパスワード強度を事前に取得し、無駄な試行を省いた効率的なログイン攻撃を実施します。
- **SID フィルタリングによるドメイン構造把握**: ドメインの SID を取得し、ドメインコントローラーや関連する信頼関係にあるドメインを特定します。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
基本的に関連なし。インターネットに公開されている Samba サーバーがある場合のみ、Reconnaissance と同様のコマンドを使用します。

---

### 🔎 Reconnaissance（技術的偵察）
```bash
# SMB バージョン情報の取得とサービスの生存確認
enum4linux-ng -v 10.10.10.1

# NetBIOS 名とワークグループ/ドメイン名の特定
enum4linux-ng -n 10.10.10.1
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# 共有名（Shares）とアクセスの可否を確認
enum4linux-ng -S 10.10.10.1

# ユーザー一覧を列挙（RPC 経由）
enum4linux-ng -U 10.10.10.1

# グループ名とそのメンバーを列挙
enum4linux-ng -G 10.10.10.1

# パスワードポリシー（最小長、有効期間など）を取得
enum4linux-ng -P 10.10.10.1

# ドメイン SID (Security Identifier) の取得
enum4linux-ng -i 10.10.10.1
```

---

### 🚪 Initial Access（初期侵入）
```bash
# 書き込み権限のある共有フォルダの特定
enum4linux-ng -S 10.10.10.1 | grep "WRITE"

# RID サイクルによる有効なユーザー名の強制的な列挙
enum4linux-ng -R 500-1100 10.10.10.1
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# 侵入済みの端末からループバックアドレスに対して実行し、隠れたサービスを確認
enum4linux-ng -A 127.0.0.1
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# ローカルグループ（Administrators 等）のメンバーを確認
enum4linux-ng -G 10.10.10.1 | grep -i "admin"

# マシンアカウントや特権を持つサービスアカウントの特定
enum4linux-ng -U 10.10.10.1 | grep "\$"
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# ユーザーリストを作成するためにユーザー名のみをクリーンに抽出
enum4linux-ng -U 10.10.10.1 | grep "user:" | cut -d "[" -f2 | cut -d "]" -f1 > users.txt
```

---

### 🌐 Internal Enumeration（内部ネットワーク探索）
```bash
# テキストファイルに記載された IP リストに対して一括スキャン
enum4linux-ng -iL targets.txt -A

# 特定のネットワーク範囲内のホストに対して SMB 情報を列挙
enum4linux-ng -A 192.168.50.0/24
```

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# Proxychains 経由で内部ネットワークの SMB サーバーを調査
proxychains enum4linux-ng -A 172.16.5.10
```

---

### 🏢 Active Directory（AD）
```bash
# LDAP 経由でのドメイン情報の列挙
enum4linux-ng -L 10.10.10.1

# プリンタ情報を介した情報の列挙
enum4linux-ng -p 10.10.10.1

# ドメインコントローラー（DC）の特定と情報収集
enum4linux-ng -d 10.10.10.1
```

---

## 🧰 便利オプション一覧
- `-A`：全ての列挙（All）を実行します。
- `-U`：ユーザーの一覧を取得します。
- `-S`：共有の一覧を取得します。
- `-P`：パスワードポリシー情報を取得します。
- `-G`：グループ情報を取得します。
- `-i`：ドメイン SID などの基本情報を取得します。
- `-R <range>`：指定した範囲の RID でユーザーを総当たり（サイクル）します。
- `-u <user>` / `-p <pass>`：認証が必要な場合に使用します。
- `-oJ <file>`：結果を JSON ファイルに保存します。
- `-oY <file>`：結果を YAML ファイルに保存します。

---

## 🚀 実践例（10個）

```bash
# 【1】OSCP初期段階： Null Sessionを前提とした全自動スキャン
enum4linux-ng -A $IP

# 【2】ユーザー名リストを作成するためにユーザー列挙のみを実行
enum4linux-ng -U $IP -oJ users_discovery.json

# 【3】ドメイン環境で入手した一般ユーザー権限を用いて詳細情報を再列挙
enum4linux-ng -u "dave" -p "password123" -d "corp.com" -A $IP

# 【4】匿名アクセスが許可されている共有フォルダを素早く特定
enum4linux-ng -S $IP | grep -C 5 "Mapping: OK"

# 【5】特定のパスワード文字列でロックアウトされないかポリシーを確認
enum4linux-ng -P $IP

# 【6】標準的な列挙でユーザーが出ない場合、RID範囲を広げて再試行
enum4linux-ng -R 500-2000 $IP

# 【7】LDAPサービスが動いているドメインコントローラーへの列挙
enum4linux-ng -L $IP

# 【8】内部ネットワークのホストリストを読み込んでバッチ処理
enum4linux-ng -iL alive_smb_hosts.txt -A -oY internal_network_report.yaml

# 【9】NetBIOS名解決を行いホストの役割（DC, SQL等）を推測
enum4linux-ng -n $IP

# 【10】特定のグループに誰が所属しているかピンポイントで調査
enum4linux-ng -G $IP | grep -A 10 "Administrators"
```

---

## 🧩 他ツールとの組み合わせ
- **impacket** → `enum4linux-ng` で有効なユーザーと共有を特定した後、`psexec.py` や `smbclient.py` でシェル取得を試みます。
- **netexec** → 大規模ネットワークにおいて、`netexec smb <range> --shares` の結果を補完するために `enum4linux-ng` を使用します。
- **nmap** → `nmap -p 445 --open` で見つかったターゲットの詳細調査に `enum4linux-ng` を適用します。
- **John the Ripper** → `enum4linux-ng` の結果から得たパスワードポリシーに合わせて辞書を作成し、ハッシュ解析の効率を高めます。

---

## 📝 注意点（OSCP 試験での使い方）
- **出力の保存**: `-A` オプションの結果は非常に長くなるため、必ず `-oJ` や `tee` コマンドでファイルに残してください。レポート作成時に役立ちます。
- **Null Session の可否**: 最初に出力される「Session Check」セクションで、匿名ログインが成功したかどうかを必ず確認してください。失敗している場合、その後の情報の多くは不正確または空になります。
- **タイムアウト**: ネットワークが不安定な場合、特定の列挙項目で止まることがあります。その場合は `-A` ではなく、必要なフラグ（`-U`, `-S` 等）を個別に実行してください。

---

## 🔗 参考リンク
- 公式：[enum4linux-ng GitHub Repository](https://github.com/cddmp/enum4linux-ng)
- 解説：[SMB Enumeration Guide (HackTricks)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb)

## 📝 補足（Notes）

445ポート（SMB）が開放されているのを確認した際の、**「Nmap NSE」「enum4linux-ng」「smbclient」の使い分けと違い**について、以下の表にまとめました。

### SMB調査ツールの比較と使い分け

| 手法 / ツール | 主な目的 | 実行のタイミング | 得られる主な情報 |
| :--- | :--- | :--- | :--- |
| **Nmap NSE** (初期スキャン) | サービスの基本特定と**OSの特定** | 445ポート開放を確認した直後 | OSバージョン、NetBIOS名、ドメイン名、SMBv1の有無 |
| **enum4linux-ng** | 包括的な**プロファイリング** | ターゲットがWindows/Sambaと判明した後 | **ユーザー一覧**、パスワードポリシー、グループ所属、共有フォルダ名のリスト |
| **smbclient** | 共有フォルダ内への**直接介入** | 調査対象の共有フォルダ名が判明した後 | フォルダ内のファイル一覧、**機密ファイルの取得(get)**、リバースシェルの配置(put) |

---

### 実戦的なアクションフローの要約

1.  **Nmap `-sC`**: 
    *   まずは「相手が何者か」を素早く把握します。脆弱性スキャン（`--script vuln`）を同時に行うこともあります。
2.  **enum4linux-ng `-A`**: 
    *   次に「匿名ログイン（Null Session）」が通るか確認し、**攻撃対象となるユーザー名のリスト**やパスワードの最低文字数などのポリシーを根こそぎ奪取します。
3.  **smbclient `-L` / `-U`**: 
    *   最後に見つかった共有フォルダ（`Backup`や`Users`など）の中に入り、具体的な**情報の窃取**や侵入の足がかりとなるファイルの操作を行います。

実戦では、**Nmap `-sC` → enum4linux-ng `-A` → smbclient `-L`** の順で深掘りしていくのが定石です。
このように、**「情報の解像度を段階的に上げていく」**のが、ペネトレーションテストにおける効率的な流れとなります。
