# Scroll-Driven Storytelling — 滚动叙事设计模式

所有 5 个案例共同的底层模式：**用滚动代替点击，用进度代替翻页，让用户在纵向滑动中体验一个完整的叙事弧线。**

---

## 一、叙事弧线结构

```
                     ┌──── 高潮 ────┐
                     │              │
                     │              │
  ┌── 引入 ──┐       │              │       ┌── 结尾 ──┐
  │ preloader│       │  核心展示     │       │ CTA      │
  │ hero     │───────│  产品/品牌   │───────│ footer   │
  │ 建立期待  │       │  深度体验    │       │ 收束     │
  └──────────┘       └──────────────┘       └──────────┘
      ↑                      ↑                    ↑
   快速抓注意力          信息密度最高         低密度，行动召唤
    0-15%               15-75%              75-100%
```

### 5 个项目的叙事结构对比

| 项目 | 引入 (0-15%) | 核心 (15-75%) | 结尾 (75-100%) |
|------|-------------|--------------|---------------|
| **Zentry** | Portal 展开 + 标题逐字弹入 | Bento Grid + Token 区域 + Who We Are | Partners marquee + Footer CTA |
| **landonorris** | Hero sticky + 头盔动画 + 签名 | 水平画廊 + On/Off Track + 项目展示 | Footer SVG mask + 社交媒体 |
| **Oryzo** | 3D 杯垫旋转 + 标语 | Wearable Gallery + Feature 滑块 + Encryption | Product Selector + 伪论文 + Footer |
| **Luke Baffait** | Preloader + 名字逐字 + 红黑幕布 | WebGL 渐变 + 帧序列 + Circle Gallery | Contact Blob + ASCII footer |
| **Persepolis** | Preloader + 片头视频 | 6 房间探索 (固定路径) | 退出体验 |

### 设计原则

1. **前 3 秒决定去留** — Hero 区域必须"立即传达身份 + 制造好奇心 gap"。Zentry 用 Portal 展开 + "REDEFINE REALITY" 双关；Oryzo 用 3D 杯垫旋转 + 反直觉标语 "Made for mugs. Built for tables."

2. **每 3-5 屏一个节奏变化** — 用户不会读完所有内容。必须通过"视觉密度切换"重新抓住注意力：
   - 全屏 3D → 文字密集区 → 交互滑块 → 视频 → 网格展示
   - 节奏变化信号：背景色切换（landonorris 的 `data-gl-change`）、全屏/留白切换、动效速度变化

3. **结尾要有"行动感"** — Footer 不能只是信息。Zentry 的 "Build the foundation for tomorrow's civilization" + Oryzo 的 "If we can sell a coaster, imagine what we can do for your brand" — 都在给用户一个"带走的念头"。

---

## 二、滚动驱动的三种变体

### 变体 A：Scrub（精准同步）

ScrollTrigger `scrub: true`，滚动位置精确映射到动画进度。

**使用场景：** 需要"帧级"精确同步的效果
- Zentry Portal: clip-path 菱形展开，要求展开速度和手指滑动速度完全一致
- Luke Baffait 341 帧序列：每帧对应一个滚动位置
- Oryzo 3D 杯垫旋转：模型角度与滚动位置 1:1

**Design Rationale：** Scrub 创造"操控感"——用户感觉自己在直接操纵视觉元素，而不是触发一个预先设计好的动画。这种操控感显著增加停留时间（dwell time）。

**不适用的场景：** 文字密集区。用户快速浏览时 scrub 可能导致文字动画跟不上阅读速度 → 改用 ScrollTrigger `toggleActions` 或 IntersectionObserver。

### 变体 B：Play-on-Enter（触发播放）

滚动到特定位置时播放一次动画，播放完成后不再重复。

**使用场景：** 叙事性动画、需要控制节奏的展示
- landonorris 的 `_play` Rive 动画：头盔动画一次性播放
- Oryzo 的 Social Grid：每个网格元素 `clip-path` 依次展开

**Design Rationale：** 适合"讲一个完整的小故事"。用户滚动到这个区域，动画从头到尾讲完一个动作。Play-on-enter 有明确的"开始→结束"弧线，符合人类对"故事"的认知模式。

**调参要点：**
- `once: true`（GSAP）确保只触发一次，避免回滚时混乱
- 延迟触发（`start: "top 80%"`）让元素在进入视口前就开始"准备"，等用户看到时已经在运动中

### 变体 C：Toggle（状态切换）

滚动经过某个阈值时切换元素的 class/状态，保持在切换后的状态。

**使用场景：** UI 主题切换、导航栏显示/隐藏
- landonorris：`data-nav-theme="dark"` / `"light"` 随滚动切换
- Zentry：HUD 导航在滚动方向变化时显示/隐藏

**Design Rationale：** 适合"持久状态改变"。用户不需要看到完整的过渡过程，只需要"发生了变化"这个事实。Toggle 的优点是性能开销极低（只触发一次 class 切换）。

---

## 三、滚动叙事的设计检查清单

| 检查项 | 为什么重要 | 参考案例 |
|--------|----------|---------|
| Hero 3 秒内建立 identity | 用户决定是否留下 | Zentry "REDEFINE REALITY" |
| 每 2-3 屏有节奏变化 | 防止视觉疲劳 | landonorris 背景色切换 |
| 滚动方向有功能含义 | 向下=前进，向上=后退（直觉） | 全案例一致 |
| Progress indicator | 让用户知道"还有多少" | landonorris 滚动条, Zentry HUD |
| 关键动画有 scrub | 制造"控制感" | Oryzo 3D 模型, Luke 帧序列 |
| 内容区的动画用 play-on-enter | 让用户读完再触发 | 所有 SplitText 效果 |
| 至少一次"花哨时刻" | 创造记忆点/分享动机 | Zentry Portal, Luke Blob |

---

## 四、应用指南

### 适合 Scroll-Driven 叙事的场景

| 场景 | 推荐叙事结构 | 参考模式 |
|------|------------|---------|
| 品牌官网 (Brand Site) | Play-on-enter + Toggle，少用 Scrub | Zentry |
| 产品发布页 (Product Launch) | Scrub 为主，3D 产品模型驱动 | Oryzo |
| 个人作品集 (Portfolio) | Play-on-enter，文字动画为主 | Luke Baffait |
| 运动员/艺人网站 | Play-on-enter + Rive 矢量动画 | landonorris |
| 虚拟导览/教育 | 固定路径 + Scrub 或点击触发 | Persepolis |
| E-commerce 产品页 | 轻量 ScrollTrigger，避免全屏锁定 | — |

### 不适合 Scroll-Driven 叙事的场景

- 工具型页面（dashboard, admin panel, 文档站）
- 需要精准定位的页面（搜索结果、地图）
- 用户高频重复访问的页面（收件箱、日历）
