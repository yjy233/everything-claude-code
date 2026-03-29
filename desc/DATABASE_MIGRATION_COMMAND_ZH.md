# /database-migration 中文版

> 这是 [database-migration.md](/Users/bill/code/everything-claude-code/.claude/commands/database-migration.md) 的中文说明版，放在 `desc/` 下用于阅读和理解。  
> 它是文档，不是会被 Claude Code 直接加载的 command 文件。

## 命令名称

`/database-migration`

## 适用场景

当你在 `everything-claude-code` 中处理数据库结构变更、迁移文件新增或类型同步这类任务时，可以参考这个工作流。

## 目标

处理数据库 schema 变更，并配套维护 migration 文件。

## 常见涉及文件

- `**/schema.*`
- `migrations/*`

## 建议执行顺序

1. 先理解当前数据库结构、已有迁移以及当前失败或变更需求。
2. 在满足迁移目标的前提下，尽量做最小且完整的改动。
3. 对改动过的 schema、migration 和相关类型生成结果运行最相关的验证。
4. 总结本次修改内容，以及仍然需要人工复核的部分。

## 常见改动信号

- 新增 migration 文件
- 更新 schema 定义
- 生成或更新类型定义

## 说明

- 这个命令是一个 workflow scaffold，也就是工作流脚手架，不是硬编码脚本。
- 它提供的是数据库迁移工作的基本结构，但不会自动替你处理所有数据库工具链细节。
- 如果仓库里的迁移流程、schema 管理方式或类型生成方式发生明显变化，这条命令本身也应该同步更新。

## 一句话理解

`/database-migration` 的作用不是“自动完成数据库迁移”，而是告诉 Claude：

> 当我要在 ECC 项目里做数据库 schema 变更时，应该按什么顺序检查、修改、验证和总结。
