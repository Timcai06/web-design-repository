# Contact Blob + Skills Accordion + ASCII Art Footer

源码: `index.js:1314-1868`

---

## 一、Skills 手风琴

`index.js:1314-1343`:

```js
var groups = document.querySelectorAll('.skill-group');
// 初始展开第一个
var firstBody = groups[0].querySelector('.skill-body');
firstBody.style.height = firstBody.scrollHeight + 'px';

groups.forEach(function (group) {
  group.querySelector('.skill-header').addEventListener('click', function () {
    if (group.classList.contains('open')) return;     // 已展开点击无效

    // 关闭所有已展开的
    groups.forEach(function (g) {
      if (g.classList.contains('open')) {
        g.classList.remove('open');
        gsap.to(g.querySelector('.skill-body'), { height: 0, duration: 0.45, ease: 'power3.inOut' });
      }
    });

    // 打开当前
    group.classList.add('open');
    gsap.to(body, {
      height: body.scrollHeight,        // 自动高度
      duration: 0.45, ease: 'power3.inOut',
      onComplete: function () { ScrollTrigger.refresh(); }
    });
  });
});
```

**单例模式**：任何时候只有一个 group 展开。使用 `scrollHeight` 获取内容实际高度（非固定值），GSAP 做 `height` 过渡。

### 箭头滚动位移

`index.js:1346-1368`:

```js
gsap.fromTo(arrow,
  { xPercent: 0 },
  {
    xPercent: 100,
    x: function () {
      var left = arrow.parentElement;
      var pad = parseFloat(getComputedStyle(left).paddingLeft) + parseFloat(getComputedStyle(left).paddingRight);
      return left.clientWidth - pad - arrow.offsetWidth;   // 计算可移动范围
    },
    ease: 'none',
    scrollTrigger: {
      trigger: '#skills',
      start: 'top top',
      endTrigger: '#contact',
      end: 'top center',
      scrub: 0.5,
    }
  }
);
```

红色箭头从 skills 区域滚动到 contact 区域，完整横向穿越父容器。`scrub: 0.5` 产生平滑跟随。

---

## 二、Contact Blob — 伪液态膨胀

### 核心 CSS

`styles/index.css`:

```css
.contact-blob {
  position: absolute;
  bottom: 0; left: 50%;
  width: 300vmax;                       // 所有屏幕上都远大于视口
  height: 300vmax;
  background: #f0f0f0;                  // 白色
  border-radius: 50%;                   // 完美圆形
  transform: translate(-50%, 50%) scale(0);  // 锚定底部中心 + 初始 0
  will-change: transform;
}
```

### GSAP 驱动

`index.js:1372-1488`:

```js
// 可见性控制
ScrollTrigger.create({
  trigger: '#contact',
  start: 'top bottom',
  endTrigger: '#footer-transition',
  end: 'bottom bottom',
  onEnter: function () { blobWrap.style.visibility = 'visible'; contactBg.style.display = 'block'; },
  onLeave: function () { blobWrap.style.visibility = 'hidden'; contactBg.style.display = 'none'; },
});

// 主时间线
var tl = gsap.timeline({
  scrollTrigger: {
    trigger: '#contact',
    start: 'top bottom',
    end: 'bottom bottom',
    scrub: true,
  }
});

// 1. Blob 膨胀：0 → 1
tl.fromTo(blob, { scale: 0 }, { scale: 1, duration: 0.6, ease: 'none' }, 0);

// 2. 隐藏侧边导航
tl.to([stTimeline, pctEl], { opacity: 0, duration: 0.08 }, 0.1);

// 3. Title "Contact" 从 x: window.innerWidth*1.1 飞入
gsap.set(title, { yPercent: 0, x: function () { return window.innerWidth * 1.1; } });
tl.to(title, { x: 0, duration: 0.3, ease: 'power3.out' }, 0.18);

// 4. Socials 从底部展开 (clipPath)
tl.fromTo(socials, { clipPath: 'inset(0 0 100% 0)' }, { clipPath: 'inset(0 0 0% 0)', duration: 0.2 }, 0.28);

// 5. Mail 从底部展开
tl.fromTo(mailEl, { clipPath: 'inset(0 0 100% 0)' }, { clipPath: 'inset(0 0 0% 0)', duration: 0.2 }, 0.36);
```

