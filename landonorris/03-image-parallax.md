# 水平滚动画廊 + 滚动揭示动画

---

## 一、水平滚动画廊 (Horizontal Scroll Gallery)

landonorris.com 首页使用 GSAP ScrollTrigger 实现水平滚动画廊。**没有逐帧图片序列**——实际效果是多张静态图片随页面滚动水平平移。

### HTML 结构

```html
<div data-gl-change-to="white, dark-green-tint-1-low"
     data-horizontal-section
     data-h-color-from="dark-green"
     data-h-color-to="white"
     data-gl-change-track
     data-gl-change-from="dark-green, dark-green-tint-1"
     class="s is-horizontal-track">
  <div class="horizontal-pin-wrap">
    <div class="horizontal-pin-spacer">
      <div class="horizontal-pin-sticky">
        <div class="horizontal-track">
          <div class="horizontal-grid-col">
            <div class="horizontal-item-w is-home-1">
              <img src="..." class="image is-horizontal-scroll" />
            </div>
          </div>
          <div class="horizontal-grid-col">
            <div class="horizontal-item-w is-home-2">
              <img src="..." class="image is-horizontal-scroll" />
            </div>
          </div>
          <!-- 更多 horizontal-item-w -->
        </div>
      </div>
    </div>
  </div>
</div>
```

### JS 实现

GSAP ScrollTrigger 查找 `[data-horizontal-section]` 容器，**固定 (pin)** 容器并水平平移 `.horizontal-track`：

```javascript
// 伪代码 — 通过 `data-horizontal-section` 选择器绑定
gsap.to('.horizontal-track', {
  xPercent: -100 * (sections - 1), // 根据项目数量平移百分比
  ease: 'none',
  scrollTrigger: {
    trigger: '[data-horizontal-section]',
    pin: true,
    scrub: 1,
    start: 'top top',
    end: () => '+=' + horizontalTrack.scrollWidth,
    invalidateOnRefresh: true,
  },
})
```

`data-h-color-from` / `data-h-color-to` 在滚动过程中联动触发全局颜色过渡。

---

## 二、"On Track / Off Track" 左右分屏

首页核心段落——左右分屏布局，各自独立 GSAP 文字揭示 + Rive Canvas 动画。

### HTML 结构

```html
<section class="s is-otot">
  <canvas data-rive-object data-rive-file="phrases"
          data-rive-artboard="phrase_on"
          data-rive-state-machine="phrase_on"
          data-rive-scrolltrigger="true"
          data-rive-scrolltrigger-start="top center">
  </canvas>

  <div class="otot-grid">
    <div class="otot-side is-left">
      <p class="otot-label" data-anim-high="left, dark-green-tint-1, 200">ON TRACK</p>
      <p class="otot-desc" data-anim-high="left, dark-green-tint-1, 200">... 描述文字 ...</p>
      <a class="btn is-otot" href="/on-track">Explore</a>
      <!-- 按钮内含 Rive Canvas 动画 -->
    </div>
    <div class="otot-side is-right">
      <p class="otot-label" data-anim-high="right, lime">OFF TRACK</p>
      <p class="otot-desc" data-anim-high="right, lime">... 描述文字 ...</p>
      <a class="btn is-otot" href="/off-track">Explore</a>
    </div>
  </div>
</section>
```

### GSAP 文字揭示

`data-anim-high` 定义方向 (`left`/`right`) 和颜色，GSAP 结合 ScrollTrigger 实现元素从侧边滑入 + 颜色高亮：

```javascript
// 伪代码 — 每个 data-anim-high 元素单独绑定
gsap.fromTo(el, {
  x: dir === 'left' ? -200 : 200,
  opacity: 0,
}, {
  x: 0,
  opacity: 1,
  duration: 1,
  ease: 'power3.out',
  scrollTrigger: {
    trigger: el,
    start: 'top 85%',
    toggleActions: 'play none none none',
  },
})
```

### Rive Canvas

页面使用 Rive 运行矢量动画（不是 Lottie），通过 `data-rive-*` 属性配置。`data-rive-scrolltrigger="true"` 让动画实例与 ScrollTrigger 联动，滚动进入视口时触发播放。

---

## 三、"On Track / Off Track" 底部全幅图

```html
<section data-otot-bottom class="s is-otot-end">
  <img src="...ln-home-helm-large.webp" alt="Lando lifting helmet" />
</section>
```

GSAP 实现 fade-in + scale-in 揭示：

```javascript
gsap.fromTo('[data-otot-bottom] img', {
  opacity: 0,
  scale: 0.85,
  y: 60,
}, {
  opacity: 1,
  scale: 1,
  y: 0,
  duration: 1.2,
  ease: 'power2.out',
  scrollTrigger: {
    trigger: '[data-otot-bottom]',
    start: 'top 75%',
    toggleActions: 'play none none none',
  },
})
```

---

## 四、Helmets 段落——Split-Text 逐行揭示

```html
<h2 class="helmets-heading" split-text="lines" data-anim-high="">HELMETS</h2>
```

`split-text="lines"` 将文本按行拆分，GSAP ScrollTrigger 逐行 animation（y 偏移 + opacity），无额外视差逻辑。

---

## 五、总结：实际效果 vs 假设

| 假设的效果 | 实际实现 |
|---|---|
| Canvas 逐帧序列 (73 frames) | **不存在** |
| 多层 translate 视差 | ScrollTrigger 文字/图片 reveal |
| CSS mask / clip-path 复杂擦除 | 不存在 |
| 实际效果 | 水平滚动画廊 + fade-in/slide-in 揭示动画 + Rive + SplitText |

视觉上的"深度感"来自：GSAP ScrollTrigger 驱动的 **opacity + y 平移揭示**，以及 Rive 矢量动画的叠加，而非传统的多层视差或帧序列。
