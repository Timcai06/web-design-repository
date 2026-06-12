# 技术栈 + Project Detail + 页面过渡 + Awards

源码: `index.js:1870-2411`

---

## 一、Project Detail — Flying Title 过渡

### 开启动画

`index.js:1969-2060`:

```js
function openProject(id, clickedItem) {
  const proj = PROJECTS[id];
  if (!proj || projectOpen) return;
  projectOpen = true;

  // 1. 记录来源元素位置
  const rect = clickedItem.getBoundingClientRect();
  const cs = getComputedStyle(clickedItem);
  flyingTitle.textContent = clickedItem.textContent;
  flyingTitle.style.fontSize = startFontSize + 'px';
  gsap.set(flyingTitle, { left: rect.left, top: rect.top, opacity: 1 });

  // 2. 隐藏原元素
  clickedItem.style.visibility = 'hidden';

  // 3. 填充详情数据
  detailTitle.textContent = clickedItem.textContent;
  detailYear.textContent = proj.year;
  detailDesc.textContent = proj.desc;
  detailTags.innerHTML = proj.tags.map(t => '<span class="detail-tag">' + t + '</span>').join('');

  // 4. 构建图库
  var allImages = [clickedItem.dataset.img].concat(proj.images);
  detailThumbsInner.innerHTML = allImages.map(src => '<img src="' + src + '">').join('');
  detailSelected.innerHTML = '<img src="' + allImages[0] + '">';

  // 5. 主时间线
  var tl = gsap.timeline();
  tl.to(pageFade, { opacity: 1, duration: 0.8 }, 0);                    // 页面淡黑
  tl.to(flyingTitle, {                                                    // 标题飞到左上
    top: targetTop, left: targetLeft,
    fontSize: targetFontSize,                                             // clamp(3rem, 5vw, 5rem)
    duration: 1, ease: 'power3.inOut',
  }, 0.3);
  tl.to(detailEl, { opacity: 1, duration: 0.4 }, 1.0);                   // 详情面板淡入
  tl.set(flyingTitle, { opacity: 0 }, 1.1);                               // 隐藏飞入标题
  tl.set(detailTitleWrap, { opacity: 1 }, 1.1);                           // 显示正式标题
  tl.to(detailDesc, { opacity: 1, duration: 0.6 }, 1.2);                 // 描述淡入
  tl.to(detailTags, { opacity: 1, duration: 0.5 }, 1.3);                 // 标签淡入
  tl.fromTo(detailGalleryWrap, { opacity: 0 }, { opacity: 1, duration: 0.8 }, 1.2);
}
```

### 图库交互

`index.js:1942-1967`:

```js
function updateActiveThumb() {
  // 找到最靠近视口中心的缩略图
  var cy = thumbsRect.top + thumbsRect.height / 2;
  _thumbImgs.forEach((img, i) => {
    var dist = Math.abs(imgCy - cy);
    // 靠近中心的放大，远离的缩小
    var t = Math.max(0, 1 - dist / (thumbsRect.height * 0.45));
    img.style.width = (100 + t * 40) + '%';     // 100% ~ 140%
  });
  // 最中心的设为 active → 大图切换
  if (closestIdx !== _activeThumbIdx) {
    _thumbImgs[closestIdx].classList.add('active');
    _cachedSelImg.src = _thumbImgs[closestIdx].src;
  }
  _galleryRAF = requestAnimationFrame(updateActiveThumb);   // 持续循环
}
```

**连续 RAF 循环**：在 `openProject` 期间不停执行，不是事件驱动的。缩略图列表用 `gsap.quickTo` 做平滑滚动。

### 关闭动画

`index.js:2062-2128`:

