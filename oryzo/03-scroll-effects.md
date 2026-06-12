# Scroll-Driven 效果集 — Social Grid / Features / Grip / Encryption

---

## 一、Social Content Grid

8 格社交内容网格，滚动触发逐项 reveal + 视差。

### HTML

```html
<section id="social-content" class="section">
  <div class="section__inner">
    <div class="social-content-grid">
      <!-- 每个 item 由 JS 注入 -->
      <div class="social-content-item"
           data-id="edge"
           data-offset="0.49">
        <div class="social-content-item__clipper">
          <div class="social-content-item__inner">
            <div class="social-content-item__bg">
              <picture>
                <source srcset="edge.webp" media="(min-width: 768px)" />
                <img src="edge_MOBILE.webp" loading="lazy" />
              </picture>
            </div>
            <div class="social-content-item__cover">
              <div class="social-content-item__title">Runs on the edge</div>
              <div class="social-content-item__note">Refuses the cloud</div>
            </div>
          </div>
        </div>
      </div>
      <!-- sticker_1, always_on, color, 3090, perfect, drop_test, sticker_2, legacy_support -->
    </div>
  </div>
</section>
```

### Scroll 触发 Reveal

```javascript
// 每个 item 的 data-offset 控制其出现时机
const items = document.querySelectorAll('.social-content-item')

items.forEach(item => {
  const offset = parseFloat(item.dataset.offset) || 0
  const clipper = item.querySelector('.social-content-item__clipper')
  const inner = item.querySelector('.social-content-item__inner')

  // clip-path 展开 (从 0% → 100% 高度)
  ScrollTrigger.create({
    trigger: item,
    start: `top ${80 - offset * 40}%`,
    end: `top ${40 - offset * 40}%`,
    scrub: true,
    onUpdate: (self) => {
      const p = self.progress
      clipper.style.clipPath = `inset(0 0 ${(1 - p) * 100}% 0)`
      inner.style.transform = `translateY(${(1 - p) * 50}px)`
      inner.style.opacity = p
    },
  })

  // 背景图片视差
  const bg = item.querySelector('.social-content-item__bg')
  ScrollTrigger.create({
    trigger: item,
    start: 'top bottom',
    end: 'bottom top',
    scrub: true,
    onUpdate: (self) => {
      bg.style.transform = `translateY(${self.progress * 30}px)`
    },
  })
})
```

### 布局

```css
.social-content-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2px;              /* 极细间距，制造"拼图"感 */
}

.social-content-item {
  overflow: hidden;
  aspect-ratio: var(--ratio);  /* 每个 item 可能不同比例 */
}

.social-content-item__clipper {
  clip-path: inset(0 0 100% 0);   /* 初始即隐藏 */
  will-change: clip-path;
}

.social-content-item__inner {
  will-change: transform, opacity;
}

/* 不同 data-id 占据不同的网格跨度和比例 */
[data-id="edge"]       { grid-column: span 2; --ratio: 16/9; }
[data-id="sticker_1"]  { grid-column: span 1; --ratio: 1/1; }
[data-id="always_on"]  { grid-column: span 1; --ratio: 4/3; }
[data-id="color"]      { grid-column: span 1; --ratio: 3/4; }
[data-id="3090"]       { grid-column: span 2; --ratio: 16/9; }
[data-id="drop_test"]  { grid-column: span 1; --ratio: 4/3; }
```

---

## 二、Features — 温度滑块 + 3D 曲线

### Temperature 滑块

```html
<div class="features-item temperature" data-class="temperature">
  <div class="features-temperature-slider">
    <div class="features-temperature-slider__bar" id="features-temperature-slider__bar">
      <div class="features-temperature-slider__bar-fill" />
      <div class="features-temperature-slider__bar-handle"
           id="features-temperature-slider__bar-handle" />
    </div>
    <span class="features-temperature-slider__text">Thermal Diffusion Model (TDM)</span>
  </div>
</div>
```

```javascript
const slider = document.getElementById('features-temperature-slider__bar')
const handle = document.getElementById('features-temperature-slider__bar-handle')

let isDragging = false

slider.addEventListener('mousedown', (e) => {
  isDragging = true
  updateSlider(e)
})

window.addEventListener('mousemove', (e) => {
  if (!isDragging) return
  updateSlider(e)
})

window.addEventListener('mouseup', () => { isDragging = false })

function updateSlider(e) {
  const rect = slider.getBoundingClientRect()
  const x = Math.max(0, Math.min(1, (e.clientX - rect.left) / rect.width))
  handle.style.left = `${x * 100}%`
  slider.style.setProperty('--progress', x)

  // 同步到 Three.js 曲线
  if (featuresCurve) {
    featuresCurve.updateProgress(x)
  }
}
```

### Curve — Three.js 曲线可视化

```html
<canvas id="features-curve-canvas" />
```

