# GrowthTrace 网站运营插件

此插件提供 GrowthTrace 内容运营技能与插件展示信息，并通过包内 MCP 配置连接服务，地址为 `https://leecrafthub.com/api/mcp`。

## 插件展示与技能

0.11.0 包含远程 MCP 连接配置、品牌图标、详细介绍、五条示例提示词与七个技能入口：网站运营、文章管理、写作项目、内容健康、文章展示、组件设计、文章测验。本版本移除模板功能，展示稿由外部模型依据文章和内置/个人组件直接撰写；MCP 只负责校验、保存候选与预览。文章关联测验：题目作为独立子资源，支持 MCP 生成候选、校验、草稿保存和显式启用，文章类型与正文接口保持不变。继续提供普通文章正文内嵌预览，支持私有草稿，仅需 `site:read`，不生成展示稿。继续提供一次调用即可完成的静态资源上传，支持宿主附件以及公开或签名 HTTPS URL，可保存图片、文档、GLB 和 glTF 模型包，单次总量最多 50 MiB。文章展示能力由外部模型通过 `get_article_display_context` 获取规范并直接编写 MDX，再通过 `save_article_display_content` 保存候选，用户在 GrowthTrace 文章页的“产物”面板中审阅和应用。支持 MCP Apps 的客户端可把普通文章、展示稿候选和生成组件预览直接显示在工具结果中。插件展示由 `.codex-plugin/plugin.json` 的 `interface` 配置；各技能的中文名称、简介和默认提示词由自身的 `agents/openai.yaml` 配置。

## 从 GitHub 远程安装

在“插件 → 添加 → 添加插件市场”填写：

| 字段     | 内容                                               |
| -------- | -------------------------------------------------- |
| 来源     | `https://github.com/nmnmtttt/personal-growing.git` |
| Git 引用 | `main`，或留空使用默认分支                         |
| 稀疏路径 | 留空                                               |

该仓库为私有，客户端需要有权读取。添加后从 GrowthTrace 市场安装插件，再完成 MCP OAuth 连接并开启新对话。已添加旧独立分支的用户，应把市场 Git 引用改为 `main`；如果客户端不支持编辑，则移除旧市场来源后重新添加。

仓库根目录市场入口复用本目录源码，插件随 `main` 更新；无需再同步独立插件分支。

## 从本地文件夹安装

通过 ChatGPT 桌面端“插件 → 添加 → 添加插件市场”安装：

1. 下载并校验对应版本的 ZIP 市场包，将其完整解压到本机一个固定文件夹，例如 `C:\Plugins\growthtrace-marketplace`。
2. 确认该文件夹内同时有 `.agents` 与 `plugins`。如果解压工具自动加了一层目录，选择实际包含这两个目录的那一层。
3. 在“添加插件市场”弹窗中填写下表，然后点击“添加市场”。
4. 在新出现的 GrowthTrace 市场中安装 GrowthTrace 插件；首次使用时按提示连接插件提供的 GrowthTrace MCP 并完成 OAuth。
5. 确认插件版本为 0.11.0，再开启新对话测试。

| 弹窗字段 | 本地文件夹安装时填写                                                  |
| -------- | --------------------------------------------------------------------- |
| 来源     | 解压后市场根目录的绝对路径，例如 `C:\Plugins\growthtrace-marketplace` |
| Git 引用 | 留空                                                                  |
| 稀疏路径 | 留空                                                                  |

这里接收的是当前运行 ChatGPT 的电脑上的目录，不是 ZIP 文件、MCP URL、应用详情页链接，也不是单独的 `plugins/growthtrace-operations` 子目录。生成机器上的 Linux 路径不能直接填入另一台 Windows 电脑。

市场包采用标准目录结构：

```text
growthtrace-marketplace/
├── .agents/plugins/marketplace.json
└── plugins/growthtrace-operations/
    ├── .codex-plugin/plugin.json
    ├── .mcp.json
    ├── assets/
    └── skills/
```

marketplace 根目录必须同时包含 `.agents/plugins/marketplace.json` 和 `plugins/`；解压或复制时不要遗漏隐藏目录。仅生成 ZIP 或上传到网站不会自动注册到 Codex。

命令行安装也可以使用同一市场目录。在项目根目录开发时执行：

```bash
codex plugin marketplace add ./integrations/codex
codex plugin list --marketplace growthtrace --available
codex plugin add growthtrace-operations@growthtrace
```