```js
function closeProject() {
  // 逆操作：detail 内容淡出 → flying title 飞回原位置 → pageFade 淡出 → 恢复
  tl.to([detailDesc, detailTags, detailBack], { opacity: 0 }, 0);
  tl.to(detailGalleryWrap, { opacity: 0 }, 0);
  tl.to(detailEl, { opacity: 0 }, 0.2);
  tl.to(flyingTitle, {                                                  // 飞回原位
    left: itemRect.left, top: itemRect.top,
    fontSize: parseFloat(itemCs.fontSize),
    duration: 0.9, ease: 'power3.inOut',
  }, 0.3);
  tl.to(pageFade, { opacity: 0, duration: 0.6 }, 0.5);                 // 黑幕消失
  tl.add(function () {
    detailEl.classList.remove('active');
    _flyingSourceItem.style.visibility = '';                            // 恢复原元素
    lenis.start();
    ScrollTrigger.refresh();
  });
}
```

**关闭动画 = 开启动画的逆向重播**，相同的 flying title 从目标位置飞回源位置。

---

## 二、页面过渡系统

`index.js:2269-2356`:

```js
function runPageTransition(linkEl, label, sessionKey, href) {
  const overlay = document.getElementById('work-transition-overlay');
  const flyText = document.getElementById('work-flying-text');

  const rect = linkEl.getBoundingClientRect();
  flyText.textContent = label;                               // "Work" / "Info" / "Contact"

  gsap.set(overlay, { opacity: 0 });
  gsap.set(flyText, {
    left: rect.left, top: rect.top,                          // 从链接位置出发
    fontSize: getComputedStyle(linkEl).fontSize,
    opacity: 1,
  });

  const tl = gsap.timeline();
  tl.to(overlay, { opacity: 1, duration: 0.7, ease: 'power2.inOut' }, 0);    // 黑幕渐出
  tl.to(flyText, {                                                            // 文字飞到左上角
    left: '3rem', top: '3rem',
    fontSize: 'clamp(2.5rem, 5vw, 4.5rem)',
    duration: 1, ease: 'power3.inOut',
  }, 0.2);
  tl.add(() => {
    sessionStorage.setItem(sessionKey, '1');    // 记录过渡标记
    window.location.href = href;                // 导航
  }, 1.3);
}
```

**路由映射**：
```js
const PAGE_LINK_ROUTES = {
  work:    { label: 'Work',    sessionKey: 'work-transition',    href: 'works/' },
  info:    { label: 'Info',    sessionKey: 'info-transition',    href: 'info/' },
  contact: { label: 'Contact', sessionKey: 'contact-transition', href: 'contact/' },
};
```

**恢复处理**（`index.js:2234-2258`）：
```js
function handleIndexPageShow(e) {
  var hasReturnFlag = !!sessionStorage.getItem('index-return-fade');
  if (hasReturnFlag) sessionStorage.removeItem('index-return-fade');  // 消费标记
  // 从过渡恢复 → pageFade 从 opacity:1 → 0 （淡入揭示页面）
  if (hasReturnFlag || isBfcacheRestore) {
    playIndexReturnFadeIn();
  }
}
```

**流程**：
```
当前页面 → 点击 Work 链接
  → 拦截导航 (e.preventDefault)
  → overlay 从 0→1 (黑幕)
  → "Work" 文字从链接位置飞到左上角并放大
  → 1.3s 后 window.location.href = 'works/'
  → works/ 页面加载
  → 检测 sessionStorage 标记
  → pageshow 事件触发 → playIndexReturnFadeIn()
  → pageFade opacity:1→0 (页面淡入揭示)
```

---

## 三、Awards 区域

`index.js:2358-2411`:

```js
document.addEventListener("DOMContentLoaded", () => {
  const awardItems = gsap.utils.toArray('.award-item');

  // 1. ScrollTrigger 高亮最靠近视口中心的奖项行
  awardItems.forEach(item => {
    ScrollTrigger.create({
      trigger: item,
      start: "top center+=15%",               // 元素顶部到达视口中心偏下 15%
      end: "bottom center-=15%",              // 元素底部离开视口中心偏上 15%
      toggleClass: { targets: item, className: "active-award" }
    });
  });

  // 2. 自定义 cursor：hover 时显示项目封面
  const awardCursor = document.createElement('img');
  gsap.set(awardCursor, { xPercent: -50, yPercent: -50, scale: 0.8, opacity: 0 });

  awardItems.forEach(item => {
    item.addEventListener('mouseenter', (e) => {
      isAwardHovered = true;
      const imgSrc = item.getAttribute('data-cursor-img');
      if (imgSrc) awardCursor.src = imgSrc;         // 动态切换图片
      gsap.to(awardCursor, { opacity: 1, scale: 1, duration: 0.3 });
    });
    item.addEventListener('mouseleave', () => {
      isAwardHovered = false;
      gsap.to(awardCursor, { opacity: 0, scale: 0.8, duration: 0.3 });
    });
  });
});
```

