# landonorris — Design Rationale & Application Guide

landonorris.com 的核心设计语言：**矢量风格 + 运动气息 + 亲切感**。用 Rive 矢量动画替代传统 3D/视频，打造"活泼但不轻浮"的品牌调性。

---

## 一、设计语言分析

### 视觉 DNA

```
矢量插图（Rive） ← 核心视觉资产
     ↓
高饱和度色彩（绿色为主 + 白色 + 深绿）
     ↓
有机形状（SVG mask、clip-path）
     ↓
平滑运动（Rive morph + GSAP scroll）
```

**为什么这套语言有效：**
- **矢量风格**呼应 F1 的"工程感"——F1 车队使用大量 CAD 和技术图表，矢量风格的干净线条与 F1 的品牌语言自然对齐
- **主色调绿色**是 McLaren 品牌色，"lime" 绿色是高饱和度 + 高明度的"能量色"——与 F1 的速度感直接关联
- **有机形状 mask**（footer 轮廓、头盔形状）打破网站的"盒子感"，让页面在品牌框架内仍然有呼吸感
- **Rive 动画**比视频轻量、比 CSS 灵活、比 Three.js 易维护——OFF+BRAND 的选择体现了"做对的事而不是最酷的事"的设计哲学

### 与 Lusion 风格对比

| 维度 | landonorris (OFF+BRAND) | Oryzo (Lusion) |
|------|------------------------|----------------|
| 渲染核心 | Rive 矢量 (canvas) | Three.js 3D (WebGL) |
| 视觉温度 | 温暖、亲和 | 冷感、科技 |
| 开发哲学 | 极简工具链（Webflow + vanilla JS） | Astro + 模块化 JS |
| 动效气质 | "从容的运动"— ease 较慢 | "活泼的产品"— ease 较快 |
| 加载策略 | Preload Rive .riv (轻量) | Preload 3D model (较重) |
| 适用场景 | 品牌/艺人/体育站 | 产品营销/Creative Showreel |

---

## 二、每个效果的 Design Rationale + Application Guide

### 1. Hero — Sticky + Touch Lock

**为什么这样设计：** Hero 区域是全站最重要的"第一印象"。landonorris 的 Hero 做了两件聪明的事：
1. Sticky（固定）而非 scroll-through（滚过）— 用户到达时 Hero 停住，让用户有时间看到完整动画（头盔 + 签名 + 文字），而不是一进来就被滚上去
2. Touch Lock（滑动锁定）— 在手机上，Hero 区域可以通过滑动切换内容，锁定后继续滚动。这解决了"移动端 Hero 内容太多"的经典问题

**适合放在：**
- 需要展示"多内容块"的 Hero（运动员/艺人的多个高光时刻、产品的多个角度的展示）
- 有视频/动画资源需要用户看完的页面

**不适合：**
- 信息型页面（用户想快速到达内容）
- 只有一个核心 call-to-action 的页面

**改编方案：**
```css
/* 通用 touch lock 实现 */
.hero-sticky {
  position: sticky;
  top: 0;
  height: 100vh;
  overflow: hidden;
}
.hero-locked {
  pointer-events: none;  /* 锁定状态下穿透滚动 */
}
.hero-unlocked {
  pointer-events: auto;
}
```

### 2. CSS Helmet Hover (Clip-Path Ellipse)

**为什么好：** clip-path ellipse 从顶部展开，模拟"光从上方打下"或"幕布从上拉开"。没有用 JS——纯 CSS transition。这使得 hover 响应极快（vs JS 驱动的 paint）。

**适合放在：**
- 产品网格（电商、作品集、艺人周边）
- 团队介绍（头像 hover 显示动态内容）
- 任何需要"两张图片切换"的 hover

**改编为通用模式：**
```css
.reveal-hover {
  position: relative;
  overflow: hidden;
}
.reveal-hover__base {
  display: block;
}
.reveal-hover__overlay {
  position: absolute;
  inset: 0;
  clip-path: ellipse(0% 0% at 50% 0%);
  transition: clip-path 0.75s cubic-bezier(0.65, 0.05, 0, 1);
}
.reveal-hover:hover .reveal-hover__overlay {
  clip-path: ellipse(120% 120% at 50% 0%);
}
```

**调参：** 电商 0.4s，品牌站 0.75s，奢侈品牌 1.2s。

### 3. Rive + ScrollTrigger Bridge

**为什么好：** Rive 的矢量动画引擎擅长"帧间变形"（morphing），但不擅长"滚动驱动"。GSAP 的 ScrollTrigger 擅长"滚动驱动"，但不能渲染矢量动画。两者的桥接创造了——**用滚动直接控制 Rive 动画的每一帧**。

