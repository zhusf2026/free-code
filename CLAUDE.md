# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

## 常用命令

```bash
# 安装依赖
bun install

# 标准构建 (./cli)
bun run build

# 开发构建 (./cli-dev)
bun run build:dev

# 包含所有实验功能的开发构建 (./cli-dev)
bun run build:dev:full

# 编译构建 (./dist/cli)
bun run compile

# 不编译直接从源码运行
bun run dev
```

使用 `./cli` 或 `./cli-dev` 运行构建的二进制文件。在环境中设置 `ANTHROPIC_API_KEY` 或通过 `./cli /login` 使用 OAuth。

## 高层架构

- **入口点/UI 循环**：src/entrypoints/cli.tsx 启动 CLI，主交互 UI 位于 src/screens/REPL.tsx (Ink/React)。
- **命令/工具注册**：src/commands.ts 注册斜杠命令；src/tools.ts 注册工具实现。实现位于 src/commands/ 和 src/tools/。
- **LLM 查询管道**：src/QueryEngine.ts 协调消息流、工具使用和模型调用。
- **核心子系统**：
  - src/services/: API 客户端、OAuth/MCP 集成、分析桩
  - src/state/: 应用状态存储
  - src/hooks/: UI/流程使用的 React hooks
  - src/components/: 终端 UI 组件 (Ink)
  - src/skills/: 技能系统
  - src/plugins/: 插件系统
  - src/bridge/: IDE 桥接
  - src/voice/: 语音输入
  - src/tasks/: 后台任务管理

## 构建系统

- scripts/build.ts 是构建脚本和特性标志打包器。特性标志通过构建参数设置（如 `--feature=ULTRAPLAN`）或预设如 `--feature-set=dev-full`（详见 README）。