**CSS 高亮**（`styles/index.css`）：
```css
.active-award { color: #000000 !important; }
.active-award::before {
  clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);  /* 白色背景从左到右展开 */
}
```

**交互组合**：`toggleClass` + `::before` 伪元素配合 `clip-path` 过渡 + 自定义图片 cursor。

---

## 四、项目数据

`index.js:1872-1915`:

```js
const PROJECTS = {
  'cyberdiag': {
    desc: "Showcase website for the CyberDiag app...",
    category: 'Website', year: '2026',
    tags: ['Gsap', 'Lenis', 'Three.js'],
    images: ['assets/images/projects/CyberDiagWebsite/image1.png', ...],
  },
  'anima':       { /* 动物保护网站 */ },
  'cyberdiag-app': { /* 安全诊断桌面应用 */ },
  'zenith':      { /* Electron 浏览器 */ },
  'skymcdb':     { /* Minecraft 工具桌面应用 */ },
  'chromablock': { /* SkymcDB 的 Web 版 */ },
  'symphony':    { /* 音乐流媒体 Web 应用 */ },
  'echo':        { /* 本地 AI 对话界面 */ },
};
```

---

## 五、真实技术栈验证

### 从源码确认

| 技术 | 源码证据 | 文件 | 行 |
|---|---|---|---|
| **无框架** | 纯 `index.html`，无 `import`/`export`，无 bundler | index.html | 入口 |
| **GSAP 3+** | `gsap.registerPlugin(ScrollTrigger)` | index.js | 55 |
| **ScrollTrigger** | `gsap.registerPlugin(ScrollTrigger)` + 30+ 个 `.create()` | index.js | 全局 |
| **Lenis** | `new Lenis({ lerp: 0.06 })` | index.js | 252 |
| **无 Three.js** | Hero 背景用了自建 WebGL `CoreRenderer`，不含 three.js 依赖 | core-renderer.js | — |
| **GLSL 300 es** | 所有着色器 `#version 300 es` | hero-project.js | — |
| **无 Primer CSS** | 无 `primer.css` 引用。实际字体来自 `@font-face` | styles/index.css | 6-22 |
| **自定义字体** | `Breton` (woff2)、`Machine` (otf)、`Zirena` (woff2) | styles/index.css | 6-22 |
| **Canvas 2D** | `ctx.drawImage()` 绘制帧序列 | index.js | 305-314 |
| **`mix-blend-mode`** | `difference` 模式使名字在任何背景色上都可见 | index.js | 195 |
| **`backdrop-filter`** | `blur(16px)` 在滚动过渡阶段 | index.js | 391-394 |

### 字体栈对比

**知识库推测**（错误）：
```
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans", ...
```
这是 GitHub Primer CSS 默认字体栈。

**实际源码**（styles/index.css）：
```css
body { font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif; }
```

### 性能策略

| 策略 | 源码 |
|---|---|
| Canvas DPR 限制 | `dpr = isSlowHardware ? 1 : Math.min(dpr, 1.5)` |
| 帧率节流 | `if (now - _lastDrawMs < 32) return` (~30fps) |
| 硬件检测 | `isSlowHardware = isMobile \|\| navigator.hardwareConcurrency <= 4` |
| 帧跳过 | 慢速设备采样率 `skip = 3` |
| 后台加载 | `requestIdleCallback(loadBatch)` |
| CSS will-change | 动画元素显式设置 |
| 移动端 blur 关闭 | `if (isMobile) words.forEach(w => w.style.filter = 'none')` |
