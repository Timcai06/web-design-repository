# 滚动供应商标志效果 — Marquee 技术实现

## 效果描述

landonorris.com 多处横向无限滚动标志条：
- **首页 `.home-marquee`**：合作伙伴/赞助商 logo 匀速滚动
- **Footer**：底部品牌信息无限循环（含 SVG 元素）

---

## 一、CSS Keyframe 驱动循环

来源：`css-home` embed

```css
@keyframes translateXLeft {
  to { transform: translateX(-100%); }
}
@keyframes translateXRight {
  to { transform: translateX(100%); }
}

[data-css-marquee-list="left"] {
  animation: translateXLeft 30s linear;
  animation-iteration-count: infinite;
  animation-play-state: paused;
}
[data-css-marquee-list="right"] {
  animation: translateXRight 30s linear;
  animation-iteration-count: infinite;
  animation-play-state: paused;
  justify-content: flex-end;
}
```

设计要点：`animation-play-state: paused` 默认暂停由 JS 控制播放；`linear` 匀速匹配 F1 调性；`translateX(-100%)` 移动正好一个内容宽度（配合 JS 克隆）。

---

## 二、JS 控制系统

### 自定义 data 属性

| 属性 | 值 | 作用 |
|---|---|---|
| `data-marquee-direction` | `left` / `right` | 滚动方向 |
| `data-marquee-speed` | `30` (秒) | 循环时长 |
| `data-marquee-duplicate` | `4` | 克隆份数 |
| `data-css-marquee-list` | `left` / `right` | 应用对应 CSS keyframe |
| `data-marquee-scroll-speed-target` | CSS selector | 速度绑定到 scroll 位置 |

### JS 克隆逻辑（从 `data-marquee-duplicate` 读取）

```javascript
function setupMarquee(el) {
  const list = el.querySelector('[data-css-marquee-list]')
  const count = parseInt(el.dataset.marqueeDuplicate) || 2
  for (let i = 0; i < count - 1; i++) {
    const clone = list.cloneNode(true)
    clone.setAttribute('data-marquee-clone', i + 1)
    el.appendChild(clone)
  }
}
```

克隆 n 份 → `translateX(-100%)` 移动一个完整宽度 → 内容一致形成无限循环。

---

## 三、桌面端渐变遮罩

仅在 **≥1920px** 生效，两端淡入淡出：

```css
@media (min-width: 1920px) {
  .marquee-offest-w {
    -webkit-mask-image: linear-gradient(90deg, hsla(0,0%,100%,0), white 7.5%, white 92.5%, hsla(0,0%,100%,0));
    mask-image: linear-gradient(90deg, hsla(0,0%,100%,0), white 7.5%, white 92.5%, hsla(0,0%,100%,0));
  }
}
```

左端 0–7.5% 渐隐，中间 7.5–92.5% 完全可见，右端 92.5–100% 渐隐。

---

## 四、综合流程

1. HTML 声明 `data-marquee-*` 属性
2. JS 读取 `data-marquee-duplicate` 克隆 `.marquee-list`
3. 设置 `animation-play-state: running` 启动动画
4. 如有 `data-marquee-scroll-speed-target`，监听 scroll 动态调整 `animation-duration`

完全声明式——通过 data 属性驱动无限滚动，适合纯视觉装饰性场景。