### 图片对平行飞入

```js
var pairStart = 0.22;
var frameDur = 0.65;
var frameY = function () { return window.innerHeight * 1.1; };      // 从屏幕下方
var frameYEnd = function () { return -window.innerHeight * 1.4; };  // 到屏幕上方

// 第1对：frame + dispo
gsap.set(frame, { yPercent: -50, y: frameY });
gsap.set(frameImg, { yPercent: -30 });           // 内部视差偏移
tl.to(frame, { y: frameYEnd, duration: frameDur, ease: 'none' }, pairStart);
tl.to(frameImg, { yPercent: 30, duration: frameDur, ease: 'none' }, pairStart);
tl.to(dispo, { y: dispoYEnd, duration: frameDur }, pairStart);
tl.to(dispo, { opacity: 0, clipPath: 'inset(100% 0 0% 0)', duration: 0.15 }, pairStart + 0.45);

// 第2对：frame2 + dispo2 (0.07s 偏移)
gsap.set(frame2, { yPercent: -50, y: function () { return window.innerHeight * 1.3; } });
tl.to(frame2, { y: frameYEnd, duration: frameDur, ease: 'none' }, pairStart + 0.07);
tl.to(frameImg2, { yPercent: 30, duration: frameDur }, pairStart + 0.07);
```

**"液体"错觉的原理**：`300vmax` 圆的边界在 `scale: 1` 时远超视口，底部看过去像是白色液体从底部涌出。圆的弯曲边缘穿过视口底部时看起来像**表面张力**。

### Footer 过渡

`index.js:1819-1868`:

```js
var ftl = gsap.timeline({
  scrollTrigger: {
    trigger: '#footer-transition',
    start: 'top bottom+=550',
    end: 'bottom bottom',
    scrub: true,
  }
});

ftl.set(contactBlobWrap, { height: '110vh' }, 0);
ftl.to(contactBlobWrap, { borderRadius: '0 0 50px 50px', duration: 0.15 }, 0);
ftl.to(contactBlobWrap, {
  y: function () { return -(window.innerHeight * 1.8 + 400); },  // 向上拉开
  duration: 1.0, ease: 'none'
}, 0);
ftl.to(contactPin, { y: '-40vh', duration: 1.0 }, 0);

// 白色背景上的内容消失 (clipPath 反向)
ftl.fromTo([ctSocials, ctMail], { clipPath: 'inset(0 0 0% 0)' }, { clipPath: 'inset(0 0 100% 0)' }, 0);
ftl.fromTo(ctTitle, { clipPath: 'inset(0 0 0% 0)' }, { clipPath: 'inset(0 0 100% 0)' }, 0);
```

---

## 三、ASCII Art Footer

### 图片→ASCII 转换

`index.js:1491-1539`:

```js
var POOLS = [
  ' ',               // 最暗 → 空白
  '·.,',             // 极暗
  ':;`-~^',          // 暗
  '=+<>?!:;',        // 中暗
  '|/\\()[]{}«»',    // 中亮
  '÷×±≈≠≤≥∞∑∏√∫',  // 亮
  '¤†‡§¶©®™°¬',     // 很亮
  '%&#$@¥€£¢'        // 最亮
];

