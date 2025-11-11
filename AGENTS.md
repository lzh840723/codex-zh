# Repository Guidelines

## Project Structure & Module Organization
- `src/` — core source for the Codex Chinese bridge (CLI + PTY/UI/translate).
- `scripts/` — helper shell scripts (installation, smoke tests).
- `tests/` — integration and end‑to‑end tests (CLI transcripts, PTY logs).
- `assets/` — screenshots, small fixtures, sample logs.
- `docs/` — design notes and usage examples.

## Build, Test, and Development Commands
- Build (Rust): `cargo build` (release: `cargo build --release`).
- Lint/Format: `cargo fmt --all && cargo clippy -D warnings`.
- Shell hygiene (if scripts present): `shfmt -w scripts && shellcheck scripts/*.sh`.
- Run locally: `target/debug/codex-zhx resume -m gpt-5-codex-mini <RUN_ID> --search`.
- Tests: `cargo test` (integration tests live under `tests/`).

## Coding Style & Naming Conventions
- Rust 2021，4 空格缩进；`snake_case` 文件/模块；`CamelCase` 类型；`snake_case` 函数。
- 函数要小而专注；公共边界偏好显式类型。
- Shell: `#!/usr/bin/env bash` + `set -euo pipefail`，长参数优先，变量需引用。
- 禁止泄露密钥；使用 `TRANS_ENGINE`、`LIBRE_URL` 等环境变量。

## Testing Guidelines
- 单元测试靠近实现（`#[cfg(test)]`）；集成测试在 `tests/`。
- 命名示例：`tests/resume_live_translate.rs`。
- 重点覆盖：PTY I/O、ANSI/OSC 解析、代码块保护、分块与回退。

## Commit & Pull Request Guidelines
- 提交遵循 Conventional Commits：`feat:`/`fix:`/`docs:`/`refactor:`/`test:`/`chore:`。
- 分支命名：`feat/<topic>` 或 `fix/<issue-id>`。
- PR 必含：目的、用户影响、测试计划、前后对比截图/转录，关联 issue。
- 变更集中且可审；CI（build+lint+tests）需通过。

## Security & Configuration Tips
- 不打印 token；错误信息脱敏；提供 `example.env`。
- 日志默认写入 `~/Library/Logs/codex-zh/`（macOS）；避免破坏性操作。
- 同时验证 Terminal.app 与 iTerm2；正确处理窗口大小变更。

## Agent‑Specific Instructions
- 使用小补丁（apply_patch），路径相对；修改行为需同步文档。
- UI/PTY/翻译链路的跨模块改动请拆分 PR，并补足测试。
