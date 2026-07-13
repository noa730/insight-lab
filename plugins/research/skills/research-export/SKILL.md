---
name: research-export
description: Phase 5 of research — convert Markdown report to other formats
---

# Research Export

## Input
- Read `research-output/<project>/report.md` (must be user-approved)
- User's chosen format: PDF / PPTX / HTML

## Process

### PDF Export
Use Python with markdown and weasyprint (or pandoc if available):
```bash
pip install markdown weasyprint
python -c "
import markdown
from weasyprint import HTML
# Convert report.md → report.pdf with embedded images and charts
"
```
Fallback: suggest user install pandoc and run:
```bash
pandoc report.md -o report.pdf --pdf-engine=xelatex -V mainfont='Microsoft YaHei'
```

### PPTX Export
Invoke the `/pptx` skill with report.md content. The skill handles slide generation.

### HTML Export
Convert Markdown to standalone HTML with embedded images:
```bash
pip install markdown
python -c "
import markdown
# Convert with extensions: tables, fenced_code, toc
# Inline CSS for professional styling
# Embed chart AND user-provided images as base64
"
```

HTML export 必须包含以下关键要素：
1. **Mermaid 渲染**：在 `</body>` 前加载 mermaid.js CDN 并初始化：
   ```html
   <script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>
   <script>mermaid.initialize({startOnLoad:true, theme:'neutral', fontFamily:'Microsoft YaHei, SimHei, sans-serif'});</script>
   ```
2. **Mermaid 块预处理**：在 markdown 转换前，将 ` ```mermaid ``` ` 代码块转为 `<div class="mermaid">...</div>`，确保 mermaid.js 能识别并渲染
3. **图片嵌入**：用正则匹配 `![alt](src)` 语法，将 `charts/` 和 `images/` 下的图片转为 base64 内联，使 HTML 完全独立可分发
4. **中文字体**：CSS font-family 优先使用 `Microsoft YaHei, SimHei`

## Output
- Converted file in the project directory
- Update status.md: export → done
