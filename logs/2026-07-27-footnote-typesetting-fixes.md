# 2026-07-27 · 脚注排版连环问题排查与修复

网站上线第 7–10 章后，用户在手机/电脑上陆续发现脚注区一系列排版问题。本文记录**全部根因与修法**，供日后避坑。相关自定义样式在 `docs/stylesheets/extra.css`（已挂进 `mkdocs.yml` 的 `extra_css`）。

## 背景：本站脚注机制

- Markdown `footnotes` 扩展：正文 `[^label]` + 章末 `[^label]: 说明`；渲染为 `<div class="footnote"><ol><li>…`。
- **编号规则**：脚注按**定义顺序**编号，`<ol>` 也按定义顺序渲染。→ 定义顺序若 ≠ 正文出现顺序，正文上标数字就跳（1,3,5,2…）。
- Material 主题对脚注的关键默认（`main.*.min.css`）：
  - `.footnote{font-size:.64rem}`；`.footnote>ol{margin-left:0}`（比普通列表更窄！）
  - `.footnote>ol>li{margin-left:1.25em}` ← **序号就摆在这段左边距里**
  - `.footnote-backref{display:inline-block;opacity:0;…}` ← **返回箭头默认隐形，却占约 0.8rem 布局空间**，悬停才现（手机不可用）

## 五类问题与根因、修法

### 1. 条目间"多一道空白"（主因，最隐蔽）
- **现象**：某些脚注条目之间多出一道空行；手机竖屏明显、横屏/电脑时有时无；HTML 逐字节验证结构完全一致。
- **根因**：`.footnote-backref` 隐形(opacity:0)但 `display:inline-block` 占约 0.8rem。窄屏时它被挤到下一行，成了"隐形元素独占的空行"。跟每条文字长度有关 → 只有特定条目、特定屏宽中招。
- **修法**：`.footnote-backref{display:none!important}` 抹掉（默认本就隐形、手机用不上，脚注跳转功能不受影响）。**通用根治**，一次治所有章。

### 2. 两位数序号被裁（10/11/12 → 0/1/2）
- **现象**：仅 ch9（12 条脚注、唯一出现两位数的章）在手机窄屏上十位被裁。
- **根因**：脚注 `ol` 的 margin 被 Material 归 0，序号仅靠 `li{margin-left:1.25em}`，装不下两位数。
- **修法**：`.footnote>ol>li{margin-left:2.2em!important}` 加宽序号空间。**⚠️ 切勿把 li 的 margin-left 归零**（曾犯，导致序号无处安放被裁）。

### 3. 全角符号触发字体回退，顶高行 → 视觉空隙
- **现象**：ch9 手机端某条下方空白；ch10 第8条（含全角等号「＝」）附近空白。
- **根因**：全角斜杠「／」(U+FF0F)、全角等号「＝」(U+FF1D) 等罕见全角符号触发浏览器字体回退，把所在行行高顶高。
- **修法**：并列经文用**顿号「、」**（不用「／」）；「＝」改为「即」。全书扫除。

### 4. 同一脚注被正文引用两次 → 双返回箭头 ↩↩
- **现象**：ch10 诗51、ch9 诗36 各被引两次，生成 `#fnref` 与 `#fnref2` 双 backref，窄屏换行时出空白。
- **修法**：拆成两条独立脚注（如 `hb-ps51` / `hb-ps51b`，内容可相同），各带单一 backref。全书仅此两处。

### 5. 文内标注跳号（正文上标非 1,2,3…）
- **现象**：ch2、ch9 正文上标跳号（因脚注定义顺序 ≠ 正文出现顺序）。
- **修法**：脚本把章末脚注定义**按正文首次出现顺序重排**。全 10 章检查后仅这两章需改。

## 诊断方法论（有用的套路）

- **逐字节比对 HTML**：先确认是不是结构性成因（空 `<p>`、`<br>`、多余 `</ol>`、条间空白字符）。本次全部排除 → 锁定渲染层/CSS。
- **"空隙跟着谁走"**：重排脚注后观察空隙是否跟着某条移动 → 定位到具体条目（双箭头、全角符号）。
- **全书唯一性佐证**：某触发物全书唯一（唯一双引用、唯一「／」、唯一「＝」）且正好在报告位置 → 坐实。
- **读 Material 源 CSS**：`site/assets/stylesheets/main.*.min.css` 里搜 `.footnote` / `ol li` 的 margin、`footnote-backref` 的 opacity/display，才发现隐形占位这个总根因。
- **部署确认**：`.gitignore` 忽略 `site/`，Cloudflare 自己跑 `mkdocs build`（Build command: `pip install -r requirements.txt && mkdocs build`）。改完 push 后用 `curl` 轮询线上，确认是新版再让用户刷新，避免看缓存误判。

## 涉及提交

`4373397`(重排) `632bdeb`(ch10诗51拆) `c0fda8c`(ch9诗36拆) `3cc2cff`(ch9 ／→、) `52fe9ee`(加CSS) `615ff9b`→`48ed25f`→`4289fc0`(CSS序号空间几次修订) `d41b650`(backref display:none + ＝→即) `f667a41`(ch2/ch9 重排修跳号)

## 结论 / 经验

- 写脚注时守三条（已入 `reformed-translation` 的 CONVENTIONS）：**①一条脚注只在正文引用一次；②脚注定义按正文出现顺序排；③并列经文用顿号「、」，禁用全角「／」「＝」等罕见全角符号**。
- 站点侧 `docs/stylesheets/extra.css` 已通用兜底 backref 占位与序号空间；勿删、勿覆盖 li 的 margin-left。
