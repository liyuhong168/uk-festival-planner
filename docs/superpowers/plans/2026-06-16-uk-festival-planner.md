# UK节日选品备货决策工具 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个单一自包含HTML文件（`uk-festival-planner.html`），为亚马逊UK电商团队提供节日选品备货决策工具，覆盖2026年7-12月共23个节日节点。

**Architecture:** 纯原生HTML+CSS+JS，无框架无依赖。所有节日数据作为JS对象内联，渲染逻辑遍历数据生成DOM。localStorage持久化进度。采用阶段式构建：骨架→数据→渲染→交互→打磨。

**Tech Stack:** 原生 HTML5 / CSS3 / JavaScript (ES6+) / localStorage。无构建工具、无测试框架（手动在浏览器验证）。

**Spec:** `docs/superpowers/specs/2026-06-16-uk-festival-selection-planner-design.md`

---

## 文件结构

本项目交付物是**单一HTML文件**，但内部代码按职责分区（用注释分隔），便于维护：

**交付文件：**
- `uk-festival-planner.html` — 唯一交付物，内含：
  - `<style>` 区块：全部CSS（变量定义、布局、组件、响应式）
  - `<body>` 结构：header / dashboard / filter / main / footer
  - `<script>` 区块：
    - `CONFIG`：物流方式、颜色、阈值等常量
    - `FESTIVALS`：23个节日完整数据
    - `State`：状态管理（localStorage读写）
    - `Render`：渲染函数（看板、卡片、时间线、选品表）
    - `Interact`：交互绑定（筛选、勾选、切换、导入导出）

**说明：** 由于是单文件交付，不拆分多文件。但代码内严格分区，每个函数单一职责。

---

## 阶段总览

| 阶段 | 任务 | 产出 |
|---|---|---|
| **P1 骨架** | T1 HTML结构 + CSS变量 + 布局 | 空页面框架，样式就位 |
| **P2 数据** | T2 CONFIG + T3 FESTIVALS数据 | 完整数据层 |
| **P3 渲染** | T4 看板 + T5 筛选栏 + T6 节日卡片 + T7 时间线 + T8 选品表 | 静态展示完整 |
| **P4 交互** | T9 筛选逻辑 + T10 进度追踪 + T11 物流切换 + T12 导入导出 | 全交互可用 |
| **P5 打磨** | T13 响应式 + T14 细节 + T15 验收 | 交付 |

---

## Task 1: HTML骨架 + CSS基础 + 布局系统

**Files:**
- Create: `uk-festival-planner.html`

- [ ] **Step 1: 创建HTML文件，写入基础结构和CSS变量**

创建 `uk-festival-planner.html`，内容如下（这是骨架，数据区先留空）：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>UK节日选品备货决策工具</title>
<style>
:root {
  /* 背景与文字 */
  --bg: #f8fafc;
  --bg-card: #ffffff;
  --text: #1e293b;
  --text-muted: #64748b;
  --border: #e2e8f0;

  /* 紧急度色板 */
  --urgent: #ef4444;       /* 🔴紧急 */
  --week: #f97316;         /* 🟠本周 */
  --month: #eab308;        /* 🟡本月 */
  --plan: #22c55e;         /* 🟢规划 */
  --past: #94a3b8;         /* ⚫已过 */

  /* 强调色 */
  --accent: #3b82f6;
  --accent-hover: #2563eb;

  /* 风险色 */
  --risk-low: #22c55e;
  --risk-mid: #f97316;
  --risk-high: #ef4444;

  /* 布局 */
  --max-width: 1200px;
  --radius: 8px;
  --shadow: 0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.04);
  --shadow-hover: 0 4px 12px rgba(0,0,0,0.1);
}

* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: -apple-system, BlinkMacSystemFont, "Microsoft YaHei", "Segoe UI", sans-serif;
  background: var(--bg);
  color: var(--text);
  line-height: 1.6;
  font-size: 14px;
}

/* === ① 顶部固定导航栏 === */
.header {
  position: sticky;
  top: 0;
  z-index: 100;
  background: var(--bg-card);
  border-bottom: 1px solid var(--border);
  box-shadow: var(--shadow);
}
.header-inner {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: 12px 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}
.header-title { font-size: 18px; font-weight: 700; }
.header-title small { font-size: 12px; color: var(--text-muted); font-weight: 400; margin-left: 8px; }
.month-nav { display: flex; gap: 4px; flex-wrap: wrap; }
.month-nav a {
  padding: 4px 10px;
  border-radius: 4px;
  text-decoration: none;
  color: var(--text-muted);
  font-size: 13px;
  transition: all 0.15s;
}
.month-nav a:hover { background: var(--bg); color: var(--accent); }

/* === ② 紧急度看板 === */
.dashboard {
  max-width: var(--max-width);
  margin: 20px auto;
  padding: 0 20px;
}
.dashboard-hero {
  background: var(--bg-card);
  border-radius: var(--radius);
  padding: 20px;
  box-shadow: var(--shadow);
  margin-bottom: 16px;
}
.countdown { font-size: 14px; color: var(--text-muted); margin-bottom: 4px; }
.countdown strong { color: var(--text); font-size: 20px; }
.stat-cards { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
.stat-card {
  background: var(--bg-card);
  border-radius: var(--radius);
  padding: 14px;
  box-shadow: var(--shadow);
  cursor: pointer;
  border-left: 4px solid var(--border);
  transition: all 0.15s;
}
.stat-card:hover { box-shadow: var(--shadow-hover); transform: translateY(-1px); }
.stat-card.urgent { border-left-color: var(--urgent); }
.stat-card.week { border-left-color: var(--week); }
.stat-card.month { border-left-color: var(--month); }
.stat-card.plan { border-left-color: var(--plan); }
.stat-card .num { font-size: 28px; font-weight: 700; }
.stat-card .label { font-size: 12px; color: var(--text-muted); }
.stat-card.urgent .num { color: var(--urgent); }
.stat-card.week .num { color: var(--week); }
.stat-card.month .num { color: var(--month); }
.stat-card.plan .num { color: var(--plan); }

/* === ③ 筛选工具栏 === */
.filter-bar {
  position: sticky;
  top: 57px;
  z-index: 90;
  background: var(--bg-card);
  border-bottom: 1px solid var(--border);
  box-shadow: var(--shadow);
}
.filter-inner {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: 10px 20px;
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  align-items: center;
}
.filter-group { display: flex; gap: 4px; align-items: center; }
.filter-group label { font-size: 12px; color: var(--text-muted); margin-right: 4px; }
.filter-bar select, .filter-bar input[type="text"] {
  padding: 5px 8px;
  border: 1px solid var(--border);
  border-radius: 4px;
  font-size: 13px;
  background: var(--bg-card);
  color: var(--text);
}
.filter-bar input[type="text"] { min-width: 160px; }
.filter-bar button {
  padding: 5px 10px;
  border: 1px solid var(--border);
  border-radius: 4px;
  background: var(--bg-card);
  cursor: pointer;
  font-size: 13px;
}
.filter-bar button:hover { background: var(--bg); }

/* === ④ 节日卡片列表 === */
.main {
  max-width: var(--max-width);
  margin: 20px auto;
  padding: 0 20px;
}
.month-section { margin-bottom: 32px; }
.month-section h2 {
  font-size: 18px;
  margin-bottom: 12px;
  padding-bottom: 6px;
  border-bottom: 2px solid var(--border);
  display: flex;
  align-items: center;
  gap: 8px;
}
.month-section h2 .count {
  font-size: 13px;
  color: var(--text-muted);
  font-weight: 400;
}
.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(380px, 1fr));
  gap: 16px;
}

/* === 节日卡片 === */
.festival-card {
  background: var(--bg-card);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  overflow: hidden;
  border-left: 4px solid var(--border);
  transition: box-shadow 0.15s;
}
.festival-card:hover { box-shadow: var(--shadow-hover); }
.festival-card.urgent { border-left-color: var(--urgent); }
.festival-card.week { border-left-color: var(--week); }
.festival-card.month { border-left-color: var(--month); }
.festival-card.plan { border-left-color: var(--plan); }
.festival-card.past { border-left-color: var(--past); opacity: 0.7; }

