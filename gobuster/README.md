# 1. Basic Information（基本情報）

## Tool Name

正式名称：

**Gobuster**

公式リポジトリ：

https://github.com/OJ/gobuster

開発言語：

- Go

---

## Overview

`gobuster` は、Webサイトやネットワークサービスに対して高速な列挙（Enumeration）を行うためのツールです。

主に以下の用途で利用されます。

- Webディレクトリ探索
- ファイル探索
- DNSサブドメイン列挙
- Virtual Host探索
- Amazon S3バケット探索
- TFTPサーバ探索

Pentest、CTF、OSCP試験対策で頻繁に利用される代表的なEnumerationツールです。

---

## Purpose

Gobusterの目的：

- WebアプリケーションのHidden Resource発見
- Attack Surface Discovery
- サブドメイン列挙
- 未公開ファイル・ディレクトリ発見
- セキュリティ診断時の初期Recon

---

## Main Use Cases

### Reconnaissance

対象環境の情報収集。

例：

- Web Directory Discovery
- Subdomain Discovery

---

### Enumeration

対象：

- `/admin`
- `/backup`
- `/uploads`
- `/api`

などの隠されたリソース探索。

---

### Web Assessment

発見対象：

- 管理画面
- 設定ファイル
- バックアップファイル
- 古いアプリケーション

---

### DNS Enumeration

例：

```text
dev.example.com
test.example.com
admin.example.com
````

などのサブドメイン探索。

---

## Installation

## Linux

### Kali Linux

```bash
sudo apt update
sudo apt install gobuster
```

確認：

```bash
gobuster version
```

---

### Go Install

```bash
go install github.com/OJ/gobuster/v3@latest
```

PATH追加：

```bash
export PATH=$PATH:$HOME/go/bin
```

---

## macOS

Homebrew：

```bash
brew install gobuster
```

---

## Windows

Chocolatey：

```powershell
choco install gobuster
```

またはGo：

```powershell
go install github.com/OJ/gobuster/v3@latest
```

---

## Basic Syntax

基本構文：

```bash
gobuster <mode> [options]
```

例：

```bash
gobuster dir \
-u http://target.com \
-w wordlist.txt
```

---

| 要素   | 説明          |
| ---- | ----------- |
| mode | 探索種類        |
| -u   | Target URL  |
| -w   | Wordlist    |
| -x   | Extension指定 |
| -t   | Thread数     |
| -o   | Output      |

---

# 2. How It Works（仕組み）

GobusterはWordlistを利用して大量のリクエストを生成し、対象サーバからのレスポンスを解析します。

基本処理：

```
          Wordlist
              |
              |
              v
        Gobuster Engine
              |
              |
      HTTP/DNS Request
              |
              |
              v
          Target
              |
              |
      Response Analysis
              |
              |
        Found Resource
