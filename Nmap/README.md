# nmap（& NSE）

## 🎯 概要
Nmap（Network Mapper）は、ネットワーク探索、セキュリティ監査、サービスおよびOSの指紋特定を目的とした、オープンソースの強力なネットワークスキャナーです。ペネトレーションテストやOSCPにおいて、ターゲットの攻撃対象領域（Attack Surface）を特定するための最も基本的かつ不可欠なツールであり、偵察から脆弱性調査まで幅広いステージで活用されます。

---

## 🛠 代表的な用途
- **ホスト発見**: ネットワーク内で稼働しているライブホストを特定。
- **ポートスキャニング**: オープン、クローズ、またはフィルタリングされているポートの状態を調査。
- **サービス・バージョン検出**: 稼働しているソフトウェアとその正確なバージョンを特定。
- **OS検出**: ターゲットのオペレーティングシステムの種類やバージョンを推測。
- **脆弱性スキャン**: Nmap Scripting Engine (NSE) を使用して、既知の脆弱性を自動的に検出。
- **OSCPでの利用ポイント**: 最初のステップとして全ポートスキャン（`-p-`）を実行し、得られたサービス情報を元に `gobuster` や `searchsploit` などの次段階のツールへ繋げる起点となります。

---

## 🔍 基本コマンド
```bash
# 単一のIPアドレスをスキャン
nmap 10.10.10.1

# ホスト名でスキャン
nmap www.target.local

# 特定の範囲のIPアドレスをスキャン
nmap 10.10.10.1-20

# サブネット全体をスキャン
nmap 10.10.10.0/24

# テキストファイルからターゲットリストを読み込んでスキャン
nmap -iL targets.txt
```

---

## 🧪 よく使う攻撃シナリオ
- **サービス詳細の徹底調査**: `-sV -sC` を使用して、サービスバージョンとデフォルトスクリプトによる詳細情報を取得。
- **全ポートの網羅的確認**: 標準のトップ1000ポート以外の隠れたサービス（高位ポートの管理画面など）を見つけるため、`-p-` で全65535ポートを調査。
- **NSEによる自動脆弱性診断**: `vulners` や `vuln` カテゴリのスクリプトを実行し、ターゲットに存在するCVEを即座にリストアップ。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
```bash
# ドメイン名から公開されている情報を収集（WHOIS, ASN, ジオロケーション等）
nmap --script asn-query,whois,ip-geolocation-maxmind <target>

# ターゲットに関連するサブドメインの公開情報を探索
nmap --script dns-brute <domain>
```

### 🔎 Reconnaissance（技術的偵察）
```bash
# ネットワーク内の稼働ホストを確認（Pingスイープ）
nmap -sn -PE 10.10.10.0/24

# 特定のネットワーク範囲で稼働しているホストをリスト表示（スキャンなし）
nmap -sL 10.10.10.0/24

# 高速にオープンポートのみをリストアップ
nmap --open 10.10.10.1

# TCPとUDPを同時にスキャン（特権が必要）
sudo nmap -v -Pn -sU -sT -p U:53,111,137,T:21-25,80,443 10.10.10.1

# ネットワーク全体に対して特定のポート（例：80番）が空いているホストをスイープ
nmap -p 80 10.10.10.0/24 -oG web-hosts.txt

# トップ20のTCPポートを高速スキャンし、OS検出を試みる
nmap -sT -A --top-ports=20 10.10.10.1
```

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# 標準的なサービス・OS・デフォルトスクリプトスキャン（OSCP推奨）
nmap -A -v 10.10.10.1

# Webサーバー上の隠しディレクトリやファイルを列挙
nmap --script http-enum 10.10.10.1

# WebサービスのHTTPヘッダー情報を取得
nmap --script http-headers 10.10.10.1

# Webサービスのタイトル情報を一括取得
nmap --script http-title 10.10.10.0/24

# SMBの共有フォルダ、ドメイン情報、OS詳細を特定
nmap -p 445 --script "smb-os-discovery,smb-enum-shares,smb-enum-users" 10.10.10.1

