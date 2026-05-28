# 代码审计指南

## 目录
- [审计流程](#审计流程)
- [PHP审计要点](#php审计要点)
- [Java审计要点](#java审计要点)
- [Python审计要点](#python审计要点)
- [JavaScript审计要点](#javascript审计要点)
- [业务逻辑漏洞](#业务逻辑漏洞)
- [自动化工具](#自动化工具)

---

## 审计流程

### 1. 环境准备

```bash
# 代码获取
git-dumper http://target.com/.git /tmp/target
svn checkout svn://target.com/.svn /tmp/target
# 备份文件
wget http://target.com/backup.zip

# 环境搭建
docker-compose up -d
php -S localhost:8080
```

### 2. 代码结构分析

```bash
# 目录结构
find . -type f -name "*.php" | head -20
tree -L 3

# 入口文件
cat index.php
cat routing.php

# 配置文件
find . -name "config*" -o -name "*config*.php"
```

### 3. 敏感函数追踪

```bash
# 危险函数搜索
grep -rn "eval\|system\|exec\|shell_exec\|passthru" --include="*.php"
grep -rn "mysql_query\|mysqli_query" --include="*.php"
grep -rn "\$_GET\|\$_POST\|\$_REQUEST" --include="*.php"
grep -rn "include\|require" --include="*.php"
```

---

## PHP审计要点

### SQL注入

**危险模式**:
```php
// 直接拼接
$id = $_GET['id'];
$sql = "SELECT * FROM users WHERE id = $id";
mysql_query($sql);

// 窄过滤
$input = str_replace("'", "", $_POST['data']);

// 伪过滤
$input = addslashes($_GET['data']); // 仍有绕过可能
```

**安全写法**:
```php
// 预处理语句
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);

// 参数绑定
$sql = "SELECT * FROM users WHERE id = :id";
$query = $db->prepare($sql);
$query->execute(['id' => $_GET['id']]);
```

### 命令注入

**危险模式**:
```php
system($_POST['cmd']);
exec($_GET['command']);
shell_exec($_SERVER['CMD']);
passthru($_REQUEST['input']);
popen($_GET['data'], "r");
proc_open($_POST['cmd'], ...);
`$cmd`;  // 反引号
```

**审计检查点**:
- 用户输入是否进入命令函数
- 是否有escapeshellarg/escapeshellcmd保护
- 是否存在多个命令分隔符(`;`, `|`, `&`, `$()`)

### 文件操作

**路径穿越**:
```php
// 危险
include($_GET['page'] . ".php");
$f = fopen($_GET['file'], "r");
readfile($_GET['path']);

// 绕过技巧
....//....//....//etc/passwd
%00 (NULL字节截断)
%2e%2e%2f (URL编码)
```

**安全写法**:
```php
// 白名单
$allowed = ['home', 'about', 'contact'];
if (in_array($page, $allowed)) {
    include($page . ".php");
}

// 真实路径校验
$real = realpath($base . $_GET['file']);
if (str_starts_with($real, $base)) {
    include($real);
}
```

### 反序列化

**危险模式**:
```php
unserialize($_COOKIE['data']);
unserialize($_SESSION['user']);
```

**Magic Methods触发点**:
```php
class Test {
    public $cmd;
    
    function __wakeup() {
        system($this->cmd);
    }
}
```

**PHPGGC工具**:
```bash
phpggc Laravel/RCE1 system whoami
```

### XSS

**输出点审计**:
```php
// 危险
echo $_GET['name'];
print_r($user_data);
<div><?= $content ?></div>

// 检查是否存在过滤
echo htmlspecialchars($_GET['name']); // 正确
echo strip_tags($_GET['content']); // 可能绕过
```

---

## Java审计要点

### SQL注入

**危险模式**:
```java
String sql = "SELECT * FROM users WHERE id = " + id;
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);
```

**安全写法**:
```java
PreparedStatement ps = conn.prepareStatement(
    "SELECT * FROM users WHERE id = ?"
);
ps.setInt(1, Integer.parseInt(id));
```

### 命令注入

**危险模式**:
```java
Runtime.getRuntime().exec(cmd);
ProcessBuilder pb = new ProcessBuilder(cmd);
```

**审计检查**:
- String[] vs String参数
- 用户输入是否进入命令
- shell=true风险

### 反序列化

**危险反序列化点**:
```java
ObjectInputStream ois = new ObjectInputStream(input);
Object obj = ois.readObject();

XMLDecoder xd = new XMLDecoder(input);
Object obj = xd.readObject();

YAML.load(userInput);  // SnakeYAML
XStream.fromXML(userInput);
```

**CVE指纹**:
```java
// Spring JDBC
jdbcRowSet.execute();

// Jackson
ObjectMapper mapper = new ObjectMapper();
mapper.enableDefaultTyping();
```

**工具**:
```bash
java-ysoserial.jar CommonsCollections6 "whoami"
java -jar JNDI-Injection-Exploit.jar your_ip:port
```

### SSRF

**危险模式**:
```java
URL url = new URL(userInput);
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
InputStream in = new URL(userInput).openStream();
```

**审计检查点**:
- URL是否验证协议(http/https)
- 是否阻止内网地址
- 是否有重定向跟随

---

## Python审计要点

### SQL注入

**危险模式**:
```python
# 字符串拼接
query = f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(query)

# 格式化字符串
cursor.execute("SELECT * FROM users WHERE id = %s" % user_id)
```

**安全写法**:
```python
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
cursor.execute("SELECT * FROM users WHERE id = :id", {"id": user_id})
```

### 命令注入

**危险模式**:
```python
os.system(user_input)
os.popen(user_input)
subprocess.call(user_input, shell=True)
subprocess.run(f"ls {user_input}", shell=True)
eval(user_input)
exec(user_input)
```

**安全写法**:
```python
subprocess.run(["ls", user_input])  # 列表形式
```

### SSTI模板注入

**危险框架**:
```python
# Jinja2
render_template_string(user_input)
template.render(name=user_input)

# Django
Template(user_input)
```

**检测Payload**:
```python
{{7*7}}     # Jinja2输出49
${7*7}      # Mako输出49
<%= 7*7 %>  # ERB输出49
```

### 反序列化

**危险函数**:
```python
pickle.loads(data)
yaml.load(data)  # 未用Loader
marshal.loads(data)
```

**安全写法**:
```python
yaml.safe_load(data)
```

---

## JavaScript审计要点

### Node.js安全

**命令注入**:
```javascript
child_process.exec(userInput);  // 危险
child_process.execSync(userInput);  // 危险
eval(userInput);  // 危险
new Function(code);  // 危险
```

**路径遍历**:
```javascript
const path = req.query.file;
fs.readFile('./static/' + path);
```

### 前端XSS

**危险输出**:
```javascript
document.write(userInput);
element.innerHTML = userInput;
eval(userInput);
location.href = userInput;
```

**安全写法**:
```javascript
element.textContent = userInput;
element.setAttribute(name, sanitizedValue);
```

### Express框架

**模板注入**:
```javascript
res.render(userInput);  // 取决于模板引擎
```

---

## 业务逻辑漏洞

### 认证绕过

```php
// 只检查session存在
if ($_SESSION['user']) {
    showAdminPanel();
}

// 弱检查
if ($role == 'admin') {  // 可被参数覆盖
    adminAction();
}

// 条件可绕过
if (checkAuth() OR isLocalHost()) {  // 逻辑错误
    adminAction();
}
```

### 越权访问

**IDOR示例**:
```javascript
// 水平越权
GET /api/user/1001/orders  // 查看他人订单

// 垂直越权
GET /api/admin/users  // 普通用户访问管理员接口
```

**审计要点**:
- 资源ID是否可预测
- 是否校验当前用户与资源归属
- 接口权限是否后端验证

### 条件竞争

```php
// 银行转账场景
$balance = getBalance($userId);  // 1000
if ($balance >= $amount) {
    sleep(1);  // 时间窗口
    updateBalance($userId, $balance - $amount);
    processOrder($amount);
}
```

**测试方法**:
```bash
# 并发请求
for i in {1..50}; do curl -X POST & done
```

### 金额篡改

```javascript
// 信任客户端价格
const order = {
    amount: response.price,  // 前端直接使用
    item: itemId
};
submitOrder(order);

// 正确做法
// 服务端重新计算价格
```

### 验证码绕过

```php
// 验证码存储在前端
$code = $_SESSION['captcha'];  // 可被预测

// 验证码复用
if ($_POST['code'] == $_SESSION['last_code']) {
    // 逻辑错误
}
```

### 业务流程绕过

```javascript
// 步骤校验缺失
function processOrder(step, data) {
    // 应该验证step=1->2->3顺序
    // 但直接处理任意step
    if (step == 3) {
        deliverOrder(data);
    }
}

// 测试步骤
POST /api/order?step=3&item_id=100
```

---

## 自动化工具

### Semgrep(推荐)

```bash
# 安装
pip install semgrep

# 安全规则
semgrep --config=p/security audits /path/to/code

# 自定义规则
semgrep --config=semgrep-rules/ /path/to/code
```

### Bandit(Python)

```bash
pip install bandit
bandit -r ./python_project/
bandit -r ./python_project/ -f json -o report.json
```

### SonarQube

```bash
# Docker部署
docker run -d --name sonarqube -p 9000:9000 sonarqube

# 扫描项目
sonar-scanner -Dsonar.projectKey=myproject
```

### RIPS(PHP)

```bash
# Docker部署
docker run -d -p 8888:8888 rips/rips

# Web界面扫描
```

### CodeQL

```bash
# 创建数据库
codeql database create --language=python codeql-db --source-root=./src

# 执行查询
codeql query run --database=codeql-db queries/python-ql-sql-injection.ql
```

### 常用规则

| 漏洞类型 | Semgrep规则 | Bandit检查 |
|---------|------------|-----------|
| SQL注入 | `java.lang.security.audit.sql-injection.sql-injection` | B608 |
| 命令注入 | `java.lang.security.audit.command-injection.command-injection` | B602/B603 |
| XSS | `javascript.lang.security.detect-inner-html` | B703 |
| 反序列化 | `java.lang.security.audit.deserialization.unsafe-deserialization` | B300 |
