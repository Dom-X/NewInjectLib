# QiuChenly 应用商店 - 安装说明

## 项目简介

QiuChenly 应用商店是一个基于 HTTP 服务的 macOS 应用注入工具商店，以系统守护进程方式运行，提供 Web 界面管理和使用各种 macOS 应用注入脚本。

文档篇幅较长，很详细，善用搜索，记得看完。

## 预览

![main](imgs/浅色模式.png)
![main1](imgs/深色模式.png)
![main2](imgs/单独App.png)

**主要特点：**

- 无需关闭 SIP（系统完整性保护），在 SIP 开启状态下即可正常使用
- 系统守护进程方式运行，开机自动启动
- Web 界面访问，方便直观
- 支持数百款 macOS 应用的注入和破解
- **支持临时运行**：无需安装系统服务，可直接命令行启动测试

**使用方式：**

- **临时运行**：适合测试，无需安装，命令行启动即可
- **安装服务**：推荐长期使用，开机自启，自动重启

---

## 系统要求

- **操作系统**：macOS 10.15 及以上版本
- **权限要求**：需要管理员权限（sudo）进行安装
- **网络端口**：15200 端口（用于 Web 界面访问）
- **SIP 状态**：无需关闭，保持开启状态即可

---

## 安装前准备

### 1. 确认以下文件存在

- `InjectLib` - 主程序二进制文件
- `config.json` - 应用配置文件
- `tool/` - 工具目录
- `frontend/` - 前端界面文件

### 2. 确认文件完整性

在安装前，请确保项目目录包含以下关键文件：

```
InjectLib/
├── InjectLib          # 主程序（编译生成）
├── config.json        # 配置文件
├── tool/              # 工具目录
├── frontend/          # 前端界面
└── res/               # 资源目录
    ├── install_daemon.sh      # 安装脚本
    ├── uninstall_daemon.sh    # 卸载脚本
    └── com.qiuchenly.hayaku.daemon.plist  # 守护进程配置
```

---

## 临时运行（无需安装）

如果您只是想测试或临时使用，可以不必安装系统服务，直接使用命令行临时启动：

### 如何临时运行

1. **打开终端**
   - 按下快捷键 **Command ⌘ + 空格**
   - 输入 **终端** 并打开

2. **进入项目目录**
   ```bash
   cd /Volumes/MySSD/InjectLib
   # 或者你的项目实际路径
   ```

3. **直接运行程序**
   ```bash
   sudo ./InjectLib --daemon config.json
   ```

4. **保持终端窗口打开**
   - 服务会在当前终端窗口中运行
   - 不要关闭终端窗口
   - 按 **Ctrl + C** 可以停止服务

5. **访问 Web 界面**
   - 打开浏览器访问：`http://localhost:15200`

### 临时运行的优缺点

**优点：**
- 无需安装，快速测试
- 不修改系统配置
- 停止后不留任何痕迹
- 适合临时使用或测试

**缺点：**
- 关闭终端窗口后服务会停止
- 不会开机自动启动
- 进程异常退出后不会自动重启
- 需要手动管理进程
- 仍然需要配置系统权限（完全磁盘访问 + App管理）

### 重要提示

**即使临时运行，也必须授予系统权限：**

1. **配置完全磁盘访问权限**
   - 系统设置 → 隐私与安全性 → 完全磁盘访问权限
   - 添加项目路径下的 `InjectLib` 程序

2. **配置 App 管理权限**
   - 系统设置 → 隐私与安全性 → App管理
   - 添加项目路径下的 `InjectLib` 程序

详细的权限配置方法请参考后面的"安装后配置"章节。

---

## 详细安装步骤（推荐：开机自启）

如果您希望服务开机自动启动、异常退出后自动重启，建议按照以下步骤安装系统服务。

### 步骤 1：进入项目目录

```bash
cd InjectShell/res
```

或者您的项目实际所在路径：

```bash
cd /path/to/InjectShell/res
```

### 步骤 2：赋予安装脚本执行权限

```bash
chmod +x install_daemon.sh
```

### 步骤 3：执行安装脚本

**重要：必须使用 sudo 权限运行**

```bash
sudo bash install_daemon.sh
```

### 步骤 4：等待安装完成

安装过程会自动执行以下操作：

