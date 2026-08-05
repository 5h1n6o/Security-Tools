# Google Hacking (Google Dorks)

## 🎯 概要
Google Hacking（別名：Google Dorking）は、Google検索エンジンの高度な検索演算子（演算子）を駆使して、一般的な検索では到達しにくい機密情報や脆弱な構成、公開されているバックドアなどを特定する手法です。ペネトレーションテストやOSCPのOSINT（情報収集）フェーズにおいて、ターゲットに直接パケットを送信することなく、受動的（Passive）に攻撃対象領域（Attack Surface）を広げ、脆弱性を発見するための極めて強力な手段となります。

---

## 🛠 代表的な用途
- **機密ドキュメントの発見**: 誤って公開されているPDF、Excel、Wordファイルや社内限定のメモを抽出。
- **設定ファイル・バックアップの探索**: `.env`、`config.php`、`.bak`、`.sql` などの認証情報を含むファイルを特定。
- **管理パネルの特定**: ログイン画面やデバッグ画面、管理コンソールの露出を確認。
- **ディレクトリリスティングの発見**: サーバーの設定ミスによりファイル一覧が表示されているディレクトリを特定。
- **OSCPでの利用ポイント**: ターゲットドメイン（`site:target.com`）に対して、初期調査で公開されている技術スタックやサブドメイン、漏洩した認証情報を素早く特定するために使用されます。

---

## 🔍 基本コマンド
```bash
# 特定のドメイン内から結果を絞り込む
site:example.com

# ページタイトルに特定の文字列を含むものを検索
intitle:"index of"

# URLの中に特定の文字列を含むものを検索
inurl:/admin/login.php

# 特定のファイル拡張子を指定して検索
filetype:pdf
```

---

## 🧪 よく使う攻撃シナリオ
- **構成ファイルの漏洩**: `filetype:env` や `inurl:web.config` を使用して、データベースの接続文字列やAPIキーを盗み出します。
- **既知の脆弱なパスの探索**: 特定のCMS（WordPressなど）の脆弱なプラグインパスを `inurl:/wp-content/plugins/` で一括検索します。
- **公開ディレクトリの探索**: `intitle:"index of" "parent directory"` を用いて、サーバー内のファイル構造を直接閲覧します。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
```bash
# 特定ドメインに属する全てのサブドメインを探索
site:*.example.com -www

# LinkedInから特定の組織に所属する従業員のプロフィールを特定
site:linkedin.com/in "Target Company"

# 公開されている社員名簿や連絡先リストのPDFを検索
site:example.com filetype:pdf "directory" | "staff" | "contact"

# ペーストサイト（Pastebin等）に投稿されたドメイン関連の情報を探索
site:pastebin.com "example.com"

# 組織のGitHubリポジトリやコード片の露出を検索
site:github.com "example.com"
```

---

### 🔎 Reconnaissance（技術的偵察）
```bash
# ターゲットが使用している技術（phpinfoなど）の露出を確認
site:example.com "phpinfo()"

# インストール済みのWebアプリケーションのREADMEやマニュアルを特定
site:example.com inurl:readme.txt | inurl:license.txt

# サーバーのバナー情報やエラーメッセージがインデックスされているか確認
site:example.com "Apache Server at" | "nginx/"

# 公開されているネットワーク構成図やトポロジー図を検索
site:example.com filetype:vsd | filetype:png "network diagram"
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# ディレクトリリスティングが有効なディレクトリを特定
site:example.com intitle:"index of"

# 開発用・テスト用と思われるサブドメインやディレクトリの特定
site:example.com inurl:dev | inurl:test | inurl:staging

# WordPressのユーザー名列挙に繋がるパスを特定
site:example.com inurl:author

# 特定のAPIエンドポイントやドキュメントの特定
site:example.com inurl:api/v1 | inurl:swagger.json
```

---

### 🚪 Initial Access（初期侵入）
```bash
# 認証なしでアクセス可能な管理パネルを特定
site:example.com intitle:"login" "admin"

# .env ファイルなどの環境設定ファイルから認証情報を取得
site:example.com filetype:env "DB_PASSWORD"

# 公開されているSQLダンプファイルからデータベースの中身を取得
site:example.com filetype:sql

# バックアップファイル（.old, .bak, .zip）の露出を確認
site:example.com filetype:bak | filetype:old | filetype:zip

# SSH秘密鍵（id_rsa）が誤って公開されていないか検索
site:example.com "BEGIN RSA PRIVATE KEY" filetype:key | filetype:txt
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# 侵入後に確認した内部ファイル名やプロジェクト名をGoogleで検索し、関連資料を取得
"ProjectName_Internal_Specs" filetype:pdf

# ターゲットサーバーで見つかった特有のエラーログの内容で検索し、既知の解決策や構成ミスを特定
site:example.com "Specific Error String from Logs"
```

