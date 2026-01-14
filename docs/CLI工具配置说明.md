# CLI 工具配置说明

## 问题：找不到 CLI 命令

当您看到类似这样的错误时：

```
System.ComponentModel.Win32Exception (2): An error occurred trying to start process 'codex' with working directory 'D:\git\WebCodeCli\WebCodeCli'. 系统找不到指定的文件。
```

这表示系统无法找到指定的 CLI 命令。

## 解决方案

### 1. 确认 CLI 工具的安装位置

使用 PowerShell 查找命令：

```powershell
Get-Command codex | Select-Object -ExpandProperty Source
```

输出示例：
```
C:\Users\28651\AppData\Roaming\npm\codex.ps1
```

### 2. 识别命令类型

#### A. PowerShell 脚本 (.ps1)
如果命令是 PowerShell 脚本，需要通过 PowerShell 执行：

**配置示例：**
```json
{
  "Id": "codex",
  "Name": "OpenAI Codex",
  "Command": "powershell",
  "ArgumentTemplate": "-NoProfile -ExecutionPolicy Bypass -Command \"codex {prompt}\"",
  "Enabled": true
}
```

#### B. 可执行文件 (.exe)
如果是可执行文件，可以直接使用：

**配置示例：**
```json
{
  "Id": "gh-copilot",
  "Name": "GitHub Copilot",
  "Command": "gh",
  "ArgumentTemplate": "copilot suggest {prompt}",
  "Enabled": true
}
```

#### C. 批处理文件 (.bat / .cmd)
批处理文件可以直接使用：

**配置示例：**
```json
{
  "Id": "my-tool",
  "Name": "My Tool",
  "Command": "cmd",
  "ArgumentTemplate": "/c my-tool.bat {prompt}",
  "Enabled": true
}
```

### 3. 常见 CLI 工具配置

#### Node.js / npm 全局包

大多数 npm 全局安装的 CLI 工具在 Windows 上会创建两个文件：
- `命令.cmd` - Windows 批处理文件
- `命令.ps1` - PowerShell 脚本

**推荐配置：**
```json
{
  "Command": "powershell",
  "ArgumentTemplate": "-NoProfile -ExecutionPolicy Bypass -Command \"命令 {prompt}\""
}
```

或者使用完整路径：
```json
{
  "Command": "C:\\Users\\用户名\\AppData\\Roaming\\npm\\命令.cmd",
  "ArgumentTemplate": "{prompt}"
}
```

#### Python CLI 工具

**配置示例：**
```json
{
  "Command": "python",
  "ArgumentTemplate": "-m 模块名 {prompt}"
}
```

或使用完整路径：
```json
{
  "Command": "C:\\Python\\Scripts\\命令.exe",
  "ArgumentTemplate": "{prompt}"
}
```

## 当前配置的 CLI 工具

### 1. 测试回显工具 ✅ 已启用
```json
{
  "Id": "test-echo",
  "Name": "测试回显工具",
  "Command": "powershell",
  "ArgumentTemplate": "-ExecutionPolicy Bypass -File wwwroot/test-cli-tools/echo-assistant.ps1 -prompt {prompt}",
  "Enabled": true
}
```

**用途：** 本地测试工具，无需额外安装。

### 2. OpenAI Codex ✅ 已启用
```json
{
  "Id": "codex",
  "Name": "OpenAI Codex",
  "Command": "powershell",
  "ArgumentTemplate": "-NoProfile -ExecutionPolicy Bypass -Command \"codex {prompt}\"",
  "Enabled": true
}
```

**前提：** 需要安装 codex CLI 工具（npm 包）。

### 3. 其他工具 ❌ 已禁用
- Claude Code
- QwenCode
- Gemini CLI
- GitHub Copilot CLI

这些工具默认禁用，需要安装后再启用。

## 如何启用新的 CLI 工具

### 步骤 1：安装 CLI 工具

例如安装 GitHub Copilot CLI：
```bash
gh extension install github/gh-copilot
```

### 步骤 2：查找命令路径

```powershell
Get-Command gh | Select-Object -ExpandProperty Source
```

### 步骤 3：更新配置文件

编辑 `appsettings.Development.json`：

```json
{
  "Id": "copilot-cli",
  "Name": "GitHub Copilot CLI",
  "Command": "gh",
  "ArgumentTemplate": "copilot suggest {prompt}",
  "Enabled": true,  // 改为 true
  "TimeoutSeconds": 300
}
```

### 步骤 4：重启应用

停止并重新启动应用以加载新配置。

## 调试技巧

### 1. 测试命令是否可用

在 PowerShell 中手动执行：
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "codex '你好'"
```

### 2. 查看详细错误日志

应用日志会显示详细的执行信息：
```
[INF] 执行 CLI 工具: OpenAI Codex, 命令: powershell -NoProfile ...
[ERR] 启动 CLI 进程失败: ...
```

### 3. 使用完整路径

如果命令仍然找不到，使用完整路径：
```json
{
  "Command": "C:\\Users\\28651\\AppData\\Roaming\\npm\\codex.ps1",
  "ArgumentTemplate": "{prompt}"
}
```

## 参数转义说明

系统会自动转义用户输入以防止命令注入。但在配置模板时需要注意：

### Windows (PowerShell)
```json
"ArgumentTemplate": "-Command \"codex {prompt}\""
```
- 使用 `\"` 转义双引号
- `{prompt}` 会被自动转义

### Linux/Mac (Bash)
```json
"ArgumentTemplate": "-c 'codex {prompt}'"
```
- 使用单引号包裹
- `{prompt}` 会被自动转义

## 环境变量配置

某些 CLI 工具需要特定的环境变量：

```json
{
  "Id": "my-tool",
  "Command": "my-tool",
  "ArgumentTemplate": "{prompt}",
  "EnvironmentVariables": {
    "API_KEY": "your-api-key",
    "MODEL": "gpt-4"
  }
}
```

## 故障排除清单

- [ ] CLI 工具已安装
- [ ] 可以在命令行手动执行
- [ ] 配置文件语法正确（JSON 格式）
- [ ] Command 路径正确
- [ ] ArgumentTemplate 正确转义
- [ ] Enabled 设置为 true
- [ ] 应用已重启以加载新配置

## 获取帮助

如果遇到问题：
1. 查看应用日志获取详细错误信息
2. 在命令行手动测试 CLI 命令
3. 检查 CLI 工具的官方文档
4. 使用测试回显工具验证基本功能

---

**更新配置后请重启应用！**

