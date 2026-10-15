# Windows系统中文乱码问题解决方案

## 问题概述

在Windows系统开发时，由于系统默认使用GBK/CP936编码，而现代项目代码、配置文件通常统一使用UTF-8编码，会导致以下问题：
- Java/Maven编译时中文注释乱码
- 控制台日志输出乱码
- Python脚本输出乱码
- 文件保存后中文变成乱码
- Git提交消息乱码

本文档提供**完整且通用**的解决方案，适用于Java、Python、Node.js等各类开发环境，确保整个开发环境统一使用UTF-8编码。

---

## 1. Cursor IDE 设置

### 1.1 文件编码设置

**方法1：全局配置（推荐）**

打开Cursor设置（`Ctrl + ,`），搜索 `encoding`，配置：

```json
{
  "files.encoding": "utf8",
  "files.autoGuessEncoding": false,
  "[java]": {
    "files.encoding": "utf8"
  },
  "[xml]": {
    "files.encoding": "utf8"
  },
  "[yaml]": {
    "files.encoding": "utf8"
  },
  "[properties]": {
    "files.encoding": "utf8"
  }
}
```

**方法2：工作区配置**

在项目根目录创建/修改 `.vscode/settings.json`：

```json
{
  "files.encoding": "utf8",
  "files.autoGuessEncoding": false,
  "files.eol": "\n"
}
```

### 1.2 Java运行环境设置

在 `.vscode/settings.json` 中添加Java编码配置：

```json
{
  "java.jdt.ls.vmargs": "-Dfile.encoding=UTF-8 -Duser.language=zh -Duser.region=CN",
  "java.debug.settings.vmArgs": "-Dfile.encoding=UTF-8"
}
```

### 1.3 Maven集成设置

配置Maven运行环境：

```json
{
  "maven.terminal.customEnv": [
    {
      "environmentVariable": "JAVA_TOOL_OPTIONS",
      "value": "-Dfile.encoding=UTF-8"
    },
    {
      "environmentVariable": "MAVEN_OPTS",
      "value": "-Dfile.encoding=UTF-8"
    }
  ]
}
```

### 1.4 终端编码设置

配置Cursor内置终端：

```json
{
  "terminal.integrated.defaultProfile.windows": "PowerShell",
  "terminal.integrated.profiles.windows": {
    "PowerShell": {
      "source": "PowerShell",
      "icon": "terminal-powershell",
      "overrideName": true
    }
  }
}
```

---

## 2. Maven 设置

### 2.1 项目配置（pom.xml）

在 `pom.xml` 中添加编码配置：

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
</properties>
```

### 2.2 Maven全局配置

在项目根目录创建 `.mvn/maven.config`：

```
-Dfile.encoding=UTF-8
-Dmaven.compiler.encoding=UTF-8
-Dproject.build.sourceEncoding=UTF-8
-Dproject.reporting.outputEncoding=UTF-8
```

### 2.3 环境变量设置

**方式1：临时设置（当前会话）**

在运行Maven命令前，先执行：

```bash
# PowerShell
$env:JAVA_TOOL_OPTIONS = '-Dfile.encoding=UTF-8'
$env:MAVEN_OPTS = '-Dfile.encoding=UTF-8'

# CMD
set JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8
set MAVEN_OPTS=-Dfile.encoding=UTF-8
```

**方式2：永久设置（系统环境变量）**

1. `Win + R` 输入 `sysdm.cpl`
2. "高级" → "环境变量"
3. 在"用户变量"中新建：
   - 变量名：`JAVA_TOOL_OPTIONS`，变量值：`-Dfile.encoding=UTF-8`
   - 变量名：`MAVEN_OPTS`，变量值：`-Dfile.encoding=UTF-8`
4. 重启终端或IDE

**方式3：创建UTF-8启动脚本（推荐）**

创建 `mvnw-utf8.bat` 脚本：

```bat
@echo off
REM Maven UTF-8环境启动脚本
chcp 65001 > nul
set "JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8"
set "MAVEN_OPTS=-Dfile.encoding=UTF-8"
echo [INFO] Maven UTF-8 编码已设置
mvnw.cmd %*
```

使用方法：
```bash
# 编译项目
.\mvnw-utf8.bat clean compile

