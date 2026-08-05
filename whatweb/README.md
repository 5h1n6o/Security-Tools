# whatweb

## 🎯 概要
WhatWebは、Webサイトで使用されている技術スタックを特定するための次世代型Webスキャナです。コンテンツ管理システム（CMS）、ブログプラットフォーム、統計/解析パッケージ、JavaScriptライブラリ、Webサーバー、埋め込みデバイスなど、1,700以上のプラグインを駆使してターゲットを識別します。ペネトレーションテストやOSCPにおいて、サービスの詳細調査（Enumeration）の初期段階でWebサーバーの指紋特定（Fingerprinting）を行うための標準的なツールです。

---

## 🛠 代表的な用途
- **技術スタックの特定**: WebサイトがWordPress、Apache、PHP、jQueryのどのバージョンを使用しているかを判別。
- **攻撃対象領域の評価**: 特定されたバージョン情報に基づき、既知の脆弱性（CVE）が存在するかを判断する材料を収集。
- **ネットワーク全体のプロファイリング**: 大規模なIP範囲をスキャンし、特定の技術（例：古いバージョンのIIS）を使用しているホストを抽出。
- **OSCPでの利用ポイント**: Nmapで80/443ポートを確認した後、ブラウザで開く前に「何が動いているか」を素早く把握するために使用されます。

---

## 🔍 基本コマンド
```bash
# 単一のドメイン/IPをスキャン
whatweb 10.10.10.1

# 詳細情報を表示（Verboseモード）
whatweb -v 10.10.10.1

# 複数のターゲットをスペース区切りで指定
whatweb google.com facebook.com 192.168.1.1
```

---

## 🧪 よく使う攻撃シナリオ
- **脆弱なCMSの発見**: `/wp-content/` などのディレクトリ構造を解析し、特定の脆弱なバージョンが動作しているWordPressサイトを特定します。
- **デフォルト管理画面の推測**: Webサーバー（例：Niktoで検出されるような既知の構成）を特定し、そこから `admin/` や `setup.php` などの存在を予測します。
- **WAF回避のためのバナー調査**: Webサーバーの応答ヘッダーを詳細に分析し、背後に隠れている実際の技術やWAFの存在を推測します。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
```bash
# 公開されているドメインに対してパッシブにスキャンを実行
whatweb --no-errors example.com

# 検索エンジンを介さず直接ターゲットのヘッダー情報を収集
whatweb -v --user-agent "Mozilla/5.0" target.local
```

---

### 🔎 Reconnaissance（技術的偵察）
```bash
# ネットワーク範囲（/24）に対してWebサービスが動いているホストを特定
whatweb 192.168.1.0/24

# 複数のIPが記載されたファイルから一括スキャン
whatweb -i targets.txt

# サブドメインのリストに対して技術特定を実行
whatweb -i subdomains.txt --log-brief results.txt
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# アグレッシブレベルを上げて（Lv.3）詳細なバージョン情報を取得
whatweb -a 3 10.10.10.1

# 非常に強力なスキャン（Lv.4）を実行し、全プラグインで調査
whatweb -a 4 10.10.10.1

# リダイレクトを自動的に追跡して最終的な移動先を調査
whatweb --follow-redirect=always http://10.10.10.1

# 特定のプラグイン（例：Apache）に絞って情報を抽出
whatweb -p Apache 10.10.10.1
```

---

### 🔑 Credential Access（認証情報探索）
基本的に関連なし。ただし、特定された技術（例：phpMyAdmin, Tomcat Manager）に基づいて、デフォルトの資格情報を試す判断材料になります。

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# プロキシ（例：SOCKS）を介して内部ネットワークのWebサーバーをスキャン
whatweb --proxy socks5://127.0.0.1:1080 http://internal.target.local
```

---

## 🧰 便利オプション一覧
- `-a <level>`: アグレッシブ度（1: Stealthy, 3: Aggressive, 4: Heavy）。デフォルトは1。
- `-i <file>`: スキャン対象のリストが記載されたファイルを指定。
- `-v`: 詳細な説明を含む出力を表示。
- `--log-brief`: 各ターゲットに対して1行の簡潔な結果を出力。
- `--log-xml <file>`: スキャン結果をXML形式で保存。
- `--no-errors`: エラーを表示せず、成功した結果のみをクリーンに出力。
- `--user-agent <UA>`: カスタムUser-Agentを指定して、スキャナであることを隠蔽。

---

## 🚀 実践例

```bash
# 【1】OSCPでWebポートを発見した直後の定石コマンド
whatweb -a 3 -v $IP

# 【2】ネットワーク全体のWeb技術を簡潔なリストとして保存
whatweb 192.168.50.0/24 --log-brief web_tech_list.txt

# 【3】特定のCMS（WordPress等）のプラグインやテーマを深掘り
whatweb -a 3 http://10.10.10.1/blog/ --p=WordPress

# 【4】HTTPS証明書の不備を無視してスキャンを強行
whatweb --no-errors https://10.10.10.1/

# 【5】WAFにブロックされにくいよう、アグレッシブ度を下げて実行
whatweb -a 1 target.local

# 【6】JSON形式で結果を保存し、後でjqコマンドでパース準備
whatweb 10.10.10.1 --log-json results.json

# 【7】特定のキーワード（例：PHP）を含むサイトのみをネットワーク内から探す
whatweb 192.168.1.0/24 | grep -i "PHP"

# 【8】複数のURLを並列で高速スキャン（xargsとの組み合わせ）
cat urls.txt | xargs -P 10 -I {} whatweb {}

# 【9】Cookieを使用して認証が必要な領域の技術を特定
whatweb --cookie "session=12345" http://target.local/admin/

# 【10】特定のヘッダーを付与してスキャン（例：ホストヘッダーの偽装）
whatweb -H "Host: dev.target.local" http://10.10.10.1/
```

---

## 🧩 他ツールとの組み合わせ
- **nmap** → `nmap -p80 --open -oG - 10.10.10.0/24 | awk '/open/{print $2}' | whatweb -i -` でWebが動いているホストのみを自動的にwhatwebに渡す。
- **searchsploit** → whatwebで特定した「Apache 2.4.49」などのバージョンをそのまま `searchsploit` に入力してエクスプロイトを検索。
- **ffuf** → `ffuf` で見つかったディレクトリ（例：`/backup`）に対し、`whatweb` を実行してそのディレクトリが特定の管理ツールかどうかを確認。
- **Burp Suite** → whatwebで見つかった特異なヘッダー（例：`X-Backend-Server`）をBurpのIntruderで操作する際のヒントにする。

---

## 📝 注意点（OSCP 試験での使い方）
- **アグレッシブ度の選択**: `-a 3` や `-a 4` はターゲットサーバーに対して大量のリクエストを送信するため、IDS/IPSに検知されるリスクや、サービスを不安定にさせる可能性があります。
- **バージョン情報の過信**: Webサーバーの設定（セキュリティによる隠蔽）によっては、意図的に嘘のバナーを返している場合があるため、他のスキャナ（Nikto等）の結果と照合することが重要です。
- **自動化と手動確認のバランス**: whatwebの結果でCMSが判明したら、すぐに `wpscan` や `droopescan` などの特化型スキャナへ移行するのが効率的です。

---

## 🔗 参考リンク
- 公式：[WhatWeb GitHub Repository](https://github.com/urbanadventurer/WhatWeb)
- 公式：[WhatWeb Wiki](https://github.com/urbanadventurer/WhatWeb/wiki)
- 良質解説：[Kali Linux WhatWeb Tool Documentation](https://www.kali.org/tools/whatweb/)
