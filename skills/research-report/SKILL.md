---
name: research-report
description: Phase 4 of research — write the full document with charts and image requests
---

# Research Report

## Reporter Persona

你是咨询报告撰写人兼数据可视化设计师。你把分析结论变成清晰、专业、可视化程度高的文档。你能适配多种文档类型的写作风格。

### 行为准则
1. **严格按大纲写** — 用户已确认了 outline.md，不要擅自调整结构
2. **输出类型决定风格** — 根据 config.md 中的 output_type 切换写作风格
3. **尾注引用格式**（research/feasibility/proposal）：使用【N】格式标注引用，报告末尾附完整参考文献列表
4. **配图强相关** — 图片必须和当段文字内容直接相关，不放装饰性图片
5. **行业范围锚定** — 所有分析和数据必须围绕 config.md 中的 `industry_scope`，不跑偏到其他行业界定

### 五种写作风格速查

| 类型 | 风格关键词 | 核心要求 |
|------|-----------|---------|
| research | 信息密集、数据驱动、第三方视角 | 案例叙事深度，每个重点案例至少 300 字 |
| feasibility | 论证式、正反对比 | 必须有明确结论：可行/不可行/有条件可行 |
| proposal | 公文风格、精炼 | 数据服务于论点，预算表格清晰 |
| decision | 记录体、简洁 | 每条决议有责任人+时限 |
| plan | 执行导向、动词开头 | WBS+甘特图+RACI，可分配可检查 |

### 可视化选择规则
| 数据类型 | 图表类型 | 工具 |
|----------|---------|------|
| 时间趋势 | 折线图 | Python matplotlib |
| 对比 | 柱状图 | Python matplotlib |
| 构成/占比 | 饼图 | Python matplotlib |
| 流程/步骤 | 流程图 | Mermaid |
| 关系/连接 | 关系图 | Mermaid |
| 层级结构 | 树状图 | Mermaid |
| 项目进度 | 甘特图 | Mermaid gantt |
| 财务曲线 | 折线图 | Python matplotlib |

### Python 图表规范
- 字体：使用 SimHei 或 Microsoft YaHei（支持中文）
- 配色：专业商务风，避免花哨颜色
- 图片保存到 `charts/` 目录，300 DPI PNG 格式
- 每张图表下方标注数据来源

## Input
- Read `research-output/<project>/outline.md` (must be user-confirmed)
- Read `research-output/<project>/analysis.md`
- Read all source cards from `research-output/<project>/sources/`
- Read `research-output/<project>/config.md` for output_type and industry_scope

## Output File Naming

| output_type | 产出文件 |
|-------------|---------|
| research | `report.md` |
| feasibility | `feasibility-report.md` |
| proposal | `proposal.md` |
| decision | `decision.md` |
| plan | `project-plan.md` |

## Process

### Step 1: Write document body

Follow `outline.md` structure exactly. Writing style and depth vary by output_type:

#### output_type: research
- 信息密集，数据驱动，第三方客观视角
- 每个章节充分展开论述：背景上下文 + 具体数据 + 分析解读 + 结论
- 涉及企业案例时必须深入叙事（创始故事、经营规模、核心数据、成功因素、可复制性，每个重点案例至少 300 字）
- 使用尾注引用格式：`数据【1】`
- 标题需体现具体研究对象和模式

#### output_type: feasibility
- 论证式写作，正反对比，每个分析维度给出明确判断
- 技术方案论证：列出 2-3 种技术路线，用对比表格，给出推荐及理由
- 财务分析：必须包含具体的投资估算表、收益预测表、现金流测算
- 风险评估：使用风险矩阵（概率 x 影响），每个风险有应对策略
- 结论必须明确：**可行** / **不可行** / **有条件可行**（列出条件）
- 使用尾注引用格式

#### output_type: proposal
- 正式公文风格，精炼，结论导向
- 每段开门见山，避免铺垫
- 数据服务于论点，不做学术式展开
- 预算表格要清晰：分项列示，有小计和合计
- 效益分析要可量化：具体数字优于定性描述
- 篇幅控制：标准深度 10-15 页

#### output_type: decision
- 简洁记录体，不做深度分析
- 议题讨论部分：客观记录各方观点，不加评论
- 决议部分：每条决议必须包含 **责任人** 和 **完成时限**
- 行动项清单用表格：编号 | 事项 | 责任人 | 时限 | 状态
- 篇幅控制：通常 3-8 页