```

---

## Directory Mode 動作例

Wordlist：

```
admin
backup
login
test
```

実行：

```bash
gobuster dir \
-u http://target.com \
-w wordlist.txt
```

生成：

```
http://target.com/admin
http://target.com/backup
http://target.com/login
http://target.com/test
```

---

## 使用プロトコル

対応：

* HTTP
* HTTPS
* DNS
* S3
* TFTP

---

# 3. Command Structure（コマンド構造）

基本構造：

```bash
gobuster <mode> [options]
```

例：

```bash
gobuster dir \
-u https://example.com \
-w /usr/share/wordlists/dirb/common.txt
```

---

| 要素    | 説明                         |
| ----- | -------------------------- |
| dir   | Directory/File Enumeration |
| dns   | DNS Subdomain Enumeration  |
| vhost | Virtual Host Enumeration   |
| s3    | Amazon S3 Enumeration      |
| fuzz  | URL Fuzzing                |
| tftp  | TFTP Enumeration           |

---

# 4. Options Reference（オプション一覧）

## General Options

| Option | Long Option   | Description | Frequency | Example                             |
| ------ | ------------- | ----------- | --------- | ----------------------------------- |
| -u     | --url         | Target URL  | ★★★★★     | -u [https://target](https://target) |
| -w     | --wordlist    | Wordlist指定  | ★★★★★     | -w word.txt                         |
| -t     | --threads     | Thread数     | ★★★★★     | -t 50                               |
| -o     | --output      | 結果保存        | ★★★★☆     | -o result.txt                       |
| -q     | --quiet       | バナー非表示      | ★★★☆☆     | -q                                  |
| -z     | --no-progress | Progress非表示 | ★★★☆☆     | -z                                  |

---

# Output Options

| Option | Description | Example         |
| ------ | ----------- | --------------- |
| -o     | Output File | `-o result.txt` |
| -q     | Quiet Mode  | `-q`            |

---

# Directory Mode Options

| Option | Description      | Example          |
| ------ | ---------------- | ---------------- |
| -x     | Extension指定      | `-x php,txt`     |
| -r     | Redirect追跡       | `-r`             |
| -e     | Full URL表示       | `-e`             |
| -s     | Status Code指定    | `-s 200,301,403` |
| -b     | Blacklist Status | `-b 404`         |

---

# Performance Options

| Option | Description | Example      |
| ------ | ----------- | ------------ |
| -t     | Thread数     | `-t 100`     |
| -delay | Request間隔   | `--delay 1s` |

---

# DNS Mode Options

| Option | Description  | Example          |
| ------ | ------------ | ---------------- |
| -d     | Domain指定     | `-d example.com` |
| -r     | DNS Resolver | `-r 8.8.8.8`     |

---

# Advanced Options

| Option | Description  | Example                    |
| ------ | ------------ | -------------------------- |
| -k     | TLS証明書無視     | `-k`                       |
| -H     | Header追加     | `-H "Cookie:xxx"`          |
| -U     | User Agent指定 | `-U Mozilla`               |
| -p     | Proxy指定      | `-p http://127.0.0.1:8080` |

---

# 5. Frequently Used Commands TOP10（使用頻度TOP10）

| Rank | Command                                         | Purpose           | Frequency |
| ---- | ----------------------------------------------- | ----------------- | --------- |
| 1    | `gobuster dir -u URL -w WORDLIST`               | Directory Scan    | ★★★★★     |
| 2    | `gobuster dir -u URL -w WORDLIST -x php,txt`    | Extension Scan    | ★★★★★     |
| 3    | `gobuster dir -u URL -w WORDLIST -t 50`         | 高速Scan            | ★★★★★     |
| 4    | `gobuster dns -d domain.com -w subdomains.txt`  | Subdomain Scan    | ★★★★★     |
| 5    | `gobuster vhost -u URL -w hosts.txt`            | Virtual Host Scan | ★★★★☆     |
| 6    | `gobuster dir -u URL -w WORDLIST -x bak,old`    | Backup探索          | ★★★★☆     |
| 7    | `gobuster dir -u URL -w WORDLIST -s 200,403`    | Status指定          | ★★★★☆     |
| 8    | `gobuster dir -u URL -w WORDLIST -k`            | SSL無視             | ★★★☆☆     |
| 9    | `gobuster dir -u URL -w WORDLIST -o result.txt` | 結果保存              | ★★★☆☆     |
| 10   | `gobuster s3 -w WORDLIST`                       | S3探索              | ★★★☆☆     |

---

# Example 1

## Directory Discovery

Command:

```bash
gobuster dir \
-u http://target.com \
-w /usr/share/wordlists/dirb/common.txt
```

Purpose:

Webディレクトリ探索。

When to use:

初期Recon。

Important Point:

404レスポンスを除外する設定が重要。

---

# Example 2

## PHP File Discovery

Command:

```bash
gobuster dir \
-u http://target.com \
-w wordlist.txt \
-x php,txt,bak
```

Purpose:

PHPファイルやバックアップファイル探索。

---

# 6. Common Usage Examples（実践例）

## Basic Usage

```bash
gobuster dir \
-u https://target.com \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

説明：

一般的なWeb Directory Enumeration。

---

## Troubleshooting

```bash
gobuster dir \
-u https://target.com \
-w wordlist.txt \
-b 404
```

説明：

404ページによるFalse Positive除外。

---

## Advanced Usage

```bash
gobuster dir \
-u https://target.com \
-w wordlist.txt \
-x php,html,bak \
-t 100 \
-k
```

説明：

高速な詳細探索。

---

## Automation Example

```bash
#!/bin/bash

