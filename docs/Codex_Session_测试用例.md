# Codex Session Resume 测试用例

## 测试环境准备

1. 确保已安装并配置 OpenAI Codex CLI
2. 启动 WebCodeCli 应用
3. 打开浏览器访问编程助手页面

## 测试用例 1：首次请求解析 Session ID

### 测试步骤

1. 打开编程助手页面
2. 选择 "OpenAI Codex" 工具
3. 输入提示词：`创建一个简单的HTML页面，包含标题和段落`
4. 点击发送

### 预期结果

1. 系统应显示 Codex 的输出，包含版本信息和 session id
2. 后台日志应显示：
   ```
   [Information] 使用持久化进程模式执行 CLI 工具: OpenAI Codex, 会话: xxx, Codex Session: 新会话
   [Information] 向持久化进程发送初始命令: PID=xxx, Prompt长度=xxx
   [Debug] 从输出中解析到session id: 019a3994-f978-7fe2-8028-3bf4365b4af7
   [Information] 解析到Codex Session ID: 019a3994-f978-7fe2-8028-3bf4365b4af7 for 会话: xxx
   ```
3. 工作区文件列表应显示生成的 HTML 文件

### 验证点

- [x] 成功解析 session id
- [x] 生成了 HTML 文件
- [x] 日志中记录了 session id

---

## 测试用例 2：后续请求使用 Resume

### 前置条件

完成测试用例 1，确保已有 Codex session id

### 测试步骤

1. 在同一个会话中，输入新的提示词：`给这个页面添加CSS样式，使标题居中且为蓝色`
2. 点击发送

### 预期结果

1. 系统应使用 resume 命令
2. 后台日志应显示：
   ```
   [Information] 使用持久化进程模式执行 CLI 工具: OpenAI Codex, 会话: xxx, Codex Session: 019a3994-f978-7fe2-8028-3bf4365b4af7
   [Information] 向持久化进程发送resume命令: PID=xxx, Session=019a3994-f978-7fe2-8028-3bf4365b4af7, Prompt长度=xxx
   ```
3. Codex 应基于之前的 HTML 文件进行修改
4. 工作区文件应更新或新增 CSS 文件

### 验证点

- [x] 使用了 resume 命令
- [x] Codex 理解了上下文（修改了之前的文件而不是创建新的）
- [x] 日志中显示使用的 session id
- [x] 文件正确更新

---

## 测试用例 3：多次连续对话

### 前置条件

完成测试用例 2

### 测试步骤

依次发送以下提示词：

1. `添加一个按钮，点击时显示警告框`
2. `给按钮添加悬停效果`
3. `创建一个JavaScript文件，将脚本从HTML中分离`

### 预期结果

1. 每次请求都应使用相同的 session id
2. Codex 应保持对话上下文，理解"这个页面"、"这个按钮"等指代
3. 最终应有完整的 HTML + CSS + JS 文件结构

### 验证点

- [x] 所有请求使用同一个 session id
- [x] 上下文连贯，没有重复创建
- [x] 文件结构合理
- [x] 代码质量符合预期

---

## 测试用例 4：清空对话后重新开始

### 前置条件

完成测试用例 3

### 测试步骤

1. 点击"清空对话"按钮
2. 输入新的提示词：`创建一个待办事项列表应用`
3. 点击发送

### 预期结果

1. 系统应清除之前的 session id
2. 日志显示"新会话"
3. Codex 不会引用之前的文件
4. 工作区文件被清空

### 验证点

- [x] Session id 已清除
- [x] 日志显示为新会话
- [x] 工作区文件已清空
- [x] 新的 session id 被解析和保存

---

## 测试用例 5：并发用户测试

### 测试步骤

1. 打开两个浏览器窗口（或使用隐私模式）
2. 两个窗口同时使用 Codex
3. 窗口A：创建一个博客页面
4. 窗口B：创建一个登录页面
5. 窗口A：继续修改博客页面
6. 窗口B：继续修改登录页面

