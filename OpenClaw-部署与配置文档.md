# OpenClaw 部署与使用配置文档

> 项目仓库：https://github.com/hh-macro/openclaw

---

## 1. 环境准备

### 1.1 Node.js（要求 ≥ 22.12.0）

推荐使用 [NVM for Windows](https://github.com/coreybutler/nvm-windows) 管理 Node 版本：

```cmd
:: 安装指定版本
nvm install 22.22.1

:: 切换到该版本
nvm use 22.22.1

:: 验证版本
node -v
```

### 1.2 pnpm

> 每次切换 Node 版本后，pnpm 需要重新安装

```cmd
npm install -g pnpm

:: 验证安装
pnpm -v
```

### 1.3 代理配置（可选）

国内网络环境建议配置代理（以 Clash 端口 7890 为例）：

```cmd
:: 设置当前 CMD 会话的代理
set HTTP_PROXY=http://127.0.0.1:7890
set HTTPS_PROXY=http://127.0.0.1:7890
```

---

## 2. 项目安装

### 2.1 克隆仓库

```cmd
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 2.2 安装依赖

```cmd
pnpm install
```

### 2.3 构建控制台 UI (可选)

```cmd
pnpm ui:build
```

构建产物输出到 `dist/control-ui/`，即 WebChat 控制台界面。

---

## 3. 配置文件说明

### 3.1 文件位置

| 文件 | 路径 | 用途 |
|------|------|------|
| 主配置 | `C:\Users\<用户名>\.openclaw\openclaw.json` | 模型、Agent、网关配置 |
| 环境变量 | `C:\Users\<用户名>\.openclaw\.env` | API Key 等敏感信息 |
| 工作目录 | `C:\Users\<用户名>\.openclaw\workspace` | Agent 工作文件存放 |

### 3.2 主配置文件结构

```json
{
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "你的访问令牌"
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "提供商名/模型ID"
      },
      "workspace": "C:\\Users\\<用户名>\\.openclaw\\workspace"
    }
  },
  "models": {
    "providers": {
      "提供商名": {
        "baseUrl": "API 基础地址",
        "apiKey": "API Key",
        "api": "openai-completions",
        "models": [
          {
            "id": "模型ID",
            "name": "模型显示名称"
          }
        ]
      }
    }
  }
}
```

### 3.3 字段说明

| 字段 | 说明 |
|------|------|
| `gateway.mode` | 网关模式，本地部署填 `local` |
| `gateway.auth.token` | 访问令牌，用于客户端连接鉴权 |
| `agents.defaults.model.primary` | 默认使用的模型，格式为 `提供商名/模型ID` |
| `agents.defaults.workspace` | Agent 工作目录（Windows 路径需双反斜杠） |
| `models.providers.<name>` | 自定义提供商名称，避免与内置名称冲突 |
| `models.providers.<name>.api` | API 协议类型，兼容 OpenAI 格式填 `openai-completions` |

---

## 4. 接入 AI 模型

### 4.1 DeepSeek

DeepSeek 兼容 OpenAI API 格式，接入最稳定

**配置示例：**

```json
{
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "a2c2d2547f9fff4005e1f41da686fc9d9eec206389b15495"
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "deepseek/deepseek-chat"
      },
      "workspace": "C:\\Users\\macroh\\.openclaw\\workspace"
    }
  },
  "models": {
    "providers": {
      "deepseek": {
        "baseUrl": "https://api.deepseek.com/v1",
        "apiKey": "sk-abcdefg1234356",
        "api": "openai-completions",
        "models": [
          {
            "id": "deepseek-chat",
            "name": "DeepSeek Chat"
          }
        ]
      }
    }
  }
}
```

> **关键注意**：`providers` 的键名（如 `deepseek`）不要使用 `openai`，否则会与内置 OpenAI 提供商冲突，导致模型找不到

### 4.2 其他兼容 OpenAI 格式的服务

只需修改 `baseUrl`、`apiKey` 和 `models` 即可接入其他服务：

| 服务 | baseUrl |
|------|---------|
| DeepSeek | `https://api.deepseek.com/v1` |
| 硅基流动 | `https://api.siliconflow.cn/v1` |
| Groq | `https://api.groq.com/openai/v1` |
| OpenRouter | `https://openrouter.ai/api/v1` |

---

## 5. 启动与使用

### 5.1 启动 Gateway

打开第一个 CMD 窗口：

```cmd
cd E:\你的项目路径\openclaw
pnpm openclaw gateway --port 18789 --verbose
```

启动成功后会显示：

```
Gateway running at ws://127.0.0.1:18789
WebChat UI at http://127.0.0.1:18789
```

### 5.2 命令行发送消息

打开第二个 CMD 窗口：

```cmd
cd E:\你的项目路径\openclaw
pnpm openclaw agent --message "你好，介绍一下你自己" --session-id main
```

| 参数 | 说明 |
|------|------|
| `--message` | 发送的消息内容 |
| `--session-id` | 会话 ID，同一 ID 共享上下文 |

### 5.3 WebChat 控制台

浏览器访问：`http://127.0.0.1:18789`

在登录界面输入配置文件中的 `gateway.auth.token` 即可使用图形化对话界面

---

## 6. Telegram链接

**第一步：创建 Telegram Bot**

在 Telegram 搜索 `@BotFather`，发送 `/newbot`，按提示输入 Bot 名称（显示名）和用户名（必须以 `bot` 结尾，如 `my_openclaw_bot`）。BotFather 返回一串 Token，格式类似 `123456789:ABCdefGHI...`，复制保存好，这是你 Bot 的唯一凭证

**第二步：修改 openclaw.json**

```bash
"channels": {
  "telegram": {
    "enabled": true,
    "botToken": "8227545999:AAGrCDsqoJXn......o",
    "dmPolicy": "pairing",
    "allowFrom": [你的数字ID]
  }
}
```

**第三步：重启 Gateway**

```bash
# Ctrl+C  停止当前 Gateway
pnpm openclaw gateway --port 18789 --verbose
```

**第四步：配对自己的账号**

```
pnpm openclaw agent --message "openclaw pairing approve telegram ABC123" --session-id main
```

或者可以直接在配置中输入TelegramId 也能配对

### 6.1 完整配置 Json

```json
{
  "models": {
    "providers": {
      "deepseek": {
        "baseUrl": "https://api.deepseek.com/v1",
        "apiKey": "token......",
        "api": "openai-completions",
        "models": [
          {
            "id": "deepseek-chat",
            "name": "DeepSeek Chat"
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "deepseek/deepseek-chat"
      },
      "workspace": "C:\\Users\\macroh\\.openclaw\\workspace",
      "compaction": {
        "mode": "safeguard"
      }
    }
  },
  "commands": {
    "native": "auto",
    "nativeSkills": "auto",
    "restart": true,
    "ownerDisplay": "raw"
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "botToken": "8227545999:AAGrCDs.....",
      "allowFrom": [
        8305935514
      ],
      "groupPolicy": "allowlist",
      "streaming": "partial"
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "lan",
    "controlUi": {
      "allowedOrigins": [
        "http://localhost:18789",
        "http://127.0.0.1:18789"
      ]
    },
    "auth": {
      "mode": "token",
      "token": "a2c2d2547f9fff4005e1f41da686fc9d9eec206389b15495"
    }
  },
  "meta": {
    "lastTouchedVersion": "2026.3.9",
    "lastTouchedAt": "2026-03-12T02:16:40.829Z"
  }
}

```