# 运行测试
.\mvnw-utf8.bat test

# 打包项目
.\mvnw-utf8.bat clean package
```

### 2.4 验证Maven编码

运行以下命令验证：

```bash
mvn help:system | findstr encoding
```

预期输出：
```
file.encoding=UTF-8
sun.jnu.encoding=UTF-8
```

---

## 3. PowerShell 设置

### 3.1 临时设置（当前会话）

在PowerShell中运行：

```powershell
# 设置控制台代码页为UTF-8
chcp 65001

# 设置输出编码
$OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::InputEncoding = [System.Text.Encoding]::UTF8

# 设置Java和Maven编码
$env:JAVA_TOOL_OPTIONS = '-Dfile.encoding=UTF-8'
$env:MAVEN_OPTS = '-Dfile.encoding=UTF-8'

# 设置Python编码
$env:PYTHONIOENCODING = 'utf-8'
```

### 3.2 永久设置（推荐）✅

**配置文件位置**：
- 查看路径：`echo $PROFILE`
- 通常位于：`C:\Users\{用户名}\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`

**配置内容**：

1. 检查配置文件是否存在：
```powershell
Test-Path $PROFILE
```

2. 如果不存在，创建配置文件：
```powershell
New-Item -Path $PROFILE -Type File -Force
```

3. 编辑配置文件，添加以下内容：
```powershell
# UTF-8编码配置
$OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::InputEncoding = [System.Text.Encoding]::UTF8

# 设置控制台代码页为UTF-8
chcp 65001 > $null

# Java和Maven编码
$env:JAVA_TOOL_OPTIONS = '-Dfile.encoding=UTF-8'
$env:MAVEN_OPTS = '-Dfile.encoding=UTF-8'

# Python编码
$env:PYTHONIOENCODING = 'utf-8'

# 显示加载提示
Write-Host "PowerShell UTF-8 Loaded" -ForegroundColor Green
```

4. 重新加载配置：
```powershell
. $PROFILE
```

### 3.3 执行策略设置

如果提示"无法加载，因为在此系统上禁止运行脚本"，需要设置执行策略：

```powershell
# 以管理员身份运行PowerShell，执行：
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 查看当前策略
Get-ExecutionPolicy -List
```

---

## 4. Python 设置

### 4.1 环境变量设置

**方式1：通过PowerShell配置（推荐）✅**

在PowerShell配置文件中添加（参见3.2节）：
```powershell
$env:PYTHONIOENCODING = 'utf-8'
```

**方式2：系统环境变量**

1. `Win + R` 输入 `sysdm.cpl`
2. "高级" → "环境变量"
3. 在"用户变量"中新建：
   - 变量名：`PYTHONIOENCODING`
   - 变量值：`utf-8`

### 4.2 Python脚本头部设置

确保Python脚本文件头部声明UTF-8编码：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
```

### 4.3 JSON输出设置

在输出JSON时，禁用ASCII转义：

```python
import json

# 正确方式
print(json.dumps(data, ensure_ascii=False, indent=2))

# 错误方式（会导致Unicode转义）
print(json.dumps(data, indent=2))
```

### 4.4 创建Python运行脚本（推荐）

为Python脚本创建UTF-8启动脚本（如 `run-py-utf8.bat`）：

```bat
@echo off
REM Python UTF-8环境启动脚本
chcp 65001 > nul
set PYTHONIOENCODING=utf-8
echo [INFO] Python UTF-8 编码已设置
python %*
pause
```

使用方法：
```bash
# 运行Python脚本
.\run-py-utf8.bat your_script.py

# 或在脚本目录创建专用启动脚本
python "%~dp0your_script.py"
```

