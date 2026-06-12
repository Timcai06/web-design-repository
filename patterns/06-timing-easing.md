# Timing & Easing — 节奏与缓动设计

效果的好坏 70% 取决于 timing 和 easing，而不是技术实现。

---

## 一、Easing 的感知语言

| Easing | cubic-bezier | 感知 | 适合 |
|--------|--------------|------|------|
| **power4.out** | `cubic-bezier(0.16, 1, 0.3, 1)` | 有力、自信、利落 | Hero 标题、大动作 |
| **power3.out** | `cubic-bezier(0.33, 1, 0.68, 1)` | 自然、舒适 | Section 标题、卡片 |
| **power2.out** | `cubic-bezier(0.33, 1, 0.68, 1)` | 温和、不抢眼 | 正文、小元素 |
| **expo.out** | `cubic-bezier(0.16, 1, 0.3, 1)` | 加速极快，果断 | 计数器、数字 |
| **back.out(1.7)** | `cubic-bezier(0.33, 1, 0.68, 1.7)` | 弹性、活泼、粘性 | Badge、小标签、呼出 |
| **cubic-default (Lando)** | `cubic-bezier(0.65, 0.05, 0, 1)` | 先慢后快再慢，从容 | 品牌级过渡 |
| **power4.inOut** | `cubic-bezier(0.76, 0, 0.24, 1)` | 戏剧性展开 | Zentry Portal |

**核心原则：** ease-out 是默认选择。ease-in-out 只在"进入+离开"需要对称时使用。ease-in 几乎永远不要用（除非模拟重力落下）。

---

## 二、所有案例的 Easing 选择分析

### Zentry Portal: `power4.inOut`

```
展开: 慢→快→慢
效果: 先是"抗拒地展开"，然后"用力推开"，最后"温柔停下"
```

**为什么选这个：** Portal 展开是"一个有过程的事件"——不是闪现，也不是简单滑动。`inOut` 让展开有"叙事弧线"：开始时的缓慢像是"门在确认是否要开"，中间的加速是"决意打开"，末端的减速是"让门精准停在完全打开的位置"。

如果用 `power4.out`（只有加速），Portal 感觉像是"被踹开的"——虽然有力，但缺少仪式感。

### landonorris 全站: `cubic-bezier(0.65, 0.05, 0, 1)`

这是一个**非对称的 ease-in-out**。前半段慢（感知为"准备"），后半段快（感知为"动作"）。

**为什么好：** 几乎所有品牌类的滚动揭示都想传达"从容而自信"。这个 easing 模拟了"手动执行一个精确动作"——先小心定位，再坚定完成。

对比：
- `power4.out`（Lusion 风格）→ 活泼、产品感
- `cubic-bezier(0.65, 0.05, 0, 1)`（OFF+BRAND 风格）→ 从容、品牌感

### Luke Baffait: `power4.out`

个人作品集追求"干脆利落"。power4.out 让动画"毫不犹豫地完成"——传达自信和技术能力。

### Oryzo: 混合使用

Lusion 根据效果选择 easing：
- 3D 模型旋转：linear（匀速旋转更"机械感"）
- 文字揭示：power3.out（温和）
- Grid 展开：power2.out（轻快）

---

## 三、Duration 设计的感知法则

| Duration | 感知 | 适合 |
|----------|------|------|
| **0.1-0.2s** | 瞬间、即刻 | hover 反馈、微交互 |
| **0.3-0.4s** | 快速、灵敏 | 按钮、面板切换 |
| **0.5-0.7s** | 自然、舒适 | 文字动画、卡片入场 |
| **0.8-1.2s** | 从容、庄重 | Hero 动画、大场景过渡 |
| **1.5-3s** | 仪式感、缓慢 | Preloader、片头、计数器 |

![Duration perception curve](https://...)

**100ms** 是"即时反馈"的临界值（低于 100ms 人类感觉不到延迟）。
**1s** 是"等待感"的临界值（超过 1s 用户开始不耐烦）。
**3s** 是注意力的临界值（超过 3s 用户可能离开页面）。

---

## 四、Stagger 设计

Stagger（延迟播放）是将一组元素以微小时间差依次播放的技术。它创造"波浪"或"连锁反应"感。

### Stagger 值速查

| 元素数量 | Stagger each | 总时长 (0.5s each + stagger) | 用途 |
|---------|-------------|---------------------------|------|
| 3-5 个 | 0.08-0.12s | ~1s | 导航链接、tagline 词 |
| 5-10 个 | 0.05-0.08s | ~1.2s | 网格卡片、列表 |
| 10-20 个 | 0.03-0.05s | ~1.5s | 字符拆分、图库 |
| >20 个 | 0.02-0.03s | ~2s | 大段字符动画 |

**经验法则：** 总动画时长（单个时长 + stagger 总延时）不应超过 2s。超过 2s 用户就开始"等"。

### GSAP Stagger 进阶用法

```javascript
// 标准线性 stagger
stagger: 0.05

// 从中间向两边 (Zentry 风格)
stagger: {
  each: 0.05,
  from: "center"
}

// 随机顺序
stagger: {
  each: 0.05,
  from: "random"
}

// 非线性（先快后慢）
stagger: {
  each: 0.05,
  ease: "power2.out"
}
```

**Design Rationale of "from: center"：** 从中间向两边展开，创造"对称爆炸"感。视觉上，中心是最"稳定"的起点，元素从中心向外扩散，看起来像是"一个中心事件引发了连锁反应"。相比之下，从头到尾的线性 stagger 更像是"排队出场"。

---

## 五、5 个案例的 timing signature

每个项目有自己独特的 timing 特征，构成了它的"节奏指纹"：

| 项目 | Hero 时长 | 文字 stagger | Easing signature | 节奏性格 |
|------|----------|-------------|-----------------|---------|
| **Zentry** | 1.2s | 0.04s/char | power4.inOut | 戏剧、仪式感、未来 |
| **landonorris** | 0.75s | 0.05s/char | cubic(0.65,0.05,0,1) | 优雅、从容、运动 |
| **Oryzo** | 1s | 0.06s/char | power3.out | 活泼、产品感、讽刺 |
| **Luke Baffait** | 0.8s | 0.04s/char | power4.out | 直接、自信、技术 |
| **Persepolis** | 2s+ | — | power2.inOut | 庄重、文化、沉稳 |

**关键洞察：** Niche / 品牌站用慢 timing（从容），Conversion / 产品站用快 timing（效率）。用户的"等待意愿"与内容价值成正比——博物馆体验等待 2s 是合理的，电商页等待 2s 用户就走了。