使用下载包时，将第一条命令的路径替换为解压后的 marketplace 根目录。安装后开启新会话加载 Skill 和 MCP；桌面端未刷新时重启客户端。这是本机发现与安装流程，不代表已发布到账号、工作区或公共插件目录。

插件包不包含访问令牌、站长用户 ID、数据库凭证或部署密钥。安装不等于获得数据权限；MCP 只会在 OAuth 批准后返回站点内容。写作项目能力需要 `project:read` / `project:write`，旧授权需重新批准这些 scope。

## 普通文章内嵌预览

定位已保存文章后调用 `create_article_preview(articleId)`，直接预览当前 Markdown 正文；创建并预览时先保存草稿，再使用返回的文章 ID。正文预览不要求 `layout:read`、AI 配置或 MCP 写开关，不创建生成产物，也不会修改或发布文章。

预览会话保留24小时，链接令牌有效30分钟。过期时调用 `refresh_article_preview_url(previewId)`，旧令牌立即失效；刷新仍显示创建时的冻结正文，查看最新修改应重新创建预览。MCP App 只提供阅读和“打开文章”入口；不支持 MCP Apps 的客户端使用返回的预览链接。

## 参数契约与旧连接同步

项目创建使用顶层 `type` 与 `profile.dimensions`，历史项目读取仍可返回旧固定档案。新建时不把旧 `profile.type/worldview/goals` 原样提交。完整示例与其他写入约束见 [输入契约](skills/growthtrace-operations/references/mcp-input-contracts.md)。

插件源码更新不代表已有连接已同步。若服务端报 `profile.dimensions` 缺失、旧字段不识别，或工具定义缺少组件预览的 `dependencies`、展示稿提交的 `componentBindings/expectedUpdatedAt/idempotencyKey`，应让宿主重新获取 `tools/list` 并在新会话核对。只有出现权限或令牌错误时才重新授权；不要通过旧工具、私有 API 或删除参数绕过契约。

## MCP 连接与更新

0.11.0 使用 `.mcp.json` 声明 `growthtrace` HTTP 服务，插件清单的 `mcpServers` 指向该文件，七项技能仍由 `skills` 声明，并通过 `Interactive` 能力标记声明交互界面。安装端需支持插件内远程 MCP 与 OAuth；授权在客户端发起，服务端权限检查仍然生效。`upload_static_asset` 使用 `asset:write`，由服务端完成来源下载、校验、存储和登记；旧的 `prepare_image_upload` 与 `complete_image_upload` 仅保留为兼容入口。文章展示稿提交需要 `article:write` 与 `layout:read`，结果只保存为待审阅候选。

`create_generated_component_preview` 通过 `ui://growthtrace/generated-component-preview/v1.html` 关联 MCP App 资源。在支持 MCP Apps 的 Codex 客户端中，工具成功后会内嵌现有的只读隔离预览页；不支持 UI 的客户端继续读取 JSON 文本结果中的 `previewUrl` 和 `reviewUrl`。内嵌界面不提供批准或 finalize 操作，站长仍需进入登录审核页确认。

0.5.1 的 `.app.json` 误用了插件详情页 ID，导致连接器查询返回 `404 Connector not found`。本版本移除该映射，不再依赖注册应用目录中的 ID。不要将 `chatgpt.com/plugins/` 后面的 ID 直接填入 `.app.json`；后续如需关联注册应用，必须先验证它实际可访问的连接器 ID。

已有 Git 市场用户先刷新市场（CLI 为 `codex plugin marketplace upgrade growthtrace`），再更新或重新安装 `growthtrace-operations@growthtrace`，确认版本 0.11.0 后开启新会话并完成插件 MCP 的 OAuth。此前手动创建的同名应用不会被插件自动删除，也不保证复用其 OAuth 授权。包版本与 MCP 服务端版本各自维护，修复插件连接不需要更改服务端的版本字符串。

仓库仍不附带 `.app.json`。如需把插件关联到 Developer Mode 注册应用，必须先取得并验证可访问的 `plugin_asdk_app_...` 技术 ID，再把映射加入插件清单；不要写入占位 ID，也不要复用插件详情页 ID。

测试时确认插件版本、图标、五条示例提示词和七个技能入口已经更新，再在新对话中测试草稿保存、文章展示候选与只读内容检查等场景。应用连接的授权页还必须实际包含文章展示所需的 `article:write`、`layout:read`，以及项目任务所需的 `project:read`、`project:write`。