function imageToAscii(img, cols) {
  seed = 42;  // 固定种子 → 每次渲染相同
  var c = document.createElement('canvas');
  var ctx = c.getContext('2d');
  var rows = Math.round(cols * aspect * charAspect);
  c.width = cols; c.height = rows;
  ctx.drawImage(img, 0, 0, cols, rows);
  var data = ctx.getImageData(0, 0, cols, rows).data;

  for (var y = 0; y < rows; y++) {
    for (var x = 0; x < cols; x++) {
      var i = (y * cols + x) * 4;
      var brightness = (0.299 * r + 0.587 * g + 0.114 * b) / 255; // 标准亮度公式
      brightness *= (a / 255);
      var pi = Math.floor(brightness * (POOLS.length - 1) * 0.8);  // 映射到字符池索引
      line += pool[Math.floor(rand() * pool.length)];              // 同级随机选字符
    }
  }
}
```

使用**标准 ITU-R BT.601 亮度公式** `0.299R + 0.587G + 0.114B`。`seed = 42` 固定保证每次渲染结果一致。

### 交互式翻转

`index.js:1541-1636`:

```js
function setupHover(preEl, poolGrid) {
  var radius = 2.5;   // 影响半径（字符数）
  var noise = [];
  var hitTime = [];
  var cellDuration = [];

  // 每格赋予不同噪声
  for (var ny = 0; ny < rows; ny++) {
    for (var nx = 0; nx < cols; nx++) {
      var h = (Math.sin(nx * 12.9898 + ny * 78.233) * 43758.5453 % 1 + 1) % 1;
      noise.push(h * 5 - 2.5);           // ±2.5 的噪声半径
      hitTime.push(0);
      cellDuration.push(h > 0.5 ? 200 : 100);  // 随机持续 100ms 或 200ms
    }
  }

  preEl.addEventListener('mousemove', function (e) {
    // 计算鼠标在字符矩阵中的坐标
    mxC = (e.clientX - rect.left) / charW;
    myC = (e.clientY - rect.top) / charH;
    // 圆形范围内所有字符标记为"被击中"
    for (var y = yMin; y <= yMax; y++) {
      for (var x = xMin; x <= xMax; x++) {
        if (dx * dx + dy * dy < (radius + noise[y][x]) * (radius + noise[y][x])) {
          hitTime[y][x] = now;
        }
      }
    }
  });

  function tick() {
    // 被击中的字符切换到"高亮"状态（反色 + 随机字符）
    // 未击中的保持原始字符
    // 高亮字符在 cellDuration 后恢复
    html += '<span style="color:#0a0a0a;background:#ff3b14">' + esc(ch) + '</span>';
    // ...
    if (anyActive) requestAnimationFrame(tick);
  }
}
```

**噪声种子** `Math.sin(nx * 12.9898 + ny * 78.233) * 43758.5453` 是经典的哈希函数，产生伪随机但空间一致的值。

### 视差跟随

`index.js:1690-1704`:

```js
var mx = 0, my = 0, sx = 0, sy = 0;
document.addEventListener('mousemove', function (e) {
  mx = (e.clientX / window.innerWidth - 0.5) * 2;     // [-1, 1]
  my = (e.clientY / window.innerHeight - 0.5) * 2;
});
function parallaxLoop() {
  sx += (mx - sx) * 0.05;                              // 低通滤波
  sy += (my - sy) * 0.05;
  var lx = Math.min(0, sx * -15 - 15);                 // 左半边向左偏移
  var rx = Math.max(0, sx * 15 + 15);                  // 右半边向右偏移
  var py = sy * -10;
  if (asciiLeftPre) asciiLeftPre.style.transform = 'translate(' + lx + 'px, ' + py + 'px)';
  if (asciiRightPre) asciiRightPre.style.transform = 'translate(' + rx + 'px, ' + py + 'px)';
  requestAnimationFrame(parallaxLoop);
}
```

两个 ASCII 面板朝相反方向偏移，配合鼠标产生立体感。`0.05` 的系数实现平滑跟随便，从不突兀跳变。

### Footer 名字逐字入场

`index.js:1742-1796`:

```js
// 将 "Luke Baffait." 按字符拆分，交错排列入场
var lukeRev = lukeChars.slice().reverse();    // "ekuL"
var rightSide = baffaitChars.concat(dotChars);
var maxLen = Math.max(lukeRev.length, rightSide.length);
for (var i = 0; i < maxLen; i++) {
  if (rightSide[i]) ordered.push(rightSide[i]);   // 先右后左
  if (lukeRev[i]) ordered.push(lukeRev[i]);
}
// 从两侧向中心交错弹入
gsap.set(ordered, { yPercent: 110 });
gsap.to(ordered, {
  yPercent: 0, ease: 'power3.out',
  stagger: { each: 0.04, from: 'start' },
  scrollTrigger: {
    trigger: '#footer-transition',
    start: 'center bottom+=500',
    end: 'bottom bottom',
    scrub: true,
  }
});
```

**交错逻辑**：将 "Lorem" 反转 → "meroL"，再与剩余字符交替排列，实现从两侧向中间依次弹出的效果。
