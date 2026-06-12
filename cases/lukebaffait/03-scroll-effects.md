# ScrollTrigger 文字揭示、项目 Sticky + 3D Tilt、Circle Gallery、流体 SVG

源码: `index.js:419-1311`

---

## 一、文字词级 blur 渐显

### 单词拆分

`index.js:420-438`:

```js
function wrapWords(el) {
  const walker = document.createTreeWalker(el, NodeFilter.SHOW_TEXT);
  const textNodes = [];
  while (walker.nextNode()) textNodes.push(walker.currentNode);
  textNodes.forEach(node => {
    const words = node.textContent.split(/(\s+)/);
    const frag = document.createDocumentFragment();
    words.forEach(w => {
      if (/^\s+$/.test(w)) {
        frag.appendChild(document.createTextNode(w));    // 保留空白
      } else if (w) {
        const span = document.createElement('span');
        span.className = 'word';
        span.textContent = w;
        frag.appendChild(span);
      }
    });
    node.parentNode.replaceChild(frag, node);
  });
}
```

**不 split 空格** — 正则 `/(\s+)/` 在 split 时保留空白符作为独立片段，用 `createTextNode` 插入，不会影响布局。

### ScrollTrigger 逐词驱动

`index.js:448-460`:

```js
[...aboutText.querySelectorAll('.word'), ...aboutVersion.querySelectorAll('.word')].forEach(word => {
  gsap.to(word, {
    opacity: 1,
    ...(isMobile ? {} : { filter: 'blur(0px)' }),      // 移动端不 blur
    ease: 'none',
    scrollTrigger: {
      trigger: word,            // 每个词作为自己的 trigger
      start: 'top 75%',         // 词顶部进入 75% 视口时开始
      end: 'top 60%',           // 到 60% 时完成
      scrub: true,              // 滚动位置驱动进度
    },
  });
});
```

**每个 `.word` 都有一个独立的 ScrollTrigger**（共约 30+ 个）。CSS 初始状态 `opacity: 0; filter: blur(8px)`，滚动经过时逐个变为 `opacity: 1; filter: blur(0px)`。

---

## 二、About 左图视差

`index.js:469-482`:

```js
function initPhotoScroll() {
  var tl = gsap.timeline({
    scrollTrigger: {
      trigger: photoWrap,
      start: 'top bottom',
      end: 'bottom top',
      scrub: true,
    },
  });
  tl.fromTo(photo, { y: '-50%' }, { y: '50%', ease: 'none' }, 0);
  tl.fromTo(photo, { opacity: 0, filter: 'blur(20px)' }, { opacity: 1, filter: 'blur(0px)', ease: 'none', duration: 0.3 }, 0);
}
```

照片从 `y: -50%` 到 `y: 50%`，在滚动范围内移动 100% 距离，产生**视差悬浮**效果。同时伴随 blur→clear 渐显。

---

## 三、Projects 区域

### 布局结构

`styles/index.css`:
```css
.projects-list {
  position: sticky; top: 0;
  height: 100vh; width: 45%;
  display: flex; flex-direction: column;
  justify-content: center;
}
.proj-preview {
  position: fixed; top: 0; right: 0;
  height: 100vh; width: 50%;
  perspective: 800px;
  opacity: 0; transition: opacity 0.4s;
}
```

左侧 sticky，右侧 fixed 卡片预览。

### 自动高亮最近项目

`index.js:557-589`:

```js
function onProjectsScroll() {
  if (!_projectsVisible) { deactivateAll(); return; }
  const cy = window.innerHeight / 2;
  let closestIdx = -1, closestDist = Infinity;
  items.forEach((item, i) => {
    const rect = item.getBoundingClientRect();
    const itemCy = rect.top + rect.height / 2;
    const dist = Math.abs(itemCy - cy);
    // 非活跃项目向右偏移（距离越远偏移越大，最大 80px）
    itemQuickX[i](Math.min(dist / halfH, 1) * 80);
    if (dist < closestDist) { closestDist = dist; closestIdx = i; }
  });
  if (closestDist < window.innerHeight * 0.45) {
    activateProject(closestIdx);
  }
}
lenis.on('scroll', onProjectsScroll);
```

**`gsap.quickTo`** — 在 ticker 中高效更新 x 位置。越远离视口中心的项目越靠右偏移。

### 3D Tilt 跟随鼠标

`index.js:919-980`:

