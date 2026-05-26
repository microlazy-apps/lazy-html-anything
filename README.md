# lazy-html-anything

懒猫微服上的 [HTML Anything](https://github.com/nexu-io/html-anything) 包装版 —— 把 Markdown / CSV / JSON / SQL / 纯文本，通过内置的 Claude Code 一键变成可发布的单页 HTML，再一键导出微信 / X / 知乎 / `.html` / `.png`。

## 安装

在懒猫微服开发者中心搜索 **HTML Anything** 安装；或从本仓库 [Releases](https://github.com/microlazy-apps/lazy-html-anything/releases) 下载 `.lpk`：

```sh
lzc-cli lpk install cloud.lazycat.app.html-anything-X.Y.Z.lpk
```

## 配置

安装时会弹出部署参数面板，填两个就能用：

| 参数 | 必填 | 含义 |
|---|---|---|
| `ANTHROPIC_API_KEY` | **是**（实际） | 从 https://console.anthropic.com/settings/keys 申请。若使用第三方 Anthropic 中转/代理则用代理的 key |
| `ANTHROPIC_BASE_URL` | 否 | 例 `https://api.packy.cc`、自建中转 URL 等；留空走官方 `api.anthropic.com` |
| `ANTHROPIC_MODEL` | 否 | 例 `claude-opus-4-7` / `claude-sonnet-4-6` / `claude-haiku-4-5`；留空交由 Claude Code 默认选择 |

填好之后访问 `https://html-anything.{你的微服域名}` 即可。

## 用法

1. 顶栏「Agent」会自动检测到内置的 **Claude Code** —— 直接选中
2. 左侧粘贴 Markdown / CSV / TSV / JSON / SQL / 文本
3. 中间从 75 个模板里挑一个（按 mode/scenario 过滤）
4. ⌘+Enter，右侧 iframe 实时流式渲染
5. 顶栏一键导出：微信（juice 内联 CSS）/ X（2× PNG）/ 知乎（LaTeX 占位符）/ `.html` / `.png`

## 与上游的差异

上游 `nexu-io/html-anything` 是**本地优先**工具，会自动检测你本机已经登录的 8 种 Coding Agent CLI（Claude Code / Cursor / Codex / Gemini / Copilot / OpenCode / Qwen / Aider）并直接复用其登录态。**容器内无法读取你本机的 CLI 会话**，所以本包装版做了一个折中：

- **只内置 Claude Code**（其他 CLI 没法分发）
- 通过 deploy_param 注入 `ANTHROPIC_API_KEY` —— 用 API key 模式跑 Claude Code

如果你已经在用某个 Anthropic 中转（如 packy.cc、yourapi.cn 等），把 `ANTHROPIC_BASE_URL` 也填上即可，调用全部走代理；这种方式通常比直连官方 API 便宜。

## 数据持久化

所有状态写到 `/lzcapp/var/persist/`：

- `.claude/` —— Claude Code 会话状态、设置
- `.html-anything/skills/` —— 通过 Marketplace 安装的第三方 skill 包
- `.html-anything/{vercel,cloudflare-pages}.json` —— Vercel / Cloudflare Pages 部署凭证（如果用一键部署功能）

升级或重启都会保留。

## 上游 & License

- 上游项目：[nexu-io/html-anything](https://github.com/nexu-io/html-anything)，Apache-2.0
- 本仓库（懒猫包装）：[microlazy-apps/lazy-html-anything](https://github.com/microlazy-apps/lazy-html-anything)，Apache-2.0
