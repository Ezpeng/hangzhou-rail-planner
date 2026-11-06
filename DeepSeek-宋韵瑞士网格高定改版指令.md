# 《杭州铁路线路规划》高定改版规范与 DeepSeek 执行指令
> **使用方法**：在 DeepSeek 对话框中，同时上传同目录下的 `index.html` 和本文件 `DeepSeek-宋韵瑞士网格高定改版指令.md`，然后输入一句：**“请严格按照这份指令文档，帮我重构上传的 `index.html`”** 即可。

---

## 一、 核心重构任务声明（给 DeepSeek 的角色与边界要求）

你现在是一位兼具**瑞士国际主义排版功底（Swiss Typographic Grid）**与**中国宋代美学造诣（Song Dynasty Minimalist Aesthetics）**的殿堂级前端架构师与 UI 设计师。

请对上传的单文件应用 `index.html`（《杭州铁路线路规划》，共 1,458 行）进行彻底的视觉、布局与微交互重构，将其从目前的“普通后台/SaaS模板感”升级为**「宋韵江南 · 现代瑞士网格风」**博物馆级数字铁道舆图网站。

### 严守的代码红线（绝对不可破坏）：
1. **数据零删减**：必须 **100% 原样保留** 文件中以下 6 个 `<script type="application/json">` 数据块及其 ID：
   - `#rail-lines`（125 条全国铁路干线）
   - `#rail-special`（特殊目的地说明）
   - `#rail-alias`（城市别名映射）
   - `#rail-direct`（杭州跨线贯通直达城市）
   - `#rail-province`（省份归属）
   - `#rail-citypos`（全国城市 SVG 坐标）
2. **算法零破坏**：完整保留 `findRoutes(from, to)` BFS 寻路算法、`resolveCity(raw)`、`midStopsOf(seg)`、搜索下拉键盘导航、复制路线与 12306 跳转逻辑。

---

## 二、 现有代码“普通感”诊断与重构对照表

| 模块 / 选择器 | 现有代码问题（普通感根源） | 重构目标（宋韵江南 × 现代瑞士网格风） |
| :--- | :--- | :--- |
| **全局色盘 `:root`** | `--bg:#f3f5f9` 冷灰蓝后台色、`--accent:#0052d9` 标准组件库亮蓝、顶部四色拼接条 | **宣纸暖白 `#F6F4EE`** 底色 + **汝窑天青 `#2B5B6C` / 朱砂 `#B83B2B` / 竹青 `#3D6E58`** 矿物色系 + 2.5% SVG 宣纸纤维纹理 |
| **页面容器 `.wrap`** | `max-width: 880px` 单栏堆叠，向下滚动看方案卡时地图滚出视口，导致 `mouseenter` 地图联动形同虚设 | **1380px 瑞士双栏非对称网格（左 480px Sticky 固定舆图与统计 + 右侧自适应方案长卷）**，悬停卡片时同屏实时切换地图高亮路线 |
| **边框与圆角 `--r`** | 满屏 `16px` 大圆角 + `999px` 胶囊按钮 + 模糊阴影，缺乏秩序感 | **`2px` 微切角 + 外露 `1px` 水墨细线网格 (`1px solid rgba(24,26,27,0.12)`)**，用宋刻版心线分割信息 |
| **字体系统** | 全站单一无衬线体（`PingFang SC`），标题、站名、数字毫无层级张力 | **思源宋体 (`Noto Serif SC`)** 渲染大标题与起终站名 + **等宽字体 (`JetBrains Mono`)** 渲染所有换乘数、站数、坐标与编号 |
| **轨道图 `.schem`** | `repeating-linear-gradient` 粗枕木略显卡通；途经站 `.stop` 是零散胶囊堆砌 | **双平行极细金属轨线 + 双环玉璧换乘节点 + 2px 切角方印车次标**；途经站改为**经折式连线时间轴** |
| **中国地图 `#mapSvg`** | 仅有粗略多边形配浅蓝底，缺乏地理坐标与杭州核心枢纽锚点 | 底层增加 **100px 瑞士经纬十字网格与度数刻度**；杭州起点加入**西湖双环水波纹脉冲**；路线改为**大圆微拱贝塞尔弧线** |

