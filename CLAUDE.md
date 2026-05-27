# lazy-html-anything

懒猫微服 lpk 包装：[nexu-io/html-anything](https://github.com/nexu-io/html-anything)。

## Lazycat appstore identifiers

- **package id**: `cloud.lazycat.app.lazy-html-anything`
  (v0.0.8 起；v0.0.1-v0.0.7 用过 `cloud.lazycat.app.html-anything`，被第三方占了，403)
- **app_id**: `5633` (recorded 2026-05-27, via bootstrap-app.yml @ v0.0.8)
- **subdomain**: `html-anything` → `https://html-anything.{box-domain}`
- **bootstrap workflow**: when re-running `bootstrap-app.yml` to resubmit a
  fix, pass `app_id=<NUM>` so the workflow skips `/app/create` (which would
  500 on duplicate package).

## 架构（核心差异 vs 上游）

上游是 **本地优先** 工具：Next.js 服务器 `spawn` 用户机器上的 8 种 Coding
Agent CLI（Claude Code / Cursor / Codex / Gemini / Copilot / OpenCode /
Qwen / Aider），复用 CLI 已有的登录会话。在容器内这两个前提都不成立 ——
既没有那些 CLI，也没有用户的 OAuth 会话状态。

为了让上游产品在懒猫上能跑，本包装做了三件事：

1. **只内置 Claude Code**（`npm i -g @anthropic-ai/claude-code`），
   通过 deploy_param 注入 `ANTHROPIC_API_KEY` 让它以 API key 模式工作。
2. **HOME 重定向到 `/lzcapp/var/persist`**，于是 `~/.claude/` 和
   `~/.html-anything/` 都自动落到持久化 bind mount 上，重启 / 升级保留。
3. **Next.js 用 standalone 模式编译**（`NEXT_PRIVATE_STANDALONE=true`），
   image 体积压到 ~400 MB（vs full node_modules ~1 GB）。

```text
/app/next/server.js                   ← Next.js standalone 入口
/app/next/.next/static/               ← 编译后静态资源
/app/next/public/                     ← 公共资源
/app/next/src/lib/templates/skills/   ← 75 个内置 skill（按需复制到镜像）

$HOME/.claude/             ← Claude Code 会话与设置（持久化）
$HOME/.html-anything/      ← Marketplace 安装的 skill 包 + Vercel/CF 凭证
```

`tini` 作为 PID 1 收割 claude-code 派生的子进程（claude 启动后会 fork
`git` / `rg` / `node` 等若干个工具进程）。

## Deploy params

| 参数 | 必填 | 含义 |
|---|---|---|
| `ANTHROPIC_API_KEY` | 实际必填，名义可选 | Claude Code 的 API key；留空时所有 Agent 调用都会失败 |
| `ANTHROPIC_BASE_URL` | 否 | 第三方 Anthropic 中转/代理（如 packy.cc / yourapi.cn） |
| `ANTHROPIC_MODEL` | 否 | 覆盖 Claude Code 的默认模型 |

之所以把 `ANTHROPIC_API_KEY` 标记为 `optional: true`：用户即便不填也想
先看看 UI、试模板，再决定是否要花一个 key。这与
[lazycat-lpk-wrapper.md 中 deploy params 的告诫](https://github.com/microlazy-apps/lazycat-ci)
"`optional: false` blocks lpk-manager start" 一致。

## Vendor 模式与 patches

满足 lazycat-lpk-wrapper 的"复杂项目"判据 #4（"the wrapper needs an
entrypoint shim ... change that lives inside the image filesystem"），
所以走 `vendor/html-anything` git subtree + `patches/0N-*.patch`，
**vendor 永远不修改**。

当前 patch:
- `01-lazycat-entrypoint.patch` 添加 `Dockerfile`、`.dockerignore`、
  `lazycat-entrypoint.sh`

### 改 patch 的流程

```sh
# 把 patch 应用到 vendor 以编辑
git apply patches/01-lazycat-entrypoint.patch -p1 --directory=vendor/html-anything

# 在 vendor/html-anything/ 里改文件，stage 让 git diff 能看到新文件
git add -N vendor/html-anything/<any-new-files>

# 重新生成 patch
git diff --no-color --relative=vendor/html-anything \
  vendor/html-anything/ > patches/01-lazycat-entrypoint.patch

# 复原 vendor（只跟踪 patches/*）
git checkout HEAD -- vendor/html-anything/<modified-files>
rm vendor/html-anything/<new-files-listed-in-patch>

# 验证
git apply --check patches/01-lazycat-entrypoint.patch \
  -p1 --directory=vendor/html-anything
```

## Release flow

参考 [lazycat-lpk-wrapper.md 的 Release flow ordering](https://github.com/microlazy-apps/lazycat-ci)：

1. `git tag v0.0.1 && git push --tags` → `release.yml` 构出 lpk 但
   `publish-appstore` 会失败（应用还没注册）—— 这是预期。
   `publish-release` 成功，lpk 落到 GitHub Release。
2. 从 GitHub Release 下载 lpk → `lzc-cli lpk install` 装到测试盒子上。
3. 浏览器打开 `https://html-anything.{box-domain}/`，
   填好 ANTHROPIC_API_KEY，跑通核心流程，截图替换 `lazycat/screenshots/`。
4. 把真实 UI 截图换成 icon.png（v0.0.1 的 icon 是 banner.png 上
   "OPEN SOURCE" 球的 crop，是 placeholder）。
5. 提交 v0.0.2，运行 `bootstrap-app.yml` 把它注册到 appstore。

## 已知 gotchas

- **Next.js 16 standalone + pnpm workspace**：必须设
  `NEXT_PRIVATE_STANDALONE=true` 而不是修改 next.config.ts（patch 会污染
  upstream）。配合 `outputFileTracingRoot` 默认行为，standalone 输出会
  把 workspace 根的必要文件一并带上。
- **`src/lib/templates/skills/` 在 standalone 输出里会被丢掉**（不在
  `.next/standalone` 的 trace 里），所以 Dockerfile 显式 `COPY` 一份到
  `/app/next/src/lib/templates/`，让 `loader.ts` 里的
  `path.join(process.cwd(), "src/lib/templates/skills")` 能找到。
- **`favicon.ico` 是上游默认的 Next.js 三角图标**（不是 html-anything 的
  品牌标记）；不用作 lazycat icon。
- **Healthcheck 路径**：用 `/api/agents`（GET 返回 JSON，无依赖），
  不用 `/`（会触发 React 服务端渲染，cold start 慢）。
- **`start_period: 120s`**：Next.js + claude-code npm 首次加载在 lazycat
  盒子上要 60–90s。短于 120s 会导致 `unhealthy` 误报。

## 升级上游

```sh
git subtree pull --prefix=vendor/html-anything \
  https://github.com/nexu-io/html-anything.git main --squash

# 验证 patch 还能干净 apply
git apply --check patches/*.patch -p1 --directory=vendor/html-anything
```

若上游改了 `next/next.config.ts`、Next.js 版本、或 templates 路径，
重新生成 patch。
