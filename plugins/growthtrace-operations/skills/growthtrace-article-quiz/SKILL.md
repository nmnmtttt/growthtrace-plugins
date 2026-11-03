---
name: growthtrace-article-quiz
description: Generate, edit, or activate a quiz linked to a GrowthTrace article, including “给这篇文章出题”, “根据正文生成测验”, or “修改文章练习题”. Does not apply to standalone exam writing without GrowthTrace intent.
---

# GrowthTrace 文章测验

通过已连接的 GrowthTrace MCP 为文章管理独立关联测验。沿用同一插件的[网站运营规则](../growthtrace-operations/SKILL.md)。文章仍是普通文章，一篇文章最多关联一份包含多道题目的测验；不要新增课程文章类型或改写正文来承载答案。

## 读取与生成

- 用 `search_articles` 或 `list_articles` 定位文章，再用 `get_article` 读取实际正文，并用 `get_article_quiz` 核对已有测验及最新版本。文章不明确且无法从工具结果区分时，请用户选择。
- 先调用 `get_article_quiz_capabilities`，按返回的 `definitionSchema` 构建测验，并以实时契约确定题型、图像/绘图格式、分值和输入限制，不凭记忆编造枚举或绘图 JSON。语义校验仍以 `validate_article_quiz` 为准。
- 用户希望站内 AI 出题时，调用 `generate_article_quiz`，提供题量、题型、难度和重点等已有要求。工具依据站内实际阅读正文生成 `QUIZ_DRAFT` 产物候选，返回 `reviewUrl`；将此链接交给用户审阅应用。生成候选不代表已保存为测验定义或已启用，不能绕过产物审阅直接应用该候选。
- 由当前助手直接编写或修改题目时，根据文章已有知识生成题干、稳定题目/选项 ID、参考答案、解析及主观题评分要点。不得把文章没有教授的知识作为默认必答内容；图片是题干材料，按实时能力与选择、判断、简答或绘图作答组合。

## 保存与启用

- 直接编写的题目先调用 `validate_article_quiz`。按结构化问题修正，校验成功后才调用 `save_article_quiz` 保存测验草稿，并返回工具确认的审阅地址。校验本身不写入。
- 更新或启停前，使用 `get_article_quiz` 返回的最新 `revision` 作为 `expectedRevision` 乐观锁；初次创建使用 `expectedRevision: 0`。题目或选项排序保留原 ID。
- 写入、生成和启停使用新的幂等键；重试同一请求沿用原键。版本冲突后重读并说明差异，不能覆盖较新版本。
- 默认只保存草稿。只有用户明确要求“启用测验”“公开练习”等对应动作时才调用 `set_article_quiz_activation`；保存文章、生成题目或发布文章不隐含启用测验授权。公开启用另需 `article:publish`。
- 启用会产生不可变测验版本，编辑草稿不会改变已启用题目。停用保留数据，不等于删除。正文变化提示复核，不自动重出题或启用。
- 创建文章并出题：先用现有普通文章工具保存草稿，再用返回的 `articleId` 关联测验。不向普通文章创建/更新接口加入测验字段。

## 边界

- 参考答案和评分标准只写入测验契约，不写进公开正文或公开变量；Markdown 题目清单不能声称为可交互测验。
- MCP 不读取读者答卷或成绩。客观题由站内判分，主观题由读者自己的 AI 配置评阅，生成题目不能承诺自动完成读者评分。
- Scope、写开关、AI 配置或版本错误按工具返回解释，不使用本地源码、数据库或其他接口绕过。完成后区分“候选待审阅”“草稿已保存”和“测验已启用”，只报告工具确认的状态。
