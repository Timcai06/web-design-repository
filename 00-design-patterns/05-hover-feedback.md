# Hover & Feedback — 交互反馈设计模式

好的反馈让用户感觉"这个页面是活的"。所有 5 个案例在 hover/交互设计上都有精心设计的反馈系统。

---

## 一、反馈层级

```
层级 1: 光标 (cursor)   — 最低成本，最连续
层级 2: 元素本身变化     — 最直观，增加可点击感
层级 3: 周围元素响应     — "生态反应"，增加精致感
层级 4: 页面级响应      — 沉浸感，适合叙事站点
```

| 层级 | 例子 | 成本 |
|------|------|------|
| 光标 | Luke 的自定义 cursor | Low |
| 元素自身 | landonorris helmet clip-path hover | Low |
| 周围响应 | landonorris circuit map 悬停 + 颜色联动 | Medium |
| 页面级 | Zentry perspective tilt 跟随鼠标 | Medium |

---

## 二、五种可复用的 Hover 模式

### 模式 1：Clip-Path Reveal Hover

```
初始 → clip-path: ellipse(0% 0% at 50% 0%)
hover → clip-path: ellipse(100% 120% at 50% 0%)
transition: 0.75s cubic-bezier(0.65, 0.05, 0, 1)
```

**案例：** landonorris 头盔网格

**为什么好：** 不是简单的 opacity 变化，而是"形状揭示"。让用户感觉"图片是从遮罩后面被拉出来的"。

**调参：**
- E-commerce 产品图：0.4s，更快响应购物意图
- 品牌站：0.75-1s，更从容优雅
- 文艺/摄影站：1.2s+，近乎不动声色的渐变

### 模式 2：Rive Vector Hover

```
<canvas data-rive-hover="false">
mouseenter → data-rive-hover="true" → Rive input triggers morph
```

**案例：** landonorris button arrows

**为什么好：** Rive 动画可以做出矢量变形（morphing）、颜色过渡、路径动画——比 CSS transition 丰富得多。箭头从"→"变成"↗"的流畅变形是 CSS 做不到的。

**成本：** 需要 Rive 设计师配合，无法纯前端完成。

### 模式 3：3D Perspective Tilt Hover (Zentry)

```
mouse move → 计算鼠标在元素上的相对位置
→ transform: perspective(800px) rotateX(±5deg) rotateY(±8deg)
→ 用 lerp/damp 平滑跟随
```

**案例：** Zentry Bento Grid cards

**为什么好：** 物理模拟。卡片跟随鼠标产生 3D 倾斜，模拟"用指尖拨动真实卡片"的感觉。阻尼（lerp）让运动滞后于鼠标毫秒级延迟，模拟物理惯性——这是"润"（juicy）的来源。

**实现要点：**
```javascript
// useDamp — 滞后数值
const dampX = ref(0)
const dampY = ref(0)

// 每帧 lerp 靠近目标
useRaf(() => {
  dampX.value += (targetX.value - dampX.value) * 0.12
  dampY.value += (targetY.value - dampY.value) * 0.12
})
```
`0.12` = lerp factor。越大跟踪越快（0.2 = 灵敏），越小越滞后（0.05 = 粘滞感）。

### 模式 4：Video Stream on Hover

```
鼠标悬停 → video.play()
鼠标离开 → video.pause() + 回到 placeholder image
```

**案例：** landonorris social media cards, Oryzo wearable gallery

**为什么好：** 视频默认隐藏（节省带宽 + 避免干扰），但在用户感兴趣的瞬间立刻播放。这是"渐进式加载"在交互层面的应用——内容在需要时才被激活。

**调参：**
- 视频必须从静音状态开始（避免突兀的声音）
- 最好预加载前几帧，否则 hover 后出现黑屏等待
- 视频长度建议 <15s，循环播放

### 模式 5：SVG/Canvas ASCII Interactive (Luke Baffait)

```
鼠标在 ASCII art 上移动 → 计算光斑位置
→ 重新渲染周边区域的 ASCII 字符（亮度/字符密度变化）
```

**案例：** Luke Baffait footer ASCII art

**为什么好：** ASCII art 本身是"复古"视觉语言（80 年代计算机美学）。给 ASCII art 添加鼠标交互创造了一种"活在 80 年代但拥有现代交互"的反差美感。它也是谈话起点（"你见过这种 footer 吗？"）。

---

## 三、反馈设计决策矩阵

| 业务场景 | 推荐反馈模式 | 优先级 |
|---------|------------|--------|
| **E-commerce 产品卡** | Clip-path reveal hover 0.4s + 放大 1.05x | 必须：产品图清晰可见 |
| **SaaS Dashboard 卡片** | Perspective tilt hover 5° + 阴影加深 | 必须：可点击信号 |
| **品牌官网导航** | 下划线动画 + 当前页高亮 | 必须：导航清晰 |
| **作品集 thumbnail** | Clip-path 或 scale hover + 标题出现 | 推荐：增加互动感 |
| **游戏/娱乐站** | Rive morph hover + 粒子/声效 | 可选：超出预期 |
| **阅读类网站** | 最小反馈（颜色变化即可） | 尽量避免干扰 |

---

## 四、常见错误

| 错误 | 为什么不好 | 修正 |
|------|-----------|------|
| hover 动画 >1s | 用户觉得"迟钝" | 交互反馈 <0.4s，展示动画可用 0.6-0.8s |
| 忽略 touch 设备 | 移动端没有 hover | touch 设备用 click 或 press 代替 |
| 所有元素 hover 一样 | 没有层次感 | CTA > 卡片 > 装饰元素，渐弱反馈强度 |
| hover 时才加载资源 | 首次 hover 卡顿 | 图片预加载，视频 preload="metadata" |
