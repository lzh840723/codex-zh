# Codex‑zhx — Chinese Bridge for OpenAI Codex CLI

简体中文 | English

## 简介 · Overview
Codex‑zhx 是一个基于终端的轻量插件（TUI 代理），把 OpenAI Codex CLI 的交互界面与“英语↔中文”的翻译无缝结合：
- 左侧原样渲染 Codex TUI；右/下侧显示中文实时/回放译文。
- 键盘输入100%透传给 Codex，不吞字符、不卡回车。
- 零侵入：不修改 Codex；失败时降级为英文直出。

核心设计与执行计划见 `plans/codex-zhx-design.md` 与 `plans/codex-zhx-execution-plan.md`；贡献者指南见 `AGENTS.md`。

## 安装 · Install
首选 Homebrew（建议在发布后使用 Tap；此处以 your-org 为示例）：
```bash
brew tap your-org/tap
brew install codex-zhx
```
从源码构建（需 Rust stable）：
```bash
cargo build --release
# 可执行文件：target/release/codex-zhx
```

## 快速开始 · Quick Start
使用本地/云端的 RUN_ID 恢复会话并开启搜索：
```bash
codex-zhx resume -m gpt-5-codex-mini <RUN_ID> --search
```
- 默认显示中文实时 Pane；可在配置中切换为“仅回放”。
- 若你不想改变模型提示，去掉 `-m gpt-5-codex-mini` 即可。

## 快捷键 · Key Bindings
- F9：显示/隐藏中文 Pane
- F10：实时/仅回放 模式切换
- F11：仅翻译“最近 N 行”（默认 600）
- Alt+S：导出当前会话中文回放（Markdown）
- Alt+R：切换翻译引擎（auto / bing / deepl / libre）
- Alt+M：快捷切换模型（如 gpt‑5‑codex‑mini）

## 配置 · Configuration
用户配置文件：`~/.config/codex-zhx/config.toml`
```toml
model = "gpt-5-codex-mini"   # 默认模型
engine = "auto"               # auto|bing|deepl|libre
live = true                   # true=实时中文 Pane，false=仅回放
recent_window = 600           # 仅翻译最近 N 行窗口
```
也可通过环境变量覆盖：`TRANS_ENGINE`、`LIBRE_URL` 等。

## 日志与导出 · Logs & Export
- 英文：`~/Library/Logs/codex-zhx/english.log`
- 中文：`~/Library/Logs/codex-zhx/zh.log`
- 导出：`codex-zhx --export zh.md`（回放译文）

## 排错 · Troubleshooting
- 看不到中文：检查网络/翻译引擎；切换到 `engine=bing` 或 `libre`（并设置 `LIBRE_URL`）。
- 输入被吞/需重复回车：请确保使用本工具的 TUI（非脚本管道）；不要将 stdout/stderr 重定向到同一 TTY。
- 颜色/光标错乱：属于原始 TUI 控制序列；本工具使用 vte 屏幕模型渲染以避免干扰，请升级到最新版本。

## 贡献 · Contributing
- 代码风格、测试与提交流程参见 `AGENTS.md`。
- 变更需附使用说明与前后对比截图/转录。

---

## Overview (English)
Codex‑zhx is a terminal UI proxy that augments the OpenAI Codex CLI with live Chinese translation while preserving the native TUI:
- Left pane renders Codex exactly; right/bottom pane shows live or playback Chinese.
- Keystrokes are fully forwarded—no eaten characters, no double‑Enter.
- Zero‑intrusive: does not modify Codex; gracefully falls back to English.

## Install
Preferred via Homebrew Tap:
```bash
brew tap your-org/tap
brew install codex-zhx
```
Build from source:
```bash
cargo build --release
```

## Usage
```bash
codex-zhx resume -m gpt-5-codex-mini <RUN_ID> --search
```

## Keys
F9 toggle CN pane · F10 live/playback · F11 recent‑window only · Alt+S export · Alt+R switch engine · Alt+M switch model.

## Config
See `~/.config/codex-zhx/config.toml` (same options as above). Env overrides are supported.

## Troubleshooting
Switch engines if rate‑limited; avoid mixing translated output with the same TTY; file an issue with a short transcript if rendering glitches.

## Links
- Design: `plans/codex-zhx-design.md`
- Execution Plan: `plans/codex-zhx-execution-plan.md`
- Contributor Guide: `AGENTS.md`