### 预期结果

1. 两个会话应有不同的 session id
2. 两个会话的 Codex session id 也应不同
3. 两个会话互不影响
4. 各自的工作区文件独立

### 验证点

- [x] 不同的浏览器会话 id
- [x] 不同的 Codex session id
- [x] 上下文不会混淆
- [x] 工作区文件隔离

---

## 测试用例 6：Session ID 解析失败处理

### 测试准备

临时修改代码，模拟 Codex 输出中没有 session id 的情况

### 测试步骤

1. 发送第一个请求
2. 观察系统行为

### 预期结果

1. 系统应记录警告日志：`未能从Codex输出中解析到session id`
2. 功能应继续工作，不会抛出异常
3. 后续请求会尝试重新解析

### 验证点

- [x] 记录了警告日志
- [x] 没有抛出异常
- [x] 功能仍可用

---

## 测试用例 7：长时间空闲后恢复

### 测试步骤

1. 创建一个对话，获得 session id
2. 等待 5 分钟不操作
3. 发送新的消息

### 预期结果

1. 持久化进程应仍在运行（如果在超时时间内）
2. Session id 应仍然有效
3. 能够正常使用 resume 命令

### 验证点

- [x] 进程仍在运行
- [x] Session id 有效
- [x] Resume 命令成功

---

## 测试用例 8：错误恢复

### 测试步骤

1. 创建一个对话
2. 手动终止 Codex 进程（模拟异常）
3. 发送新的消息

### 预期结果

1. 系统应检测到进程已退出
2. 应自动重启进程
3. 重新开始新的 session（因为旧的 session 已失效）
4. 记录相应的错误和恢复日志

### 验证点

- [x] 检测到进程异常
- [x] 自动重启进程
- [x] 创建新的 session
- [x] 记录了相应日志

---

## 性能测试

### 测试指标

1. **首次请求响应时间**：从发送到接收第一个输出 chunk
2. **Resume 请求响应时间**：使用 resume 时的响应速度
3. **Session ID 解析时间**：解析 session id 的耗时
4. **并发性能**：多个用户同时使用时的性能

### 性能基准

- 首次请求响应 < 2 秒
- Resume 请求响应 < 1 秒
- Session ID 解析 < 10 毫秒
- 支持至少 10 个并发用户

---

## 调试技巧

### 查看日志

在 `appsettings.json` 中设置日志级别：

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "WebCodeCli.Domain.Domain.Service.CliExecutorService": "Debug"
    }
  }
}
```

### 手动测试命令

可以在终端中手动测试 Codex 命令：

```bash
# 启动 Codex
codex

# 发送初始命令
exec --sandbox danger-full-access --json "创建一个HTML页面"

# 观察输出，复制 session id

# 发送 resume 命令
exec --sandbox danger-full-access --json "添加CSS样式" resume 019a3994-f978-7fe2-8028-3bf4365b4af7
```

### 检查会话状态

查看 `_codexSessionIds` 字典的内容（可以添加临时的日志或调试端点）

---

## 常见问题排查

### 问题：无法解析 session id

**检查项：**
1. Codex 版本是否正确
2. 输出格式是否变化
3. 正则表达式是否正确

**解决方案：**
1. 更新解析逻辑以适应新的输出格式
2. 添加更多的日志来查看原始输出

### 问题：Resume 命令失败

**检查项：**
1. Session id 是否正确
2. Session 是否已过期
3. 命令格式是否正确

**解决方案：**
1. 验证 session id 的完整性
2. 检查 Codex 的 session 超时设置
3. 查看 Codex 的错误信息

### 问题：持久化进程崩溃

**检查项：**
1. 系统资源是否充足
2. Codex 版本是否稳定
3. 是否有内存泄漏

**解决方案：**
1. 增加进程监控和自动重启
2. 升级 Codex 到稳定版本
3. 定期清理空闲进程

