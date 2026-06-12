# Luke Baffait — 源码级拆解

**URL:** https://www.lukebaffait.fr  
**分析时间:** 2026-06-02  
**分析方法:** 从实际部署的源文件逐行阅读，所有引用皆来自真实源码

---

## 核心发现

与常见的知识库推测不同，实际源码揭示：

1. **不是 Three.js** — Hero 背景是自建 WebGL 渲染器 `CoreRenderer`，编译 GLSL 300 es 着色器，4 层叠加（渐变 + 图片视差 + Perlin 流场 + 高斯模糊）
2. **不是 Primer CSS** — 无任何 CSS 框架，全部手写 CSS，使用 `@font-face` 自定义字体 Breton / other / Zirena
3. **不是多种 JS 文件** — 实际结构：`i18n.js` + `core-renderer.js`(bundled) + `hero-project.js`(数据) + `index.js`(2411行主逻辑)

---

## 源码架构

```
lukebaffait.fr/
├── index.html
├── styles/index.css                    # 全部样式 (~900行)
├── js/
│   ├── i18n.js                         # 国际化 (fr/en)
│   ├── core-renderer.js                # WebGL 渲染引擎 (bundled, 1行)
│   ├── hero-project.js                 # WebGL 项目数据 (4层着色器 JSON)
│   ├── vendor/gsap.min.js
│   ├── vendor/ScrollTrigger.min.js
│   ├── vendor/lenis.min.js
│   └── index.js?v=20260423-r14         # 主逻辑 2411 行
├── assets/
│   ├── fonts/ (Breton.woff2, Machine.otf, Zirena.woff2)
│   ├── images/hero sequence/ (0001.jpg ~ 0341.jpg)
│   ├── images/projects/
│   ├── images/footer/ (left.png, right.png → ASCII art 源图)
│   └── images/shader background/ (background.png)
```

---

## 动画系统一览

| # | 文件 | 行号 | 效果 | 技术 |
|---|---|---|---|---|
| 1 | index.js | 75-89 | 名字逐字拆分 | DOM TreeWalker |
| 2 | index.js | 137-230 | Preloader 主时间线 (stagger + 缩放 + 幕布) | GSAP Timeline |
| 3 | index.js | 55-58 | CoreRenderer WebGL 启动 (4层着色器) | `startShader()` |
| 4 | hero-project.js | — | GLSL 300 es: Perlin 流场 + 高斯模糊 + 视差 | WebGL |
| 5 | index.js | 262-400 | 341 帧序列滚动映射 + Canvas 绘制 | Canvas 2D |
| 6 | index.js | 419-460 | 文字逐个词 blur→clear 渐显 | ScrollTrigger scrub |
| 7 | index.js | 919-980 | 项目卡片 3D Tilt 跟随鼠标 (lerp 阻尼 0.12) | gsap.ticker |
| 8 | index.js | 995-1012 | 流体曲线 SVG stroke-dashoffset | ScrollTrigger |
| 9 | index.js | 1015-1204 | Circle Gallery 圆柱切片成像 | CSS 3D + ScrollTrigger |
| 10 | index.js | 1207-1311 | 侧边章节导航 + 滚动百分比 | ScrollTrigger |
| 11 | index.js | 1314-1343 | Skills 手风琴 (height 动画) | GSAP |
| 12 | index.js | 1372-1488 | Contact Blob + 内容平行飞入 | GSAP Timeline |
| 13 | index.js | 1491-1868 | ASCII Art footer + 交互悬停 | Canvas→ASCII→DOM |
| 14 | index.js | 1969-2128 | Project 详情 Flying Title 过渡 | GSAP Timeline |
| 15 | index.js | 2269-2356 | 页面过渡系统 (overlay + fly text) | GSAP + sessionStorage |
| 16 | index.js | 2358-2411 | Awards 区域 + 自定义 cursor | ScrollTrigger toggleClass |

---

## 文件索引

| 文件 | 内容 |
|---|---|
| `01-loader-intro.md` | Preloader + 逐字弹入 + 红黑幕布 + 滚动锁定 |
| `02-hero-canvas.md` | WebGL CoreRenderer 4层着色器 + 帧序列 Canvas |
| `03-scroll-effects.md` | 文字词级渐显 + Project 3D tilt + Circle Gallery + 流体线 |
| `04-contact-blob.md` | 300vmax Blob + 内容飞入 + ASCII Art footer |
| `05-tech-stack.md` | 真实技术栈 + 源码结构 + 性能策略 |
