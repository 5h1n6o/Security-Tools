# 1. Basic Information（基本情報）

## Tool Name

正式名称：

**ffuf (Fuzz Faster U Fool)**

公式リポジトリ：

[https://github.com/ffuf/ffuf](https://github.com/ffuf/ffuf)

---

## Overview

`ffuf` は、Webアプリケーションの**高速ファジング（Fuzzing）ツール**です。

主に以下の用途で利用されます。

* ディレクトリ・ファイル探索
* Webパラメータ発見
* Virtual Host 列挙
* GET/POSTパラメータ fuzzing
* API Endpoint Discovery

`wfuzz` や `gobuster` の代替として利用されることが多く、Go言語で実装されているため高速に動作します。

---

## Purpose

ffufの主な目的：

* Webサーバ上に存在する隠されたリソースを発見する
* 未公開ディレクトリ・ファイルを列挙する
* HTTPリクエストを大量送信して入力ポイントを探索する
* PentestやBug BountyでAttack Surfaceを把握する

---

## Main Use Cases

### Reconnaissance

* Webディレクトリ探索
* Hidden Endpoint探索
* Backup File探索

### Enumeration

* `/admin`
* `/backup`
* `/api`
* `/dev`

などの存在確認。

### Virtual Host Discovery

例：

```
dev.example.com
admin.example.com
test.example.com
```

などのサブドメイン発見。

### Parameter Discovery

例：

```
https://target.com/page?id=FUZZ
```

のようなパラメータ探索。

---

## Installation

### Linux

#### Kali Linux

```bash
sudo apt update
sudo apt install ffuf
```

確認：

```bash
ffuf -V
```

---

#### Go Install

```bash
go install github.com/ffuf/ffuf/v2@latest
```

PATH追加：

```bash
export PATH=$PATH:$HOME/go/bin
```

---

### macOS

Homebrew:

```bash
brew install ffuf
```

---

### Windows

Chocolatey:

```powershell
choco install ffuf
```

またはGo:

```powershell
go install github.com/ffuf/ffuf/v2@latest
```

---

## Basic Syntax

基本構文：

```bash
ffuf [options] -u URL -w WORDLIST
```

例：

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt
```

| 要素     | 説明                |
| ------ | ----------------- |
| `-u`   | Target URL        |
| `-w`   | Wordlist指定        |
| `FUZZ` | 置換ポイント            |
| `-mc`  | Match HTTP Status |
| `-fs`  | Filter Size       |

---

# 2. How It Works（仕組み）

ffufは指定したWordlistを利用してHTTPリクエストを大量生成します。

基本フロー：

```
        Wordlist
           |
           |
           v
       ffuf Engine
           |
           |
 HTTP Request Generator
           |
           |
           v
      Web Server
           |
           |
 Response Analysis
           |
           |
 Match / Filter
           |
           |
      Result Output
```

---

## 動作原理

例：

Wordlist:

```
admin
backup
test
api
```

URL:

```
http://target.com/FUZZ
```

ffuf内部では：

```
http://target.com/admin

http://target.com/backup

http://target.com/test

http://target.com/api
```

としてリクエストします。

---

## 使用プロトコル

主に：

* HTTP
* HTTPS

対応：

* GET
* POST
* HEAD

---

# 3. Command Structure（コマンド構造）

基本構造：

```bash
ffuf [options] -u <URL> -w <WORDLIST>
```

| 要素      | 説明                                   |
| ------- | ------------------------------------ |
| `-u`    | Target URL                           |
| `-w`    | Payload List                         |
| `FUZZ`  | Injection Point                      |
| Options | Filtering / Matching / Performance設定 |

---

# 4. Options Reference（オプション一覧）

## General Options

| Option   | Long Option | Description | Frequency | Example                 |
| -------- | ----------- | ----------- | --------- | ----------------------- |
| -u       | -           | Target URL  | ★★★★★     | `-u http://target/FUZZ` |
| -w       | -           | Wordlist    | ★★★★★     | `-w word.txt`           |
| -X       | -           | HTTP Method | ★★★★☆     | `-X POST`               |
| -H       | -           | Header追加    | ★★★★★     | `-H "Cookie:test"`      |
| -b       | -           | Cookie指定    | ★★★★☆     | `-b "session=xxx"`      |
| -d       | -           | POST Data   | ★★★★☆     | `-d "user=FUZZ"`        |
| -t       | -           | Thread数     | ★★★★★     | `-t 50`                 |
| -timeout | -           | Timeout     | ★★★☆☆     | `-timeout 10`           |

---

# Output Options

| Option | 説明            | Example         |
| ------ | ------------- | --------------- |
| -o     | Output File   | `-o result.txt` |
| -of    | Output Format | `-of json`      |
| -json  | JSON Output   | `-json`         |

---

# Matching Options

| Option | 説明                | Example   |
| ------ | ----------------- | --------- |
| -mc    | Status Code Match | `-mc 200` |
| -ml    | Line数 Match       | `-ml 10`  |
| -mw    | Word数 Match       | `-mw 20`  |
| -ms    | Size Match        | `-ms 500` |

---

# Filtering Options

| Option | 説明            | Example    |
| ------ | ------------- | ---------- |
| -fc    | Status Code除外 | `-fc 404`  |
| -fs    | Size除外        | `-fs 1234` |
| -fw    | Word数除外       | `-fw 10`   |
| -fl    | Line数除外       | `-fl 5`    |

---

# Performance Options

| Option | 説明             | Example     |
| ------ | -------------- | ----------- |
| -t     | Thread数        | `-t 100`    |
| -rate  | Request Rate制限 | `-rate 100` |
| -p     | Delay設定        | `-p 0.1`    |

---

# Advanced Options

| Option     | 用途               | Example             |
| ---------- | ---------------- | ------------------- |
| -recursion | Recursive Scan   | `-recursion`        |
| -e         | Extension指定      | `-e php,html`       |
| -ac        | Auto Calibration | `-ac`               |
| -sni       | SNI指定            | `-sni dev.site.com` |

---

# 5. Frequently Used Commands TOP10（使用頻度TOP10）

| Rank | Command                                                         | Purpose             | Frequency |
| ---- | --------------------------------------------------------------- | ------------------- | --------- |
| 1    | `ffuf -u http://target/FUZZ -w wordlist.txt`                    | Directory Scan      | ★★★★★     |
| 2    | `ffuf -u http://target/FUZZ -w wordlist.txt -fc 404`            | 404除外               | ★★★★★     |
| 3    | `ffuf -u http://target/FUZZ -w wordlist.txt -e .php,.txt`       | Extension Scan      | ★★★★★     |
| 4    | `ffuf -u http://target/FUZZ -w wordlist.txt -recursion`         | Recursive Scan      | ★★★★☆     |
| 5    | `ffuf -u http://target/FUZZ -w wordlist.txt -mc 200,403`        | Status指定            | ★★★★☆     |
| 6    | `ffuf -u http://target -H "Host: FUZZ.target.com" -w hosts.txt` | VHost探索             | ★★★★☆     |
| 7    | `ffuf -u http://target/page?FUZZ=test -w params.txt`            | Parameter Discovery | ★★★★☆     |
| 8    | `ffuf -X POST -d "user=FUZZ" -u URL`                            | POST Fuzzing        | ★★★★☆     |
| 9    | `ffuf -ac`                                                      | Auto Calibration    | ★★★☆☆     |
| 10   | `ffuf -o result.json -of json`                                  | 結果保存                | ★★★☆☆     |

---

# Example 1

## Directory Discovery

Command:

```bash
ffuf \
-u http://target.com/FUZZ \
-w /usr/share/wordlists/dirb/common.txt
```

Purpose:

Webサーバの隠されたディレクトリ探索。

When to use:

初期Recon。

Important Point:

レスポンスサイズやStatus Codeで不要結果を除外する。

---

# 6. Common Usage Examples（実践例）

## Basic Usage

```bash
ffuf -u https://example.com/FUZZ \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

説明：

一般的なWeb Directory Enumeration。

---

## Extension Discovery

```bash
ffuf \
-u http://target/FUZZ \
-w wordlist.txt \
-e .php,.bak,.txt
```

説明：

バックアップファイル探索。

---

## VHost Discovery

```bash
ffuf \
-u http://target/ \
-H "Host: FUZZ.target.com" \
-w subdomains.txt
```

説明：

Virtual Host列挙。

---

## POST Parameter Fuzzing

```bash
ffuf \
-u http://target/login \
-X POST \
-d "username=FUZZ&password=test" \
-w users.txt
```

説明：

入力値探索。

---

# 7. Output Interpretation（結果の読み方）

例：

```
admin       [Status: 200, Size: 5321]
backup      [Status: 403, Size: 281]
test        [Status: 404, Size: 124]
```

解析：

| 項目         | 意味          |
| ---------- | ----------- |
| Status 200 | 存在確認        |
| Status 403 | 存在するがアクセス拒否 |
| Status 404 | 未存在         |
| Size       | レスポンスサイズ    |

重要：

403は「無い」ではなく「存在する可能性あり」。

---

# 8. Practical Scenarios（実践シナリオ）

## Scenario 1

状況：

WebサーバのHidden Directoryを調査したい。

目的：

Attack Surface発見。

使用コマンド：

```bash
ffuf \
-u http://target/FUZZ \
-w common.txt \
-fc 404
```

確認ポイント：

* admin
* backup
* upload
* api

---

## Scenario 2

状況：

サブドメインではなくVirtual Host探索。

使用コマンド：

```bash
ffuf \
-u http://target \
-H "Host: FUZZ.target.com" \
-w hosts.txt
```

---

# 9. Troubleshooting Guide（障害対応）

| 症状             | 原因           | 確認方法     | 解決方法     |
| -------------- | ------------ | -------- | -------- |
| 結果が大量          | Filter不足     | Size確認   | `-fs`利用  |
| 遅い             | Thread不足     | `-t`確認   | Thread増加 |
| 403しか出ない       | WAF/権限       | Header確認 | 認証追加     |
| False Positive | Default Page | Size比較   | Filter設定 |

---

# 10. Security Considerations（セキュリティ）

## 攻撃側利用

利用例：

* Directory Discovery
* Endpoint Discovery
* Parameter Discovery

---

## 防御側観点

ログに残る情報：

* 大量HTTP Request
* 不自然な404アクセス
* User-Agent

対策：

* Rate Limit
* WAF
* IDS/IPS
* Monitoring

---

# 11. Related Commands / Tools（関連ツール）

## Alternative Tools

| Tool      | 用途                   |
| --------- | -------------------- |
| gobuster  | Directory/VHost Scan |
| dirsearch | Web Enumeration      |
| wfuzz     | Fuzzing              |

---

## Complementary Tools

| Tool       | 用途             |
| ---------- | -------------- |
| nmap       | Port Discovery |
| nikto      | Web Scan       |
| burp suite | Manual Testing |
| httpx      | HTTP Probe     |

---

## Next Step Tools

学習順：

```
nmap
 ↓
httpx
 ↓
ffuf
 ↓
Burp Suite
 ↓
Manual Exploitation
```

---

# 12. Cheat Memory（覚え方）

## FUZZ = 入れ替え場所

覚え方：

```
FUZZ = Payloadを入れる穴
```

---

重要オプション：

```
-u  URL
-w  Wordlist
-mc Match Code
-fc Filter Code
-e  Extension
-t  Thread
```

覚え方：

```
URL + Wordlist + Match/Filter
```

---

# 13. Quick Reference（クイックリファレンス）

## 基本構文

```bash
ffuf -u URL/FUZZ -w WORDLIST
```

---

## TOP10

```bash
# Directory Scan
ffuf -u http://target/FUZZ -w wordlist.txt

# 404除外
ffuf -u http://target/FUZZ -w wordlist.txt -fc 404

# Extension
ffuf -u http://target/FUZZ -w wordlist.txt -e .php,.txt

# Recursive
ffuf -u http://target/FUZZ -w wordlist.txt -recursion

# VHost
ffuf -u http://target -H "Host: FUZZ.target.com" -w hosts.txt
```

---

## 必須オプション

| Option | 用途        |
| ------ | --------- |
| -u     | URL       |
| -w     | Wordlist  |
| -mc    | Match     |
| -fc    | Filter    |
| -e     | Extension |
| -t     | Thread    |
| -H     | Header    |
| -d     | POST Data |

---

# 14. Personal Notes

```markdown
## My Notes

- 
- 
- 
```
