---
name: present-build
description: Phase 4 of present — assemble final interactive HTML presentation
---

# Present Build

## Builder Persona

你是前端工程师。你的任务是将大纲内容、视觉设计和图表素材组装为最终可演示的交互式 HTML。

### 行为准则
1. **单文件优先** — 默认产出单个自包含 `index.html`，所有 CSS、JS、图表代码、SVG 全部内联。仅当存在无法内联的资源（大量高清图片、外部字体文件等）时，才使用文件夹结构
2. **层级导航** — 实现页面树的钻入/返回机制：
   - 主线页面按顺序翻页（← →）
   - 可点击元素跳转到子页面/附件页
   - 子页面/附件页有明确的返回按钮，返回来源页面
   - 导航状态清晰：用户始终知道自己在哪一层
3. **演示功能**：
   - 键盘翻页：← → 或 Page Up/Down
   - 页码/进度指示
   - 演讲者备注：按快捷键（如 N）显示/隐藏
4. **技术栈自选** — 根据内容复杂度选择最合适的方案，不限定框架。选择依据：
   - 页面少、层级浅、交互简单 → 纯 HTML/CSS/JS
   - 页面多、层级深、需要嵌套导航 → Reveal.js 或类似框架
   - 大量数据图表交互 → 引入 ECharts CDN
5. **样式精确还原** — 严格按 design.md 的配色、字体、布局、动效实现
6. **中文排版** — 确保中文字体正确渲染，行高适当（1.6-1.8），段间距合理

### 产出结构

默认：
```
dist/
└── index.html    # 单文件，包含所有内容
```

复杂情况：
```
dist/
├── index.html
├── assets/       # 图片、字体等无法内联的资源
└── js/           # 外部图表脚本（仅当内联过大时）
```

## Input
- Read `present-output/<project>/outline.md` — page content, structure, navigation
- Read `present-output/<project>/design.md` — all visual styling
- Read `present-output/<project>/charts/manifest.md` — chart file index
- Read chart files referenced in manifest (PNG images or ECharts JS)

## Architecture: Multi-File HTML

**Always use multi-file architecture.** Single-file HTML causes performance issues and rendering bugs with 10+ pages.

### Output Structure
```
dist/
├── style.css           # shared stylesheet
├── index.html          # cover page (entry point)
├── toc.html            # table of contents
├── 1-1.html            # content pages
├── 1-2.html
├── 1-2-drill.html      # sub-pages (drill-down/attachments)
├── ...
└── images/             # chart PNGs, photos
    ├── chart-2-1.png
    ├── 全景图-01.jpg
    └── ...
```

### Each HTML File
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=1920">
  <title>Page Title</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="page">
    <div class="top-deco"></div>
    <a href="toc.html" class="toc-btn" title="返回目录">&#9776;</a>
    <div class="page-header">...</div>
    <div class="content">...</div>
    <div class="progress-bar">...</div>
    <div class="nav-bar">...</div>
  </div>
  <!-- keyboard nav -->
  <script>...</script>
  <!-- auto-scale (skip for scrollable sub-pages) -->
  <script>
  (function(){
    if(document.body.classList.contains("scrollable")) return;
    function fitScale(){
      var scale=Math.min(window.innerWidth/1920,window.innerHeight/1080);
      document.body.style.transform="scale("+scale+")";
    }
    fitScale();
    window.addEventListener("resize",fitScale);
  })();
  </script>
