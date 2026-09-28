# 2026-09-28 · 锁定构建依赖版本

## 决定

`requirements.txt` 由 `mkdocs-material>=9.5` 改为**锁死**：

```
mkdocs==1.6.1
mkdocs-material==9.7.7
```

用户在三个方案里选了锁死：①不锁、②锁范围 `>=9.7,<10`、③锁死。
理由：本站几乎不需要新功能；脚注排版（`docs/stylesheets/extra.css`）
依赖 Material 的内部 CSS（`.footnote>ol>li` 的 margin、`.footnote-backref`），
Material 一升级就可能悄悄改样式（见 `2026-07-27-footnote-typesetting-fixes.md`）。
锁死后线上构建和本地测过的完全一致。

锁的是本地 `.venv` 实装、且 Cloudflare（`PYTHON_VERSION=3.11`）实测构建通过的版本
（commit `3f36c16`，推送约 200 秒后线上生效）。

## ⚠️ 锁版本可能带来的故障：症状 → 定位

锁死的代价是**版本会慢慢变老**。如果哪天出现下面这些情况，先怀疑是锁版本造成的：

| 症状 | 可能原因 |
|------|---------|
| Cloudflare 构建日志在 `pip install` 阶段报错（`No matching distribution`、编译失败、依赖冲突） | Cloudflare 的构建镜像换了 Python 版本，老版本的包装不上或不兼容 |
| 构建报错 `ModuleNotFoundError` / `ImportError`，或提到 `pkg_resources`、`setuptools` | 新 Python 移除了老包依赖的标准库或工具 |
| 推送后线上一直不更新（curl 看还是旧内容） | 构建失败了，去 Cloudflare 面板看部署日志 |
| 本地 `.venv` 重建后 `mkdocs build` 失败 | 同上，本地 Python 升级（本机已是 3.14）|
| 需要某个新的 Material 功能或安全修复 | 版本太老，主动升级 |

**定位第一步**：Cloudflare 面板 → Workers → reformed-reading → 部署日志，看报错出在 `pip install` 还是 `mkdocs build`。
如果报错里出现 `mkdocs` / `mkdocs-material` / Python 版本字样，基本就是这里。

## 手动升级步骤

1. 查可用版本：`.venv/bin/pip index versions mkdocs-material`
2. 在本地 venv 里装新版本：`.venv/bin/pip install 'mkdocs-material==<新版本>'`
   （mkdocs 本身一般跟着 Material 的要求走；⚠️ Material 声明 `mkdocs<2`，MkDocs 2.0 不兼容，别升）
3. `.venv/bin/mkdocs build --strict`，然后 `.venv/bin/mkdocs serve`，
   **重点看脚注区**（挑一章脚注 10 条以上的，如第 9 章），在手机窄屏宽度下检查：
   - 两位数序号没有被裁
   - 条目之间没有多出的空行
4. 没问题就把 `requirements.txt` 改成新版本（`.venv/bin/pip freeze | grep -i mkdocs` 看实装版本），
   push 后用 curl 查线上确认更新
5. 在 `logs/` 记一笔升级了什么、为什么

如果是 Cloudflare 的 Python 版本变了，也可以先试着把 Cloudflare 构建变量 `PYTHON_VERSION` 固定回 3.11，
作为临时止血，再按上面的步骤从容升级。