```javascript
import * as THREE from 'three'

const curveCanvas = document.getElementById('features-curve-canvas')
const curveScene = new THREE.Scene()
const curveCamera = new THREE.OrthographicCamera(-2, 2, 1, -1, 0.1, 10)

// 贝塞尔曲线
const curve = new THREE.CubicBezierCurve3(
  new THREE.Vector3(-1.5, -0.5, 0),
  new THREE.Vector3(-0.5, 1.5, 0),
  new THREE.Vector3(0.5, -1, 0),
  new THREE.Vector3(1.5, 0.5, 0),
)

const points = curve.getPoints(50)
const geometry = new THREE.BufferGeometry().setFromPoints(points)
const material = new THREE.LineBasicMaterial({ color: 0xff6b35, linewidth: 2 })
const curveLine = new THREE.Line(geometry, material)
curveScene.add(curveLine)

// 温度"粒子"沿曲线运动的点
const dotGeo = new THREE.SphereGeometry(0.08, 16, 16)
const dotMat = new THREE.MeshBasicMaterial({ color: 0xff6b35 })
const dot = new THREE.Mesh(dotGeo, dotMat)
curveScene.add(dot)

// 滑块驱动粒子沿曲线移动
window.featuresCurve = {
  updateProgress(t) {
    const point = curve.getPoint(t)
    dot.position.copy(point)
    dot.material.color.setHSL(0.08 - t * 0.08, 1, 0.5)  // 冷暖色过渡
  }
}
```

---

## 三、Grip — 滚动驱动放大

```html
<section id="grip" class="section">
  <div class="section__inner">
    <div class="grip-zoom-box">
      <div class="grip-zoom-content">
        <span class="grip-friction-value">0.80</span>
        <span class="grip-label">Friction coefficient (est)</span>
      </div>
    </div>
  </div>
</section>
```

```javascript
ScrollTrigger.create({
  trigger: '#grip',
  start: 'top bottom',
  end: 'bottom top',
  scrub: true,
  onUpdate: (self) => {
    const box = document.querySelector('.grip-zoom-box')
    const scale = 1 + self.progress * 0.3       // 1.0 → 1.3
    const blur = Math.max(0, 4 - self.progress * 8) // 4px → 0px
    box.style.transform = `scale(${scale})`
    box.style.filter = `blur(${blur}px)`
  },
})
```

---

## 四、Encryption — 翻转 DEMO

```html
<section id="encryption" class="section">
  <div class="section__inner">
    <div class="encryption-field">
      <input class="encryption-field-line" placeholder="Write a message..." />
      <div class="is-flipper" id="encryption-field-flip-btn">
        <div class="is-flipper-target">
          <span>Encode Message</span>
        </div>
        <div class="is-flipper-target is-active">
          <span>Decode Message</span>
        </div>
      </div>
    </div>
  </div>
</section>
```

```css
.is-flipper {
  perspective: 600px;
  cursor: pointer;
}

.is-flipper-target {
  transition: transform 0.6s cubic-bezier(0.65, 0.05, 0, 1),
              opacity 0.3s ease;
  backface-visibility: hidden;
  position: absolute;
  inset: 0;
}

.is-flipper-target:first-child {
  /* 正面：正常 */
}

.is-flipper-target:last-child {
  transform: rotateX(180deg);
  /* 背面：X 轴翻转 180° */
}

/* 翻转状态 */
.is-flipper.is-flipped .is-flipper-target:first-child {
  transform: rotateX(180deg);
  opacity: 0;
}

.is-flipper.is-flipped .is-flipper-target:last-child {
  transform: rotateX(0deg);
  opacity: 1;
}
```

```javascript
const flipBtn = document.getElementById('encryption-field-flip-btn')
let isFlipped = false

flipBtn.addEventListener('click', () => {
  isFlipped = !isFlipped
  flipBtn.classList.toggle('is-flipped', isFlipped)

  const input = document.querySelector('.encryption-field-line')
  if (isFlipped) {
    input.placeholder = 'Enter encrypted message...'
  } else {
    input.placeholder = 'Write a message...'
  }
})
```

---

## 五、Scroll Indicator

```html
<div id="scroll-indicator">
  <div id="scroll-indicator__bar">
    <div id="scroll-indicator__bar-inner" />
    <div id="scroll-indicator__bar-dot" />
  </div>
</div>
```

```css
#scroll-indicator {
  position: fixed;
  right: 0;
  top: 0;
  height: 100vh;
  width: 20px;
  z-index: 100;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  pointer-events: none;
}

#scroll-indicator__bar {
  width: 1px;
  height: 30vh;
  background: rgba(255,255,255,0.15);
  position: relative;
}

#scroll-indicator__bar-inner {
  position: absolute;
  bottom: 0;
  width: 100%;
  height: 0%;
  background: #fff;
  transition: height 0.1s linear;
}

#scroll-indicator__bar-dot {
  position: absolute;
  bottom: 0;
  left: 50%;
  width: 6px;
  height: 6px;
  background: #fff;
  border-radius: 50%;
  transform: translate(-50%, 50%);
}
```

```javascript
lenis.on('scroll', (e) => {
  const progress = e.animatedScroll / (document.documentElement.scrollHeight - window.innerHeight)
  const inner = document.getElementById('scroll-indicator__bar-inner')
  inner.style.height = `${progress * 100}%`
})
```

---

## 六、交互元素一览

| 效果 | 技术 | 交互方式 |
|---|---|---|
| Social Grid | ScrollTrigger + Clip-Path | 滚动驱动 reveal + 视差 |
| Temperature Slider | JS Drag + Three.js | 鼠标拖拽 → 3D 曲线同步 |
| Grip Zoom | ScrollTrigger + Scale | 滚动驱动放大 + 模糊消失 |
| Encryption | CSS 3D Flip | 点击翻转 |
| Scroll Indicator | JS + CSS | 自动跟随滚动 |