# DNSサーバーからのゾーン転送（AXFR）を試行
nmap -p 53 --script dns-zone-transfer --script-args dns-zone-transfer.domain=example.com 10.10.10.1

# SNMPサービスのコミュニティ名やシステム情報を調査
nmap -sU -p 161 --script snmp-info,snmp-interfaces 10.10.10.1

# RPCBindサービスの詳細情報を列挙
nmap -p 111 --script rpcinfo 10.10.10.1

# MS SQLサーバーの詳細なインスタンス情報を取得
nmap -p 1433 --script ms-sql-info 10.10.10.1
```

### 🚪 Initial Access（初期侵入）
```bash
# サービスバージョンに基づき関連するCVEやエクスプロイトを表示
nmap -sV --script vulners 10.10.10.1

# ターゲットに対して既知の脆弱性（vulnカテゴリ）を一斉スキャン
nmap -sV --script vuln 10.10.10.1

# SSL/TLSの重大な脆弱性（Heartbleedなど）を個別にチェック
nmap -p 443 --script ssl-heartbleed 10.10.10.1

# FTPのAnonymous（匿名）ログイン許可を確認
nmap -p 21 --script ftp-anon 10.10.10.1

# 特定の製品（例：ClamAV）のRCE脆弱性をチェック
nmap --script clamav-exec 10.10.10.1
```

### 🔑 Credential Access（認証情報探索）
```bash
# SMBサービスに対してユーザー名・パスワードの総当たりを試行
nmap -p 445 --script smb-brute 10.10.10.1

# MySQLサーバーに対してデフォルトパスワードや空パスワードをテスト
nmap -p 3306 --script mysql-empty-password 10.10.10.1

# HTTPの基本認証に対してブルートフォースを実行
nmap --script http-brute 10.10.10.1
```

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# Proxychains経由で内部ネットワークをスキャン（Pingなし `-Pn` が必須）
proxychains nmap -vvv -sT --top-ports=20 -Pn 172.16.50.217

# MeterpreterのSOCKSプロキシ経由で内部ホストをスキャン
nmap -sT -Pn -p 80,443,445 <internal_ip>
```

### 📈 Privilege Escalation（権限昇格）
```bash
# 古いNmapがSUID権限で動作している場合、対話モードを起動
nmap --interactive

# Nmap対話モードからルートシェルを実行（上記コマンド実行後）
!sh
```

---

## 🧰 便利オプション一覧
- `-p-`：全ポート（1-65535）をスキャン。
- `-sS`：ステルスSYNスキャン（ルート権限が必要）。
- `-sV`：サービスのバージョン検出。
- `-sC`：デフォルトのNSEスクリプト群を実行。
- `-A`：OS検出、バージョン検出、スクリプトスキャン、Tracerouteを一括実行。
- `-Pn`：ホスト発見（Ping）をスキップ。FWでPingがブロックされている場合に必須。
- `-T4`：スキャン速度を「アグレッシブ」に設定（OSCPで一般的。T0-T5）。
- `-oA <file>`：正常出力、XML、grep形式の3種類すべてで結果を保存。
- `--reason`：ポートがその状態（open/closed/filtered）である理由を表示。
- `--script-help <script_name>`：NSEスクリプトの使用方法や詳細を表示。

---

## 🚀 実践例

