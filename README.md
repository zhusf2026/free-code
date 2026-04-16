<p align="center">
  <img src="assets/screenshot.png" alt="free-code" width="720" />
</p>

<h1 align="center">free-code</h1>

<p align="center">
  <strong>Claude Code 的自由构建版本。</strong><br>
  所有遥测已移除。所有护栏已拆除。所有实验功能已解锁。<br>
  一个二进制文件，零回传。
</p>

<p align="center">
  <a href="#快速安装"><img src="https://img.shields.io/badge/install-one--liner-blue?style=flat-square" alt="Install" /></a>
  <a href="https://github.com/paoloanzn/free-code/stargazers"><img src="https://img.shields.io/github/stars/paoloanzn/free-code?style=flat-square" alt="Stars" /></a>
  <a href="https://github.com/paoloanzn/free-code/issues"><img src="https://img.shields.io/github/issues/paoloanzn/free-code?style=flat-square" alt="Issues" /></a>
  <a href="https://github.com/paoloanzn/free-code/blob/main/FEATURES.md"><img src="https://img.shields.io/badge/features-88%20flags-orange?style=flat-square" alt="Feature Flags" /></a>
  <a href="#ipfs-镜像"><img src="https://img.shields.io/badge/IPFS-mirrored-teal?style=flat-square" alt="IPFS" /></a>
</p>

---

## 快速安装

```bash
curl -fsSL https://raw.githubusercontent.com/paoloanzn/free-code/main/install.sh | bash
```

检查您的系统，如需要则安装 Bun，克隆仓库，启用所有实验功能构建，并在您的 PATH 上建立 `free-code` 符号链接。

然后运行 `free-code` 并使用 `/login` 命令通过您偏好的模型提供商进行认证。

---

## 目录

