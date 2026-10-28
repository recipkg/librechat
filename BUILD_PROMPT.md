# LibreChat 多模型聊天与 Agent 管理（提取中）

## 产品目标

为不同用户提供自托管聊天界面，选择模型和提供商、查看会话及消息、管理 Agent 和工具。上游 README 还列出附件、搜索、MCP、技能、子 Agent、管理接口与实验性的代码工作区；这些功能的权限、接口、事件、存储和运行条件均需要继续提取。不要将外部模型、搜索、代码执行器或 MCP 服务当成配方已经实现的依赖。

## 已观察：消息查询与修改子集

固定提交 `api/server/index.js` 在 `/api/messages` 挂载消息路由，`api/server/routes/messages.js` 对路由使用 JWT 鉴权中间件。以下是局部源码及上游测试证据，尚未运行服务或这些测试：

- `GET /api/messages` 支持 `conversationId`、`messageId`、`cursor`、`sortBy`、`sortDirection`、`pageSize`、`search`。正页大小解析失败或非正时默认为 25；可选排序字段是 `endpoint`、`createdAt`、`updatedAt`，无效值回退 `createdAt`；`asc` 为升序，其他为降序。指定会话时读取按当前用户范围过滤，检查所有权后才能返回；对不存在/不属于用户且不符合活动任务例外的会话返回 404。活动任务读取与子 Agent 线程有专门条件，不得误写成无例外的规则。
- 使用 conversationId 与 messageId 查询时返回 `{messages, nextCursor}`，nextCursor 为 null；按会话分页时返回数据库的分页结果和消息。具体游标格式、搜索行为、字段投影与活动任务例外需要进一步提取。
- `POST /api/messages/{conversationId}` 经请求验证、配置和内容过滤后保存当前用户消息。服务器移除客户端提交的追踪引用、`contextMeta` 和若干用户提交标记，再写入服务端值；保存成功返回 201，未保存返回 400。子 Agent 的只读线程可能返回 409；精确前提由验证器和任务状态决定，需继续核实。
- `PUT /api/messages/{conversationId}/{messageId}` 按当前用户和消息 ID 读取，并检查消息属于给定会话；不匹配返回 404。`index` 若存在且不是数字或小于 0，返回 400；不存在或越界、非文本/思考内容部分也返回 400。未指定 index 时更新消息文本及 token 计数；指定时仅更新对应内容部分，保留其他元数据，结构化文本保留 annotations，思考文本可能移除推理标签元数据。成功返回 200；过滤器和只读线程可阻止写入。
- 未直接证明整数索引校验、所有角色/租户权限或多请求并发一致性，不把未观察行为写成保证。`messages-content-edit.spec.js` 使用模拟数据库和鉴权中间件，不能把其断言当真实端到端验收。

## 交互与依赖

会话列表、对话、发送、等待/流式输出、错误恢复、附件与 Agent 配置应形成清楚的任务流程；采用 `design` 及递归 `foundation` 为聊天内容、密度、键盘与无障碍建立可验证方向。`quality` 约束错误、维护和性能；两者不实现模型、认证、存储或 Agent 引擎。OpenHands SDK 属于另一个产品，未证实兼容，不强加为依赖。数据库、搜索、模型提供商、凭据和 MCP/代码执行服务按部署范围列为外部前提或另行提取的真实依赖。

按 `ACCEPTANCE.md` 在隔离实例及可控模拟模型上验收；在完整提取及真实对照前不得宣称兼容上游或已完成重建。
