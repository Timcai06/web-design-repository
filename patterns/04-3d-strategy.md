# 3D Strategy — 三种 3D 路线的设计决策

5 个案例展示了三种完全不同的 3D 方案。选哪条路不是技术问题，是设计决策。

---

## 一、方案对比

| 维度 | CSS + DOM (Zentry) | Three.js (Oryzo) | 自定义 WebGL (Luke/Persepolis) |
|------|-------------------|-----------------|-------------------------------|
| **代表** | Zentry | Oryzo | Luke Baffait, Persepolis |
| **渲染核心** | CSS `clip-path` + `perspective` | Three.js WebGL | 自建 GLSL 着色器 / Unity 导出 |
| **视觉上限** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐⭐⭐ |
| **开发成本** | 低-中 | 中-高 | 极高 |
| **加载时间** | 极快 (<2s) | 中等 (2-7s) | 慢 (5-20s) |
| **SEO** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ (canvas 不可索引) | ⭐⭐ |
| **维护难度** | 低（标准 CSS） | 中（Three.js 生态） | 极高（自研） |
| **最适规模** | 品牌站 / SaaS | 产品营销 / 游戏站 | 博物馆 / 顶级 showreel |

---

## 二、决策树

```
需要 3D 效果？
├── 效果可以用 CSS 模拟？
│   ├── ✅ 用 CSS (clip-path + perspective + transform)
│   │    → Zentry 路线：DOM-only，零 JS 负担
│   │    → 适合：品牌官网、SaaS、个人站
│   │    → 成本：Low
│   │
│   └── ❌ 需要真正的 3D 渲染？
│       ├── 需要展示真实的 3D 模型/场景？
│       │   ├── ✅ Three.js (GLTF/GLB 导入)
│       │   │    → Oryzo 路线：标准 3D 管线
│       │   │    → 适合：产品展示、营销页
│       │   │    → 成本：Medium
│       │   │
│       │   └── 需要极致性能/独特渲染？
│       │       └── ✅ 自定义 WebGL
│       │            → Luke/Persepolis 路线
│       │            → 适合：博物馆、顶级作品
│       │            → 成本：Very High
│       │
│       └── 只需要 "3D 幻觉"？
│           └── ✅ CSS perspective + tilt
│               → Zentry usePerspectiveTransform
│               → 适合：卡片、grid、面板
│               → 成本：Very Low
```

---

## 三、关键设计决策分析

### 为什么 Zentry 选择不做 WebGL？

> "任何用 WebGL 实现的效果，如果 CSS 也能做，就永远不要用 WebGL。"

Resn 的哲学基于：
1. **加载时间** — WebGL 需要加载着色器编译 + 纹理上传 + 模型解析。Zentry 的 clip-path portal 在 2G 网络上也能几乎即时响应。
2. **内存** — CSS 合成层由浏览器原生管理。WebGL 需要手动管理 GPU 内存，复杂场景容易 OOM。
3. **交互一致性** — CSS 动画与浏览器的滚动、触摸、resize 事件天然同步。WebGL 需要手动处理所有这些事件绑定。
4. **SEO + 可访问性** — DOM 内容可以被搜索引擎索引，可以被 screen reader 读取，可以缩放。

**但代价是什么？** CSS clip-path 的"3D"效果只能从特定角度看。真正的 3D（比如 Oryzo 的杯垫）不可能用 CSS 实现。

### 为什么 Oryzo 选择 Three.js？

Lusion 的每个项目都在用 Three.js，因为：
1. **产品需要** — 杯垫模型需要 360° 旋转观察，CSS 做不到
2. **品牌语言** — Lusion 的品牌就是"WebGL 炫技"，这是他们的卖点
3. **复用性** — 他们有成熟的 Three.js 管线，开发成本已被摊薄

**代价：** 首屏加载约 3-5s（含 Three.js runtime + 模型 + 纹理），长尾设备发热。

### 为什么 Luke 选择自建 WebGL？

这是他个人作品集，目标是**展示他作为"WebGL 专家"的能力**。自建渲染器本身就是作品的一部分——他在告诉潜在雇主："我不仅能调 Three.js API，我能写 GLSL。"

**这 5 个案例中只有 Luke 值得自建渲染器。** 对于其他 99% 的项目，Three.js 就足够了。

---

## 四、CSS 3D Illusion 技巧汇总（零成本方案）

### 技巧 1：Perspective Tilt（Zentry）

```vue
// 鼠标位置 → CSS transform
const { x, y } = useWindowPointerNorm() // -1 to 1
const tiltX = computed(() => y.value * 8)  // ±8deg
const tiltY = computed(() => x.value * -8)

// 应用到卡片
.card {
  perspective: 800px;
  transform: rotateX(${tiltX}deg) rotateY(${tiltY}deg);
  // 加 transition 制造阻尼平滑感
  transition: transform 0.1s ease-out;
}
```

**为什么"够用"：** 人眼对缓慢的 3D 旋转容忍度很高。只要 transformation 是 smooth 的、跟随鼠标直觉的，大脑就会接受"这是一个 3D 物体"。

### 技巧 2：多层视差堆叠（Luke Baffait WebGL）

将背景 + 内容 + 前景拆成不同层，每层以不同速度/方向移动。

```
前景: speed 1.2 (跟随鼠标)
中景: speed 1.0 (常规)
背景: speed 0.6 (慢速)
```

**为什么"够用"：** 这是电影中"多平面摄影"（multiplane camera）的原理。即使每层都是 2D 的，不同层的相对运动也会产生 3D 纵深知觉。

### 技巧 3：Scale + Blur 模拟 Z 轴（Oryzo Grip）

```
scroll 0%:  scale(1.0)  blur(4px)
scroll 100%: scale(1.3)  blur(0px)
```

**为什么"够用"：** 人类视觉系统中，物体靠近时 = 变大 + 变清晰。反之亦然。Scale + Blur 同时变化是这个运动视觉的完美模拟。

---

## 五、性能预算指南

| 用户设备 | CSS 3D | Three.js 简单 | Three.js 复杂 | 自定义 WebGL |
|---------|--------|-------------|-------------|------------|
| iPhone 12+ | ✅ 60fps | ✅ 60fps | ⚠️ 30-60fps | ⚠️ 30-60fps |
| iPhone SE/X | ✅ 60fps | ⚠️ 30fps | ❌ <30fps | ❌ <30fps |
| MacBook M1 | ✅ 60fps | ✅ 60fps | ✅ 60fps | ✅ 60fps |
| 低端 Android | ✅ 60fps | ❌ <30fps | ❌ <20fps | ❌ Not possible |
| iPad | ✅ 60fps | ⚠️ 30-60fps | ❌ 20-30fps | ❌ 20-30fps |

**经验法则：** 如果目标用户包含移动端（占比 >30%），不要用 WebGL。用 CSS 3D illusion 方案。
