# Luke Baffait — Design Rationale & Application Guide

Luke Baffait 的个人作品集展示了"一个人的能力上限"。它的设计哲学：**不用框架、不用库、不抄模板——每行代码都是自己的想法。**

---

## 一、设计语言分析

### 视觉 DNA

```
暗色调 + 高对比 ← 底色
     ↓
自定义 WebGL 渐变背景（不断旋转）
     ↓
341 帧手绘级序列（Canvas 2D）
     ↓
有机形状 + 手工艺感（Blob、ASCII Art、流体线）
```

**为什么这套语言有效：**

1. **暗色调** — 个人作品集用深色背景，让"发光/亮色"内容更突出。对比 Zentry 也用暗色，但 Luke 的暗色更"安静"——它不喊叫，只是存在，让内容自己说话。

2. **自定义 WebGL 背景** — 当别人的背景是纯色或渐变时，Luke 的背景是一个在 Perlin 噪声流场中不断扭曲的 4 层着色器。它不会夺走注意力，但如果你仔细看——"这个背景在动"——这种"低调的复杂"是个人作品集最有效的表达能力："我不说我很厉害，但你看一眼就知道。"

3. **341 帧序列** — 不是视频，不是 3D 模型——是逐帧绘制的图像序列。这传达了"手工艺感"——每一帧都是一个人手绘或精心渲染的。在 AI 生成内容泛滥的 2026 年，手工艺感是稀缺品。

4. **Blob + ASCII Art** — 两个"不实用但有性格"的元素。Blob 是"不必要的酷"，ASCII Art 是"不必要的复杂"——但它们共同塑造了"这个人有自己的品味"的印象。

---

## 二、每个效果的 Design Rationale + Application Guide

### 1. Preloader — 红黑幕布 + 名字逐字弹入

**为什么好看：**
1. **红黑幕布** — 不是常见的"转圈圈"或"进度条"，而是两侧幕布从中间拉开。这致敬了舞台开幕——暗示"这是一个表演"。
2. **名字逐字弹入** — 名字的每个字母从下往上弹入，带着 overshoot（过冲）。不是"出现"而是"跳到位置并微微回弹"——让名字有重量感。
3. **文字跳动的节奏** — STARKE（姓）和 LUKE（名）是两组独立的动画。姓先出现，名紧随其后——形成"组合"的节奏感。

**适合放在：** 个人网站、工作室网站、创意 agency 网站——任何想传递"这是一个作品"而不是"这是一个产品"的网站。

**不适合：** 产品站、SaaS、电商（用户想尽快使用产品，不需要仪式感）。

### 2. 自定义 WebGL CoreRenderer

**为什么不用 Three.js：** Luke 自己写了一个 WebGL 渲染器，因为：
1. **这是他的作品集** — 使用 Three.js 不能展示"他本人"的能力。自建渲染器本身就是能力证明。
2. **他只需要 4 个效果** — gradient + parallax + Perlin flow + Gaussian blur。Three.js 是一个全功能渲染引擎，加载它再只用 4 个功能太重了。
3. **GLSL 控制** — 自建渲染器意味着他可以精确控制每一行着色器代码，而不是在 Three.js 的抽象上修修改改。

**对于你的项目：** 除非你的项目核心价值就是"展示我是一个 WebGL 专家"，否则**不要自建渲染器**。Three.js 在 99% 的场景中都是更好的选择。

### 3. Circle Gallery — CSS 3D 圆柱画廊

**原理：** 每张图片被切成 10 条垂直 strip，每条以 `transformOrigin: 50% 50% -Rpx` 设置旋转中心，再以 `rotateY(i * 36deg)` 让 strip 围成一圈，形成圆柱。

**为什么好：** 普通画廊是平面的——左右滑动或网格排列。Circle Gallery 是"环绕"用户的空间——用户感觉自己在画廊"内部"而不是在"看"画廊。

**适合放在：**
- **个人作品集**（展示多个项目时）
- **产品 360° 展示**（用真实图片 slices 而非 3D 模型）
- **团队展示**（环绕排列成员）

