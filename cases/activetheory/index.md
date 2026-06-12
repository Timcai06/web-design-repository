# Active Theory (ATv6) — Technical Case Study

**URL:** https://activetheory.net
**Studio:** Active Theory (Venice Beach, LA + Amsterdam)
**Type:** Studio portfolio + brand showcase
**Date Analyzed:** 2026-06-12

---

## 为什么这个案例重要

Active Theory 是 WebGL 创意领域的顶级工作室之一。与 Lusion、Resn、Media.Monks、OFF+BRAND 并列。但 ATv6 的独特之处在于它**不是为客户做的，而是为自己做的**——这给了它完全的技术自由。

结果是：这个网站本身就是它要展示的作品之一。

### 与现有 5 个案例的定位差异

| 维度 | Active Theory | Lusion | Resn | OFF+BRAND |
|------|-------------|--------|------|-----------|
| **自研引擎** | ✅ Hydra 3D Engine | ❌ Three.js | ❌ 纯 CSS | ❌ Rive |
| **多用户实时** | ✅ 联网光标 | ❌ | ❌ | ❌ |
| **AI 交互** | ✅ 聊天导航 | ❌ | ❌ | ❌ |
| **团队规模** | ~34 人 | ~8 人 | ~15 人 | ~5 人 |
| **成立时间** | 2012 | 2015 | 2000 | 2015 |
| **代表作** | Google I/O, Sundance | 炫技产品站 | 品牌站 | 运动员站 |

---

## 体验流程

```
进入首页
  │
  ├─ Preloader (WebGL 加载进度)
  │
  ├─ Hero — 3D 沉浸环境
  │   ├─ 霓虹灯光 + 外星风格字体
  │   ├─ 彩色粒子管跟随鼠标 (Multi-user: 可见其他访客)
  │   ├─ 背景 3D 场景滚动推进
  │   └─ Pillbox 导航 (响应滚动速度)
  │
  ├─ 项目展示 (Portfolio)
  │   ├─ AI Chat 导航: "show me something fun"
  │   ├─ 分类筛选: Games / Multiplayer / XR-VR-AI / Installations / Websites
  │   └─ 项目卡片 3D 过渡
  │
  ├─ 关于 / 团队
  │   ├─ 3D 环境切换 (办公室主题)
  │   └─ 联系方式
  │
  └─ Footer
```

---

## 技术栈

| 技术 | 用途 |
|------|------|
| **Hydra 3D Engine** | 自研 WebGL 渲染引擎（非 Three.js） |
| **Hydra GUI** | 可视化 3D 场景编辑器（设计师不用写代码） |
| **WebGL 2.0** | 底层渲染 API（fallback 到 WebGL 1.0） |
| **GLSL** | 自定义着色器 |
| **WebSocket** | 多用户实时光标同步 |
| **AI Chat API** | LLM 驱动的项目推荐导航 |
| **ActiveFrame** | 自研 WebCodecs 视频格式（逐帧控制） |
| **Draco Compression** | 3D 网格压缩 |
| **Google Cloud / Firebase** | 后端 + 实时数据库 |
| **Node.js** | 后端运行时 |
| **Sanity.io / Contentful** | Headless CMS |

**值得注意的是：** 没有 React/Vue/Astro——核心是纯 WebGL，UI 层由 Hydra GUI 渲染。这是与 Zentry（Nuxt）和 Oryzo（Astro）的根本区别。

---

## 核心特性一览

| # | 特性 | 技术 | 独特性 |
|---|------|------|--------|
| 1 | **Hydra 3D Engine** | 自研 WebGL 框架 | 比 Three.js 精简 3x，专为特定渲染场景优化 |
| 2 | **多用户联网光标** | WebSocket + WebGL | 看到其他访客的鼠标轨迹和粒子管 |
| 3 | **AI Chat 导航** | LLM + 语义搜索 | "show me a fun project" — 自然语言筛选作品 |
| 4 | **3D 沉浸环境** | WebGL + GLSL | 场景基于 LA/Amsterdam 办公室，霓虹灯光风格 |
| 5 | **彩色粒子管** | WebGL 粒子系统 | 鼠标/触摸产生彩色管状轨迹 |
| 6 | **ActiveFrame 视频** | WebCodecs API | 自定义逐帧视频播放（不用 `<video>` 标签） |
| 7 | **Pillbox 导航** | CSS + JS | 小型胶囊导航，响应滚动速度 |
| 8 | **Draco 压缩** | 3D 网格压缩 | LCP 1.3s 即使有大量 3D 几何 |
| 9 | **WebWorker 并行** | 多线程 | 几何加载、粒子生成、物理碰撞在 Worker 中 |

---

## 与行业对比

### 自研引擎的决策逻辑

Active Theory 在 2019 年决定从 Three.js 迁移到自研 Hydra 引擎。原因：

1. **性能** — 自定义渲染管线可以精确控制 GPU 调用，Three.js 为了通用性做了大量抽象。自研引擎在已知渲染场景下比 Three.js + Electron 快 3x。
2. **工具链** — Hydra GUI 让设计师可以不写代码构建 3D 场景，Three.js 没有等效的工具。
3. **跨平台** — Hydra 跑在 Web、iOS（原生）、Android（原生）、Desktop（Node + OpenGL）、VR——Three.js 只覆盖 Web。

**这意味着什么：** 对于大多数项目，Three.js 仍然是最合理的选择。自研引擎只在达到一定规模（团队 >15 人、项目类型高度重复）时才有 ROI。

### 五个案例 + Active Theory 的技术光谱

```
纯 CSS (Zentry) ── Three.js (Oryzo) ── Rive (landonorris) ── 自研 WebGL (Luke) ── 自研引擎 + GUI (AT)
    轻量                                                 重量
    品牌站                                               沉浸式体验
    低成本                                               高投入
```

---

## 文件索引

| 文件 | 内容 |
|------|------|
| `01-core-tech.md` | Hydra 引擎架构、WebGL 管线、性能优化、ActiveFrame |
| `02-multiplayer-ai.md` | 多用户联网光标、AI Chat 导航、WebSocket 架构 |
| `03-scroll-effects.md` | 3D 环境滚动推进、粒子管系统、霓虹灯光风格 |
| `04-navigation-ux.md` | Pillbox 导航、项目筛选、Portfolio UX 设计 |
| `05-tech-stack.md` | 完整工具链、从 Three.js 迁移到 Hydra 的决策 |
| `application-guide.md` | 设计语言分析 + 每个效果的应用指南 |
