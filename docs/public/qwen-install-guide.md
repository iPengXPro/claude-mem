# 阿里云 Qwen 模型配置指南

本指南介绍如何在本地安装 claude-mem 并配置使用阿里云 Qwen 模型作为观察者。

## 前置要求

- **Node.js** (v18 或更高版本)
- **Bun** (自动安装)
- **uv** (自动安装，用于提供 Python 环境)
- **Git**

## 步骤 1：克隆代码仓库

```bash
cd C:\Users\AAA\temp  # 或其他你喜欢的目录
git clone <repository-url> claude-mem
cd claude-mem
```

## 步骤 2：安装依赖

```bash
npm install
```

## 步骤 3：构建项目

```bash
npm run build
```

构建完成后，会在 `plugin/scripts/` 目录生成以下文件：
- `worker-service.cjs` - Worker 服务
- `mcp-server.cjs` - MCP 服务器
- `context-generator.cjs` - 上下文生成器

## 步骤 4：同步到插件目录

### Windows (使用 PowerShell 或手动复制)

```powershell
# 复制 plugin 目录到 marketplace
Copy-Item -Path ".\plugin\*" -Destination "$env:USERPROFILE\.claude\plugins\marketplaces\thedotmack\plugin" -Recurse -Force

# 复制 plugin 目录到 cache (带版本号)
$version = (Get-Content ".\plugin\.claude-plugin\plugin.json" | ConvertFrom-Json).version
Copy-Item -Path ".\plugin\*" -Destination "$env:USERPROFILE\.claude\plugins\cache\thedotmack\claude-mem\$version\plugin" -Recurse -Force
```

### 或者手动复制

1. 复制 `plugin/` 文件夹到 `~/.claude/plugins/marketplaces/thedotmack/plugin`
2. 复制 `plugin/` 文件夹到 `~/.claude/plugins/cache/thedotmack/claude-mem/<version>/`

## 步骤 5：配置阿里云 Qwen 模型

编辑 `~/.claude-mem/settings.json` 文件（Windows 路径：`C:\Users\<你的用户名>\.claude-mem\settings.json`）：

```json
{
  "CLAUDE_MEM_PROVIDER": "openrouter",
  "CLAUDE_MEM_OPENROUTER_API_KEY": "你的阿里云-DashScope-API-Key",
  "CLAUDE_MEM_OPENROUTER_MODEL": "qwen3.5-plus",
  "CLAUDE_MEM_OPENROUTER_BASE_URL": "https://coding.dashscope.aliyuncs.com/v1/chat/completions",
  "CLAUDE_MEM_OPENROUTER_SITE_URL": "",
  "CLAUDE_MEM_OPENROUTER_APP_NAME": "claude-mem"
}
```

### 获取阿里云 API Key

1. 访问阿里云控制台：https://dashscope.console.aliyun.com/
2. 登录账号
3. 进入 **API Key 管理** 页面
4. 创建新的 API Key 或使用已有的 Key
5. 确保已开通 **Qwen 模型服务**
6. 复制 API Key 到配置文件中

### 配置项说明

| 配置项 | 说明 | 示例值 |
|--------|------|--------|
| `CLAUDE_MEM_PROVIDER` | 提供者，必须设置为 `openrouter` | `openrouter` |
| `CLAUDE_MEM_OPENROUTER_API_KEY` | 阿里云 DashScope API Key | `sk-xxxxxxxxxxxxxxxx` |
| `CLAUDE_MEM_OPENROUTER_MODEL` | 使用的模型 | `qwen3.5-plus` |
| `CLAUDE_MEM_OPENROUTER_BASE_URL` | API 端点地址 | `https://coding.dashscope.aliyuncs.com/v1/chat/completions` |
| `CLAUDE_MEM_OPENROUTER_SITE_URL` | 可选，站点 URL | 留空 |
| `CLAUDE_MEM_OPENROUTER_APP_NAME` | 应用名称 | `claude-mem` |

## 步骤 6：启动 Worker 服务

### 方法 1：通过 Claude Code 自动启动

当你在项目中使用 Claude Code 时，Worker 服务会自动启动。

### 方法 2：手动启动

```bash
bun run ~/.claude/plugins/marketplaces/thedotmack/plugin/scripts/worker-service.cjs --daemon
```

或从缓存目录启动：

```bash
bun run ~/.claude/plugins/cache/thedotmack/claude-mem/<version>/scripts/worker-service.cjs --daemon
```

## 步骤 7：验证配置

检查 Worker 服务是否正常运行：

```bash
curl http://localhost:37777/health
```

返回 `{"status":"ok"}` 表示服务正常。

查看设置是否正确加载：

```bash
curl http://localhost:37777/api/settings
```

检查输出中是否包含正确的配置。

## 步骤 8：查看日志

日志文件位于 `~/.claude-mem/logs/` 目录：

```bash
# 查看最新日志
tail -50 ~/.claude-mem/logs/claude-mem-$(date +%Y-%m-%d).log
```

成功的 API 调用日志示例：

```
[INFO] [SDK] OpenRouter API usage {model=qwen3.5-plus, inputTokens=5734, outputTokens=677, totalTokens=6411, estimatedCostUSD=0.0274}
```

## 常见问题

### 1. 401 认证错误

**错误信息**：`OpenRouter API error: 401 - {"error":{"message":"Incorrect API key provided"}}`

**解决方案**：
- 检查 API Key 是否正确
- 确认 API Key 格式为 `sk-` 开头
- 登录阿里云控制台验证 Key 的有效性
- 确保已开通 Qwen 模型服务

### 2. Worker 服务无法启动

**解决方案**：
- 确保 Bun 已正确安装：`bun --version`
- 检查端口 37777 是否被占用
- 查看日志文件获取详细错误信息

### 3. 构建失败

**解决方案**：
- 删除 `node_modules` 和 `plugin/` 目录
- 重新运行 `npm install`
- 重新运行 `npm run build`

### 4. 配置文件位置

| 系统 | 配置文件路径 |
|------|-------------|
| Windows | `C:\Users\<用户名>\.claude-mem\settings.json` |
| macOS/Linux | `~/.claude-mem/settings.json` |

## 可用模型

阿里云 DashScope 提供的兼容 OpenAI 格式的模型：

| 模型 | 配置值 |
|------|--------|
| Qwen3.5-Plus | `qwen3.5-plus` |
| Qwen3-Plus | `qwen-plus` |
| Qwen3-Turbo | `qwen-turbo` |
| Qwen-Max | `qwen-max` |

修改 `CLAUDE_MEM_OPENROUTER_MODEL` 即可切换模型。

## 更新代码

当代码仓库有更新时：

```bash
# 1. 拉取最新代码
git pull origin main

# 2. 安装依赖（如有需要）
npm install

# 3. 重新构建
npm run build

# 4. 同步到插件目录
# Windows PowerShell
Copy-Item -Path ".\plugin\*" -Destination "$env:USERPROFILE\.claude\plugins\marketplaces\thedotmack\plugin" -Recurse -Force

# 5. 重启 Worker
curl -X POST http://localhost:37777/api/admin/restart
```

## 参考资源

- [阿里云 DashScope 文档](https://help.aliyun.com/zh/model-studio/)
- [claude-mem 公开文档](https://docs.claude-mem.ai)
- [GitNexus 代码分析工具](https://github.com/thedotmack/gitnexus)
