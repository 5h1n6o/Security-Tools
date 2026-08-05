# gobuster

## 🎯 概要
**Gobuster**は、Go言語で記述された、URI（ディレクトリやファイル）やDNSサブドメイン、仮想ホスト（VHost）を高速に探索するためのオープンソースのブルートフォース（総当たり）ツールです。ペネトレーションテストやOSCPにおいて、ターゲットのWebサーバー上に存在する「隠しディレクトリ」や「バックアップファイル」、「公開されていないAPIエンドポイント」を特定する**列挙（Enumeration）**フェーズで最も多用されるツールの一つです。Go製であるため非常に軽量かつ並列処理に優れており、限られた時間内で広範な攻撃対象領域（Attack Surface）をマッピングする際に真価を発揮します。

---

## 🛠 代表的な用途
- **Webディレクトリ・ファイルの列挙**: 公開されていない管理画面（`/admin`）や設定ファイル（`.env`）の特定。
- **DNSサブドメインの探索**: 組織が所有する未公開のサブドメインや開発環境の発見。
- **APIエンドポイントの発見**: パターンファイルを用いたAPIバージョンやリソースの特定。
- **仮想ホスト（VHost）の特定**: 同一IPアドレス上で動作する複数のドメインの判別。
- **OSCPでの利用ポイント**: 列挙の初期段階で必ずと言っていいほど実行されます。特に、Nmapで見つかったWebポートに対して、標準的なワードリスト（`common.txt`や`directory-list-2.3-medium.txt`）を用いて構造を把握するために使用されます。

---

## 🔍 基本コマンド
```bash
# 基本的なディレクトリ探索（dirモード）
gobuster dir -u http://10.10.10.1/ -w /usr/share/wordlists/dirb/common.txt

# DNSサブドメインの探索（dnsモード）
gobuster dns -d target.local -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# VHost（仮想ホスト）の探索（vhostモード）
gobuster vhost -u http://target.local -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## 🧪 よく使う攻撃シナリオ
- **隠しバックアップ・設定ファイルの特定**: `-x` オプションで `php,bak,txt,zip,old` などの拡張子を指定し、開発者が残した機密ファイルを一斉に検索します。
- **API構造の総当たり**: `{GOBUSTER}/v1` のようなプレースホルダを含むパターンファイルを使用し、APIのバージョンや階層構造を効率的に特定します。
- **ディレクトリ階層の深掘り**: 特定されたディレクトリ（例：`/secret`）に対して再度Gobusterを実行し、ネストされた隠しリソースを段階的に暴きます。

---

## 📌 フェーズ別コマンド例  

### 🔎 Reconnaissance（技術的偵察）
```bash
# DNSのブルートフォースにより組織の公開されていないホスト名を特定
gobuster dns -d example.com -w /usr/share/wordlists/dnsmap.txt

# IPアドレスを直接叩き、Hostヘッダーを操作して内部向けの仮想ホストを発見
gobuster vhost -u http://10.10.10.1 -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# 標準的なワードリストを用いて共通のディレクトリパスを列挙
gobuster dir -u http://192.168.50.20/ -w /usr/share/wordlists/dirb/common.txt

# 5つのスレッドを指定し、サーバー負荷を抑えつつディレクトリを探索
gobuster dir -u http://192.168.50.20 -w /usr/share/wordlists/dirb/common.txt -t 5

# 拡張子を指定して、特定のファイル形式（PHP, PDF, Config）に絞ってスキャン
gobuster dir -u http://192.168.50.242 -w /usr/share/wordlists/dirb/common.txt -x txt,pdf,config

# パターンファイルを利用してAPIエンドポイント（例：/api/v1/users）を高速特定
gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern_file
```

---

### 🚪 Initial Access（初期侵入）
```bash
# 8080ポートで動作するテスト環境などに対し、ログイン画面やバックアップファイルを探索
gobuster dir -u http://192.168.56.107:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,html,txt

