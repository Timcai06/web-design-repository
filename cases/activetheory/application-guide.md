# Active Theory — Design Rationale & Application Guide

## 一、设计语言分析

### 视觉 DNA

```
暗色 + 霓虹 ← 核心视觉风格
     ↓
3D 沉浸环境（办公室场景）
     ↓
粒子系统（彩色管、发光）
     ↓
极简 UI（Pillbox + AI Chat）
```

### 为什么这套语言有效

1. **暗色 + 霓虹** — 2024-2026 年的网页设计主流是亮色、干净、留白。ATv6 的黑暗霓虹风格是对主流的有意反叛——"我们不是来迎合潮流的"。

2. **3D 办公室环境** — 不是抽象的几何体，而是真实办公室的 3D 重建。这让"公司文化"变得可感知——用户不需要读"About us"页面，他们直接"走进"了公司。

3. **粒子管轨迹** — 鼠标的每一次移动都在页面上留下痕迹。这让"浏览"本身变成了"创作"——用户的行为产生了视觉输出，而不是被动消费。

4. **极简 UI** — 3D 环境已经足够丰富，UI 的任何增加都会变成干扰。Pillbox 导航只有 3 个链接 + AI Chat——这是"最少 UI 完成最多功能"的设计。

### 对比其他工作室

| 维度 | Active Theory | Lusion | Resn | Media.Monks |
|------|-------------|--------|------|-------------|
| 视觉风格 | 霓虹 / 暗色 / 末世 | 亮色 / 产品 / 精致 | 高能量 / 冲击 | 真实 / 学术 / 历史 |
| 核心载体 | 自研 3D 引擎 | Three.js | CSS + DOM | Unity → WebGL |
| 交互深度 | 多用户 + AI | 滚动驱动 | 滚动驱动 | 点击导航 |
| UI 密度 | 极少 | 中 | 中 | 中-高 |
| 渲染复杂度 | 极高 | 高 | 低（CSS） | 极高 |
| 入门成本 | 极高（自研引擎） | 高（Three.js） | 低（CSS） | 极高（Unity） |

---

## 二、每个核心效果的 Application Guide

### 1. Hydra 自研引擎（可复用性: ⭐）

**什么时候应该自研引擎：**
- 团队 >10 人
- 项目高度重复（每年 5+ 个 WebGL 项目）
- 有设计师需要参与 3D 场景构建
- 需要跨平台（Web + 原生 + VR）

**什么时候不应该：**
- 单人/小团队 → 用 Three.js
- 第一次做 WebGL 项目 → 用 Three.js
- 项目只需要简单 3D → 用 CSS 3D 或 Three.js

**如果你决定自研：** 不要从零开始，从 Three.js fork 开始并逐步替换管线（Active Theory 的做法）。

### 2. 多用户联网光标（可复用性: ⭐⭐⭐）

**适合场景：**
- 创意工作室 portfolio
- 虚拟活动/展览
- 产品发布页（制造"大家都在看"的热闹感）

**实现建议：**
```javascript
// 最小化联网光标 — 30 行核心
const ws = new WebSocket('wss://your-server.com/cursors')
let others = {}

ws.onmessage = (e) => {
  const data = JSON.parse(e.data)
  others[data.id] = data.position
}

document.addEventListener('mousemove', (e) => {
  ws.send(JSON.stringify({
    id: myId,
    position: { x: e.clientX, y: e.clientY }
  }))
})

// 渲染其他人的光标
function renderOthers() {
  Object.values(others).forEach(pos => {
    ctx.beginPath()
    ctx.arc(pos.x, pos.y, 4, 0, Math.PI * 2)
    ctx.fill()
  })
  requestAnimationFrame(renderOthers)
}
```

**不要做的：** 在隐私敏感页面（金融、医疗、B2B）添加联网光标。

### 3. AI Chat 导航（可复用性: ⭐⭐⭐⭐）

**适合场景：**
- 作品集（用户以探索为目的）
- 内容站（用户想发现内容）
- SaaS（用户想找到特定功能）

**不适合：**
- 电商（用户想快速下单，不需要聊天）
- 登陆页（用户只需要一个 CTA）

**实现路径：**
1. 最简：关键词匹配（"fun" → 标签含 "fun" 的项目）
2. 中等：Embedding 语义搜索（用 sentence-transformers 将项目和查询转为向量）
3. 完整：LLM（GPT/Claude） + RAG（项目数据作为上下文）

### 4. 3D 环境滚动推进（可复用性: ⭐⭐）

**适合：** 品牌/工作室站，有足够的 3D 资源预算

**不适合：** 内容站、工具型页面

**替代方案（低成本）：**
```css
/* 用 CSS 3D perspective 模拟"空间推进"感 */
.scroll-world {
  perspective: 800px;
  transform-style: preserve-3d;
}
.scroll-world > * {
  transform: translateZ(calc(var(--scroll) * -1px));
}
```

### 5. 粒子管系统（可复用性: ⭐⭐⭐）

**适合：** 创意站、游戏、音乐可视化

**替代方案（Canvas 2D，无需 WebGL）：**
```javascript
// Canvas 2D 粒子管 — 适合不需要大量粒子的场景
const trail = []
canvas.addEventListener('mousemove', (e) => {
  trail.push({ x: e.clientX, y: e.clientY, life: 1 })
})

function draw() {
  trail.forEach((p, i) => {
    p.life -= 0.02
    ctx.globalAlpha = p.life
    ctx.beginPath()
    ctx.arc(p.x, p.y, 4 * p.life, 0, Math.PI * 2)
    ctx.fill()
  })
  trail = trail.filter(p => p.life > 0)
  requestAnimationFrame(draw)
}
```

### 6. Pillbox 速度响应导航（可复用性: ⭐⭐⭐⭐⭐）

**这是全站最易移植的效果。** 只需要 CSS + 20 行 JS。

```javascript
// 通用 pillbox 速度响应
const nav = document.querySelector('.nav')
let lastY = 0, lastT = Date.now()

window.addEventListener('scroll', () => {
  const now = Date.now()
  const speed = Math.abs(window.scrollY - lastY) / (now - lastT)
  nav.style.opacity = speed > 1 ? 0.15 : 1
  nav.style.transform = `scale(${Math.max(0.5, 1 - speed * 0.3)})`
  lastY = window.scrollY
  lastT = now
}, { passive: true })
```

**适合所有长滚动页面。**

---

## 三、整体可复用性评分

| 效果 | 可复用性 | 开发成本 | 视觉冲击 | 复杂度 | 推荐优先级 |
|------|---------|---------|---------|--------|-----------|
| Pillbox 速度导航 | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐ | 低 | 1 |
| AI Chat 导航 | ⭐⭐⭐⭐ | 中 | ⭐⭐⭐⭐⭐ | 中 | 2 |
| 粒子管轨迹 | ⭐⭐⭐ | 低-中 | ⭐⭐⭐⭐ | 中 | 3 |
| 多用户联网 | ⭐⭐⭐ | 高 | ⭐⭐⭐⭐ | 高 | 4 |
| 3D 环境滚动 | ⭐⭐ | 极高 | ⭐⭐⭐⭐⭐ | 极高 | 5 |
| ActiveFrame 视频 | ⭐⭐ | 高 | ⭐⭐⭐ | 高 | 6 |
| Hydra 自研引擎 | ⭐ | 极高 | ⭐⭐⭐⭐⭐ | 极高 | 仅大型团队 |
| Hydra GUI | ⭐ | 极高 | ⭐⭐⭐ | 极高 | 仅大型团队 |
