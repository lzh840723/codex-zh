# Codex‑zhx 设计方案（插件级中文桥接）

## 1. 背景与目标
- 为 Codex 交互式 TUI 提供稳定的“英↔中桥接”，解决脚本方案的输入吞字符、ANSI 干扰、回放延迟等问题。
- 不修改 Codex 本体；以“代理/插件”方式运行，保持本地与云端 `resume --search` 等场景一致体验。

## 2. 语言与技术选型
- 语言：Rust（确定采用）。理由：终端/PTY 生态成熟（portable-pty、vte、ratatui）、单二进制易发布、无 GC 停顿、跨架构打包便捷。
- 主要依赖：`tokio`、`portable-pty`、`vte`、`ratatui`、`crossterm`、`regex`、`reqwest`、`serde`、`anyhow`、`thiserror`。

## 3. 约束与非目标
- 仅做终端内 UI/翻译增强；不实现模型推理或变更 Codex 协议。
- 保持零侵入：失败时可直接透明透传 Codex。

## 4. 总体架构
- 进程与 PTY：主进程创建 PTY，子进程运行 `codex ...`；主进程负责键盘透传、窗口大小同步（SIGWINCH）。
- 渲染：使用 vte 模拟器解析 ANSI/OSC → 屏幕模型；用 ratatui 渲染左侧 Codex 原样 TUI、右/下侧中文 Pane。
- 翻译流水线：从“纯文本滚动缓冲”中增量抽取稳定片段 → 代码块保护 → 分块并发翻译 → 回填到 UI。

## 5. 组件划分（Rust）
- `src/pty.rs`：portable-pty 启动/读写/resize；粘贴模式、特殊键处理。
- `src/term.rs`：vte 模拟器，导出“屏幕模型 + 纯文本滚动缓冲”。
- `src/ui.rs`：ratatui 布局（左右/上下可配置）、刷新节流（60–120ms）。
- `src/text/blocker.rs`：```fenced```/`inline` 代码保护、段落切分与合并。
- `src/translate/`：翻译引擎抽象，失败级联与速率限制。
- `src/logging.rs`：双日志 english.log / zh.log（循环文件）、导出回放稿。
- `src/config.rs`：加载 `~/.config/codex-zhx/config.toml` 与环境变量覆盖。
- `src/main.rs`：CLI、快捷键绑定、状态机驱动。

## 6. 关键 API（示例）
```rust
#[async_trait]
pub trait Translator {
  async fn translate(&self, chunks: Vec<TextChunk>) -> anyhow::Result<Vec<TextChunkOut>>;
}
// 代码块保护 → 分块 → 调度多引擎（auto→bing→deepl→libre）→ 回填
```

## 7. 交互与快捷键
- F9 切换中文 Pane；F10 实时/仅回放；F11 仅翻译“最近 N 行”（默认 600）。
- Alt+S 导出当前会话中文回放；Alt+R 切换翻译引擎；Alt+M 快速切 mini 模型。
- 所有按键默认透传到 Codex，除非与保留快捷键冲突。

## 8. 翻译策略
- 引擎级联：translate‑shell(auto) → bing → deepl → libre（可配置密钥/URL）。
- 分块与节流：每块 2–4 KB，段落优先切分；0.8s 空闲或双换行触发；失败回退英文原文。
- 保护与回填：保留 ```fenced code``` 与 `inline code`；占位后翻译正文，最后回填占位符。

## 9. 配置与启动
- 配置：`~/.config/codex-zhx/config.toml` 示例：
```toml
model = "gpt-5-codex-mini"
engine = "auto"   # auto|bing|deepl|libre
live = true        # 实时译文 Pane
recent_window = 600
```
- 启动示例：`codex-zhx resume -m gpt-5-codex-mini <RUN_ID> --search`。

## 10. 日志与可观测性
- 写入 `~/Library/Logs/codex-zhx/english.log` 与 `zh.log`（滚动大小/保留份数可配）。
- 崩溃前 flush 缓冲；提供 `--export zh.md` 导出本次回放译文。

## 11. 安全与隐私
- 不落地密钥；从环境变量/系统钥匙串读取；日志默认脱敏（URL/Token 屏蔽）。

## 12. 测试策略
- 单元：ANSI/OSC 解析、代码块保护、分块/合并、失败回退。
- 集成：伪 PTY 回放（录制的 Codex 会话）对比中英文产出。
- 端到端：真 PTY 下的输入透传、窗口 Resize、快捷键不中断 Codex。

## 13. 发布与安装（Homebrew 一键方案）
- 发布工具：使用 cargo-dist 产出多架构二进制与 Homebrew formula（私有/公开 tap 皆可）。
- CI 流水线：GitHub Actions 在打 tag 时触发，构建 darwin-arm64 与 darwin-amd64，生成校验和并创建 Release；同时推送 formula 到 tap。
- 安装（用户侧）：
  - `brew tap your-org/tap`
  - `brew install codex-zhx`
- 运行：`codex-zhx resume -m gpt-5-codex-mini <RUN_ID> --search`
- 配置（可选）：`~/.config/codex-zhx/config.toml`，示例
  ```toml
  model = "gpt-5-codex-mini"
  engine = "auto"     # auto|bing|deepl|libre
  live = true          # 启用实时中文 Pane
  recent_window = 600  # 仅翻译最近 N 行
  ```
- 更新与卸载：`brew upgrade codex-zhx` / `brew uninstall codex-zhx`。
- 可选加分：Developer ID 签名与 notarize，减少 Gatekeeper 警告；提供 `gen-completions` 生成 zsh/bash 补全。
