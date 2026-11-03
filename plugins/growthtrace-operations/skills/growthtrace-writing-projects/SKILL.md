---
name: growthtrace-writing-projects
description: Use when users ask to inspect or create GrowthTrace writing projects, recommend or change article membership, or inspect and maintain project concepts and relationships, including “整理到写作项目” and “看看项目关系图”. Does not apply to software repositories or project resource management.
---

# GrowthTrace 写作项目

先读取同一插件内的 [网站运营规则](../growthtrace-operations/SKILL.md)，遵守其中项目草案确认、推荐和归属规则。读取需要 `project:read`，创建和项目图写入需要 `project:write`，调整文章归属还需要 `article:write`。请求涉及关系图、概念或关系时，还必须读取 [图谱工作流](../growthtrace-operations/references/graph-workflow.md)。

1. 用 `list_writing_projects` 查看已有项目，必要时用 `get_writing_project` 读取规则，优先建议复用活跃项目。
2. 创建项目时，读取[项目创建输入契约](../growthtrace-operations/references/mcp-input-contracts.md)，展示包含名称、类型、描述、动态维度档案和写作规则的完整草案；用户确认后才调用 `create_writing_project`。项目类型放在顶层 `type`，档案使用 `profile.dimensions`；每个维度有唯一 `id`、`name`、Markdown `content` 和 `enabled`。不复制历史项目的旧 `profile.type/worldview/goals` 结构作为创建参数。成功后读取项目核对。
3. 推荐归属时，先 `get_article` 读取站内文章，再调用 `match_article_writing_project`。仅把 `RECOMMEND` 展示为高可信推荐；`NO_CONFIDENT_MATCH` 不触发创建或归属写入。
4. 用户确认推荐，或明确指定、更换、取消项目后，才调用 `assign_article_writing_project`；沿用最新文章版本与工具要求的推荐校验参数。
5. 权限不足、归档项目或版本冲突时停止相关写入，说明具体原因。没有项目资料管理、项目编辑或归档能力，不使用其他接口绕过。
6. 查看关系时调用 `get_writing_project_graph`；使用节点的 `entity` 作为关系端点，不使用画布 `id` 猜测端点。
7. 概念使用 `create_project_graph_concept`、`update_project_graph_concept`、`archive_project_graph_concept`；人工关系使用 `create_project_graph_relation`、`update_project_graph_relation`、`archive_project_graph_relation`。先展示最小差异并取得精确确认，每项带 `confirmation=CONFIRM` 和幂等键，更新/归档使用最新 `expectedUpdatedAt`，最后重读核对。
8. SYSTEM 归属边继续由项目/文章事实派生，AI 关系留在站内建议审批流程；不通过项目图工具修改、归档或伪造它们，也不使用整图覆盖绕过边界。

例如“看看这篇文章适合哪个项目”只读取并建议，不自动改变归属。
