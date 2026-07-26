# reformed-reading（硬核读书）

古典改革宗译作的自建网站「**硬核读书**」——把 `reformed-translation` 项目翻译的书籍，做成可检索、可阅读的**纯文本**静态网站。

🔗 **线上地址：https://reformedreadingchinese.com** （亦可 https://reformed-reading.bohanzh6.workers.dev）
📦 源码：https://github.com/DZBohan/reformed-reading

## 技术栈

- **MkDocs + Material 主题**：章节树导航、全文搜索（中英）、深浅色、移动端适配
- **托管：Cloudflare Workers（静态资源）**——`wrangler.toml` 里 `[assets] directory = "./site"`，全球 CDN、自动 HTTPS
- **源码：GitHub**，Cloudflare 连仓库自动构建部署
- **域名**：reformedreadingchinese.com（Cloudflare Registrar）

## 目录结构

```
reformed-reading/
├── README.md            # 本文件
├── logs/                # 运行日志
├── mkdocs.yml           # 站点配置（站名/导航/主题/搜索/site_url）
├── wrangler.toml        # Cloudflare Workers 部署配置（把 ./site 当静态资源）
├── requirements.txt     # 构建依赖（mkdocs-material）
├── .gitignore           # 忽略 site/ .venv/
└── docs/                # 站点内容（markdown）
    ├── index.md         # 首页
    ├── copyright.md     # 版权/授权
    └── books/
        └── magnalia-dei/
            ├── index.md # 书籍首页（简介 + 24 章目录表）
            └── NN.md    # 各章（从 reformed-translation 译文填入）
```

## 内容来源与流程（全自动）

```
reformed-translation/books/<book>/zh/NN.md   (译文，逐章)
        │  复制到本仓库
        ▼
reformed-reading/docs/books/<book>/NN.md      (+ CC 页脚)
        │  更新 index.md 目录表状态 + mkdocs.yml 导航
        │  git push → GitHub
        ▼
Cloudflare 自动构建（pip install + mkdocs build）→ npx wrangler deploy
        ▼
几分钟内自动上线 https://reformedreadingchinese.com
```

每章页面：中文正文 + 页脚（原著出处 + CC BY-NC-SA）。

## 本地预览 / 构建

依赖已装在 `.venv/`（用系统 python3 建的 venv，非 uv）。

```bash
.venv/bin/mkdocs serve      # 本地预览 http://127.0.0.1:8000
.venv/bin/mkdocs build      # 构建静态站 → ./site
```

## Cloudflare 部署设置（Workers · 已配好）

连仓库时填（已完成，供参考/重连用）：

| 项 | 值 |
|----|----|
| Production branch | `main` |
| Build command | `pip install -r requirements.txt && mkdocs build` |
| Deploy command | `npx wrangler deploy` |
| Build variable | `PYTHON_VERSION = 3.11` |

`wrangler.toml` 指定 `./site` 为静态资源；push 到 main 即自动构建部署。

## 版权

源本为公有领域（Bavinck 荷文原著 1909）；译文为原创作品，以 **CC BY-NC-SA** 授权。现代英译本仍有版权，本站不使用，均自公有领域原文译出。详见 `docs/copyright.md`。

## 状态

- **已上线**，绑定自有域名 reformedreadingchinese.com。
- Magnalia Dei：**24 章已上线 1–5 章**（至高的善 / 认识神 / 普遍启示 / 普遍启示的价值 / 特殊启示·方式），其余待译。
- 翻译在 `reformed-translation` 项目进行；每译几章 push 一次，网站自动更新。
