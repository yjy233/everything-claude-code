# 框架介绍

`Everything Claude Code (ECC)` 不是单一功能插件，而是一套围绕 AI 编码工作流构建的完整框架。它把命令入口、子代理、技能、规则、钩子和脚本组合在一起，用来提升 Claude Code、Codex、Cursor、OpenCode 等 agent harness 的开发效率和稳定性。

## 重要模块

### `agents/`

子代理定义目录，用来描述不同角色的职责、可用工具和工作方式。

- `planner` 负责规划复杂功能
- `code-reviewer` 负责代码审查
- `security-reviewer` 负责安全检查
- `e2e-runner` 负责 E2E 测试

### `commands/`

斜杠命令目录，是用户最直接的入口。每个 `*.md` 文件都对应一个 Claude Code 命令工作流。

- `/plan` 用于实现规划
- `/tdd` 用于测试驱动开发
- `/code-review` 用于审查改动
- `/e2e` 用于生成和运行端到端测试

### `skills/`

技能目录，存放可复用的方法论、领域知识和执行流程。技能通常会被命令或子代理调用。

- `api-design` 用于 REST API 设计
- `tdd-workflow` 用于测试驱动开发
- `security-review` 用于安全审查
- `frontend-patterns` 和 `backend-patterns` 用于架构与实现模式

### `rules/`

规则目录，定义始终遵循的编码规范、安全要求、测试标准和语言特定约束。

- `common/` 提供通用规则
- `typescript/`、`python/`、`golang/` 等目录提供语言专用规则

### `hooks/`

钩子目录，用于在 Claude Code 生命周期和工具调用前后执行自动化逻辑。

- `SessionStart` 时加载上下文
- `PreToolUse` 时做检查
- `PostToolUse` 时做结果处理
- `Stop` 时保存状态或输出摘要

### `scripts/`

跨平台 Node.js 脚本目录，承载实际的安装、检测、修复、状态管理和自动化逻辑。

- `install-apply.js` 和 `install-plan.js` 负责安装流程
- `doctor.js` 用于诊断
- `status.js` 用于状态查看

### `.claude-plugin/`

Claude Code 插件清单目录，负责描述插件元数据和 marketplace 信息。

- `plugin.json` 定义插件基础信息
- `marketplace.json` 用于 `/plugin marketplace add`

### `mcp-configs/`

MCP 服务器配置目录，用于接入 GitHub、Supabase、Vercel、Railway 等外部能力。

### `contexts/`

上下文目录，用于向不同工作模式注入额外系统提示，例如开发、审查、研究等模式。

### `assets/`

静态资源目录，主要用于 README、文档和展示材料中的图片与素材。

## 模块之间的关系

整体流程通常是：

`用户输入命令` -> `commands/` 作为入口 -> `agents/` 执行专门任务 -> `skills/` 提供方法论 -> `rules/` 约束输出 -> `hooks/` 在执行前后自动处理 -> `scripts/` 承载底层实现。

## 一句话总结

ECC 的核心不是某一个单独的包，而是多个关键模块协同工作：

- `commands/` 负责入口
- `agents/` 负责分工
- `skills/` 负责知识和流程
- `rules/` 负责约束
- `hooks/` 负责自动化
- `scripts/` 负责落地执行


