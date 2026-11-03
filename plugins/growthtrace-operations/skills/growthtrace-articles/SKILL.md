---
name: growthtrace-articles
description: Use when users ask to find, preview, save, edit, publish, or unpublish GrowthTrace articles, including “内嵌预览普通文章”, “把对话保存为文章草稿” or “修改网站上的文章”. Does not apply to standalone writing without GrowthTrace website intent.
---

# GrowthTrace 文章管理

先读取同一插件内的 [网站运营规则](../growthtrace-operations/SKILL.md)，沿用其授权、幂等和版本冲突规则；仅使用已连接的 GrowthTrace MCP。

- 保存对话：整理标题、摘要、关键词和 Markdown 正文，再调用 `create_article`，默认 `mode: "draft"`。普通润色请求不等于保存授权。
- 调用站内 AI：新文章用 `generate_article_draft(prompt, idempotencyKey)`，返回未公开草稿 ID；已有文章正文、标题、摘要或关键词修改用 `generate_article_fields(articleId, prompt, idempotencyKey)`，返回站内审阅候选。
- 查找文章：使用 `search_articles` 或 `list_articles` 定位，再用 `get_article` 读取正文；同名文章无法区分时先让用户选择。
- 内嵌预览普通文章：取得已保存的 `articleId` 后调用 `create_article_preview`。支持 MCP Apps 的客户端直接显示正文；其他客户端使用返回的 `previewUrl`。预览支持未公开草稿，仅需 `site:read`，不生成模板、展示稿候选或 AI 内容。用户要求创建文章并预览时，先完成草稿保存，再使用返回的文章 ID 调用此工具。
- 预览地址过期：调用 `refresh_article_preview_url(previewId)`，新地址使旧令牌失效；刷新仍显示创建时的冻结正文。用户要求查看最新修改时重新调用 `create_article_preview`。
- 展示稿撰写、预览或应用：读取[文章展示撰写技能](../growthtrace-article-display/SKILL.md)，沿用其默认审阅、明确授权后直接应用并展示的流程；展示稿请求不使用普通文章的 `update_article`。
- 修改文章：先读取详情，使用最新 `updatedAt` 作为 `expectedUpdatedAt`，只提交用户要求的变化。版本冲突时展示差异，不覆盖较新内容。
- 写入字段遵循[输入契约](../growthtrace-operations/references/mcp-input-contracts.md)：创建和更新使用相同关键词长度限制；公开链接只能使用合法单段 slug，发布时不可缺失，已发布文章不可清空链接。
- 发布或取消公开：只有用户明确要求对应操作才调用 `set_article_publication`；“保存”不能推断为公开。
- 文章出题、编辑或启停测验：读取[文章测验技能](../growthtrace-article-quiz/SKILL.md)，使用独立测验工具；文章类型不变，不在正文写入参考答案，不给普通文章接口加入测验字段。
- 完成后返回工具确认的文章标题、ID、链接（如有）和发布状态。失败时不宣称已保存，也不自动重建另一篇文章。

例如“把刚才的讨论保存到我的 GrowthTrace”应创建草稿；“帮我润色这段文字”只处理文字，不调用写入工具。文章项目归属由写作项目技能处理。