</body>
</html>
```

## Process

### Step 1: Create style.css

#### 1920x1080 Adaptive Scaling
```css
html {
  width: 100%; height: 100%;
  overflow: hidden; background: #000;
  display: flex; justify-content: center; align-items: center;
}
/* Allow scrollable sub-pages to scroll */
html:has(body.scrollable) {
  overflow: auto; display: block; background: #FFFDF7;
}
body {
  width: 1920px; height: 1080px; overflow: hidden;
  transform-origin: center center; flex-shrink: 0;
}
body.scrollable {
  width: 100%; height: auto; min-height: 100vh;
  overflow-y: auto; position: static;
  transform: none !important;
}
```

#### Typography & Font Size

**字体从 design.md 读取**，不同项目风格不同。常见参考：
| 场景 | 标题字体 | 正文字体 |
|------|---------|---------|
| 公文汇报 | STZhongsong/SimSun（华文中宋） | FangSong/STFangsong（仿宋） |
| 商务演示 | PingFang SC/Microsoft YaHei | PingFang SC/Microsoft YaHei |
| 科技风格 | PingFang SC/Helvetica Neue | PingFang SC/Helvetica Neue |

**字号规范（投影场景通用）：**
- **最小字号 18px** — 低于此在投影上看不清
- Body text: 22-24px
- Card titles: 22-24px
- Page title (h2): 40px
- h1: 52px, h3: 28-30px
- Captions/secondary: 18px
- ECharts internal fonts also need scaling (title 20px, axis 16-20px, legend 16px)
- 如果 design.md 指定了不同字号体系，以 design.md 为准，但仍遵守 18px 最小值

#### Layout Rules
- `.content` uses `display:flex; flex-direction:column` to fill the page
- Chart/image areas use `flex:1; min-height:0` to auto-fill remaining space
- Images use `max-height + object-fit:contain` to prevent overflow
- Content must fill 1920x1080 — no large blank areas

#### Navigation Components
- **Page header**: fixed top, project name + report title, gold separator line. Hidden on cover/QA.
- **Navigation bar**: fixed bottom, prev/next links + chapter name + page number
- **Progress bar**: 3px bar above nav, width = currentPage/totalPages * 100%. Hidden on sub-pages.
- **TOC button**: right-top corner on main-line pages (&#9776; icon), links to toc.html
- **Back button**: `position: fixed` on sub-pages (NOT absolute — must stay visible during scroll)

### Step 2: Create Main-Line Pages

For each page in outline.md's main-line sequence:
1. Create an HTML file (e.g., `1-1.html`)
2. Include page header, top decoration, TOC button
3. Fill content from outline with appropriate font sizes
4. Embed chart images: `<img src="images/chart-X-X.png">`
5. Embed photos where specified in outline
6. Add clickable links to sub-pages (gold styled)
7. Add bottom navigation with correct prev/next links and page number
8. Add keyboard navigation script (← → arrow keys)
9. Add auto-scale script at the end

#### TOC Page
- Each chapter title is a clickable link to the chapter's first page
- Key chapter can be visually highlighted (e.g., gold border)

### Step 3: Create Sub-Pages

For drill-down and attachment pages:
1. Use `<body class="scrollable">` — content may exceed viewport
2. Include back button: `<a href="parent.html" class="back-btn">← 返回</a>`
3. Include Escape key handler for navigation back
4. Do NOT include progress bar or TOC button
5. Auto-scale script will skip these pages automatically

### Step 4: Chart Integration

#### PNG Charts (from markdown-viewer skills)
```html
<div class="chart-container">
  <img src="images/chart-X-X.png" alt="..." style="max-width:100%;max-height:700px;object-fit:contain;">
</div>
```

#### ECharts (only when interactive charts are needed)
- Load CDN only on that specific page: `<script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>`
- Inline the chart JS in a `<script>` block
- Remember to scale internal font sizes (see Font Size Rules)
- ECharts graphic elements (annotation boxes): rect width must be large enough for text

#### Interactive Components (e.g., upgrade/compare)
- Inline the full HTML+CSS+JS directly into the page

### Step 5: Photo Integration

Photos from `images/` directory:
- Use `object-fit: cover` for comparison photos (before/after)
- Use `object-fit: contain` for maps and diagrams
- Always set `max-height` to prevent overflow
- Add captions where appropriate
- Before/after comparison: side by side with overlay labels

### Step 6: Verify

Before returning to orchestrator:
1. All HTML files exist and link correctly (prev/next/drill/back)
2. Page numbers are sequential and total is correct
3. All images referenced in HTML exist in `images/`
4. No font-size below 18px in any file (CSS or inline)
5. Content fills each 1920x1080 page without large blank areas
6. Sub-pages can scroll and back button stays visible
7. Auto-scale works on main pages, skipped on sub-pages

## Common Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| Page content too small | Font sizes < 18px | Minimum 18px everywhere |
| Large blank area at bottom | Content doesn't flex-fill | Use `flex:1; min-height:0` |
| Image overflows page | No max-height constraint | Add `max-height + object-fit:contain` |
| Sub-page can't scroll | html has `overflow:hidden` | Use `html:has(body.scrollable)` override |
| Back button disappears on scroll | `position: absolute` | Use `position: fixed` |
| ECharts annotation box too small | rect width hardcoded | Make width >= text width + padding |
| Deleting a page breaks nav | Orphan links remain | Update all prev/next links + page totals |
| Single-file HTML laggy | Too many DOM elements | Use multi-file architecture |

## Output

Files written to `present-output/<project>/dist/`:
- `style.css` — shared stylesheet
- `*.html` — one file per page
- `images/` — chart PNGs and photos

Return control to orchestrator. Suggest user opens `dist/index.html` in a browser to preview.
