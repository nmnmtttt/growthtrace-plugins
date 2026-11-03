# 静态资源上传

使用 `upload_static_asset` 把用户明确提供的图片、文档、GLB 或 glTF 模型包保存为
GrowthTrace `StaticAsset`。一次工具调用最多创建一个逻辑资源，成功前不要把资源地址写入文章、
模板或组件。

## 调用流程

```text
用户要求保存静态资源
  -> 判断来源是宿主附件还是公开/签名 HTTPS URL
  -> GLB 单文件直接提交；glTF 收集完整文件组、相对 paths 与 mainFile
  -> 生成新的 16-128 字符 idempotencyKey
  -> 调用一次 upload_static_asset
  -> 等待工具返回
  -> 只使用 assetId/url/type/mimeType/size/originalName
```

工具负责下载、类型与签名校验、用户配额检查、R2 存储和资源登记。单次调用的全部文件合计最多
50 MiB；模型包也受服务端模型上限约束。不要在调用后自行执行上传或补登记。

## 来源

### 宿主附件

ChatGPT 对话附件、支持文件参数的 Codex 本地文件和 MCP App 文件选择都使用顶层 `files`。
调用方只发送附件，宿主把每项转换为包含 `download_url` 与稳定 `file_id` 的临时文件引用；可同时
提供 `mime_type` 和 `file_name`。GrowthTrace 不读取调用方本地路径，也不接受本地绝对路径、
`file:` 地址、base64 或放进工具 JSON 的二进制内容。

### 远程资源

其他客户端使用顶层 `urls`，每项只提供公开或带签名的 HTTPS URL，并可提供 `fileName`。不得提交
自定义 headers、Cookie、Authorization 或需要登录态的地址。普通图片、文档和 GLB 只传一个来源。

`files` 与 `urls` 必须且只能提供一个。临时下载 URL 可能刷新；宿主附件重试时保持相同
`file_id`、文件名、MIME 与路径身份。

## GLB 和 glTF

- GLB 是一个文件，不传 `paths` 或 `mainFile`。
- glTF 必须显式提交入口 `.gltf` 和全部 `.bin`、PNG、JPEG 或 WebP 依赖，最多一百个文件。
- `paths` 与 `files` 或 `urls` 按顺序一一对应，使用包内安全相对路径；`mainFile` 必须等于其中的
  `.gltf` 路径。
- 远程 glTF 也要列出入口和全部依赖 URL。不要等待服务端从 glTF 内容继续抓取遗漏文件。

## 幂等重试与结果

每次新上传生成一个 16 至 128 字符的 `idempotencyKey`。调用结果未知、连接中断或收到
`UPLOAD_IN_PROGRESS` 时，稍后使用完全相同的来源身份、路径和原幂等键重试。不要因为临时
`download_url` 刷新而更换宿主附件的 `file_id`。

收到 `IDEMPOTENCY_CONFLICT` 时，先核对该键原本对应的来源；确认这是新的资源请求后再生成新键。
收到 `UPLOAD_RECONCILIATION_REQUIRED` 时停止自动重试并报告需要核对结果。来源、大小、类型、
签名、模型或配额错误应先修正输入，再使用新幂等键发起新请求。

成功后只信任并使用工具返回的 `assetId`、`url`、`type`、`mimeType`、`size` 和
`originalName`。不要保存临时下载地址、文件 ID、R2 对象键或 OAuth 信息。

## 兼容入口

`prepare_image_upload` 与 `complete_image_upload` 仅供尚未升级的旧客户端兼容。新请求不要调用这
两个工具，也不要对旧工具返回的预签名地址执行 `PUT`；统一调用 `upload_static_asset` 并等待返回。
