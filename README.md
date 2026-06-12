# Web Design Repository — 优秀案例拆解

5 个顶尖前端动效网站的深度技术拆解 + 可复用设计模式库。

## 案例

| 项目 | 团队 | 核心技术 | 类型 |
|------|------|---------|------|
| [Zentry](zentry/) | Resn | CSS clip-path + Nuxt 3 + GSAP | Web3 品牌站 |
| [landonorris.com](landonorris/) | OFF+BRAND | Rive + Webflow + GSAP | F1 运动员站 |
| [Oryzo AI](oryzo/) | Lusion | Three.js + Astro + Rive | 产品营销站 |
| [Luke Baffait](lukebaffait/) | Lusion | 自定义 WebGL + 纯 JS | 个人作品集 |
| [Persepolis](persepolis/) | Media.Monks | Unity + 自定义 WebGL | 虚拟博物馆 |
| [Active Theory](activetheory/) | Active Theory | Hydra (自研 WebGL) + WebSocket + AI | 工作室作品集 |

## 结构

```
00-design-patterns/     ← 跨项目设计模式（推荐从这里开始）
├── scroll-storytelling
├── clip-path-effects
├── text-animation
├── 3d-strategy
├── hover-feedback
└── timing-easing

每个案例/
├── index.md             ← 项目总览 + 技术栈
├── 01-*.md ~ 05-*.md   ← 分模块技术拆解
└── application-guide.md ← 设计分析 + 应用指南
```

## 用法

1. **Agent / 开发者**：先读 `00-design-patterns/` 选模式，再到对应案例看完整实现
2. **设计师**：读每个案例的 `application-guide.md` 的 Design Rationale 部分
3. **快速参考**：`00-design-patterns/README.md` 有模式索引

## 设计哲学

这个仓库不只是"代码怎么写的"，而是：
- **为什么这样设计好看** — 感知心理学分析
- **适合放哪里** — 场景匹配指南
- **怎么改成自己的** — 通用改编方案 + 调参
