# Hero 区域 — Sticky Section + Rive + Split-Text + Touch Lock

landonorris.com 的 Hero 是一个多组件复合粘性区域，**不包含 Rive 自行车动画或 canvas 背景层**。

---

## 一、DOM 结构

```html
<div data-sticky-hero="track" class="sticky-track home-hero">
  <div class="sticky-item home-hero">
    <section data-hero-animation-container class="s home-hero">
      <div class="c home-hero">
        <!-- Canvas 占位（GSAP gl 用，非 Rive） -->
        <div data-gl="head" data-sticky-hero="canvas" class="gl-canvas"></div>

        <!-- Next Race 卡片 -->
        <div class="home-hero-next-race-w">
          <div class="home-hero-next-race-top">
            <div data-anim-high="right, lime" split-text="lines" class="text-eyebrow">
              Next Race
            </div>
          </div>
          <!-- 电路图 Rive Canvas (Webflow CMS 动态渲染) -->
          <div class="w-dyn-list">
            <div class="w-dyn-items">
              <div class="w-dyn-item">
                <canvas data-rive-object
                        data-rive-file="circuits"
                        data-rive-artboard="circuits"
                        data-rive-state-machine="circuits"
                        data-rive-input-track="monaco"
                        data-rive-input-color
                        data-rive-input-weight
                        data-rive-fit="contain">
                </canvas>
              </div>
            </div>
          </div>
          <!-- 头盔 Rive Canvas -->
          <canvas data-rive-object
                  data-rive-file="reef"
                  data-rive-artboard="helmet-reef"
                  data-rive-state-machine="helmet-reef_play"
                  data-rive-instant-play
                  data-rive-fit="contain">
          </canvas>
        </div>

        <!-- 触摸锁控制 -->
        <div data-home-swipe-wrap>
          <button data-home-swipe-toggle>Back to scroll</button>
        </div>

        <!-- "Message from Lando" 标题 -->
        <h1 split-text="chars" data-hero-anim="msg">Message from Lando</h1>

        <!-- 图片 / 移动端变体 -->
        <img data-hero-anim="img" src="..." />
        <span data-hero-anim="mob1">...</span>
        <span data-hero-anim="mob2">...</span>
      </div>
    </section>

    <!-- Signature Marquee -->
    <section class="s home-marquee">
      <canvas data-rive-object
              data-rive-file="signature"
              data-rive-artboard="signature"
              data-rive-state-machine="signature_scroll"
              data-rive-input="color_lime"
              data-rive-fit="contain"
              data-rive-scrolltrigger-target=".hero-rive-tracker"
              data-rive-scrolltrigger-start="top center"
              data-rive-scrolltrigger-end="bottom 80%">
      </canvas>
    </section>
  </div>
</div>
```

---

## 二、粘性布局机制

```css
.sticky-track.home-hero {
  /* 创建滚动空间，粘性容器在此空间内固定 */
}

.sticky-item.home-hero {
  position: sticky;
  top: 0;
  height: 100vh;
  overflow: hidden;
}
```

`data-sticky-hero="track"` 标记外层滚动轨道，`data-sticky-hero="canvas"` 标记内部 canvas 容器。非 ScrollTrigger `pin`，而是 CSS `position: sticky`。

---

## 三、组件详解

### 1. Next Race Card — 电路图 Rive

```html
<canvas data-rive-object
        data-rive-file="circuits"
        data-rive-artboard="circuits"
        data-rive-state-machine="circuits"
        data-rive-input-track="monaco"
        data-rive-input-color
        data-rive-input-weight>
</canvas>
```

- 电路内容来自 Webflow CMS (`w-dyn-items`, `w-dyn-item`)
- `data-rive-input-track` 值（如 `"monaco"`）选择 Rive 中对应的赛道动画
- `data-rive-circuit-hover="true"` 启用悬停交互 — 查找 `[data-rive-circuit-hover-target]` 元素，hover 时动态切换 `data-rive-input-track`

### 2. 头盔 Rive (`helmet-reef_play`)

```html
<canvas data-rive-object
        data-rive-file="reef"
        data-rive-artboard="helmet-reef"
        data-rive-state-machine="helmet-reef_play"
        data-rive-instant-play>
</canvas>
```

- `_play` 后缀 → `CH()` 函数处理（入场播放一次）
- `data-rive-instant-play` → 加载后立即播放，不等待滚动触发
- `reef` 文件包含 `helmet-reef` 和 `off-icons` 两个 artboard

### 3. 触摸滑动锁

```html
<div data-home-swipe-wrap>
  <button data-home-swipe-toggle>Back to scroll</button>
</div>
```

- `data-home-swipe-wrap` 监听触摸事件
- `data-home-swipe-toggle` 点击切换两种状态：
  - **锁定** (`tap to lock`) → 页面无法向下滚动，停在 Hero
  - **解锁** (`Back to scroll`) → 恢复滚动
- 控制 Lenis 的 `stop()` / `start()`

### 4. "Message from Lando" — SplitText 字符揭示

```html
<h1 split-text="chars" data-hero-anim="msg">Message from Lando</h1>
```

- `split-text="chars"` → GSAP SplitText 3.13.0 将每个字符包裹为 `<span class="char">`
- `data-hero-anim="msg"` → `o0()` 函数检测到后触发 GSAP stagger 入场动画

---

## 四、Hero 初始化流程 (`o0()`)

```javascript
function o0() {
  // 1. 查找 [data-hero-animation-container]
  // 2. 等待 allriveloaded 事件
  // 3. 在容器内查找所有 [data-rive-object] canvas
  //    确保它们的 .riv 文件已加载到 DB
  // 4. 查找所有 [data-hero-anim] 元素
  //    - "msg"  → SplitText chars stagger reveal
  //    - "img"  → fade-in animation
  //    - "mob1"/"mob2" → mobile text variant animations
  // 5. 启动 Rive 实例播放
}
```

`o0()` 在 `[data-hero-animation-container]` 上检测，确保所有依赖的 Rive 文件已预加载完成，然后统一启动动画。

---

## 五、Signature 滚动揭示

```html
<canvas data-rive-object
        data-rive-file="signature"
        data-rive-artboard="signature"
        data-rive-state-machine="signature_scroll"
        data-rive-input="color_lime"
        data-rive-fit="contain"
        data-rive-scrolltrigger-target=".hero-rive-tracker"
        data-rive-scrolltrigger-start="top center"
        data-rive-scrolltrigger-end="bottom 80%">
</canvas>
```

- `_scroll` 后缀 → `oR()` 函数处理
- `data-rive-scrolltrigger-target` / `start` / `end` 定义 GSAP ScrollTrigger 边界
- `data-rive-input="color_lime"` 将 Rive `color_lime` 输入绑定到颜色变化
- 效果：用户滚动时 Lando 签名逐笔绘制

---

## 六、关键创新点

| 特性 | 实现方式 |
|------|---------|
| 粘性布局 | CSS `position: sticky`，非 ScrollTrigger pin |
| 赛道选择 | `data-rive-input-track` + Webflow CMS 动态渲染 |
| 头盔动画 | `_play` 状态机 + `data-rive-instant-play` 立即播放 |
| 触摸锁定 | `data-home-swipe-wrap/toggle` 控制 Lenis stop/start |
| 字符揭示 | `split-text="chars"` + `data-hero-anim="msg"` 触发 GSAP SplitText |
| 签名绘制 | `_scroll` 状态机 + `oR()` 映射滚动进度到 Rive `scroll` input |
