# Web Design Repository — 优秀案例拆解

6 个顶尖前端动效网站的深度技术拆解 + 可复用设计模式库 + 审美训练系统。

## 案例

| 项目 | 团队 | 核心技术 | 类型 |
|------|------|---------|------|
| [Zentry](cases/zentry/) | Resn | CSS clip-path + Nuxt 3 + GSAP | Web3 品牌站 |
| [landonorris.com](cases/landonorris/) | OFF+BRAND | Rive + Webflow + GSAP | F1 运动员站 |
| [Oryzo AI](cases/oryzo/) | Lusion | Three.js + Astro + Rive | 产品营销站 |
| [Luke Baffait](cases/lukebaffait/) | Lusion | 自定义 WebGL + 纯 JS | 个人作品集 |
| [Persepolis](cases/persepolis/) | Media.Monks | Unity + 自定义 WebGL | 虚拟博物馆 |
| [Active Theory](cases/activetheory/) | Active Theory | Hydra (自研 WebGL) + WebSocket + AI | 工作室作品集 |

## 结构

```
cases/        ← 6 个案例拆解
├── landonorris/
├── zentry/
├── oryzo/
├── lukebaffait/
├── persepolis/
└── activetheory/

patterns/     ← 跨项目设计模式
├── scroll-storytelling
├── clip-path-effects
├── text-animation
├── 3d-strategy
├── hover-feedback
└── timing-easing

snippets/     ← 16 个可直接运行的 HTML 片段

references/   ← 索引 + 框架指南
├── EFFECT_INDEX.md
└── FRAMEWORK_GUIDE.md

taste/        ← 审美规则库（通过你的反馈积累）
└── TASTE_PROFILE.md
```

## 工作流

1. **我分析** → 读案例 + 模式，提出设计方案
2. **你评价** → 看效果，一句"好看"或"不对"
3. **我迭代** → 你的反馈写入 `taste/TASTE_PROFILE.md`
4. **进化** → 每次反馈让我下次更懂你的审美

你只负责：**看一眼，说感觉。** 我负责剩下的全部。
