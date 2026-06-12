# Oryzo AI — Technical Case Study

**URL:** https://oryzo.ai  
**Studio:** [Lusion](https://lusion.co/) (Bristol)  
**Type:** Satirical product launch campaign  
**Date Analyzed:** 2026-05-29

---

## 站点概览

Oryzo AI 是 Lusion 的自我发起项目：**把一个普通的软木杯垫包装成 AI 时代的产品发布**。全站是一个完整的营销 campaign（官网 + GitHub Paper/Model + Product Hunt + 创始人视频 + 社交媒体），用顶级的 3D 视觉和叙事手法来卖一个不存在的东西。

### 核心体验流程

```
Preloader (WebGL 动画)
  │
  进入首页 →
  │
  ├─ Hero: 全屏 Three.js 3D 杯垫模型
  │   + 滚动驱动模型旋转/缩放
  │   + 标语 "Made for mugs. Built for tables."
  │   + 视频播放按钮 (Vimeo Overlay)
  │
  ├─ AI Section: 幽默产品宣言
  │   + "Oryzo-1" 模型引用
  │   + Powered by AI* (*Adobe Illustrator)
  │
  ├─ Wearable Gallery: 水平滚动
  │   + 7 个场景 (intro/yoga/shoulder/bikini/glasses/bite/pocket/outro)
  │   + 自动播放 + 拖拽
  │   + 大图预览跟随
  │
  ├─ Features:
  │   ├─ Elevate: 3D 动画 + 文案
  │   ├─ Temperature: 交互滑块 + Three.js 曲线可视化
  │   └─ Curve: Three.js 曲线 canvas
  │
  ├─ Encryption: 翻转编码/解码 DEMO
  ├─ Grip: 滚动驱动放大 + 摩擦系数展示
  ├─ Sustainability: Rive 交互动画 (收获 + 文字)
  ├─ Testimonials: 用户评价 + 产品对比表
  ├─ Social Content Grid: 滚动触发的网格展示
  ├─ Product Selector: 三档产品选择 (Oryzo/Pro/Pro Max)
  ├─ Open-Weight: 仿论文页面 (GitHub Paper/Model/Code)
  └─ Footer: 邮件订阅 + 社交媒体链接
```

### 技术栈

| 技术 | 用途 |
|---|---|
| **Astro** | 静态站点生成器，构建管线 |
| **Three.js** | 主 3D 引擎（Hero 杯垫模型、Wearable、曲线） |
| **Rive** | Sustainability 区域交互矢量动画 |
| **GSAP + ScrollTrigger** | Scroll-driven 动画（社交网格 `data-offset`、视差） |
| **Lenis** | 平滑滚动 + 滚动进度指示器 |
| **Vimeo Player API** | 视频 overlay（产品视频） |
| **Cloudflare Analytics** | 站点分析 |

### 创意亮点

- **全站 Satire Tone** — 严肃科技产品发布语气 + 荒谬产品（杯垫），反差幽默贯穿始终
- **"AI 产品"包装** — 有 Paper/Model Checkpoints/Citation/BibTeX，GitHub 仓库认真到令人发笑
- **Lusion 的自证** — Footer CTA: "If we can sell a coaster, imagine what we can do for your brand."

---

## 文件索引

| 文件 | 内容 |
|---|---|
| `01-hero-3d.md` | 全屏 Three.js 杯垫模型、滚动驱动的 3D 交互、Preloader |
| `02-wearable-gallery.md` | 水平滚动/拖拽画廊、Thumbnail 导航 |
| `03-scroll-effects.md` | Social Content Grid、Features 滑块与 3D 曲线、Grip 放大、Encryption 翻转 |
| `04-rive-sustainability.md` | Rive 交互式矢量动画（收获 + 文字） |
| `05-tech-stack.md` | Astro 构建、Three.js 管线、Rive + GSAP 集成细节 |