---

## 三、 逐模块代码重构规范（请直接参照以下实现）

### 1. 引入字体与重写 `:root` 设计令牌（替换原第 9-62 行）

在 `<head>` 中引入 Google Fonts（带本地中文字体回退）：
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&family=Noto+Sans+SC:wght@400;500;600&family=Noto+Serif+SC:wght@600;700&display=swap" rel="stylesheet">
```

将 `:root` 与深色主题替换为以下**宋代矿物色盘与字体变量**：
```css
:root {
  color-scheme: light;
  /* 宣纸与水墨基底 */
  --bg: #F6F4EE;
  --card: #FBF9F5;
  --card2: #F1EDE4;
  --inset: #E8E3D8;
  --ink: #181A1B;
  --ink2: #4A5054;
  --mute: #7A8084;
  /* 瑞士极细水墨网格线 */
  --line: rgba(24, 26, 27, 0.12);
  --line2: rgba(24, 26, 27, 0.07);
  --line-strong: rgba(24, 26, 27, 0.24);
  /* 汝窑天青主色调 */
  --accent: #2B5B6C;
  --accent-soft: rgba(43, 91, 108, 0.09);
  --accent-ink: #1D4350;
  --accent-line: rgba(43, 91, 108, 0.35);
  --accent-grad: linear-gradient(135deg, #2B5B6C 0%, #3A6E6C 100%);
  /* 宋画矿物四色：G朱砂 / D天青 / K竹青 / F黛蓝 / 换乘泥金 */
  --g: #B83B2B;  --g-text: #962B1D;  --g-tint: rgba(184, 59, 43, 0.09);
  --d: #2B5B6C;  --d-text: #1E4654;  --d-tint: rgba(43, 91, 108, 0.10);
  --k: #3D6E58;  --k-text: #2B5442;  --k-tint: rgba(61, 110, 88, 0.10);
  --transfer: #B57C2E; --transfer-text: #8C5D1B; --transfer-tint: rgba(181, 124, 46, 0.12);
  --warn: #B85D2B; --warn-text: #8F4218; --warn-tint: rgba(184, 93, 43, 0.11);
  --ferry: #4A6875; --ferry-text: #344E59; --ferry-tint: rgba(74, 104, 117, 0.11);
  --rail: #8C9296; --rail-tie: #C8C4BA;
  /* 舆图专用色 */
  --map-fill: #EFECE4;
  --map-land: #FAF8F3;
  --map-stroke: #B8B3A8;
  --map-grid: rgba(24, 26, 27, 0.06);
  --glow: rgba(43, 91, 108, 0.22);
  --chip-ink: #FBF9F5;
  /* 克制阴影与微切角 */
  --shadow: 0 2px 10px rgba(24, 26, 27, 0.03);
  --shadow-lg: 0 12px 32px rgba(24, 26, 27, 0.08);
  --r: 2px;
  /* 字体家族 */
  --serif: "Noto Serif SC", "Songti SC", "STSong", serif;
  --sans: "Noto Sans SC", -apple-system, BlinkMacSystemFont, "PingFang SC", sans-serif;
  --mono: "JetBrains Mono", ui-monospace, SFMono-Regular, Menlo, monospace;
}

@media (prefers-color-scheme: dark) {
  :root {
    color-scheme: dark;
    --bg: #0E1112;
    --card: #14181A;
    --card2: #1A1F22;
    --inset: #101416;
    --ink: #EAE6DF;
    --ink2: #A5A29B;
    --mute: #6E6D68;
    --line: rgba(234, 230, 223, 0.11);
    --line2: rgba(234, 230, 223, 0.06);
    --line-strong: rgba(234, 230, 223, 0.22);
    --accent: #5C9BAE;
    --accent-soft: rgba(92, 155, 174, 0.14);
    --accent-ink: #89C2D4;
    --accent-line: rgba(92, 155, 174, 0.4);
    --accent-grad: linear-gradient(135deg, #3A6E7E 0%, #4E8985 100%);
    --g: #D95B4B;  --g-text: #F08578;  --g-tint: rgba(217, 91, 75, 0.16);
    --d: #5C9BAE;  --d-text: #85BFD1;  --d-tint: rgba(92, 155, 174, 0.16);
    --k: #589A7D;  --k-text: #7EC2A4;  --k-tint: rgba(88, 154, 125, 0.16);
    --transfer: #D49A4B; --transfer-text: #E8B872; --transfer-tint: rgba(212, 154, 75, 0.16);
    --warn: #D98254; --warn-text: #EDA37C; --warn-tint: rgba(217, 130, 84, 0.15);
    --ferry: #6892A3; --ferry-text: #90B8C9; --ferry-tint: rgba(104, 146, 163, 0.16);
    --rail: #545B5E; --rail-tie: #343A3D;
    --map-fill: #111517;
    --map-land: #181D20;
    --map-stroke: #384044;
    --map-grid: rgba(234, 230, 223, 0.05);
    --glow: rgba(92, 155, 174, 0.28);
  }
}
```

### 2. 宣纸纹理背景与宋刻双线书眉

- 在 `<body>` 顶部加一条宋刻古籍风格的双线书眉（外粗内细：`border-top: 3px double var(--line-strong)`），并添加一个全屏 `pointer-events: none` 的 SVG 宣纸纤维噪点层：
```css
body {
  margin: 0;
  font-family: var(--sans);
  background: var(--bg);
  color: var(--ink);
  line-height: 1.6;
  padding: 0 28px 64px;
  border-top: 4px double var(--line-strong);
  position: relative;
}
/* 宣纸微噪点肌理 */
body::after {
  content: "";
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 999;
  opacity: 0.026;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}
.wrap {
  max-width: 1380px;
  margin: 0 auto;
}
```

### 3. 瑞士杂志级刊头（`.page-head`）与控制台式搜索框（`.search-box`）

- **刊头升级**：
  - 将 `.logo` 改为一枚 **朱砂方印徽章**（`background: var(--g); color: #FBF9F5; border-radius: 2px; border: 1px solid var(--g)`），内部可用 SVG 绘制篆刻感“杭”字或极简列车印记。
  - 在 `.page-title` 上方增加一行瑞士网格微标签：`<div class="eyebrow">ARCHIVE 0571 // NATIONAL RAILWAY TOPOLOGY · 杭城铁道舆图</div>`（`font-family: var(--mono); font-size: 10.5px; letter-spacing: 0.18em; color: var(--mute)`）。
  - `.page-title` 采用 `font-family: var(--serif); font-size: 28px; font-weight: 700; letter-spacing: 0.04em`。
  - 右侧 `.pill` 改为直角方格，数字使用 `var(--mono)`。
- **搜索台与热门城市标签**：
  - 在 `.s-input` 左侧加入一个固定始发站印记标签：`<span class="origin-tag">始发 · 杭州</span>`（配极细右边框分割），让“从杭州出发 → 输入目的地”的仪式感更强。
  - `.chip`（热门城市与最近搜索）全部改为 `border-radius: 2px` 的宋式书签方格，悬停时左边框变为 `3px solid var(--accent)`。

### 4. 重构 `renderAll()`：桌面端「左图右卡」瑞士双栏 Sticky 布局

- 修改 JS 中的 `renderAll()`：当查询出结果时，不要把统计卡、筛选器、地图、方案卡平铺塞进 `#result`，而是创建如下双栏瑞士网格结构：
```javascript
// 在 renderAll() 中创建双栏容器：
var grid = document.createElement('div');
grid.className = 'editorial-grid';
var colVisual = document.createElement('div');
colVisual.className = 'col-visual';
var colRoutes = document.createElement('div');
colRoutes.className = 'col-routes';
grid.appendChild(colVisual);
grid.appendChild(colRoutes);
result.appendChild(grid);
```
- **左栏 `.col-visual`（Sticky 固定画布）**：放置 **中国铁路舆图 (`#mapWrap`)** + **方案统计矩阵 (`.stats-card`)**。
- **右栏 `.col-routes`（方案长卷流）**：顶部放置 **方案筛选栏 (`.filter-bar`)**，下方渲染 **方案卡列表 (`.card`)**。
- **对应 CSS**：
```css
.editorial-grid {
  display: grid;
  grid-template-columns: 500px minmax(0, 1fr);
  gap: 28px;
  align-items: start;
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid var(--line-strong);
}
.col-visual {
  position: sticky;
  top: 20px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}
@media (max-width: 1024px) {
  .editorial-grid {
    grid-template-columns: 1fr;
  }
  .col-visual {
    position: static;
  }
}
```

### 5. 升级 `#mapSvg`：宋韵经纬舆图 + 西湖水波纹起点 + 大圆弧线

1. **经纬十字坐标网（Graticule）**：在 `buildMap()` 中，`LAND_PATH` 底层加入纵横间隔 `100px` 的经纬网格线与边缘度数标注（`80°E`, `90°E`, `100°E`, `110°E`, `120°E`, `130°E` 与 `20°N`, `30°N`, `40°N`, `50°N`），并在地图左上角加上宋式舆图题跋角标 `FIG.01 / 华夏铁道经纬略图`。
2. **陆地轮廓质感**：`.land` 的填充色设为 `var(--map-land)`，描边设为 `1.5px solid var(--map-stroke)`，取消原有的模糊黑色 `drop-shadow`。
3. **杭州起点水波纹脉冲**：在 `drawMapRoute()` 绘制 `.from-dot`（杭州）时，除了实心朱砂点外，额外绘制一个同心扩散外环 `<circle class="hz-pulse" cx="..." cy="..." r="12"/>`，配 CSS 呼吸脉冲动画。
4. **优美大圆弧线 `smoothPath(pts)`**：将原本的水平折算中点升级为向法线方向微拱的弧线，使长途干线更具空间张力：
```javascript
function smoothPath(pts){
  if(pts.length < 2) return '';
  var d = 'M' + pts[0][0] + ',' + pts[0][1];
  for(var i=1; i<pts.length; i++){
    var p0 = pts[i-1], p1 = pts[i];
    var dx = p1[0] - p0[0], dy = p1[1] - p0[1];
    var mx = (p0[0] + p1[0]) / 2 - dy * 0.12;
    var my = (p0[1] + p1[1]) / 2 + dx * 0.12;
    d += ' Q' + Math.round(mx) + ',' + Math.round(my) + ' ' + p1[0] + ',' + p1[1];
  }
  return d;
}
```

### 6. 重塑 `.card` 方案卡、`.schem` 轨道图与 `.stops-grid` 经折时间轴

1. **方案卡头部（`.card-head`）**：
   - 在每张方案卡左上角增加瑞士等宽编号：`<span class="route-no">NO. 01 // RECOMMENDED</span>`。
   - `.od`（`杭州 → 目的地`）站名采用 `font-family: var(--serif); font-size: 22px; font-weight: 700`。
   - `.badge` 全部改为 `border-radius: 2px` 的方印标签（例如 `.badge--solid.g` 为朱砂红底白字方印，带 `1px` 内描边感）。
2. **去卡通化轨道示意图（`.schem`）**：
   - 将 `.track` 的粗枕木条纹改为精致的 **双平行极细金属轨线**（`height: 5px; border-top: 1.5px solid var(--rail-c); border-bottom: 1px solid var(--rail-c); background: none`）。
   - 将换乘节点 `.node-dot--transfer` 设计为 **双环玉璧节点**（外环 `18px` 空心圆 + 内芯 `8px` 实心圆）。
3. **经折式途经站时间轴（`.stops-grid`）**：
   - 将原本零散堆砌的圆角胶囊 `.stop`，升级为带水平连线的**经折式驿道序列**：每个 `.stop` 左侧带一个 `4px` 水墨圆点，站点之间用 `—` 或 CSS 伪元素细线串联，字号 `12px`，展开全部站点时呈现古籍名录般的秩序感。
4. **操作按钮（`.btn12306` & `.btn-copy`）**：
   - `.btn12306` 改为松烟墨实心按钮（`background: var(--ink); color: var(--bg); border-radius: 2px; font-family: var(--sans); letter-spacing: 0.05em`），悬停时切换为汝窑天青 `var(--d)`。

---

## 四、 输出要求

请输出重构后完整、可直接双击运行的 `index.html` 代码。确保所有 CSS 选择器、DOM 渲染函数与原有的 6 个 JSON 数据块无缝衔接，不要遗漏任何一处闭合标签或事件绑定。