1. 创建安装目录：`/usr/local/bin/QiuChenly/`
2. 复制所有文件到系统目录：
   - `tool/` → `/usr/local/bin/QiuChenly/tool/`
   - `config.json` → `/usr/local/bin/QiuChenly/config.json`
   - `frontend/` → `/usr/local/bin/QiuChenly/frontend/`
   - `InjectLib` → `/usr/local/bin/QiuChenly/InjectLib`
3. 安装守护进程配置：`com.qiuchenly.hayaku.daemon.plist` → `/Library/LaunchDaemons/`
4. 加载并启动服务

安装成功后，您会看到如下提示：

```
安装Hayaku HTTP守护进程...
复制二进制文件到 /usr/local/bin/QiuChenly/
安装LaunchDaemon配置...
启动服务...
安装完成！
服务状态:
xxxxx  0  com.qiuchenly.hayaku.daemon

访问 http://localhost:15200 查看应用商店
查看日志: tail -f /var/log/hayaku_daemon*
```

---

## 安装后配置（必须执行）

### ⚠️ 授予程序必要的系统权限

**这是最重要的步骤！如果不授权，服务将无法正常工作！**

#### 为什么需要这些权限？

QiuChenly 应用商店的主程序 `InjectLib` 需要以下权限才能正常运行：

- **完全磁盘访问权限**：用于读取和修改应用程序包内的文件
- **App管理权限**：用于管理和注入目标应用程序

#### 如何授予权限

**注意：必须授权给安装后的程序路径 `/usr/local/bin/QiuChenly/InjectLib`**

---

**步骤 1：配置完全磁盘访问权限**

##### 方法 A：图形界面操作（推荐）

1. **打开系统设置**
   - 点击屏幕左上角的 **苹果图标** 
   - 选择 **系统设置**（或 **系统偏好设置**，取决于 macOS 版本）

2. **进入隐私与安全性**
   - 在左侧边栏中找到并点击 **隐私与安全性**
   - 在右侧向下滚动，找到 **完全磁盘访问权限**
   - 点击进入

3. **解锁设置**
   - 点击左下角的 **锁图标**
   - 输入您的 **管理员密码**
   - 点击 **解锁**

4. **添加 InjectLib 程序**
   - 点击应用列表下方的 **+** 按钮
   - 在弹出的文件选择窗口中，按下键盘快捷键 **Command ⌘ + Shift ⇧ + G**
   - 在弹出的"前往文件夹"输入框中输入：`/usr/local/bin/QiuChenly/`
   - 点击 **前往**
   - 选中 **InjectLib** 文件
   - 点击 **打开**

5. **确认开关状态**
   - 在应用列表中找到 **InjectLib**
   - 确保它旁边的开关是 **✓ 蓝色/绿色开启状态**
   - 如果是关闭状态，点击开关将其打开

6. **锁定设置**
   - 点击左下角的 **锁图标** 锁定设置

##### 方法 B：使用终端命令快速打开

如果您熟悉终端，可以直接执行以下命令打开对应设置页面：

```bash
open "x-apple.systempreferences:com.apple.preference.security?Privacy_AllFiles"
```

然后按照方法 A 的步骤 3-6 操作即可。

---

**步骤 2：配置 App 管理权限**

##### 方法 A：图形界面操作（推荐）

1. **打开系统设置**
   - 点击屏幕左上角的 **苹果图标** 
   - 选择 **系统设置**（或 **系统偏好设置**）

2. **进入隐私与安全性**
   - 在左侧边栏中找到并点击 **隐私与安全性**
   - 在右侧向下滚动，找到 **App管理**（或 **自动化**，取决于 macOS 版本）
   - 点击进入

3. **解锁设置**
   - 点击左下角的 **锁图标**
   - 输入您的 **管理员密码**
   - 点击 **解锁**

4. **添加 InjectLib 程序**
   - 点击应用列表下方的 **+** 按钮
   - 在弹出的文件选择窗口中，按下键盘快捷键 **Command ⌘ + Shift ⇧ + G**
   - 在弹出的"前往文件夹"输入框中输入：`/usr/local/bin/QiuChenly/`
   - 点击 **前往**
   - 选中 **InjectLib** 文件
   - 点击 **打开**

