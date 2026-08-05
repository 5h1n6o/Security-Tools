# mysql

## 🎯 概要
MySQL は世界で最も広く利用されているオープンソースのリレーショナルデータベース管理システム（RDBMS）です。  
Pentest / OSCP では、**弱パスワードログイン・DB 内部情報の列挙・Web アプリのバックエンド調査**などで頻繁に利用されます。

MySQL は誤設定や弱い認証が多く、Boot2Root や OSCP 試験でも突破口になりやすいサービスです。

---

## 🛠 代表的な用途（Pentest / OSCP 視点）
- 弱パスワードでのログイン確認  
- データベース列挙（DB / テーブル / カラム）  
- Web アプリのバックエンド調査  
- 設定ファイルから認証情報を抽出  
- OSCP では DB 内の認証情報 → OS ログインに繋がることが多い  

---

## 🔍 基本コマンド
```
mysql -h <TARGET> -u root -p
mysql -h <TARGET> -u <USER> -p<PASSWORD>
show databases;
use <DB>;
show tables;
select * from <TABLE>;
```

---

## 🧪 よく使う攻撃シナリオ
- **弱パスワードログイン**  
  → root / admin / test などの簡易パスワード  
- **Web アプリの設定ファイルから認証情報を抽出**  
  → wp-config.php / config.php / .env  
- **DB 内のユーザー情報から OS ログインへ派生**  
  → パスワード再利用  
- **MySQL が外部公開されているケース**  
  → OSCP では頻出のミス構成  

---

## 📌 Pentest-Playbook との対応（フェーズ別コマンド例付き）

### 🔎 Reconnaissance（ポート・サービスの発見）
MySQL が公開されているか、バージョン情報が漏れていないかを確認する。

```
nmap -Pn -sV -p3306 <TARGET>
nmap -Pn -p3306 --script mysql-info <TARGET>
```

**目的：**  
- MySQL が外部公開されているか確認  
- バージョン情報から既知の脆弱性を推測  

---

### 🧭 Enumeration（DB の内部構造を調査）
弱パスワードログインや DB 内部の列挙を行う。

```
mysql -h <TARGET> -u root -p
mysql -h <TARGET> -u <USER> -p<PASSWORD>
show databases;
use <DB>;
show tables;
select * from <TABLE>;
```

**目的：**  
- DB / テーブル / カラムの列挙  
- 認証情報の探索  
- Web アプリのバックエンド構造の把握  

---

### 🚪 Initial Access（初期侵入の突破口）
Web アプリや設定ファイルから得た認証情報を使ってログインする。

```
mysql -h <TARGET> -u appuser -p<password>
mysql -h <TARGET> -u root -p
mysql -h <TARGET> -u admin -padmin
```

**目的：**  
- 弱パスワードログイン  
- Web アプリの設定ファイル（config.php / .env）から得た認証情報の利用  
- DB 内のユーザー情報から OS ログインへ派生  

---

### 📈 Privilege Escalation（権限昇格）
DB 内の認証情報を OS ログインに利用する。

```
select user, password from users;
select username, passwd from accounts;
```

**目的：**  
- DB 内のパスワード再利用  
- OS ログイン（SSH / WinRM）への派生  
- Web アプリの管理者アカウント奪取  

---

### 🏢 Active Directory（AD）
MySQL は AD とは直接関係しないが、  
**内部 Web アプリのバックエンドとして使われている場合、  
AD 認証情報が格納されているケースがある。**

```
select * from ad_users;
select username, password from ldap_accounts;
```

**目的：**  
- AD 認証情報の漏洩確認  
- 内部 Web アプリの AD 連携の調査  

