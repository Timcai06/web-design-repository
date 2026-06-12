# Preloader + 名字动画 + 红黑幕布

源码: `index.js:1-260`

---

## 一、实际 Render Tree

从 `index.html` 提取：

```html
<!-- index.html:50-56 -->
<div class="intro-bg" id="intro-bg"></div>                  <!-- fixed, z-index:10000 -->
<div class="name-layer" id="name-layer">                      <!-- fixed, z-index:10005 -->
  <div class="preloader-content" id="preloader-content">
    <div id="preloader-logo">L</div>                          <!-- "L" (不是 ">") -->
    <span id="preloader-luke">uke</span>
    <span id="preloader-baffait"> Baffait</span>
    <span id="preloader-dot">.</span>
  </div>
</div>
<div class="transition-panel" id="transition-panel">          <!-- fixed, z-index:10002 -->
  <div class="t-panel-dark" id="t-panel-dark"></div>           <!-- #0a0a0a -->
  <div class="t-panel-red" id="t-panel-red"></div>             <!-- #ff1e00 -->
</div>
```

名字三部分的 HTML 是分开的（不是拼接的），`preloader-logo="L"`、`preloader-luke="uke"`、`preloader-baffait=" Baffait"`（注意前面有空格）、`preloader-dot="."`.

---

## 二、字符拆分函数

`index.js:75-89`：

```js
function splitIntoChars(el) {
  const raw = el.textContent;
  el.innerHTML = '';
  const inners = [];
  raw.split('').forEach(ch => {
    const outer = document.createElement('span');
    outer.style.cssText = 'display:inline-block;overflow:hidden;vertical-align:top;padding:0.15em 0.3em;margin:-0.15em -0.3em;';
    const inner = document.createElement('span');
    inner.className = 'char';
    inner.style.display = 'inline-block';
    inner.textContent = ch === ' ' ? '\u00a0' : ch;
    outer.appendChild(inner);
    el.appendChild(outer);
    inners.push(inner);
  });
  return inners;
}
```

**结构**：每个字符被两层 `<span>` 包裹——外层 `overflow:hidden` 做裁剪容器，内层 `char` 做动画目标。`padding:0.15em 0.3em` + 负 `margin` 保证字间距不变。

---

## 三、名字布局算法

`index.js:91-115`:

```js
function layoutNames() {
  const fs = parseFloat(getComputedStyle(pBaffait).fontSize);
  if (!fs) return;
  const baselineOffset = -0.06;
  const lukeLeft = pLuke.offsetLeft;                   // "uke" 在 "L" 后面的位置
  const lukeWidth = pLuke.offsetWidth;
  const gapPx = fs * 0.55;                             // "L" 和 "uke" 之间的间隙
  const baffaitLeftPx = lukeLeft + lukeWidth + gapPx;  // " Baffait" 跟在 "uke" 后面
  pBaffait.style.left = (baffaitLeftPx / fs) + 'em';
  pBaffait.style.top = baselineOffset + 'em';
  const dotLeftPx = baffaitLeftPx + pBaffait.offsetWidth;
  pDot.style.left = (dotLeftPx / fs) + 'em';
  pDot.style.top = baselineOffset + 'em';
}
```

**"L" 是 `#preloader-logo`（独立元素），"uke" 是 `#preloader-luke`，" Baffait" 是 `#preloader-baffait`。三块各自绝对定位，用 `offsetLeft/Width` 计算位置。**

---

## 四、主时间线 (master)

`index.js:137-230`:

```js
const master = gsap.timeline({ delay: 0.2 });
```

### 阶段 1: 字符弹入 (0s - 0.4s)
```js
master
  .add(() => {
    layoutNames();
    gsap.set(pContent, { x: -(getTotalWidth() / 2 - pLogo.offsetWidth / 2) });
  })
  .to(allRevealEls, {
    yPercent: 0,                              // 从 110% 到 0
    duration: 0.4,
    ease: 'power3.out',
    stagger: { each: 0.025, from: 'center' }, // 从中间向两侧
  })
```
所有字符初始 `yPercent: 110`（下方隐藏），以 25ms 间隔从中心向两侧弹出。

### 阶段 2: 句号 + 启动 WebGL (0.4s - 0.65s)
```js
  .add(() => layoutNames())
  .to(pDot, { opacity: 1, duration: 0.25 })
  .add(() => {
    startShader();
    // 设置 will-change 优化后续动画
  })
  .to({}, { duration: 0.3 })  // 空隙
```

### 阶段 3: 名字缩小到左下角 (0.95s - 1.7s)
```js
  .add(() => {
    const mobile = isMobileViewport();
    const pad = mobile ? 20 : 48;
    const currentW = getTotalWidth();
    const targetW = viewportSize.width - pad * 2;
    const scale = targetW / currentW;          // 缩小比例
    // ... 计算 deltaY 移到左下角位置 ...
    gsap.to(pContent, {
      scale: scale,
      y: `+=${deltaY}`,
      duration: 0.75,
      ease: 'power3.inOut',
      onComplete: () => {
        // 切换为 vw 单位字体 + mix-blend-mode: difference
        gsap.set(nameLayer, { mixBlendMode: 'difference' });
        placeIntroNameAtBottom();              // 锚定在底部
        keepIntroNameAnchored = true;          // 开启 resize 监听
      },
    });
  })
```
**计算逻辑**：名字缩小到 `viewportWidth - 96px(两边padding)` 宽度，然后移动到 `vh - 80px` 的底部位置。完成后从 px 单位切换到 `vw` 单位，设置 `mix-blend-mode: difference`。

