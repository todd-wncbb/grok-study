# Grok Build 项目学习文档

这套文档是给第一次读这个项目的人准备的。目标不是把每一行代码都翻译成中文，而是帮你先建立地图：这个项目是什么、从哪里启动、用户输入怎么进入 agent、工具怎么被调用、文件和权限又在哪里处理。

## 先记住一句话

Grok Build 是一个 Rust 写的终端 AI 编程助手。它的可执行程序叫 `xai-grok-pager`，正式安装后通常以 `grok` 命令出现。代码大致分成四层：

1. `xai-grok-pager-bin`：真正的二进制入口，解析 CLI，决定启动哪种模式。
2. `xai-grok-pager`：终端 TUI，处理键盘、鼠标、滚动区、弹窗和渲染。
3. `xai-grok-shell` + `xai-grok-agent`：agent 运行时，管理认证、模型、会话、prompt、工具循环。
4. `xai-grok-tools` + `xai-grok-workspace`：具体工具、文件系统、权限、git/worktree、workspace RPC。

## 推荐阅读顺序

| 顺序 | 文档 | 适合解决的问题 |
| --- | --- | --- |
| 1 | [01_project_map.md](01_project_map.md) | 这个仓库有哪些目录和 crate？我该先看哪里？ |
| 2 | [02_startup_modes.md](02_startup_modes.md) | `grok` 命令启动后经历了什么？有哪些运行模式？ |
| 3 | [03_tui_architecture.md](03_tui_architecture.md) | TUI 如何处理输入、状态、异步任务和渲染？ |
| 4 | [04_agent_session_runtime.md](04_agent_session_runtime.md) | agent、session、prompt、模型调用的主链路是什么？ |
| 5 | [05_tools_workspace_permissions.md](05_tools_workspace_permissions.md) | 工具如何注册、执行，文件和权限在哪里管？ |
| 6 | [06_learning_path.md](06_learning_path.md) | Rust 新手怎么分阶段读这个项目？ |
| 7 | [07_code_reading_recipes.md](07_code_reading_recipes.md) | 想追某个功能时，用哪些搜索命令和入口？ |
| 8 | [08_glossary.md](08_glossary.md) | 常见项目术语和 Rust 术语是什么意思？ |

## 建议你先跑的命令

这些命令来自项目 README 和 workspace 配置，适合边读边验证：

```sh
cargo check -p xai-grok-pager-bin
cargo run -p xai-grok-pager-bin -- --version
cargo run -p xai-grok-pager-bin -- --help
```

如果你只想读代码，不急着编译，先从这些文件开始：

```text
Cargo.toml
README.md
crates/codegen/xai-grok-pager-bin/src/main.rs
crates/codegen/xai-grok-pager/src/app/mod.rs
crates/codegen/xai-grok-shell/src/agent/app.rs
crates/codegen/xai-grok-shell/src/agent/mvp_agent/acp_agent.rs
crates/codegen/xai-grok-shell/src/session/acp_session_impl/turn.rs
crates/codegen/xai-grok-tools/src/bridge.rs
crates/codegen/xai-grok-workspace/src/workspace_ops.rs
```

## 阅读心法

这个项目很大，不要试图从 `main.rs` 一路跳到每个函数。更好的方法是：先看 `lib.rs` / `mod.rs` 了解模块边界，再找一条具体链路，例如“发送 prompt”或“执行工具”，沿着枚举、trait、channel、handler 一步步追。

你会经常看到这些模式：

- `mod.rs`：目录模块的入口。
- `trait`：定义一种能力，比如文件系统、工具、workspace RPC。
- `Arc` / `Mutex` / `RefCell` / `Rc`：共享状态和单线程 actor 状态。
- `tokio::mpsc` / `oneshot`：异步任务之间传消息。
- `serde`：JSON / YAML / TOML 的序列化和反序列化。
- `clap`：命令行参数解析。

后面的文档会把这些概念放回真实代码里解释。