```bash
# 【1】OSCPで最初に叩く「全ポート詳細スキャン」
nmap -v -p- -sV -sC -A -oN initial_scan.txt 10.10.10.1

# 【2】ネットワーク内の稼働端末を高速に特定（Pingスイープ）
nmap -sn -oG live_hosts.txt 10.10.10.0/24

# 【3】Webサーバーの脆弱性を優先的に調査
nmap -p 80,443 --script vuln,http-enum,http-title 10.10.10.1

# 【4】SMBサービスの全情報を根こそぎ収集
nmap -p 445 --script "smb-*" 10.10.10.1

# 【5】UDPポートの中から重要なサービス（DNS, SNMP等）を特定
sudo nmap -sU -p 53,161,162,69 -v 10.10.10.1

# 【6】IDS/IPSを回避するためのフラグメントスキャン
nmap -f -t 500 10.10.10.1

# 【7】デコイ（おとり）IPを使用してスキャン元を隠蔽
nmap -D 10.10.10.5,10.10.10.6,ME 10.10.10.1

# 【8】ターゲットのOSを強制的に推測（精度が低くても表示）
nmap -O --osscan-guess 10.10.10.1

# 【9】特定のNSEスクリプト（Heartbleed）の脆弱性診断
nmap -p 443 --script ssl-heartbleed.nse 10.10.10.1

# 【10】IPv6アドレスを指定したポートスキャン
nmap -6 -sT fe80::80a5:26f2:8db7:5d04%12

# 【11】MS SQLサーバーの詳細構成とバージョン特定
nmap -sU --script=ms-sql-info -p 1434 10.10.10.1

# 【12】スキャン結果をXMLで保存し、MetasploitやDradisにインポート準備
nmap -p- -sV -oX scan_results.xml 10.10.10.1

# 【13】低速かつ丁寧にスキャンしFW検知を回避（Politeモード）
nmap -T2 -p 1-1024 10.10.10.1

# 【14】バナー情報をより詳細に取得する外部スクリプトの利用
nmap --script banner-plus.nse -p- 10.10.10.1

# 【15】ネットワーク内の全インターフェースに対して一括スキャンを実行
ifconfig -a | grep -Po '\b(?!255)(?:\d{1,3}\.){3}(?!255)\d{1,3}\b' | xargs nmap -A -p-
```

---

## 🧩 他ツールとの組み合わせ
- **ffuf / Gobuster** → Nmapで見つかったポート80や8080のWebサービスに対し、ディレクトリ列挙を実行。
- **Metasploit (`db_nmap`)** → Metasploit内でNmapを実行し、結果をデータベースへ自動保存。
- **searchsploit** → 特定した「サービス名 バージョン」を元に、攻撃コードを検索。
- **Proxychains** → 踏み台を経由して内部ネットワークのポートスキャンを可能にする。
- **RustScan** → Nmapよりも高速にオープンポートを特定し、詳細調査をNmapに引き継ぐ。

---

## 📝 注意点（OSCP 試験での使い方）
- **タイミング（Timing）の選択**: `-T4` は多くの場合安全ですが、ネットワークが不安定な場合は `-T3` に落とす必要があります。
- **UDPスキャンの重要性**: TCPだけでなく、SNMP（161）やDNS（53）などのUDPポートに重要な情報が隠れていることが多いため、忘れずに実行してください。
- **証跡の保存**: 試験のレポート作成には詳細な証跡が必要です。必ず `-oA` または `-oN` で全結果をログに残す習慣をつけましょう。



---