**改编方案 — 简化版：**
```css
.gallery-slice {
  position: absolute;
  width: calc(100% / 10);
  height: 100%;
  transform-origin: 50% 50% -300px;  /* -R */
  transform: rotateY(var(--slice-angle));  /* 0, 36, 72, 108... */
  background-image: var(--img-url);
  background-position: calc(var(--slice-index) * -10%) 0;
  background-size: 1000% 100%;
}
```
实际上这个效果实现较复杂，适合作为"签名式"效果放在个人站，但不适合大规模使用。

### 4. Word-Level Blur Reveal

**为什么好：** 每个 `word` 获得独立的 ScrollTrigger。当 word 进入视口时，`filter: blur(8px)` → `blur(0px)`，`opacity: 0 → 1`。

**这个效果比 char split 好在哪：**
- **阅读友好** — 用户按 word 读，不是按 char。word 级别不会打断阅读流。
- **渐进聚焦** — 模糊 → 清晰模拟"眼睛在重新对焦"——视觉系统对焦的过程本身就是逐渐清晰的。
- **大规模可用** — word 级别需要更少的 DOM 节点（一篇文章假如 200 个 word vs 800 个 char），性能更好。

**适合放在：** 文章页面、关于页面、正文内容区。

### 5. 3D Project Card Tilt

**实现：** 鼠标在卡片上移动 → 计算卡片内相对坐标 → 映射为 `rotateX` / `rotateY`。用 `gsap.ticker`（RAF 循环）驱动，lerp 阻尼 0.12。

**为什么好：** 与 Zentry 的 perspective tilt 相同原理。但 Luke 用 `gsap.ticker` 而不是 Vue composable——因为他的项目没有框架，纯 JS。

**改编为纯 JS 模式：**
```javascript
const cards = document.querySelectorAll('.project-card')
cards.forEach(card => {
  let targetX = 0, targetY = 0
  let currentX = 0, currentY = 0

  card.addEventListener('mousemove', (e) => {
    const rect = card.getBoundingClientRect()
    const x = (e.clientX - rect.left) / rect.width  // 0-1
    const y = (e.clientY - rect.top) / rect.height
    targetX = (y - 0.5) * 10   // -5 to 5 deg
    targetY = (0.5 - x) * 10
  })

  card.addEventListener('mouseleave', () => {
    targetX = 0
    targetY = 0
  })

  gsap.ticker.add(() => {
    currentX += (targetX - currentX) * 0.12
    currentY += (targetY - currentY) * 0.12
    card.style.transform =
      `perspective(800px) rotateX(${currentX}deg) rotateY(${currentY}deg)`
  })
})
```

### 6. Contact Blob — 300vmax 缩放

**为什么好：** 一个 300vmax 的圆形从 scale(0) → scale(1)。因为圆形太大（300vmax = 最大视口尺寸的 3 倍），缩放时看起来像一个液泡从底部"涨"出来——就像水面上涨。

**适合放在：** Contact section、CTA 区域、"见证"类内容。

**性能注意：** 300vmax 的圆形渲染对 GPU 没有负担（CSS 圆形的渲染成本与尺寸无关——GPU 只关心片段数量，而圆形始终是 2 个三角形）。

---

## 三、整体可复用性评分

| 效果 | 可复用性 | 开发成本 | 视觉冲击 | 推荐优先级 |
|------|---------|---------|---------|-----------|
| Word-Level Blur Reveal | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐⭐ | 1 |
| 3D Card Tilt | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐ | 2 |
| Contact Blob | ⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐ | 3 |
| Preloader 幕布 + 名字 | ⭐⭐⭐⭐ | 中 | ⭐⭐⭐⭐ | 4 |
| Circle Gallery | ⭐⭐ | 高 | ⭐⭐⭐⭐⭐ | 5（签名式效果） |
| ASCII Art Footer | ⭐⭐ | 高 | ⭐⭐⭐⭐ | 6 |
| 自建 WebGL Renderer | ⭐ | 极高 | ⭐⭐⭐⭐⭐ | 7（仅展示能力时） |
| 341 帧序列 | ⭐⭐ | 高 | ⭐⭐⭐⭐ | 8（有逐帧资源时） |