5. **确认开关状态**
   - 在应用列表中找到 **InjectLib**
   - 确保它旁边的开关是 **✓ 蓝色/绿色开启状态**
   - 如果是关闭状态，点击开关将其打开

6. **锁定设置**
   - 点击左下角的 **锁图标** 锁定设置

##### 方法 B：使用终端命令快速打开

如果您熟悉终端，可以直接执行以下命令打开对应设置页面：

```bash
open "x-apple.systempreferences:com.apple.preference.security?Privacy_AppManagement"
```

然后按照方法 A 的步骤 3-6 操作即可。

---

**步骤 3：重启服务使权限生效**

配置完权限后，需要重启服务才能使权限生效。

##### 方法 A：图形界面操作（推荐）

1. **打开"活动监视器"**
   - 按下键盘快捷键 **Command ⌘ + 空格** 打开聚焦搜索
   - 输入 **活动监视器**
   - 按下 **回车键** 打开

2. **查找并结束 InjectLib 进程**
   - 在右上角的搜索框中输入：**InjectLib**
   - 找到 **InjectLib** 进程
   - 双击该进程（或选中后点击上方的关闭按钮）
   - 点击 **退出** 或 **强制退出**

3. **等待服务自动重启**
   - LaunchDaemon 会自动重启服务（大约 1-2 秒）
   - 您可以在活动监视器中看到 InjectLib 进程再次出现

##### 方法 B：使用终端命令

如果您熟悉终端，可以执行以下命令：

```bash
sudo launchctl stop com.qiuchenly.hayaku.daemon
sudo launchctl start com.qiuchenly.hayaku.daemon
```

#### 验证权限和服务

##### 方法 A：图形界面验证（推荐）

**1. 检查服务是否运行**

- 打开 **活动监视器**（Command ⌘ + 空格，输入"活动监视器"）
- 在右上角搜索框输入：**InjectLib**
- 如果能看到 **InjectLib** 进程在运行，说明服务已启动

**2. 访问 Web 界面**

- 打开您的浏览器（Safari、Chrome 等）
- 在地址栏输入：`http://localhost:15200`
- 如果能看到应用商店界面，说明服务正常运行

**3. 检查日志文件（可选）**

方法一：使用控制台应用
- 按下快捷键 **Command ⌘ + 空格**，输入 **控制台** 并打开
- 在左侧边栏选择 **系统日志**
- 在右上角搜索框输入：**hayaku_daemon**
- 查看日志中是否有 `Operation not permitted` 或 `Permission denied` 等权限错误
- 如果没有权限错误，说明配置成功

方法二：使用终端查看
```bash
# 查看错误日志
sudo tail -n 20 /var/log/hayaku_daemon_error.log

# 实时查看日志（按 Ctrl+C 停止）
sudo tail -f /var/log/hayaku_daemon_error.log
```

##### 方法 B：使用终端验证

如果您熟悉终端，可以执行以下命令：

```bash
# 查看服务状态
sudo launchctl list | grep com.qiuchenly.hayaku.daemon

# 测试运行主程序
sudo /usr/local/bin/QiuChenly/InjectLib --daemon /usr/local/bin/QiuChenly/config.json
# 如果能正常启动说明权限配置正确，按 Ctrl+C 停止

# 检查日志是否有权限错误
tail -n 20 /var/log/hayaku_daemon_error.log

# 在浏览器中打开 Web 界面
open http://localhost:15200
```

##### 验证成功的标志

如果一切正常，您应该能够：
- 在活动监视器中看到 InjectLib 进程运行
- 访问 `http://localhost:15200` 看到 Web 界面
- 日志文件中没有权限相关的错误
- 成功注入应用程序

#### 重要提示

- **必须授权给 `/usr/local/bin/QiuChenly/InjectLib`（安装后的路径）**
- **两个权限都必须配置**：完全磁盘访问 + App管理
- **每次更新程序后都需要重新授权**
- 授权后必须重启服务才能生效
- 权限授予后会永久生效，除非手动撤销

---

## 重要说明

### 安装后可以安全删除项目源文件夹

**安装采用文件复制方式**，所有必要的文件都会复制到 `/usr/local/bin/QiuChenly/` 目录中：