### 阶段 4: 幕布落下 + 升起 (1.7s - 2.3s)
```js
  .to(tPanelDark, { y: '0%', duration: 0.45, ease: 'power3.inOut' }, '<+=0.05')
  .to(tPanelRed,  { y: '0%', duration: 0.45, ease: 'power3.inOut' }, '-=0.3')
  .set(introBg, { display: 'none' })
  .set(hero, { opacity: 1 })
  .to(tPanelRed,  { y: '-100%', duration: 0.55, ease: 'power3.inOut' }, '+=0.05')
  .to(tPanelDark, { y: '-100%', duration: 0.55, ease: 'power3.inOut' }, '-=0.4')
```
**时序**：暗色层先落下 (y:0%) → 红色层叠在暗色上 (-0.3s 负偏移产生重叠) → 隐藏 introBg，显示 hero → 红色先升起 (y:-100%) → 暗色跟随。

### 阶段 5: Hero 内容揭示 (2.1s - 2.8s)
```js
  .to('#hero-tagline', {
    opacity: 1,
    clipPath: 'inset(0 0 0% 0)',              // 从底部向上擦除
    duration: 1.1,
    ease: 'power3.inOut',
  }, '-=0.2')
  .to('#hero-bar', {
    opacity: 1,
    clipPath: 'inset(0 0 0% 0)',
    duration: 1.0,
    ease: 'power3.inOut',
  }, '-=0.8')
  .fromTo('#hero-line',
    { opacity: 1, scaleX: 0 },
    { scaleX: 1, duration: 1.0, ease: 'power3.inOut' },
    '<'
  )
```
`clipPath: 'inset(0 0 100% 0)' → 'inset(0 0 0% 0)'` 实现从下往上擦除效果。`'-=0.2'` 表示在幕布完全升起前就开始揭示。

---

## 五、Character Hover 效果

`index.js:232-250`:

```js
document.querySelectorAll('.chr-hover[data-chr]').forEach(el => {
  const text = el.dataset.chr;
  [...text].forEach((ch, i) => {
    const wrap = document.createElement('span');
    wrap.className = 'ch-wrap';
    wrap.style.setProperty('--i', i);
    const top = document.createElement('span');
    top.className = 'ch-top';
    top.innerHTML = window.getCharHTML(ch);      // 特殊字符→SVG 图标
    const bot = document.createElement('span');
    bot.className = 'ch-bot';
    bot.innerHTML = window.getCharHTML(ch);
    wrap.appendChild(top);
    wrap.appendChild(bot);
    el.appendChild(wrap);
  });
});

// 入场动画
const chrHoverTl = gsap.timeline({ paused: true });
document.querySelectorAll('.chr-hover').forEach((el, elIdx) => {
  el.querySelectorAll('.ch-top').forEach((ch, i) => {
    const pos = elIdx * 0.08 + i * 0.03;
    chrHoverTl.fromTo(ch,
      { clipPath: 'inset(100% 0 0 0)' },
      { clipPath: 'inset(0 0 0 0)', duration: 0.7, ease: 'power3.out' },
      pos
    );
  });
});
```

**字符特殊处理**（来自 `i18n.js:9-17`）：
- `🡲` / `🡺` → 箭头 SVG
- `🡼` → 旋转箭头 SVG (rotate -135deg)
- `🞣` → 菱形 SVG

**CSS 交互**（`styles/index.css: .chr-hover`）：
```css
.chr-hover:hover .ch-top { transform: translateY(-100%); }
.chr-hover:hover .ch-bot { transform: translateY(-100%); }
.ch-top, .ch-bot { transition: transform 0.6s cubic-bezier(0.87, 0, 0.13, 1); }
```
Hover 时字符向上滑动，露出底部副本。`--i` 自定义属性控制 `transition-delay: calc(var(--i) * 28ms)`，每个字符依次延迟。

---

## 六、滚动锁定

`index.js:34-49`:

```js
function _forceScrollTop() { window.scrollTo(0, 0); }
function _preventTouchScroll(e) { e.preventDefault(); }

if (!mustSkip) {
  _forceScrollTop();
  window.addEventListener('scroll', _forceScrollTop);
  if (isMobile) {
    document.addEventListener('touchmove', _preventTouchScroll, { passive: false });
  } else {
    document.documentElement.style.overflow = 'hidden';
  }
}
```

在时间线最后解锁（`index.js:260-271`）：

```js
master.add(() => {
  window.removeEventListener('scroll', _forceScrollTop);
  document.documentElement.style.overflow = '';
  lenis.start();
  lenis.scrollTo(0, { immediate: true });
  // 移除面板 DOM
  const tPanel = document.getElementById('transition-panel');
  if (tPanel) tPanel.remove();
  const iBg = document.getElementById('intro-bg');
  if (iBg) iBg.remove();
  // 启动 ScrollReveal
  requestAnimationFrame(() => setupScrollReveal());
});
```

---

## 七、skip 系统

`index.js:4-8`:

```js
const shouldSkipLongIntro = !!sessionStorage.getItem('index-return-fade');
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
const mustSkip = shouldSkipLongIntro || prefersReducedMotion;
```

如果是页面回退（通过 `sessionStorage` 标记）或用户设置了 `prefers-reduced-motion`，整个时间线直接跳到 `master.progress(1)`，跳过所有 preloader 动画。
