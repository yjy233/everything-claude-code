# Agents 实现原理详解

## 核心概念：Agents 不是程序，是"角色模式"

**最重要的事实**: 这 28 个 agents **不是独立的程序或进程**。它们是一套**精心设计的系统提示 (System Prompts)**，告诉 Claude 在特定情况下应该扮演什么角色、遵循什么流程。

```
传统理解 (错误)                实际机制 (正确)
─────────────────────          ─────────────────────
agents/planner.md ──► [运行]    agents/planner.md ──► [读取内容]
                         │                           │
                         ▼                           ▼
                    [独立进程]                    [注入系统提示]
                                                        │
                        ▲                             ▼
                        └──────────────────────── [Claude 扮演规划师角色]
```

---

## 三层架构

### 第一层：指令定义 (Markdown 文件)

```yaml
---
# YAML Frontmatter - 元数据
name: planner                           # Agent 名称
description: 什么时候使用这个 Agent      # 触发条件
tools: ["Read", "Grep", "Glob"]        # 可用工具
model: opus                             # 推荐模型
---

# Markdown 内容 - 系统提示主体
## Your Role
你是规划专家，专注于创建可执行的实施计划...

## Planning Process  
1. Requirements Analysis
2. Architecture Review
3. Step Breakdown
...

## Plan Format
必须按以下格式输出...
```

**作用**: 这是给 Claude 的"剧本"，告诉它：
- 你是谁（角色）
- 你要做什么（职责）
- 你怎么做（流程）
- 输出什么（格式）

---

### 第二层：委托机制 (AGENTS.md)

```markdown
## Agent Orchestration

Use agents proactively without user prompt:
- Complex feature requests → **planner**
- Code just written/modified → **code-reviewer**  
- Bug fix or new feature → **tdd-guide**
...
```

**Claude 的基本指令** (在 CLAUDE.md 和 AGENTS.md 中):
```
你是 Claude Code，一个 AI 编程助手。

当用户请求时，判断任务类型：
1. 如果是复杂功能 → 加载 agents/planner.md，按它的指示执行
2. 如果是代码审查 → 加载 agents/code-reviewer.md，按它的指示执行
3. ...
```

**关键机制**: 
- `CLAUDE.md` 是每个项目的"根配置"
- `AGENTS.md` 定义了什么时候该"切换角色"
- Claude 会自己读取对应的 `.md` 文件并遵循其中的指示

---

### 第三层：实际执行 (对话注入)

当触发条件满足时，实际的执行流程：

```
用户输入: "帮我添加用户认证功能"
        │
        ▼
Claude (阅读 CLAUDE.md + AGENTS.md)
        │
        ▼
判断: 这是 "Complex feature request"
        │
        ▼
加载: agents/planner.md (读取文件内容)
        │
        ▼
系统提示变成:
"""
你是 expert planning specialist...

## Your Role
- Analyze requirements and create detailed implementation plans
...

## Plan Format
...
"""
        │
        ▼
Claude 以"规划师"角色回复用户
```

---

## 技术实现细节

### 1. 文件读取机制

Claude Code 有 `Read` 工具，可以读取项目中的任何文件：

```javascript
// 伪代码 - Claude 内部工作流程
function handleUserInput(userInput) {
  // 1. 读取项目配置
  const claudeMd = readFile("CLAUDE.md");
  const agentsMd = readFile("AGENTS.md");
  
  // 2. 分析用户输入
  const taskType = analyzeTask(userInput); // "complex_feature"
  
  // 3. 根据 AGENTS.md 确定 Agent
  const agentName = routeToAgent(taskType); // "planner"
  
  // 4. 加载 Agent 定义
  const agentDef = readFile(`agents/${agentName}.md`);
  
  // 5. 组合系统提示
  const systemPrompt = `
    ${claudeMd}
    ${agentsMd}
    
    # Current Agent
    ${agentDef}
    
    # User Request
    ${userInput}
  `;
  
  // 6. 生成回复
  return claude.generate(systemPrompt);
}
```

### 2. 命令触发 (/plan, /tdd 等)

`commands/` 文件夹下的 `.md` 文件定义了斜杠命令：

```markdown
---
name: plan
description: Implementation planning command
tools: ["Read", "Grep", "Glob"]
---

## Usage
/plan "feature description"

## Execution
When this command is invoked:
1. Delegate to `planner` agent
2. Pass the feature description
3. Output the implementation plan
```

**实际执行**:
```
用户输入: "/plan 添加支付功能"
        │
        ▼
匹配: commands/plan.md
        │
        ▼
读取命令定义 → "Delegate to planner agent"
        │
        ▼
加载 agents/planner.md
        │
        ▼
执行规划流程
```

### 3. 自动触发 (Hooks)

`hooks/` 文件夹定义了自动触发机制：

```json
// hooks/hooks.json
{
  "hooks": [
    {
      "type": "PostToolUse",
      "tool": "WriteFile",
      "command": "echo 'File written, consider running code-reviewer'"
    }
  ]
}
```

