---
name: growthtrace-operations
description: >-
    Use when users ask to operate GrowthTrace website content, article quizzes, diagrams, or writing-project relationships through the authorized MCP, including “查看 GrowthTrace 的文章”, “保存到 GrowthTrace”, “给文章出题”, “把观点做成思维导图”, “整理项目关系”, or “检查我的网站内容健康”. Match “我的网站”, “我的博客”, “我的站点”, or “网站后台” when context identifies them as GrowthTrace. Excludes standalone writing without website intent, repository source-code, deployment, database, and user-administration tasks.
---

# GrowthTrace 网站运营

通过已授权的 GrowthTrace MCP 处理网站内容、文章关联测验、写作项目、静态资源、文章展示稿和生成组件。先取得 MCP 中的真实数据再判断；不要改用本地工作区、源码扫描或要求用户重复提供网站 URL。

写入前读取[输入契约与工具定义同步](references/mcp-input-contracts.md)，核对当前工具参数。遇到 `profile.dimensions` 缺失、旧字段不识别或工具缺失时，先处理宿主工具定义过期，不重复提交旧参数。

## 自然语言触发与网站指代

- 用户明确提到 GrowthTrace 并要求查看或管理网站内容时使用本技能，无需用户提供技能名或工具名。例如“看看 GrowthTrace 有哪些草稿”“把这段内容保存到 GrowthTrace”。
- 当前对话已确认“我的网站”“我的博客”“我的站点”或“网站后台”指 GrowthTrace 时，后续沿用该指代；例如“看看我的网站有哪些文章”。不要每次重复确认或要求用户重新选择插件。
- 用户使用上述泛称，但上下文尚未确定所指网站时，先简短确认是否指 GrowthTrace；用户明确指向其他网站时，不使用本技能处理该网站。不要仅因安装了插件就认定所有网站请求都指 GrowthTrace。
- 只有网站内容操作意图明确时，才将“整理成文章”“保存这段内容”等后续请求交给本技能。普通写作、润色或讨论插件本身不构成网站写入指令；只说“GrowthTrace”而没有任务时，询问希望进行什么操作。
- 自然语言匹配不改变下文的授权与发布规则；“保存到我的网站”默认保存草稿，不等同于公开发布。

## 工具速查

| 能力 | MCP 工具 |
| --- | --- |
| 站点与文章读取 | `get_site_overview`、`list_articles`、`get_article`、`search_articles`、`get_content_health` |
| 普通文章内嵌预览 | `create_article_preview`、`refresh_article_preview_url` |
| 文章写入与发布 | `create_article`、`update_article`、`set_article_publication` |
| 文章测验 | `get_article_quiz_capabilities`、`validate_article_quiz`、`get_article_quiz`、`generate_article_quiz`、`save_article_quiz`、`set_article_quiz_activation` |
| 写作项目读取 | `list_writing_projects`、`get_writing_project`、`match_article_writing_project` |
| 写作项目写入 | `create_writing_project`、`assign_article_writing_project` |
| 文章图谱契约 | `get_graph_document_capabilities`、`validate_graph_document` |
| 项目关系图读取 | `get_writing_project_graph` |
| 项目概念写入 | `create_project_graph_concept`、`update_project_graph_concept`、`archive_project_graph_concept` |
| 项目关系写入 | `create_project_graph_relation`、`update_project_graph_relation`、`archive_project_graph_relation` |
| 静态资源上传 | `upload_static_asset` |
| 站内 AI | `generate_article_draft`、`generate_article_fields`、`generate_generated_component_preview` |
| 展示稿候选、应用与预览 | `get_article_display_context`、`save_article_display_content`、`apply_article_display_content`、`create_article_display_preview`、`refresh_article_display_preview_url` |
| 生成组件 | `get_generated_component_capabilities`、`list_generated_components`、`get_generated_component`、`create_generated_component_preview`、`get_generated_component_preview`、`refresh_generated_component_preview_url`、`finalize_generated_component_preview`、`archive_generated_component` |

## 工作方式

