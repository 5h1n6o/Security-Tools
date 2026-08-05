# rpcclient

## 🎯 概要
**rpcclient** は、Sambaスイートに含まれる、MS-RPC（Microsoft Remote Procedure Call）エンドポイントを操作するための強力なクライアントツールです。ペネトレーションテストやOSCPにおいて、主にWindowsホストやSambaサーバーの**列挙（Enumeration）**フェーズで使用されます。TCP 135ポート（MSRPC）や445ポート（SMB）を介して通信し、匿名ログイン（Null Session）または有効な資格情報を用いて、ユーザー一覧、グループ構成、ドメイン情報、共有の権限、パスワードポリシーなどの機密情報を非対話的、あるいは対話的インターフェースで抽出します。

---

## 🛠 代表的な用途
- **ユーザーとグループの列挙**: ドメインやローカルに存在するユーザー名、SID、RID、説明文の取得。
- **ドメイン情報の収集**: ドメインSIDの特定（権限昇格やGolden Ticket作成の準備）。
- **共有リソースの確認**: 非公開共有のリストアップ。
- **パスワードポリシーの特定**: ロックアウトの閾値や最小文字数を確認し、ブルートフォースの戦略を策定。
- **OSCP での利用ポイント**: 認証情報を持っていない初期段階で、匿名ログインを試行して有効なユーザー名リスト（Wordlist）を作成する際に極めて重要です。

---

## 🔍 基本コマンド
```bash
# 匿名（Null Session）で接続
rpcclient -U "" -N <TARGET_IP>

# ユーザー名を指定して接続（パスワードが求められます）
rpcclient -U "username" <TARGET_IP>

# NTハッシュを使用して接続（Pass-the-Hash）
rpcclient -U "username" --pw-nt-hash <NT_HASH> <TARGET_IP>

# 接続後に特定のRPCコマンドを実行（非対話モード）
rpcclient -U "" -N <TARGET_IP> -c "enumdomusers"
```

---

## 🧪 よく使う攻撃シナリオ
- **Null Sessionによる情報漏洩**: 認証なしで `enumdomusers` や `querydispinfo` を実行し、全ユーザー名を取得。
- **RIDサイクリング**: 既知のSIDの末尾（RID）をインクリメントしながら `queryuser` を実行し、隠れた管理者アカウントやサービスアカウントを特定。
- **パスワードポリシーの確認**: `getdompwinfo` でロックアウト設定を確認し、アカウントをロックさせずにパスワードスプレー攻撃（Hydra等）を行う。

---

## 📌 フェーズ別コマンド例  
Pentest‑Playbook の各フェーズで実際に使うコマンド例を記載します。  
書式は **「# 説明 → コマンド例」** の形式で統一します。

---

### 🔎 Reconnaissance（技術的偵察）
```bash
# Nmapスキャンで135ポートの開放を確認した後の生存確認
rpcclient -U "" -N <TARGET_IP> -c "srvinfo"

# サーバーのバナー情報（OS、アーキテクチャ）を取得
rpcclient -U "" -N <TARGET_IP> -c "srvinfo"
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# ドメインユーザーを一覧表示（ユーザー名とRID）
rpcclient -U "" -N <TARGET_IP> -c "enumdomusers"

# ドメイングループを一覧表示
rpcclient -U "" -N <TARGET_IP> -c "enumdomgroups"

# 共有フォルダの情報を詳細に列挙
rpcclient -U "" -N <TARGET_IP> -c "netshareenumall"

# 特定のユーザーRID（例: 500）の詳細情報を取得
rpcclient -U "" -N <TARGET_IP> -c "queryuser 500"

# ドメインのパスワードポリシー（最小長、有効期間など）を表示
rpcclient -U "" -N <TARGET_IP> -c "getdompwinfo"

# サーバーに存在する全共有名をリストアップ
rpcclient -U "" -N <TARGET_IP> -c "netshareenum"

# RPCエンドポイントマップをルックアップして利用可能なインターフェースを特定
rpcclient -U "" -N <TARGET_IP> -c "epmlookup"
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# ドメインのSIDを取得（ハッシュ解析や偽造チケット作成に必要）
rpcclient -U "user" -P "pass" <TARGET_IP> -c "lsaquery"

# エイリアスグループ（ローカルグループ）のメンバーを確認
rpcclient -U "user" -P "pass" <TARGET_IP> -c "enumalsgroups domain"

# ユーザーの説明欄（Description）を一括取得（パスワードが含まれている場合がある）
rpcclient -U "" -N <TARGET_IP> -c "querydispinfo"
```

---

### 🚪 Initial Access（初期侵入）
```bash
# 取得したユーザーリスト（Wordlist）を用いて、特定のアカウントのパスワード変更を試行
rpcclient -U "Administrator" <TARGET_IP> -c "setuserinfo2 username 24 'new_password'"
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# サーバー上の特権（Privileges）を列挙
rpcclient -U "username" -P "password" <TARGET_IP> -c "enumprivs"

# 特定のユーザーに割り当てられた権限を調査
rpcclient -U "username" -P "password" <TARGET_IP> -c "lsaenumprivsaccount"
```

