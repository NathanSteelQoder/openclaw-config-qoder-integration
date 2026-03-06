---
name: openclaw-qoder-integration
description: 将 OpenClaw 与 Qoder CLI 对接的完整配置指南。适用于已有 OpenClaw 环境的用户，快速配置 ACP 协议支持 Qoder CLI。
---

# OpenClaw 对接 Qoder CLI 配置指南

> 让 OpenClaw 拥有 AI 编程能力 —— 5 分钟完成 ACP 协议对接

## 你能获得什么

对接完成后，你可以：
- 💬 **在聊天中直接调用 Qoder** —— "用 qoder 开发一个博客系统"
- 🚀 **自动代码生成** —— 描述需求，Qoder 自动编写代码
- 📊 **Spec 模式开发** —— 先出设计文档，再按规范实现
- 🔄 **持久化会话** —— 多轮对话保持上下文

## 前置条件

- ✅ OpenClaw 已安装并运行 (`openclaw gateway status` 显示正常)
- ✅ Qoder API Key（从 [qoder.com](https://qoder.com) 获取）

## 三步完成对接

### 第一步：启用 ACPX 插件

```bash
# 安装 ACPX 插件
openclaw plugins install @openclaw/acpx

# 启用插件
openclaw config set plugins.entries.acpx.enabled true

# 验证安装
openclaw plugins list | grep acpx
```

**预期输出：**
```
│ ACPX Runtime │ acpx     │ loaded   │ stock:acpx/index.ts │
```

### 第二步：配置 ACP 协议

```bash
# 1. 启用 ACP 功能
openclaw config set acp.enabled true

# 2. 启用 ACP 调度器
openclaw config set acp.dispatch.enabled true

# 3. 设置后端为 acpx
openclaw config set acp.backend "acpx"

# 4. 设置默认 Agent 为 qoder
openclaw config set acp.defaultAgent "qoder"

# 5. 允许 qoder Agent
openclaw config set acp.allowedAgents '["pi","claude","codex","opencode","gemini","qoder"]'

# 6. 重启 Gateway 生效
openclaw gateway restart
```

### 第三步：配置 Qoder CLI

```bash
# 1. 安装 Qoder CLI
npm install -g @qoder-ai/qodercli

# 2. 创建 acpx 配置
mkdir -p ~/.acpx

cat > ~/.acpx/config.json << 'EOF'
{
  "agents": {
    "qoder": {
      "command": "env QODER_PERSONAL_ACCESS_TOKEN=你的API密钥 qodercli --acp"
    }
  },
  "defaultAgent": "qoder",
  "defaultPermissions": "approve-all"
}
EOF
```

**获取 API 密钥：**
1. 访问 [qoder.com](https://qoder.com)
2. 登录后进入 Settings → API Keys
3. 复制你的 Personal Access Token
4. 替换上面配置中的 `你的API密钥`

## 验证对接成功

### 方法 1：查看配置
```bash
openclaw config get acp --raw
```

**预期输出：**
```json
{
  "enabled": true,
  "dispatch": { "enabled": true },
  "backend": "acpx",
  "defaultAgent": "qoder",
  "allowedAgents": ["pi", "claude", "codex", "opencode", "gemini", "qoder"]
}
```

### 方法 2：在聊天中测试

在你的 OpenClaw 聊天渠道（飞书/钉钉/Discord 等）输入：

```
/acp spawn qoder --mode persistent
```

**成功标志：**
- 返回会话创建成功的消息
- 显示 session key（如 `agent:qoder:acp:xxx`）

### 方法 3：自然语言调用

```
用 qoder 开发一个 TODO list 应用，用 Next.js + TypeScript
```

OpenClaw 会自动识别并调用 Qoder 执行开发任务。

## 常用命令速查

| 命令 | 作用 |
|------|------|
| `/acp spawn qoder` | 启动 Qoder 会话 |
| `/acp status` | 查看会话状态 |
| `/acp sessions` | 列出所有会话 |
| `/acp close` | 关闭当前会话 |
| `/acp steer 继续开发` | 给 Qoder 发指令 |

## 使用示例

### 示例 1：快速开发
```
用 qoder 帮我写个 Python 脚本，批量重命名文件夹里的图片
```

### 示例 2：Spec 模式开发
```
/quest 开发一个个人博客系统，技术栈 Next.js + MDX
```

### 示例 3：代码审查
```
用 qoder 审查 ./src 目录的代码，检查 TypeScript 类型安全
```

### 示例 4：持久会话开发
```
/acp spawn qoder --mode persistent
```
然后持续对话：
```
帮我开发一个电商网站
→ 先设计数据库模型
→ 然后写后端 API
→ 最后做前端页面
```

## 常见问题

### Q1: 提示 "Agent not allowed"
**解决：** 检查 allowedAgents 配置是否包含 "qoder"
```bash
openclaw config set acp.allowedAgents '["pi","claude","codex","opencode","gemini","qoder"]'
```

### Q2: Qoder 提示未登录
**解决：** 确认 acpx 配置中的 API Key 正确
```bash
# 验证 API Key 是否有效
export QODER_PERSONAL_ACCESS_TOKEN=你的密钥
qodercli status
```

### Q3: ACP 会话启动后立即关闭
**解决：** 检查 Gateway 日志
```bash
openclaw logs --tail 50
```

### Q4: 飞书中输入命令没反应
**解决：** 
1. 确认飞书渠道配置正确
2. 检查 OpenClaw 是否已连接飞书
3. 查看是否有错误日志

## 进阶配置

### 配置线程绑定（可选）
```bash
# 启用会话绑定
openclaw config set session.threadBindings.enabled true
openclaw config set session.threadBindings.ttlHours 24

openclaw gateway restart
```

### 配置多个 AI Agent
```bash
# 添加更多 Agent（如果有其他 ACP 兼容工具）
openclaw config set acp.allowedAgents '["qoder","codex","claude"]'

# 使用时指定
/acp spawn codex   # 使用 Codex
/acp spawn qoder   # 使用 Qoder
```

### 安全加固
```bash
# 设置权限级别
openclaw config set acp.permissions "moderate"  # strict | moderate | approve-all

# 设置超时时间（秒）
openclaw config set acp.timeout 300
```

## 架构原理

```
你 (飞书/钉钉/Discord)
    ↓
OpenClaw Gateway
    ↓  ACP 协议
ACPX 插件
    ↓  stdio
Qoder CLI
    ↓  HTTP
Qoder AI 服务
```

- **ACP (Agent Client Protocol)**: 标准化代理通信协议
- **ACPX**: OpenClaw 的 ACP 后端实现
- **Qoder CLI**: 接收 ACP 指令，调用 Qoder AI 服务

## 故障排查清单

如果对接失败，按顺序检查：

1. [ ] OpenClaw Gateway 运行中？
   ```bash
   openclaw gateway status
   ```

2. [ ] ACPX 插件已加载？
   ```bash
   openclaw plugins list | grep acpx
   ```

3. [ ] ACP 配置正确？
   ```bash
   openclaw config get acp --raw
   ```

4. [ ] Qoder CLI 能独立运行？
   ```bash
   qodercli status
   ```

5. [ ] API Key 有效？
   ```bash
   export QODER_PERSONAL_ACCESS_TOKEN=xxx
   qodercli -p "hello" --max-turns 1
   ```

6. [ ] acpx 配置正确？
   ```bash
   cat ~/.acpx/config.json
   ```

## 相关资源

- [OpenClaw 文档](https://docs.openclaw.ai)
- [Qoder 官网](https://qoder.com)
- [ACP 协议](https://agentclientprotocol.com)
- [Qoder CLI GitHub](https://github.com/qoder-ai/qoder-cli)
