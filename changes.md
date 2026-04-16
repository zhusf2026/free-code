# Codex API 支持：功能对齐与界面重构

## 概述
本次拉取请求为 OpenAI Codex 后端 (`chatgpt.com/backend-api/codex/responses`) 引入了完整的功能对齐和明确的界面支持。代码库现已完全实现后端无关，能够根据当前认证状态在 Anthropic Claude 和 OpenAI Codex 架构之间平滑切换，同时不丢失推理动画、Token 计费或多模态视觉输入等功能。

## 主要变更

### 1. Codex API 网关适配器 (`codex-fetch-adapter.ts`)
- **原生视觉转换**：Anthropic `base64` 图片架构现已精确映射到 Codex 预期的 `input_image` 载荷格式。
- **严格载荷映射**：重构了内部映射逻辑，将 `msg.content` 项精确转换为 `input_text`，避开了 OpenAI 严格的 `v1/responses` 验证规则（`Invalid value: 'text'`）。
- **工具逻辑修复**：正确将 `tool_result` 项路由到顶层 `function_call_output` 对象，确保本地 CLI 工具执行（文件读取、Bash 循环）能够干净地反馈到 Codex 逻辑中，而不会抛出 "No tool output found" 错误。
- **缓存剥离**：在传输前干净地剥离 Anthropic 专属的 `cache_control` 注解（来自工具绑定和提示），避免 Codex API 拒绝格式错误的 JSON。

### 2. 深度界面与路由集成
- **模型清理 (`model.ts`)**：更新了 `getPublicModelDisplayName` 和 `getClaudeAiUserDefaultModelDescription` 以识别 Codex GPT 字符串。像 `gpt-5.1-codex-max` 这样的模型现在在 CLI 可视化输出中会美观地映射为 `Codex 5.1 Max`，而不是传递原始代理 ID。
- **默认路由重定向**：让 `getDefaultMainLoopModelSetting` 能够感知 `isCodexSubscriber()`，自动默认使用 `gpt-5.2-codex` 而非 `sonnet46`。
- **计费可视化 (`logoV2Utils.ts`)**：重构了 `formatModelAndBilling` 逻辑，在认证时于终端头部自豪地渲染 `Codex API Billing`。

### 3. 推理与指标支持
- **思考动画**：`codex-fetch-adapter` 现有意拦截 `codex-max` 模型发出的专有 `response.reasoning.delta` SSE 帧。将其包装为 Anthropic `<thinking>` 事件，确保标准 CLI "Thinking..." 旋转器继续为 OpenAI 推理完美运行。
- **Token 精度**：绑定逻辑追踪 `response.completed` 完成事件，获取 `usage.input_tokens` 和 `output_tokens`。这些被原生注入到最终的 `message_stop` Token 处理器中，意味着 Codex 查询能够正确触发终端的 Token/价格追踪摘要逻辑。

### 4. Git 清理
- 配置 `.gitignore` 以安全持久地从提交暂存区排除 `openclaw/` 网关目录。