# Typora 主题 CSS 修改指南

> 本指南基于我们定制完成的 `new-theme.css`（dogs-choice 极地蓝版本）编写，
> 所有示例选择器都与主题文件一一对应，改完保存后刷新 Typora 即可生效。
> 文中所有以 `new- 主题` 注释标记的区块，就是本次定制的全部内容，方便你快速定位。

---

## 目录

1. [准备工作：文件位置与生效方法](#一准备工作)
2. [如何找到要改的元素：开发者工具](#二如何找到要改的元素)
3. [CSS 三分钟速成（够用版）](#三css-三分钟速成)
4. [主题文件结构地图](#四主题文件结构地图)
5. [常用修改速查（附完整代码）](#五常用修改速查)
6. [典型的坑与排查方法](#六典型的坑与排查方法)
7. [改坏了怎么办](#七改坏了怎么办)

---

## 一、准备工作

### 1.1 主题文件放在哪里

| 系统 | 路径 |
|------|------|
| Windows | `C:\Users\<你的用户名>\AppData\Roaming\Typora\themes\` |
| macOS | `~/Library/Application Support/typora/themes/` |
| Linux | `~/.config/Typora/themes/` |

找不到路径时：打开 Typora → **文件 → 偏好设置 → 外观 → 打开主题文件夹**，直接跳转。

把 `new-theme.css` 放进该文件夹（如需图标等资源，连同资源文件夹一起放）。

### 1.2 让修改生效

1. 保存 CSS 文件；
2. 在 Typora 中按 **Ctrl + Shift + R**（macOS：Cmd + Shift + R）刷新界面 —— 这是最快的方式；
3. 如果没反应：**文件 → 偏好设置 → 外观**，先切换到别的主题再切换回来。

> 注意：CSS 文件名就是主题名。想保留原版又想试新版，可以复制一份改名为
> `new-theme-v2.css`，重启 Typora 后主题列表里会出现两个主题。

### 1.3 动手前先备份

每次大改前复制一份：

```
new-theme.css  →  new-theme.css.bak（或按日期 new-theme-20260904.css）
```

改崩了直接用备份覆盖回来，零损失。

---

## 二、如何找到要改的元素

这是自己改 CSS 最重要的技能，学会它，任何元素都能自己定位。

### 2.1 打开开发者工具

1. **文件 → 偏好设置 → 通用 → 勾选「调试模式」**（只需设置一次）；
2. 在编辑器空白处 **右键 → 检查元素**，或按 **Ctrl + Shift + I**。

### 2.2 定位元素三步法

1. 点击开发者工具左上角的 **箭头图标**（或按 Ctrl + Shift + C）；
2. 在正文里点击你想改的元素（比如某个代码块）；
3. 右侧 **Styles（样式）面板** 会列出作用于它的所有 CSS 规则：
   - 上面的规则优先级高，**被划掉的属性**表示被别的规则覆盖了；
   - 每条规则右上角写着它来自哪个文件（`theme.css`、`base.css`、`*.user.css`）。

### 2.3 即时预览修改

在 Styles 面板里直接改数值、勾选/取消勾选某条规则，页面立刻变化。
**满意了再把同样的改动写进 CSS 文件**，避免反复保存刷新。

### 2.4 模拟悬停/聚焦状态

想改 hover 效果时：Styles 面板点 **`:hov`** → 勾选 **`:hover`**，
元素就会一直处于悬停状态，方便观察。

### 2.5 查看伪元素（圆点、竖条都在这里）

圆点、竖条这类效果是 `::before` / `::after` 伪元素画的，
在 Elements 面板里选中宿主元素（如 `pre.md-fences`）后，
Styles 面板会出现 `pre.md-fences::before` 这样的条目，点它就能看到圆点的样式。

---

## 三、CSS 三分钟速成

### 3.1 规则结构

```css
选择器 {
    属性: 值;    /* 每条声明以分号结尾 */
}
```

### 3.2 常用选择器

| 写法 | 含义 | 本主题示例 |
|------|------|-----------|
| `pre` | 所有 pre 标签 | `pre` 代码块 |
| `.md-toc` | class 为 md-toc 的元素 | 目录卡片 |
| `#write` | id 为 write 的元素 | 正文区域 |
| `A B` | A 里面的 B（所有后代） | `#write h2` |
| `A > B` | A 的直接子元素 B | `#write>h2.md-focus` |
| `:hover` | 鼠标悬停时 | `#write>h4:hover` |
| `:empty` | 元素没有内容时 | `pre.md-meta-block:empty` |
| `::before` | 元素内容前插入的伪元素 | 圆点、竖条都靠它 |
| `::after` | 元素内容后插入的伪元素 | YAML 空占位提示 |

### 3.3 优先级（谁说了算）

从高到低：

```
!important  >  行内样式 style=""  >  #id  >  .class/伪类  >  标签
```

同优先级时，**写在文件后面的覆盖前面的**。
加载顺序：Typora 自带 `base.css` 先加载 → 主题 css 后加载 → 最后还可能加载
`base.user.css` / `主题名.user.css`（用户自定义层）。

**实用结论**：主题里想让某个样式在任何情况下都不被覆盖，就加 `!important`
（本主题的标题竖条已全部加上，别摘掉）。

### 3.4 单位（重点，容易踩坑）

| 单位 | 含义 | 注意 |
|------|------|------|
| `px` | 固定像素 | 最直观 |
| `rem` | 相对根字号（`html { font-size: 16px }` → 1rem = 16px） | 改根字号全局缩放 |
| `em` | **相对当前元素自己的字号** | ⚠️ 伪元素上用 em 时，是相对伪元素继承/设置的字号，不是宿主的 |

> ⚠️ **真实踩过的坑**：竖条规则里 `height: 1.12em`，如果同一规则里写了
> `font-size: 0px`，em 就按 0 计算 → 竖条高度变成 0px → 直接隐形。
> 所以竖条规则里必须写 `font-size: inherit`（继承标题字号）。

### 3.5 伪元素铁律

`::before` / `::after` 没有 `content` 属性就完全不显示。
本主题圆点 = `content: ""` + 宽高 + 背景色 + `box-shadow` 复制出另外两颗。

---

## 四、主题文件结构地图

打开 `new-theme.css`，按顺序是这样几大块（搜注释即可跳转）：

```
:root { ... }                        ← 全部颜色/圆角变量（第 1~21 行）
html / body / #write                 ← 根字号、正文字体、正文宽度
h1 ~ h6                              ← 标题字号颜色 + 左侧竖条（!important 保护）
img / mark / blockquote              ← 图片边框、高亮、引用块卡片
pre.md-fences                        ← 代码块卡片 + mac 三圆点 + mermaid 面板
code                                 ← 行内代码
.md-toc                              ← 目录卡片 + 圆点 + 弹出目录条
#write pre.md-meta-block             ← YAML frontmatter 卡片 + 圆点 + 空占位提示
table                                ← 表格斑马纹
.mathjax-block                       ← 公式块卡片
.side-bar / .outline-item / ...      ← 侧边栏（大纲、文件树）
@media print                         ← 打印样式（一般不用动）
```

> 修改建议：**只改值，别删规则、别动结构**；改完顺手在行尾加一句注释说明改了什么，
> 例如 `/* 改成绿色 */`，以后回看一目了然。

---

## 五、常用修改速查

以下每条都给出了"要改哪个规则 + 完整代码"，直接搜注释定位。

### 5.1 卡片圆角（一处改，全部生效）

主题第 20 行左右：

```css
:root {
    --card-border-radius: 0px;   /* 改成 6px、8px 即全部卡片变圆角 */
}
```

YAML、目录、代码块、mermaid、引用块、公式卡的圆角全部引用这个变量。

表格如需同步圆角，在文件末尾追加：

```css
table {
    border-radius: var(--card-border-radius);
    overflow: hidden;
}
```

### 5.2 配色体系

全在 `:root` 里，常用的几个：

```css
:root {
    --card-bg-color: #e8f1fa;            /* 卡片底色 */
    --card-border-color: #b8d0e8;        /* 卡片描边 */
    --header-lr-border-color: #2c5f8a;   /* 标题竖条 + h1 两侧色块 + 卡片描边 */
    --codeblock-bg-color: #eaf2fb;       /* 代码块底色 */
    --code-inline-bg-color: #dbe9f8;     /* 行内代码底色 */
    --blockquote-bg-color: #e8f1fa;      /* 引用块底色 */
    --side-bar-bg-color: #1f3a5f;        /* 侧边栏底色 */
}
```

改完刷新即可；数字是 `#红红绿绿蓝蓝` 十六进制，可从取色器工具获取。

### 5.3 mac 三圆点（YAML / 目录 / 代码块 / mermaid 共用一套逻辑）

圆点由一条 `::before` 规则画出，以代码块为例（搜 `pre.md-fences::before`）：

```css
pre.md-fences::before {
    content: "";
    display: block;
    width: 11px;                 /* 圆点直径，三处 11px 联动 */
    height: 11px;
    border-radius: 50%;          /* 正圆 */
    background-color: #ff5f56;   /* 第一颗的颜色 */
    box-shadow: 18px 0 0 #ffbd2e, 36px 0 0 #27c93f;
    /*             ↑黄点：水平偏移18px    ↑绿点：水平偏移36px */
    margin: 2px 0 10px 16px;     /* 上2 右0 下10 左16 —— 控制圆点位置 */
}
```

**改位置**：只动 `margin`。上边距加大 → 圆点下移；左边距加大 → 圆点右移。
（当前主题统一标准：卡片顶部 padding 12px + 圆点 margin-top 2px = 距顶 14px；
圆点与内容的间距 = margin-bottom 10px。想整体微调就同步改这几处。）

**改颜色**：`background-color` 是第一颗（左），`box-shadow` 里第二、三个值是另外两颗。
比如换成莫兰迪配色：`#f28b82 / #fdd663 / #81c995`。

**改大小**：`width`、`height` 一起改（保持相等），同时把 box-shadow 里的
`18px / 36px` 按比例调整（建议 = 直径 + 7px 的倍数），三颗才不重叠。

⚠️ YAML 和 TOC 的圆点是各自独立的规则（搜 `md-meta-block::before` 和
`.md-toc::before`），改位置/颜色时三处要一起改才能保持统一。

### 5.4 标题左侧竖条（h2~h6）

搜 `左侧竖条`，两条规则（常驻 + hover/聚焦保护），常用可调项：

```css
#write h2:before, ... {
    width: .3rem !important;      /* 竖条粗细：0.2rem 更细，0.4rem 更粗 */
    margin-right: .5rem !important; /* 竖条与文字的间距 */
    height: 1.12em !important;    /* 竖条高度（标题行高的80%） */
    background: var(--header-lr-border-color) !important; /* 颜色 */
}
```

> ⚠️ 千万不要在这条规则里加 `font-size: 0px` —— 会让 `1.12em` 按字号 0 计算，
> 竖条高度归零直接隐形（这就是之前"竖线全消失"事故的根因）。

h1 两侧的深蓝色块用 `--header-lr-border-color` 变量，与竖条同色，一处改两处变。

### 5.5 YAML 空占位提示（文案/颜色/字号）

搜 `md-meta-block:empty::after`：

```css
#write pre.md-meta-block:empty::after {
    content: "YAML Front Matter（内容以 --- 开头和结尾）";  /* 随便改文案 */
    color: #9aa5b1;      /* 灰蓝色，想更淡就改浅一点 */
    font-size: 85%;
}
```

想完全不显示提示：把这条规则里的 `content` 改成 `content: ""`。
（注意：不要动同文件里 `:empty::before` 的强制圆点规则，那是防止占位符错乱的保护层。）

### 5.6 目录卡片 / 弹出条

- 卡片内边距：搜 `md-toc`，`padding: 12px 16px 14px;`（上 右 下 左）；
- 弹出目录条（点击 TOC 出现的"目录/删除"横条）：搜 `md-toc-tooltip`，
  当前已设为与卡片同宽对齐（`left: 0; right: 0;`）。

### 5.7 代码块

```css
pre.md-fences {
    background-color: var(--codeblock-bg-color);  /* 或直接写色值 */
    padding: 12px 4px 9px 4px;   /* 顶部留出圆点空间，别小于 12px */
}
.CodeMirror-line {
    font-size: 15px;             /* 代码字号 */
}
```

mermaid 图的白色衬底：搜 `md-diagram-panel`（`padding: 8px 12px` 控制图与圆点的距离）。

### 5.8 表格

搜 `table` / `thead`：

```css
thead { background: #dbe9f8; }        /* 表头底色 */
tbody tr:nth-child(even) { background: #edf4fc; }  /* 斑马纹偶数行 */
```

### 5.9 正文字体与字号

```css
html {
    font-size: 16px;    /* 根字号：主题内 rem/em 大量基于它，改 1px 全局缩放 */
}
body {
    font-family: "Noto Sans SC Medium", "HanChanJinShuSong", Helvetica, Arial, sans-serif;
    /* 字体栈从左到右依次尝试：装了用第一个，没装落到下一个 */
}
```

**写字体栈的正确姿势**：`"英文字体", "中文字体", 通用族` ——
英文/数字命中前一个，中文回退到后一个，最后必须留 `sans-serif`（无衬线）或
`serif`（衬线）兜底。字体名带空格要加引号。
若显示成宋体而这不是你想要的，说明栈里的字体都没装，换成系统一定有的：
`"微软雅黑", "Microsoft YaHei", sans-serif`。

### 5.10 侧边栏

文件树字体字号间距（搜 `文件树`）：

```css
#file-library-tree .file-node-content,
#file-library-list .file-node-content {
    font-family: "Segoe UI", "微软雅黑", "Microsoft YaHei", sans-serif;
    font-size: 13px;
    padding: 2px 8px;      /* 每行的上下/左右内边距 */
    line-height: 1.5;
}
```

大纲里的行内代码颜色（深色侧边栏专用，搜 `outline-label code`）。

### 5.11 其他小件

```css
img { border: 1px solid var(--img-border-color); }   /* 图片描边，改 0 去掉 */
mark { background: var(--mark-bg-color); }           /* ==高亮== 底色 */
strong { color: #2c5f8a; }                            /* 粗体颜色（如主题有此规则） */
a { color: #3a7ca8; }                                 /* 链接颜色 */
```

---

## 六、典型的坑与排查方法

### 6.1 改了不生效？

按顺序检查：

1. 保存了吗？按 **Ctrl + Shift + R** 刷新了吗？
2. 改的是 themes 文件夹里正在用的那份文件吗？（偏好设置里看当前主题名）
3. CSS 语法错了：漏了分号、括号不配对 —— 浏览器会**跳过出错的那条规则**，
   后面规则不受影响。用编辑器的括号高亮或搜索 `{` `}` 数量核对；
4. 选择器写错了：拿开发者工具对照真实类名（Typora 内部类名和直觉经常不同）。

### 6.2 样式被别的规则盖掉了

开发者工具 Styles 面板里**被划掉的属性**就是被覆盖的。
解决：把你的选择器写得更具体（加前缀，如 `#write`），或加 `!important`。
覆盖可能来自三处：`base.css`（Typora 自带）、主题 css、`*.user.css`（用户层，
**优先级最高**，注意检查自己 themes 文件夹里有没有这个文件）。

### 6.3 em 单位 + font-size: 0 的隐形坑

伪元素里 `height: 1.5em` 这类写法，em 按**伪元素自己的字号**算。
如果同一条规则里为了隐藏文字写了 `font-size: 0px`，高度会一起变成 0。
需要隐藏伪元素文字时，改用 `content: ""`（内容置空）而不是 font-size: 0。

### 6.4 占位伪元素错乱（[TOC]、空 YAML 显示异常）—— 通用修法

Typora 的空占位提示（`[TOC]`、YAML 提示）是画在 `::before/::after` 上的，
定位经常与主题卡片样式冲突"跑出卡片"。以下片段**可贴到任何主题末尾**：

```css
/* ===== 修复空 TOC 占位溢出（通用） ===== */
#write .md-toc { position: relative; }
#write .md-toc .md-toc-content { margin: 0; min-height: 1.4em; }
#write .md-toc .md-toc-content::before,
#write .md-toc .md-toc-content::after {
    position: static !important;
    float: none !important;
    display: inline;
    font-size: 1em;
}

/* ===== 修复空 YAML 占位错乱（通用，无圆点主题也能用） ===== */
#write pre.md-meta-block:empty::before,
#write pre.md-meta-block:empty::after {
    display: none !important;                 /* 先隐藏 Typora 画错的占位 */
}
#write pre.md-meta-block:empty::after {
    display: inline !important;               /* 再显示自己的提示文字 */
    content: "YAML Front Matter（内容以 --- 开头和结尾）";
    color: #9aa5b1;
    font-size: 85%;
    position: static !important;
    float: none !important;
    white-space: normal;
}
#write pre.md-meta-block::after {
    position: static !important;
    float: none !important;
    white-space: normal;
}
#write pre.md-meta-block:empty { min-height: 2.5em; }
```

提示文案与颜色随改；某主题想完全不显示，删掉"再显示自己的提示文字"那段即可。

### 6.5 伪元素不见了怎么排查

按顺序核对（对应圆点/竖条/提示）：

1. 有 `content` 吗？（没有必不显示）
2. `display` 是 `block/inline-block` 吗？（默认 inline，宽高无效）
3. `width/height` 大于 0 吗？
4. 用了 em 单位时，伪元素自身 `font-size` 是不是 0？（见 6.3）
5. 是不是被更高优先级规则覆盖？（开发者工具看划线）
6. 是不是被相邻元素盖住了？（检查宿主的 padding 是否为 0、有无绝对定位遮挡）

---

## 七、改坏了怎么办

1. **有备份** → 直接用备份覆盖；
2. **没备份** → 从上一轮能正常使用的版本重新对照修改
   （本主题每一处定制都有 `new- 主题` 注释标记，方便逐块恢复）；
3. **只是想撤销某一条** → 删掉你加的规则、或把值改回注释里记录的原值；
4. **彻底乱套** → 删掉整个主题文件重新放置一份干净的 `new-theme.css`。

**三条好习惯**：

- 每次只改一类东西，刷新确认没问题再继续；
- 改过的行尾写注释（改了什么、原值是多少）；
- 大改前备份，文件名带日期。

---

*本指南配套主题：new-theme.css（dogs-choice 极地蓝定制版），定制于 2026-09-04。*
