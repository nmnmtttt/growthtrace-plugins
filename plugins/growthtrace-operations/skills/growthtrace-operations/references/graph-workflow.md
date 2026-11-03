# GrowthTrace 图谱工作流

本参考只用于构筑、校验、解释或保存图谱。实时 MCP capability 是当前枚举和数量限制的权威来源；
本文只保存稳定的数据形状、选择规则和安全流程。

## 选择正确的图

| 需求 | 使用的数据 | 是否持久化 | 工具 |
| --- | --- | --- | --- |
| 在文章中解释观点、知识、思维导图或流程 | `GraphDocument v1` | 作为文章 Markdown 的 `growthtrace-graph` fence | `get_graph_document_capabilities`、`validate_graph_document`，再用文章工具保存 |
| 查看项目、文章、资料、概念之间的真实关系 | `ProjectGraphDTO` | 由项目领域服务维护 | `get_writing_project_graph` |
| 新增或维护项目概念/人工关系 | 细粒度 concept/relation 请求 | 数据库中的 CONCEPT 与 USER ACTIVE relation | 六个 `*_project_graph_*` 写工具 |

不要把 GraphDocument 当作项目数据库快照，也不要用 ProjectGraphDTO 代替文章展示 JSON。不存在整图上传、
整图覆盖或任意图查询工具。

## 文章 GraphDocument v1

开始构图时先调用：

```json
{"name":"get_graph_document_capabilities","arguments":{}}
```

使用响应中的 `version`、`presets`、`nodeTypes`、`edgeTypes`、布局枚举与 `limits`；不要从本参考猜测
可能变化的数值。稳定的最小结构为：

```json
{
  "version": 1,
  "preset": "mindmap",
  "nodes": [
    { "id": "feedback-loop", "type": "topic", "label": "反馈循环" },
    { "id": "observe", "type": "concept", "label": "观察结果" }
  ],
  "edges": [
    {
      "id": "feedback-loop-observe",
      "source": "feedback-loop",
      "target": "observe",
      "type": "includes"
    }
  ],
  "layout": { "type": "tree", "direction": "LR" },
  "options": { "fitView": true }
}
```

节点可包含 `id`、受支持的 `type`、`label`、`description`、`content`、JSON-safe `data`、`position`、
`parentId`、`extent`、`expandParent` 和受控语义外观字段。边使用 `id + source + target`，并可包含
受支持的 `type`、`label`、`description`、JSON-safe `data`、`animated`、`edgeStyle` 和 `emphasis`。
布局与语义分离：改变 position、layout 或 viewport 不能改变关系含义。

必须满足：

- 节点 ID 和边 ID 各自唯一；`source`、`target`、`parentId` 只能引用同一文档内节点。
- 所有数据可安全 JSON 序列化；禁止函数、事件处理器、任意 CSS/style、className 或可执行源码。
- `mindmap` 必须恰好一个根节点且无有向环。
- 使用稳定、可读的本地 ID；不要把数据库 ID、OAuth 数据或私有路径放入文章图。
- 精确类型和限制以本次 capability 响应为准。

构造后调用：

```json
{
  "name": "validate_graph_document",
  "arguments": { "document": { "version": 1, "nodes": [], "edges": [] } }
}
```

`valid=true` 会返回补全默认值后的规范文档；`valid=false` 只返回有界 `code/path`。修正所有 issue 后
再次验证。验证成功只说明 JSON 可用，不表示文章已保存。

在 Markdown/MDX 文章中使用专用 fence：

````markdown
```growthtrace-graph
{
  "version": 1,
  "preset": "flow",
  "nodes": [
    { "id": "draft", "type": "start", "label": "形成草案" },
    { "id": "review", "type": "decision", "label": "是否成立" }
  ],
  "edges": [
    { "id": "draft-review", "source": "draft", "target": "review", "type": "depends" }
  ]
}
```
````

保存前读取最新文章并使用其 `updatedAt` 调用既有 `update_article`。除非用户明确要求公开，否则保持
现有发布状态；创建文章仍默认草稿。普通 `json` fence 不会渲染为图。若文章已有多个
`growthtrace-graph` fence，不猜测替换目标；在文章修改草案中明确要替换的图或说明将追加新图。

## 私有项目关系图

先调用：

```json
{"name":"get_writing_project_graph","arguments":{"projectId":"project-1"}}
```

返回的节点包含：

```ts
{
  id: 'ARTICLE:article-1',       // 画布/布局键，不作为 relation endpoint 整体传入
  entity: { kind: 'ARTICLE', id: 'article-1' }, // 关系端点的权威值
  label: '文章标题',
  description: null,
  href: '/article/article-1',
  readOnly: true,
  updatedAt: null
}
```

端点必须逐字复制已返回节点的 `entity.kind + entity.id`。不要猜 `graphId`、复合节点 ID 或数据库 ID；
这些工具没有 `graphId` 参数，统一使用 `projectId`。新概念的端点 ID 来自
`create_project_graph_concept` 返回的 `id`。

实体种类：`PROJECT | ARTICLE | RESOURCE | CONCEPT`。关系类型：`RELATES_TO | DEPENDS_ON |
REFERENCES | SUPPORTS | CONTRADICTS | PART_OF | CUSTOM`。`CUSTOM` 必须有 label。关系不能连接同一实体。

来源与生命周期：

