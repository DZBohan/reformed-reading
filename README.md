# reformed-reading

古典改革宗译作的自建网站「硬核读书」——把 `reformed-translation` 项目翻译的书籍，做成可检索、可下载的**纯文本**静态网站。

## 技术栈

- **MkDocs + Material 主题**：为文档/书库而生，自带章节树导航、全文搜索、深浅色、移动端适配
- **托管：Cloudflare Pages**（免费、全球 CDN、自动 HTTPS、push 自动部署）
- **源码：GitHub**（版本管理），Cloudflare Pages 连仓库自动构建
- 域名：可选，先用免费 `xxx.pages.dev`，将来再绑自有域名

## 目录结构

```
reformed-reading/
├── README.md            # 本文件
├── logs/                # 运行日志
├── mkdocs.yml           # 站点配置（导航/主题/搜索）
├── requirements.txt     # 构建依赖（mkdocs-material 等）
├── .gitignore
├── overrides/           # 主题自定义（可选）
└── docs/                # 站点内容（markdown）
    ├── index.md         # 首页
    ├── about.md         # 关于/宗旨/版权
    └── books/
        └── magnalia-dei/
            ├── index.md # 书籍首页（简介 + 章节目录）
            └── NN.md    # 各章（从译文填入）
```

## 内容来源与流程

```
reformed-translation/books/<book>/zh/NN.md   (译文)
        │  复制/软链
        ▼
reformed-reading/docs/books/<book>/NN.md
        │  git push → GitHub
        ▼
Cloudflare Pages 自动构建 (mkdocs build) → 部署 → xxx.pages.dev
```

每章页面结构（模板，硬核极简·纯文本）：
- 中文正文（译文）
- 页脚：原著出处 + 译文授权（CC BY-NC-SA）

## 本地预览 / 构建（待讲道批处理跑完再装依赖）

```bash
# 装依赖（用 uv 建独立环境，避免污染系统 py3.14）
uv venv && uv pip install -r requirements.txt
# 本地预览
mkdocs serve      # http://127.0.0.1:8000
# 构建静态站
mkdocs build      # 产出 ./site
```

## Cloudflare Pages 部署设置（在其控制台填）

| 项 | 值 |
|----|----|
| Framework preset | None |
| Build command | `pip install -r requirements.txt && mkdocs build` |
| Build output directory | `site` |
| 环境变量 | `PYTHON_VERSION=3.11`（Cloudflare 构建环境用） |

绑 GitHub 仓库后，push 即自动部署。

## 版权

源本为公有领域（如 Bavinck 荷文原著 1909）；译文为原创作品，建议以 **CC BY-NC-SA** 授权：欢迎非商业转载、注明出处、相同方式共享。每本书页脚标注原著出处与译者。

## 状态

- 2026-07-25：项目创建，站点骨架 + MkDocs 配置就绪；依赖安装与首次构建待讲道批处理完成后进行。
