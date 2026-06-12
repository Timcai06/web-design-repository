# Oryzo AI — Design Rationale & Application Guide

Oryzo 是 Lusion 的"炫技之作"。它的设计哲学：**用最顶级的技术做一个最 absurd 的产品，反差本身即是幽默。**

---

## 一、设计语言分析

### 视觉 DNA

```
高保真 3D 产品模型（Three.js） ← 核心视觉
     ↓
干净的白色/浅色背景 + 产品高亮
     ↓
精确的交互控件（滑块、拖拽、翻转）
     ↓
"太认真了" 的讽刺文案
```

**为什么这套语言有效：**

1. **3D 产品模型 + 旋转展示** — 模仿 Apple 式的产品展示语言。"严肃地展示一个荒谬的产品"——反差制造幽默。3D 模型的质感（金属触点、软木纹理、编织表面）让杯垫看起来"真的值得被认真对待"。

2. **全白/浅色背景** — Apple 式简约。浅色背景让 3D 模型成为唯一的视觉焦点，没有任何干扰。同时白色在科技行业=干净/现代/专业——与"卖杯垫"产生第二层反差。

3. **交互控件的精确感** — 温度滑块有精确的数值显示，3D 曲线有坐标轴，产品配置有三档选择器。这些 UI 控件模仿"专业工具"界面——第三层反差："卖杯垫"却用了专业软件级别的交互。

4. **伪学术内容** — GitHub Paper 风格页面、BibTeX 引用、Model Checkpoints。字体/排版完全模仿 AI 论文——第四层反差。"AI 论文"是最严肃的学术形式，用来介绍一个"杯垫"。

**三层反差叠加的效果：** 用户访问时的心理路径："这个产品看起来真的很厉害...等等...这是个杯垫？...等等...他们还写了论文？...哈哈哈哈" → 记住品牌 → 分享。

### Oryzo 的设计可迁移模式

Oryzo 的"反差营销"设计模式可以被迁移到：
1. **普通产品 → 用顶级 3D 展示**（即使产品很简单）
2. **严肃 tone → 荒谬内容**（制造幽默和分享动机）
3. **专业交互控件 → 简单功能**（让用户感觉"这个产品很精密"）

---

## 二、每个效果的 Design Rationale + Application Guide

### 1. Hero — 全屏 3D 模型 + 滚动驱动旋转

**为什么好看：** 
1. 模型绕 Y 轴 0-360° 旋转 = 用户完整看完产品所有面
2. 旋转速度与滚动速度同步 = "用户控制展示节奏"
3. 模型从缩放 0.8 → 1.0 + 位置偏移 = 模拟"产品从远处靠近"

**最适合的场景：** 任何**实体产品**的 landing page（硬件、消费品、电子产品、服装）。

**改编方案 — 通用 3D 产品展示器：**
```javascript
// Three.js 场景
const model = loadGLTF('product.glb')
ScrollTrigger.create({
  trigger: '.hero',
  start: 'top top',
  end: 'bottom top',
  scrub: 1,
  onUpdate: (self) => {
    model.rotation.y = self.progress * Math.PI * 2  // 0-360°
    model.scale.setScalar(0.8 + self.progress * 0.2)  // 0.8-1.0
  }
})
```

### 2. Preloader — WebGL 动画

**为什么好看：** Preloader 不是简单的"进度条 + logo"——它是一个微缩的 WebGL 动画场景。用户等待时在看一段"小电影"而不是一个进度条。

**适合放在：** 有 WebGL/3D 内容的页面。Preloader 本身就应该暗示"等内容加载好你会看到更多这样的效果"——它是"预告片"而非"等待提示"。

**改编指南：** 如果页面本身没有 WebGL 内容，不要用 WebGL preloader——会误导用户预期。

### 3. Wearable Gallery — 水平滚动画廊

**为什么好：** 水平 + 自动播放 + 拖拽，三个交互模式的组合适应不同用户习惯：
- 想看下一张？→ 自动播放 4s 后自动推进
- 想控制在某张？→ 拖拽/滚动暂停自动播放
- 想快速浏览？→ 拖拽滑动
- 大图预览跟随 — 当前格放大显示，制造"选中"的视觉反馈

