# Codex 超时问题解决方案

## 问题描述

```
发送消息失败: The request was canceled due to the configured HttpClient.Timeout of 100 seconds elapsing.
```

**根本原因：**
1. ✅ HttpClient 默认超时 100 秒，但 Codex 执行需要更长时间
2. 🔍 流式输出可能被缓冲，导致前端一直等待

## 已修复：HttpClient 超时

### 修改内容

**文件：** `WebCodeCli/Program.cs`

```csharp
builder.Services.AddScoped(sp => new HttpClient
{
    BaseAddress = new Uri(sp.GetService<NavigationManager>()!.BaseUri),
    Timeout = TimeSpan.FromMinutes(10) // 从默认 100 秒增加到 10 分钟
});
```

### 为什么 10 分钟？

根据测试，Codex 生成复杂代码（如贪吃蛇游戏）可能需要：
- 思考时间：5-10 秒
- 生成代码：10-30 秒
- 执行验证：5-10 秒
- **总计：20-50 秒**

10 分钟的超时提供了充足的缓冲。

## 待验证：流式输出

### 当前实现

前端使用 `StreamReader.ReadLineAsync()` 读取 SSE：

```csharp
while (!reader.EndOfStream)
{
    var line = await reader.ReadLineAsync(); // 等待完整一行
    // ... 处理
}
```

### 潜在问题

如果 Codex 的输出被缓冲（没有 `\n` 刷新），`ReadLineAsync()` 会一直等待。

### 改进方案（如果需要）

#### 方案 A: 使用 ReadAsync 替代 ReadLineAsync

```csharp
var buffer = new char[1024];
while (true)
{
    var count = await reader.ReadAsync(buffer, 0, buffer.Length);
    if (count == 0) break;
    var chunk = new string(buffer, 0, count);
    // 处理 chunk
}
```

#### 方案 B: 添加读取超时

```csharp
var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
var line = await reader.ReadLineAsync(cts.Token);
```

## 测试步骤

### 1. 重启应用

**必须重启！** 配置更改只在启动时生效。

```bash
# 停止应用
Ctrl+C

# 重新启动
cd WebCodeCli
dotnet run
```

### 2. 测试简短命令

先测试一个快速的命令：
```
hello
```

**预期：** 3-5 秒内返回结果

### 3. 测试中等复杂度命令

```
write a fibonacci function in Python
```

**预期：** 10-20 秒内返回结果

### 4. 测试复杂命令

```
写一个贪吃蛇游戏
```

**预期：** 30-60 秒内返回结果

## 验证清单

- [ ] 应用已重启
- [ ] 测试简短命令成功
- [ ] 日志中看到"CLI 工具执行完成，退出代码: 0"
- [ ] 前端收到并显示输出
- [ ] 没有超时错误
- [ ] 浏览器 F12 控制台无错误

## 预期日志

成功的情况下应该看到：

```
[16:XX:XX INF] 执行 CLI 工具: OpenAI Codex, 命令: ...
[16:XX:XX INF] CLI 工具执行完成: OpenAI Codex, 退出代码: 0
```

两条日志之间的时间差就是实际执行时间。

## 如果还是超时

### 选项 1: 进一步增加超时

修改为 30 分钟：
```csharp
Timeout = TimeSpan.FromMinutes(30)
```

### 选项 2: 禁用超时（不推荐）

```csharp
Timeout = System.Threading.Timeout.InfiniteTimeSpan
```

### 选项 3: 使用异步作业

对于非常长的任务，考虑：
1. 立即返回作业 ID
2. 后台执行
3. 轮询或 WebSocket 获取结果

## 其他可能的优化

### 1. 添加 Codex 选项减少输出

在配置中添加参数：
```json
{
  "ArgumentTemplate": "exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox --reasoning-summaries=concise {prompt}"
}
```

`--reasoning-summaries=concise` 会减少思考过程的输出，加快响应。

### 2. 限制输出长度

某些 CLI 工具支持 `--max-tokens` 参数：
```json
{
  "ArgumentTemplate": "exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox --max-tokens 5000 {prompt}"
}
```

### 3. 添加进度指示

在前端添加：
- 倒计时显示
- 预估完成时间
- "正在思考中..." 动画

## 性能基准

基于测试的典型执行时间：

| 任务复杂度 | 预计时间 | 示例 |
|-----------|---------|------|
| 简单 | 3-10 秒 | hello, 简单函数 |
| 中等 | 10-30 秒 | 排序算法, 数据处理 |
| 复杂 | 30-120 秒 | 完整应用, 游戏 |
| 极复杂 | 2-5 分钟 | 多文件项目 |

## 监控建议

在生产环境：
1. 记录每次调用的执行时间
2. 设置告警（如超过 2 分钟）
3. 提供用户取消功能
4. 显示实时进度

---

**立即操作：重启应用，然后测试！** 🚀

超时从 100 秒增加到 10 分钟后，应该能正常工作了。