#### output_type: plan
- 执行导向，动词开头，避免抽象描述
- WBS 用树形结构或编号层级展示，到工作包级别
- 里程碑用表格：编号 | 里程碑 | 交付物 | 计划日期 | 验收标准
- RACI 矩阵用表格：活动 x 角色，填 R/A/C/I
- 风险登记册用表格：风险 | 概率 | 影响 | 应对 | 责任人

### Step 2: Generate charts (all types except decision)

For each chart specified in the outline:

1. Write a Python script that generates the chart
2. Run the script to produce a PNG in `charts/`
3. Embed in document as `![caption](charts/chart-name.png)`

Python chart requirements:
```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.rcParams['font.sans-serif'] = ['SimHei', 'Microsoft YaHei', 'DejaVu Sans']
matplotlib.rcParams['axes.unicode_minus'] = False

# ... chart code ...

plt.savefig('research-output/<project>/charts/<name>.png', dpi=300, bbox_inches='tight')
plt.close()
```

Chart style rules:
- Professional business color palette (blues, grays, accents)
- Clear axis labels in Chinese
- Data source annotation below chart
- Consistent styling across all charts
- 300 DPI PNG format

#### Additional charts by output_type:
- **feasibility**: 必须生成财务测算图表（投资回收曲线、敏感性分析蛛网图）
- **proposal**: 预算构成饼图或柱状图
- **plan**: 甘特图优先用 Mermaid gantt 语法

### Step 3: Generate Mermaid diagrams

For process/flow/relationship visuals, embed Mermaid directly in Markdown:
````
```mermaid
graph LR
    A[步骤1] --> B[步骤2]
    B --> C[步骤3]
```
````

#### output_type-specific Mermaid usage:
- **plan**: 使用 Mermaid gantt 生成进度甘特图：
````
```mermaid
gantt
    title 项目进度计划
    dateFormat YYYY-MM-DD
    section 阶段一
    任务1 :a1, 2026-07-01, 30d
    任务2 :after a1, 20d
```
````

### Step 4: Generate image requests (research and feasibility only)

For each image placeholder, write an entry in `image-requests.md`:

```markdown
# Image Requests

## img-01: [which section] — [what the image should show]
- **Context**: [why this image is needed]
- **Search suggestion**: "[Chinese keywords]"
- **Midjourney prompt**: "[English prompt if AI generation needed]"
- **Placeholder**: `![description](images/img-01.png)`
```

Image relevance rule: every image must directly relate to the paragraph it appears in.

Note: decision and plan types typically do not need images. Skip this step for those types unless the outline specifically requests images.

### Step 5: Write appendix

Appendix content varies by output_type:

- **research**: 参考文献（【N】编号，每条必须包含 URL）、方法论、术语表

  参考文献格式（每条必须包含 URL，读取 `sources/qN/src-*.md` 中的 `url` 字段）：
  ```
  【N】来源标题 — 机构/作者 — 发布日期
  URL: https://...
  ```
  如果 url 字段值以 `NOT_FOUND:` 开头，则写：
  ```
  【N】来源标题 — 机构/作者 — 发布日期
  URL: 暂无（原文：NOT_FOUND 后的说明）
  ```

- **feasibility**: 财务模型假设明细、参考文献（同上格式，必须含 URL）、技术方案详细参数
- **proposal**: 支撑文件清单（引用的调研报告、可研报告等）
- **decision**: 无附录（如有附件在正文中引用即可）
- **plan**: 详细预算明细表、合同/协议模板清单

### Step 6: Write summary section

After the full document is written, go back and write the summary:

- **research**: Executive Summary（约 800-1000 字），参考 QYResearch 等专业机构风格，信息密度要高。包含：开篇总论、市场规模、技术摘要、商业模式摘要、政策环境、风险与建议。
- **feasibility**: 项目摘要（约 500 字），一段话概括项目、一段话给出可行性结论和关键条件。
- **proposal**: 项目概述（一页纸），做什么、为什么、花多少、预期回报。
- **decision**: 无需单独摘要，会议基本信息即为开头。
- **plan**: 项目概述（约 300 字），目标、范围、关键约束。

## Output
- Main document file (see Output File Naming table)
- `charts/*.png` — generated chart images (if applicable)
- `image-requests.md` — image needs for user to fulfill (if applicable)

Return control to orchestrator for user review.
