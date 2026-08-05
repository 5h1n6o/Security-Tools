# ffuf

## 🎯 概要
**ffuf**（Fuzz Faster U Fool）は、Go言語で開発された極めて高速かつ柔軟なオープンソースのWebファジングツールです。ペネトレーションテストの各フェーズにおいて、Webサーバー上の隠しディレクトリ、ファイル、サブドメイン、仮想ホスト（VHost）、さらにはHTTPリクエスト内のパラメータなどを特定するために使用されます。その高速性と「FUZZ」キーワードを用いた自由度の高い配置により、モダンなWebセキュリティ調査やOSCP試験において「必須ツール」としての地位を確立しています。

---

## 🛠 代表的な用途
- **Webコンテンツの列挙**: 非公開のディレクトリやバックアップファイル、管理パネルの特定。
- **インフラの偵察**: 同一IPアドレス上でホストされているサブドメインやVHostの発見。
- **APIセキュリティ調査**: 有効なAPIエンドポイントやメソッド、バージョンの列挙。
- **脆弱性パラメータの特定**: LFI、SQLi、RCEなどの入り口となるパラメータ名や値の探索。
- **認証突破の試行**: ログインフォームへの単純な辞書攻撃やパスワードスプレー。
- **OSCPでの利用ポイント**: 時間制限がある中で、Gobusterよりも柔軟にリクエストを構成（カスタムヘッダーの挿入など）して高速に列挙を行う際に多用されます。

---

## 🔍 基本コマンド
```bash
# 最も基本的なディレクトリ列挙
ffuf -u http://10.10.10.1/FUZZ -w /usr/share/wordlists/dirb/common.txt

# 複数のワードリストを使用して、ファイル名と拡張子を同時にファジング
ffuf -u http://10.10.10.1/W1W2 -w names.txt:W1 -w extensions.txt:W2

# 特定のステータスコード（200 OK）のみを出力
ffuf -u http://10.10.10.1/FUZZ -w common.txt -mc 200
```

---

## 🧪 よく使う攻撃シナリオ
- **VHost列挙による隠しサイト発見**: 公開DNSには登録されていないが、内部で稼働している開発環境や管理サイトをHostヘッダーの操作により特定します。
- **APIパラメータの全数調査**: GET/POSTパラメータをファジングして、ドキュメント化されていないデバッグ用パラメータや権限昇格に繋がる隠しオプションを探します。
- **WAF/IPSのフィルタリング回避テスト**: 特定の記号やキーワードをリクエストに混ぜ、サーバーがどのパターンで403（拒否）を返すか、あるいは200（通過）を返すかを分析します。

---

## 📌 フェーズ別コマンド例  

### 🔎 Reconnaissance（技術的偵察）
```bash
# サブドメインの存在確認（DNS列挙）
ffuf -u http://FUZZ.target.local -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Hostヘッダーを操作して仮想ホスト（VHost）を特定（IP直接指定）
ffuf -u http://10.10.10.1 -H "Host: FUZZ.target.local" -w vhost-list.txt

# ページタイトルから特定のサービスやCMSを特定（-v で詳細表示）
ffuf -u http://target.local/FUZZ -w common.txt -r -v
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# 特定の拡張子（.php, .txt, .bak）を持つファイルを探索
ffuf -u http://$IP/FUZZ -w common.txt -e .php,.txt,.bak

# APIのバージョン番号をファジングして有効なパスを特定
ffuf -u http://$IP/api/vFUZZ/users -w numbers.txt

# 200 OK 以外の有益なコード（301リダイレクト, 403アクセス拒否）を含めて表示
ffuf -u http://$IP/FUZZ -w common.txt -mc 200,301,403

# APIで使用可能なHTTPメソッド（GET, POST, PUT, DELETE等）を特定
ffuf -u http://$IP/api/v1/resource -X FUZZ -w methods.txt

# 見つかったディレクトリに対して再帰的にスキャンを実行
ffuf -u http://$IP/FUZZ -w common.txt -recursion -recursion-depth 2
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# ログインフォームに対するユーザー名の列挙
ffuf -u http://$IP/login -X POST -d "username=FUZZ&password=password123" -w usernames.txt -fc 200

# adminユーザーに対するパスワードスプレー（302リダイレクト成功を狙う）
ffuf -u http://$IP/login -X POST -d "username=admin&password=FUZZ" -w rockyou.txt -mc 302

# セッションCookieや認証トークンのファジング
ffuf -u http://$IP/admin -b "session=FUZZ" -w session-wordlist.txt
```

---

