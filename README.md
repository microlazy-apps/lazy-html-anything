# lazy-html-anything

懒猫微服上的 [HTML Anything](https://github.com/nexu-io/html-anything) 包装版 —— 把 Markdown / CSV / JSON / SQL / 纯文本，通过内置的 Claude Code 一键变成可发布的单页 HTML，再一键导出微信 / X / 知乎 / `.html` / `.png`。

## 安装

在懒猫微服开发者中心搜索 **HTML Anything** 安装；或从本仓库 [Releases](https://github.com/microlazy-apps/lazy-html-anything/releases) 下载 `.lpk`：

```sh
lzc-cli lpk install cloud.lazycat.app.lazy-html-anything-X.Y.Z.lpk
```

## 首次配置

本包装版**不用部署参数**，所有 CLI 配置在 webshell 里按 CLI 自己的方式做。

1. 装好后访问 `https://html-anything.{你的微服域名}/shell/`（末尾斜杠必须有）
2. 看到 xterm 终端，提示符在 `/lzcapp/var/persist`（这是容器里的 `$HOME`）
3. 选一条登录路径：

   **A. OAuth（推荐给 Claude Pro / Max 订阅）**
   ```sh
   claude /login
   ```
   按提示打开浏览器完成 OAuth，凭证落到 `~/.claude/.credentials.json`。

   **B. API key（apiKeyHelper 方式，热加载）**
   ```sh
   echo 'sk-ant-...' > ~/.claude/api-key.txt
   chmod 600 ~/.claude/api-key.txt
   cat > ~/.claude/settings.json <<'EOF'
   {"apiKeyHelper": "cat /lzcapp/var/persist/.claude/api-key.txt"}
   EOF
   ```
   Claude Code 每次 spawn 时跑 helper 拿 key，**改 `api-key.txt` 立即生效**，不用重启。

4. 验证：`claude -p "hi"` 应该返回一条短回复。
5. 回主 UI `https://html-anything.{你的微服域名}/`，顶栏选 Claude Code，粘内容 + 选模板 + ⌘+Enter。

## 用法

1. 顶栏「Agent」自动检测到内置的 **Claude Code** —— 选中
2. 左侧粘贴 Markdown / CSV / TSV / JSON / SQL / 文本
3. 中间从 75 个模板里挑一个（按 mode/scenario 过滤）
4. ⌘+Enter，右侧 iframe 实时流式渲染
5. 顶栏一键导出：微信（juice 内联 CSS）/ X（2× PNG）/ 知乎（LaTeX 占位符）/ `.html` / `.png`

## 与上游的差异

上游 `nexu-io/html-anything` 是**本地优先**工具，会自动检测你本机已经登录的 8 种 Coding Agent CLI（Claude Code / Cursor / Codex / Gemini / Copilot / OpenCode / Qwen / Aider）并直接复用其登录态。**容器内无法读取你本机的 CLI 会话**，所以本包装版做了三个折中：

- **只内置 Claude Code**（其他 CLI 暂未打包）
- **额外跑一个 ttyd webshell** 在 `/shell/`，让你按 CLI 自己的登录方式配置 —— `claude /login` 也好、`apiKeyHelper` 也好、直接 `nano ~/.claude/settings.json` 也好，wrapper 完全不介入
- **`HOME=/lzcapp/var/persist`**，让 `~/.claude/` 自动落到持久化卷，凭证升级 / 重启 / 重装都保留

## 数据持久化

所有状态写到 `/lzcapp/var/persist/`（容器里的 `$HOME`）：

- `.claude/` —— Claude Code 会话状态、settings、credentials
- `.html-anything/skills/` —— 通过 Marketplace 安装的第三方 skill 包
- `.html-anything/{vercel,cloudflare-pages}.json` —— Vercel / Cloudflare Pages 部署凭证（如果用一键部署功能）

升级或重启都会保留。

## 已知约束

- **不支持移动端**（iOS / Android）：上游 UI 是为桌面三栏布局设计的，竖屏无法使用；lpk metadata 显式标 `unsupported_platforms: [ios, android]`。
- **容器内以 root 运行**：因为 `--permission-mode bypassPermissions` 默认拒绝 root，Dockerfile 设了 `IS_SANDBOX=1` 让 Claude Code 跳过这层检查（懒猫沙箱本身已经隔离）。
- **`/shell/` 末尾斜杠**：lazycat 反代对子路径要求 `location: /shell/`（带斜杠），不带斜杠会 308 → 404。

## 上游 & License

- 上游项目：[nexu-io/html-anything](https://github.com/nexu-io/html-anything)，Apache-2.0
- 本仓库（懒猫包装）：[microlazy-apps/lazy-html-anything](https://github.com/microlazy-apps/lazy-html-anything)，Apache-2.0