```js
// 鼠标移动 → 计算目标角度
cover.addEventListener('mousemove', (e) => {
  const rect = card.getBoundingClientRect();
  const cx = rect.left + rect.width / 2;
  const cy = rect.top + rect.height / 2;
  const ry = Math.max(-1, Math.min(1, (e.clientX - cx) / (rect.width / 2)));
  const rx = Math.max(-1, Math.min(1, (e.clientY - cy) / (rect.height / 2)));
  _tiltTargetRY = ry * 6;       // Y 轴 ±6°
  _tiltTargetRX = -rx * 5;      // X 轴 ±5°（反向）
});

// ticker lerp 插值
gsap.ticker.add(() => {
  if (_projectsVisible) {
    _tiltRY += (_tiltTargetRY - _tiltRY) * 0.12;     // 阻尼 0.12
    _tiltRX += (_tiltTargetRX - _tiltRX) * 0.12;
    card.style.transform = 'rotateY(' + _tiltRY.toFixed(2) + 'deg) rotateX(' + _tiltRX.toFixed(2) + 'deg)';
  }
});
```

**注意**：`gsap.ticker` 每帧执行，比 `mousemove` 事件频率低且稳定。`0.12` 的阻尼系数产生柔和弹簧尾随效果。

### 流体曲线 SVG

`index.js:995-1012`:

```html
<svg class="fluid-line-svg" viewBox="0 0 1400 1400">
  <path class="fluid-line" id="fluid-line" d="
    M -80,0
    C 300,-20  600,150  540,400
    C 490,650   0,655    300,1050
    C 600,1385 650,1250 850,1200
    C 1050,1150 1350,1250 1540,1300
  " />
</svg>
```

```css
.fluid-line {
  fill: none; stroke: #ff1e00; stroke-width: 72;
  stroke-linecap: round;
}
```

```js
const lineLen = linePath.getTotalLength();              // 自动计算路径长度
gsap.set(linePath, { strokeDasharray: lineLen, strokeDashoffset: lineLen });
gsap.to(linePath, {
  strokeDashoffset: 0,
  scrollTrigger: {
    trigger: '#projects', start: 'top 70%', end: 'bottom 20%', scrub: 1,
  },
});
```

**SVG `getTotalLength()`** API 返回路径的实际像素长度。`stroke-dashoffset` 从 `lineLen` 到 `0` 实现"画线"效果。

### Lenis 动态 lerp

`index.js:983-993`:

```js
items.forEach(item => {
  ScrollTrigger.create({
    trigger: item,
    start: 'top 52%', end: 'bottom 48%',
    onEnter: () => lenis.options.lerp = 0.04,     // 项目区域减速
    onLeave: () => lenis.options.lerp = 0.06,     // 离开恢复
  });
});
```

在项目列表区域降低 Lenis 滚动阻尼，使滚动更平滑。

---

## 四、Circle Gallery — 3D 圆柱环绕

### 图片切片系统

`index.js:1015-1057`:

```js
(function buildSlices() {
  var SLICES = 10;
  var imgW = Math.min(Math.max(120, vw * 0.14), 210);   // clamp 120-210px
  var imgH = imgW * 2 / 3;
  var orbitR = (vw * 0.34 + 500) / 2;                    // 轨道半径 ≈ 视口相关
  var bendRad = imgW / orbitR;                            // 弯曲弧度
  var sliceW = imgW / SLICES;
  var stepDeg = (bendRad * 180 / Math.PI) / SLICES;      // 每片旋转角度

  document.querySelectorAll('.cg-img').forEach(function (img) {
    var wrapper = document.createElement('div');
    wrapper.className = 'cg-img';                          // 替换原 img

    for (var s = 0; s < SLICES; s++) {
      var sl = document.createElement('div');
      sl.className = 'cg-slice';
      sl.style.width = (sliceW + 1.5) + 'px';              // +1.5px 消除间隙
      sl.style.backgroundImage = 'url(' + src + ')';
      sl.style.backgroundPosition = (-s * sliceW).toFixed(1) + 'px 0';
      sl.style.transformOrigin = '50% 50% ' + (-cylR).toFixed(1) + 'px';  // 曲率中心
      var angle = (s - (SLICES - 1) / 2) * stepDeg;
      sl.style.transform = 'rotateY(' + angle.toFixed(2) + 'deg)';         // 绕 Y 弯曲
      wrapper.appendChild(sl);
    }
    img.parentNode.replaceChild(wrapper, img);
  });
})();
```