| 文件/目录        | 源路径                 | 目标路径                                  | 方式   |
| ---------------- | ---------------------- | ----------------------------------------- | ------ |
| tool 工具目录    | `项目目录/tool/`       | `/usr/local/bin/QiuChenly/tool/`          | 复制 |
| config.json 配置 | `项目目录/config.json` | `/usr/local/bin/QiuChenly/config.json`    | 复制 |
| frontend 前端    | `项目目录/frontend/`   | `/usr/local/bin/QiuChenly/frontend/`      | 复制 |
| InjectLib 程序   | `项目目录/InjectLib`   | `/usr/local/bin/QiuChenly/InjectLib`      | 复制 |

安装完成后，**服务完全独立运行**，不再依赖项目源文件夹。您可以：

- 安全地删除项目源文件夹
- 移动项目文件夹到其他位置
- 重命名项目文件夹

### 如何验证安装

安装完成后，可以执行以下命令验证文件：

```bash
ls -la /usr/local/bin/QiuChenly/
```

正常输出应该包含：

```
drwxr-xr-x  tool/
-rw-r--r--  config.json
drwxr-xr-x  frontend/
-rwxr-xr-x  InjectLib
```

所有文件都是实际复制的文件，不是符号链接

---

## 验证安装

### 1. 检查服务状态

```bash
# 查看守护进程是否运行
sudo launchctl list | grep com.qiuchenly.hayaku.daemon
```

输出示例：

```
12345  0  com.qiuchenly.hayaku.daemon
```

- 第一列是进程 ID（PID）
- 第二列是退出状态码（0 表示正常）
- 第三列是服务标识符

### 2. 检查端口监听

```bash
# 查看15200端口是否被监听
lsof -i :15200
```

输出示例：

```
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
InjectLib 12345 root  3u  IPv4 0x...  0t0  TCP localhost:15200 (LISTEN)
```

### 3. 访问 Web 界面

打开浏览器，访问：

```
http://localhost:15200
```

如果能正常打开应用商店界面，说明安装成功！

---

## 使用说明

### 访问应用商店

安装成功后，服务会自动在后台运行，您可以：

1. **打开浏览器**，访问 `http://localhost:15200`
2. **浏览应用列表**，查看支持的所有应用
3. **选择应用**，点击注入按钮进行安装
4. **查看状态**，实时监控注入进度

### 服务特点

- **开机自启**：系统启动时自动运行，无需手动启动
- **自动重启**：如果服务异常退出，LaunchDaemon 会自动重启
- **系统级服务**：以 root 权限运行，确保注入功能正常
- **Web 界面**：通过浏览器访问，跨平台兼容

---

## 日志管理

### 查看运行日志

系统会自动记录服务的运行日志和错误日志。

#### 方法 A：使用控制台应用（推荐）

1. **打开控制台应用**
   - 按下快捷键 **Command ⌘ + 空格**
   - 输入 **控制台** 并打开

2. **查看系统日志**
   - 在左侧边栏选择 **系统日志**
   - 在右上角搜索框输入：**hayaku_daemon**
   - 可以实时查看日志更新

3. **筛选日志**
   - 点击搜索框旁边的筛选按钮
   - 可以按照时间、级别（错误、警告、信息）筛选

#### 方法 B：使用终端命令

```bash
# 实时查看标准输出日志
sudo tail -f /var/log/hayaku_daemon.log

# 实时查看错误日志
sudo tail -f /var/log/hayaku_daemon_error.log

# 查看最近100行日志
sudo tail -n 100 /var/log/hayaku_daemon.log

# 查看所有日志
sudo cat /var/log/hayaku_daemon.log
```

**注意：** 查看 `/var/log/` 目录下的文件需要使用 `sudo` 管理员权限。

### 日志文件说明

| 日志文件 | 路径                               | 内容                   | 查看方式              |
| -------- | ---------------------------------- | ---------------------- | --------------------- |
| 标准输出 | `/var/log/hayaku_daemon.log`       | 正常运行信息、调试信息 | 控制台应用或终端+sudo |
| 错误输出 | `/var/log/hayaku_daemon_error.log` | 错误信息、异常堆栈     | 控制台应用或终端+sudo |

---

## 服务管理

### 手动控制服务

虽然服务会自动启动，但您也可以手动控制：

#### 停止服务