- `SYSTEM`：由文章/资料项目归属派生，只读；文章归属继续使用 `assign_article_writing_project`。
- `USER ACTIVE`：用户确认后创建；只有这类关系可由项目图 MCP 更新或归档。
- `AI PENDING/ACTIVE`：站内建议流程拥有，项目图 MCP 不创建、不冒充、不修改、不归档。
- `readOnly` 是服务端输出，不是写入参数。调用方不得提交 `userId`、`origin`、`status`、
  `confidence` 或 `readOnly`。

## 确认与最小写入

固定流程：

1. `get_writing_project_graph` 读取权威节点、关系和版本。
2. 形成最小差异，逐项列出概念变更及 `source — relationType/label → target`。
3. 向用户展示这份精确差异并取得明确确认。提前的笼统授权或“后面都不用问”不覆盖尚未展示的
   实质变更；差异改变后必须重新确认。
4. 每项调用一个细粒度工具，带 `confirmation: "CONFIRM"` 和新的 `idempotencyKey`。
5. 再次 `get_writing_project_graph`，只把服务端返回结果作为成功事实。

创建示例：

```json
{
  "name": "create_project_graph_concept",
  "arguments": {
    "projectId": "project-1",
    "label": "反馈循环",
    "description": "用结果修正下一轮行动",
    "confirmation": "CONFIRM",
    "idempotencyKey": "concept-feedback-20260912-001"
  }
}
```

使用返回的 `id` 创建关系：

```json
{
  "name": "create_project_graph_relation",
  "arguments": {
    "projectId": "project-1",
    "source": { "kind": "ARTICLE", "id": "article-1" },
    "target": { "kind": "CONCEPT", "id": "concept-id-from-create-result" },
    "relationType": "SUPPORTS",
    "label": "提供实践依据",
    "confirmation": "CONFIRM",
    "idempotencyKey": "relation-feedback-20260912-001"
  }
}
```

更新/归档概念或关系时，`expectedUpdatedAt` 必须来自最新图读取或创建/更新响应。不得用整图重写
模拟批量同步，也不得猜测用户会接受的关系方向或 CUSTOM label。

## 幂等与失败恢复

- 同一个已确认调用发生网络超时或结果不确定时，用完全相同的 payload 和同一个幂等键重试。
- 新的语义操作使用新键；同一键绑定不同 payload 会返回 `IDEMPOTENCY_CONFLICT`。
- 多项写入不是跨工具事务。每项成功后保留返回 ID；后续失败时重新读取图，核对已成功项，再决定
  是否用原键重试同一请求。不要重复创建概念，也不要用猜测性补偿写入模拟回滚。

| 错误 | 处理 |
| --- | --- |
| `GRAPH_DOCUMENT_INVALID` 或 validation issues | 按 path 修正文档并重新校验，不写文章 |
| `WRITING_PROJECT_NOT_FOUND` | 停止；不可用 ID 探测其他用户项目 |
| `PROJECT_GRAPH_ENTITY_NOT_FOUND` | 重读项目图，使用当前 `node.entity` 重建差异并重新确认 |
| `PROJECT_GRAPH_RELATION_DUPLICATE` | 重读并复用已有关系，不创建平行边 |
| `PROJECT_GRAPH_VERSION_CONFLICT` | 重读、重新形成差异、重新展示并确认，不覆盖新版本 |
| `IDEMPOTENCY_CONFLICT` | 不换 payload 重试旧键；为经过新确认的新操作生成新键 |
| `INSUFFICIENT_SCOPE` | 停止写入并让用户完成所需 OAuth 授权 |

## 行业兼容边界

不存在所有 UI 图、图数据库和文本图语言共同遵循的单一 JSON 标准。GrowthTrace GraphDocument 是
**React Flow/XYFlow-compatible semantic subset and extension**：共享 `nodes + edges`、节点
`id/position/data/type/parentId` 与边 `id/source/target/data/type`，但增加版本、语义 preset、受控外观
和持久化安全限制。它不是 React Flow 运行时对象的无过滤序列化。

参考：[ReactFlowJsonObject](https://reactflow.dev/api-reference/types/react-flow-json-object)、
[React Flow Node](https://reactflow.dev/api-reference/types/node)。

```text
GraphDocument.nodes[n].id            <-> React Flow node.id
GraphDocument.nodes[n].position      <-> React Flow node.position
GraphDocument.edges[n].source/target <-> React Flow edge.source/target
```

Cytoscape.js requires an adapter:

```text
GraphDocument.nodes[n].id            <-> cytoscape node.data.id
GraphDocument.nodes[n].parentId      <-> cytoscape node.data.parent
GraphDocument.nodes[n].position      <-> cytoscape node.position
GraphDocument.edges[n].id            <-> cytoscape edge.data.id
GraphDocument.edges[n].source/target <-> cytoscape edge.data.source/target
```

不导入 Cytoscape scratch、任意 style 或交互状态。参考：
[Cytoscape.js elements JSON](https://js.cytoscape.org/#notation/elements-json)。

GraphSON is a conceptual property-graph mapping，适合 Apache TinkerPop 的图数据库类型和遍历生态，
不是当前文章 UI 文档的直接输入。参考：[Apache TinkerPop GraphSON](https://tinkerpop.apache.org/docs/current/dev/io/)。

Mermaid is a textual DSL, not an interchangeable editor-state format。首期不自动互转，避免丢失位置、
自定义 data、项目来源和生命周期语义。
