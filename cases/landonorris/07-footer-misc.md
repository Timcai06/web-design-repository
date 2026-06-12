# Footer SVG Mask / Nav Theme / Page Transitions / 杂项效果

---

## 一、Footer SVG Mask Clip

Footer 使用 CSS `mask-image` 实现有机轮廓。通过 CSS 自定义属性切换桌面/移动遮罩。

### HTML

```html
<footer data-footer-theme="white">
  <div class="footer-layout-clip">
    <div class="footer-content"></div>
  </div>
  <div class="footer-bg" />
</footer>
```

### CSS

```css
.footer-layout-clip {
  --mask-url: url('https://cdn.prod.website-files.com/.../ln4-footer-mask-desktop.svg');
  -webkit-mask-image: var(--mask-url);
  mask-image: var(--mask-url);
  mask-size: cover;
  mask-repeat: no-repeat;
  mask-position: center top;
}
@media only screen and (max-width: 479px) {
  .footer-layout-clip {
    --mask-url: url('.../ln4-footer-mask-mobile.svg');
  }
}
```

### 头盔 Mask

```css
[data-helmet-item] {
  --mask-url: url('.../ln4-2-helm-mask-fill.svg');
  -webkit-mask-image: var(--mask-url);
  mask-image: var(--mask-url);
  mask-size: cover;
}
@media (max-width: 479px) {
  [data-helmet-item] {
    --mask-url: url('.../ln4-helm-mob-refactor-lime-fill.svg');
  }
}
```

Footer theme variant: `data-footer-theme="white|black|green"`。

---

## 二、Scroll-Driven Nav Theme

导航栏滚动切换明/暗主题，使用 `data-nav-theme-target` + GSAP ScrollTrigger。

### HTML

```html
<div data-nav-theme-target="dark" class="hero-nav-theme is-1"></div>
<div data-nav-theme-target="light" class="hero-nav-theme is-2"></div>

<nav data-nav-wrap data-nav-theme="light">
  <a href="/" class="nav-logo">
    <canvas data-rive-ln4></canvas>
  </a>
  <div class="nav-middle">Links ...</div>
  <button data-nav-ham tabindex="0">
    <canvas data-rive-nav-hamburger
            data-rive-artboard="hamburger"
            data-rive-state-machine="hamburger"
            data-rive-fit="contain">
    </canvas>
  </button>
</nav>
```

### CSS

```css
[data-nav-wrap] {
  transition: color 0.4s var(--cubic-default);
}
[data-nav-wrap] .nav-middle {
  transition: color 0.4s var(--cubic-default);
}
[data-nav-wrap] .nav-ham-w {
  transition: border-color 0.4s var(--cubic-default);
}
[data-nav-wrap] svg path {
  transition: fill 0.4s var(--cubic-default);
}
```

### JS（基于 ScrollTrigger）

```javascript
ScrollTrigger.create({
  trigger: '[data-nav-theme-target="dark"]',
  start: 'top center',
  onEnter: () => nav.dataset.navTheme = 'dark',
  onLeaveBack: () => nav.dataset.navTheme = 'light',
})
```

---

## 三、LN4 Logo Rive Animation

```html
<canvas data-rive-ln4></canvas>
```

Custom JS `nR()` 初始化 Rive，使用 `logo` artboard / state machine：

| Input 名 | 类型 | 说明 |
|---|---|---|
| `logo-active` | boolean | 激活态 |
| `hover` | boolean | 悬停态 |
| `color_dark-green` | number | 深绿颜色 |
| `color_white` | number | 白色 |
| `color_lime` | number | 荧光绿 |

全局控制函数：`pK(bool)` 控制激活态，`AC(color)` 设置主题色。

---

## 四、Hamburger Menu Rive

```html
<button data-nav-ham tabindex="0">
  <canvas data-rive-nav-hamburger
          data-rive-artboard="hamburger"
          data-rive-state-machine="hamburger"
          data-rive-fit="contain">
  </canvas>
</button>
```

Rive 文件 `btn-ui`，由 `iR()` 函数初始化：

| Input 名 | 类型 | 说明 |
|---|---|---|
| `hover` | boolean | 悬停 |
| `close` | boolean | 关闭（菜单展开时 true） |
| `color-transparent` | number | 透明色 |
| `color-white` | number | 白色 |

- Nav theme light → `color-transparent`，dark → `color-white`
- 点击 `data-nav-ham` 切换 `close` boolean → 菜单开闭
- 支持键盘 Enter / Space，移动端 touch

---

## 五、Page Transitions — Rive Overlay

```html
<div class="transition-w">
  <canvas data-rive-primary
          data-rive-file="page-transition"
          data-rive-state-machine="page_out">
  </canvas>
</div>
```

使用独立 `transitions-rive-isolate.js`（HTML 中被注释掉）。全屏 Rive canvas 播放 page_out transition，完成后跳转页面。

---

## 六、Scroll Indicator

```html
<div class="scroll-indicator">
  <div class="scroll-indicator-bar"></div>
</div>
```

```css
.scroll-indicator { position: fixed; right: 0; top: 0; bottom: 0; width: 3px; z-index: 9999; pointer-events: none; }
.scroll-indicator-bar { width: 100%; height: 0%; background: currentColor; will-change: height; }
```

JS 根据 `scrollY / (totalH - viewportH)` 更新 `scroll-indicator-bar` 高度。

---

## 七、Mobile Rotation Prompt

```html
<div data-mob-landscape class="mob-landscape-block">
  <canvas data-rive-mob-landscape></canvas>
  <div class="text-title-reg-mona">Please rotate your device,<br/>This is a vertical drive.</div>
</div>
```

手机竖屏时全屏覆盖，提示用户横屏浏览。Rive canvas 播放旋转动画。

---

## 八、Nav Helmet Rive

```html
<canvas data-rive-nav-object
        data-rive-file="reef"
        data-rive-artboard="helmet-reef"
        data-rive-state-machine="helmet-reef_scroll"
        data-rive-color-input="color_green-off-white-2"
        data-rive-fit="contain">
</canvas>
```

导航菜单内的迷你头盔 Rive 动画，随滚动驱动。