- 用户要求检查内容健康、内容库存或主题覆盖时，优先调用 `get_content_health` 和 `get_site_overview`；需要证据时再使用 `list_articles`、`search_articles` 或 `get_article`。
- 所有结论区分“工具返回的事实”和“基于事实的建议”。当结果为 `analytics: not_configured` 时，明确说明没有访客、来源或阅读量数据，不能推断流量表现。
- 用户要求“整理为文章”时，将当前讨论整理成合适的标题、摘要、关键词和 Markdown 正文，调用 `create_article` 并使用 `mode: "draft"`。
- 用户要求在 Codex 内嵌查看普通文章时，先定位已保存文章，再调用 `create_article_preview(articleId)`。创建文章并预览的任务先保存草稿，再调用该工具。结果关联 `ui://growthtrace/article-preview/v1.html`；支持 MCP Apps 的客户端内嵌显示，否则提供 `previewUrl`。正文仍按 GrowthTrace Markdown 渲染，不需要模板、生成产物、文章公开或站内应用。
- 正文预览仅需 `site:read`，沿用24小时临时会话与30分钟只读令牌。链接过期调用 `refresh_article_preview_url`；刷新保留创建时的冻结正文。查看文章最新修改时重新创建预览，不通过再次生成文章刷新链接。
- 只有用户明确要求“发布”“公开”或等价表达时，才调用带 publish 行为的工具；如写入参数不完整或内容风险会改变公开含义，先请求用户确认。
- 修改既有文章前先读取其详情并使用返回的 `updatedAt` 作为 `expectedUpdatedAt`；遇到版本冲突时展示差异并请求用户决定，不覆盖较新版本。
- 用户要求上传本地附件、宿主附件或公开/签名 HTTPS 静态资源时，先完整读取 `references/static-asset-upload.md`。按来源准备文件引用或 URL，调用一次 `upload_static_asset` 并等待返回；只有成功后才使用返回的资源 URL。

## 文章关联测验

用户要求“给文章出题”“根据正文生成测验”或编辑、启停练习时，读取[文章测验技能](../growthtrace-article-quiz/SKILL.md)。测验独立关联普通文章，不改变文章类型或正文接口。先读取实时能力与现有测验；站内 AI 生成的是 `QUIZ_DRAFT` 审阅候选，手写题目通过校验后保存草稿。只在明确要求时启用，启用需要发布权限；不读取读者答卷或成绩。

## 写作项目与文章归属

- 查询或创建项目时先调用 `list_writing_projects`；存在足以承载当前文章的活跃项目时优先建议复用，不重复创建。
- 用户要求创建项目时，根据其描述形成包含名称、类型、描述、动态维度档案和写作规则的完整草案。`type` 保留在顶层，档案用 `profile.dimensions`，不发送旧 `profile.type`。先展示草案；只有用户确认全部字段后才调用 `create_writing_project`。确认后改变实质字段必须重新确认，写入后用 `get_writing_project` 核对真实结果。
- 为文章推荐归属时先调用 `get_article`，再调用 `match_article_writing_project`。匹配读取的是服务端已保存文章；不要把本地文本当作站内事实。
- 只有 `decision=RECOMMEND` 才作为高可信推荐展示，并同时说明项目名称、置信度和原因。`NO_CONFIDENT_MATCH` 时不包装低可信候选、不自行选项目，也不自动创建项目。
- 推荐本身不写入。用户明确确认具体推荐后，才调用 `assign_article_writing_project`，使用 `basis=CONFIRMED_RECOMMENDATION` 和匹配结果的 `articleUpdatedAt`。若结果过期，重新读取和推荐，不能把变化后的候选视作已确认。
- 用户在原始指令中已明确指定文章和项目时，可使用 `basis=EXPLICIT_USER_CHOICE`；这份授权只覆盖该文章与项目。更换或取消已有归属前展示当前变化并取得确认，禁止静默迁移。
- `ALREADY_ASSIGNED` 只报告当前归属。归属写入后重新调用 `match_article_writing_project` 核对当前项目，区分 MCP 返回的事实和 Codex 的建议。

## 图谱与流程说明

请求涉及构筑、校验、解释或保存关系图、知识图、思维导图、流程图、项目概念或项目关系时，必须完整读取 `references/graph-workflow.md`。

- 文章图先调用 `get_graph_document_capabilities`，按实时契约形成 `GraphDocument v1`，再调用 `validate_graph_document`。校验成功不等于保存；使用专用 `growthtrace-graph` fence，并沿用文章最新版本与草稿优先规则。
- 项目图先调用 `get_writing_project_graph`。关系端点逐字使用返回节点的 `entity`，不猜 `graphId`、复合节点 ID 或站内实体 ID。
- 写入前展示最小概念/关系差异；明确确认只覆盖已经展示的差异。随后逐项最小写入并重读核对，不进行整图覆盖。
- SYSTEM 关系只读，AI 关系由站内建议流程管理；项目图 MCP 只创建、更新或归档 USER ACTIVE 关系。

## 文章展示稿

