# 康奈尔笔记模板填充指南

## 模板变量说明

模板文件 `assets/template-{style}.html` 使用 `{{VARIABLE}}` 占位符，四种风格可选：`tech`（科技/默认）、`business`（商务）、`github`（GitHub）、`apple`（苹果）。所有风格共用相同的变量和 HTML 结构，仅 CSS 不同。

### 变量清单

| 变量 | 说明 | 示例 |
|------|------|------|
| `{{REPORT_TITLE}}` | 报告/主题标题 | "咖啡市场分析报告" |
| `{{DATE}}` | 日期 | "2026-07-29" |
| `{{AUTHOR}}` | 作者 | "SunwuAI" |
| `{{CATEGORY}}` | 分类标签 | "市场分析" |
| `{{CUE_ITEMS}}` | 线索栏问题列表（HTML） | 见下文 |
| `{{NOTE_SECTIONS}}` | 笔记栏核心要点（HTML） | 见下文 |
| `{{FEYNMAN_EXPLANATION}}` | 费曼笔记白话解释 | 见下文 |
| `{{DIAGRAM_CONTENT}}` | 图示法内容（HTML） | 见下文 |
| `{{SUMMARY}}` | 总结栏一句话概括 | 见下文 |
| `{{TAGS}}` | 标签列表（HTML） | 见下文 |

---

## 各区域填充规范

### 1. 线索栏 `{{CUE_ITEMS}}`

将报告核心结论提炼为 3-5 个关键问题，每个问题一个 `.cue-item`：

```html
<div class="cue-item"><span class="q-mark">?</span> 咖啡市场整体增速如何？</div>
<div class="cue-item"><span class="q-mark">?</span> 主要增长驱动力是什么？</div>
<div class="cue-item"><span class="q-mark">?</span> 各品牌竞争力如何排名？</div>
```

**规范**：
- 每个问题必须能用右侧笔记栏回答
- 问题用疑问句，不用陈述句
- 覆盖报告的所有核心结论

### 2. 笔记栏 `{{NOTE_SECTIONS}}`

笔记栏是信息密度最高的区域。除基础的 `.note-section` 要点列表外，还可使用高密度组件（`numbered-header` 编号分区、`vs-block` 对比块、`card-grid` 卡片网格、`stat-row` 数据栏、多色高亮）——**组件的识别规则和 HTML 示例统一见 [extraction-guide.md](extraction-guide.md)**。

基础要点列表示例：

```html
<div class="note-section">
  <h4><span class="dot"></span>市场规模与增速</h4>
  <ul>
    <li>2026年中国咖啡市场规模达 <span class="note-highlight">2800亿元</span>，同比增长 15%</li>
    <li>现磨咖啡占比提升至 42%，超过速溶咖啡</li>
  </ul>
</div>
```

**规范**：
- 报告结构复杂时用 `numbered-header` 分区，每区内放一个组件
- 要点用短句，不写完整段落
- 按"结论 → 数据 → 解释"顺序排列

### 3. 费曼笔记 `{{FEYNMAN_EXPLANATION}}`

用大白话把人话讲清楚，假设给非专业人士解释：

```html
<strong>咖啡市场为什么在涨？</strong> 简单说，就是年轻人不喝茶了，开始喝咖啡。上班族每天一杯提神，学生党社交打卡。瑞幸靠便宜和外卖抢了星巴克的地盘，但星巴克靠"第三空间"留住了一批忠实用户。
```

**规范**：
- 禁用行业术语，必须用日常口语
- 用类比/比喻（如"就像外卖打败了堂食"）
- 长度控制在 2-3 句
- 加 `<strong>` 标记核心问题，后面跟大白话回答

### 4. 图示法 `{{DIAGRAM_CONTENT}}`

根据内容类型选择合适的图示：

#### 流程图（因果/时序关系）

```html
<div style="text-align:center;">
  <div class="flow-row">
    <div class="flow-box">原材料涨价</div>
    <div class="flow-arrow">→</div>
    <div class="flow-box warn">成本压力</div>
    <div class="flow-arrow">→</div>
    <div class="flow-box">提价策略</div>
    <div class="flow-arrow">→</div>
    <div class="flow-box accent">利润率回升</div>
  </div>
</div>
```

#### 对比表（竞品/方案对比）

```html
<table class="compare-table">
  <tr><th>维度</th><th>品牌A</th><th>品牌B</th><th>品牌C</th></tr>
  <tr><td>市占率</td><td>38%</td><td>25%</td><td>12%</td></tr>
  <tr><td>客单价</td><td>15元</td><td>35元</td><td>22元</td></tr>
  <tr><td>核心优势</td><td>性价比+外卖</td><td>空间体验</td><td>精品品质</td></tr>
</table>
```

#### 关系图（分层/包含关系）

```html
<div style="text-align:center;">
  <div class="flow-box accent" style="display:inline-block;">咖啡市场 2800亿</div>
  <div class="flow-v-arrow">↓</div>
  <div class="flow-row">
    <div class="flow-box">现磨 42%</div>
    <div class="flow-box">速溶 35%</div>
    <div class="flow-box">即饮 23%</div>
  </div>
</div>
```

**规范**：
- 只选一种图示类型，不要混用
- 因果/时序 → 流程图
- 多维对比 → 对比表
- 分层/包含 → 关系图
- 不超过 6 个节点，保持一页纸可容纳

### 5. 总结栏 `{{SUMMARY}}`

用一句话概括整篇报告的核心结论：

```html
中国咖啡市场持续双位数增长，<strong>现磨替代速溶</strong>是主趋势，瑞幸凭借性价比+外卖模式领先，但下沉市场仍是增量蓝海。
```

**规范**：
- 必须是一句话（允许用分号或逗号）
- 包含：核心趋势 + 关键数据 + 最重要的结论
- `<strong>` 标记最核心的关键词

### 6. 标签 `{{TAGS}}`

```html
<span class="tag primary">咖啡市场</span>
<span class="tag">竞争分析</span>
<span class="tag">核心结论</span>
```

---

## 生成流程

1. 根据用户偏好选择模板风格（默认 `tech`），读取对应 `assets/template-{style}.html`
2. 读取报告文件，提取结论性章节（执行摘要、关键发现、结论与建议）
3. 提炼 3-5 个核心问题 → 填入 `{{CUE_ITEMS}}`
4. 将结论按主题分组为要点 → 填入 `{{NOTE_SECTIONS}}`
5. 选最核心的结论用大白话解释 → 填入 `{{FEYNMAN_EXPLANATION}}`
6. 分析结论间的逻辑关系，选择合适的图示 → 填入 `{{DIAGRAM_CONTENT}}`
7. 写一句话总结 → 填入 `{{SUMMARY}}`
8. 从报告元数据提取标题、日期、分类 → 填入 header 变量
9. 将填充后的 HTML 保存为 `reports/{{标题}}_康奈尔笔记.html`
