# GenClaw

轻量、安全、可扩展的个人 AI Agent 系统。运行在本地 Mac 上，通过 WhatsApp / Telegram / 飞书等即时通讯频道接收消息，在隔离的 Docker 容器中运行 AI Agent（支持 Claude / Qwen 等模型），并将结果回复到对应群组。

---

## 架构概览

```
消息频道（WhatsApp / Telegram / 飞书）
        ↓
   宿主进程（Node.js）
   ├── 消息路由 & 持久化（SQLite）
   ├── 任务调度器（cron / interval）
   └── 容器管理器
              ↓
        Docker 容器（隔离沙箱）
        └── agent-runner
              ├── Claude Agent SDK
              └── LLM Provider（OpenAI / Anthropic）
```

- **宿主进程**：监听消息、路由分发、管理容器生命周期
- **agent-runner**：容器内运行，接收任务、调用 LLM、返回结果
- **Group 隔离**：每个群组有独立的文件目录、会话状态和容器实例

---

## 快速开始

### 环境要求

- Node.js >= 20
- Docker Desktop（已启动）
- 至少一个频道的 API 凭据

### 安装

```bash
git clone git@github.com:deng1986/genclaw.git
cd genclaw
npm install
```

### 配置

复制并编辑 `.env`：

```bash
cp .env.example .env
```

`.env` 关键配置项：

```env
# AI 助手名字（触发词，默认 @Andy）
ASSISTANT_NAME=Andy

# LLM 提供商：openai 或 anthropic
LLM_PROVIDER=openai

# OpenAI / 兼容接口（如阿里云百炼）
OPENAI_API_KEY=sk-xxx
OPENAI_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
OPENAI_MODEL=qwen3-coder-plus

# Anthropic（Claude）
# ANTHROPIC_API_KEY=sk-ant-xxx

# Telegram（可选）
TELEGRAM_BOT_TOKEN=xxx

# 飞书（可选）
FEISHU_APP_ID=xxx
FEISHU_APP_SECRET=xxx
```

### 构建容器镜像

```bash
docker build -f container/Dockerfile container/ -t genclaw-agent:latest
```

### 启动

```bash
# 开发模式（热重载）
npm run dev

# 生产模式
npm run build && npm start
```

---

## 使用方式

### 触发 Agent

在已注册的群组里发送：

```
@Andy 帮我查一下今天的天气
```

### 主控群组（isMain）

主控群组无需触发词，Agent 可以管理其他群组的注册、任务调度等。

---

## 项目结构

```
genclaw/
├── src/                    # 宿主进程源码
│   ├── index.ts            # 主入口，消息循环
│   ├── config.ts           # 配置项
│   ├── container-runner.ts # 容器启动 & 通信
│   ├── container-runtime.ts# Docker 运行时管理
│   ├── channels/           # 频道适配器（Telegram、飞书等）
│   ├── db.ts               # SQLite 数据库
│   ├── task-scheduler.ts   # 定时任务调度
│   └── types.ts            # 类型定义
├── container/
│   ├── Dockerfile          # Agent 容器镜像
│   └── agent-runner/       # 容器内运行的 Agent
│       └── src/
│           ├── index.ts    # Agent 入口
│           └── llm-provider.ts  # LLM 适配层
├── groups/                 # 群组配置目录
│   ├── global/CLAUDE.md    # 全局 Agent 指令
│   └── main/CLAUDE.md      # 主控群组指令
├── setup/                  # 初始化脚本
└── skills-engine/          # Skill 管理引擎
```

---

## 安全机制

- **容器隔离**：每个 Agent 任务在独立 Docker 容器中运行，无法访问宿主文件系统
- **挂载白名单**：`~/.config/genclaw/mount-allowlist.json` 控制哪些目录允许挂载，且该文件本身不会挂载进容器
- **发送者白名单**：`~/.config/genclaw/sender-allowlist.json` 限制哪些用户可触发 Agent
- **输出大小限制**：容器输出默认最大 10MB，防止异常输出

---

## 支持的 LLM

| 提供商 | 模型示例 | 配置方式 |
|--------|----------|----------|
| 阿里云百炼 | `qwen3-coder-plus` | `LLM_PROVIDER=openai` + 百炼 base URL |
| Anthropic | `claude-opus-4-5` | `LLM_PROVIDER=anthropic` |
| OpenAI | `gpt-4o` | `LLM_PROVIDER=openai` |
| Ollama（本地） | `llama3` | 通过 Skill 扩展 |

---

## 开发

```bash
# 类型检查
npm run typecheck

# 运行测试
npm test

# 格式化代码
npm run format
```

---

## License

MIT