按 `$growthtrace-article-display` 工作流操作：读取 `get_article_display_context`，由外部模型依据文章及内置/个人组件的真实规范直接编写 MDX，再通过 `save_article_display_content` 校验、幂等保存候选并预览。提交包含 `articleId/displayContent/componentBindings/expectedUpdatedAt/idempotencyKey`，不依赖模板或站内展示稿生成服务。

上下文还返回 `currentCandidate: null` 或 `{productId, productUpdatedAt, displayContent, componentBindings, generationContext?}`。服务端先确认母稿所有权，再读取同用户、同文章最新 `LAYOUT_OUTPUT`，不返回产物变量、metadata 或凭据。只要求查看、审阅或应用现有候选时直接使用该结果定位，不重新提交覆盖；候选缺少受支持的来源时停止直接应用并说明原因，不伪造来源。

保存只更新当前 `LAYOUT_OUTPUT` 候选，返回 `productId/productUpdatedAt/reviewUrl/previewUrl/generationContext`，默认展示预览并等待用户审阅。“生成、排版、预览、展示”以及 MCP 连接或 `site:full` OAuth 授权均不构成应用授权。

用户明确要求应用当前候选，或授权本次任务“生成后直接应用并展示”时，可调用 `apply_article_display_content`，无需再次确认或要求用户到网站手动点击。输入绑定实际 `articleId/productId`、当前上下文 `article.updatedAt` 对应的 `expectedUpdatedAt`、`currentCandidate.productUpdatedAt`（新保存候选则取保存结果 `productUpdatedAt`）对应的 `expectedProductUpdatedAt`、`confirmation: "CONFIRM"` 和新幂等键。文章版本必须与候选 `generationContext.articleUpdatedAt` 一致，过期不应用。应用仍检查权限、写开关、来源和组件绑定，并在同一应用事务内检查文章与候选版本；返回 `applied: true` 后核对实际保存结果，`previewStatus: "ready"` 时通过 MCP App 展示已应用稿，宿主不支持时提供预览链接。`previewStatus: "unavailable"` 时说明已应用、稳定的 `previewError` 和文章查看入口；可用原请求与原幂等键重建预览，不重复写文章或重新生成。文章已再次修改时停止重试，不展示错误版本。

应用保持发布状态，已公开文章会同步更新展示快照；公开发布另需明确指令。展示稿的直接应用授权不替代生成组件的登录审批、owner READY 和 finalize。未取得应用授权时提供 `reviewUrl`，等待用户审阅或在站内应用。过期链接用 `refresh_article_display_preview_url`；版本冲突后必须重读并核对变化，不能仅替换时间戳或静默应用其他候选。普通文章正文只用 `create_article_preview`，禁止通过 `update_article` 绕过展示稿专用流程。

## 组件生成的实现方式

- 生成的是单文件、单一 `default function` 导出的受限 React TSX 组件，通过站点的编译、校验与版本管理接入现有组件系统。不要生成完整 HTML 页面、iframe、srcDoc、独立 React 根节点或自行注入 script；案例不需要在 iframe 内实现。
- 文章负责介绍、说明和代码示例，组件负责交互，静态资源负责图片与媒体。优先复用现有组件，需要共享筛选、选中项等状态的交互放在同一组件内，通过 Props / Props Schema 和受控变量传入数据。
- 工坊预览在主页面共享 React/UI runtime 中直接渲染。文章使用自闭合别名标签（如 `<BakeryShowcase />`），由正文外的 `componentBindings` 锁定 alias、componentId 和 versionId；不生成旧 `<GeneratedComponent />` 包装器，也不把内部 ID 写进正文。
- 公开页先输出已发布版本的 SSR 快照；交互组件再加载同源不可变资源，在原 DOM 位置接管。静态组件不需要客户端执行；不要为案例另建 iframe 承载页或在线代码执行环境。
- 图片和媒体使用站点已有的受控资源能力或文章内置媒体组件。不能因为直接渲染就假定任意 URL、原生资源标签或网络 API 均可用；以当前组件编译策略和可用资源接口为准。
- MCP 临时审核页当前仍有独立 iframe 实现，这是审核工具的现状，不是生成组件的目标架构。其网络限制不能推导为文章公开页不能展示静态资源，也不能作为案例必须改造 iframe 的理由。若临时审核不能验证某项效果，应说明验证范围，保留审批流程，不宣称已完成该项验证。

## 生成组件

执行任何生成组件任务前，必须完整读取以下四份运行时 reference；它们是本 Skill 的离线契约和决策状态机，不要凭已有组件源码或第三方包文档猜测能力：

