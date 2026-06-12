# Text Animation Patterns — 文字动效设计模式

5 个案例全部使用文字动画。文字动画是"成本最低、回报最高"的 UI 增强手段。

---

## 一、拆解层级

文字动效从微观到宏观有三个层级：

```
字符级 (char)
  └── 单词级 (word)
        └── 行级 (line)
              └── 段落级 (block)
```

| 粒度 | 视觉节奏 | 阅读干扰 | 适合 |
|------|---------|---------|------|
| **Char** | 快、密集、花哨 | 高 | Hero 标题、品牌 tagline（短文本） |
| **Word** | 中、温和 | 中 | 正文标题、section 标题 |
| **Line** | 慢、沉稳 | 低 | 正文、段落 |
| **Block** | 最慢、大气 | 最低 | 全文出现、大面积文本 |

---

## 二、5 种核心文字动效

### 效果 1：逐字弹入 (Zentry Hero / Luke Baffait name)

```
初始: opacity: 0; y: 80; rotateX: 40
最终: opacity: 1; y: 0; rotateX: 0
stagger: 0.04-0.06s / char
```

**为什么好看：** 每个字符从下方弹起，模拟"文字从地面长出"或"水面上浮"。`rotateX: 40` 增加 3D 纵深感——字符从"躺下"到"立起"。

**应用指南：**
- stagger 0.03-0.04s = 活泼/快速/街头感（适合运动/潮牌）
- stagger 0.05-0.07s = 优雅/从容/正式（适合奢侈品/品牌）
- stagger >0.1s = 太慢，用户会觉得卡顿
- `y` 偏移量与字体大小成正比：大字（hero）用 60-100px，小字用 20-40px

**不适合：** 超过 5 个词的句子（用户等不及看完）。

### 效果 2：逐词 Blur → Clear (Luke Baffait)

```
初始: opacity: 0; filter: blur(8px)
最终: opacity: 1; filter: blur(0px)
```

**为什么好看：** Blur → Clear 模拟"视觉聚焦"——文字从模糊到清晰，就像眼睛在重新对焦。这利用了人类的"凹视觉感知"——模糊的东西自动被认为是背景，清晰的东西是焦点。文字从 blur 到 clear 的过程，就是"从背景中浮现"的感知过程。

**应用指南：**
- blur 值: hero 大字 8-12px，正文 4-6px
- 配合 `y: 10-20` 增加动势，但不必须（blur 本身就有"靠近"感）
- 适合正文段落、section 说明文字
- stagger 按 word 而非 char，速度 0.1-0.2s

**性能：** `filter: blur()` 是性能大户。对 GPU 压力 `blur(4px)` ≈ 10x opacity。大量文字 blur 动画时使用 `will-change: filter, opacity` 并限制 DOM 节点数量。

### 效果 3：Clip-Path 擦除 (Zentry / landonorris line)

```
clip-path: polygon(0 0, 0 0, 0 100%, 0 100%) 
        → polygon(0 0, 100% 0, 100% 100%, 0 100%)
```

**为什么好看：** 文字像被"擦除"或"揭开"——模拟遮罩从左侧移开。这利用了阅读方向（左到右）的惯性——遮罩跟随眼睛的移动方向。

**变体：**
```css
/* 从左到右（阅读方向，最自然） */
polygon(0 0, 0 0, 0 100%, 0 100%) → polygon(0 0, 100% 0, 100% 100%, 0 100%)

/* 中间向两边（对称感，庄重） */
polygon(50% 0, 50% 0, 50% 100%, 50% 100%) → polygon(0 0, 100% 0, 100% 100%, 0 100%)

/* 从下往上（升起感） */
polygon(0 100%, 100% 100%, 100% 100%, 0 100%) → polygon(0 0, 100% 0, 100% 100%, 0 100%)
```

**应用指南：**
- 文字擦除适合需要"线性阅读"的文本——用户的眼睛本来就沿着这一方向移动
- 不适合短标题（太短的文字看不出擦拭过程）

### 效果 4：Split Line + Y Offset + Opacity (最常见)

```
.line { clip-path: polygon(0 -2%, 0 94%, 100% 94%, 100% -2%) }
.line { opacity: 0; transform: translateY(40px) }
.line.in-view { opacity: 1; transform: translateY(0) }
```

**为什么好看：** 组合了"升起"（Y）和"揭开"（clip-path）两个信号。Y 偏移让文字有"从下方浮出"的空间感，clip-path 增加一帧精度的切割感。

**应用指南：** 这是最"安全"的文字动画模式，适合 80% 的场景。它是"有动效但不抢戏"的通用解。

### 效果 5：Staggered Line 波浪 (Oryzo / landonorris)

```
每行 stagger: 0.1-0.2s
每行起点 offset: 0-30px（Y 方向）
```

**为什么好看：** 多行文字不是同时出现，而是依次推进，形成"波浪"。模拟"打字"或"印刷"的感觉。

---

## 三、文字动画的 Timing 圣经

| 元素类型 | 动画时长 | 延迟 (stagger) | Easing |
|---------|---------|---------------|--------|
| **Hero 标题** | 0.6-0.8s | 0.03-0.06s/char | `power4.out` |
| **Section 标题** | 0.5-0.7s | 0.1-0.15s/word | `power3.out` |
| **正文** | 0.4-0.6s | 0.08-0.12s/line | `power2.out` |
| **数字/计数器** | 1-2s | — | `power1.out` 或 `expo.out` |
| **小标签/badge** | 0.3-0.4s | — | `back.out(1.7)` |

**核心规则：** 总动画时长不超过 2s。用户不会看完超过 2s 的文字动画。

---

## 四、文字动画的 4 个常见错误

| 错误 | 为什么不好 | 修正 |
|------|-----------|------|
| 长句子用 char split | 用户读完最后一个字时已经忘了前面 | 改用 word 或 line 级别 |
| 正文用过度花哨的动效 | 干扰阅读，用户会跳过去 | 只用 Y + Opacity 或 clip-path |
| stagger 间隔不一致 | 感觉卡顿、不流畅 | 用 GSAP 的 `stagger: { each: 0.05 }` |
| 所有文字用同一种动画 | 缺乏层次和重点 | Hero 用 char，正文用 line，关键数字用 counter |

---

## 五、3D 质感文字技巧 (Zentry)

```css
.hero__title-line {
  perspective: 800px;
}
.char {
  display: inline-block;
  transform-style: preserve-3d;
  /* rotateX 配合 perspective 创造"字符立起来"的效果 */
}
```

**为什么好看：** 文字本身是 2D 的。给文字添加 `perspective` 和 `rotateX`，让 2D 文字短暂地"拥有厚度"——视觉系统会脑补出"这些字母是立体的物体"。这种错觉只在动画过程中存在，但足以留下深刻印象。