**关键参数**：
- `SLICES = 10` — 每个图片 10 片
- `transformOrigin: '50% 50% -cylRpx'` — 旋转中心在 Z 轴负方向 `cylR` 处，形成圆柱弯曲
- `+1.5px` 微调避免切片间像素缝隙

### 滚动驱动位置

`index.js:1086-1166`:

```js
// 椭圆轨道参数
const rx = vw * 0.34;          // X 轴半径
const rz = 500;                // Z 轴半径
const tiltY = vw <= 768 ? 80 : 180;   // Y 轴倾斜

function getPos(t) {
  if (t <= 0.12) {
    // 入场阶段：从屏幕外（-offX）进入，z 从 0→rz
    return { x: -offX * (1 - p), y: tiltY, z: rz * p, rotY: 0 };
  }
  if (t <= 0.88) {
    // 环绕阶段：沿椭圆轨道运动，自身旋转
    var angle = entryAngle - p * Math.PI * 2;
    return {
      x: Math.cos(angle) * rx,
      y: (z / rz) * tiltY,          // z 越大 y 越高（立体感）
      z: Math.sin(angle) * rz,
      rotY: p * Math.PI * 2         // 自身旋转 360°
    };
  }
  // 出场阶段：向右飞出屏幕
  return { x: offX * p, y: tiltY, z: rz * (1 - p), rotY: Math.PI * 2 };
}

var stagger = 0.09;                // 每张图 9% 偏移
var totalRange = 1 + stagger * (count - 1);

ScrollTrigger.create({
  trigger: '#circle-gallery',
  start: 'top top', end: 'bottom bottom',
  pin: '#circle-gallery-pin',       // 固定容器，实现全屏滚动
  onUpdate: function (self) {
    var progress = self.progress;
    cgImgs.forEach(function (img, i) {
      var imgT = progress * totalRange - i * stagger;   // 错开每张图
      if (imgT <= 0 || imgT >= 1) { img.style.opacity = '0'; return; }
      var pos = getPos(imgT);
      img.style.transform = 'translate3d(' + pos.x.toFixed(1) + 'px,' + pos.y.toFixed(1) + 'px,' + pos.z.toFixed(1) + 'px) rotateY(' + rotDeg + 'deg)';
      img.style.zIndex = Math.round(pos.z + 600);     // 深度排序
    });
  }
});
```

**stagger 效果**：8 张图片以 `0.09` 的时间偏移依次入场/环绕/出场，在滚动中形成连续的圆柱旋转队列。

### 短语文字同步

```js
var phraseStart = 0.25, phraseEnd = 0.75;
var travelY = 200;
// 文字随滚动在 Y 轴上下浮动（正弦轨迹）
var yOffset = travelY * (0.5 - globalP);
cgPhrase.style.transform = 'translateY(' + yOffset.toFixed(1) + 'px)';
// 单词按顺序 blur→clear
var wordT = revealP * (cgPhraseWords.length + 4) - wi;
var wP = Math.max(0, Math.min(1, wordT / 3));
w.style.opacity = wP;
w.style.filter = 'blur(' + (8 * (1 - wP)).toFixed(1) + 'px)';
```

---

## 五、侧边滚动导航

`index.js:1207-1311`:

```js
var sections = [
  { id: 'about', name: 'About' },
  { id: 'projects', name: 'Projects' },
  { id: 'circle-gallery', name: 'Gallery' },
  { id: 'skills', name: 'Skills' },
  { id: 'contact', name: 'Contact' },
].filter(function (sec) {
  if (sec.id === 'circle-gallery' && isMobileViewport()) return false;  // Mobile 无画廊
  return true;
});

// 比例计算
sections.forEach(function (sec) {
  sec.ratio = el.offsetHeight / zoneH;    // 每节高度 / 总高度
  var seg = document.createElement('div');
  seg.className = 'st-seg';
  seg.style.flex = sec.ratio.toFixed(4);  // flex 比例 = 高度占比
  bar.appendChild(seg);
});

// 滚动更新
ScrollTrigger.create({
  onUpdate: function (self) {
    var progress = self.progress;
    // 计算当前激活章节
    sections.forEach((sec, i) => {
      segEls[i].fill.style.height = (填充百分比) + '%';
    });
    label.textContent = sections[activeIdx].name;
    label.style.top = (progress * 100).toFixed(1) + '%';   // 标签跟随进度
  }
});
```

**交互**：每段可点击 → `lenis.scrollTo(target, { duration: 1.2 })`。hover 时 `scaleX(3)` 横向展开。
