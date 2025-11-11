# 执行计划（Codex‑zhx 插件）

## 1) 范围与目标
- 交付一个“单二进制”的终端 TUI 代理：左侧原样渲染 Codex，右/下侧实时/回放中文译文。
- 不修改 Codex；对失败场景具备优雅降级（透明透传、英文可用）。

## 2) 可交付物
- 可执行程序 `codex-zhx`（darwin‑arm64/amd64）。
- 配置 `~/.config/codex-zhx/config.toml`（可选）。
- Homebrew 安装通道（tap + formula）。
- 文档与故障排查指南；端到端测试用例与录制转录。

## 3) 前置条件
- Rust toolchain（stable）、cargo、GitHub 仓库与 Actions 权限。
- 本机可运行 Codex CLI，且可获得 RUN_ID 用于本地验证。

## 4) 工作分解（无日程，按依赖顺序）
1. 仓库初始化：crate、LICENSE、README、`rustfmt`/`clippy`、CI（lint/build/test）。
2. PTY 透传：`portable-pty` 启动 `codex ...`，stdin/out/err 透传、SIGWINCH。
3. 终端解析：`vte` 解析 ANSI/OSC → 屏幕模型；导出纯文本滚动缓冲。
4. TUI 渲染：`ratatui` 分栏与刷新节流（60–120ms），快捷键 F9/F10/F11。
5. 翻译流水线：稳定片段抽取 → 代码块保护 → 2–4KB 分块并发 → 失败回退。
6. 引擎抽象：translate‑shell(auto) → bing → deepl → libre（可配置、超时/退避）。
7. 日志与回放：`english.log`/`zh.log` 滚动；`--export zh.md`。
8. 配置与降级：`config.toml` + env 覆盖；无网络/限流时提示并回退英文。
9. 打包发布：`cargo-dist` + GitHub Actions + Homebrew formula。
10. 文档与样例：README、键位表、常见错误；录制一段 resume 会话 GIF/转录。

## 5) 验收标准（Definition of Done）
- 交互稳定：不吞首字符、一次回车；窗口缩放无异常。
- 视觉稳定：左侧 Codex 无抖动；中文 Pane 刷新不扰动输入。
- 翻译稳健：至少两条引擎可用；失败回退英文不空白。
- 端到端：`codex-zhx resume -m gpt-5-codex-mini <RUN_ID> --search` 正常；F9/F10/F11 有效。
- 安装：`brew tap <tap> && brew install codex-zhx` 可用。

## 6) 风险与对策
- PTY 差异（Terminal/iTerm2）：用屏幕模型渲染；仅拦截保留快捷键。
- 限流/失败：引擎级联、超时/退避、逐块回退；可配置备用 API。
- 性能内存：仅翻译“最近 N 行”；节流 0.8s；日志轮转。
- 控制序列变体：扩展清洗规则；保留原文兜底。

## 7) 扩展与后续
- 额外引擎：OpenAI/Azure/DeepL（密钥管理）。
- 交互增强：中文 Pane 搜索/复制、高亮、导出带时间轴 md。

## 8) 任务清单（从 T0 开始）
- [ ] T0: 仓库初始化与基础 CI（rustfmt/clippy/test）。
- [ ] T1: PTY 启动与键盘透传（portable-pty，含 SIGWINCH）。
- [ ] T2: 终端解析（vte）与屏幕模型；导出纯文本滚动缓冲。
- [ ] T3: TUI 分栏渲染（ratatui），60–120ms 刷新与快捷键框架。
- [ ] T4: 文本清洗（CSI/OSC/BS/CR）与“稳定片段”抽取。
- [ ] T5: 代码块保护与占位回填（fenced/inline）。
- [ ] T6: 翻译引擎抽象与级联（auto→bing→deepl→libre），超时/退避。
- [ ] T7: 实时译文 Pane 与仅回放模式切换；“最近 N 行”窗口。
- [ ] T8: 日志写入与导出（english.log/zh.log，--export zh.md）。
- [ ] T9: 配置文件与环境变量覆盖（~/.config/codex-zhx/config.toml）。
- [ ] T10: 端到端验收（resume -m gpt-5-codex-mini <RUN_ID> --search）。
- [ ] T11: 覆盖关键测试（PTY I/O、解析、保护、分块、回退）。
- [ ] T12: 发布流水线（cargo-dist + GitHub Actions + Brew formula）。
- [ ] T13: 文档与故障排查（README、键位表、常见错误）。