**适合放在：**
- 滚动驱动的品牌故事讲述
- 需要"绘图效果"的动画（签名绘制、地图路径）
- 矢量插画风格的交互元素

**不适合：**
- 已经有大量 Rive 素材的项目（Rive 文件较大，多个 Rive 实例占用内存）
- 简单 hover 效果（CSS transition 就足够）

**改编为通用模式：**
```javascript
function riveScrollBridge(canvas, riveInstance, { start, end, scrub = 0.5 }) {
  const scrollInput = riveInstance.stateMachineInputs()
    .find(input => input.name === 'scroll')

  ScrollTrigger.create({
    trigger: canvas,
    start: start || 'top bottom',
    end: end || 'bottom top',
    scrub: scrub,
    onUpdate: (self) => {
      if (scrollInput) {
        scrollInput.value = self.progress * 1000
      }
    }
  })
}
```

### 4. Horizontal Scroll Gallery

**为什么好：** 水平滚动在垂直滚动的页面中创造了"格式塔断裂"——用户的预期是向下滚，突然变成向左滚 → 注意力被重新捕获。

**适合放在：**
- 作品展示（水平滚动适合"横向构图"的图片，如宽屏截图、全景照片）
- 时间线（横向天然适合"从左到右"的时间表述）
- 对比展示（On Track / Off Track 并列）

**改编方案：**
```javascript
function horizontalScroll(sectionSelector, trackSelector) {
  const section = document.querySelector(sectionSelector)
  const track = document.querySelector(trackSelector)

  ScrollTrigger.create({
    trigger: section,
    pin: true,
    start: 'top top',
    end: () => `+=${track.scrollWidth - window.innerWidth}`,
    scrub: 1,
    onUpdate: (self) => {
      const x = -(track.scrollWidth - window.innerWidth) * self.progress
      gsap.set(track, { x })
    }
  })
}
// 使用：horizontalScroll('[data-horizontal-section]', '[data-horizontal-track]')
```

**性能注意：** `x` 变换是 GPU 加速的，但内部内容如果有大量 DOM 元素，仍需限制。建议内部元素 <50。

### 5. SVG Mask Footer

**为什么好：** 大多数网站的 footer 是"一个矩形框底部放链接"。landonorris 用 SVG mask 把 footer 切成了 LN4 图标形状——这让"品牌 logo"和"footer 边界"合二为一。

**适合放在：**
- 品牌站 footer（用 logo 轮廓作为 mask 形状）
- 个人站 footer（用签名/头像轮廓）
- Section 分割（用有机形状代替直线分割）

**改编方案：**
- 在 Illustrator/Figma 中导出品牌元素的 SVG path
- 用 `mask-image: url(path.svg)` 应用
- 移动端使用不同的 SVG（形状可以简化）

### 6. Nav Theme Toggle (滚动驱动主题切换)

**为什么好：** 导航随滚动切换 light/dark 主题，解决了"亮色内容 + 暗色导航"或反之的对比度问题。用户在深色 Hero 区域获得深色导航，滚动到亮色区域时导航自动变亮。

**适合放在：**
- 有多个不同亮度 section 的页面
- 需要"导航融入背景"的品牌体验

**实现要点：**
- 用空 div 作为触发器（`<div class="hero-nav-theme.is-2">`）— 在特定滚动位置放置标记元素
- 触发器到达视口顶部时切换导航 class
- CSS transition 处理导航元素的颜色变化（0.4-0.6s）

---

## 三、整体可复用性评分

| 效果 | 可复用性 | 开发成本 | 视觉冲击 | 推荐优先级 |
|------|---------|---------|---------|-----------|
| CSS Helmet Hover (clip-path) | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐⭐ | 1 |
| Rive ScrollTrigger Bridge | ⭐⭐⭐ | 中 | ⭐⭐⭐⭐⭐ | 2（有 Rive 资源时） |
| SVG Mask Footer | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐ | 3 |
| Horizontal Gallery | ⭐⭐⭐⭐ | 中 | ⭐⭐⭐⭐ | 4 |
| SplitText Char Reveal | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐⭐ | 5 |
| Nav Theme Toggle | ⭐⭐⭐⭐ | 低 | ⭐⭐ | 6 |
| Touch Lock Hero | ⭐⭐⭐ | 中 | ⭐⭐⭐⭐ | 7 |
| Page Transitions (Rive) | ⭐⭐ | 高 | ⭐⭐⭐⭐ | 8 |
