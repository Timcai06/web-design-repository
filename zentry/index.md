# Zentry — Technical Case Study

**URL:** https://zentry.com  
**Studio:** [Resn](https://resn.co.nz/) (Wellington, New Zealand)  
**Client:** Zentry (Web3 / Gaming / AI)  
**Date Analyzed:** 2026-05-29

---

## 站点概览

Zentry 是一个 Web3 游戏生态品牌，官网由 Resn 设计开发。全站以**高能量、影院级视觉叙事**为特色，采用"无 WebGL"的 DOM-only 3D 效果路线——这是 Resn 刻意区别于 Lusion 的技术选择。

### 核心体验流程

```
进入首页
  │
  ├─ Preloader (品牌 logo 动画)
  │
  ├─ Hero 区域
  │   ├─ "REDEFINE REALITY" 双词渐进揭示
  │   ├─ 视频背景 + 3D Portal Mask (clip-path)
  │   ├─ HUD 风格导航 (浮动, 滚动方向感知)
  │   └─ 音频可视化器 (navbar)
  │
  ├─ Intro 区域
  │   ├─ 大标题 "Welcome to Zentry"
  │   ├─ 文字逐词/逐字入场
  │   └─ 视频背景
  │
  ├─ Nexus / Products 区域
  │   ├─ Bento Grid 布局 (3 列)
  │   ├─ 3D 悬停倾斜 (perspective tilt)
  │   ├─ 卡片 hover mask 揭示
  │   └─ 文字 clip-path 擦除
  │
  ├─ ZENT Token 区域
  │   ├─ 3 列卡片 (Shaping / Unlocking / Sharing)
  │   ├─ 滚动驱动入场
  │   └─ 数字计数器动画
  │
  ├─ Who We Are 区域
  │   ├─ 大标题动画
  │   ├─ 统计数字滚动
  │   └─ 3D Spline 元素
  │
  ├─ Partners / Backers 区域
  │   ├─ Logo 滚动条 (marquee)
  │   └─ Scroll-driven reveal
  │
  └─ Footer
      ├─ CTA "Build the foundation for tomorrow's civilization"
      ├─ 网格链接
      └─ 主题切换
```

### 核心技术决策

**"No WebGL for core effects"** — Resn 刻意避免使用 WebGL/Three.js 渲染主要视觉效果（3D Portal Masks），改用 **CSS `clip-path` + JS 驱动 DOM 变换**。Three.js 仅用于 Spline 嵌入的 3D 元素（非核心渲染）。

### 技术栈

| 技术 | 用途 |
|---|---|
| **Nuxt 3 + Vue 3** | 前端框架 |
| **Storyblok** | Headless CMS |
| **GSAP + ScrollTrigger** | 全部 scroll-driven 动画 |
| **Lenis** | 平滑滚动 |
| **Three.js (模块化)** | Spline 3D 元素嵌入 |
| **Spline** | 3D 设计 → Three.js 导出 |
| **ScrollyVideo** | Scroll-driven 视频播放 |
| **Vite** | 构建工具 |
| **CSS `clip-path`** | Portal Mask、文字揭示、卡片 hover |

### 自定义 Vue Composables

| Composable | 用途 |
|---|---|
| `useSplitText` | 文字拆分为字符/单词，逐项动画 |
| `useCssClipMask` | CSS clip-path 蒙版动画（Portal Mask） |
| `useDamp` | 阻尼/滞后数值（平滑跟随鼠标） |
| `usePerspectiveTransform` | 3D 透视倾斜跟随鼠标 |
| `usePerspectiveFadeEntry` | 带透视的淡入动画 |
| `useBaseScrollTrigger` | GSAP ScrollTrigger 封装 |
| `useTheme` | 颜色主题切换 |
| `useWindowPointer` / `useWindowPointerNorm` | 鼠标位置跟踪 |
| `useThresholdTrigger` | 阈值触发 |
| `useRaf` / `useRafBool` | requestAnimationFrame 管理 |
| `useOnScroll` | 滚动事件封装（与 Lenis 集成） |

---

## 文件索引

| 文件 | 内容 |
|---|---|
| `01-hero-portal.md` | Hero 视频背景、3D Portal Mask (clip-path)、HUD 导航、音频可视化 |
| `02-bento-grid.md` | Bento Grid 布局、perspective tilt hover、卡片 mask reveal |
| `03-scroll-effects.md` | 文字拆分动画、clip-path 擦除、数字计数器、Spline 3D、marquee |
| `04-tech-stack.md` | Nuxt+Vue 架构、composables 系统、性能策略、Resn vs Lusion 对比 |