- `references/generated-component-runtime.md`：版本化能力快照、模块/导出、Schema 与安全限制。
- `references/generated-component-contract.json`：由 canonical registry 确定性生成的完整机器契约；实时 MCP 不可用时以此为精确能力边界。
- `references/generated-component-workflow.md`：复用、新建、更新、浏览器/无浏览器审核和审批流程。
- `references/generated-component-errors.md`：结构化诊断、自动恢复和何时才反馈用户。

随后调用 `get_generated_component_capabilities` 获取实时权威契约。实时契约优先于离线快照；同版本 hash 不同意味着插件分发不一致，停止组件写入并报告。新 major 版本超出本 Skill 支持范围时，只能使用双方明确共有的能力，不能试探性写入。

站内 AI 可通过 `generate_generated_component_preview` 生成组件候选；外部助手提供源码时使用 `create_generated_component_preview`，二者共用审批和最终保存流程。

核心闭环是 capabilities → 查重复用 → 必要时生成最小 TSX/Schema/预览数据 → 编译修复 → 浏览器检查或无浏览器降级 → 用户在登录审核页批准 → owner READY → finalize。更新必须先读取当前 source，并同时绑定 `componentId + baseVersionId + expectedSourceHash`；冲突后重新读取并在新基线上生成最小差异，绝不覆盖较新版本。编译修复不计入视觉候选轮次，视觉调整最多三轮。

状态查询不会刷新 preview URL/token。只有地址丢失或过期时才调用 `refresh_generated_component_preview_url`，且旧地址会立即失效。`agentRuntimeStatus` 只表示 token 预览运行结果；`ownerRuntimeStatus=READY` 才是批准和 finalize 的服务端前置条件，Codex 不能代批、伪造 READY 或从 agent READY 推断 owner READY。finalize 成功后只使用服务端返回的 `componentId + versionId` 作为可引用的不可变版本。

信息不足时依次使用站内 list/search/get、实时 capabilities、结构化编译诊断和安全默认值补齐；只有 OAuth/scope、用户审批、品牌资产、核心交互含义、公开范围、重大降级选择必须立即询问用户。确认平台不支持后，先给出受支持替代方案；只有没有等价方案或选择会实质改变结果时才反馈阻塞。

组件 finalize 后重新调用 `get_article_display_context`，使用返回的确切别名、`componentId + versionId` 编写展示稿；组件流程本身不会保存文章展示稿。

## 安全边界

- MCP 未连接、未授权或工具不可用时，说明需要完成 GrowthTrace MCP 连接/OAuth 授权；不要退化为扫描本地项目或模拟网站数据。
- 只使用 GrowthTrace 的文章、关联测验、写作项目、内容健康、静态资源、文章展示稿和生成组件工具。项目资料不在当前 MCP 能力内；不得借项目工具恢复已退役 Topic 或网页采集能力。远程静态资源只把公开或签名 HTTPS 地址作为 `upload_static_asset` 参数交给 GrowthTrace；不要自行抓取它、添加 headers/Cookie，或对预签名 R2 地址执行 `PUT`。仍然不要访问或修改仓库源码，或进行 Git、Shell、数据库、部署、环境变量、用户管理及第三方发布操作。生成组件 bundle 只能通过 MCP 返回的 `previewUrl` 在隔离 iframe 中检查，不能下载后在 Shell、本地项目或其他运行时执行。
- 诊断报告只保留在当前对话；除非用户明确要求创建文章，不把报告写回 GrowthTrace。
- 静态资源上传或校验失败时停止使用资源 URL，并向用户说明稳定错误；不要把 OAuth token、本地绝对路径、临时下载地址或资源二进制写入文章。
- 项目创建、文章归属、项目图概念/关系写入、展示稿候选保存、展示稿应用和组件 finalize 必须使用新的幂等键；临时 preview 创建/查询/URL 刷新不接收幂等键。项目图写入还必须带 `confirmation=CONFIRM`，更新/归档使用最新 `expectedUpdatedAt`，并且只接受当前 principal 拥有项目中的 CONCEPT 或 USER ACTIVE 关系。文章更新和候选保存的 `expectedUpdatedAt` 必须来自实际读取结果；展示稿应用同时绑定候选返回的文章与产物版本，并且必须有明确应用授权及 `confirmation=CONFIRM`。项目归属使用匹配返回的 `articleUpdatedAt`，组件源码更新使用 `baseVersionId + expectedSourceHash`。scope、功能开关或版本冲突失败时停止，不尝试绕过确认或版本边界。