**适合放在：**
- **产品使用场景展示**（杯垫在不同场景中）
- **服装/穿戴产品展示**
- **案例/作品集**（每个项目一张缩略图）

### 4. Temperature Slider + Three.js 曲线

**为什么好：** 交互式的 3D 可视化滑块——用户拖动滑块时，一条粒子沿着 Cubic Bezier 曲线运动，粒子的颜色从蓝色平滑过渡到橙色（温度+颜色映射）。

**这是"功能可视化"的典范：** 即使用户不看文案，仅仅拖动滑块就能理解"这个产品可以调节温度"。

**适合放在：** 需要展示"变量变化"的产品功能——温度、速度、压力、时间推移。

**改编方案：**
```javascript
// 通用参数可视化
const curve = new THREE.CubicBezierCurve3(v0, v1, v2, v3)
// 滑块值 0-1 映射到曲线位置
slider.addEventListener('input', (e) => {
  const t = e.target.value / 100
  const point = curve.getPoint(t)
  particle.position.copy(point)
})
```

### 5. Scroll-Driven Grip Zoom

**为什么好：** 滚动驱动 scale(1.0→1.3) + blur(4px→0px)。blur 的消退配合 scale 的放大，模拟"从远到近的聚焦"——或者说"从模糊到清晰"。

**感觉像什么：** 手机相机在自动对焦——物体从模糊变清晰的过程。这种视觉"太自然了"，用户不会意识到这是设计的效果，但会感觉"这个区感觉很好"。

**改编方案：**
```css
.grip-zoom {
  transform: scale(1);
  filter: blur(4px);
  transition: none; /* GSAP 控制 */
}
```
```javascript
ScrollTrigger.create({
  trigger: '.grip-section',
  start: 'top bottom',
  end: 'bottom top',
  scrub: 1,
  onUpdate: (self) => {
    const p = self.progress
    gsap.set('.grip-zoom', {
      scale: 1 + p * 0.3,
      filter: `blur(${4 * (1 - p)}px)`
    })
  }
})
```

**适合放在：** 需要强调某件产品的"细节"时——滚动放大产品局部，blur 消退让细节更清晰。

### 6. Rive Sustainability Section

**为什么用 Rive 而不是 Three.js：** 这个区域展示的是"软木收获过程"——2D 矢量插画 + 循环动画。Rive 比 Three.js 轻量得多（`.riv` 文件通常 <100KB vs Three.js 模型可能 >2MB），且矢量动画在表现"流程图/过程图"时更清晰。

**决策原则：** "用最轻的工具完成工作"——3D 不能用 CSS 才用 Three.js，2D 动效用 Rive/Lottie。

### 7. Social Content Grid

**为什么好：** 网格元素 staggered `clip-path: inset()` 展开——每个元素从上到下依次出现，形成"波浪"。每个元素有自己的 `data-offset` 控制延迟偏差，让波浪不是均匀的——有不规则的"手工感"。

**适合放在：** 任何社交证明（testimonials、用户案例、媒体报道）的展示区域。

---

## 三、整体可复用性评分

| 效果 | 可复用性 | 开发成本 | 视觉冲击 | 推荐优先级 |
|------|---------|---------|---------|-----------|
| 3D 产品旋转 Hero | ⭐⭐⭐⭐ | 高 | ⭐⭐⭐⭐⭐ | 1（有 3D 产品时） |
| Scroll-Driven Zoom + Blur | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐⭐ | 2 |
| 参数可视化 Slider | ⭐⭐⭐⭐ | 中 | ⭐⭐⭐⭐ | 3 |
| Staggered Grid Reveal | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐⭐ | 4 |
| 水平 Gallery | ⭐⭐⭐⭐ | 中 | ⭐⭐⭐⭐ | 5 |
| Rive 矢量动画 | ⭐⭐⭐ | 中 | ⭐⭐⭐ | 6（有 Rive 资源时） |
| 讽刺 Tone 文案策略 | ⭐⭐⭐ | 低 | ⭐⭐⭐⭐⭐ | 7（品牌调性匹配时） |
| Flip Encryption Demo | ⭐⭐ | 低 | ⭐⭐ | 8 |