**实际效果**:
- 用户写完代码后 → Hook 触发 → 提示进行代码审查 → Claude 加载 code-reviewer.md

---

## 为什么用 Markdown？

### 1. 人类可读 + AI 可解析

```markdown
# 人类看到                      # Claude 看到
规划指南                       系统提示指令
├── 角色定义                    ├── identity="planner"
├── 工作流程                    ├── workflow=[step1, step2...]  
├── 输出格式                    ├── output_format="markdown"
└── 示例                        └── examples=[...]
```

### 2. 版本控制友好
- 纯文本，Git 友好
- 可追溯修改历史
- 支持 PR 审查

### 3. 易于扩展
添加新 Agent = 创建新 Markdown 文件：

```bash
cp agents/planner.md agents/my-custom-agent.md
# 编辑内容
# 立即生效，无需编译
```

### 4. 跨平台兼容
- 不需要运行时
- 任何编辑器都可修改
- 与操作系统无关

---

## 完整数据流

```
┌─────────────────────────────────────────────────────────────────┐
│                         用户层                                   │
│  用户输入: "/plan 添加支付功能"                                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      命令解析层                                  │
│  读取: commands/plan.md                                          │
│  解析: "Delegate to planner agent"                               │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      配置加载层                                  │
│  读取: CLAUDE.md (项目配置)                                      │
│  读取: AGENTS.md (委托规则)                                      │
│  读取: agents/planner.md (Agent 定义)                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      提示组装层                                  │
│  系统提示 =                                                      │
│    [CLAUDE.md 基础指令]                                          │
│    + [AGENTS.md 委托规则]                                        │
│    + [planner.md 角色定义]                                       │
│    + [用户输入]                                                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      LLM 执行层                                  │
│  Claude API (Anthropic)                                          │
│  - model: opus (由 frontmatter 指定)                             │
│  - system prompt: [组装的提示]                                   │
│  - user message: [当前上下文]                                    │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      输出生成层                                  │
│  Claude 以 "planner" 角色回复：                                  │
│  "# Implementation Plan: Payment System..."                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 不同平台的实现差异

### Claude Code (官方)
- 直接读取项目中的 `CLAUDE.md`
- 原生支持 `commands/` 斜杠命令
- 支持 `hooks/` 自动触发

### Cursor
- 使用 `.cursor/` 目录下的配置
- 通过 `.cursorrules` 文件加载规则
- Agent 定义需要适配 Cursor 格式

### OpenCode
- 使用 `.opencode/` 目录
- 插件系统支持自定义指令
- Agent 以 "instructions" 形式存在

### Codex CLI
- 使用 `.codex/` 目录
- 基于 `codex.md` 配置
- 通过 prompts 实现类似功能

---

## 自定义 Agent 示例

### 步骤 1: 创建 Agent 文件

```markdown
# agents/my-api-designer.md

---
name: my-api-designer
description: REST API 设计专家。Use when designing new API endpoints.
tools: ["Read", "Grep"]
model: sonnet
---

## Your Role
你是 API 设计专家，专注于设计符合 RESTful 规范的 API。

## Design Process
1. 分析业务需求
2. 设计资源 URI
3. 定义 HTTP 方法和状态码
4. 设计请求/响应格式
5. 考虑错误处理

## Output Format
```markdown
# API Design: [Resource Name]

## Endpoints
| Method | Path | Description |
|--------|------|-------------|
| GET    | /api/users | 列出用户 |
...

## Request/Response Examples
...
```

## Best Practices
- 使用名词复数形式 (/users, 不是 /user)
- 正确使用 HTTP 状态码
- 提供清晰的错误消息
```

### 步骤 2: 注册到 AGENTS.md

```markdown
## Agent Orchestration
...
- API design tasks → **my-api-designer**
```

### 步骤 3: 创建命令 (可选)

```markdown
# commands/api-design.md

---
name: api-design
description: Design REST API endpoints
---

## Usage
/api-design "描述要设计的 API"

## Execution
Delegate to `my-api-designer` agent.
```

### 完成！

现在可以使用了：
```bash
/api-design "设计用户管理 API，包括 CRUD 操作"
```

---

## 总结

| 问题 | 答案 |
|------|------|
| Agents 是程序吗？ | **不是**，它们是 Markdown 格式的系统提示模板 |
| 怎么执行的？ | Claude **读取** .md 文件内容，作为系统提示的一部分 |
| 为什么能工作？ | Claude 遵循指令，切换不同的"角色模式" |
| 需要编译吗？ | **不需要**，纯文本，即时生效 |
| 能脱离 Claude Code 吗？ | **不能**，需要 Claude/AI 来读取和执行 |

**本质**: ECC 的 agents 是一套**高级的提示工程 (Prompt Engineering)**，通过结构化的 Markdown 文件，让 Claude 在不同场景下表现出专业领域专家的行为模式。