## 🔗 参考リンク
- 公式：[Nmap Official Documentation](https://nmap.org/docs.html)
- NSEポータル：[Nmap Scripting Engine (NSE) Docs](https://nmap.org/nsedoc/)

---

## 🛠 NSEの基本概念
*   **場所**: Kali Linuxでは通常 `/usr/share/nmap/scripts/` に `.nse` ファイルとして格納されています。
*   **言語**: スクリプトは **Lua言語** で記述されています。
*   **データベース**: `script.db` ファイルが全スクリプトのインデックスとして機能します。

---

## 📌 NSEスクリプトカテゴリー
スクリプトはその目的やリスクに応じて以下のカテゴリーに分類されます。

| カテゴリー | 説明 |
| :--- | :--- |
| **auth** | ターゲットの認証回避やデフォルト認証情報の利用を試みます。 |
| **brute** | HTTP、SNMP、MySQL、VNCなどへのパスワード総当たりを行います。 |
| **default** | 安全で信頼性が高く、`-sC` オプションで実行される標準セットです。 |
| **discovery** | SNMPやディレクトリサービスなどから詳細な情報を探索します。 |
| **vuln** | ターゲットに既知の脆弱性が存在するかをチェックします。 |
| **exploit** | 脆弱性を実際に悪用しようと試みます。 |
| **safe** | ターゲットの安定性に影響を与えない、リスクの低いスクリプトです。 |
| **intrusive** | ターゲットをクラッシュさせたりリソースを消費させたりする可能性のあるノイジーな調査です。 |
| **malware** | ターゲットがマルウェアに感染している兆候を探します。 |

---

## 🔍 NSEの基本コマンド・管理
```bash
# 特定のスクリプトに関するヘルプと詳細情報を表示
nmap --script-help <script_name>

# 特定のカテゴリー（例: vuln）に属する全スクリプトを一覧表示
cat /usr/share/nmap/scripts/script.db | grep "\"vuln\""

# 新しく追加したスクリプトを認識させるためにデータベースを更新
sudo nmap --script-updatedb

# スクリプトに引数（ドメイン名や認証情報など）を渡して実行
nmap --script <script_name> --script-args <name=value>
```

---

## 🧪 フェーズ別・サービス別実践例

### 🧭 Enumeration（詳細調査）
```bash
# [HTTP] サーバーがサポートするHTTPヘッダーを確認
nmap --script http-headers <target>

# [HTTP] robots.txt ファイルから隠しパスを抽出
nmap --script http-robots.txt <target>

# [SMB] Windowsのバージョン、ドメイン名、共有フォルダーを特定
nmap -p 445 --script "smb-os-discovery,smb-enum-shares" <target>

# [DNS] DNSサーバーからゾーン転送（AXFR）を試行
nmap -p 53 --script dns-zone-transfer --script-args dns-zone-transfer.domain=<domain> <target>

# [NFS] 公開されているNFS共有ディレクトリとファイルを列挙
nmap -p 111 --script nfs-ls <target>
```

### 🚪 Initial Access（脆弱性スキャン・初期侵入）
```bash
# [脆弱性診断] 検出されたバージョンをVulnersデータベースと照合（CVE, CVSSを表示）
nmap -sV --script vulners <target>

# [脆弱性診断] ターゲットに対して「vuln」カテゴリの全スクリプトを実行
nmap --script vuln <target>

# [SSL/TLS] 重大な脆弱性 Heartbleed の有無をチェック
nmap -p 443 --script ssl-heartbleed <target>

# [FTP] Anonymous（匿名）ログインが許可されているか確認
nmap -p 21 --script ftp-anon <target>

# [ClamAV] 未認証のコマンド実行脆弱性が存在するか確認
nmap --script clamav-exec <target>
```

### 🔑 Credential Access（認証情報探索）
```bash
# [SMB] ユーザー名とパスワードの組み合わせをブルートフォース
nmap -p 445 --script smb-brute <target>

# [MySQL] パスワードが設定されていないrootアカウントを探す
nmap -p 3306 --script mysql-empty-password <target>

# [HTTP] 基本認証（Basic Auth）に対して辞書攻撃を実行
nmap --script http-brute <target>
```

---

## 🚀 実践的なヒント
*   **脆弱性PoCの確認**: `vulners` スクリプトの出力には、**「*EXPLOIT*」** とマークされた公開済みの概念実証（PoC）コードへのリンクが含まれることがあります。
*   **独自スクリプトの導入**: インターネットで見つけた特定のCVE用スクリプト（例: `http-vuln-cve2021-41773.nse`）は、scriptsディレクトリにコピーして `updatedb` を実行することで即座に使用可能です。
*   **安全性の確認**: 本番環境で `intrusive` や `dos` カテゴリを実行すると、**サービスを停止させる恐れ**があります。実行前に必ず `--script-help` で挙動を確認してください。

---

## 🧩 他ツールとの連携
*   **Metasploit**: Nmapで特定した脆弱性（CVE番号）をMetasploitの `search` コマンドに入力することで、適切な攻撃モジュールを素早く発見できます。
*   **SearchSploit**: サービスバージョン情報を `searchsploit` に渡して、ローカル環境でエクスプロイトを検索する際の精度を高めます。

---

## 🔗 参考リンク
*   公式ドキュメント (NSEDoc): [https://nmap.org/nsedoc/](https://nmap.org/nsedoc/)
*   Nmap Scripting Engineマニュアル: [https://nmap.org/boo
