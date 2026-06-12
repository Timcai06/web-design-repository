# 3D 环境 + 粒子管系统

ATv6 的视觉核心：**滚动即旅行**——每次滚动都在一个霓虹灯风格的 3D 世界中推进。

---

## 一、Scroll-Driven 3D 环境

### 效果描述

与 Lusion 和 Zentry 不同，ATv6 的滚动不是触发"动画"或"揭示内容"，而是**在一个完整的 3D 世界中推进相机位置**。

```
滚动前: 相机在位置 A (走廊起点)
   │
   ▼
滚动中: 相机沿路径滑行 (穿过走廊)
   │
   ▼
滚动后: 相机在位置 B (新区域起点)
   │
   ▼
内容面板淡入 (项目介绍)
```

### 设计 Rationale

**为什么 ATv6 选择"世界中的旅行"而不是"页面上的动画"？**

1. **叙事连续性** — 传统的 scroll-triggered 动画是"片段式"的：A 效果触发 → 结束 → B 效果触发。ATv6 的 3D 世界推进是**连续的**——你始终在同一个空间中前进。
2. **环境叙事** — 场景基于 Active Theory 的 LA 和 Amsterdam 办公室。用户在浏览作品集的同时，也在"参观"这个公司的物理空间。这比"在白页上看项目卡片"传达了更多品牌信息。
3. **"展示而非讲述"** — ATv6 不写"我们是一个创意工作室"，它让你"走进"一个创意工作室。

### 技术实现

```javascript
// Hydra 场景中的相机路径
const cameraPath = [
  { position: [0, 1.6, 5], target: [0, 1.6, 0] },      // 起点
  { position: [2, 1.6, 12], target: [0, 1.6, 5] },     // 1/4
  { position: [-1, 1.8, 20], target: [0, 1.8, 10] },   // 1/2
  { position: [0, 2, 30], target: [0, 1.8, 15] },      // 3/4
  { position: [0, 2, 40], target: [0, 2, 25] },         // 终点
]

// 滚动绑定
window.addEventListener('scroll', () => {
  const progress = getScrollProgress()
  const keyframe = interpolatePath(cameraPath, progress)
  camera.position.copy(keyframe.position)
  camera.lookAt(keyframe.target)
})
```

### 与类似效果的对比

| 项目 | 滚动驱动 | 3D 环境 | 相机路径 |
|------|---------|---------|---------|
| **ATv6** | ✅ 连续 | ✅ 完整世界 | ✅ 自由曲线 |
| **Persepolis** | ❌ 点击推进 | ✅ 完整世界 | ✅ 固定路径 |
| **Oryzo** | ✅ 连续 | ❌ 单一模型 | ❌ 模型旋转 |
| **Luke** | ✅ 连续 | ❌ 背景着色器 | ❌ 无 |

---

## 二、彩色粒子管系统

### 效果描述

鼠标/触摸滑动时产生**管状的彩色粒子轨迹**。管的颜色不断变化，粒子沿着曲线路径流动。

### 技术实现

```javascript
// 粒子管 — 简化示意
class ParticleTube {
  constructor() {
    this.particles = []
    this.trail = []  // 存储鼠标路径点
  }

  update(mouseX, mouseY) {
    this.trail.push({ x: mouseX, y: mouseY })

    // 每帧从路径采样粒子位置
    this.particles.forEach((p, i) => {
      const point = this.trail[
        Math.floor((i / this.particles.length) * this.trail.length)
      ]
      p.position.set(point.x, point.y, 0)
      p.color = this.getColor(i)  // HSL 渐变
    })
  }
}
```

### 设计 Rationale

**为什么粒子管而不是简单的鼠标轨迹？**

1. **物理模拟感** — 粒子管有"流动感"，像水或光在管道中流动。这比"鼠标后面拖一条线"更有生命力。
2. **网络化** — 当这些管来自其他访客时，它们变成了"陌生的他在这个空间中的痕迹"——一种异步社交信号。
3. **技术炫示** — 粒子系统是 WebGL 性能的经典 benchmark。流畅的粒子管告诉用户"这个网站的技术底子很厚"。

### 适合的场景

| 场景 | 适合度 | 说明 |
|------|--------|------|
| 技术工作室 / 创意站 | ✅✅ | 直接展示技术能力 |
| 产品展示 | ⚠️ | 如果与品牌调性匹配（如音乐/游戏） |
| 内容站 | ❌ | 干扰阅读 |
| 电商 | ❌ | 干扰购买决策 |

---

## 三、霓虹灯光风格

### 视觉语言分析

ATv6 的视觉风格关键词：**霓虹、外星、末世感**。

```
暗色基底 (纯黑背景)
   ↓
发光元素 (霓虹灯管、发光字体)
   ↓
粒子光效 (彩色管、闪烁)
   ↓
倾斜/扭曲 (非标准视角、畸变)
```

**为什么选这套视觉语言：**

1. **反主流** — 2024 年前后的网页设计趋势是"干净、留白、柔和"。ATv6 的黑暗 + 霓虹在视觉上形成了强烈对比——它不追赶潮流，这本身就是态度。
2. **技术能力信号** — 黑暗场景的 WebGL 渲染比亮色场景更难（需要正确处理发光、bloom、HDR）。选择黑暗 + 霓虹意味着"我们不满足于简单的事"。
3. **品牌记忆** — 如果你见过一次 ATv6 的霓虹走廊，你不会忘记它。视觉独特性 = 品牌记忆度。

### 调参指南

```css
/* 霓虹感 = 发光 + 高频色彩 + 高对比 */
.neon-text {
  color: #ff00ff;
  text-shadow:
    0 0 7px #ff00ff,
    0 0 10px #ff00ff,
    0 0 21px #ff00ff,
    0 0 42px #ff00ff;  /* 多层 shadow 模拟发光 */
}

/* 闪烁效果（霓虹灯管的"不稳定"感） */
@keyframes flicker {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.8; }
  92% { opacity: 1; }
  94% { opacity: 0.6; }
  96% { opacity: 1; }
  98% { opacity: 0.3; }
}
```

**注意：** 霓虹风格需要搭配正确的品牌调性。不适合：金融、医疗、教育、B2B SaaS。