---

### 🏢 Active Directory（AD）
```bash
# ドメインコントローラーへの信頼関係を確認
rpcclient -U "domain_user" <TARGET_IP> -c "dsroamdamin"

# ADドメイン内のコンピュータアカウントを列挙
rpcclient -U "domain_user" <TARGET_IP> -c "enumdomadcomms"
```

---

## 🧰 便利オプション一覧
- `-U <user>`：接続に使用するユーザー名。`user%password` の形式でパスワードも指定可能。
- `-N`：パスワードなし（No password）で接続。
- `-c "<command>"`：接続後に指定したコマンドを実行して即座に終了。
- `-W <workgroup>`：ワークグループまたはドメインを指定。
- `-I <IP>`：接続先IPを直接指定（ホスト名解決が不安定な場合）。
- `--pw-nt-hash`：パスワードの代わりにNTハッシュを使用して認証（PtH）。

---

## 🚀 実践例（10個以上）

```bash
# 【1】認証なしでドメインSIDを取得（OSCPでの基本）
rpcclient -U "" -N 192.168.50.101 -c "lsaquery"

# 【2】匿名ログインで全ユーザー名を抽出し、Wordlistを作成するための準備
rpcclient -U "" -N 192.168.50.101 -c "enumdomusers" | awk -F'[' '{print $2}' | cut -d']' -f1

# 【3】querydispinfoを使用して、各ユーザーのアカウント説明（Description）を確認
rpcclient -U "" -N 192.168.50.101 -c "querydispinfo"

# 【4】RIDサイクリング用のBashループ（1000から1100までのユーザーを強制列挙）
for rid in $(seq 1000 1100); do rpcclient -U "" -N 192.168.50.101 -c "queryuser $rid" | grep "User Name" && echo "RID: $rid"; done

# 【5】ドメインのロックアウトポリシー（閾値）を特定し、パスワードスプレーの安全性を確認
rpcclient -U "" -N 192.168.50.101 -c "getdompwinfo"

# 【6】接続先のホスト名とOSバージョンを精密に特定
rpcclient -U "" -N 192.168.50.101 -c "srvinfo"

# 【7】共有フォルダのリストとそれぞれのパス、コメントを確認
rpcclient -U "" -N 192.168.50.101 -c "netshareenumall"

# 【8】ドメイングループ「Domain Admins」のメンバーRIDを特定
rpcclient -U "user" -P "pass" 192.168.50.101 -c "enumdomgroups"

# 【9】特定のグループ（例: 512 = Domain Admins）の所属ユーザーを特定
rpcclient -U "user" -P "pass" 192.168.50.101 -c "querygroupmem 512"

# 【10】特定のIP範囲に対して匿名接続が可能かスイープ（スクリプト内での利用例）
rpcclient -U "" -N 192.168.50.101 -c "srvinfo" > /dev/null 2>&1 && echo "Vulnerable to Null Session"

# 【11】NTハッシュを用いたPass-the-Hashによる管理者権限情報の列挙
rpcclient -U "Administrator" --pw-nt-hash 7a38310ea6f0027ee955abed1762964b 192.168.50.101 -c "enumdomusers"

# 【12】プリンタ共有情報を列挙（システム構成の推測に利用）
rpcclient -U "" -N 192.168.50.101 -c "enumprinters"
```

---

## 🧩 他ツールとの組み合わせ
- **nmap** → `nmap -p 135,445` でポート開放を確認後、`rpcclient` で詳細調査を開始。
- **enum4linux-ng** → `enum4linux-ng` が内部で `rpcclient` を呼び出し、結果を自動で整形して表示。
- **Hydra** → `rpcclient` で得たユーザーリスト（`enumdomusers`）を `hydra` の `-L` オプションに渡し、パスワードスプレーを実施。
- **impacket** → `rpcclient` で特定したドメインSIDを、`impacket-ticketer` でのチケット偽装に利用。

---

## 📝 注意点（OSCP 試験での使い方）
- **空のユーザー情報の罠**: `enumdomusers` で結果が返ってこない場合でも、`queryuser` でRIDを直接指定すると情報が得られる「隠れた」Null Session環境があります。RIDサイクリングは必ず試すべきです。
- **認証エラーの確認**: `NT_STATUS_ACCESS_DENIED` が出る場合は、匿名アクセスが制限されています。その際は別のサービス（HTTP、SMB共有内のファイル等）から資格情報を入手することを優先してください。
- **対話モードの活用**: コマンドライン引数（`-c`）で実行するだけでなく、`rpcclient` の対話プロンプト内で `Tab` キーによる補完を使い、利用可能なコマンドをその場で探すのが効率的です。

---

## 🔗 参考リンク
- 公式：[rpcclient man page](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html)
- 解説：[HackTricks - MSRPC Enumeration](https://book.hacktricks.xyz/network-services-pentesting/135-pentesting-msrpc)
- 解説：[Null Sessions on Windows](https://www.hackingarticles.in/rpcclient-port-135-enumeration-guide/)
