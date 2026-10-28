# LibreChat 配方（提取中）

```toml
format = "recipkg/1"

[package]
name = "librechat"
version = "0.1.0"
kind = "software"
description = "多模型聊天和 Agent 管理平台重建配方草案。"
entry = "BUILD_PROMPT.md"

[dependencies.quality]
git = "https://github.com/recipkg/recipkg.git"
rev = "41e5b55c6612a9a5a10e499b5ace6a1a3fed0f2b"
subdir = "recipes/software-quality"

[dependencies.design]
git = "https://github.com/recipkg/design-directions.git"
rev = "f86b3eca87356a073274248eb48b6a5b6756db59"
```

来源 [LibreChat-AI/LibreChat](https://github.com/LibreChat-AI/LibreChat/tree/8d3da9bbb220a0a96be9330a2e1cbd385fb786d5)，源码提交 `8d3da9bbb220a0a96be9330a2e1cbd385fb786d5`；上游根 LICENSE 标示 MIT。当前配方只完成部分源码观察，未完整提取或重建，见 `COVERAGE.md`。