gobuster dir \
-u $1 \
-w /usr/share/wordlists/common.txt \
-o gobuster_result.txt
```

---

# 7. Output Interpretation（結果の読み方）

例：

```
/admin              (Status: 200)
/backup             (Status: 403)
/login.php          (Status: 200)
```

解析：

| 結果      | 意味       |
| ------- | -------- |
| 200     | 存在確認     |
| 301/302 | Redirect |
| 403     | 存在するが拒否  |
| 404     | 不存在      |

---

重要ポイント：

403は重要。

理由：

```
存在する
↓
アクセス制限
↓
認証突破対象になる可能性
```

---

# 8. Practical Scenarios（実践シナリオ）

## Scenario 1

状況：

Webサーバの隠しディレクトリを探したい。

目的：

Attack Surface Discovery。

使用コマンド：

```bash
gobuster dir \
-u http://target \
-w common.txt
```

確認ポイント：

* admin
* login
* backup
* upload

---

## Scenario 2

状況：

サブドメイン探索。

目的：

DNS Enumeration。

使用コマンド：

```bash
gobuster dns \
-d target.com \
-w subdomains.txt
```

---

# 9. Troubleshooting Guide（障害対応）

| 症状        | 原因                | 確認方法     | 解決方法   |
| --------- | ----------------- | -------- | ------ |
| 結果が大量     | Wildcard Response | サイズ比較    | -b設定   |
| 遅い        | Thread不足          | CPU確認    | -t増加   |
| SSL Error | 証明書問題             | -k利用     | 証明書確認  |
| 404大量     | Filter不足          | Status確認 | -b 404 |

---

# 10. Security Considerations（セキュリティ）

## 攻撃側利用

用途：

* Hidden Directory Discovery
* Backup File Discovery
* Subdomain Discovery

---

## 防御側観点

ログに残る情報：

* 大量GET Request
* 多数404
* 同一User-Agent

対策：

* Rate Limit
* WAF
* IDS監視
* 不要ファイル削除

---

# 11. Related Commands / Tools（関連ツール）

## Alternative Tools

| Tool        | 用途                  |
| ----------- | ------------------- |
| ffuf        | 高速Fuzzing           |
| dirsearch   | Web Enumeration     |
| feroxbuster | Directory Discovery |
| wfuzz       | Web Fuzzing         |

---

## Complementary Tools

| Tool       | 用途             |
| ---------- | -------------- |
| nmap       | Port Scan      |
| httpx      | HTTP Probe     |
| nikto      | Web Scanner    |
| Burp Suite | Manual Testing |

---

## Next Step Tools

推奨学習順：

```
nmap
 ↓
httpx
 ↓
gobuster
 ↓
ffuf
 ↓
Burp Suite
 ↓
Manual Exploitation
```

---

# 12. Cheat Memory（覚え方）

## Mode

覚え方：

```
dir = Directory
dns = Domain
vhost = Virtual Host
```

---

## 必須オプション

```
-u  URL
-w  Wordlist
-t  Thread
-x  Extension
-o  Output
```

覚え方：

```
URL + Wordlist + Thread + Extension
```

---

# 13. Quick Reference（クイックリファレンス）

## 基本構文

```bash
gobuster dir -u URL -w WORDLIST
```

---

## TOP Commands

```bash
# Directory Scan
gobuster dir -u http://target -w wordlist.txt

# Extension Scan
gobuster dir -u http://target -w wordlist.txt -x php,txt

# DNS Scan
gobuster dns -d target.com -w subdomains.txt

# VHost Scan
gobuster vhost -u http://target -w hosts.txt

# 高速Scan
gobuster dir -u URL -w wordlist.txt -t 100
```

---

## 必須オプション

| Option | 用途         |
| ------ | ---------- |
| -u     | URL        |
| -w     | Wordlist   |
| -t     | Thread     |
| -x     | Extension  |
| -o     | Output     |
| -s     | Status指定   |
| -k     | SSL Ignore |

---

# 14. Personal Notes

```markdown
## My Notes

- 
- 
- 
```