.card-header {
  padding: 14px 16px;
  cursor: pointer;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 8px;
}
.card-title { display: flex; align-items: center; gap: 8px; font-size: 16px; font-weight: 600; }
.card-title .icon { font-size: 22px; }
.card-title .name-en { font-size: 12px; color: var(--text-muted); font-weight: 400; }
.card-meta { font-size: 12px; color: var(--text-muted); margin-top: 4px; }
.badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
  margin-left: 6px;
}
.badge.urgent { background: #fef2f2; color: var(--urgent); }
.badge.week { background: #fff7ed; color: var(--week); }
.badge.month { background: #fefce8; color: var(--month); }
.badge.plan { background: #f0fdf4; color: var(--plan); }
.badge.past { background: #f1f5f9; color: var(--past); }
.badge.importance-S { background: #fef2f2; color: var(--urgent); }

.card-body { padding: 0 16px 16px; display: none; }
.festival-card.expanded .card-body { display: block; }

/* === 时间线 === */
.timeline {
  background: var(--bg);
  border-radius: 6px;
  padding: 12px;
  margin-bottom: 12px;
}
.timeline-track {
  display: flex;
  justify-content: space-between;
  position: relative;
  margin: 8px 0 4px;
}
.timeline-track::before {
  content: "";
  position: absolute;
  top: 8px;
  left: 8px;
  right: 8px;
  height: 2px;
  background: var(--border);
}
.timeline-node {
  position: relative;
  z-index: 1;
  text-align: center;
  flex: 1;
}
.timeline-dot {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: var(--bg-card);
  border: 2px solid var(--border);
  margin: 0 auto 4px;
  cursor: pointer;
}
.timeline-dot.done { background: var(--plan); border-color: var(--plan); }
.timeline-node .ms-name { font-size: 11px; color: var(--text); }
.timeline-node .ms-date { font-size: 10px; color: var(--text-muted); }

/* === 选品表 === */
.products-section { margin-bottom: 12px; }
.products-section h4 { font-size: 13px; margin-bottom: 8px; color: var(--text); }
.product-cat-tabs { display: flex; gap: 4px; margin-bottom: 8px; flex-wrap: wrap; }
.product-cat-tab {
  padding: 3px 10px;
  border-radius: 12px;
  font-size: 12px;
  cursor: pointer;
  background: var(--bg);
  border: 1px solid var(--border);
}
.product-cat-tab.active { background: var(--accent); color: white; border-color: var(--accent); }
.product-table { width: 100%; font-size: 12px; border-collapse: collapse; }
.product-table th {
  text-align: left;
  padding: 6px 8px;
  background: var(--bg);
  color: var(--text-muted);
  font-weight: 600;
  font-size: 11px;
}
.product-table td { padding: 6px 8px; border-top: 1px solid var(--border); vertical-align: top; }
.product-table tr:hover { background: var(--bg); }
.stars { color: #f59e0b; letter-spacing: -1px; }
.risk { font-size: 11px; font-weight: 600; padding: 1px 6px; border-radius: 8px; }
.risk-low { background: #f0fdf4; color: var(--risk-low); }
.risk-mid { background: #fff7ed; color: var(--risk-mid); }
.risk-high { background: #fef2f2; color: var(--risk-high); }

/* === 验证指引 === */
.validation-section {
  background: #eff6ff;
  border-radius: 6px;
  padding: 10px;
  margin-bottom: 12px;
  font-size: 12px;
}
.validation-section h4 { font-size: 12px; margin-bottom: 6px; color: var(--accent); }
.validation-section ul { margin-left: 16px; color: var(--text-muted); }
.validation-section li { margin-bottom: 2px; }

/* === 备注与控制 === */
.card-controls {
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 8px;
}
.card-controls label { font-size: 12px; color: var(--text-muted); }
.card-controls select { padding: 4px 8px; font-size: 12px; border: 1px solid var(--border); border-radius: 4px; }
.logistics-toggle { display: flex; gap: 2px; }
.logistics-toggle button {
  padding: 4px 10px;
  font-size: 11px;
  border: 1px solid var(--border);
  background: var(--bg-card);
  cursor: pointer;
  border-radius: 4px;
}
.logistics-toggle button.active { background: var(--accent); color: white; border-color: var(--accent); }
.notes-area {
  width: 100%;
  padding: 6px 8px;
  border: 1px solid var(--border);
  border-radius: 4px;
  font-size: 12px;
  min-height: 40px;
  resize: vertical;
  font-family: inherit;
}

/* === ⑤ 页脚 === */
.footer {
  max-width: var(--max-width);
  margin: 40px auto 20px;
  padding: 20px;
  background: var(--bg-card);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  font-size: 12px;
  color: var(--text-muted);
}
.footer h3 { font-size: 14px; color: var(--text); margin-bottom: 8px; }
.footer .disclaimer {
  background: #fffbeb;
  border-left: 3px solid var(--week);
  padding: 8px 12px;
  margin: 12px 0;
  border-radius: 4px;
}
.footer-actions { display: flex; gap: 8px; margin-top: 12px; }
.footer-actions button {
  padding: 6px 14px;
  border: 1px solid var(--border);
  border-radius: 4px;
  background: var(--bg-card);
  cursor: pointer;
  font-size: 13px;
}
.footer-actions button:hover { background: var(--bg); }
.footer-actions button.danger { color: var(--urgent); border-color: var(--urgent); }
.footer-actions button.danger:hover { background: #fef2f2; }

/* === 回到顶部 === */
.back-to-top {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: var(--accent);
  color: white;
  border: none;
  cursor: pointer;
  box-shadow: var(--shadow-hover);
  display: none;
  font-size: 18px;
  z-index: 80;
}
.back-to-top.show { display: block; }

/* === 空状态 === */
.empty-state {
  text-align: center;
  padding: 40px;
  color: var(--text-muted);
}

/* === 响应式 === */
@media (max-width: 768px) {
  .stat-cards { grid-template-columns: repeat(2, 1fr); }
  .cards-grid { grid-template-columns: 1fr; }
  .header-inner { flex-direction: column; align-items: flex-start; }
  .filter-bar { top: auto; position: relative; }
}

/* hidden util */
.hidden { display: none !important; }
</style>
</head>
<body>

<!-- ① 顶部导航 -->
<header class="header">
  <div class="header-inner">
    <div class="header-title">
      🇬🇧 UK节日选品备货决策工具
      <small>2026年7月 - 12月</small>
    </div>
    <nav class="month-nav" id="monthNav"></nav>
  </div>
</header>

<!-- ② 紧急度看板 -->
<section class="dashboard">
  <div class="dashboard-hero">
    <div class="countdown" id="countdown"></div>
  </div>
  <div class="stat-cards" id="statCards"></div>
</section>

<!-- ③ 筛选工具栏 -->
<div class="filter-bar">
  <div class="filter-inner">
    <div class="filter-group">
      <label>品类</label>
      <select id="filterCategory">
        <option value="">全部</option>
        <option value="decor">🎃装饰</option>
        <option value="gift">🎁礼品</option>
        <option value="apparel">👕服饰</option>
        <option value="home">🏠家居</option>
      </select>
    </div>
    <div class="filter-group">
      <label>月份</label>
      <select id="filterMonth">
        <option value="">全部</option>
        <option value="7">7月</option>
        <option value="8">8月</option>
        <option value="9">9月</option>
        <option value="10">10月</option>
        <option value="11">11月</option>
        <option value="12">12月</option>
      </select>
    </div>
    <div class="filter-group">
      <label>紧急度</label>
      <select id="filterUrgency">
        <option value="">全部</option>
        <option value="urgent">🔴紧急</option>
        <option value="week">🟠本周</option>
        <option value="month">🟡本月</option>
        <option value="plan">🟢规划</option>
        <option value="past">⚫已过</option>
      </select>
    </div>
    <div class="filter-group">
      <label>状态</label>
      <select id="filterStatus">
        <option value="">全部</option>
        <option value="none">未启动</option>
        <option value="selection">选品中</option>
        <option value="ordered">已下单</option>
        <option value="arrived">已到仓</option>
        <option value="listed">已上架</option>
      </select>
    </div>
    <div class="filter-group">
      <input type="text" id="filterSearch" placeholder="🔍 搜索节日/SKU/关键词">
    </div>
    <div class="filter-group">
      <button id="resetFilter">重置</button>
    </div>
  </div>
</div>

<!-- ④ 节日卡片列表 -->
<main class="main" id="main"></main>

<!-- ⑤ 页脚 -->
<footer class="footer">
  <h3>📌 使用说明</h3>
  <p>• 点击节日卡片标题展开详情；点击时间线圆点勾选里程碑进度。</p>
  <p>• 切换物流方式（海运/卡航/空运）会重算该节日的时间线和紧急度。</p>
  <p>• 所有进度自动保存在浏览器本地（localStorage），建议定期导出备份。</p>
  <div class="disclaimer">
    <strong>⚠️ 数据免责声明：</strong>本工具的SKU建议、售价区间、毛利率为基于行业经验的参考值，非实时销量数据。请通过 Google Trends、亚马逊BSR、Keepa 等工具验证后再决策。拿货成本以1688实际询价为准。
  </div>
  <div class="footer-actions">
    <button onclick="exportData()">📤 导出备份</button>
    <button onclick="document.getElementById('importFile').click()">📥 导入恢复</button>
    <input type="file" id="importFile" accept=".json" class="hidden" onchange="importData(event)">
    <button class="danger" onclick="resetAllProgress()">🗑 清除所有进度</button>
  </div>
</footer>

<button class="back-to-top" id="backToTop" onclick="window.scrollTo({top:0,behavior:'smooth'})">↑</button>

<script>
// ============================================
// 数据与逻辑将在后续任务中填充
// ============================================
</script>

</body>
</html>
```

- [ ] **Step 2: 在浏览器打开验证骨架**

双击 `uk-festival-planner.html` 在浏览器打开。
预期：看到标题栏、空的看板区、空的筛选栏（下拉有选项）、空的主区域、页脚（含免责声明和3个按钮）。页面应为浅色背景，无报错。

- [ ] **Step 3: 初始化git并提交**

工作区目前非git仓库，先初始化：

```bash
cd /d "D:\软件\Zcode工作区"
git init
git add uk-festival-planner.html docs/
git commit -m "feat: HTML骨架与CSS样式系统 (Task 1)"
```

---

## Task 2: CONFIG常量定义

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块

- [ ] **Step 1: 在 `<script>` 标签内，替换注释，写入CONFIG**

找到 `<script>` 内的 `// 数据与逻辑将在后续任务中填充` 注释，替换为：

```js
// ============================================
// CONFIG：全局常量配置
// ============================================
const CONFIG = {
  // 物流方式：总周期 = 生产 + 头程运输
  logisticsModes: {
    air:   { leadTime: 15, production: 7,  transit: 8,  label: "空运" },
    truck: { leadTime: 35, production: 10, transit: 25, label: "卡航/快铁" },
    sea:   { leadTime: 55, production: 15, transit: 40, label: "海运" }
  },
  defaultLogistics: "truck",

  // 双段物流里程碑（tMinus = 距节日的负天数）
  // 总提前期 = leadTime + 14（14天为到仓上架预留缓冲）
  milestones: [
    { id: "selection",  tMinusLead: 14, name: "选品+下单",   actions: "定SKU；同时下空运首批50件+卡航大货" },
    { id: "airArrival", tMinusLead: 8,  name: "空运首批到仓", actions: "空运到仓，开始测款观察" },
    { id: "truckShip",  tMinusLead: -10, name: "卡航大货发运", actions: "大货出厂发UK（卡航25天）" },
    { id: "arrival",    tMinusLead: -21, name: "大货到仓",     actions: "FBA收货上架，Listing转Active" },
    { id: "festival",   tMinusLead: -35, name: "节日销售",     actions: "广告加投，促销启动" }
  ],
  // 说明：tMinusLead 是相对于 (节日日期 - leadTime) 的偏移
  // selection 在 leadTime起点+14天前（即 T-(leadTime+14)）
  // airArrival 在 leadTime起点+8天前 = 生产7天+1 = 选品后8天空运到仓
  // 实际换算见 getMilestoneDate()

  // 紧急度阈值（基于选品截止日距今天数）
  urgencyThresholds: {
    urgent: 0,      // 选品截止日已过
    week: 7,        // ≤7天
    month: 30,      // ≤30天
    plan: Infinity  // >30天
  },

  // 品类标签
  categories: {
    decor:   { label: "🎃装饰", color: "#fb923c" },
    gift:    { label: "🎁礼品", color: "#ec4899" },
    apparel: { label: "👕服饰", color: "#8b5cf6" },
    home:    { label: "🏠家居", color: "#14b8a6" }
  },

  // 月份配置
  months: [
    { num: 7,  label: "7月" },
    { num: 8,  label: "8月" },
    { num: 9,  label: "9月" },
    { num: 10, label: "10月" },
    { num: 11, label: "11月" },
    { num: 12, label: "12月" }
  ],

  // 存储键
  storageKey: "uk_festival_planner_v1"
};

// 物流方式选项（用于切换按钮）
const LOGISTICS_OPTIONS = [
  { id: "sea",   label: "海运55天", icon: "🚢" },
  { id: "truck", label: "卡航35天", icon: "🚆" },
  { id: "air",   label: "空运15天", icon: "✈️" }
];
```

- [ ] **Step 2: 验证语法无报错**

在浏览器打开HTML，按F12控制台，输入 `CONFIG.logisticsModes.truck.leadTime` 应返回 `35`。无报错。

- [ ] **Step 3: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: CONFIG常量定义-物流/品类/阈值 (Task 2)"
```

---

## Task 3: FESTIVALS节日数据（23个节点）

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块（在CONFIG后追加）

这是内容量最大的任务。按月份分批写入，每批验证。

- [ ] **Step 1: 写入7月和8月节日数据（5个节点）**

在 `LOGISTICS_OPTIONS` 定义之后追加：

```js
// ============================================
// FESTIVALS：23个节日节点完整数据（2026年7-12月）
// ============================================
const FESTIVALS = [

// ===== 7月（2个）=====
{
  id: "summer-sale-2026",
  name: "夏季促销", nameEn: "Summer Sale", icon: "☀️",
  date: "2026-07-15", month: 7, importance: "A", category: "activity",
  themeColor: "#14b8a6",
  products: [
    { sku: "便携小风扇(USB充电)", skuEn: "Portable USB Mini Fan", category: "home", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "注意电池运输规定；选无电池款更安全", keywords: ["mini fan","portable fan","usb fan"], sourcing: "1688: 迷你USB风扇" },
    { sku: "冰丝凉爽坐垫", skuEn: "Cooling Gel Seat Cushion", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "低", riskNote: "常规品类无特殊风险", keywords: ["cooling cushion","gel seat pad"], sourcing: "1688: 冰丝凉垫" },
    { sku: "防晒冰袖套", skuEn: "UV Protection Cooling Arm Sleeves", category: "apparel", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约50-60%", matchScore: 5, riskLevel: "低", riskNote: "注意UK成人尺码", keywords: ["arm sleeves","uv protection","cooling sleeves"], sourcing: "1688: 防晒冰袖" },
    { sku: "便携BBQ烧烤垫", skuEn: "Portable BBQ Grill Mat", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约45-55%", matchScore: 4, riskLevel: "中", riskNote: "需食品级材质认证", keywords: ["bbq mat","grill mat","barbecue accessories"], sourcing: "1688: 烧烤垫" }
  ],
  validation: {
    googleTrends: ["summer sale 2026","cooling products uk"],
    amazonCheck: "BSR < 8000 in Garden & Outdoors；头部评论 < 300",
    sourcing: "1688搜索：夏季用品/防晒/便携风扇",
    riskFlags: ["电子产品注意UKCA认证","食品接触材质需检测报告"]
  }
},
{
  id: "summer-bbq-2026",
  name: "夏季户外/BBQ季", nameEn: "Summer Outdoor & BBQ Season", icon: "🍖",
  date: "2026-07-01", month: 7, importance: "B", category: "trend",
  themeColor: "#f97316",
  products: [
    { sku: "户外野餐垫(防水)", skuEn: "Waterproof Picnic Blanket", category: "decor", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["picnic blanket","outdoor mat"], sourcing: "1688: 户外野餐垫" },
    { sku: "烧烤工具套装(便携)", skuEn: "Portable BBQ Tool Set", category: "home", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 3, riskLevel: "中", riskNote: "食品级不锈钢认证；体积偏大注意FBA费用", keywords: ["bbq tools","grill set"], sourcing: "1688: 烧烤工具套装" }
  ],
  validation: {
    googleTrends: ["bbq accessories uk","picnic blanket"],
    amazonCheck: "BSR < 10000；季节性强，7月峰值",
    sourcing: "1688搜索：BBQ/野餐/户外",
    riskFlags: ["体积大者FBA费用高，需核算毛利"]
  }
},

// ===== 8月（3个）=====
{
  id: "summer-bank-holiday-2026",
  name: "夏季银行假", nameEn: "Summer Bank Holiday", icon: "🏖️",
  date: "2026-08-31", month: 8, importance: "A", category: "activity",
  themeColor: "#0ea5e9",
  products: [
    { sku: "海滩沙滩垫", skuEn: "Beach Beach Mat", category: "decor", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["beach mat","sand free blanket"], sourcing: "1688: 沙滩垫" },
    { sku: "防水手机袋", skuEn: "Waterproof Phone Pouch", category: "home", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 5, riskLevel: "低", riskNote: "轻小件高毛利", keywords: ["waterproof phone case","beach pouch"], sourcing: "1688: 防水手机袋" },
    { sku: "夏日主题派对装饰套装", skuEn: "Summer Party Decoration Set", category: "decor", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["summer party decorations","beach party"], sourcing: "1688: 夏日派对装饰" },
    { sku: "冷感毛巾", skuEn: "Cooling Towel Microfiber", category: "apparel", costRange: "¥4-8", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["cooling towel","microfiber towel"], sourcing: "1688: 冰感毛巾" }
  ],
  validation: {
    googleTrends: ["bank holiday","beach accessories uk"],
    amazonCheck: "BSR < 8000；8月底峰值",
    sourcing: "1688搜索：沙滩/防水/派对",
    riskFlags: ["季节末，注意不要积压库存"]
  }
},
{
  id: "back-to-school-prep-2026",
  name: "返校准备期", nameEn: "Back to School Prep", icon: "🎒",
  date: "2026-08-20", month: 8, importance: "A", category: "activity",
  themeColor: "#3b82f6",
  products: [
    { sku: "学生文具套装", skuEn: "Student Stationery Set", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "常规品类", keywords: ["stationery set","school supplies"], sourcing: "1688: 学生文具套装" },
    { sku: "儿童午餐盒(分格)", skuEn: "Kids Bento Lunch Box", category: "home", costRange: "¥5-10", priceRange: "£6.49-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "中", riskNote: "食品级材质BPA-Free认证", keywords: ["lunch box","bento box","kids lunch"], sourcing: "1688: 儿童午餐盒" },
    { sku: "书包挂饰/标签", skuEn: "Backpack Bag Tag", category: "gift", costRange: "¥2-5", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 5, riskLevel: "低", riskNote: "轻小件高毛利", keywords: ["bag tag","backpack accessories"], sourcing: "1688: 书包挂件" },
    { sku: "卡通姓名贴纸", skuEn: "Personalized Name Stickers", category: "gift", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约60-70%", matchScore: 5, riskLevel: "低", riskNote: "需支持定制印刷", keywords: ["name stickers","school labels"], sourcing: "1688: 姓名贴定制" }
  ],
  validation: {
    googleTrends: ["back to school 2026","school supplies uk"],
    amazonCheck: "BSR < 5000；8-9月强季节性",
    sourcing: "1688搜索：文具/午餐盒/学生用品",
    riskFlags: ["食品接触材质需认证","定制类需确认生产周期"]
  }
},
{
  id: "youth-day-2026",
  name: "国际青年节/亲子主题", nameEn: "International Youth Day", icon: "👨‍👩‍👧",
  date: "2026-08-12", month: 8, importance: "B", category: "trend",
  themeColor: "#a855f7",
  products: [
    { sku: "亲子装T恤套装", skuEn: "Family Matching T-Shirt Set", category: "apparel", costRange: "¥10-20", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 3, riskLevel: "中", riskNote: "UK尺码对照；退货率偏高", keywords: ["family matching shirts","parent child"], sourcing: "1688: 亲子装" },
    { sku: "DIY手工材料包", skuEn: "DIY Craft Kit for Kids", category: "gift", costRange: "¥5-10", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "注意小零件 choking hazard 标注", keywords: ["craft kit","diy for kids"], sourcing: "1688: 儿童手工包" }
  ],
  validation: {
    googleTrends: ["family matching","kids craft kit"],
    amazonCheck: "BSR < 15000；小众但有稳定需求",
    sourcing: "1688搜索：亲子/手工/儿童DIY",
    riskFlags: ["儿童用品注意年龄标注和安全说明"]
  }
},

// ===== 9月（4个）=====
{
  id: "back-to-school-2026",
  name: "返校季", nameEn: "Back to School", icon: "📚",
  date: "2026-09-01", month: 9, importance: "A", category: "activity",
  themeColor: "#3b82f6",
  products: [
    { sku: "笔记本套装(A5 5本)", skuEn: "A5 Notebook Set 5 Pack", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "常规品类", keywords: ["notebook set","a5 notebooks","school stationery"], sourcing: "1688: A5笔记本套装" },
    { sku: "铅笔盒/笔袋", skuEn: "Pencil Case Large Capacity", category: "gift", costRange: "¥4-8", priceRange: "£6.49-8.49", margin: "约50-60%", matchScore: 5, riskLevel: "低", riskNote: "轻小件高毛利", keywords: ["pencil case","pen bag"], sourcing: "1688: 笔袋" },
    { sku: "学生水壶(保温)", skuEn: "Kids Insulated Water Bottle", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约40-50%", matchScore: 4, riskLevel: "中", riskNote: "食品级不锈钢认证；保温效果测试", keywords: ["water bottle","kids flask","insulated bottle"], sourcing: "1688: 儿童保温杯" },
    { sku: "课程表/计划表磁贴", skuEn: "Magnetic Weekly Planner", category: "home", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 4, riskLevel: "低", riskNote: "轻小件", keywords: ["magnetic planner","weekly schedule"], sourcing: "1688: 磁性计划表" }
  ],
  validation: {
    googleTrends: ["back to school supplies","stationery uk"],
    amazonCheck: "BSR < 5000；9月初峰值",
    sourcing: "1688搜索：文具/笔记本/学生用品",
    riskFlags: ["食品接触材质需认证"]
  }
},
{
  id: "apple-day-2026",
  name: "下午茶季/Apple Day", nameEn: "Afternoon Tea Season", icon: "🫖",
  date: "2026-09-21", month: 9, importance: "A", category: "trend",
  themeColor: "#84cc16",
  products: [
    { sku: "陶瓷下午茶杯碟套装", skuEn: "Ceramic Teacup & Saucer Set", category: "home", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 3, riskLevel: "中", riskNote: "易碎品包装成本；FBA尺寸重量", keywords: ["teacup set","afternoon tea","ceramic cups"], sourcing: "1688: 下午茶杯碟" },
    { sku: "木质茶点托盘", skuEn: "Wooden Serving Tray", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约40-50%", matchScore: 3, riskLevel: "低", riskNote: "体积偏大注意FBA费用", keywords: ["serving tray","wooden tray","tea tray"], sourcing: "1688: 木质托盘" },
    { sku: "杯垫套装(软木)", skuEn: "Cork Coaster Set", category: "decor", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 5, riskLevel: "低", riskNote: "轻小件高毛利", keywords: ["cork coasters","drink coasters"], sourcing: "1688: 软木杯垫" },
    { sku: "英式花纹餐巾纸套装", skuEn: "Floral Paper Napkin Set", category: "decor", costRange: "¥4-8", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 4, riskLevel: "低", riskNote: "食品接触材质", keywords: ["paper napkins","floral napkins"], sourcing: "1688: 花纹餐巾纸" }
  ],
  validation: {
    googleTrends: ["afternoon tea","tea accessories uk"],
    amazonCheck: "BSR < 10000；秋季稳定需求",
    sourcing: "1688搜索：茶具/杯垫/托盘",
    riskFlags: ["陶瓷易碎需加强包装","体积大者核算FBA"]
  }
},
{
  id: "pirate-day-2026",
  name: "海盗说话日", nameEn: "Talk Like a Pirate Day", icon: "🏴‍☠️",
  date: "2026-09-19", month: 9, importance: "B", category: "trend",
  themeColor: "#92400e",
  products: [
    { sku: "海盗主题派对套装", skuEn: "Pirate Party Decoration Kit", category: "decor", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["pirate party","pirate decorations"], sourcing: "1688: 海盗派对装饰" },
    { sku: "海盗眼罩/帽子配件", skuEn: "Pirate Eye Patch & Hat", category: "apparel", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 5, riskLevel: "低", riskNote: "轻小件", keywords: ["pirate costume","eye patch"], sourcing: "1688: 海盗道具" }
  ],
  validation: {
    googleTrends: ["pirate day","pirate party uk"],
    amazonCheck: "BSR < 20000；小众但精准",
    sourcing: "1688搜索：海盗/派对道具",
    riskFlags: ["小众节日，控制首批量"]
  }
},
{
  id: "harvest-festival-2026",
  name: "收获节", nameEn: "Harvest Festival", icon: "🌾",
  date: "2026-09-27", month: 9, importance: "B", category: "festival",
  themeColor: "#ca8a04",
  products: [
    { sku: "秋季主题门环装饰", skuEn: "Autumn Wreath Decoration", category: "decor", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 3, riskLevel: "低", riskNote: "体积偏大注意FBA", keywords: ["autumn wreath","harvest wreath","fall decor"], sourcing: "1688: 秋季门环" },
    { sku: "南瓜主题摆件", skuEn: "Pumpkin Figurine Decor", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["pumpkin decor","fall decorations"], sourcing: "1688: 南瓜摆件" }
  ],
  validation: {
    googleTrends: ["harvest festival","autumn decorations uk"],
    amazonCheck: "BSR < 15000；与万圣节衔接",
    sourcing: "1688搜索：秋季/南瓜/丰收装饰",
    riskFlags: ["与万圣节备货可合并考虑"]
  }
}
];
```

- [ ] **Step 2: 验证7-9月数据**

浏览器控制台输入 `FESTIVALS.length`，预期返回 `9`（7月2 + 8月3 + 9月4）。
输入 `FESTIVALS[0].products.length`，预期返回 `4`（夏季促销有4个SKU）。

- [ ] **Step 3: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: 7-9月节日数据(9个节点) (Task 3a)"
```

- [ ] **Step 4: 写入10-12月节日数据（14个节点）**

在 `FESTIVALS` 数组的最后一个 `}`（harvest-festival）之后、数组闭合 `];` 之前，追加10-12月数据。注意逗号。

```js
,

// ===== 10月（5个）=====
{
  id: "halloween-2026",
  name: "万圣节", nameEn: "Halloween", icon: "🎃",
  date: "2026-10-31", month: 10, importance: "S", category: "festival",
  themeColor: "#fb923c",
  products: [
    { sku: "南瓜LED串灯", skuEn: "Pumpkin LED String Lights", category: "decor", costRange: "¥8-15", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "注意LED的UKCA/CE认证", keywords: ["halloween lights","pumpkin lights","outdoor halloween"], sourcing: "1688: 万圣节南瓜灯串" },
    { sku: "鬼怪窗贴纸套装", skuEn: "Halloween Window Clings Set", category: "decor", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 4, riskLevel: "低", riskNote: "轻小件高毛利", keywords: ["window clings","halloween stickers","window decor"], sourcing: "1688: 万圣节窗贴" },
    { sku: "蜘蛛网装饰套装", skuEn: "Spider Web Decoration Set", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["spider web","halloween decorations","fake web"], sourcing: "1688: 蜘蛛网装饰" },
    { sku: "万圣节主题糖果袋(10只装)", skuEn: "Halloween Treat Bags 10 Pack", category: "gift", costRange: "¥6-12", priceRange: "£7.99-9.99", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "食品接触材质", keywords: ["treat bags","halloween candy bags","trick or treat"], sourcing: "1688: 万圣节糖果袋" },
    { sku: "南瓜主题儿童斗篷", skuEn: "Kids Pumpkin Cape Costume", category: "apparel", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 3, riskLevel: "中", riskNote: "UK儿童尺码；含阻燃标签要求", keywords: ["kids costume","pumpkin cape","halloween costume"], sourcing: "1688: 儿童南瓜斗篷" },
    { sku: "万圣节主题餐垫套装", skuEn: "Halloween Placemat Set", category: "home", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["halloween placemats","table decor","party supplies"], sourcing: "1688: 万圣节餐垫" },
    { sku: "骷髅骨架摆件", skuEn: "Skeleton Decoration Prop", category: "decor", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["skeleton decoration","halloween props","bone decor"], sourcing: "1688: 骷髅摆件" },
    { sku: "万圣节派对气球套装", skuEn: "Halloween Party Balloon Set", category: "decor", costRange: "¥4-8", priceRange: "£5.99-7.99", margin: "约50-60%", matchScore: 5, riskLevel: "低", riskNote: "轻小件", keywords: ["halloween balloons","party balloons"], sourcing: "1688: 万圣节气球" },
    { sku: "主题马克杯(南瓜造型)", skuEn: "Pumpkin Shaped Mug", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约40-50%", matchScore: 4, riskLevel: "中", riskNote: "陶瓷易碎包装", keywords: ["pumpkin mug","halloween mug","novelty cup"], sourcing: "1688: 南瓜造型杯" },
    { sku: "幽灵主题抱枕套", skuEn: "Ghost Throw Pillow Cover", category: "home", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 4, riskLevel: "低", riskNote: "注意是枕套非枕头", keywords: ["pillow cover","ghost decor","halloween home"], sourcing: "1688: 万圣节抱枕套" },
    { sku: "吸血鬼假牙道具套装", skuEn: "Vampire Fangs Prop Set", category: "apparel", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约60-70%", matchScore: 5, riskLevel: "中", riskNote: "入口道具需食品级材质", keywords: ["vampire fangs","halloween props","fake teeth"], sourcing: "1688: 吸血鬼假牙" },
    { sku: "万圣节礼物盒套装(5只)", skuEn: "Halloween Gift Boxes 5 Pack", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["gift boxes","halloween boxes","party favors"], sourcing: "1688: 万圣节礼盒" }
  ],
  validation: {
    googleTrends: ["halloween decorations 2026","halloween decor uk"],
    amazonCheck: "BSR < 3000；10月强季节性峰值",
    sourcing: "1688搜索：万圣节装饰/南瓜/骷髅",
    riskFlags: ["LED产品需UKCA认证","儿童服装需阻燃标签","避免Disney/IP侵权","入口道具需食品级"]
  }
},
{
  id: "teachers-day-2026",
  name: "世界教师日", nameEn: "World Teachers Day", icon: "🍎",
  date: "2026-10-05", month: 10, importance: "B", category: "festival",
  themeColor: "#dc2626",
  products: [
    { sku: "教师主题马克杯", skuEn: "Teacher Themed Mug", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "中", riskNote: "陶瓷易碎", keywords: ["teacher mug","teacher gift","best teacher"], sourcing: "1688: 教师节马克杯" },
    { sku: "感谢卡+小礼品套装", skuEn: "Thank You Card & Gift Set", category: "gift", costRange: "¥4-8", priceRange: "£5.99-7.99", margin: "约50-60%", matchScore: 5, riskLevel: "低", riskNote: "轻小件", keywords: ["teacher card","thank you gift","appreciation"], sourcing: "1688: 感谢卡套装" }
  ],
  validation: {
    googleTrends: ["teacher gift","teachers day uk"],
    amazonCheck: "BSR < 15000；小众节日",
    sourcing: "1688搜索：教师节礼物/马克杯",
    riskFlags: ["避免印刷具体学校logo"]
  }
},
{
  id: "mental-health-day-2026",
  name: "世界精神卫生日", nameEn: "World Mental Health Day", icon: "💚",
  date: "2026-10-10", month: 10, importance: "B", category: "trend",
  themeColor: "#10b981",
  products: [
    { sku: "励志手环套装", skuEn: "Inspirational Bracelet Set", category: "apparel", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 5, riskLevel: "低", riskNote: "轻小件", keywords: ["inspirational bracelet","mental health awareness"], sourcing: "1688: 励志手环" },
    { sku: "减压捏捏玩具", skuEn: "Stress Relief Squeeze Toy", category: "gift", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 5, riskLevel: "低", riskNote: "注意材质安全", keywords: ["stress ball","fidget toy","anxiety relief"], sourcing: "1688: 减压玩具" }
  ],
  validation: {
    googleTrends: ["mental health awareness","stress relief"],
    amazonCheck: "BSR < 15000；趋势上升",
    sourcing: "1688搜索：减压/励志/心理",
    riskFlags: ["避免医疗功效宣传"]
  }
},
{
  id: "boss-day-2026",
  name: "Boss's Day老板节", nameEn: "Boss's Day", icon: "👔",
  date: "2026-10-16", month: 10, importance: "B", category: "festival",
  themeColor: "#1e40af",
  products: [
    { sku: "老板主题幽默马克杯", skuEn: "Funny Boss Mug", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "中", riskNote: "陶瓷易碎", keywords: ["boss mug","boss gift","funny mug"], sourcing: "1688: 老板节杯子" },
    { sku: "桌面收纳套装", skuEn: "Desk Organizer Set", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约40-50%", matchScore: 3, riskLevel: "低", riskNote: "体积偏大", keywords: ["desk organizer","office accessories","boss gift"], sourcing: "1688: 桌面收纳" }
  ],
  validation: {
    googleTrends: ["boss day gift","boss gift uk"],
    amazonCheck: "BSR < 20000；小众",
    sourcing: "1688搜索：办公礼品/马克杯",
    riskFlags: ["小众节日控制首批量"]
  }
},
{
  id: "diwali-2026",
  name: "排灯节", nameEn: "Diwali", icon: "🪔",
  date: "2026-10-20", month: 10, importance: "B", category: "festival",
  themeColor: "#facc15",
  products: [
    { sku: "LED装饰灯串(暖白)", skuEn: "LED Diya String Lights", category: "decor", costRange: "¥8-15", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "UKCA认证", keywords: ["diwali lights","string lights","diya decor"], sourcing: "1688: LED灯串" },
    { sku: "排灯节主题贺卡套装", skuEn: "Diwali Card Set", category: "gift", costRange: "¥4-8", priceRange: "£5.99-7.99", margin: "约50-60%", matchScore: 4, riskLevel: "低", riskNote: "轻小件", keywords: ["diwali cards","festival cards"], sourcing: "1688: 排灯节贺卡" }
  ],
  validation: {
    googleTrends: ["diwali decorations","diwali gifts uk"],
    amazonCheck: "BSR < 10000；UK印度裔社区需求",
    sourcing: "1688搜索：排灯节/Diwali装饰",
    riskFlags: ["尊重宗教文化，避免不当元素"]
  }
},

// ===== 11月（6个）=====
{
  id: "bonfire-night-2026",
  name: "篝火节", nameEn: "Bonfire Night", icon: "🎆",
  date: "2026-11-05", month: 11, importance: "B", category: "festival",
  themeColor: "#ea580c",
  products: [
    { sku: "发光发光棒套装", skuEn: "Glow Stick Set", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 5, riskLevel: "中", riskNote: "化学发光产品需安全认证", keywords: ["glow sticks","bonfire night","light sticks"], sourcing: "1688: 荧光棒套装" },
    { sku: "保暖暖手宝", skuEn: "Reusable Hand Warmers", category: "home", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "中", riskNote: "注意材质安全", keywords: ["hand warmers","reusable warmers","bonfire night"], sourcing: "1688: 暖手宝" }
  ],
  validation: {
    googleTrends: ["bonfire night","guy fawkes"],
    amazonCheck: "BSR < 12000；11月初峰值",
    sourcing: "1688搜索：荧光棒/暖手宝",
    riskFlags: ["荧光棒化学安全认证","暖手宝材质检测"]
  }
},
{
  id: "remembrance-day-2026",
  name: "国殇纪念日", nameEn: "Remembrance Day", icon: "🌹",
  date: "2026-11-11", month: 11, importance: "B", category: "festival",
  themeColor: "#7f1d1d",
  products: [
    { sku: "虞美人主题胸针", skuEn: "Poppy Brooch Pin", category: "gift", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 4, riskLevel: "高", riskNote: "罂粟图案有使用规范，建议中性设计；避免政治符号", keywords: ["poppy pin","remembrance day","memorial brooch"], sourcing: "1688: 胸针定制" },
    { sku: "纪念主题木牌摆件", skuEn: "Memorial Wooden Sign", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约45-55%", matchScore: 3, riskLevel: "高", riskNote: "情绪敏感，文案需谨慎", keywords: ["remembrance","memorial decor","lest we forget"], sourcing: "1688: 木质摆件" }
  ],
  validation: {
    googleTrends: ["remembrance day","poppy appeal"],
    amazonCheck: "BSR < 15000；情绪敏感品类",
    sourcing: "1688搜索：纪念/胸针/木牌",
    riskFlags: ["⚠️高度敏感：罂粟图案使用受Royal British Legion规范","避免军事/政治符号","建议只做中性纪念类"]
  }
},
{
  id: "kindness-day-2026",
  name: "世界友善日", nameEn: "World Kindness Day", icon: "💛",
  date: "2026-11-13", month: 11, importance: "B", category: "trend",
  themeColor: "#eab308",
  products: [
    { sku: "友善主题贴纸套装", skuEn: "Kindness Sticker Set", category: "gift", costRange: "¥2-5", priceRange: "£5.99-7.99", margin: "约60-70%", matchScore: 5, riskLevel: "低", riskNote: "轻小件高毛利", keywords: ["kindness stickers","positive stickers","school stickers"], sourcing: "1688: 励志贴纸" },
    { sku: "友善主题手环", skuEn: "Kindness Bracelet", category: "apparel", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 5, riskLevel: "低", riskNote: "轻小件", keywords: ["kindness bracelet","be kind","awareness"], sourcing: "1688: 励志手环" }
  ],
  validation: {
    googleTrends: ["world kindness day","kindness gifts"],
    amazonCheck: "BSR < 20000；趋势型",
    sourcing: "1688搜索：友善/励志/贴纸",
    riskFlags: ["小众节日控制首批量"]
  }
},
{
  id: "mens-day-2026",
  name: "国际男人节", nameEn: "International Men's Day", icon: "👨",
  date: "2026-11-19", month: 11, importance: "A", category: "festival",
  themeColor: "#1d4ed8",
  products: [
    { sku: "男士主题马克杯", skuEn: "Men's Day Mug", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "中", riskNote: "陶瓷易碎", keywords: ["mens day mug","men gift","man mug"], sourcing: "1688: 男士马克杯" },
    { sku: "胡须护理套装", skuEn: "Beard Care Kit", category: "gift", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-50%", matchScore: 3, riskLevel: "中", riskNote: "化妆品类需成分合规", keywords: ["beard kit","beard care","mens grooming"], sourcing: "1688: 胡须护理套装" },
    { sku: "幽默领带/袜子", skuEn: "Funny Novelty Socks", category: "apparel", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "UK尺码", keywords: ["novelty socks","funny socks","mens socks"], sourcing: "1688: 搞怪袜子" },
    { sku: "多功能钥匙扣工具", skuEn: "Multi-tool Keychain", category: "home", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "轻小件实用", keywords: ["keychain tool","multi tool","edc keychain"], sourcing: "1688: 钥匙扣工具" }
  ],
  validation: {
    googleTrends: ["mens day gift","men gifts uk"],
    amazonCheck: "BSR < 10000；11月稳定需求",
    sourcing: "1688搜索：男士礼品/胡须/袜子",
    riskFlags: ["化妆品类注意UK成分合规"]
  }
},
{
  id: "black-friday-2026",
  name: "黑五", nameEn: "Black Friday", icon: "🛍️",
  date: "2026-11-27", month: 11, importance: "S", category: "activity",
  themeColor: "#111827",
  products: [
    { sku: "保暖手套(触屏)", skuEn: "Touch Screen Winter Gloves", category: "apparel", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "UK尺码", keywords: ["touch screen gloves","winter gloves","warm gloves"], sourcing: "1688: 触屏手套" },
    { sku: "LED化妆镜", skuEn: "LED Vanity Mirror", category: "home", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-50%", matchScore: 4, riskLevel: "中", riskNote: "电子+易碎", keywords: ["led mirror","vanity mirror","makeup mirror"], sourcing: "1688: LED化妆镜" },
    { sku: "加湿器(便携)", skuEn: "Portable Humidifier", category: "home", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 4, riskLevel: "中", riskNote: "电子UKCA认证", keywords: ["humidifier","portable humidifier","desk humidifier"], sourcing: "1688: 便携加湿器" },
    { sku: "蓝牙耳机包/收纳", skuEn: "Earbuds Case Pouch", category: "gift", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约55-65%", matchScore: 5, riskLevel: "低", riskNote: "轻小件", keywords: ["earbuds case","airpods pouch","storage case"], sourcing: "1688: 耳机收纳包" },
    { sku: "冬季围巾", skuEn: "Winter Scarf", category: "apparel", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "低", riskNote: "UK尺码", keywords: ["winter scarf","warm scarf","mens scarf"], sourcing: "1688: 冬季围巾" },
    { sku: "手机支架(床头)", skuEn: "Phone Bed Stand Holder", category: "home", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "实用刚需", keywords: ["phone stand","bed stand","phone holder"], sourcing: "1688: 手机支架" },
    { sku: "保温杯(不锈钢)", skuEn: "Insulated Tumbler", category: "home", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-50%", matchScore: 4, riskLevel: "中", riskNote: "食品级认证", keywords: ["tumbler","insulated cup","thermal flask"], sourcing: "1688: 保温杯" },
    { sku: "桌面小夜灯", skuEn: "Desk Night Light", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "中", riskNote: "电子UKCA", keywords: ["night light","desk lamp","bedroom light"], sourcing: "1688: 小夜灯" },
    { sku: "旅行收纳袋套装", skuEn: "Travel Organizer Bags Set", category: "gift", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "实用", keywords: ["travel organizer","packing cubes","luggage bags"], sourcing: "1688: 旅行收纳套装" },
    { sku: "宠物玩具套装", skuEn: "Pet Toy Set", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "注意宠物安全材质", keywords: ["dog toys","pet toys","chew toys"], sourcing: "1688: 宠物玩具" },
    { sku: "厨房收纳架", skuEn: "Kitchen Organizer Rack", category: "home", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 3, riskLevel: "低", riskNote: "体积偏大", keywords: ["kitchen organizer","storage rack","pantry organizer"], sourcing: "1688: 厨房收纳" },
    { sku: "圣诞预热装饰小件", skuEn: "Mini Christmas Decorations", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 5, riskLevel: "低", riskNote: "衔接圣诞", keywords: ["mini christmas","small decor","tree ornaments"], sourcing: "1688: 迷你圣诞装饰" }
  ],
  validation: {
    googleTrends: ["black friday deals 2026","gift ideas uk"],
    amazonCheck: "BSR < 2000；全年最大流量节点",
    sourcing: "1688搜索：冬季/电子/家居/礼品",
    riskFlags: ["电子产品需UKCA认证","竞争最激烈需早备货","食品接触材质认证"]
  }
},
{
  id: "cyber-monday-2026",
  name: "网一", nameEn: "Cyber Monday", icon: "💻",
  date: "2026-11-30", month: 11, importance: "A", category: "activity",
  themeColor: "#4338ca",
  products: [
    { sku: "笔记本内胆包", skuEn: "Laptop Sleeve Case", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "低", riskNote: "尺寸适配UK常见型号", keywords: ["laptop sleeve","laptop case","notebook bag"], sourcing: "1688: 笔记本内胆包" },
    { sku: "USB扩展坞/集线器", skuEn: "USB Hub Splitter", category: "home", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 3, riskLevel: "中", riskNote: "电子UKCA认证；品牌敏感", keywords: ["usb hub","usb splitter","data hub"], sourcing: "1688: USB集线器" },
    { sku: "键盘清洁套装", skuEn: "Keyboard Cleaning Kit", category: "home", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 5, riskLevel: "低", riskNote: "轻小件实用", keywords: ["keyboard cleaner","cleaning kit","pc accessories"], sourcing: "1688: 键盘清洁套装" },
    { sku: "手机壳套装(3只)", skuEn: "Phone Case Set 3 Pack", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约50-55%", matchScore: 4, riskLevel: "低", riskNote: "注意适配新机型", keywords: ["phone case","phone cover","smartphone case"], sourcing: "1688: 手机壳套装" }
  ],
  validation: {
    googleTrends: ["cyber monday deals","tech accessories uk"],
    amazonCheck: "BSR < 5000；与黑五衔接",
    sourcing: "1688搜索：数码配件/电脑周边",
    riskFlags: ["电子产品需UKCA认证","注意品牌专利（手机型号）"]
  }
},

// ===== 12月（3个）=====
{
  id: "christmas-2026",
  name: "圣诞节", nameEn: "Christmas", icon: "🎄",
  date: "2026-12-25", month: 12, importance: "S", category: "festival",
  themeColor: "#dc2626",
  products: [
    { sku: "圣诞树挂件套装(24只)", skuEn: "Christmas Tree Ornaments 24 Pack", category: "decor", costRange: "¥8-15", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "常规品类", keywords: ["christmas ornaments","tree decorations","baubles"], sourcing: "1688: 圣诞挂件套装" },
    { sku: "圣诞主题LED串灯", skuEn: "Christmas LED String Lights", category: "decor", costRange: "¥8-15", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 5, riskLevel: "低", riskNote: "UKCA认证", keywords: ["christmas lights","string lights","outdoor christmas"], sourcing: "1688: 圣诞LED灯串" },
    { sku: "圣诞袜(悬挂装饰)", skuEn: "Christmas Stocking Set", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 5, riskLevel: "低", riskNote: "常规品类", keywords: ["christmas stocking","hanging stocking","xmas decor"], sourcing: "1688: 圣诞袜" },
    { sku: "圣诞主题礼品袋套装", skuEn: "Christmas Gift Bags Set", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约50-55%", matchScore: 5, riskLevel: "低", riskNote: "刚需高销量", keywords: ["gift bags","christmas bags","present bags"], sourcing: "1688: 圣诞礼品袋" },
    { sku: "圣诞主题包装纸套装", skuEn: "Christmas Wrapping Paper Set", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约50-55%", matchScore: 5, riskLevel: "低", riskNote: "刚需", keywords: ["wrapping paper","christmas wrap","gift wrap"], sourcing: "1688: 圣诞包装纸" },
    { sku: "圣诞主题马克杯套装", skuEn: "Christmas Mug Set", category: "home", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-50%", matchScore: 4, riskLevel: "中", riskNote: "陶瓷易碎", keywords: ["christmas mug","xmas mug","festive cup"], sourcing: "1688: 圣诞马克杯" },
    { sku: "圣诞主题抱枕套", skuEn: "Christmas Pillow Cover", category: "home", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 5, riskLevel: "低", riskNote: "枕套非枕头", keywords: ["pillow cover","christmas pillow","xmas decor"], sourcing: "1688: 圣诞抱枕套" },
    { sku: "圣诞主题餐垫套装", skuEn: "Christmas Placemat Set", category: "home", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["christmas placemats","table decor","xmas dining"], sourcing: "1688: 圣诞餐垫" },
    { sku: "圣诞主题餐具套装", skuEn: "Christmas Tableware Set", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "中", riskNote: "食品接触材质", keywords: ["christmas tableware","paper plates","party supplies"], sourcing: "1688: 圣诞餐具" },
    { sku: "圣诞帽套装(6只)", skuEn: "Santa Hat Set 6 Pack", category: "apparel", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约50-55%", matchScore: 5, riskLevel: "低", riskNote: "派对刚需", keywords: ["santa hat","christmas hat","party hat"], sourcing: "1688: 圣诞帽" },
    { sku: "驯鹿发箍套装", skuEn: "Reindeer Headband Set", category: "apparel", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 5, riskLevel: "低", riskNote: "派对配件", keywords: ["reindeer headband","christmas accessory","party headband"], sourcing: "1688: 驯鹿发箍" },
    { sku: "圣诞老人胡子道具套装", skuEn: "Santa Beard Prop Set", category: "apparel", costRange: "¥4-8", priceRange: "£5.99-7.99", margin: "约55-60%", matchScore: 5, riskLevel: "低", riskNote: "轻小件", keywords: ["santa beard","christmas prop","party accessory"], sourcing: "1688: 圣诞老人胡子" },
    { sku: "圣诞主题门环装饰", skuEn: "Christmas Wreath", category: "decor", costRange: "¥10-20", priceRange: "£8.99-9.99", margin: "约40-45%", matchScore: 3, riskLevel: "低", riskNote: "体积大注意FBA", keywords: ["christmas wreath","door wreath","xmas decor"], sourcing: "1688: 圣诞门环" },
    { sku: "圣诞日历盲盒(玩具)", skuEn: "Advent Calendar Toys", category: "gift", costRange: "¥10-18", priceRange: "£8.99-9.99", margin: "约40-50%", matchScore: 4, riskLevel: "中", riskNote: "儿童玩具安全认证", keywords: ["advent calendar","christmas countdown","toy calendar"], sourcing: "1688: 圣诞日历" },
    { sku: "圣诞主题蜡烛套装", skuEn: "Christmas Candle Set", category: "home", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "中", riskNote: "蜡烛安全标签要求", keywords: ["christmas candles","scented candles","festive candle"], sourcing: "1688: 圣诞蜡烛" }
  ],
  validation: {
    googleTrends: ["christmas decorations 2026","christmas gifts uk"],
    amazonCheck: "BSR < 1500；全年最大消费节点",
    sourcing: "1688搜索：圣诞装饰/礼品/派对",
    riskFlags: ["⚠️最大节点需最早备货(T-55约10月底)","避免IP侵权(Disney等)","电子需UKCA","蜡烛/儿童玩具有安全要求"]
  }
},
{
  id: "tree-dressing-day-2026",
  name: "National Tree Dressing Day", nameEn: "Tree Dressing Day", icon: "🌳",
  date: "2026-12-05", month: 12, importance: "A", category: "festival",
  themeColor: "#15803d",
  products: [
    { sku: "户外装饰彩带", skuEn: "Outdoor Decorative Ribbon", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["tree ribbon","outdoor ribbon","decorative ribbon"], sourcing: "1688: 装饰彩带" },
    { sku: "木质挂饰套装", skuEn: "Wooden Hanging Ornament Set", category: "decor", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "与圣诞衔接", keywords: ["wooden ornaments","tree decorations","hanging decor"], sourcing: "1688: 木质挂饰" }
  ],
  validation: {
    googleTrends: ["tree dressing day","outdoor decorations"],
    amazonCheck: "BSR < 15000；小众与圣诞衔接",
    sourcing: "1688搜索：彩带/木质挂饰",
    riskFlags: ["与圣诞备货可合并"]
  }
},
{
  id: "boxing-week-2026",
  name: "节礼周", nameEn: "Boxing Week (Boxing Day + NYE)", icon: "🎊",
  date: "2026-12-26", month: 12, importance: "A", category: "activity",
  themeColor: "#7c3aed",
  products: [
    { sku: "新年主题装饰套装", skuEn: "New Year Decoration Set", category: "decor", costRange: "¥8-15", priceRange: "£7.99-9.99", margin: "约45-50%", matchScore: 4, riskLevel: "低", riskNote: "常规品类", keywords: ["new year decorations","2027 decor","nye party"], sourcing: "1688: 新年装饰" },
    { sku: "派对popper套装", skuEn: "Party Poppers Set", category: "decor", costRange: "¥5-10", priceRange: "£6.49-8.49", margin: "约50-55%", matchScore: 4, riskLevel: "中", riskNote: "烟花类产品安全认证", keywords: ["party poppers","new year","celebration"], sourcing: "1688: 派对popper" },
    { sku: "新年主题眼镜道具", skuEn: "New Year Glasses Props", category: "apparel", costRange: "¥3-6", priceRange: "£5.99-7.99", margin: "约60-70%", matchScore: 5, riskLevel: "低", riskNote: "轻小件", keywords: ["new year glasses","2027 glasses","party props"], sourcing: "1688: 新年眼镜" },
    { sku: "清仓特价包装礼品", skuEn: "Clearance Gift Sets", category: "gift", costRange: "¥6-12", priceRange: "£6.99-8.99", margin: "约45-55%", matchScore: 4, riskLevel: "低", riskNote: "消化圣诞余货", keywords: ["gift sets","sale items","clearance"], sourcing: "1688: 礼品套装" }
  ],
  validation: {
    googleTrends: ["boxing day sale","new year decorations"],
    amazonCheck: "BSR < 5000；清仓+新年衔接",
    sourcing: "1688搜索：新年装饰/派对用品",
    riskFlags: ["烟花类需安全认证","主要消化圣诞库存"]
  }
}

]; // FESTIVALS结束
```

- [ ] **Step 5: 验证完整数据**

浏览器控制台：
- `FESTIVALS.length` → 预期 `23`
- `FESTIVALS.filter(f => f.importance === "S").length` → 预期 `3`（万圣节/黑五/圣诞）
- `FESTIVALS.filter(f => f.month === 12).length` → 预期 `3`
- `FESTIVALS.find(f => f.id === "halloween-2026").products.length` → 预期 `12`

- [ ] **Step 6: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: 10-12月节日数据(14个节点)+完整23节日数据集 (Task 3b)"
```

---

## Task 4: State状态管理（localStorage）

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块（FESTIVALS后追加）

- [ ] **Step 1: 写入State管理模块**

在 `]; // FESTIVALS结束` 之后追加：

```js
// ============================================
// State：localStorage 状态管理
// ============================================
const State = {
  data: null,

  // 加载（首次访问初始化空状态）
  load() {
    const raw = localStorage.getItem(CONFIG.storageKey);
    if (raw) {
      try {
        this.data = JSON.parse(raw);
      } catch (e) {
        console.warn("存储数据损坏，重置为空", e);
        this.init();
      }
    } else {
      this.init();
    }
    return this.data;
  },

  // 初始化空结构
  init() {
    this.data = {
      version: "1.0",
      lastUpdated: new Date().toISOString(),
      festivals: {}
    };
    this.save();
  },

  // 保存
  save() {
    this.data.lastUpdated = new Date().toISOString();
    localStorage.setItem(CONFIG.storageKey, JSON.stringify(this.data));
  },

  // 获取某节日状态（不存在则返回默认）
  getFestival(festivalId) {
    if (!this.data.festivals[festivalId]) {
      this.data.festivals[festivalId] = {
        status: "none",
        logistics: CONFIG.defaultLogistics,
        milestones: {
          selection: false,
          airArrival: false,
          truckShip: false,
          arrival: false,
          festival: false
        },
        notes: "",
        selectedSkus: []
      };
    }
    return this.data.festivals[festivalId];
  },

  // 更新某节日状态
  updateFestival(festivalId, updates) {
    const fest = this.getFestival(festivalId);
    Object.assign(fest, updates);
    this.save();
  },

  // 切换里程碑
  toggleMilestone(festivalId, milestoneId) {
    const fest = this.getFestival(festivalId);
    fest.milestones[milestoneId] = !fest.milestones[milestoneId];
    this.save();
  },

  // 切换SKU选中
  toggleSku(festivalId, skuName) {
    const fest = this.getFestival(festivalId);
    const idx = fest.selectedSkus.indexOf(skuName);
    if (idx >= 0) {
      fest.selectedSkus.splice(idx, 1);
    } else {
      fest.selectedSkus.push(skuName);
    }
    this.save();
  },

  // 导出
  export() {
    return JSON.stringify(this.data, null, 2);
  },

  // 导入
  import(jsonStr) {
    const parsed = JSON.parse(jsonStr);
    if (!parsed.version || !parsed.festivals) {
      throw new Error("文件格式不正确");
    }
    this.data = parsed;
    this.save();
  },

  // 重置
  reset() {
    localStorage.removeItem(CONFIG.storageKey);
    this.init();
  }
};

// 当前筛选状态（非持久化）
const Filter = {
  category: "",
  month: "",
  urgency: "",
  status: "",
  search: "",
  statCardUrgency: ""  // 点击统计卡片设置的紧急度筛选
};
```

- [ ] **Step 2: 验证State加载**

浏览器控制台：
```js
State.load();
State.getFestival("halloween-2026");
```
预期返回包含 `status: "none"` 的默认对象。
刷新页面后 `State.data.festivals["halloween-2026"]` 应仍存在（localStorage持久化）。

- [ ] **Step 3: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: State状态管理模块(localStorage读写) (Task 4)"
```

---

## Task 5: 工具函数（日期计算、紧急度判定）

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块

- [ ] **Step 1: 写入Utils工具函数**

在 `Filter` 对象之后追加：

```js
// ============================================
// Utils：工具函数（日期、紧急度、渲染辅助）
// ============================================
const Utils = {
  // 解析日期字符串为Date对象
  parseDate(str) {
    return new Date(str + "T00:00:00");
  },

  // 今日（去掉时分秒）
  today() {
    const d = new Date();
    d.setHours(0, 0, 0, 0);
    return d;
  },

  // 日期差（返回相差天数，a - b）
  diffDays(a, b) {
    return Math.round((a - b) / 86400000);
  },

  // 格式化日期为 MM/DD
  fmtMD(date) {
    const m = date.getMonth() + 1;
    const d = date.getDate();
    return `${m}/${d}`;
  },

  // 格式化完整日期 YYYY-MM-DD
  fmtYMD(date) {
    const y = date.getFullYear();
    const m = String(date.getMonth() + 1).padStart(2, "0");
    const d = String(date.getDate()).padStart(2, "0");
    return `${y}-${m}-${d}`;
  },

  // 根据物流方式获取选品截止日（T-(leadTime+14)）
  getSelectionDeadline(festivalDate, logistics) {
    const mode = CONFIG.logisticsModes[logistics];
    const deadline = new Date(this.parseDate(festivalDate));
    deadline.setDate(deadline.getDate() - (mode.leadTime + 14));
    return deadline;
  },

  // 获取某节日在指定物流下的5个里程碑日期
  getMilestoneDates(festival) {
    const festState = State.getFestival(festival.id);
    const logistics = festState.logistics;
    const mode = CONFIG.logisticsModes[logistics];
    const fDate = this.parseDate(festival.date);
    // 基准点：节日 - leadTime（即大货到仓的最早理论点）
    // 各里程碑相对此基准偏移
    return CONFIG.milestones.map(ms => {
      const date = new Date(fDate);
      // tMinusLead: 相对 (fDate - leadTime) 的偏移天数
      // selection 在 leadTime 前的14天 → fDate - leadTime - 14
      // airArrival: fDate - leadTime + 8 ... 实际换算: fDate - (leadTime - tMinusLead)
      date.setDate(date.getDate() - (mode.leadTime - ms.tMinusLead));
      return {
        ...ms,
        date: date,
        dateStr: this.fmtMD(date)
      };
    });
  },

  // 判定紧急度
  getUrgency(festival) {
    const festState = State.getFestival(festival.id);
    const fDate = this.parseDate(festival.date);
    const today = this.today();

    // 节日已过
    if (fDate < today) return "past";

    // 节日今天或已开始（但未结束处理）
    const deadline = this.getSelectionDeadline(festival.date, festState.logistics);
    const daysToDeadline = this.diffDays(deadline, today);

    if (daysToDeadline < 0) {
      // 截止日已过，但若已完成选品则不算紧急
      if (festState.milestones.selection) return "plan";  // 已选品，转为正常推进
      return "urgent";
    } else if (daysToDeadline <= CONFIG.urgencyThresholds.week) {
      return "week";
    } else if (daysToDeadline <= CONFIG.urgencyThresholds.month) {
      return "month";
    } else {
      return "plan";
    }
  },

  // 紧急度中文标签
  urgencyLabel(urgency) {
    const map = { urgent: "🔴紧急", week: "🟠本周启动", month: "🟡本月备货", plan: "🟢规划中", past: "⚫已过" };
    return map[urgency] || urgency;
  },

  // 生成星级HTML
  stars(score) {
    return "★".repeat(score) + "☆".repeat(5 - score);
  },

  // HTML转义
  escape(str) {
    const div = document.createElement("div");
    div.textContent = str;
    return div.innerHTML;
  }
};
```

- [ ] **Step 2: 验证工具函数**

浏览器控制台：
```js
const h = FESTIVALS.find(f => f.id === "halloween-2026");
Utils.getUrgency(h);  // 预期 "month" 或 "plan"（取决于今日）
Utils.getSelectionDeadline("2026-10-31", "truck");
// 预期：2026-10-31 往前推 35+14=49天 → 约 2026-09-12
Utils.getMilestoneDates(h).length;  // 预期 5
```

- [ ] **Step 3: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: Utils工具函数-日期/紧急度/里程碑计算 (Task 5)"
```

---

## Task 6: 渲染 - 紧急度看板 + 月份导航

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块

- [ ] **Step 1: 写入渲染模块（看板部分）**

在 `Utils` 对象之后追加：

```js
// ============================================
// Render：渲染函数
// ============================================
const Render = {

  // 渲染月份导航
  monthNav() {
    const nav = document.getElementById("monthNav");
    nav.innerHTML = CONFIG.months.map(m =>
      `<a href="#month-${m.num}">${m.label}</a>`
    ).join("");
  },

  // 渲染紧急度看板
  dashboard() {
    const today = Utils.today();
    const countdown = document.getElementById("countdown");

    // 统计各紧急度数量
    const stats = { urgent: 0, week: 0, month: 0, plan: 0, past: 0 };
    FESTIVALS.forEach(f => {
      stats[Utils.getUrgency(f)]++;
    });

    // 找最近一个需备货的节日
    const upcoming = FESTIVALS
      .filter(f => Utils.getUrgency(f) !== "past")
      .map(f => ({
        fest: f,
        deadline: Utils.getSelectionDeadline(f.date, State.getFestival(f.id).logistics)
      }))
      .sort((a, b) => a.deadline - b.deadline)[0];

    if (upcoming) {
      const days = Utils.diffDays(upcoming.deadline, today);
      const festState = State.getFestival(upcoming.fest.id);
      const u = Utils.getUrgency(upcoming.fest);
      countdown.innerHTML = `今日 <strong>${Utils.fmtYMD(today)}</strong> · 最近备货节点：
        <strong>${upcoming.fest.icon} ${upcoming.fest.name}</strong>（${upcoming.fest.date}）
        · 选品截止 <strong>${Utils.fmtYMD(upcoming.deadline)}</strong>
        · <span class="badge ${u}">${Utils.urgencyLabel(u)}</span>`;
    }

    // 统计卡片
    const cards = [
      { key: "urgent", cls: "urgent", num: stats.urgent, label: "🔴 紧急（已过截止）" },
      { key: "week", cls: "week", num: stats.week, label: "🟠 本周必须启动" },
      { key: "month", cls: "month", num: stats.month, label: "🟡 本月需备货" },
      { key: "plan", cls: "plan", num: stats.plan, label: "🟢 规划观察中" }
    ];
    document.getElementById("statCards").innerHTML = cards.map(c =>
      `<div class="stat-card ${c.cls}" onclick="Filter.statCardUrgency='${c.key}';Filter.search='';document.getElementById('filterSearch').value='';Interact.applyFilter();">
        <div class="num">${c.num}</div>
        <div class="label">${c.label}</div>
      </div>`
    ).join("");
  }
};
```

- [ ] **Step 2: 调用渲染并验证**

在 `<script>` 末尾（所有定义之后）追加初始化调用：

```js
// ============================================
// 初始化
// ============================================
State.load();
Render.monthNav();
Render.dashboard();
```

浏览器刷新。预期：
- 顶部导航显示7-12月链接
- 看板显示今日日期、最近备货节日、4个统计卡片（有数字）

- [ ] **Step 3: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: 渲染紧急度看板和月份导航 (Task 6)"
```

---

## Task 7: 渲染 - 节日卡片主体

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块

- [ ] **Step 1: 在 Render 对象内追加 main() 方法**

找到 `Render = {` 对象，在 `dashboard()` 方法后（`dashboard` 的闭合 `},` 之后）追加：

```js
  // 渲染主区域（节日卡片，按月份分组）
  main() {
    const mainEl = document.getElementById("main");

    // 应用筛选
    const filtered = FESTIVALS.filter(f => {
      const festState = State.getFestival(f.id);
      const urgency = Utils.getUrgency(f);

      // 品类筛选
      if (Filter.category && !f.products.some(p => p.category === Filter.category)) return false;
      // 月份筛选
      if (Filter.month && f.month !== parseInt(Filter.month)) return false;
      // 紧急度筛选（统计卡片优先）
      const effUrgency = Filter.statCardUrgency || Filter.urgency;
      if (effUrgency && urgency !== effUrgency) return false;
      // 状态筛选
      if (Filter.status && festState.status !== Filter.status) return false;
      // 搜索
      if (Filter.search) {
        const q = Filter.search.toLowerCase();
        const haystack = (f.name + f.nameEn + f.icon +
          f.products.map(p => p.sku + p.skuEn + p.keywords.join("")).join("")).toLowerCase();
        if (!haystack.includes(q)) return false;
      }
      return true;
    });

    if (filtered.length === 0) {
      mainEl.innerHTML = `<div class="empty-state">没有匹配的节日，试试调整筛选条件。</div>`;
      return;
    }

    // 按月份分组
    const byMonth = {};
    filtered.forEach(f => {
      if (!byMonth[f.month]) byMonth[f.month] = [];
      byMonth[f.month].push(f);
    });

    // 渲染各月份
    let html = "";
    CONFIG.months.forEach(m => {
      if (!byMonth[m.num]) return;
      const fests = byMonth[m.num].sort((a, b) => new Date(a.date) - new Date(b.date));
      html += `<section class="month-section" id="month-${m.num}">
        <h2>${m.label} <span class="count">(${fests.length}个节点)</span></h2>
        <div class="cards-grid">
          ${fests.map(f => this.festivalCard(f)).join("")}
        </div>
      </section>`;
    });

    mainEl.innerHTML = html;
    // 绑定卡片交互
    Interact.bindCards();
  },

  // 渲染单张节日卡片
  festivalCard(f) {
    const festState = State.getFestival(f.id);
    const urgency = Utils.getUrgency(f);
    const deadline = Utils.getSelectionDeadline(f.date, festState.logistics);
    const daysToDeadline = Utils.diffDays(deadline, Utils.today());

    // 默认展开S级
    const expanded = f.importance === "S" ? "expanded" : "";

    let header = `
      <div class="card-header" onclick="Interact.toggleCard('${f.id}')">
        <div>
          <div class="card-title">
            <span class="icon">${f.icon}</span>
            <span>${f.name}</span>
            <span class="name-en">${f.nameEn}</span>
            ${f.importance === "S" ? '<span class="badge importance-S">S级</span>' : ""}
            <span class="badge ${urgency}">${Utils.urgencyLabel(urgency)}</span>
          </div>
          <div class="card-meta">
            ${f.date} | ${f.category === "festival" ? "节日" : f.category === "activity" ? "活动" : "趋势"}
            | 选品截止 ${Utils.fmtYMD(deadline)}（${daysToDeadline >= 0 ? "剩" + daysToDeadline + "天" : "已过" + (-daysToDeadline) + "天"}）
          </div>
        </div>
        <div>${expanded ? "▼" : "▶"}</div>
      </div>
    `;

    let body = `
      <div class="card-body">
        ${this.logisticsToggle(f.id, festState.logistics)}
        ${this.timeline(f)}
        ${this.products(f)}
        ${this.validation(f)}
        ${this.controls(f)}
      </div>
    `;

    return `<div class="festival-card ${urgency} ${expanded}" id="card-${f.id}" style="border-left-color:${f.themeColor}">
      ${header}${body}
    </div>`;
  }
```

- [ ] **Step 2: 占位辅助方法（后续Task填充，先返回空避免报错）**

在 `Render` 对象内，`festivalCard()` 方法之后追加占位方法：

```js
  ,

  // 以下方法在后续Task实现，先占位避免报错
  logisticsToggle(id, logistics) { return `<div class="card-controls"></div>`; },
  timeline(f) { return ""; },
  products(f) { return ""; },
  validation(f) { return ""; },
  controls(f) { return ""; }
```

- [ ] **Step 3: 在初始化区追加调用**

修改初始化区（在 `Render.dashboard();` 后追加）：

```js
Render.main();
```

- [ ] **Step 4: 添加 Interact 空对象占位（后续Task填充）**

在 `Render` 对象之后追加：

```js
// ============================================
// Interact：交互绑定（后续Task填充）
// ============================================
const Interact = {
  toggleCard(id) { /* Task 8 */ },
  bindCards() { /* Task 8 */ },
  applyFilter() { /* Task 8 */ }
};
```

- [ ] **Step 5: 验证渲染**

浏览器刷新。预期：
- 6个月份区块出现，各显示对应节日卡片
- S级卡片（万圣节/黑五/圣诞）默认展开，显示卡片头（标题/紧急度/截止日）
- 点击卡片头可展开/折叠（toggleCard暂空，展开折叠先靠CSS的expanded类，后续Task 8实现切换）

注意：toggleCard暂未实现，点击无反应是正常的，下个Task实现。

- [ ] **Step 6: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: 渲染节日卡片主体+筛选逻辑框架 (Task 7)"
```

---

## Task 8: 交互 - 卡片展开/折叠 + 筛选应用

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块

- [ ] **Step 1: 实现 Interact 对象的卡片与筛选交互**

替换 `Interact` 占位对象为完整实现：

```js
// ============================================
// Interact：交互绑定
// ============================================
const Interact = {
  // 展开/折叠卡片
  toggleCard(id) {
    const card = document.getElementById("card-" + id);
    if (card) {
      card.classList.toggle("expanded");
      // 更新箭头
      const arrow = card.querySelector(".card-header > div:last-child");
      if (arrow) arrow.textContent = card.classList.contains("expanded") ? "▼" : "▶";
    }
  },

  // 绑定卡片内动态元素（渲染后调用）
  bindCards() {
    // 卡片头点击已在onclick处理，此处预留
  },

  // 应用筛选（重新渲染main）
  applyFilter() {
    Render.main();
  },

  // 重置筛选
  resetFilter() {
    Filter.category = "";
    Filter.month = "";
    Filter.urgency = "";
    Filter.status = "";
    Filter.search = "";
    Filter.statCardUrgency = "";
    document.getElementById("filterCategory").value = "";
    document.getElementById("filterMonth").value = "";
    document.getElementById("filterUrgency").value = "";
    document.getElementById("filterStatus").value = "";
    document.getElementById("filterSearch").value = "";
    Render.main();
  }
};

// 绑定筛选栏事件（初始化时调用）
function bindFilters() {
  document.getElementById("filterCategory").addEventListener("change", e => {
    Filter.category = e.target.value;
    Filter.statCardUrgency = "";  // 切换筛选时清除统计卡片筛选
    Render.main();
  });
  document.getElementById("filterMonth").addEventListener("change", e => {
    Filter.month = e.target.value;
    Render.main();
  });
  document.getElementById("filterUrgency").addEventListener("change", e => {
    Filter.urgency = e.target.value;
    Filter.statCardUrgency = "";
    Render.main();
  });
  document.getElementById("filterStatus").addEventListener("change", e => {
    Filter.status = e.target.value;
    Render.main();
  });
  document.getElementById("filterSearch").addEventListener("input", e => {
    Filter.search = e.target.value;
    Render.main();
  });
  document.getElementById("resetFilter").addEventListener("click", () => {
    Interact.resetFilter();
  });
}
```

- [ ] **Step 2: 在初始化区追加筛选绑定**

在初始化区（`Render.main();` 之后）追加：

```js
bindFilters();
```

- [ ] **Step 3: 验证交互**

浏览器刷新。测试：
- 点击卡片头 → 展开/折叠（箭头变化，body显隐）
- 月份下拉选"10月" → 只显示10月节日
- 品类下拉选"🎃装饰" → 只显示含装饰类SKU的节日
- 搜索框输入"圣诞" → 只显示圣诞节
- 点击"重置" → 恢复全部

- [ ] **Step 4: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: 卡片展开折叠+筛选交互 (Task 8)"
```

---

## Task 9: 渲染 - 物流切换器 + 时间线

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块

- [ ] **Step 1: 实现 Render.logisticsToggle()**

替换 Render 对象内的占位：

```js
  logisticsToggle(id, logistics) {
    const buttons = LOGISTICS_OPTIONS.map(opt =>
      `<button class="${opt.id === logistics ? "active" : ""}"
        onclick="Interact.switchLogistics('${id}','${opt.id}')">${opt.icon} ${opt.label}</button>`
    ).join("");
    return `<div class="card-controls">
      <label>物流方式：</label>
      <div class="logistics-toggle">${buttons}</div>
    </div>`;
  },
```

- [ ] **Step 2: 实现 Render.timeline()**

替换占位：

```js
  timeline(f) {
    const festState = State.getFestival(f.id);
    const milestones = Utils.getMilestoneDates(f);
    const nodes = milestones.map(ms => {
      const done = festState.milestones[ms.id];
      return `<div class="timeline-node">
        <div class="timeline-dot ${done ? "done" : ""}"
          onclick="Interact.toggleMilestone('${f.id}','${ms.id}')" title="${ms.actions}"></div>
        <div class="ms-name">${ms.name}</div>
        <div class="ms-date">${ms.dateStr}</div>
      </div>`;
    }).join("");
    return `<div class="timeline">
      <div style="font-size:12px;color:var(--text-muted);margin-bottom:4px">
        📅 备货时间线（${CONFIG.logisticsModes[festState.logistics].label}，总提前期${CONFIG.logisticsModes[festState.logistics].leadTime}天）
      </div>
      <div class="timeline-track">${nodes}</div>
      <div style="font-size:11px;color:var(--text-muted);margin-top:6px">
        💡 策略：空运首批50件测款 + 卡航大货同时下单。点击圆点标记完成。
      </div>
    </div>`;
  },
```

- [ ] **Step 3: 实现 Interact 切换物流和里程碑**

在 Interact 对象内（`resetFilter()` 之后）追加：

```js
  ,

  // 切换物流方式
  switchLogistics(festivalId, logistics) {
    State.updateFestival(festivalId, { logistics });
    Render.main();
    Render.dashboard();  // 物流变化影响紧急度，更新看板
  },

  // 切换里程碑完成状态
  toggleMilestone(festivalId, milestoneId) {
    State.toggleMilestone(festivalId, milestoneId);
    Render.main();
    Render.dashboard();
  }
```

- [ ] **Step 4: 验证时间线**

浏览器刷新，展开一个节日卡片。预期：
- 看到物流切换器（3个按钮：海运/卡航/空运，当前物流高亮）
- 点击"海运" → 时间线日期变化（变早），卡片紧急度可能变化
- 看到5个里程碑节点（圆点 + 名称 + 日期）
- 点击圆点 → 变绿（done状态），再点恢复
- 顶部看板数字可能随之变化

- [ ] **Step 5: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: 物流切换器+备货时间线渲染 (Task 9)"
```

---

## Task 10: 渲染 - 选品表 + 验证指引 + 控制区

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块

- [ ] **Step 1: 实现 Render.products()**

替换占位：

```js
  products(f) {
    const festState = State.getFestival(f.id);
    // 品类标签
    const cats = ["decor", "gift", "apparel", "home"];
    const tabs = `<div class="product-cat-tabs" id="tabs-${f.id}">
      <div class="product-cat-tab active" onclick="Interact.filterProductCat('${f.id}','')">${f.products.length}全部</div>
      ${cats.filter(c => f.products.some(p => p.category === c)).map(c => {
        const count = f.products.filter(p => p.category === c).length;
        return `<div class="product-cat-tab" onclick="Interact.filterProductCat('${f.id}','${c}')">${CONFIG.categories[c].label} ${count}</div>`;
      }).join("")}
    </div>`;

    // 表格行
    const rows = f.products.map(p => {
      const selected = festState.selectedSkus.includes(p.sku);
      return `<tr class="prod-row" data-cat="${p.category}">
        <td><input type="checkbox" ${selected ? "checked" : ""} onchange="Interact.toggleSku('${f.id}','${Utils.escape(p.sku)}')" title="标记要做"></td>
        <td><strong>${p.sku}</strong><br><span style="color:var(--text-muted);font-size:11px">${p.skuEn}</span></td>
        <td>${p.costRange}</td>
        <td><strong>${p.priceRange}</strong></td>
        <td style="color:var(--text-muted)">${p.margin}</td>
        <td><span class="stars">${Utils.stars(p.matchScore)}</span></td>
        <td><span class="risk risk-${p.riskLevel === "低" ? "low" : p.riskLevel === "中" ? "mid" : "high"}" title="${Utils.escape(p.riskNote)}">${p.riskLevel}</span></td>
      </tr>`;
    }).join("");

    return `<div class="products-section">
      <h4>🛒 选品清单（${f.products.length}个SKU建议）</h4>
      ${tabs}
      <table class="product-table">
        <thead><tr>
          <th></th><th>SKU</th><th>1688拿货</th><th>建议售价</th><th>毛利率</th><th>匹配度</th><th>风险</th>
        </tr></thead>
        <tbody>${rows}</tbody>
      </table>
    </div>`;
  },
```

- [ ] **Step 2: 实现 Render.validation()**

替换占位：

```js
  validation(f) {
    const v = f.validation;
    return `<div class="validation-section">
      <h4>🔍 验证指引（数据依据）</h4>
      <ul>
        <li><strong>Google Trends：</strong>${(v.googleTrends || []).join(" / ")}</li>
        <li><strong>亚马逊验证：</strong>${Utils.escape(v.amazonCheck || "")}</li>
        <li><strong>1688拿货：</strong>${Utils.escape(v.sourcing || "")}</li>
      </ul>
      ${v.riskFlags && v.riskFlags.length ? `<div style="margin-top:6px;color:var(--urgent)"><strong>⚠️ 风险提示：</strong>${v.riskFlags.map(r => Utils.escape(r)).join("；")}</div>` : ""}
    </div>`;
  },
```

- [ ] **Step 3: 实现 Render.controls()**

替换占位：

```js
  controls(f) {
    const festState = State.getFestival(f.id);
    const statusOptions = [
      { v: "none", l: "未启动" },
      { v: "selection", l: "选品中" },
      { v: "ordered", l: "已下单" },
      { v: "arrived", l: "已到仓" },
      { v: "listed", l: "已上架" }
    ];
    return `<div class="card-controls" style="margin-top:8px">
      <label>状态：</label>
      <select onchange="Interact.setStatus('${f.id}',this.value)">
        ${statusOptions.map(o => `<option value="${o.v}" ${festState.status === o.v ? "selected" : ""}>${o.l}</option>`).join("")}
      </select>
    </div>
    <div>
      <label style="font-size:12px;color:var(--text-muted)">📝 备注（店铺分布/供应商等）：</label>
      <textarea class="notes-area" placeholder="如：A店计划上架8款，B店5款；供应商：义乌XX"
        onchange="Interact.setNotes('${f.id}',this.value)">${Utils.escape(festState.notes)}</textarea>
    </div>`;
  }
```

- [ ] **Step 4: 实现 Interact 的SKU/状态/备注/品类筛选方法**

在 Interact 对象内（`toggleMilestone` 之后）追加：

```js
  ,

  // 切换SKU选中
  toggleSku(festivalId, skuName) {
    State.toggleSku(festivalId, skuName);
    // 不重渲染整个卡片（避免输入框失焦），只更新checkbox状态即可，State已保存
  },

  // 设置节日状态
  setStatus(festivalId, status) {
    State.updateFestival(festivalId, { status });
    Render.dashboard();
  },

  // 设置备注
  setNotes(festivalId, notes) {
    State.updateFestival(festivalId, { notes });
  },

  // 筛选卡片内品类
  filterProductCat(festivalId, cat) {
    const card = document.getElementById("card-" + festivalId);
    if (!card) return;
    // 更新标签高亮
    card.querySelectorAll(".product-cat-tab").forEach(t => t.classList.remove("active"));
    event.target.classList.add("active");
    // 筛选行
    card.querySelectorAll(".prod-row").forEach(row => {
      if (!cat || row.dataset.cat === cat) {
        row.classList.remove("hidden");
      } else {
        row.classList.add("hidden");
      }
    });
  }
```

- [ ] **Step 5: 验证选品表与控制区**

浏览器刷新，展开万圣节卡片。预期：
- 物流切换器 + 时间线
- 选品清单表（12行SKU），含品类标签（全部/装饰/礼品/服饰/家居）
- 点击品类标签 → 表格行过滤
- 复选框勾选SKU → 刷新后仍保持
- 验证指引蓝色框（Google Trends/亚马逊/1688/风险提示）
- 状态下拉 + 备注文本框
- 输入备注后刷新仍保留

- [ ] **Step 6: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: 选品表+验证指引+状态备注控制区 (Task 10)"
```

---

## Task 11: 导入/导出/重置 + 回到顶部

**Files:**
- Modify: `uk-festival-planner.html` 的 `<script>` 区块

- [ ] **Step 1: 实现导入导出重置函数**

在 `<script>` 末尾（初始化区之前）追加：

```js
// ============================================
// 数据导入/导出/重置
// ============================================
function exportData() {
  const json = State.export();
  const blob = new Blob([json], { type: "application/json" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = `uk-festival-backup-${Utils.fmtYMD(new Date())}.json`;
  a.click();
  URL.revokeObjectURL(url);
}

function importData(event) {
  const file = event.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = e => {
    try {
      State.import(e.target.result);
      alert("✅ 导入成功！");
      Render.dashboard();
      Render.main();
    } catch (err) {
      alert("❌ 导入失败：" + err.message);
    }
  };
  reader.readAsText(file);
  event.target.value = "";  // 重置input以便重复导入同文件
}

function resetAllProgress() {
  if (!confirm("⚠️ 确定清除所有备货进度吗？此操作不可恢复！\n\n建议先点击\"导出备份\"保存当前数据。")) return;
  if (!confirm("再次确认：真的要清除所有进度？")) return;
  State.reset();
  Render.dashboard();
  Render.main();
  alert("已清除所有进度。");
}
```

- [ ] **Step 2: 实现回到顶部按钮**

追加：

```js
// 回到顶部按钮显隐
window.addEventListener("scroll", () => {
  const btn = document.getElementById("backToTop");
  if (window.scrollY > 400) {
    btn.classList.add("show");
  } else {
    btn.classList.remove("show");
  }
});
```

- [ ] **Step 3: 验证导入导出**

浏览器测试：
- 先勾选几个SKU、改几个状态、写备注
- 点击"📤 导出备份" → 下载JSON文件
- 点击"🗑 清除所有进度" → 二次确认 → 进度清空
- 点击"📥 导入恢复" → 选择刚才的JSON → 进度恢复
- 滚动页面 → 右下角出现"↑"按钮 → 点击回到顶部

- [ ] **Step 4: 提交**

```bash
git add uk-festival-planner.html
git commit -m "feat: 导入导出重置+回到顶部 (Task 11)"
```

---

## Task 12: 最终验收与打磨

**Files:**
- Modify: `uk-festival-planner.html`（如发现问题）

- [ ] **Step 1: 完整功能验收清单**

逐项验证（对照spec §11）：

- [ ] 双击HTML可在浏览器正常打开，无控制台报错
- [ ] 23个节日全部显示，数据完整
- [ ] 紧急度看板4统计卡片数字正确，可点击筛选
- [ ] 倒计时显示今日日期和最近备货节日
- [ ] 筛选栏（品类/月份/紧急度/状态/搜索）组合生效
- [ ] 重置按钮恢复全部
- [ ] S级卡片默认展开，A/B级折叠
- [ ] 点击卡片头展开/折叠
- [ ] 物流切换器3选项，切换后时间线和紧急度重算
- [ ] 时间线5个里程碑，点击圆点切换done状态
- [ ] 选品表按品类标签筛选
- [ ] SKU复选框可勾选，刷新保留
- [ ] 状态下拉切换，看板数字更新
- [ ] 备注文本框输入，刷新保留
- [ ] JSON导出/导入/重置正常
- [ ] 回到顶部按钮工作
- [ ] 移动端窗口缩窄，布局响应式（卡片单列）

- [ ] **Step 2: 检查数据准确性**

浏览器控制台运行：
```js
// 验证节日总数和分布
console.log("总数:", FESTIVALS.length);  // 23
[7,8,9,10,11,12].forEach(m => {
  console.log(`${m}月:`, FESTIVALS.filter(f => f.month === m).length);
});  // 2,3,4,5,6,3
console.log("S级:", FESTIVALS.filter(f => f.importance === "S").map(f => f.name));  // 万圣节,黑五,圣诞
// 验证每个节日都有products和validation
FESTIVALS.forEach(f => {
  if (!f.products || f.products.length === 0) console.error("缺products:", f.id);
  if (!f.validation) console.error("缺validation:", f.id);
});
```

- [ ] **Step 3: 修复发现的问题（如有）**

根据验收发现的问题，逐一修复。常见可能问题：
- milestone日期计算偏差 → 检查 `Utils.getMilestoneDates` 的 `tMinusLead` 偏移逻辑
- 筛选逻辑遗漏 → 检查 `Render.main` 的filter条件
- 移动端布局溢出 → 调整CSS媒体查询

- [ ] **Step 4: 最终提交**

```bash
git add uk-festival-planner.html
git commit -m "chore: 最终验收与打磨 (Task 12)"
```

- [ ] **Step 5: 验收总结**

打开 `uk-festival-planner.html`，确认：
- 这是一个可直接交付给电商团队的决策工具
- 双击即开，可发微信/邮件
- 所有进度本地保存，可备份恢复

完成。

---

## Self-Review 自审

### Spec覆盖检查
- §1 信息架构（header/dashboard/filter/main/footer）→ Task 1 ✅
- §2 节日数据模型（23节点）→ Task 3 ✅
- §3 选品建议结构（含风险评级）→ Task 3数据 + Task 10渲染 ✅
- §4 备货时间线（双段物流）→ Task 2 CONFIG + Task 5计算 + Task 9渲染 ✅
- §5 交互与本地存储 → Task 4 State + Task 8筛选 + Task 10进度 + Task 11导入导出 ✅
- §6 视觉风格 → Task 1 CSS ✅
- §7 验证指引 → Task 10 validation渲染 ✅
- §11 验收标准 → Task 12 ✅

无遗漏。

### 类型一致性检查
- `milestone.id` 使用：selection/airArrival/truckShip/arrival/festival → CONFIG.milestones、State.init、Render.timeline、Interact.toggleMilestone 全部一致 ✅
- `logistics` 取值：air/truck/sea → CONFIG.logisticsModes、LOGISTICS_OPTIONS、State.defaultLogistics、Render.logisticsToggle 全部一致 ✅
- `urgency` 取值：urgent/week/month/plan/past → Utils.getUrgency、Render.dashboard/cards、CSS类名 全部一致 ✅
- `status` 取值：none/selection/ordered/arrived/listed → State.init、Render.controls、filterStatus下拉 全部一致 ✅

无类型不一致。

### 占位符扫描
- 所有"占位方法"在Task 7引入后，Task 9/10已替换为真实实现 ✅
- 所有代码块完整，无TODO/TBD ✅

---

## 执行方式选择

计划已完成并自审通过。两种执行方式：

1. **Subagent驱动（推荐）** - 每个Task派发独立子代理，任务间review，快速迭代
2. **内联执行** - 在当前会话按批次执行，带检查点review