# robots.txtで拒否されている可能性のある隠しディレクトリを手動リストで再確認
gobuster dir -u http://target.local/ -w custom_list.txt
```

---

### 📈 Privilege Escalation（権限昇格）
基本的に関連なし。ただし、侵入後にローカルのWeb管理インターフェース（127.0.0.1のみ許可されているものなど）の構造を調査する際に、ポートフォワーディングと組み合わせて使用されることがあります。

---

### 🏢 Active Directory（AD）
基本的に関連なし。ただし、AD環境内の内部Webサイトや、ドメインコントローラーがホストしている公開ディレクトリの列挙に使用されることがあります。

---

## 🧰 便利オプション一覧
- `dir`：ディレクトリ・ファイル列挙モード。
- `dns`：DNSサブドメイン列挙モード。
- `vhost`：仮想ホスト列挙モード。
- <code>-u</code>：ターゲットURL。
- <code>-w</code>：ワードリスト（辞書）のパス。
- <code>-x</code>：検索対象の拡張子リスト（例：`php,txt,html`）。
- <code>-t</code>：実行スレッド数（デフォルト10）。速度を上げたい場合は増やす。
- <code>-p</code>：パターンファイルのパス。`{GOBUSTER}`プレースホルダを使用。
- <code>-o</code>：結果を指定したファイルに出力。
- <code>-k</code>：SSL/TLSの証明書検証をスキップ（自己署名証明書の場合に必須）。
- <code>-s</code>：ポジティブなステータスコードの指定（例：`200,204,301,302,307,403`）。
- <code>-b</code>：除外したいステータスコード（例：`404`）。

---

## 🚀 実践例

```bash
# 【1】OSCP環境でよく使われる標準的な高速ディレクトリ列挙
gobuster dir -u http://192.168.56.112:80/ -w /usr/share/wordlists/dirb/common.txt -t 20

# 【2】特定のWebアプリ（WordPress等）の構成ファイルを拡張子付きで探索
gobuster dir -u http://target.local/blog -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,js

# 【3】APIサーバーのバージョンアップに伴うエンドポイントの変更をパターンで特定
# pattern_fileの内容: {GOBUSTER}/v1, {GOBUSTER}/v2
gobuster dir -u http://api.target.local -w words.txt -p pattern_file

# 【4】サブディレクトリで見つかった「/secret」などを起点に深掘りスキャン
gobuster dir -u http://192.168.56.109/secret/ -w /usr/share/wordlists/dirb/common.txt -x php,txt

# 【5】403 Forbidden（アクセス拒否）が出るパスも、リソースが存在する証拠として記録
gobuster dir -u http://target.local -w common.txt -s "200,204,301,302,403"

# 【6】自己署名証明書（HTTPS）を使用している古いサーバーを強制スキャン
gobuster dir -u https://192.168.1.50 -w /usr/share/wordlists/dirb/common.txt -k

# 【7】巨大なワードリストを使用し、出力を後で解析するために保存
gobuster dir -u http://target.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster_results.txt

# 【8】.htaccess や .htpasswd などのドットファイルを重点的に探索
gobuster dir -u http://target.local -w /usr/share/wordlists/dirb/common.txt -a "Mozilla/5.0"

# 【9】特定のディレクトリ配下にバックアップファイル（.bak, .old）がないか一斉調査
gobuster dir -u http://target.local/config/ -w /usr/share/wordlists/dirb/common.txt -x bak,old,save,tmp