```bash
sudo launchctl stop com.qiuchenly.hayaku.daemon
```

#### 启动服务

```bash
sudo launchctl start com.qiuchenly.hayaku.daemon
```

#### 重启服务

```bash
sudo launchctl stop com.qiuchenly.hayaku.daemon
sudo launchctl start com.qiuchenly.hayaku.daemon
```

#### 卸载服务（不删除文件）

```bash
sudo launchctl unload /Library/LaunchDaemons/com.qiuchenly.hayaku.daemon.plist
```

#### 重新加载服务（不删除文件）

```bash
sudo launchctl load /Library/LaunchDaemons/com.qiuchenly.hayaku.daemon.plist
```

### 查看服务配置

```bash
# 查看LaunchDaemon配置文件
cat /Library/LaunchDaemons/com.qiuchenly.hayaku.daemon.plist
```

---

## 卸载说明

如果您需要完全卸载 QiuChenly 应用商店：

### 步骤 1：进入项目目录

**重要：必须在原始安装的项目目录中执行卸载！**

```bash
cd /path/to/InjectShell/res
```

### 步骤 2：执行卸载脚本

```bash
sudo bash uninstall_daemon.sh
```

### 步骤 3：确认卸载

卸载脚本会自动执行：

1. 停止服务
2. 卸载 LaunchDaemon 配置
3. 删除系统中的所有文件和软链接
4. 删除日志文件

卸载完成后输出：

```
卸载Hayaku HTTP守护进程...
停止服务...
删除文件...
卸载完成！
```

### 步骤 4：验证卸载

```bash
# 检查服务是否还在运行
sudo launchctl list | grep com.qiuchenly.hayaku.daemon
# 应该没有任何输出

# 检查系统文件是否已删除
ls -la /usr/local/bin/QiuChenly/
# 应该提示目录不存在

# 或者检查整个安装路径
ls -d /usr/local/bin/QiuChenly 2>/dev/null
# 应该没有任何输出
```

### 卸载说明

卸载操作会：

- 删除 `/usr/local/bin/QiuChenly/` 目录及所有文件
- 删除 LaunchDaemon 配置文件
- 删除日志文件
- 项目源文件夹**不会被删除**（如果您还保留着的话）

卸载完成后，项目源文件夹可以保留供以后重新安装使用

---

## 重新安装

### 重要提示

**当前版本暂未开发重新安装功能**，如果需要重新安装，请按以下步骤操作：

---

## 故障排查

### 问题 1：服务无法正常工作或注入失败

**症状：**

- 服务启动了，但无法注入应用
- 日志中显示权限错误
- 提示"Operation not permitted"
- 无法访问应用程序包

**原因：** 主程序 `InjectLib` 没有必要的系统权限

**解决方案：**

#### 步骤 1：检查当前权限状态

```bash
# 检查服务状态
sudo launchctl list | grep com.qiuchenly.hayaku.daemon

# 查看错误日志
tail -n 50 /var/log/hayaku_daemon_error.log
```

如果日志中出现以下关键词，说明是权限问题：
- `Operation not permitted`
- `Permission denied`
- `sandbox`
- `TCC`

#### 步骤 2：为安装后的程序授权

**重要：安装后程序位于 `/usr/local/bin/QiuChenly/InjectLib`，需要给这个位置的程序授权！**

**A. 配置完全磁盘访问权限**

图形界面操作：
1. 点击屏幕左上角的 **苹果图标** → **系统设置**
2. 点击 **隐私与安全性** → **完全磁盘访问权限**
3. 点击左下角 **🔒** 输入密码解锁
4. 点击 **+** 按钮
5. 按 **Command ⌘ + Shift ⇧ + G**，输入：`/usr/local/bin/QiuChenly/`
6. 选择 **InjectLib** 并添加
7. 确保开关是 **✓ 开启状态**

终端快捷方式（可选）：
```bash
# 打开系统设置（然后按照上面的步骤操作）
open "x-apple.systempreferences:com.apple.preference.security?Privacy_AllFiles"
```

**B. 配置 App 管理权限**

