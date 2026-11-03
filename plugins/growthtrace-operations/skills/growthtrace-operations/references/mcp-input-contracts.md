# MCP 输入契约与工具定义同步

## 定义优先级

调用参数以当前连接从服务端 `tools/list` 获取的 `inputSchema` 为准；能力工具返回的业务契约决定组件、图谱和测验的内容格式。插件版本、服务端版本和客户端缓存的工具定义是不同状态，更新插件不代表已有会话已经刷新工具定义。

如果宿主仍把项目档案定义成 `profile.type`，而服务端返回 `profile.dimensions` 缺失或旧字段 `Unrecognized keys`，停止重复写入，提示重新同步连接的工具列表、更新插件并开启新会话。不要伪造额外字段绕过宿主 Schema、改用网站私有 API，或删除用户档案内容来满足旧定义。工具缺失、组件工具没有 `dependencies` 或展示稿工具缺少绑定、版本锁和应用确认参数时，也应先核对连接版本。仅工具定义过期时刷新定义；只有 `INSUFFICIENT_SCOPE` 或授权失效才进入对应授权流程。

## 项目创建

`type` 只在项目顶层，`profile` 使用动态维度；名称、类型、描述、档案和写作规则均来自已确认草案。

```json
{
    "name": "北城长篇",
    "type": "NOVEL",
    "description": "围绕北城三代人的长篇小说",
    "profile": {
        "dimensions": [
            { "id": "worldview", "name": "世界观", "content": "近未来的北方城市", "enabled": true }
        ]
    },
    "instructions": [{ "content": "保持人物视角一致" }],
    "idempotencyKey": "create-project-example-0001"
}
```

维度 `id` 必须唯一，每项包含 `id/name/content/enabled`。档案没有内容时可显式使用 `profile: { "dimensions": [] }`，写作规则没有内容时使用 `instructions: []`。历史项目读取仍可能返回旧的 `audience/worldview/characters/goals` 等固定字段；这是读取兼容，不能把历史返回对象直接复制成创建参数。创建时将每个已有字段保留为有名称的维度，字符串保留原文，列表转换为 Markdown；仅改变序列化形式，不能擅自改写已确认的业务含义。

## 文章写入

- 标题 1–300 字符，正文 1–200000 字符；摘要、描述各最多 1000 字符；关键词最多 30 项、每项最多 100 字符。
- `slug` 使用安全的单段公开路径，不能含 `/` 或占用 `api/login/projects` 等站点路由。发布时必须提供有效 slug；已发布文章不能清空链接。草稿可省略链接。
- `expectedUpdatedAt` 使用最新读取结果中的完整 ISO 时间戳，不使用本地日期、自然语言或猜测的时间。
- `create_article` 默认草稿。发布使用显式 `mode: "publish"` 或独立发布工具；字段生成只保存审阅候选。

## 文章展示稿

先调用 `get_article_display_context({"articleId":"实际文章ID"})` 获取正文、组件规范、文章版本与 `currentCandidate`。服务端先确认母稿所有权，再读取同用户、同文章最新 `LAYOUT_OUTPUT`；`currentCandidate` 为 `null` 或 `{productId, productUpdatedAt, displayContent, componentBindings, generationContext?}`，不返回产物变量、metadata 或凭据。用户要求查看、审阅或应用现有候选时，直接使用该结果定位，不重新提交覆盖。需要新稿或修改时，外部自行撰写完整 MDX。保存参数如下；标识和时间必须替换为实际读取结果：

```json
{
  "articleId": "实际文章ID",
  "expectedUpdatedAt": "2026-11-02T02:00:00.000Z",
  "displayContent": "# 标题\n\n<Admonition type=\"note\">正文</Admonition>",
  "componentBindings": [],
  "idempotencyKey": "external-mdx-save-0001"
}
```

提交 `save_article_display_content` 不接受模板 ID、调用方 userId 或 generationContext。个人组件使用真实目录中的 `alias/componentId/versionId`，没有个人组件仍传空数组。文章版本冲突后重读上下文并调整 MDX，不伪造时间戳。保存只产生待审阅候选，并返回 `productId/productUpdatedAt/generationContext`；默认展示预览并等待用户审阅。

只有用户明确要求应用当前候选，或授权本次任务“生成后直接应用并展示”，才调用 `apply_article_display_content`；已有授权无需重复确认。应用参数使用同一候选保存结果，不猜测或刷新时间戳来绕过冲突：

```json
{
  "articleId": "实际文章ID",
  "productId": "实际候选ID",
  "expectedUpdatedAt": "2026-11-02T02:00:00.000Z",
  "expectedProductUpdatedAt": "2026-11-02T02:01:00.000Z",
  "confirmation": "CONFIRM",
  "idempotencyKey": "external-mdx-apply-0001"
}
```

`expectedUpdatedAt` 取当前上下文的 `article.updatedAt`，且必须与所选候选的 `generationContext.articleUpdatedAt` 一致；`expectedProductUpdatedAt` 取 `currentCandidate.productUpdatedAt`，新保存候选则取保存结果的 `productUpdatedAt`。`generationContext` 缺失或来源不受支持时停止直接应用并说明原因，需要重新撰写时使用真实上下文，不伪造来源。应用从服务端读取该文章的锁定候选，不接受调用方覆盖正文、展示稿、来源或绑定，并在同一应用事务内检查文章和候选版本。候选被替换、文章变化或来源校验失败时停止应用并重读核对，不能只换时间戳重试。

“生成、排版、预览、展示”或 MCP/OAuth 连接授权本身不含应用授权，`confirmation: "CONFIRM"` 只能表达已经取得的用户授权。应用需要 `article:write + layout:read`、MCP 写开关、双版本锁和幂等键，保持当前发布状态；已公开文章同步更新展示快照，另行公开仍需明确指令。

返回 `applied: true` 后核对实际 `displayContent/componentBindings/updatedAt/published`；`previewStatus: "ready"` 时展示 MCP App 预览，不支持内嵌时提供 `previewUrl`。`previewStatus: "unavailable"` 仍表示已经应用，应说明稳定的 `previewError` 并提供 `reviewUrl`。可用完全相同的请求和原幂等键重试以重建预览，不重复写文章；不要换键重新应用或生成。文章已再次修改时预览拒绝 `ARTICLE_UPDATE_CONFLICT`，不能把新版本当成本次结果。展示稿直接应用授权不改变生成组件的独立审批流程。

## 组件与上传

- 生成组件 Props Schema 使用逐属性规则，例如 `{"count":{"type":"number","required":true}}`；类型为 `string/number/boolean/array/object`，不是根节点带 `type/properties` 的通用 JSON Schema，也没有 `integer` 类型。
- 组件更新使用同次读取的 `componentId/baseVersionId/expectedSourceHash`，不能只发送版本基线而省略组件 ID。个人组件依赖用 `dependencies: [{alias, componentId, versionId}]` 锁定；其他限制读取实时 capabilities。
- 上传只用 HTTPS 来源。普通资源只传一项；多项来源仅用于带 `paths/mainFile` 的 glTF 包。路径必须唯一且为安全相对路径，入口以 `.gltf` 结尾；详细步骤见 [静态资源上传](static-asset-upload.md)。

`idempotencyKey` 对应一次确定的业务请求；请求内容修正后使用新键，结果未知的原请求则保留原键核对或重试。
