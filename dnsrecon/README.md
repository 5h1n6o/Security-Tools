# dnsrecon

## 🎯 概要
dnsreconは、情報収集（Information Gathering）フェーズにおいて不可欠な、強力なPythonベースのDNS偵察ツールです。ターゲットドメインに対して、標準的なレコードの列挙、ゾーン転送（AXFR）の試行、サブドメインの総当たり（ブルートフォース）、逆引きDNS調査などを自動化します。ペネトレーションテストやOSCPにおいて、攻撃対象領域（Attack Surface）を特定し、組織のネットワーク構造をマッピングするための出発点として非常に重要です。

---

## 🛠 代表的な用途
- **DNSレコードの列挙**: A, AAAA, MX, SRV, TXTレコードなどの一括取得。
- **ゾーン転送の試行**: 構成ミスのあるDNSサーバーからドメイン情報を丸ごと取得。
- **サブドメインの総当たり**: 辞書ファイルを用いて、公開されていないホスト名を特定。
- **逆引きDNSスキャン**: 特定のIP範囲に対してPTRレコードを照会し、ホスト名を特定。
- **OSCPでの利用ポイント**: 列挙フェーズの初期段階で、ターゲットのドメイン名からIPアドレスの範囲や特定のサーバー（メールサーバー、ドメインコントローラー等）を迅速に特定するために使用されます。

---

## 🔍 基本コマンド
```bash
# 特定ドメインに対して標準的な列挙を実行
dnsrecon -d example.com -t std

# ターゲットドメインのネームサーバーからゾーン転送（AXFR）を試行
dnsrecon -d example.com -t axfr

# サブドメインの総当たり（ブルートフォース）を実行
dnsrecon -d example.com -D /usr/share/wordlists/dnsmap.txt -t brt
```

---

## 🧪 よく使う攻撃シナリオ
- **全ドメイン情報の漏洩確認**: DNSサーバーの設定不備を突き、ゾーン転送（AXFR）によって内部ホストを含む全ての情報を取得します。
- **隠し開発環境の特定**: 総当たり攻撃（brt）により、`dev.example.com` や `test.example.com` などの公開されていないが脆弱な可能性のある環境を特定します。
- **内部ネットワークのマッピング**: 特定されたIP範囲に対して逆引き（PTR）を実行し、ネットワーク内のサーバーの役割（例：`sql-server.internal`）を推測します。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
```bash
# 公開されている全ての一般的なレコード（MX, TXT, NS等）を一括収集
dnsrecon -d target.local -t std

# Googleを使用してサブドメインをパッシブに探索
dnsrecon -d target.local -t google
```

---

### 🔎 Reconnaissance（技術的偵察）
```bash
# ターゲットのネームサーバーを特定し、それぞれの接続性を確認
dnsrecon -d target.local -n 8.8.8.8

# キャッシュスヌーピングを利用して、特定のドメインが参照されているか確認
dnsrecon -t snoop -n <nameserver_ip> -D wordlist.txt
```

---

### 🧭 Enumeration（詳細調査）
```bash
# 辞書ファイルを使用してサブドメインを高速にブルートフォース
dnsrecon -d target.local -D /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t brt

# 特定のIPアドレス範囲（範囲指定）に対して逆引きDNS（PTR）を実行
dnsrecon -r 192.168.50.0-192.168.50.255

# 特定のCIDR範囲に対して逆引きDNSを実行
dnsrecon -r 192.168.50.0/24

# トップレベルドメイン（TLD）を拡張して、関連するドメイン（.net, .org等）を探索
dnsrecon -d target -t tld
```

---

### 🏢 Active Directory（AD）
```bash
# AD環境で重要なSRVレコード（LDAP, Kerberos等）を列挙し、DCを特定
dnsrecon -d corp.local -t srv

# 内部DNSサーバーから特定のドメインに関連する情報を全て取得
dnsrecon -d corp.local -t std --xml results.xml
```

---

## 🧰 便利オプション一覧
- `-d <domain>`：ターゲットドメインを指定。
- `-n <nameserver>`：使用する特定のネームサーバーを指定。
- `-t <type>`：スキャンタイプを指定（std, axfr, brt, srv, ptr, tld, google）。
- `-D <file>`：総当たりに使用する辞書ファイルを指定。
- `-r <range>`：逆引き用のIP範囲（開始-終了）またはCIDRを指定。
- `-a`：ゾーン転送試行時に、全てのネームサーバーに対して実行。
- `-j <file>`：結果をJSON形式で保存。
- `-c <file>`：結果をCSV形式で保存。
- `--threads <number>`：並列実行するスレッド数を指定。

---

## 🚀 実践例

```bash
# 【例1】ターゲットドメインに対する完全な初期DNS調査（標準レコード + AXFR）
dnsrecon -d megacorpone.com -t std,axfr

# 【例2】特定のクラスC IP範囲全体に対してホスト名を特定する逆引きスキャン
dnsrecon -r 51.222.169.0/24

# 【例3】大規模な辞書ファイルを使用し、ワイルドカードDNSのチェックを含めて総当たり
dnsrecon -d target.local -D /usr/share/wordlists/rockyou.txt -t brt --iw

# 【例4】ADドメインにおいて、ドメインコントローラーやグローバルカタログ等の重要サービスを特定
dnsrecon -d industries.internal -t srv

# 【例5】スキャン結果を後でパースしやすいようにJSON形式でファイルに書き出す
dnsrecon -d target.com -t std -j dns_report.json
```

---

## 🧩 他ツールとの組み合わせ
- **nmap** → `dnsrecon` で特定したIPアドレス範囲を `nmap -iL` で読み込み、ポートスキャンを実行します。
- **ffuf** → 発見したサブドメインの一覧を `ffuf -H "Host: FUZZ.target.com"` のように使用し、仮想ホスト（VHost）の列挙へ繋げます。
- **impacket** → `srv` レコードで見つかったドメインコントローラー（DC）のホスト名に対して、`GetNPUsers.py` などのAD攻撃を実行します。
- **whatweb** → 特定されたサブドメインのURLを `whatweb` に渡し、各サーバーの技術スタック（CMS、OS等）を特定します。

---

## 📝 注意点（OSCP 試験での使い方）
- **情報の確実な保存**: `dnsrecon` の出力は非常に多岐にわたるため、`-j` や `-c` を用いてログを保存し、後で整理できるようにしておくことが推奨されます。
- **ワイルドカードの罠**: 全てのサブドメインに対して応答を返す「Wildcard DNS」が有効な場合、総当たりの結果が膨大かつ無意味になることがあります。その場合は `--iw`（Ignore Wildcard）オプションの使用を検討してください。
- **権限の確認**: ゾーン転送（axfr）が成功することは稀ですが、成功した場合はターゲットのネットワーク構成が全て露呈するため、最初期に必ず試すべき項目です。

---

## 🔗 参考リンク
- 公式：[dnsrecon GitHub Repository](https://github.com/darkoperator/dnsrecon)
- 良質解説：[Kali Linux dnsrecon Tool Documentation](https://www.kali.org/tools/dnsrecon/)
