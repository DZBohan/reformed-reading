# reformed-reading —— 硬核读书（发布层）

先读 `README.md`，再看 `logs/`。

## 定位

只做**发布**：把 `../reformed-translation/` 译好的章节搬上网站。
翻译、校对、术语、经节编号规则都在上游，**不在这里改译文**——
要改正文，先改上游 `books/<上游书目>/zh/NN_*.md`，再同步过来，两边保持一致
（发布稿 = 上游译文 + 章末 CC 页脚）。

⚠️ 别和 `video-subtitle`（硬核字幕组）混：两个「硬核」是不同的牌子，互不相干。

## ⛔ 不能碰的

- **`docs/stylesheets/extra.css`**：别删，别覆盖脚注 `li` 的 `margin-left`。
  那是给两位数脚注序号留的位置，删了手机上 10+ 号会被裁。
  来龙去脉见 `logs/2026-07-27-footnote-typesetting-fixes.md`。
- **版权**：只发公有领域原著的自译。上游 `work/proofing-en/` 是有版权的出版英译，
  **绝不能**出现在本仓库或网站上。

## 发布一章的步骤

⚠️ 上下游书目目录名**不一样**，别照抄 slug：

| 书 | 上游（reformed-translation） | 本站 |
|----|-----|-----|
| Magnalia Dei | `books/bavinck_wonderful-works/zh/NN_<荷文slug>.md` | `docs/books/magnalia-dei/NN.md` |

1. 按上表复制上游章节到本站对应目录，末尾加 CC 页脚（照已有章节抄）
2. 更新本站书目的 `index.md` 的目录状态（标题改成链接，状态改为「✅ 已译」）
3. 在 `mkdocs.yml` 的 `nav` 里加上这一章
4. 本地 `.venv/bin/mkdocs build --strict` 检查无报错（`site/` 已被 gitignore）
5. push 等于公开发布，须用户同意。push `main` → Cloudflare Workers 自动构建部署。推完用 `curl` 查线上，
   确认已是新版本再让人刷新，免得看到缓存误判

## 构建环境

- `.venv/` 是普通 `python3 -m venv`，不是 uv
- `requirements.txt` **锁死**了 `mkdocs==1.6.1` 与 `mkdocs-material==9.7.7`（2026-09-28 用户定）。
  ⚠️ **构建失败 / 推送后线上不更新时，先怀疑是锁版本老化**（Cloudflare 换了 Python、老包装不上）。
  症状对照、定位方法和手动升级步骤见 `logs/2026-09-28-pin-build-deps.md`。
  升级必须本地先构建，并在手机窄屏宽度下看过脚注排版再改。
- 托管用的是 **Cloudflare Workers 静态资源**（`wrangler.toml`），不是 Pages