---

## 5. Git 设置

### 5.1 全局编码配置

```bash
# 设置提交和日志编码
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8

# 设置GUI编码
git config --global gui.encoding utf-8

# 禁用自动转换换行符（避免文件改动）
git config --global core.autocrlf false
```

### 5.2 Git Bash编码

如果使用Git Bash，在 `~/.bashrc` 中添加：

```bash
export LANG=zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8
export JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8
export MAVEN_OPTS=-Dfile.encoding=UTF-8
export PYTHONIOENCODING=utf-8
```

---

## 6. Java应用运行时设置

### 6.1 启动参数

在运行Java应用时添加UTF-8参数：

```bash
java -Dfile.encoding=UTF-8 -Duser.language=zh -Duser.region=CN -jar app.jar
```

### 6.2 Spring Boot配置

在 `application.yml` 中配置：

```yaml
spring:
  messages:
    encoding: UTF-8
  http:
    encoding:
      charset: UTF-8
      enabled: true
      force: true

server:
  servlet:
    encoding:
      charset: UTF-8
      enabled: true
      force: true
```

### 6.3 创建应用启动脚本

创建 `start.bat` 启动脚本：

```bat
@echo off
chcp 65001 >nul
echo ========================================
echo 启动 Spring Boot 应用
echo ========================================
echo.

set "JAVA_OPTS=-Dfile.encoding=UTF-8 -Duser.language=zh -Duser.region=CN"
echo [INFO] 正在启动应用（UTF-8编码）...
java %JAVA_OPTS% -jar target/your-app.jar

pause
```

或使用Maven启动：
```bat
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Dfile.encoding=UTF-8"
```

---

## 7. 数据库连接设置

### 7.1 MySQL连接URL

确保连接URL包含字符集参数：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/your_database?useUnicode=true&characterEncoding=utf8mb4&serverTimezone=Asia/Shanghai
```

或JDBC连接字符串：
```
jdbc:mysql://localhost:3306/your_database?useUnicode=true&characterEncoding=utf8mb4&useSSL=false&serverTimezone=Asia/Shanghai
```

### 7.2 数据库字符集配置

检查MySQL数据库字符集：

```sql
-- 查看数据库字符集
SHOW VARIABLES LIKE 'character%';