### 🚪 Initial Access（初期侵入）
```bash
# LFI（ローカルファイルインクルード）の脆弱なパラメータを探索
ffuf -u http://$IP/index.php?page=FUZZ -w lfi-wordlist.txt -fs 0

# コマンドインジェクションのポイント特定（ステータスコードの変化を確認）
ffuf -u http://$IP/search?q=test&cmd=FUZZ -w cmd-payloads.txt

# SQLインジェクションの脆弱なIDを特定（レスポンスサイズでフィルタ）
ffuf -u http://$IP/item.php?id=FUZZ -w sql-payloads.txt -fs 1250
```

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# 踏み台経由で設定したプロキシ（例：BurpやSOCKS）を通して内部Webをスキャン
ffuf -u http://internal.local/FUZZ -w common.txt -x http://127.0.0.1:8080
```

---

## 🧰 便利オプション一覧

| オプション | 説明 |
| :--- | :--- |
| <code>-u</code> | ターゲットURLを指定。キーワード <code>FUZZ</code> を配置した場所が置換されます。 |
| <code>-w</code> | ワードリストのパスを指定。<code>path:ALIAS</code> 形式で複数の辞書を使用可能です。 |
| <code>-mc</code> | 出力に含めるHTTPステータスコードを指定（例：<code>-mc 200,403</code>）。 |
| <code>-fc</code> | 出力から除外するHTTPステータスコードを指定（例：<code>-fc 404</code>）。 |
| <code>-fs</code> | 特定のレスポンスサイズ（Size）を持つ結果を除外。誤検知排除に必須。 |
| <code>-fw</code> | 特定の単語数（Words）を持つ結果を除外。 |
| <code>-fl</code> | 特定の行数（Lines）を持つ結果を除外。 |
| <code>-t</code> | スレッド数（同時接続数）を指定。デフォルトは40。 |
| <code>-X</code> | HTTPメソッド（GET, POST, PUT等）を明示的に指定。 |
| <code>-H</code> | カスタムヘッダーを追加（例：<code>-H "Authorization: Bearer FUZZ"</code>）。 |
| <code>-d</code> | POSTリクエスト等のメッセージボディのデータ。 |
| <code>-r</code> | 301/302リダイレクトを自動的に追跡。 |
| <code>-v</code> | 冗長表示。URLやリダイレクト先などを詳細に出力。 |
| <code>-o</code> | 結果を指定したファイル（JSON, CSV, MD等）に保存。 |

---

## 🚀 実践例

```bash
# 【1】誤検知を排除するため、存在しないページ(404等)のサイズを確認して除外
ffuf -u http://$IP/FUZZ -w common.txt -fs 1492

# 【2】複数の辞書を使い、ディレクトリパスとファイル名を同時に組み合わせて探索
ffuf -u http://$IP/W1/W2 -w dirs.txt:W1 -w files.txt:W2

# 【3】HostヘッダーでのVHost探索において、共通の403ページサイズを除外してヒットのみを抽出
ffuf -u http://$IP -H "Host: FUZZ.target.com" -w subdomains.txt -fs 2392

# 【4】GETパラメータ名をファジングして、システムが受け付ける「隠し引数」を特定
ffuf -u http://$IP/index.php?FUZZ=test -w param-names.txt -fw 82

# 【5】特定のヘッダー値（例: X-Forwarded-For）をファジングしてIP制限回避を試行
ffuf -u http://$IP/admin -H "X-Forwarded-For: FUZZ" -w ip-list.txt -mc 200

# 【6】APIスキャンで「401 Unauthorized」以外（アクセス可能なエンドポイント）を重点調査
ffuf -u http://$IP/api/FUZZ -w api-endpoints.txt -fc 401

# 【7】ログイン失敗時のメッセージサイズが変わる点を利用して、有効なユーザー名を特定
ffuf -u http://$IP/login -X POST -d "user=FUZZ&pass=test" -w users.txt -fs 1530

# 【8】.envや.git、.htaccessなどのドットファイルをワードリストの先頭に付与して探索
ffuf -u http://$IP/.FUZZ -w common.txt

# 【9】スループットが速すぎてエラーが出る場合、スレッド数を下げて実行
ffuf -u http://$IP/FUZZ -w common.txt -t 10

# 【10】特定のキーワードを含むレスポンスのみを表示（正規表現フィルタ）
ffuf -u http://$IP/FUZZ -w common.txt -mr "admin_panel"

# 【11】JSONボディ内の特定の値をファジングしてAPIの挙動をテスト
ffuf -u http://$IP/api/v1/config -X POST -H "Content-Type: application/json" -d '{"key": "FUZZ"}' -w values.txt

# 【12】全ての全ポート詳細スキャン後のWebサービスに対し、一括で出力結果をJSON保存
ffuf -u http://$IP/FUZZ -w common.txt -o result_web.json -of json
```

---

## 🧩 他ツールとの組み合わせ
- **Burp Suite** → `ffuf -x http://127.0.0.1:8080` を指定し、ffufの高速なリクエストをBurpの履歴に流し込み、後から手動で詳細解析を行う。
- **SecLists** → `/usr/share/seclists/` 内の豊富なディレクトリ・パラメータ・ペイロードリストを直接指定。
- **jq** → `ffuf ... -o json | jq -r '.results[] | select(.status == 200) | .url'` のように組み合わせて、成功したURLだけを抽出。
- **curl** → ffufで特定した怪しいエンドポイントに対し、curlで詳細なリクエスト（`-v`オプション等）を送り挙動を確認。

---

## 📝 注意点（OSCP 試験での使い方）
- **DoSリスク**: デフォルト（-t 40）は非常に高速ですが、ターゲットのスペックが低い場合にサービスが不安定になる可能性があります。試験環境の安定性を保つため、最初は `-t 10` 程度から始めるのが無難です。
- **ワイルドカードDNS**: 全てのサブドメインリクエストに 200 OK を返す設定がある場合、ffufは全てのワードを「成功」と見なします。必ず `-fs` でデフォルトページのサイズを除外してください。
- **認証情報**: POSTデータのファジング時、特殊文字（& や = など）が含まれる場合は、`--data-urlencode` のような配慮が必要な場合があります。

---

## 🔗 参考リンク
- 公式：[ffuf GitHub Repository](https://github.com/ffuf/ffuf)
- 解説：[ffuf Documentation Wiki](https://github.com/ffuf/ffuf/wiki)
- 辞書：[SecLists Project](https://github.com/danielmiessler/SecLists)
