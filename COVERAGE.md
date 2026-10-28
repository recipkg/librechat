# 来源与范围

- [LibreChat](https://github.com/LibreChat-AI/LibreChat/tree/8d3da9bbb220a0a96be9330a2e1cbd385fb786d5)，固定提交 `8d3da9bbb220a0a96be9330a2e1cbd385fb786d5`。根 LICENSE 声明 MIT；第三方组件、模型和资产许可须分别核对。
- 已阅读 README、`api/server/index.js` 路由挂载、`api/server/routes/messages.js` 查询/创建/更新部分及 `api/server/routes/__tests__/messages-content-edit.spec.js`、`messages-get.spec.js` 部分。相关测试文件使用模拟，未运行上游测试或应用。
- 本轮 `node --check api/server/routes/messages.js` 与 `node --check api/server/routes/__tests__/messages-content-edit.spec.js` 均通过；这只是语法检查，不证明依赖可用或行为正确。
- 模型/Agent 编排、认证和租户、查询游标、搜索、上传、MCP、代码运行、数据库、权限、安全、完整 UI 与部署尚未完成提取。未从配方独立重建。
