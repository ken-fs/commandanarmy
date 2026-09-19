# Command An Army Wiki

Roblox《Command An Army》的粉丝 wiki 与攻略站。

**线上**：https://commandanarmy.cc · **部署**：Cloudflare Workers（static assets）· **模板**：[AnvilWiki](https://github.com/PNGTRID/AnvilWiki)（MIT）

---

## 技术栈

| 层 | 选型 |
| --- | --- |
| 框架 | Astro 7（`output: 'static'`，零 JS 优先，Lighthouse 4×100） |
| 内容 | Content Collections + MDX，Zod 构建时硬校验（`src/content.config.ts`） |
| 样式 | Tailwind CSS 3 + CSS 变量主题（品牌色 `#b91c1c`） |
| 部署 | Cloudflare Workers static assets，Git 集成自动构建 |
| 包管理 | pnpm 11（需 Node ≥ 22.13，仓库 `.nvmrc` = 22） |

## 目录结构（三层分离，改动前先读）

```
src/pages, src/components, src/lib   → 框架层（fork-once，不逐游戏改）
src/config, src/locales, globals.css → 配置层（游戏标识/主题/文案）
src/content/wiki/<locale>/<category> → 内容层（文章 MDX，随游戏更新）
```

详细规范见 [`AGENTS.md`](./AGENTS.md) 与 [`docs/content-format.md`](./docs/content-format.md)。

## 本地开发

```bash
pnpm install
pnpm dev            # http://localhost:4321

# 生产构建（SITE_URL 是构建时变量，必须带上）
SITE_URL=https://commandanarmy.cc PUBLIC_GA_ID=G-C57DVL3WBH pnpm build
pnpm preview        # 预览 dist/
```

## 内容工作流

文章路径即 URL：`src/content/wiki/en/<category>/<slug>.mdx` → `/<category>/<slug>/`

**分类（`src/config/navigation.ts`）**：`codes` · `guides` · `units`

**写文章硬规则**（构建时 Zod 校验，不过就 build 失败）：

- `title` ≤ 80 字符；`description` 40–165 字符
- `category` 必须是 navigation 里的 key
- 正文从 H2 起（H1 由 frontmatter title 渲染）
- 内链必须带尾斜杠（`/units/all-units/`）
- 每篇正文 ≥ 3 条内链
- 不确定的数据不写；创作者口播的数值标记为 creator-reported

**常用命令**：

```bash
pnpm check-content      # 内容 lint（frontmatter/内链/长度）
pnpm check-links        # 全站内链审计
pnpm gen-covers         # 用标题+品牌色生成封面并写入 frontmatter
pnpm new-post           # 交互式新建文章
pnpm submit-indexnow -- --site https://commandanarmy.cc   # 推送 URL 给 Bing/Yandex
```

**数据来源纪律**：codes 用 ≥2 个新鲜源交叉验证（Pocket Tactics / GameRant / Sportskeeda / GamesRadar / UrGameTips），单位与战术用官方游戏描述 + 多个创作者视频共识；来源写进 frontmatter 的 `codes[].source` 或正文。

## 部署（关键：部署 = commit + push）

**Cloudflare Workers Builds（Git 集成）**：push 到 `main` → 自动构建 → 部署。**本地 `wrangler deploy` 只是临时生效，会被下一次 Git 构建覆盖——任何改动必须 commit + push。**

- Build command（dashboard 里配置）：`SITE_URL=https://commandanarmy.cc PUBLIC_GA_ID=G-C57DVL3WBH pnpm run build`
- Deploy command：`npx wrangler deploy`
- 输出目录：`dist/`（由 `wrangler.jsonc` 的 `assets.directory` 指定）

### 踩过的坑（改配置前必读）

1. **`wrangler.jsonc` 必须在仓库根目录**：wrangler 的配置发现会向上找，父目录若有 `wrangler.jsonc` 会抢占本仓库配置（.toml 会输给父目录的 .jsonc）。本项目用 `wrangler.jsonc`（Workers static assets 格式，非 Pages 的 `pages_build_output_dir`）。
2. **构建时环境变量走 build command**：`SITE_URL` / `PUBLIC_GA_ID` / `INDEXNOW_KEY` 都是构建时读取（`import.meta.env` / `process.env`），dashboard 的运行时 vars 对静态站无效。
3. **IndexNow key 文件已提交**：`public/e701289517f739c7b7566798851dd8bf.txt`（内容=文件名），勿删。
4. **模板同步**：上游为 `upstream` remote（`git fetch upstream && git merge upstream/main`），合并时注意 `wrangler.jsonc` 与构建命令的本地差异。

## 配置速查

| 配置 | 位置 | 当前值 |
| --- | --- | --- |
| 站点标识/域名/社交 | `src/config/site.ts` | Command An Army Wiki · commandanarmy.cc |
| 主题色 | `src/styles/globals.css` | `#b91c1c`（4 个变量） |
| 导航/分类 | `src/config/navigation.ts` | codes / guides / units |
| 首页模块 | `src/locales/en.json` → `home` | hero/start/explore/faq |
| GA4 | 构建命令 `PUBLIC_GA_ID` | G-C57DVL3WBH（同意门控） |
| IndexNow | `public/<key>.txt` | e7012895…（已提交） |
| 广告位 | `PUBLIC_ADSENSE_*` / `PUBLIC_ADSTERRA_*` | 未启用（有流量再接） |

## 运营链接

| 项 | 地址 |
| --- | --- |
| GSC 属性 | `sc-domain:commandanarmy.cc`（服务账号 gsc-bot@ken-seo-tools 已授权） |
| Cloudflare | Workers & Pages → commandanarmy |
| 验收 | 本地 `node ~/Desktop/david/Ship/scripts/verify.mjs`（含本站页面体检） |
| 上游模板 | https://github.com/PNGTRID/AnvilWiki |

## 内容清单（2026-09-19）

| 分类 | 页面 |
| --- | --- |
| codes | all-codes（7 有效码 ×5 源验证 + 5 过期码） |
| units | all-units（6 兵种/进化树/Ascension）· spartan · lancer · samurai |
| guides | best-units-tier-list · beginner-guide · evolution-guide · strategy-guide |

## 许可

站点代码基于 [AnvilWiki](https://github.com/PNGTRID/AnvilWiki)（MIT）。《Command An Army》游戏内容与素材版权归 Fight, Fight, Fight! 与 Roblox Corporation 所有；本站为粉丝站，无官方关联。
