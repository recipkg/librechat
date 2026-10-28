# Agent 管理与执行入口：固定源码合同

来源为 LibreChat 固定提交 `8d3da9bbb220a0a96be9330a2e1cbd385fb786d5` 的 [Agent 路由](https://github.com/LibreChat-AI/LibreChat/blob/8d3da9bbb220a0a96be9330a2e1cbd385fb786d5/api/server/routes/agents/index.js)、[管理路由](https://github.com/LibreChat-AI/LibreChat/blob/8d3da9bbb220a0a96be9330a2e1cbd385fb786d5/api/server/routes/agents/management.js)、[管理数据合同](https://github.com/LibreChat-AI/LibreChat/blob/8d3da9bbb220a0a96be9330a2e1cbd385fb786d5/packages/api/src/agents/management.ts)、[机器认证](https://github.com/LibreChat-AI/LibreChat/blob/8d3da9bbb220a0a96be9330a2e1cbd385fb786d5/packages/api/src/middleware/management.ts)、`packages/api/src/agents/{creates,reads,updates,deletion}.ts` 及 [聊天路由](https://github.com/LibreChat-AI/LibreChat/blob/8d3da9bbb220a0a96be9330a2e1cbd385fb786d5/api/server/routes/agents/chat.js)。以下是源码观察；尚未启动该提交、执行测试或重建软件。

## 分离的认证与权限

- 管理入口为 `/api/agents/v1/agents`，在一般 JWT 聊天路由之前挂载，采用单独的机器身份认证。源码要求配置启用的 OIDC 发行方及受众，或带所需 scope 的 access token 配置；从 `Authorization: Bearer ...` 验证令牌，并将有效客户端绑定到指定租户和用户。验证客户端 ID、过期时间和 subject；禁用、缺失或不匹配的绑定不会变成其他用户。该接口并非自动接受浏览器 JWT 或 Agent 执行 API key。
- 认证缺失、无效、配置未启用或绑定不成立返回 HTTP 401 的 `{error: "Unauthorized"}`；被绑定账户正处于删除流程的分支返回 409 和 `ACCOUNT_DELETION_IN_PROGRESS`。这些认证响应与下面管理处理器的错误信封不同。真实 OIDC 配置、签名密钥轮换、租户中间件及其他失败路径仍须实测。
- 管理列表及详情要求 Agent 的 `USE` 权限，并按已绑定用户的租户查询。没有管理全部 Agent 能力时，列表只查具有 `EDIT` 权限的资源，详情同样需要 `EDIT`；有该管理能力时可绕过单资源权限。创建要求 `USE` 和 `CREATE`；更新还需目标资源的 `EDIT`，删除还需 `DELETE`，单资源权限在有管理全部 Agent 能力时可被绕过。不存在于该租户的目标按未找到处理，不能依 ID 跨租户读取或修改。
- 浏览器聊天链路与管理链路分开：`POST /api/agents/chat`、`POST /api/agents/chat/:endpoint` 和 `POST /api/agents/chat/resume` 经一般 JWT 路由，进一步经过消息 ID 验证、内容过滤、Agent 权限和会话权限等中间件。具体 Agent 资源校验请求 `VIEW`；存在 `skipAgentCheck` 等例外，不能把它简化为所有请求都执行完全相同的判断。`resume` 在其他检查前尝试从待处理动作恢复原有图配置。不能拿管理 API 的 `EDIT` 规则直接替代聊天规则。

## 管理 API 数据与错误

- `POST /api/agents/v1/agents` 创建；`GET` 同路径列出；`GET`、`PATCH`、`DELETE /api/agents/v1/agents/{id}` 分别读取、修改和删除。创建输入需提供字符串 `provider`、`model`，支持的配置另包括 `name`、`instructions`、工具和技能等。未知顶层字段会被严格拒绝，不能由调用方指定内部 `id`、`tenantId`、`versions` 或 credentials；`instructionsPrompt` 是 Builder 专用字段，不被该管理 API 接受或返回。
- 更新只接受允许的字段；`name`、`description`、`instructions`、`model` 不能设为 `null`，而 `avatar: null` 和 `code_environment_id: null` 是验证器接受的清除值。对未列出的字段、空值、版本历史和局部更新的实际持久化效果仍需逐项验证，不能把所有 `null` 等同于删除或保留。
- 列表 `limit` 默认为 20，必须为 1 到 100 的整数；`cursor` 是有长度和结构校验的不透明游标。响应包含 `object: "list"`、`data`、`first_id`、`last_id`、`has_more` 和 `after`；有更多结果时应返回下一游标，无更多结果时 `after` 为 `null`。响应投影只暴露允许的配置和公开元数据，包括 `id`、`provider`、`model`、`version`、`createdAt`、`updatedAt`，而不是返回数据库内部全部字段。成功删除返回 `{id, deleted: true}`。
- 管理处理器的标准错误包装为 `{error: {code, message, ...}}`：`invalid_request` 400、`not_found` 404、`permission_denied` 403、`conflict` 409、`internal_error` 500；验证错误可附字段路径细节，其他内部错误不应暴露原始异常。该映射不覆盖认证中间件和所有上传限流分支；创建/更新对底层控制器状态还做各自映射，不可假定所有底层 409 都原样透传。

## 未完成

Agent 模型调用、工具执行、流式事件、重连、暂停审批、子 Agent、MCP、文件、版本及并发状态尚未形成完整合同。路由存在不代表依赖服务已安装、工具已授权或用户可执行；固定源码运行、浏览器和独立重建均未验证。源码中的管理合同测试使用单元/模拟场景，不能当作真实服务通过。