-- 设置数据库为UTF-8
ALTER DATABASE your_database CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 创建表时指定字符集
CREATE TABLE your_table (
    id INT PRIMARY KEY,
    name VARCHAR(100)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 7.3 MyBatis/JPA配置

确保Mapper XML文件和Entity类使用UTF-8编码保存。

---

## 8. Node.js 设置

### 8.1 设置环境变量

```bash
# PowerShell
$env:NODE_OPTIONS = '--input-type=utf8'

# CMD
set NODE_OPTIONS=--input-type=utf8
```

### 8.2 package.json 脚本

在 `package.json` 中设置：

```json
{
  "scripts": {
    "start": "node --input-type=utf8 index.js",
    "dev": "nodemon --input-type=utf8 index.js"
  }
}
```

### 8.3 文件读写

确保Node.js代码中指定UTF-8编码：

```javascript
const fs = require('fs');

// 读取文件
fs.readFile('file.txt', 'utf8', (err, data) => {
  console.log(data);
});

// 写入文件
fs.writeFile('file.txt', content, 'utf8', (err) => {
  if (err) throw err;
});
```

---

## 9. 验证与测试

### 9.1 验证PowerShell配置

打开新的PowerShell窗口，应该看到：
```
PowerShell UTF-8 Loaded
```

检查环境变量：
```powershell
echo $env:PYTHONIOENCODING      # 应输出: utf-8
echo $env:JAVA_TOOL_OPTIONS     # 应输出: -Dfile.encoding=UTF-8
echo $env:MAVEN_OPTS            # 应输出: -Dfile.encoding=UTF-8
```

### 9.2 验证Java编码

```bash
java -XshowSettings:properties -version 2>&1 | findstr encoding
```

预期输出包含：
```
file.encoding = UTF-8
sun.jnu.encoding = UTF-8
```

### 9.3 验证Maven编码

```bash
mvn help:system | findstr encoding
```

预期输出包含：
```
file.encoding=UTF-8
sun.jnu.encoding=UTF-8
```

### 9.4 验证Python编码

创建测试脚本 `test_encoding.py`：

```python
import sys
import locale

print(f"Default encoding: {sys.getdefaultencoding()}")
print(f"Stdout encoding: {sys.stdout.encoding}")
print(f"Stdin encoding: {sys.stdin.encoding}")
print(f"Filesystem encoding: {sys.getfilesystemencoding()}")
print(f"Locale: {locale.getpreferredencoding()}")

# 测试中文输出
print("中文测试：你好世界")
```

运行测试：
```bash
python test_encoding.py
```

预期输出：
```
Default encoding: utf-8
Stdout encoding: utf-8
Stdin encoding: utf-8
Filesystem encoding: utf-8
Locale: cp936
中文测试：你好世界
```

### 9.5 验证文件编码

创建包含中文的文件，检查是否正确保存和读取：

```python
# test_file_encoding.py
# 写入中文
with open('test.txt', 'w', encoding='utf-8') as f:
    f.write('测试中文编码：你好世界')

# 读取并打印
with open('test.txt', 'r', encoding='utf-8') as f:
    content = f.read()
    print(content)
```

### 9.6 验证Git编码

```bash
# 查看Git配置
git config --global --get i18n.commitencoding
git config --global --get i18n.logoutputencoding

# 测试提交消息
git commit --allow-empty -m "测试中文提交消息"
git log -1
```

---

## 10. 常见问题排查

### Q1: PowerShell配置不生效？

**原因**：执行策略限制或配置文件路径错误

**解决**：
```powershell
# 1. 检查执行策略
Get-ExecutionPolicy

# 2. 如果是Restricted，设置为RemoteSigned
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 3. 确认配置文件路径
echo $PROFILE
Test-Path $PROFILE

# 4. 手动加载配置
. $PROFILE
```

### Q2: Maven编译仍然乱码？

**解决步骤**：
1. 检查源文件编码是否为UTF-8（使用Cursor查看文件编码）
2. 清理Maven缓存：`mvn clean`
3. 使用 `mvnw-utf8.bat` 脚本代替直接调用mvn
4. 检查IDEA/Cursor的Maven配置

### Q3: Python脚本输出乱码？

**解决步骤**：
1. 确认 `$env:PYTHONIOENCODING` 已设置为 `utf-8`
2. 使用项目提供的 `.bat` 脚本运行
3. 检查脚本文件本身是否为UTF-8编码
4. 确保 `json.dumps()` 使用 `ensure_ascii=False`

### Q4: Git提交后中文注释乱码？

**解决**：
```bash
# 设置Git编码
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8

# 检查文件编码
git config --global core.quotepath false
```

### Q5: 日志文件中中文乱码？

**解决**：
检查 `logback-spring.xml` 配置：
```xml
<configuration>
    <property name="CONSOLE_LOG_CHARSET" value="UTF-8"/>
    <property name="FILE_LOG_CHARSET" value="UTF-8"/>
    
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <encoder>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
</configuration>
```

### Q6: 数据库中文数据乱码？

**解决**：
1. 检查数据库字符集：
```sql
SHOW VARIABLES LIKE 'character%';
```

2. 确保使用 `utf8mb4`：
```sql
ALTER DATABASE your_database CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

3. 检查连接字符串参数（参见7.1节）

### Q7: IDE控制台输出乱码？

**解决（Cursor/VS Code）**：
1. 检查 `settings.json` 中的 `files.encoding` 设置
2. 检查终端配置：`terminal.integrated.defaultProfile.windows`
3. 重启IDE

**解决（IntelliJ IDEA）**：
1. Help → Edit Custom VM Options
2. 添加：`-Dfile.encoding=UTF-8 -Dconsole.encoding=UTF-8`
3. 重启IDEA

---

## 11. 推荐配置流程

### 开发环境初始化（一次性配置，约15分钟）

#### 步骤1：配置PowerShell（5分钟）✨ 最重要

```powershell
# 1. 设置执行策略（以管理员身份运行PowerShell）
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 2. 编辑配置文件
notepad $PROFILE

# 3. 将以下内容复制到配置文件（参考3.2节完整配置）
$OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::InputEncoding = [System.Text.Encoding]::UTF8
chcp 65001 > $null
$env:JAVA_TOOL_OPTIONS = '-Dfile.encoding=UTF-8'
$env:MAVEN_OPTS = '-Dfile.encoding=UTF-8'
$env:PYTHONIOENCODING = 'utf-8'
Write-Host "PowerShell UTF-8 Loaded" -ForegroundColor Green

# 4. 保存并重新加载
. $PROFILE
```

#### 步骤2：配置IDE（3分钟）

**Cursor / VS Code**
1. 打开设置（`Ctrl + ,`）
2. 搜索 `encoding`，设置为 `utf8`
3. 在项目根目录创建/修改 `.vscode/settings.json`，添加（参考1节）：
```json
{
  "files.encoding": "utf8",
  "files.autoGuessEncoding": false
}
```

**IntelliJ IDEA**
1. File → Settings → Editor → File Encodings
   - Global Encoding: UTF-8
   - Project Encoding: UTF-8
2. Help → Edit Custom VM Options，添加：
   ```
   -Dfile.encoding=UTF-8
   -Dconsole.encoding=UTF-8
   ```

#### 步骤3：配置Git（2分钟）

```bash
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
git config --global gui.encoding utf-8
git config --global core.autocrlf false
git config --global core.quotepath false
```

#### 步骤4：配置项目（5分钟）

**Java/Maven项目**
1. 在 `pom.xml` 添加编码配置（参考2.1节）
2. 创建 `.mvn/maven.config`（参考2.2节）
3. 创建 `mvnw-utf8.bat` 启动脚本（参考2.3节）

**Python项目**
1. 所有 `.py` 文件头部添加：`# -*- coding: utf-8 -*-`
2. 创建 `run-py-utf8.bat` 脚本（参考4.4节）

**Node.js项目**
1. 在 `package.json` 配置脚本（参考8.2节）
2. 文件读写指定 `utf8` 编码

#### 步骤5：验证配置（2分钟）

```bash
# 验证PowerShell
echo $env:JAVA_TOOL_OPTIONS

# 验证Java
java -XshowSettings:properties -version 2>&1 | findstr encoding

# 验证Python
python -c "import sys; print(sys.stdout.encoding)"

# 验证Git
git config --global --get i18n.commitencoding
```

### 日常开发（无需额外配置）

配置完成后，日常开发无需关心编码问题：

```bash
# Java/Maven项目
.\mvnw-utf8.bat clean compile
.\mvnw-utf8.bat test
java -Dfile.encoding=UTF-8 -jar target/app.jar

# Python项目
python your_script.py    # PowerShell已配置环境变量

# Node.js项目
npm start               # package.json已配置

# Git操作
git commit -m "提交消息"  # 自动使用UTF-8
```

---

## 12. 参考资料

### 官方文档
- [Spring Boot - 国际化](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.internationalization)
- [Maven - 编码配置](https://maven.apache.org/general.html#encoding-warning)
- [Python - Unicode HOWTO](https://docs.python.org/3/howto/unicode.html)
- [MySQL - 字符集支持](https://dev.mysql.com/doc/refman/8.0/en/charset.html)
- [Git - 字符编码](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git)

### 相关技术文档
- [PowerShell Profile配置](https://docs.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_profiles)
- [VS Code - 文件编码](https://code.visualstudio.com/docs/editor/codebasics#_file-encoding-support)
- [IntelliJ IDEA - 文件编码](https://www.jetbrains.com/help/idea/encoding.html)

---

## 13. 总结

Windows系统中文乱码问题的核心原因是**编码不统一**：系统默认GBK，而现代开发统一使用UTF-8。

通过本文档的配置，实现了**全栈UTF-8编码环境**：

### 配置效果

| 层级 | 配置内容 | 效果 |
|-----|---------|------|
| ✅ 文件存储 | IDE编码设置 | 所有文件统一UTF-8保存 |
| ✅ 控制台输出 | PowerShell配置 | 中文显示正常 |
| ✅ Java运行时 | JAVA_TOOL_OPTIONS | Maven/Spring Boot无乱码 |
| ✅ Python运行时 | PYTHONIOENCODING | 脚本输出中文正常 |
| ✅ Node.js运行时 | 文件读写编码 | JSON/文件操作正常 |
| ✅ 数据库 | 连接字符串 | 中文数据正确存储 |
| ✅ Git | i18n配置 | 提交消息/日志正常 |

### 关键配置点（优先级排序）

1. **PowerShell配置文件（`$PROFILE`）** ⭐⭐⭐⭐⭐ 最重要
   - 影响所有命令行操作
   - 配置一次永久生效
   - 包含所有环境变量设置

2. **IDE编码设置** ⭐⭐⭐⭐
   - 确保文件正确保存
   - 避免源文件乱码

3. **项目配置文件** ⭐⭐⭐
   - pom.xml（Java）
   - package.json（Node.js）
   - 文件头部声明（Python）

4. **Git配置** ⭐⭐
   - 提交消息编码
   - 团队协作一致性

### 验证标准

配置完成后，以下操作应全部正常：

- ✅ PowerShell启动显示 "PowerShell UTF-8 Loaded"
- ✅ Java编译中文注释无警告
- ✅ Maven运行输出中文正常
- ✅ Python脚本打印中文正确
- ✅ 日志文件中文可读
- ✅ 数据库存储中文无乱码
- ✅ Git提交消息显示正常

### 适用范围

本文档适用于以下开发环境：
- ✅ Java（Spring Boot、Maven、Gradle）
- ✅ Python（各版本）
- ✅ Node.js（npm、yarn）
- ✅ 数据库（MySQL、PostgreSQL等）
- ✅ 版本控制（Git）
- ✅ IDE（Cursor、VS Code、IntelliJ IDEA）

### 最佳实践

1. **新项目初始化时立即配置**，避免后期修复成本
2. **团队统一配置标准**，纳入项目文档
3. **定期验证编码设置**，特别是新成员加入后
4. **使用脚本封装**（`.bat`），屏蔽编码细节
5. **CI/CD环境同步配置**，确保构建一致性

---

## 附录：快速配置清单

### 最小配置（5分钟快速上手）

只需配置PowerShell，即可解决90%的乱码问题：

```powershell
# 1. 打开PowerShell配置文件
notepad $PROFILE

# 2. 复制粘贴以下内容
$OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::InputEncoding = [System.Text.Encoding]::UTF8
chcp 65001 > $null
$env:JAVA_TOOL_OPTIONS = '-Dfile.encoding=UTF-8'
$env:MAVEN_OPTS = '-Dfile.encoding=UTF-8'
$env:PYTHONIOENCODING = 'utf-8'
Write-Host "PowerShell UTF-8 Loaded" -ForegroundColor Green

# 3. 保存并重新加载
. $PROFILE
```

### 完整配置（15分钟生产就绪）

依次完成第11节的5个步骤，实现完整UTF-8环境。

---

**配置完成后，开发者无需再关心编码问题，可专注于业务开发！**