# 【10】環境変数 $URL を用いたスクリプト内での自動列挙
gobuster dir -u $URL -w /usr/share/wordlists/dirb/common.txt
```

---

## 🧩 他ツールとの組み合わせ
- **ffuf** → Gobusterはパスの探索に特化している一方、ffufはパラメータやヘッダーのより複雑なファジングに適しています。パス探索はGobuster、パラメータ特定はffufと使い分けるのが効率的です。
- **nmap** → Nmapの `-sV` オプションでWebサーバーを特定した後、そのポートをGobusterに引き継いで内部構造を明らかにします。
- **Nikto** → NiktoでWebサーバーの全体的な脆弱性や既知の構成ミスをスキャンした後、GobusterでNiktoの辞書に載っていない独自の隠しリソースを深掘りします。
- **Burp Suite** → Gobusterで特定した「ステータスコード200や403」のURLをBurpのRepeaterに送り、手動で詳細なリクエスト分析を行います。

---

## 📝 注意点（OSCP 試験での使い方）
- **速度の調整**: デフォルトのスレッド数（`-t 10`）は多くの場合安全ですが、ターゲットが低スペックな仮想マシンの場合、リクエストが速すぎるとDoS状態になりサービスが落ちる（あるいは自分のIPがBANされる）ことがあります。応答が不安定な場合はスレッド数を下げてください。
- **ワードリストの選定**: 最初から巨大なリスト（`directory-list-2.3-medium.txt`など）を使うと時間がかかりすぎます。まずは `common.txt` などの軽量なリストで構造を把握するのがセオリーです。
- **ワイルドカードDNS**: DNSモードで全てのドメインが同一IPを返す「ワイルドカード設定」がされている場合、Gobusterは全ての結果を「ヒット」として返してしまいます。この場合は誤検知をフィルタリングする設定が必要です。

---

## 🔗 参考リンク
- 公式：[Gobuster GitHub Repository](https://github.com/OJ/gobuster)
- 良質解説：[Gobuster Usage Guide (OJ Reeves)](https://github.com/OJ/gobuster#usage)

---

## 📝 補足1（Notes）

Webアプリケーションの偵察や列挙フェーズで頻用される**ffuf**と**gobuster**について、それぞれの違いや得意とするポイント、使いどころを比較表にまとめました。

| 比較項目 | **gobuster** | **ffuf** |
| :--- | :--- | :--- |
| **ツールの本質** | **列挙（Enumeration）**に特化したブルートフォースツール | 非常に高速かつ柔軟な**Webファジング**フレームワーク |
| **開発言語** | **Go言語**（高速・軽量） | **Go言語**（極めて高速） |
| **得意な探索** | ディレクトリ、ファイル、DNS、VHostの**高速な総当たり** | パラメータ名、ヘッダー値、POSTデータなど、**リクエストのあらゆる箇所のファジング** |
| **柔軟性** | `dir`、`dns`、`vhost`などの**モードベース**で使い勝手がシンプル | `FUZZ`キーワードを任意の位置に配置でき、**リクエストの微調整**が容易 |
| **主な用途** | ターゲットのWebサイト構造（`/admin`など）やAPIパスの**迅速なマッピング** | 隠しパラメータの発見、複雑なリクエストが必要な脆弱性検証、APIの全数調査 |
| **OSCP/試験での利点** | 列挙の初期段階で全ポートスキャン後のパス特定を**素早く行う**のに最適 | Burp Intruder（無料版）の**速度制限を回避する代替手段**として非常に強力 |

---

### それぞれの使いどころとポイント

#### **gobuster の使いどころ**
*   **初期調査の迅速化**: Webサーバーを発見した直後、どのようなディレクトリやファイルが公開されているか、標準的なワードリストを使って**短時間で構造を把握**したい場合に最適です。
*   **APIエンドポイントの特定**: `{GOBUSTER}`というプレースホルダとパターンファイルを使用することで、`/api/v1/resource`のような**バージョン管理されたAPIパス**を効率的に列挙できます。
*   **特定モードの活用**: DNSモードを用いて、組織の**未公開サブドメイン**を高速に列挙する際にも威力を発揮します。

#### **ffuf の使いどころ**
*   **複雑なリクエストの検証**: HTTPリクエストのヘッダー（`X-Forwarded-For: FUZZ`など）やCookie、メッセージボディを**動的に変更しながらテスト**したい場面で非常に強力です。
*   **隠しパラメータの発見**: GETリクエストのパラメータ名自体をファジングして、ドキュメント化されていない**隠しオプションやデバッグ用引数**を探すのに適しています。
*   **大規模な辞書攻撃**: Burp Suite CEのIntruderよりもはるかに高速にリクエストを送信できるため、**数万ワード規模の辞書**を用いた列挙を無料で行いたい場合に推奨されます。

### 結論
単純にWebサイトの**隠しパスやDNS情報をリストアップ**したいなら**gobuster**、パラメータの特定やカスタムヘッダーの操作など、**高度で柔軟なリクエスト改ざん**が必要なら**ffuf**を使用するのが最も効率的です。

---

## 📝 補足2（Notes）

Gobusterは、Go言語で開発された高速な総当たり（ブルートフォース）ツールであり、主に**Webディレクトリの列挙（dirモード）**と**DNSサブドメインの列挙（dnsモード）**に使用されます。

これら2つのモードの主な違いを詳しく解説します。

### 1. dirモード (Directory Enumeration)
**dirモード**は、Webサーバー上の公開されているがリンクされていないファイルやディレクトリを特定するために使用されます。

*   **目的:** ターゲットWebサイトの「隠しパス」や「バックアップファイル」、「管理パネル」などを発見すること。
*   **仕組み:** 指定されたワードリストの各単語をベースURLの末尾に付加し、HTTPリクエストを送信します。
*   **主なオプション:**
    *   <code>-u</code>: ターゲットの**URL**を指定します。
    *   <code>-w</code>: 使用する**ワードリスト**を指定します。
    *   <code>-x</code>: 特定の**拡張子**（php, txt, pdfなど）を持つファイルを探索します。
    *   <code>-b</code>: 特定のHTTPステータスコード（例: 404）を除外します。
*   **確認ポイント:** サーバーからの応答に含まれる**HTTPステータスコード**（200 OK, 301/302 Redirect, 403 Forbiddenなど）を確認して、リソースの存在を判断します。

### 2. dnsモード (DNS Subdomain Enumeration)
**dnsモード**は、ターゲットドメインに対して存在しうるサブドメインを総当たりで調査するために使用されます。

*   **目的:** 組織が所有する「公開されていないサブドメイン」や「開発用環境」などのホストを特定すること。
*   **仕組み:** ワードリストにある単語をドメイン名の前に付加（例: `FUZZ.example.com`）し、DNSクエリを送信して名前解決ができるかを確認します。
*   **結果:** 解決に成功したサブドメインとその**IPアドレス**がリストアップされます。これにより、当初は知られていなかった新しい攻撃対象ホストを発見できます。

---

### モード別の主な違い比較表

| 比較項目 | **dirモード** | **dnsモード** |
| :--- | :--- | :--- |
| **調査対象** | Webサーバー内のパス（ディレクトリ・ファイル） | ドメインに属するサブドメイン（ホスト） |
| **使用プロトコル** | **HTTP / HTTPS** | **DNS** |
| **ターゲット指定** | URL（例: `http://192.168.50.20/`） | ドメイン名（例: `megacorpone.com`） |
| **主な結果** | HTTPステータスコード（200, 403等） | IPアドレスの名前解決結果 |
| **拡張子の探索** | 可能（`-x`オプションを使用） | 基本的に不要（ホスト名を探索するため） |

### 具体的な使い分け
*   **dirモードを使う場面:** Nmap等でポート80や443が開いているWebサーバーを見つけた際、その**Webアプリケーション内部の構造を調査**したい場合に使用します。
*   **dnsモードを使う場面:** ターゲット組織の全体像を把握する偵察フェーズで、**他にどのようなサーバー（ホスト）がネットワーク上に存在するか**を特定したい場合に使用します。

Gobusterは非常に高速ですが、総当たりの性質上**大量のトラフィックを生成する**ため、検知を避けたい環境ではスレッド数（`-t`）を調整するなどの注意が必要です。