图形界面操作：
1. 点击屏幕左上角的 **苹果图标** → **系统设置**
2. 点击 **隐私与安全性** → **App管理**
3. 点击左下角 **🔒** 输入密码解锁
4. 点击 **+** 按钮
5. 按 **Command ⌘ + Shift ⇧ + G**，输入：`/usr/local/bin/QiuChenly/`
6. 选择 **InjectLib** 并添加
7. 确保开关是 **✓ 开启状态**

终端快捷方式（可选）：
```bash
# 打开系统设置（然后按照上面的步骤操作）
open "x-apple.systempreferences:com.apple.preference.security?Privacy_AppManagement"
```

#### 步骤 3：重启服务

图形界面操作：
1. 打开 **活动监视器**（Command ⌘ + 空格，输入"活动监视器"）
2. 搜索 **InjectLib**
3. 选中进程，点击 **退出**
4. 等待服务自动重启（1-2秒）

终端命令（可选）：
```bash
sudo launchctl stop com.qiuchenly.hayaku.daemon
sudo launchctl start com.qiuchenly.hayaku.daemon
```

#### 步骤 4：验证权限

图形界面验证：
1. 打开浏览器访问 `http://localhost:15200`
2. 在活动监视器中确认 InjectLib 进程运行正常
3. 打开"控制台"应用（Command ⌘ + 空格，输入"控制台"），搜索 **hayaku_daemon** 检查是否有权限错误

终端验证（可选）：
```bash
# 测试运行
sudo /usr/local/bin/QiuChenly/InjectLib --daemon /usr/local/bin/QiuChenly/config.json
# 按 Ctrl+C 停止

# 如果能正常启动且没有权限错误，说明配置成功
```

#### 常见权限错误示例

```
# 错误1：没有完全磁盘访问权限
Error: Failed to access application bundle
Operation not permitted

# 错误2：没有 App 管理权限  
Error: Failed to inject into process
sandbox: denied(1) process-exec

# 解决方法：按照上面的步骤授权即可
```

---

### 问题 2：执行安装脚本提示权限不足

**错误信息：**

```
请使用sudo运行此脚本
```

**解决方案：**

```bash
# 确保使用 sudo 命令
sudo bash install_daemon.sh
```

---

### 问题 3：找不到 InjectLib 文件

**错误信息：**

```
错误: 找不到 InjectLib 文件
```

**原因：** 下载文件有问题

**解决方案：**

```bash
# 下载后重新安装
cd ../res
sudo bash install_daemon.sh
```

---

### 问题 4：服务无法启动

**检查方法：**

```bash
# 查看错误日志
tail -f /var/log/hayaku_daemon_error.log

# 检查服务状态
sudo launchctl list | grep com.qiuchenly.hayaku.daemon
```

**可能原因和解决方案：**

#### 原因 1：端口 15200 已被占用

```bash
# 查看谁占用了15200端口
lsof -i :15200

# 如果是其他程序，关闭该程序或修改配置使用其他端口
```

#### 原因 2：文件缺失或损坏

```bash
# 检查安装文件是否完整
ls -la /usr/local/bin/QiuChenly/

# 应该包含：InjectLib, tool/, config.json, frontend/
# 如果文件缺失，需要重新安装
```

#### 原因 3：权限问题

```bash
# 检查文件权限
ls -la /usr/local/bin/QiuChenly/InjectLib

# 确保InjectLib有执行权限
sudo chmod +x /usr/local/bin/QiuChenly/InjectLib

# 重启服务
sudo launchctl stop com.qiuchenly.hayaku.daemon
sudo launchctl start com.qiuchenly.hayaku.daemon
```

---

### 问题 5：无法访问 http://localhost:15200

**检查清单：**

1. **服务是否运行？**

```bash
sudo launchctl list | grep com.qiuchenly.hayaku.daemon
# 应该有输出，且第二列为0
```

2. **端口是否监听？**

```bash
lsof -i :15200
# 应该显示InjectLib正在监听
```

3. **防火墙是否阻止？**

```bash
# 打开系统偏好设置 → 安全性与隐私 → 防火墙
# 检查是否阻止了InjectLib
```

4. **浏览器是否正确？**

- 尝试使用不同的浏览器（Safari、Chrome、Firefox）
- 清除浏览器缓存
- 尝试使用 `curl http://localhost:15200` 测试

---

### 问题 6：修改配置文件或工具后不生效

**症状：**

