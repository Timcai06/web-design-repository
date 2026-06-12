# Rive 交互动画 — Sustainability 区域

oryzo.ai 在 Sustainability 区域使用 Rive 实现两个交互式矢量动画：软木收获周期 + 文字动画。

---

## 一、HTML 结构

```html
<section id="sustainability" class="section">
  <div class="section__inner">
    <!-- Rive 动画 1: 软木收获动画 -->
    <div class="sustainability-item">
      <div class="sustainability-item__title">Average age of first harvest</div>
      <canvas id="sustainability-rive-canvas-harvesting" />
      <div class="sustainability-item__num">25</div>
      <div class="sustainability-item__desc">
        Cork oaks are typically first harvested at around 25 years...
      </div>
    </div>

    <!-- Rive 动画 2: 文字动画 -->
    <div class="sustainability-item">
      <canvas id="sustainability-rive-canvas-text" />
      <div class="sustainability-item__title">Harvesting interval</div>
      <div class="sustainability-item__desc">
        After each harvest, the bark takes about 9 years to regrow...
      </div>
    </div>

    <!-- 静态统计信息 -->
    <div class="sustainability-item">
      <div class="sustainability-item__title">Power draw while in use</div>
      <div class="sustainability-item__desc">
        No compute. No tokens. So you can say "please" and "thank you"...
      </div>
    </div>
  </div>
</section>
```

---

## 二、Rive 初始化

```javascript
import Rive from 'rive-js'   // 或 @rive-app/canvas

// ── 收获动画 ──
const riveHarvest = new Rive({
  src: '/rive/sustainability-harvesting.riv',
  canvas: document.getElementById('sustainability-rive-canvas-harvesting'),
  autoplay: true,
  stateMachines: 'harvest_loop',
  onLoad: () => {
    riveHarvest.resizeDrawingSurfaceToCanvas()
  },
})

// ── 文字动画 ──
const riveText = new Rive({
  src: '/rive/sustainability-text.riv',
  canvas: document.getElementById('sustainability-rive-canvas-text'),
  autoplay: true,
  stateMachines: 'text_anim',
  onLoad: () => {
    riveText.resizeDrawingSurfaceToCanvas()
  },
})
```

---

## 三、Rive 动画内容

### 1. Harvesting (软木收获)

| 属性 | 值 |
|---|---|
| Rive 文件 | `sustainability-harvesting.riv` |
| Artboard | `harvesting` |
| State Machine | `harvest_loop` |
| 动画类型 | 循环播放 (loop) |
| 内容 | 软木橡树 → 树皮被剥离 → 年轮显示 25 → 树皮再生 |

动画循环展示：
1. 橡树出现
2. 树皮剥离动画（模拟收获）
3. 数字 "25" 弹出（首次收获年龄）
4. 树皮再生（9 年周期）
5. 回到步骤 1

### 2. Text (文字动画)

| 属性 | 值 |
|---|---|
| Rive 文件 | `sustainability-text.riv` |
| Artboard | `text_anim` |
| State Machine | `text_anim` |
| 动画类型 | 循环播放 |
| 内容 | 文字逐字动画 + 装饰性矢量图形 |

---

## 四、Scroll 触发逻辑

```javascript
// 当 sustainability 区域可见时，确保 Rive 正在播放
ScrollTrigger.create({
  trigger: '#sustainability',
  start: 'top bottom',
  onEnter: () => {
    riveHarvest.play()
    riveText.play()
  },
  onLeave: () => {
    // 不在视口中暂停以省性能
    riveHarvest.pause()
    riveText.pause()
  },
  onEnterBack: () => {
    riveHarvest.play()
    riveText.play()
  },
  onLeaveBack: () => {
    riveHarvest.pause()
    riveText.pause()
  },
})
```

---

## 五、为什么在这个位置用 Rive？

Sustainabilty 区域是网站上唯一使用 Rive 的地方。选择 Rive 的原因：

| 理由 | 说明 |
|---|---|
| **矢量动画表现** | 树皮剥离/生长需要流畅的矢量形变，CSS/GSAP 难以实现 |
| **交互丰富** | Rive State Machine 可以处理多种动画状态 |
| **独立的动画循环** | 不需要与滚动位置绑定，独立 loop 即可 |
| **Canvas 渲染** | 不触发 layout/paint，支持高帧率 |
| **文件小** | 矢量定义，远小于视频或 GIF |

而 Three.js 不适合此处，因为：
- 收益场景是 2D 插画风格，不需要 3D
- Rive 的矢量形变功能比 Three.js 几何体变形更易用

---

## 六、与 Lusion 其他项目的关联

Luke Baffait 的站点没有使用 Rive（他用 Three.js CoreRenderer + GSAP），而在 oryzo 中 Rive 被用于 Sustainability 区域。Lusion 对 Rive 的态度是 **"合适的工具用在对的地方"**：

| 项目 | Rive 使用 |
|---|---|
| landonorris.com | **核心动画引擎**（签名绘制、赛道地图、头盔动画、按钮、页面过渡） |
| lukebaffait.fr | **不使用 Rive**（全部 Three.js + GSAP） |
| oryzo.ai | **Sustainability 区域专用**（软木收获 + 文字动画） |
