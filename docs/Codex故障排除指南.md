# Codex CLI 故障排除指南

## 错误：系统找不到指定的文件

```
System.ComponentModel.Win32Exception (2): An error occurred trying to start process 'codex' with working directory 'D:\git\WebCodeCli\WebCodeCli'. 系统找不到指定的文件。
```

### 原因分析

这个错误表示应用无法找到 `codex` 命令。可能的原因：

1. **配置未重新加载** ⚠️ 最常见
2. **PATH 环境变量问题**
3. **权限问题**
4. **命令路径不正确**

## 解决步骤

### 步骤 1: 重启应用 🔄

**这是最重要的步骤！**

ASP.NET Core 应用在启动时加载配置文件，修改配置后必须重启应用。

#### Visual Studio:
1. 按 `Shift+F5` 停止调试
2. 按 `F5` 重新启动

#### 终端:
```bash
# 按 Ctrl+C 停止应用
# 然后重新运行
cd WebCodeCli
dotnet run
```

### 步骤 2: 验证配置已加载

重启后，访问：
```
http://localhost:5000/api/chat/tools
```

查看返回的 JSON，确认 Codex 的配置：
```json
{
  "id": "codex",
  "command": "codex",  // 应该是 "codex" 而不是 "powershell"
  "argumentTemplate": "exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox {prompt}"
}
```

### 步骤 3: 测试命令行

在 PowerShell 中测试：

```powershell
# 测试 codex 是否可用
codex --version

# 测试完整命令
codex exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox "你好"
```

如果这些命令能工作，说明 codex 安装正确。

### 步骤 4: 检查 PATH

如果命令行能工作但应用不行，可能是 PATH 问题：

```powershell
# 查看 codex 位置
Get-Command codex | Select-Object -ExpandProperty Source

# 输出示例：
# C:\Users\28651\AppData\Roaming\npm\codex.ps1
```

### 步骤 5: 使用完整路径（如果需要）

如果重启后仍然无法找到命令，修改配置使用完整路径：

#### 配置 A: 使用 .cmd 文件

```json
{
  "Id": "codex",
  "Name": "OpenAI Codex",
  "Description": "OpenAI Codex 代码生成",
  "Command": "C:\\Users\\28651\\AppData\\Roaming\\npm\\codex.cmd",
  "ArgumentTemplate": "exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox {prompt}",
  "WorkingDirectory": "",
  "Enabled": true,
  "TimeoutSeconds": 300,
  "EnvironmentVariables": {}
}
```

#### 配置 B: 通过 PowerShell 调用

```json
{
  "Id": "codex",
  "Name": "OpenAI Codex",
  "Description": "OpenAI Codex 代码生成",
  "Command": "powershell",
  "ArgumentTemplate": "-NoProfile -ExecutionPolicy Bypass -Command \"& 'C:\\Users\\28651\\AppData\\Roaming\\npm\\codex.ps1' exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox '{prompt}'\"",
  "WorkingDirectory": "",
  "Enabled": true,
  "TimeoutSeconds": 300,
  "EnvironmentVariables": {}
}
```

**注意：** 替换路径中的用户名为您自己的用户名。

### 步骤 6: 添加 npm 到 PATH

如果 npm 全局包不在 PATH 中，可以添加：

```powershell
# 获取 npm 全局路径
npm config get prefix

# 添加到当前 PowerShell 会话
$env:PATH += ";C:\Users\28651\AppData\Roaming\npm"

# 永久添加到系统 PATH（需要管理员权限）
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\Users\28651\AppData\Roaming\npm", "User")
```

重启应用后 PATH 更改才会生效。

## 验证清单

完成以下步骤后再测试：

- [ ] 已修改 `appsettings.Development.json`
- [ ] **已重启应用**（重要！）
- [ ] 在命令行能成功运行 `codex --version`
- [ ] 访问 `/api/chat/tools` 确认配置已加载
- [ ] 查看应用启动日志，确认没有配置加载错误

## 常见错误

### 错误 1: "stdout is not a terminal"

**原因：** 使用了交互式模式而不是 `exec` 子命令

**解决：** 确保 `ArgumentTemplate` 包含 `exec`

### 错误 2: "Not inside a trusted directory"

**原因：** 缺少 `--skip-git-repo-check` 标志

**解决：** 确保 `ArgumentTemplate` 包含该标志

### 错误 3: 退出代码 1 但无输出

**原因：** 命令参数不正确或缺少必要标志

**解决：** 使用完整的参数模板：
```
exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox {prompt}
```

## 调试技巧

### 1. 查看详细日志

修改 `appsettings.Development.json`：

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "WebCodeCli.Domain": "Debug"
    }
  }
}
```

### 2. 手动测试进程启动

在 C# 代码中添加调试输出：

```csharp
_logger.LogInformation("命令: {Command}", tool.Command);
_logger.LogInformation("参数: {Arguments}", arguments);
_logger.LogInformation("工作目录: {WorkingDirectory}", startInfo.WorkingDirectory);
```

### 3. 测试最小配置

创建一个简单的测试工具验证进程启动：

```json
{
  "Id": "test-cmd",
  "Name": "测试CMD",
  "Command": "cmd",
  "ArgumentTemplate": "/c echo {prompt}",
  "Enabled": true
}
```

如果这个能工作，说明进程启动机制正常。

## 成功标志

应用正常工作时，日志应该显示：

```
[INF] 执行 CLI 工具: OpenAI Codex, 命令: codex exec --skip-git-repo-check...
[INF] CLI 工具执行完成: OpenAI Codex, 退出代码: 0
```

退出代码为 **0** 表示成功。

## 需要帮助？

如果以上步骤都无法解决问题：

1. 检查 `codex login` 状态
2. 确认 Codex API 额度充足
3. 查看完整的错误日志
4. 尝试使用测试回显工具验证基础功能

## 快速测试脚本

创建一个 PowerShell 脚本来测试所有配置：

```powershell
# test-codex.ps1

Write-Host "=== Codex 配置测试 ===" -ForegroundColor Cyan

# 1. 检查命令
Write-Host "`n1. 检查 codex 命令..." -ForegroundColor Yellow
try {
    $codexPath = (Get-Command codex -ErrorAction Stop).Source
    Write-Host "✓ Codex 路径: $codexPath" -ForegroundColor Green
} catch {
    Write-Host "✗ 找不到 codex 命令" -ForegroundColor Red
    exit 1
}

# 2. 测试版本
Write-Host "`n2. 测试 codex 版本..." -ForegroundColor Yellow
$version = codex --version
Write-Host "✓ 版本: $version" -ForegroundColor Green

# 3. 测试执行
Write-Host "`n3. 测试 codex exec..." -ForegroundColor Yellow
$result = codex exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox "echo hello" 2>&1
if ($LASTEXITCODE -eq 0) {
    Write-Host "✓ 执行成功" -ForegroundColor Green
} else {
    Write-Host "✗ 执行失败，退出代码: $LASTEXITCODE" -ForegroundColor Red
    Write-Host $result
}

Write-Host "`n=== 测试完成 ===" -ForegroundColor Cyan
```

运行：
```powershell
.\test-codex.ps1
```

---

**记住：修改配置后务必重启应用！** 🔄

