# CSS Clip-Path Effects — 裁剪路径设计模式

5 个案例中 4 个使用了 `clip-path` 作为核心视觉手段。clip-path 是"低成本 + 高视觉回报"的第一选择。

---

## 一、Clip-Path 的设计价值

### 为什么 clip-path 看起来"高级"？

1. **打破屏幕矩形惯性** — 人类每天看到的 UI 99% 是矩形。任何非矩形的边界都会触发视觉警觉。这就是 Zentry 菱形 Portal 和 landonorris 椭圆头盔 hover 第一眼就抓眼球的原因。

2. **动势方向暗示** — clip-path 动画的方向暗示"力"的来源：
   - `ellipse(0% → 100%)` = 从中心扩散（膨胀感）
   - `polygon(top → bottom)` = 从上往下（幕布拉开感）
   - `inset(left → 0)` = 从左到右（阅读方向）
   - `polygon(center point → full)` = 从一点爆发（能量感）—— Zentry Portal

3. **双图层 illusion** — clip-path 本质上是在"揭示"下层内容。两个图层的视觉叠加创造纵深感和"窥探感"，比单层 fade-in 有趣得多。

---

## 二、Clip-Path 效果谱系

### 效果 1：Portal Mask（Zentry）

```
clip-path: polygon(50% 50%, 50% 50%, 50% 50%, 50% 50%)
              → polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%)
```

**为什么好看：** 从中心点爆发展开，模拟"传送门开启"的视觉语言。四个顶点从中心向外拉，形成星爆（star burst）效果。

**变体：**
```
/* 3D 感更强的菱形 */
clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%)
/* 斜角展开 */
clip-path: polygon(50% 0%, 100% 0%, 100% 50%, 50% 100%, 0% 50%, 0% 0%)
```

**技术备注：** clip-path 动画需要 GPU 合成器参与，因为路径插值是在 GPU 上计算的。使用 `will-change: clip-path` 触发独立合成层。

### 效果 2：Ellipse Hover Reveal（landonorris 头盔）

```
clip-path: ellipse(0% 0% at 50% 0%)    →  hidden
clip-path: ellipse(100% 120% at 50% 0%) →  hovered
```

**为什么好看：** 从顶部中心向下展开，模拟"幕布从上往下拉开"或"灯光从上方打下"。120% 的垂直半径让椭圆在结束时超出容器底部，制造"溢出感"——视觉上更饱满。

**应用场景：** 产品图切换、头像 hover、thumbnail reveal。

**调参指南：**
```css
/* 温和版（适合文字hover） */
clip-path: circle(0% at 50% 50%) → circle(100% at 50% 50%)
/* 动感版（适合图片） */
clip-path: ellipse(0% 0% at 50% 0%) → ellipse(120% 120% at 50% 0%)
/* 优雅版（适合品牌） */
clip-path: inset(50% 0% 50% 0%) → inset(0% 0% 0% 0%)
```

### 效果 3：Inset Grid Reveal（Oryzo Social Grid）

```
clip-path: inset(0% 0% 100% 0%) → inset(0% 0% 0% 0%)
```

**为什么好看：** 从底部向上揭示，模拟"文字/图片从地面升起"。配合 staggered delay（每个元素延迟 50-100ms），形成波浪效果。

### 效果 4：SVG Mask Organic Shapes（landonorris footer）

```css
--mask-url: url('...ln4-footer-mask-desktop.svg')
-webkit-mask-image: var(--mask-url)
```

**为什么好看：** 有机形态（非几何形）的 mask 创造"手工艺感"和"呼吸感"。相比 clip-path 只能做几何裁剪，SVG mask 可以做出流动的、不规则的边界——让 footer 不再是简单矩形底栏。

---

## 三、什么时候用哪种 Clip-Path

| 效果 | 视觉暗示 | 适合 | 不适合 |
|------|---------|------|-------|
| Polygon 中心展开 | 爆发、传送门、能量释放 | Hero、大标题 | 文字密集区 |
| Ellipse 圆形展开 | 柔和、聚焦、揭示 | 图片 hover、头像、产品图 | 大段文本 |
| Inset 从下往上 | 升起、浮现、生长 | 网格元素、列表、卡片 | 全屏效果 |
| Polygon 对角线 | 速度、切割、动感 | 运动品牌、游戏 | 正式/优雅品牌 |
| SVG Mask 有机形 | 自然、手工、柔和 | Footer、装饰性分割、个人品牌 | 科技/金融网站 |

---

## 四、性能评估

| 方法 | GPU 加速 | 动画性能 | 浏览器兼容 | 着色能力 |
|------|---------|---------|-----------|---------|
| CSS clip-path | ✅ 自动 | ⭐⭐⭐⭐⭐ | ⚠️ Safari 需 `-webkit-` | 仅 polygon/circle/ellipse/inset |
| SVG mask | ✅ 自动 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 任意矢量路径 |
| WebGL mask | ❌ 自建 | ⭐⭐(JS overhead) | ⭐⭐⭐ | 任意 + 动态效果 |

**推荐：** 优先 CSS clip-path，复杂形状用 SVG mask，全线 fallback 用 opacity fade。

---

## 五、Easing 设计

clip-path 动画的 `easing` 选择直接决定"质感"：

```css
/* 弹性感（Zentry Portal） */
ease: cubic-bezier(0.34, 1.56, 0.64, 1)

/* 从容揭示（landonorris helmet） */
ease: cubic-bezier(0.65, 0.05, 0, 1)

/* 机械感（Oryzo grid） */
ease: cubic-bezier(0.4, 0, 0.2, 1)

/* 不要用的：linear */
/* clip-path linear 动画看起来像"机器在裁剪"，没有任何质感 */
```

**Design Rationale：** Clip-path 动画本质上是"空间变形"，easing 决定了变形的"力"来自哪里。`cubic-bezier(0.34, 1.56, 0.64, 1)` 初始加速快 + 末尾 overshoot，模拟"弹簧释放"——Zentry 的 portal 展开就像一扇弹开的门。`cubic-bezier(0.65, 0.05, 0, 1)` 先慢后快再慢，模拟"手动拉开幕布"——这是 landonorris 头盔 hover 的从容感来源。
