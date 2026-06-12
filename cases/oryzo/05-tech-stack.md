# Oryzo AI — 技术栈深度分析

---

## 一、构建管线

### Astro

Oryzo 使用 **Astro** 作为静态站点生成器：

```
astro.config.mjs
├── output: 'static'          # 静态导出 (SSG)
├── integrations: []
├── Vite (under the hood)     # 打包 JS/CSS
└── _astro/                   # 构建产物
    ├── hoisted.CRsATKbF.js   # 所有 JS 的单一 bundle (ES Module)
    └── index.TL6TuoJb.css    # 所有 CSS 的单一 bundle
```

**无框架** — Astro 页面组件输出纯 HTML，无 React/Vue/Svelte 客户端 hydration 标记。

### Vite 打包

- `hoisted.CRsATKbF.js` — 所有 JS 逻辑打成一个 ES Module 文件（代码分割可能针对较复杂的站点，这里单一 bundle）
- `index.TL6TuoJb.css` — 所有 CSS 打成一个文件
- 文件名 hash 用于缓存破除

---

## 二、依赖清单

| 依赖 | 角色 | 为什么选它 |
|---|---|---|
| **Three.js** | 3D 渲染引擎 (Hero 杯垫、Wearable 3D、曲线) | WebGL 标准，生态完善，Lusion 核心技能栈 |
| **@rive-app/canvas** | Rive 矢量动画渲染 | Canvas 渲染的矢量动画，适合 2D 插画 |
| **GSAP** | 动画引擎 (ScrollTrigger) | 业界标准的 scroll-driven 动画，配合 Lenis |
| **ScrollTrigger** | Scroll-driven 动画绑定 | 所有 data-offset 和视差效果 |
| **Lenis** | 平滑滚动 | 替代 locomotive-scroll，性能更好 |
| **@vimeo/player** | Vimeo 视频嵌入 API | 视频 overlay 控制（播放/暂停/静音） |

### 不需要的

| 技术 | 原因 |
|---|---|
| React / Vue / Svelte | 纯展示型营销页面，不需要响应式状态 |
| Tailwind CSS | 自定义 BEM 风格更适合 Lusion 的设计系统 |
| Webpack | Astro 内置 Vite，开箱即用 |
| Locomotive Scroll | Lenis 更轻量 |

---

## 三、Three.js 渲染管线

### Hero 场景

```javascript
// 初始化
const renderer = new THREE.WebGLRenderer({
  canvas: document.getElementById('canvas'),
  alpha: true,
  antialias: true,
})

// 渲染设置
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
renderer.toneMapping = THREE.ACESFilmicToneMapping
renderer.toneMappingExposure = 1.2

// 模型加载
const loader = new GLTFLoader()
// 或 Lusion 的自定义 Houdini → GLTF 管线
```

### 管线特性

| 特性 | 值 |
|---|---|
| Tone Mapping | ACES Filmic |
| Exposure | 1.2 |
| Pixel Ratio | min(DPR, 2) — 性能保护 |
| Background | Alpha (透明, 由 CSS 背景控制) |
| Shadows | 不需要 (环境照明足够) |

### Lusion 的生产管线

Lusion 的 3D 资产管线：

```
Houdini FX (程序化几何体) → OBJ/GLTF 导出
  │
  ▼
Three.js 运行时
  ├─ GLTFLoader 加载模型
  ├─ 材质调整 (roughness/metalness)
  ├─ HDR 环境贴图 (Studio 光照)
  └─ Scroll-driven 动画
```

---

## 四、CSS 体系

### 命名规范

```
.o-*         → Object (布局)  .o-container, .o-grid, .o-dashline
.is-*        → State (状态)   .is-dark, .is-active, .is-flipper
.__*         → Element (BEM) .section__inner, .hero-video__border
```

### 文字比例尺

```css
.sub1 { /* 最小字号 */ }
.sub2 { }
.body1 { /* 正文字号 */ }
.body2 { }
.body3 { /* 最大字号 */ }
.quote { }
```

### 响应式

```css
.desktop-only { display: block; }
.mobile-only  { display: none; }

@media (max-width: 767px) {
  .desktop-only { display: none; }
  .mobile-only  { display: block; }
}
```

所有图片使用 `<picture>` 实现桌面/移动端版本：

```html
<picture>
  <source srcset="edge.webp" media="(min-width: 768px)" />
  <img src="edge_MOBILE.webp" loading="lazy" alt="" />
</picture>
```

---

## 五、性能策略

### 1. Three.js Canvas 优化

```javascript
// 限制像素比
const dpr = Math.min(window.devicePixelRatio, 2)

// 视口外暂停渲染
ScrollTrigger.create({
  trigger: '#canvas',
  start: 'top top',
  end: 'bottom bottom',
  onLeave: () => {
    renderer.setAnimationLoop(null)  // 暂停渲染循环
  },
  onEnter: () => {
    renderer.setAnimationLoop(animate)
  },
})
```

### 2. 图片加载

```javascript
// loading="lazy" + decoding="async"
// 非首屏图片延迟加载
// Social Content Grid 使用 1x1 透明占位 GIF，JS 替换为真实图片
```

### 3. Rive 暂停策略

```javascript
// 视口外暂停 Rive
ScrollTrigger.create({
  trigger: '#sustainability',
  onLeave: () => {
    riveInstance.pause()
  },
  onEnterBack: () => {
    riveInstance.play()
  },
})
```

---

## 六、satirical UX 设计注记

oryzo 的技术实现服务于一个核心创意：**超级认真的产品发布语气 × 荒谬的产品（杯垫）**。

| 技术 | 讽刺用途 |
|---|---|
| Three.js 3D 模型 | 给一个杯垫做 360° 产品展示，仿佛它是 iPhone |
| Rive 动画 | 给软木收获做交互动画，好像这个杯垫需要复杂供应链 |
| Paper / BibTeX / Checkpoints | AI 论文格式发布杯垫 3D 模型 (OBJ) |
| "Open-weight model" | 双关语：AI open-weight + 杯垫物理重量 |
| Product Variants (Pro/Pro Max) | 杯垫分三档：单层/双层/三层 |
| "Powered by AI* (*Adobe Illustrator)" | 脚注拆穿 AI 营销话术 |
| "RTX 3090 required" | 嘲讽 AI 产品的高硬件要求 |
| "Runs on the edge / Refuses the cloud" | 杯垫真的不需要联网 |

---

## 七、站点架构图

```
Astro Build Pipeline
│
├── src/pages/
│   └── index.astro              # 单页应用 (所有内容在一页)
│
├── src/components/
│   ├── Hero.astro
│   ├── WearableGallery.astro
│   ├── Features.astro
│   ├── Sustainability.astro
│   ├── SocialContent.astro
│   ├── ProductSelector.astro
│   ├── OpenWeight.astro
│   └── Footer.astro
│
├── src/scripts/
│   ├── three-scene.js           # Three.js 主场景
│   ├── wearable-gallery.js      # 水平画廊
│   ├── scroll-effects.js        # ScrollTrigger 绑定
│   ├── rive-sustainability.js   # Rive 初始化
│   └── main.js                  # 入口
│
├── public/
│   ├── models/                  # GLTF 3D 模型
│   ├── rive/                    # .riv 动画文件
│   ├── images/                  # WebP 图片
│   ├── env/                     # HDR 环境贴图
│   └── video/                   # 产品视频
│
└── astro.config.mjs
```
