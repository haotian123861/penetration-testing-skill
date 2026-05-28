# 中间件与CVE速查

## 目录
- [Web服务器](#web服务器)
- [应用服务器](#应用服务器)
- [数据库](#数据库)
- [缓存/消息队列](#缓存消息队列)
- [容器/云原生](#容器云原生)
- [高危CVE汇总](#高危cve汇总)

---

## Web服务器

### Apache

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| 2.4.49 | 路径穿越+RCE | CVE-2021-41773 | High |
| 2.4.50 | 修复绕过 | CVE-2021-42013 | Critical |
| 2.4.48 | 内存损坏 | CVE-2021-40438 | High |
| 全版本 | mod_proxy SSRF | CVE-2021-40438 | High |

**检测命令**:
```bash
apache2 -v
curl -I http://target.com  # 查看Server头
nmap --script http-apache* target.com
```

### Nginx

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=1.20.0 | 堆缓冲区溢出 | CVE-2021-23017 | High |
| <=1.14.0 | 变量处理缺陷 | CVE-2021-23017 | High |
| 全版本 | alias配置绕过 | CVE-2021/23017 | Medium |

**检测命令**:
```bash
nginx -v
nmap --script http-nginx* target.com
```

### IIS

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| 6.0 | 缓冲区溢出 | CVE-2017-7269 | Critical |
| 7.5 | 认证绕过 | CVE-2009-1535 | High |
| 10.0 | HTTP.sys DoS | CVE-2015-1635 | High |

**检测命令**:
```bash
curl -I http://target.com  # 查看Server头
nmap --script iis* target.com
```

---

## 应用服务器

### Tomcat

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=10.0.0 | RCE | CVE-2020-1938 | Critical |
| <=9.0.30 | 幽灵猫 | CVE-2020-1938 | Critical |
| <=8.5.51 | AJP配置错误 | CVE-2020-1938 | Critical |
| 全版本 | 弱默认口令 | - | Medium |

**检测命令**:
```bash
curl -s http://target.com:8080/  # 检查默认页面
nmap --script http-tomcat* target.com -p 8080
# 检查管理后台
curl http://target.com:8080/manager/html
```

### JBoss

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=7.x | 反序列化RCE | CVE-2017-12149 | Critical |
| <=6.x | invoker路径 | CVE-2015-7501 | Critical |
| 全版本 | Java反序列化 | - | Critical |

**检测命令**:
```bash
nmap --script=jboss* target.com -p 8080
curl http://target.com:8080/jmx-console/
curl http://target.com:8080/web-console/
```

### WebLogic

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=12.2.1.3 | 反序列化RCE | CVE-2021-2109 | Critical |
| <=14.1.1.0 | LDAP远程代码执行 | CVE-2021-2109 | Critical |
| 全版本 | T3/IIOP反序列化 | CVE-2018-2628 | Critical |

**检测命令**:
```bash
nmap --script weblogic* target.com -p 7001
curl -s http://target.com:7001/console  # 检查控制台
```

### Jetty

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=9.4.x | 部署问题信息泄露 | CVE-2021-28164 | Medium |
| <=9.4.37 | 敏感信息泄露 | CVE-2021-34428 | Medium |

### Node.js/Express

| 框架 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| Express | 路径遍历 | CVE-2022-24999 | High |
| Node.js | HTTP请求走私 | CVE-2021-23337 | Medium |
| Node.js | DNS重绑定 | CVE-2020-8203 | Medium |

### Spring

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=5.3.18 | Spring4Shell RCE | CVE-2022-22965 | Critical |
| <=5.3.17 | DataBinding绕过 | CVE-2022-22968 | Medium |
| 全版本 | SpEL注入 | CVE-2018-1273 | Critical |

**检测命令**:
```bash
# 检查Spring Boot actuator
curl http://target.com/actuator/env
curl http://target.com/actuator/configprops
```

---

## 数据库

### MySQL

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=8.0.29 | UDF提权 | CVE-2022-32081 | High |
| <=5.7.30 | 堆溢出 | CVE-2020-2574 | High |
| <=5.6.48 | 权限绕过 | CVE-2020-2752 | Medium |

**检测命令**:
```bash
mysql -h target.com -u root -p''  # 空密码测试
mysql -h target.com -u root -e "SELECT @@version"
nmap --script mysql* target.com -p 3306
```

### PostgreSQL

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=14.5 | UDF提权 | CVE-2022-41862 | High |
| <=13.8 | 任意代码执行 | CVE-2022-41862 | High |

**检测命令**:
```bash
psql -h target.com -U postgres -W
nmap --script pgsql* target.com -p 5432
```

### MongoDB

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| 全版本 | 未授权访问 | - | High |
| <=4.4 | 权限绕过 | CVE-2021-23036 | Medium |

**检测命令**:
```bash
mongo mongodb://target.com:27017
mongosh mongodb://target.com:27017
nmap --script mongodb* target.com -p 27017
```

### Redis

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=6.2.6 | Lua沙箱逃逸 | CVE-2022-0543 | Critical |
| <=5.0.14 | Lua沙箱逃逸 | CVE-2022-0543 | Critical |
| 全版本 | 未授权访问 | - | High |
| 全版本 | 写SSH密钥 | - | Critical |

**检测命令**:
```bash
redis-cli -h target.com
redis-cli -h target.com info
redis-cli -h target.com CONFIG GET *  # 未授权检查
```

### MSSQL

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=2019 | 身份验证绕过 | CVE-2021-1636 | Medium |

**检测命令**:
```bash
sqsh -H target.com -U sa
nmap --script ms-sql* target.com -p 1433
```

### Oracle

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| 全版本 | TNS Poison | CVE-2012-1675 | High |

---

## 缓存/消息队列

### Memcached

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| 全版本 | 未授权访问 | - | High |
| <=1.6.6 | 整数溢出 | CVE-2021-3278 | Medium |

**检测命令**:
```bash
nc -vn target.com 11211
stats
nmap --script memcached* target.com -p 11211
```

### RabbitMQ

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=3.10.0 | 认证绕过 | CVE-2022-2551 | High |
| 全版本 | 默认口令 | - | Medium |

**检测命令**:
```bash
curl -u guest:guest http://target.com:15672/api/overview
nmap --script rabbitmq* target.com -p 15672
```

### Kafka

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=3.3.0 | 未授权访问 | CVE-2023-25194 | High |

### Elasticsearch

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=7.17.5 | 权限绕过 | CVE-2023-31451 | High |
| 全版本 | 未授权访问 | - | Critical |
| 全版本 | Groovy脚本注入 | CVE-2015-1427 | Critical |

**检测命令**:
```bash
curl http://target.com:9200/_cat/indices
curl http://target.com:9200/_search?pretty
```

---

## 容器/云原生

### Docker

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=19.03.15 | 容器逃逸 | CVE-2021-41091 | High |
| <=20.10.9 | cgroup特权逃逸 | CVE-2022-0492 | High |
| API | 未授权访问 | - | Critical |

**检测命令**:
```bash
curl http://target.com:2375/version
docker -H tcp://target.com:2375 ps
nmap --script docker* target.com -p 2375
```

### Kubernetes

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=1.26.0 | API Server DoS | CVE-2022-3933 | Medium |
| <=1.24.0 | kubectl proxy绕过 | CVE-2022-3294 | Medium |
| API | 未授权访问 | - | Critical |

**检测命令**:
```bash
kubectl --server=https://target.com:6443 get pods --insecure-skip-tls-verify
curl -k https://target.com:6443/api/v1/secrets
```

### Harbor

| 版本 | 关键漏洞 | CVE | 等级 |
|------|---------|-----|------|
| <=2.6.0 | API未授权 | CVE-2022-31065 | High |
| <=2.5.0 | SQL注入 | CVE-2022-25237 | High |

---

## 高危CVE汇总

### 2021-2023 Critical级CVE

| CVE | 产品 | 类型 | 影响 |
|-----|------|------|------|
| CVE-2021-44228 | Log4j | RCE | Java远程代码执行 |
| CVE-2022-22965 | Spring4Shell | RCE | Spring框架RCE |
| CVE-2022-0543 | Redis Lua | 沙箱逃逸 | 远程命令执行 |
| CVE-2021-41773 | Apache | 路径穿越+RCE | Web目录穿透 |
| CVE-2020-1938 | Apache Tomcat | 文件读取/RCE | AJP协议漏洞 |
| CVE-2022-0185 | Linux Kernel | 容器逃逸 | Ubuntu/Debian |
| CVE-2022-0492 | Ubuntu/Debian | 容器逃逸 | cgroup通知漏洞 |
| CVE-2023-25194 | Kafka | 未授权访问 | 影响2.3.0-3.3.2 |

### 历史经典CVE

| CVE | 产品 | 类型 | 影响 |
|-----|------|------|------|
| CVE-2017-0144 | SMB | 永恒之蓝 | Windows RCE |
| CVE-2019-0708 | RDP | BlueKeep | Windows RCE |
| CVE-2018-2628 | WebLogic | 反序列化 | 远程命令执行 |
| CVE-2017-12149 | JBoss | 反序列化 | Java RCE |
| CVE-2017-7269 | IIS 6.0 | 缓冲区溢出 | Windows RCE |

### CVE查询资源

```bash
# NVD官方
https://nvd.nist.gov/vuln/search

# CVE详情
https://cve.mitre.org/cve/

# Exploit-DB
https://www.exploit-db.com/

# POC列表
https://github.com/nomi-sec/PoC-in-GitHub

# 漏洞响应
https://github.com/CVEProject/cvelistV5
```