- 修改了项目源文件夹中的 `config.json` 或 `tool/` 目录，但服务没有变化

**原因：**

安装时使用的是文件复制，服务运行的是 `/usr/local/bin/QiuChenly/` 中的文件，不是源文件夹中的文件。

**解决方案：**

修改配置后需要重新安装：

```bash
# 1. 停止服务
sudo launchctl stop com.qiuchenly.hayaku.daemon

# 2. 手动更新文件
sudo cp /path/to/InjectShell/config.json /usr/local/bin/QiuChenly/config.json
# 或者更新工具目录
sudo cp -r /path/to/InjectShell/tool /usr/local/bin/QiuChenly/

# 3. 启动服务
sudo launchctl start com.qiuchenly.hayaku.daemon
```

或者完全重新安装：

```bash
cd /path/to/InjectShell/res
sudo bash uninstall_daemon.sh
sudo bash install_daemon.sh
```

---

### 问题 7：卸载后仍能访问 Web 界面

**原因：** 服务可能还在运行，或浏览器缓存

**解决方案：**

```bash
# 1. 强制停止服务
sudo launchctl stop com.qiuchenly.hayaku.daemon

# 2. 查找并杀死进程
ps aux | grep InjectLib
sudo kill -9 <PID>

# 3. 清除浏览器缓存或使用无痕模式测试
```

---

### 问题 8：更新项目后功能异常

**原因：** 服务使用的是安装时复制的文件，不会自动更新

**解决方案：**

```bash
# 1. 重新编译项目（如果修改了源代码）
cd /path/to/InjectShell/build
make clean
cmake ..
make

# 2. 停止服务
sudo launchctl stop com.qiuchenly.hayaku.daemon

# 3. 更新系统中的文件
sudo cp /path/to/InjectShell/InjectLib /usr/local/bin/QiuChenly/InjectLib
sudo chmod +x /usr/local/bin/QiuChenly/InjectLib

# 如果修改了配置或工具，也需要更新
sudo cp /path/to/InjectShell/config.json /usr/local/bin/QiuChenly/config.json
sudo cp -r /path/to/InjectShell/tool /usr/local/bin/QiuChenly/
sudo cp -r /path/to/InjectShell/frontend /usr/local/bin/QiuChenly/

# 4. 启动服务
sudo launchctl start com.qiuchenly.hayaku.daemon
```

**或者更简单的方法：重新安装**

```bash
cd /path/to/InjectShell/res
sudo bash uninstall_daemon.sh
sudo bash install_daemon.sh
```

---

## 获取帮助

如果遇到文档中未涵盖的问题，可以：

1. **查看项目 README**：`readme.md`
2. **加入 Telegram 群组**：https://t.me/+VvqTr-2EFaZhYzA1
3. **关注 Telegram 频道**：https://t.me/qiuchenlymac
4. **加入 QQ 群**：
   - 一群：1042610918（已满）
   - 二群：1049674046
5. **访问在线文档**：https://qiuchenlyopensource.github.io/Documentaions/
6. **提交 Issue**：在 GitHub 仓库提交问题

---

## 更新日志

### 当前版本功能

- LaunchDaemon 守护进程安装
- 文件复制安装（所有文件独立运行）
- 安装后可安全删除源文件夹
- 系统级服务运行
- Web 界面访问（端口 15200）
- 自动启动和崩溃重启
- 日志记录功能
- 完整的卸载功能

### 最近更新（v2.0）

**安装机制重大变更：**

- 从软链接改为文件复制方式
- 统一安装目录到 `/usr/local/bin/QiuChenly/`
- 安装后完全独立运行，不再依赖源文件夹
- 简化了部署和维护流程

### 待开发功能

- 一键重新安装功能
- 配置文件热更新
- 在线更新功能

---

## 许可证

本项目遵循项目根目录中的 LICENSE 文件。

**重要提醒：**

- 本项目仅供学习和研究使用
- 请勿用于商业用途
- 请支持正版软件

---

## 致谢

本项目由 QiuChenly 创意制作，暂时不必向任何人致谢。

---

**文档版本：** v2.0  
**最后更新：** 2025-10-14  
**适用项目版本：** InjectShell 4.0+  
**主要变更：** 安装机制从软链接改为文件复制

---
