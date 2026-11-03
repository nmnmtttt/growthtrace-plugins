---
name: growthtrace-article-display
description: >-
    Use when users ask to write, revise, preview, submit, or apply a GrowthTrace article display or presentation draft based on a saved article and project components. Does not apply to ordinary writing, standalone visual design, or repository UI implementation.
---

# GrowthTrace 文章展示撰写

外部模型依据已保存文章和项目组件直接编写标准 MDX。GrowthTrace MCP 提供真实上下文、校验、候选保存、预览与明确授权后的应用；站内不调用 AI 生成展示稿。默认等待用户审阅，布局模板功能已移除。

参数遵循[输入契约](../growthtrace-operations/references/mcp-input-contracts.md)。宿主缺少下面的新工具时，先同步 `tools/list` 或升级插件，不改用正文更新绕过候选流程。

## 工作流

1. 调用 `get_article_display_context(articleId)`，取得文章 `content/updatedAt/componentBindings/variableNames`、内置组件 `builtInComponents`、个人组件 `personalComponents`、`mdxGuide` 和 `currentCandidate`。文章必须已保存；服务端先确认母稿所有权，再读取同用户、同文章最新的 `LAYOUT_OUTPUT`。`currentCandidate` 为 `null` 或 `{productId, productUpdatedAt, displayContent, componentBindings, generationContext?}`，不包含产物变量、metadata 或凭据。只要求查看或应用当前候选时，使用该结果定位候选，跳过步骤 2–4，不重新提交覆盖。
2. 根据正文与展示目标选择组件。内置组件按用途、Props 与默认 MDX 示例使用；个人组件按描述、Props Schema、变量依赖和静态/交互模式选择，沿用返回的确切别名和版本锁。
3. 由当前外部模型编写完整 MDX，保留文章事实与内容。普通段落使用 Markdown；内置组件按规范组合，个人组件使用自闭合别名，组件 ID/版本写入独立 `componentBindings`。不虚构数据、变量或组件。
4. 调用 `save_article_display_content`，提交 `articleId`、完整 `displayContent`、`componentBindings`、读取时的 `updatedAt` 作为 `expectedUpdatedAt`，以及新的 `idempotencyKey`。没有个人组件时显式传 `componentBindings: []`。服务器只校验并保存当前 `LAYOUT_OUTPUT` 候选，不调用生成服务；保留返回的 `productId/productUpdatedAt/generationContext`，用于锁定后续应用的候选与文章版本。
5. 新保存候选使用返回的预览；只查看已有候选时，调用 `create_article_display_preview(articleId, currentCandidate.productId)` 获取 `previewUrl/reviewUrl`。支持 MCP App 的宿主直接显示预览，其他宿主提供预览链接。已有明确应用授权时可直接进入步骤 7，再展示实际应用结果。地址过期用 `refresh_article_display_preview_url`，不要为刷新地址重新提交候选。
6. 未取得应用授权时，提供 `reviewUrl`，说明候选已保存、尚未应用，等待用户审阅。用户可在 GrowthTrace 文章页“产物”面板审阅并点击“应用”，也可在对话中明确要求应用当前候选。
7. 用户明确要求应用当前候选，或已授权本次任务“生成后直接应用并展示”时，调用 `apply_article_display_content`。提交实际 `articleId/productId`、当前文章上下文的 `article.updatedAt` 作为 `expectedUpdatedAt`、所选 `currentCandidate.productUpdatedAt`（新保存候选则取保存结果 `productUpdatedAt`）作为 `expectedProductUpdatedAt`、`confirmation: "CONFIRM"` 和新的 `idempotencyKey`。文章版本必须与候选的 `generationContext.articleUpdatedAt` 一致；来源缺失、不受支持或已过期时停止直接应用并说明原因，不伪造来源或仅更换时间戳。已有明确授权无需再次确认或要求用户到网站手动点击；工具缺失时先同步定义，不改用 `update_article`。
8. 依据 `applied: true` 与返回的实际保存内容、文章版本核对结果。`previewStatus: "ready"` 时通过 MCP App 展示已应用的展示稿；不支持内嵌展示时提供返回的预览链接。候选预览不能冒充已应用结果。`previewStatus: "unavailable"` 时准确说明“已应用，预览暂不可用”，提供 `reviewUrl` 并说明稳定的 `previewError`。可用完全相同的请求和原幂等键重试以重建预览，不重复写文章；不要更换幂等键重新应用或重新生成。文章已再次修改时停止重试，避免把其他版本展示为本次结果。

同一文章再次提交会替换当前候选。用户明确需要保留旧候选时，先提示这一影响。内容校验失败按安全 `code/path/line/column` 修正；版本冲突先重读上下文，再基于新正文调整，不能仅替换时间戳重交旧稿。

## 应用授权

- “生成一份”“排版一下”“预览”“展示给我看”只授权生成或预览，不构成应用授权。MCP 连接或 `site:full` OAuth 授权也不等于每次产物的应用授权。
- “应用当前展示稿”“生成后直接应用并展示”允许在明确指定的文章与本次任务范围内调用专用应用工具，不能扩展到其他文章或后续任务。`confirmation: "CONFIRM"` 只能表达已经取得的授权，不能由助手自行推定。
- 应用保留当前发布状态；公开发布另需明确指令和独立发布操作。对已公开文章应用展示稿会同步更新其公开展示快照。

## 组件不足时

优先复用内置或已发布组件。只有必要交互无法表达时，才使用 `$growthtrace-component-design` 的组件设计、预览、用户批准与 finalize 流程。展示稿直接应用授权不替代组件的登录审批、`ownerRuntimeStatus=READY` 与 finalize。获得真实 `componentId + versionId` 后，重新读取撰写上下文并在 MDX 与绑定中引用该版本。

## 安全边界

- JSX 属性表达式只接受 JSON 值；变量属性使用 `bindings={{"prop":"existing.path"}}`，遵循返回的 `mdxGuide`。
- 不输出 import/export、正文 JavaScript、spread、事件处理器、脚本、未注册标签或旧 `GeneratedComponent` 包装器；个人组件不能接收 children。
- 不修改文章变量，不读取组件 bundle，不自行执行组件源码。
- 不调用 `update_article` 写入展示稿，不伪造或删去 `generationContext`；只有明确应用授权才调用专用应用工具，权限、写开关、来源和组件绑定校验始终保留，文章与候选版本在同一应用事务内检查。
- 只查看普通文章正文时使用 `create_article_preview`，无需撰写展示稿。