- [这是什么](#这是什么)
- [模型提供商](#模型提供商)
- [快速安装](#快速安装)
- [系统要求](#系统要求)
- [构建](#构建)
- [使用方法](#使用方法)
- [实验功能](#实验功能)
- [项目结构](#项目结构)
- [技术栈](#技术栈)
- [IPFS 镜像](#ipfs-镜像)
- [贡献指南](#贡献指南)
- [许可证](#许可证)

---

## 这是什么

这是 Anthropic [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI 的干净、可构建分支——终端原生 AI 编码助手。上游源代码于 2026 年 3 月 31 日通过 npm 发行版中的 source map 暴露而公开可用。

此分支在该快照之上应用了三类变更：

### 遥测已移除

上游二进制通过 OpenTelemetry/gRPC、GrowthBook 分析、Sentry 错误报告和自定义事件日志进行回传。在此构建中：

- 所有出站遥测端点已通过死代码消除或桩化处理
- GrowthBook 特性标志评估仍本地工作（运行时特性门控所需），但不回传报告
- 无崩溃报告，无使用分析，无会话指纹

### 安全提示护栏已移除

Anthropic 在每次对话中注入系统级指令，限制 Claude 的行为超出模型本身所施加的范围。这些包括硬编码的拒绝模式、注入的"网络风险"指令块，以及从 Anthropic 服务器推送的托管设置安全覆盖层。

此构建剥离了这些注入。模型自身的安全训练仍然适用——这只是移除了 CLI 在其周围包装的额外提示层限制。

### 实验功能已解锁

Claude Code 内置 88 个通过 `bun:bundle` 编译时开关门控的特性标志。大多数在公开 npm 发行版中禁用。此构建解锁所有 54 个可正常编译的标志。见下方[实验功能](#实验功能)，或参阅 [FEATURES.md](FEATURES.md) 获取完整审计。

---

## 模型提供商

free-code 开箱即支持**五种 API 提供商**。设置对应的环境变量即可切换提供商——无需修改代码。

### Anthropic（直接 API）—— 默认

直接使用 Anthropic 的第一方 API。

| 模型 | ID |
|---|---|
| Claude Opus 4.6 | `claude-opus-4-6` |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` |
| Claude Haiku 4.5 | `claude-haiku-4-5` |

### OpenAI Codex

使用 OpenAI 的 Codex 模型进行代码生成。需要 Codex 订阅。

| 模型 | ID |
|---|---|
| GPT-5.3 Codex（推荐） | `gpt-5.3-codex` |
| GPT-5.4 | `gpt-5.4` |
| GPT-5.4 Mini | `gpt-5.4-mini` |

```bash
export CLAUDE_CODE_USE_OPENAI=1
free-code
```

### AWS Bedrock

通过 Amazon Bedrock 经您的 AWS 账户路由请求。

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION="us-east-1"   # 或 AWS_DEFAULT_REGION
free-code
```

使用您标准的 AWS 凭证（环境变量、`~/.aws/config` 或 IAM 角色）。模型自动映射为 Bedrock ARN 格式（如 `us.anthropic.claude-opus-4-6-v1`）。

| 变量 | 用途 |
|---|---|
| `CLAUDE_CODE_USE_BEDROCK` | 启用 Bedrock 提供商 |
| `AWS_REGION` / `AWS_DEFAULT_REGION` | AWS 区域（默认：`us-east-1`） |
| `ANTHROPIC_BEDROCK_BASE_URL` | 自定义 Bedrock 端点 |
| `AWS_BEARER_TOKEN_BEDROCK` | Bearer token 认证 |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | 跳过认证（测试用） |

### Google Cloud Vertex AI

通过 Vertex AI 经您的 GCP 项目路由请求。

```bash
export CLAUDE_CODE_USE_VERTEX=1
free-code
```

使用 Google Cloud 应用默认凭证（`gcloud auth application-default login`）。模型自动映射为 Vertex 格式（如 `claude-opus-4-6@latest`）。

### Anthropic Foundry

使用 Anthropic Foundry 进行专用部署。

```bash
export CLAUDE_CODE_USE_FOUNDRY=1
export ANTHROPIC_FOUNDRY_API_KEY="..."
free-code
```

支持自定义部署 ID 作为模型名称。

### 提供商选择摘要

| 提供商 | 环境变量 | 认证方式 |
|---|---|---|
| Anthropic（默认） | -- | `ANTHROPIC_API_KEY` 或 OAuth |
| OpenAI Codex | `CLAUDE_CODE_USE_OPENAI=1` | OAuth via OpenAI |
| AWS Bedrock | `CLAUDE_CODE_USE_BEDROCK=1` | AWS 凭证 |
| Google Vertex AI | `CLAUDE_CODE_USE_VERTEX=1` | `gcloud` ADC |
| Anthropic Foundry | `CLAUDE_CODE_USE_FOUNDRY=1` | `ANTHROPIC_FOUNDRY_API_KEY` |

---

## 系统要求

- **运行时**：[Bun](https://bun.sh) >= 1.3.11
- **操作系统**：macOS 或 Linux（Windows 通过 WSL）
- **认证**：所选提供商的 API 密钥或 OAuth 登录

```bash
# 如没有 Bun 则安装
curl -fsSL https://bun.sh/install | bash
```

---

## 构建

```bash
git clone https://github.com/paoloanzn/free-code.git
cd free-code
bun build
./cli
```

### 构建变体

| 命令 | 输出 | 功能 | 描述 |
|---|---|---|---|
| `bun run build` | `./cli` | 仅 `VOICE_MODE` | 类生产二进制 |
| `bun run build:dev` | `./cli-dev` | 仅 `VOICE_MODE` | 开发版本戳 |
| `bun run build:dev:full` | `./cli-dev` | 所有 54 个实验标志 | 完全解锁构建 |
| `bun run compile` | `./dist/cli` | 仅 `VOICE_MODE` | 替代输出路径 |

### 自定义特性标志

启用特定标志而无需完整捆绑：

```bash
# 仅启用 ultraplan 和 ultrathink
bun run ./scripts/build.ts --feature=ULTRAPLAN --feature=ULTRATHINK

# 在开发构建之上添加标志
bun run ./scripts/build.ts --dev --feature=BRIDGE_MODE
```

---

## 使用方法

```bash
# 交互式 REPL（默认）
./cli

# 单次模式
./cli -p "这个目录里有什么文件？"

# 指定模型
./cli --model claude-opus-4-6

# 从源码运行（启动较慢）
bun run dev

# OAuth 登录
./cli /login
```

### 环境变量参考

| 变量 | 用途 |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API 密钥 |
| `ANTHROPIC_AUTH_TOKEN` | 认证 token（替代方案） |
| `ANTHROPIC_MODEL` | 覆盖默认模型 |
| `ANTHROPIC_BASE_URL` | 自定义 API 端点 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 自定义 Opus 模型 ID |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 自定义 Sonnet 模型 ID |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 自定义 Haiku 模型 ID |
| `CLAUDE_CODE_OAUTH_TOKEN` | 通过环境变量传递 OAuth token |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | API 密钥助手缓存 TTL |

---

## 实验功能

`bun run build:dev:full` 构建启用所有 54 个可用特性标志。亮点：

### 交互与 UI

| 标志 | 描述 |
|---|---|
| `ULTRAPLAN` | Claude Code 网站上的远程多 Agent 规划（Opus 级） |
| `ULTRATHINK` | 深度思考模式——输入 "ultrathink" 以提升推理强度 |
| `VOICE_MODE` | 按键说话语音输入和听写 |
| `TOKEN_BUDGET` | Token 预算追踪和使用警告 |
| `HISTORY_PICKER` | 交互式提示历史选择器 |
| `MESSAGE_ACTIONS` | UI 中的消息操作入口点 |
| `QUICK_SEARCH` | 提示快速搜索 |
| `SHOT_STATS` | 镜头分布统计 |

### Agent、记忆与规划

| 标志 | 描述 |
|---|---|
| `BUILTIN_EXPLORE_PLAN_AGENTS` | 内置探索/规划 Agent 预设 |
| `VERIFICATION_AGENT` | 任务验证的验证 Agent |
| `AGENT_TRIGGERS` | 后台自动化的本地 cron/触发器工具 |
| `AGENT_TRIGGERS_REMOTE` | 远程触发器工具路径 |
| `EXTRACT_MEMORIES` | 查询后自动记忆提取 |
| `COMPACTION_REMINDERS` | 上下文压缩周围的智能提醒 |
| `CACHED_MICROCOMPACT` | 通过查询流程的缓存微压缩状态 |
| `TEAMMEM` | 团队记忆文件和监视器钩子 |

### 工具与基础设施

| 标志 | 描述 |
|---|---|
| `BRIDGE_MODE` | IDE 远程控制桥接（VS Code、JetBrains） |
| `BASH_CLASSIFIER` | 分类器辅助的 Bash 权限决策 |
| `PROMPT_CACHE_BREAK_DETECTION` | 压缩/查询流程中的缓存中断检测 |

见 [FEATURES.md](FEATURES.md) 获取所有 88 个标志的完整审计，包括 34 个失败标志的重建说明。

---

## 项目结构

```
scripts/
  build.ts                # 带特性标志系统的构建脚本

src/
  entrypoints/cli.tsx     # CLI 入口点
  commands.ts             # 命令注册（斜杠命令）
  tools.ts                # 工具注册（Agent 工具）
  QueryEngine.ts          # LLM 查询引擎
  screens/REPL.tsx        # 主交互式 UI (Ink/React)

  commands/               # /slash 命令实现
  tools/                  # Agent 工具实现（Bash、Read、Edit 等）
  components/             # Ink/React 终端 UI 组件
  hooks/                  # React hooks
  services/               # API 客户端、MCP、OAuth、分析
    api/                  # API 客户端 + Codex fetch 适配器
    oauth/                # OAuth 流程（Anthropic + OpenAI）
  state/                  # 应用状态存储
  utils/                  # 工具函数
    model/                # 模型配置、提供商、验证
  skills/                 # 技能系统
  plugins/                # 插件系统
  bridge/                 # IDE 桥接
  voice/                  # 语音输入
  tasks/                  # 后台任务管理
```

---

## 技术栈

| | |
|---|---|
| **运行时** | [Bun](https://bun.sh) |
| **语言** | TypeScript |
| **终端 UI** | React + [Ink](https://github.com/vadimdemedes/ink) |
| **CLI 解析** | [Commander.js](https://github.com/tj/commander.js) |
| **Schema 验证** | Zod v4 |
| **代码搜索** | ripgrep（捆绑） |
| **协议** | MCP, LSP |
| **API** | Anthropic Messages, OpenAI Codex, AWS Bedrock, Google Vertex AI |

---

## IPFS 镜像

此仓库的完整副本通过 Filecoin 在 IPFS 上永久固定：

| | |
|---|---|
| **CID** | `bafybeiegvef3dt24n2znnnmzcud2vxat7y7rl5ikz7y7yoglxappim54bm` |
| **网关** | https://w3s.link/ipfs/bafybeiegvef3dt24n2znnnmzcud2vxat7y7rl5ikz7y7yoglxappim54bm |

如果此仓库被下架，代码仍会存活。

---

## 贡献指南

欢迎贡献。如果您正在恢复 34 个失败特性标志之一，请先查看 [FEATURES.md](FEATURES.md) 中的重建说明——许多接近编译，只需一个小包装器或缺失资产。

1. Fork 仓库
2. 创建特性分支（`git checkout -b feat/my-feature`）
3. 提交您的更改（`git commit -m 'feat: add something'`）
4. 推送到分支（`git push origin feat/my-feature`）
5. 开启 Pull Request

---

## 许可证

原始 Claude Code 源代码是 Anthropic 的财产。此分支存在是因为源代码通过其 npm 发行版公开暴露。请自行决定使用。