---

### 🏢 Active Directory（AD）
基本的に関連なし。ただし、外部公開されているAD管理ツール（ADFSログイン等）の特定に使用される場合があります。

---

## 🧰 便利オプション一覧
- `site:`：検索対象のドメインを指定。
- `intitle:`：ページタイトルにキーワードが含まれるものを検索。
- `allintitle:`：ページタイトルに全てのキーワードが含まれるものを検索。
- `inurl:`：URL内にキーワードが含まれるものを検索。
- `filetype:`：特定の拡張子を持つファイルを検索。
- `allintext:`：ページ本文内にキーワードが含まれるものを検索。
- `cache:`：Googleに保存されているキャッシュページを表示（削除済みページの閲覧に有効）。
- `-` (マイナス記号)：特定のキーワードやドメインを結果から除外。
- `|` (パイプ)：OR検索（「A または B」）。

---

## 🚀 実践例

```bash
# 【1】OSCP初期調査：特定ドメインの設定ミスによる機密ファイル一括検索
site:target.com filetype:env | filetype:config | filetype:xml | filetype:txt "password"

# 【2】ログインページの特定とバイパス情報の探索
site:target.com inurl:login | inurl:admin | inurl:manage

# 【3】ディレクトリリスティングを狙った機密フォルダの特定
site:target.com intitle:"index of" "backup" | "private" | "confidential"

# 【4】公開されているログファイルからの情報収集
site:target.com filetype:log intext:"error" | "debug" | "sql"

# 【5】SQLインジェクションの可能性があるパラメータの特定
site:target.com inurl:id= | inurl:item= | inurl:page= filetype:php

# 【6】古いバージョンのソフトウェアや構成の特定
site:target.com "powered by" "WordPress 4.5"

# 【7】特定の社内システム（Jira, Confluence等）の露出確認
site:target.com inurl:atlassian.net | intitle:"Dashboard - Jira"

# 【8】SSL/TLS証明書に関連する情報の特定
site:target.com inurl:cert | filetype:crt | filetype:p12

# 【9】メールアドレス形式の文字列を組織ドメインから収集
site:target.com "@target.com"

# 【10】特定のドキュメントテンプレート（見積書、請求書）の特定
site:target.com filetype:doc | filetype:docx "invoice" | "quotation"

# 【11】削除された可能性があるページの過去の内容を確認
cache:target.com/admin/hidden_page.php

# 【12】Ciscoなどのネットワーク機器の設定画面を検索
intitle:"Cisco Systems" inurl:level/15/exec/-
```

---

## 🧩 他ツールとの組み合わせ
- **theHarvester** → Googleなどの検索エンジンからメールアドレス、サブドメイン、従業員名を自動で収集。
- **Metagoofil** → Google検索で見つかったドキュメントを一括ダウンロードし、メタデータ（作成者、OS情報等）を抽出。
- **Recon-ng** → `google_site_web` モジュールなどを使用して、Googleの検索結果をDBに自動登録。
- **GHDB (Google Hacking Database)** → Exploit-DBが提供する膨大なDorkリストを参照し、最新の脆弱なパスを特定。

---

## 📝 注意点（OSCP 試験での使い方）
- **パッシブリコンの徹底**: Google検索自体はターゲットに通知されませんが、検索結果のリンクをクリックするとターゲットにアクセスが発生します。
- **キャプチャ制限**: 短時間に大量の高度な検索を繰り返すと、Googleからボットと判定され、IPが一時的に制限される場合があります。
- **GHDBの活用**: 自分でクエリを考える前に、[Exploit-DBのGHDB](https://www.exploit-db.com/google-hacking-database)で目的のサービスに合うDorkがないか確認するのが定石です。

---

## 🔗 参考リンク
- 公式：[Google Search Operators](https://support.google.com/websearch/answer/2466433)
- データベース：[Google Hacking Database (GHDB)](https://www.exploit-db.com/google-hacking-database)
- 良質解説：[Google Hacking for Penetration Testers (Johnny Long)](https://www.amazon.com/Google-Hacking-Penetration-Testers-Johnny/dp/1597491764)
