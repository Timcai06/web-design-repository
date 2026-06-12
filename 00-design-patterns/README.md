# Design Pattern Library — 设计模式库

这是一个跨项目的设计模式集合，从 5 个顶尖案例中提炼出可复用的 UI/动效设计模式。

---

## 使用方式

### 如果你是 Agent

1. 接到"改善某个页面的 UI 效果"的任务
2. 先通读 `01-scroll-storytelling.md` — 理解用户的浏览叙事框架
3. 根据具体需求定位到对应模式：
   - 需要文字动效？→ `03-text-animation.md`
   - 需要 hover 交互？→ `05-hover-feedback.md`
   - 需要 3D 效果？→ `04-3d-strategy.md` + `02-clip-path-effects.md`
   - 需要调 timing？→ `06-timing-easing.md`
4. 回到具体项目的 `*-application-guide.md` 看该效果的适用场景和调参

### 如果你是设计师/开发者

1. 先看 `01-scroll-storytelling.md` — 规划页面叙事结构
2. 选择需要的效果类型 → 参考对应模式文件
3. 选好模式后 → 到具体案例看完整实现代码
4. 用 `06-timing-easing.md` 调好节奏

---

## 模式索引

| 文件 | 解决的问题 | 核心概念 |
|------|----------|---------|
| **01-scroll-storytelling.md** | 如何组织长页面的叙事节奏 | Scrub / Play-on-enter / Toggle |
| **02-clip-path-effects.md** | 低成本 3D 视觉效果的实现 | Portal / Ellipse / Inset / SVG Mask |
| **03-text-animation.md** | 从字符到段落的动效层次 | Char / Word / Line / Block |
| **04-3d-strategy.md** | 三种 3D 路线的决策逻辑 | CSS 3D / Three.js / Custom WebGL |
| **05-hover-feedback.md** | 有层次感的交互反馈系统 | 5 种可复用 hover 模式 |
| **06-timing-easing.md** | 节奏和缓动的感知心理学 | Easing 语言 / Duration 法则 / Stagger |

---

## 项目对照表

| 案例 | 滚动叙事 | Clip-Path | 文字动画 | 3D 路线 | Hover | Timing |
|------|---------|-----------|---------|--------|-------|--------|
| **Zentry** | Play-on-enter + Scrub | Portal Mask | Char split + clip wipe | CSS-only | Perspective tilt | power4.inOut, 戏剧 |
| **landonorris** | Scrub + Toggle | Ellipse helmet / SVG Mask | Char split + Line clip | Rive (canvas) | Clip reveal + Rive morph | cubic(0.65,0.05,0,1), 从容 |
| **Oryzo** | Scrub + Play-on-enter | Inset grid | Y + Opacity | Three.js | Video hover + tilt | power3.out, 活泼 |
| **Luke Baffait** | Play-on-enter + Scrub | — | Word blur + Char bounce | Custom WebGL | ASCII interactive + 3D tilt | power4.out, 直接 |
| **Persepolis** | Click/Progress-bar | — | — | Unity → WebGL | Click reveal | power2.inOut, 庄重 |
