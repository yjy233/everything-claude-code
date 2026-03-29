# Commands 实现原理详解

## 核心结论

在这个仓库里，Claude Code 的 command 不是独立可执行程序，也不是传统 CLI 子命令。它本质上是放在 `commands/` 目录下的 Markdown 工作流定义文件。

可以先把它理解成下面这个模型：

```text
用户输入 /plan "添加支付功能"
        │
        ▼
Claude Code 匹配 commands/plan.md
        │
        ▼
读取命令文件内容作为执行提示
        │
        ▼
根据命令定义调用 agent / skill / Bash / Read / Write
        │
        ▼
输出结果
```

也就是说，`commands/*.md` 更像“命令模板”或“命令剧本”，而不是二进制程序。

---

## 一、命令定义放在哪里

这个仓库里和 Claude Code command 相关的定义主要有两层：

### 1. 根目录 `commands/`

这是主要的命令目录，存放绝大多数 slash commands。

典型文件包括：

- `commands/plan.md`
- `commands/tdd.md`
- `commands/code-review.md`
- `commands/e2e.md`
- `commands/sessions.md`

这些文件通常会随插件或手动安装一起提供给 Claude Code。

### 2. 项目内 `.claude/commands/`

这是仓库自己额外维护的一小组本地 workflow scaffold，例如：

- `.claude/commands/add-language-rules.md`
- `.claude/commands/feature-development.md`
- `.claude/commands/database-migration.md`

它们更偏“项目内流程脚手架”，不是这套命令系统的主目录。

---

## 二、命令文件长什么样

一个 command 文件通常由两部分构成：

1. YAML Frontmatter
2. Markdown 正文

例如一个最小结构通常是：

```md
---
name: plan
description: Create an implementation plan
allowed_tools: ["Read", "Grep", "Glob"]
---

# /plan

## Usage

/plan "feature description"

## Process

1. Analyze the request
2. Read the relevant files
3. Create a plan
4. Return the result
```

各字段的作用：

- `name`
  决定命令名称
- `description`
  用于描述命令用途
- `allowed_tools`
  限制命令执行时允许使用哪些工具

正文部分则定义：

- 这个命令什么时候使用
- 具体应该怎么执行
- 要不要委托给某个 agent
- 是否要跑脚本
- 参数如何传递
- 输出格式是什么

---

## 三、Claude Code 如何“执行”命令

### 第一步：匹配命令

当用户输入：

```text
/e2e Test the market search and view flow
```

Claude Code 会匹配到 `commands/e2e.md` 这个文件。

### 第二步：读取命令文件

它不会把这个文件“运行”成一个进程，而是读取里面的内容，把它当作当前任务的执行说明。

这就是 command 和传统 shell 命令最大的区别：

- 传统命令：直接执行程序
- Claude Code command：先读取 Markdown 定义，再按定义调用工具

### 第三步：按命令内容进行推理和工具调用

如果命令是偏提示词型的，Claude 会按文档要求自己分析并执行。

如果命令里明确包含脚本片段，Claude 会调用 Bash 或 Node 去执行这些脚本。

### 第四步：输出结果

最终返回给用户的，是 Claude 根据命令定义和工具执行结果整理后的响应。

---

## 四、这个仓库里的两种主要实现模式

这个仓库里的 commands，大体可以分成两类。

### 1. Prompt-driven command

这种命令主要是“工作流提示词”，强调让 Claude 按某个方法去完成任务。

代表例子：

- `commands/plan.md`
- `commands/e2e.md`
- `commands/tdd.md`

以 `commands/e2e.md` 为例，它明确写着：

- 这个命令会调用 `e2e-runner` agent
- 会分析用户流程
- 会生成 Playwright 测试
- 会运行测试并输出报告

这类命令的特点是：

- 重点在流程描述
- 输出结果较灵活
- 更像“高级 prompt”

### 2. Script-backed command

这种命令除了说明流程，还在文件里直接嵌入了实际脚本。

代表例子：

- `commands/sessions.md`

这个文件里直接放了多段 `node -e "..."` 脚本，用来读取会话数据、解析 alias、打印格式化结果。

这类命令的特点是：

- 行为更稳定
- 结果更结构化
- 更接近“真正的工具型命令”

可以把它理解成：

```text
Markdown 负责描述命令
Node/Bash 负责真正干活
Claude 负责把两者串起来
```

---

## 五、command 和 agent 的关系

很多命令并不是自己完成所有事情，而是把任务交给 `agents/` 里的专用角色。

