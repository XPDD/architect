# 信息要素抽取指南

从报告中识别并抽取高密度信息要素，映射到对应的模板组件。目标：一页纸承载尽可能多的有效信息，同时保持可读性。

## 信息要素识别规则

阅读报告时，按以下信号扫描全文：

### 1. 对比关系 → `vs-block` 组件

**信号词**：旧/新、之前/之后、传统/现代、从…走向…、vs、相比、改为、取代

**抽取方法**：成对抽取，左右各 3-5 条，结构对称。

```html
<div class="vs-block">
  <div class="vs-col vs-old">
    <div class="vs-title">旧版：有状态协议</div>
    <ul>
      <li>服务端分配 session ID</li>
      <li>请求绑定到固定实例</li>
    </ul>
  </div>
  <div class="vs-arrow">→</div>
  <div class="vs-col vs-new">
    <div class="vs-title">新版：无状态协议</div>
    <ul>
      <li>每个请求独立</li>
      <li>可路由到任意实例</li>
    </ul>
  </div>
</div>
```

### 2. 并列多主题 → `card-grid` 组件

**信号词**：第一/第二/第三、A/B/C/D、一方面/另一方面、多个并列小标题

**抽取方法**：每个并列项一张卡片，badge 标记序号或字母，每卡 2-4 条要点。2 列网格，最多 4 张卡。

```html
<div class="card-grid">
  <div class="card">
    <div class="card-badge">A</div>
    <div class="card-title">MRTR 多轮请求</div>
    <ul class="card-list">
      <li>解决执行中途需用户确认的问题</li>
      <li>服务端可返回"需要输入"状态</li>
    </ul>
  </div>
  <div class="card">
    <div class="card-badge">B</div>
    <div class="card-title">新增请求头</div>
    <ul class="card-list">
      <li>Mcp-Method / Mcp-Name</li>
      <li>网关可按请求头直接路由</li>
    </ul>
  </div>
</div>
```

### 3. 关键数字 → `stat-row` 组件

**信号词**：百分比、倍数、绝对数值、排名、下载量、增长率

**抽取方法**：挑 3-5 个最有冲击力的数字，突出数值本身。

```html
<div class="stat-row">
  <div class="stat-item"><div class="stat-value">5亿</div><div class="stat-label">月下载量</div></div>
  <div class="stat-item"><div class="stat-value">10亿+</div><div class="stat-label">TS/Python SDK 累计</div></div>
  <div class="stat-item"><div class="stat-value">12个月</div><div class="stat-label">废弃过渡期</div></div>
</div>
```

### 4. 章节结构 → `numbered-header` 组件

**信号**：报告本身的章节划分（核心变化 / 迁移方案 / 生态 / 影响…）

**抽取方法**：笔记栏内容按编号分区组织，每个分区内部自由组合其他组件。

```html
<div class="numbered-header"><span class="num">1</span>核心变化</div>
<div class="numbered-header"><span class="num">2</span>生态与迁移</div>
```

### 5. 关键词强调 → 多色高亮

**信号**：首次出现的技术名词、版本号、协议名、核心结论词、警示词

**抽取方法**：每类关键词固定一种颜色，全篇保持一致。

| 组件 | 用途 | 示例 |
|------|------|------|
| `note-highlight` | 默认高亮（主色） | 核心数据、主要结论 |
| `hl-orange` | 警示/风险/被废弃项 | session ID、迁移成本 |
| `hl-green` / `hl-red` | 正面/负面 | 新特性 / 缺陷 |
| `hl-purple` / `hl-navy` | 技术术语/实体名 | MRTR、CIMD |

### 6. 因果/流程链 → `flow-box` 组件（已有）

**信号词**：导致、因此、然后、接着、最终、A→B→C

### 7. 厂商/列表枚举 → `card-list` 或双栏列表

**信号**：平台支持列表、SDK 列表、客户名单

## 抽取优先级

信息超过一页时，按此顺序取舍：

1. **必保留** — 核心结论、新旧对比、关键数字、废弃/风险项
2. **尽量保留** — 并列子主题（卡片网格压缩为 2-4 张）、流程链
3. **可压缩** — 详细推导过程（转为费曼一句话）、背景介绍
4. **可舍弃** — 未来展望细节、致谢、重复论证

## 密度目标

| 区域 | 容量 |
|------|------|
| 线索栏 | 5-8 个问题（覆盖每个编号分区） |
| 笔记栏 | 3-6 个编号分区，每区 1 个组件（vs-block / card-grid / stat-row / 列表） |
| 费曼块 | 1 段，2-3 句 |
| 图示区 | 1 个流程图或对比表 |
| 总结栏 | 1 句话 + 3-5 个标签 |

**校验标准**：随机抽一个线索栏问题，能在笔记栏 10 秒内找到答案；每个编号分区至少支撑一个线索问题。
