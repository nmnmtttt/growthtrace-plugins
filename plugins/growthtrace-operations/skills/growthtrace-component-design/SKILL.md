---
name: growthtrace-component-design
description: Use when users ask to create, reuse, update, or preview GrowthTrace generated display components or use them in article display drafts. Does not apply to repository UI implementation or automatically applying article display content.
---

# GrowthTrace 组件设计

必须先完整读取同一插件的 [网站运营规则](../growthtrace-operations/SKILL.md) 及其“生成组件”章节指定的四份 reference。运行时约束和审核状态机以这些资源及实时 capability contract 为准，本入口不另行定义能力。

提交前核对[输入契约](../growthtrace-operations/references/mcp-input-contracts.md)中的 Props Schema 方言、组件依赖和完整更新基线；宿主仍暴露旧字段时先同步工具定义，不改用其他入口绕过。

1. 调用 `get_generated_component_capabilities`，再按需列出或读取已有组件，判断复用、新建或更新。
2. 需要调用站内 AI 时用 `generate_generated_component_preview`，传入用户要求和新的幂等键；它复用用户模型配置并返回相同 MCP App 预览。外部助手自行编写源码时仍使用 `create_generated_component_preview`。更新时两条路径都带最新版本和源码 hash；编译错误按结构化 diagnostics 修复。
3. 有浏览器能力时检查 desktop/mobile 预览；没有时提供 review URL 并明确未做助手视觉检查。临时审核页的 iframe 限制不改变文章直接渲染的目标架构。
4. 用户在网站审核页批准，且服务端确认 owner READY 后，才调用 `finalize_generated_component_preview`。不代替用户审批，不伪造运行状态。
5. 更新绑定最新 `componentId + baseVersionId + expectedSourceHash`；最终只引用工具返回的不可变版本。任务还包含文章展示稿时，继续按[文章展示撰写技能](../growthtrace-article-display/SKILL.md)重新读取上下文并组合 MDX；默认提交候选供审阅，只有已取得该文章本次应用授权时才直接应用并展示。展示稿授权不替代第 4 步的组件审批。

“设计一个文章时间线组件”只授权组件设计流程；不会自动发布文章、保存展示稿，或用 `update_article` 覆盖文章正文。