例如：

- `/plan` -> `planner`
- `/tdd` -> `tdd-guide`
- `/code-review` -> `code-reviewer`
- `/e2e` -> `e2e-runner`

所以执行链经常是：

```text
用户输入 /plan
    │
    ▼
匹配 commands/plan.md
    │
    ▼
命令定义要求调用 planner
    │
    ▼
读取 agents/planner.md
    │
    ▼
Claude 以 planner 角色输出实现计划
```

因此 command 更像入口层，agent 更像执行角色层。

---

## 六、command 和 skill 的关系

skill 通常不是直接由用户输入触发，而是作为命令或 agent 的方法论支持。

关系可以理解成：

- `command` 决定“现在做什么”
- `agent` 决定“谁来做”
- `skill` 决定“按什么方法做”

例如：

- `/tdd` 可能调用 `tdd-guide`
- `tdd-guide` 执行时会遵循 `tdd-workflow`

所以从架构层面看：

```text
commands = 入口
agents   = 角色
skills   = 知识和工作流
```

---

## 七、参数是怎么传进去的

一些命令会直接在正文里使用 `$ARGUMENTS` 占位符。

例如：

```bash
node -e "..." "$ARGUMENTS"
```

这表示用户在 `/command` 后面输入的参数，会被传给这段脚本。

典型场景：

- `/sessions load abc123`
- `/sessions alias 2026-02-01 today-work`

这也是 script-backed command 能处理复杂子命令和参数的关键机制。

---

## 八、为什么命令文件是 Markdown

使用 Markdown 有几个明显好处：

### 1. 可读性高

普通人直接打开就能看懂命令要做什么。

### 2. 同时适合人和模型

- 对人来说，它是文档
- 对 Claude 来说，它是执行说明

### 3. 容易混合“说明”和“脚本”

一个文件里可以同时包含：

- 用法说明
- 示例
- 执行规则
- Bash 片段
- Node 脚本

### 4. 易于版本管理

命令改动是纯文本，适合 diff、review、PR 讨论和校验。

---

## 九、这个仓库如何校验 commands

仓库里有专门的校验脚本：

- `scripts/ci/validate-commands.js`

它会检查：

- `commands/` 是否存在
- 每个命令文件是否非空
- 是否引用了不存在的 command
- 是否引用了不存在的 agent
- 是否引用了不存在的 skill 目录
- workflow 文本里提到的 agent 名称是否真实存在

这说明在 ECC 里，command 虽然是 Markdown，但不是随便写的文档，而是受 CI 约束的“结构化执行定义”。

---

## 十、完整执行链

把整个实现流程串起来，大概是这样：

```text
用户输入 /command args
        │
        ▼
Claude Code 在 commands/ 中找到对应 .md
        │
        ▼
读取 frontmatter 和正文
        │
        ▼
理解这个命令需要：
  - 直接回答
  - 调用某个 agent
  - 使用某个 skill
  - 执行 Bash / Node 脚本
        │
        ▼
运行对应工具
        │
        ▼
返回格式化结果
```

如果是更具体的仓库内链路，可以写成：

```text
/plan "添加支付功能"
    -> commands/plan.md
    -> agents/planner.md
    -> 读取相关代码
    -> 输出 implementation plan

/sessions load abc123
    -> commands/sessions.md
    -> node -e 脚本
    -> scripts/lib/session-manager
    -> 输出会话信息
```

---

## 十一、和传统 CLI 命令的差异

| 维度 | 传统 CLI 命令 | Claude Code command |
|------|---------------|--------------------|
| 本质 | 可执行程序 | Markdown 定义 |
| 执行方式 | 直接运行二进制/脚本 | 先读定义，再由 Claude 调用工具 |
| 输出 | 程序固定输出 | 模型结合规则和工具输出 |
| 扩展方式 | 新增脚本/子命令 | 新增 `commands/*.md` |
| 可读性 | 偏工程实现 | 同时是文档和执行规范 |

---

## 十二、结论

如果只记一个结论，可以记这个：

> ECC 里的 command 不是 shell 命令实现，而是用 Markdown 编写的命令工作流定义；Claude Code 读取这些定义后，再去调用 agent、skill 和脚本来真正完成任务。

因此：

- `commands/` 是入口层
- `agents/` 是角色层
- `skills/` 是知识层
- `scripts/` 是具体实现层

四层配合起来，才构成这个仓库里完整的 command 实现方式。
