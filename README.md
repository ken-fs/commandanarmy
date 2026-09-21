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
4. **模板同步**：上游为 `upstream` remote（`git fetch upstream && git merge upstream/main`），合并时注意以下 fork 差异。

### 本 fork 对模板的本地修复（upstream 合并时保留）

| 文件 | 修复 | 原因 |
| --- | --- | --- |
| `.github/actions/gates/action.yml` | check-config 步骤补 `SITE_URL` env | 本 fork 删了 wrangler.toml，check-config 没地方读域名 |
| `scripts/write-indexnow-key.ts` | 部署标记兼容 `WORKERS_CI_COMMIT_SHA`；标记写入独立于 INDEXNOW_KEY | Workers Builds（非 Pages）环境变量名不同；没配 INDEXNOW_KEY 时也要写标记 |
| `tests/` 删除 4 个文件 | apply-template / agents-consistency / handbook / redirects 测试 | 模板自维护测试（drift 守卫/landing 文档中心），定制 fork 不适用 |
| `tests/url·tags·seo·content-utils·routing-flags` | ja → `'ja' as Locale`；demo 游戏名 → Command An Army | 本站单语言（en）+ 非 demo 游戏名 |
| `src/locales/en.json` | `home.hero.videoId: ""` | HomePage 引用该字段（空 = 不渲染 trailer 区） |

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

## 内容清单（2026-09-21 扩容后：20 篇）

### 第一批（9 篇，2026-09-19）

codes/all-codes · guides/beginner-guide · guides/best-units-tier-list · guides/evolution-guide · guides/strategy-guide · units/all-units · units/lancer · units/samurai · units/spartan

### 第二批（11 篇，2026-09-21）

| 分类 | 页面 | 数据来源 |
| --- | --- | --- |
| guides | **unit-counters** —— 盾/矛/骑/弓四方克制三角 + 「悬停看 block damage」判定法 | 7 份创作者视频字幕（79K 字符） |
| guides | **star-limit-and-lineup** —— 星数上限系统（每 5 级 +1 星，进化会抬高星耗） | 同上 |
| guides | **update-2** —— Ranked / Guilds / Rapier / Reaper | 同上 |
| guides | best-units-tier-list **重写** —— 补 Update 2 与 1.5 补丁后的 meta | 同上 |
| units | archer（Archer→Longbowman→Flame Archer）· shieldman（→Sentinel→Imperial Wall）· halberdier（Spearman→Spear Militia→Halberdier→Imperial Halberdier）· scout-rider（→Cavalryman→Lancer）· berserker · immortal · mace · reaper | 同上 |

**两条别处查不到的机制**（本站独有）：

1. **星数上限**：每个单位按星级占额度，上限随账号等级增长（每 5 级 +1 星，等级封顶 50）。进化会抬高星耗 —— 所以必须**先升等级再进化**，否则会把单位进化出自己阵容的可负担范围。
2. **判断单位能否破盾**：游戏从不显示克制表。**鼠标悬停单位看有没有 `block damage`** —— 有就是破盾单位。这是唯一可靠的 in-game 判定法。

**关于 Mace 的争议**：两个创作者一个给 S 级（破盾近战 DPS），一个说「全游戏最差冲击步兵之一」。双方对机制描述完全一致，分歧在「盾牌阵容出现频率够不够高」。站上**两方观点都写了**，没有假装没有分歧。

## 数据来源纪律（本类站通用）

1. **Roblox badge API** = 权威实体名单（每个解锁有独立 badge，创建日期即上线日期）：
   `https://badges.roblox.com/v1/universes/<id>/badges?limit=100&sortOrder=Asc`（翻页带 cursor）
2. **官方美术素材**：`/v1/games/<id>/media` 返回**完整媒体库**（比 `multiget/thumbnails` 强 —— 后者只给 1 张，media 给了 6 张）；badge 图标走 `/v1/assets?assetIds=<iconImageId>`（`/v1/badges/icons` 返回空）
3. **创作者数据**：`yt-dlp --write-auto-subs --sub-langs en --sub-format vtt` 直接抓自动字幕（**不需要 whisper 转录**，7 个视频 79K 字符几分钟搞定）
4. **Fandom**：网页端 403，走 MediaWiki API（`api.php?action=parse&prop=wikitext`）

## 竞品格局（2026-09-21 复查）

| 竞品 | 状态 |
| --- | --- |
| `command-an-army.fandom.com` | 12 页，标注「launch 后第一次更新」，**过时 6 个月** |
| `commandanarmy-wiki.wiki` | ⚠️ **AI 幻觉壳站** —— 单位页零真实数值，还把另一款游戏（Master of Command 的普鲁士/英国）内容混进来了 |
| `command-an-army-wiki.wiki` | Astro 站，措辞谨慎但**数据贫弱** —— 明确写「没有受控的伤害/血量/速度对比」，只列 9 个单位（缺全部进化形态） |
| `command-an-army.wiki` | 20K+ 字符多语言程序化站 |
| 大媒体 | progameguides / games.gg / sportskeeda / techwiser 都有 tier list |

**结论**：竞品在**结构**上不弱，但**数据**普遍贫弱。本站的护城河是「真实创作者数据 + 明确标注分歧 + 机制解释」。

## 许可

站点代码基于 [AnvilWiki](https://github.com/PNGTRID/AnvilWiki)（MIT）。《Command An Army》游戏内容与素材版权归 Fight, Fight, Fight! 与 Roblox Corporation 所有；本站为粉丝站，无官方关联。
