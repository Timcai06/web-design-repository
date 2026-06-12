# Snippets — 可复用代码片段

独立 HTML 文件，浏览器直接打开即可查看效果。每个 snippet 包含：
- **实现代码** — 完整可运行
- **Design Rationale** — 为什么这样设计好看
- **Decision** — 什么时候选这个方案
- **调参指南** — 如何适配不同场景

## 完整列表

| # | 片段 | 来源 | 核心技术 | 分类 |
|---|------|------|---------|------|
| 1 | [Clip-Path Ellipse Hover](clip-path-ellipse-hover.html) | landonorris | CSS | Hover |
| 2 | [Perspective Tilt Card](perspective-tilt-card.html) | zentry | JS + CSS | Hover |
| 3 | [Word-Level Blur Reveal](word-blur-reveal.html) | lukebaffait | JS + CSS | Text |
| 4 | [Scroll Grip Zoom](scroll-grip-zoom.html) | oryzo | JS | Scroll |
| 5 | [Contact Blob](contact-blob.html) | lukebaffait | CSS | Interactive |
| 6 | [HUD Nav Direction](hud-nav-direction.html) | zentry | JS + CSS | Navigation |
| 7 | [Marquee Infinite Scroll](marquee-infinite-scroll.html) | landonorris / zentry | CSS / GSAP | Scroll |
| 8 | [Split Line Clip Reveal](split-line-clip-reveal.html) | landonorris | CSS + JS | Text |
| 9 | [Nav Theme Toggle](nav-theme-toggle.html) | landonorris | ScrollTrigger | Navigation |
| 10 | [Stats Counter](stats-counter.html) | zentry | GSAP | Interactive |
| 11 | [Comparison Slider](comparison-slider.html) | persepolis | JS + CSS | Interactive |
| 12 | [Staggered Grid Reveal](staggered-grid-reveal.html) | oryzo | CSS + JS | Gallery |
| 13 | [3D Flip Card](flip-card-3d.html) | oryzo | CSS | Interactive |
| 14 | [SVG Mask Footer](svg-mask-footer.html) | landonorris | CSS + SVG | Footer |
| 15 | [Audio Visualizer](audio-visualizer.html) | zentry | JS | Navigation |
| 16 | [Scroll Progress Indicator](scroll-progress-indicator.html) | landonorris / oryzo | JS | Navigation |

## 使用方式

```bash
# 在浏览器中直接打开
open snippets/clip-path-ellipse-hover.html
```

每个 HTML 是自包含的（通过 CDN 加载 GSAP 等库），不依赖本仓库其他文件。
