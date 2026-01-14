# 编程助手快速启动指南

## 快速开始

### 1. 安装依赖

确保已安装 .NET 10 SDK：

```bash
dotnet --version
```

### 2. 恢复 NuGet 包

```bash
cd WebCodeCli
dotnet restore
```

### 3. 运行应用

```bash
dotnet run
```

应用将在 `http://localhost:5000` 启动。

### 4. 访问编程助手

在浏览器中打开：`http://localhost:5000/code-assistant`

## 测试功能

### 使用内置测试工具

项目已配置了一个测试回显工具，在开发环境下默认启用：

1. 在 CLI 工具下拉框中选择"测试回显工具"
2. 在输入框中输入任何问题，例如：
   - "如何创建一个 ASP.NET Core API？"
   - "生成一个排序算法"
   - "解释什么是依赖注入"
3. 点击"发送"或按 `Ctrl+Enter`
4. 查看右侧预览区的输出

### 配置真实的 CLI 工具

#### GitHub Copilot CLI

1. 安装 GitHub CLI：
   ```bash
   # Windows (使用 winget)
   winget install GitHub.cli
   
   # 或使用 Chocolatey
   choco install gh
   ```

2. 安装 Copilot 扩展：
   ```bash
   gh extension install github/gh-copilot
   ```

3. 在 `appsettings.Development.json` 中启用：
   ```json
   {
     "Id": "copilot-cli",
     "Enabled": true
   }
   ```

4. 重启应用并测试

#### 其他 CLI 工具

根据需要安装其他编程助手 CLI 工具，并在配置文件中添加相应配置。

## 功能测试清单

- [ ] 发送消息并接收响应
- [ ] 流式输出正常显示
- [ ] 切换不同的 CLI 工具
- [ ] 代码预览 Tab 显示语法高亮
- [ ] Markdown 渲染 Tab 正确渲染格式
- [ ] 原始输出 Tab 显示完整内容
- [ ] 清空对话功能正常
- [ ] 错误处理和提示正常

## 常见问题

### PowerShell 执行策略错误

如果遇到 PowerShell 执行策略错误，运行：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 测试工具无法执行

确保 PowerShell 脚本路径正确：
- 脚本位置：`WebCodeCli/wwwroot/test-cli-tools/echo-assistant.ps1`
- 工作目录应该是项目根目录

### Monaco Editor 未加载

1. 检查网络连接（需要访问 CDN）
2. 查看浏览器控制台是否有错误
3. 尝试刷新页面

## 下一步

1. 配置您常用的编程助手 CLI 工具
2. 根据需要调整超时和并发设置
3. 自定义界面样式和布局
4. 添加更多功能和扩展

## 开发调试

### 查看日志

应用使用 Serilog 记录日志，日志会输出到控制台。

### 调试 CLI 执行

在 `CliExecutorService.cs` 中设置断点，查看：
- CLI 工具配置
- 参数构建
- 进程启动
- 流式输出

### 前端调试

使用浏览器开发者工具：
- Console：查看 JavaScript 错误
- Network：监控 API 请求
- Elements：检查 DOM 结构

## 部署到生产环境

1. 更新 `appsettings.json` 配置
2. 禁用测试工具
3. 配置真实的 CLI 工具
4. 设置适当的超时和并发限制
5. 启用 HTTPS
6. 配置日志和监控

```bash
dotnet publish -c Release
```

## 性能优化建议

1. **流式输出**：已实现，无需额外优化
2. **并发控制**：通过 `MaxConcurrentExecutions` 配置
3. **超时处理**：为每个工具设置合理的超时时间
4. **资源清理**：进程会自动清理，无需手动处理

## 获取帮助

查看完整文档：`README_CodeAssistant.md`

祝您使用愉快！🚀

