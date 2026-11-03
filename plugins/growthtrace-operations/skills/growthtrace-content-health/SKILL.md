---
name: growthtrace-content-health
description: Use when users ask to inspect GrowthTrace content inventory, duplicate titles, metadata completeness, or content organization priorities, including “检查网站内容健康”. Does not apply to infrastructure monitoring or analytics without a configured data source.
---

# GrowthTrace 内容健康

先读取同一插件内的 [网站运营规则](../growthtrace-operations/SKILL.md)。本技能执行只读诊断，不能因为发现问题就自动修改文章。

1. 调用 `get_site_overview` 和 `get_content_health` 获取站内事实。
2. 对值得处理的问题，用 `list_articles`、`search_articles`、`get_article` 补充依据；按需读取，避免无目的拉取全部正文。
3. 区分重复标题、缺失元数据等已证实问题，以及基于内容的整理建议。没有配置 analytics 时，明确没有阅读量、来源和访客数据，不推断流量表现。
4. 给出简短的优先事项，附相关文章标题、ID 或工具返回的链接，并说明每项建议的理由。
5. 报告留在当前对话。只有用户明确要求保存或修改后，才交给文章管理流程执行相应写入。

例如“检查我的博客健康，给三个建议”应返回有证据的建议，不新建诊断文章或批量改标题。
