# 项目卡片系统 — Rive Circuit Map / Helmet Hover / Rive Button Hover

---

## 一、Next Race Card — Rive Circuit Map（`I4()` 函数）

### HTML

```html
<canvas data-rive-object
        data-rive-file="circuits"
        data-rive-artboard="circuits"
        data-rive-state-machine="circuits"
        data-rive-input-track="monaco"
        data-rive-input-color=""
        data-rive-input-weight=""
        data-rive-fit="contain"
        data-rive-circuit-hover="">
</canvas>
```

### JS — `I4()` 函数核心逻辑

```javascript
function I4() {
  // 查找所有 data-rive-object canvas
  document.querySelectorAll('[data-rive-object]').forEach(canvas => {
    const rive = new Rive({
      src: `/rive/${canvas.dataset.riveFile}.riv`,
      artboard: canvas.dataset.riveArtboard,
      stateMachine: canvas.dataset.riveStateMachine,
      canvas,
      autoplay: true,
      onLoad: () => {
        // 1. 将 scroll 输入映射到滚动进度
        const scrollInput = rive.stateMachineInputs.find(i => i.name === 'scroll')
        if (scrollInput) {
          window.addEventListener('scroll', () => {
            scrollInput.value = window.scrollY / (document.documentElement.scrollHeight - window.innerHeight)
          })
        }

        // 2. 处理 data-rive-input-track → 设置当前赛道 boolean input
        const track = canvas.dataset.riveInputTrack
        if (track) {
          const input = rive.stateMachineInputs.find(i => i.name === track)
          if (input) input.value = true
        }

        // 3. data-rive-input-color / data-rive-input-weight 同理
      }
    })

    // 4. data-rive-circuit-hover：监听所有 [data-rive-circuit-hover-target] 的 hover
    document.querySelectorAll('[data-rive-circuit-hover-target]').forEach(target => {
      target.addEventListener('mouseenter', () => {
        rive.stateMachineInputs.find(i => i.name === 'hover')?.value = true
      })
      target.addEventListener('mouseleave', () => {
        rive.stateMachineInputs.find(i => i.name === 'hover')?.value = false
      })
    })
  })
}
```

---

## 二、Helmet Hover — 纯 CSS Dual-Image Reveal

零 JavaScript，纯 CSS `clip-path: ellipse()` 动画。

```html
<div class="helmet-grid-item" data-helmet-item>
  <div class="helmet-img-mask-w">
    <img src="helmet-base.webp" alt="" class="helmet-img-base" />
  </div>
  <div class="helmet-img-reveal-w">
    <img src="helmet-reveal.webp" alt="" class="helmet-img-reveal" />
  </div>
  <img src="helmet-frame.svg" alt="" class="helmet-frame" />
</div>
```

```css
.helmet-img-reveal-w {
  position: absolute;
  inset: 0;
  clip-path: ellipse(0% 0% at 50% 0%);
  transition: clip-path 0.6s var(--cubic-default);
  /* --cubic-default: cubic-bezier(0.65, 0.05, 0, 1) */
}

[data-helmet-item]:hover .helmet-img-reveal-w {
  clip-path: ellipse(100% 120% at 50% 0%);
}

.helmet-frame {
  position: absolute;
  inset: 0;
  opacity: 0;
  transition: opacity 0.4s var(--animation-default);
}

[data-helmet-item]:hover .helmet-frame {
  opacity: 1;
}
```

---

## 三、Button Rive Hover（`C4()` 函数）

`C4()` 函数为每个 `[data-btn-rive-hover]` 初始化 Rive 箭头动画。

```html
<a data-btn-rive-rotate="true" data-btn-rive-hover href="/on-track">
  <div class="btn-icon-w">
    <canvas data-rive-object
            data-rive-file="btn-ui"
            data-rive-artboard="arrow"
            data-rive-state-machine="arrow"
            data-rive-hover="false"
            data-rive-fit="contain">
    </canvas>
  </div>
</a>
```

```javascript
function C4() {
  document.querySelectorAll('[data-btn-rive-hover]').forEach(wrapper => {
    const canvas = wrapper.querySelector('[data-rive-object]')
    if (!canvas) return

    const rive = new Rive({
      src: `/rive/btn-ui.riv`,
      artboard: 'arrow',
      stateMachine: 'arrow',
      canvas,
      autoplay: true,
      onLoad: () => {
        const hoverInput = rive.stateMachineInputs.find(i => i.name === 'hover')

        const enter = () => { if (hoverInput) hoverInput.value = true }
        const leave = () => { if (hoverInput) hoverInput.value = false }

        wrapper.addEventListener('mouseenter', enter)
        wrapper.addEventListener('mouseleave', leave)
        wrapper.addEventListener('touchstart', enter)
        wrapper.addEventListener('touchend', leave)
      }
    })
  })
}
```

### CSS — 旋转 & 反色

```css
[data-btn-rive-rotate="true"] .btn-rive-w { transform: rotate(180deg); }
[data-btn-rive-rotate="90"]  .btn-rive-w { transform: rotate(90deg); }
[data-rive-btn-invert="true"] .btn-icon-w { filter: invert(1); }
```

---

## 四、"On Track / Off Track" 分流区

主页上下分屏，分别链接到 `/on-track` 和 `/off-track`，每侧含 Rive 箭头按钮 + split-text 标题。

| 左侧 | 右侧 |
|------|------|
| ON TRACK | OFF TRACK |
| → `/on-track` | → `/off-track` |

---

## 五、Color Theme Transition System

基于滚动进度插值颜色，通过 data attributes 声明式配置。

```html
<section data-gl-change-to="white, dark-green-tint-1-low"
         data-gl-change-from="dark-green, dark-green-tint-1"
         data-horizontal-section
         data-h-color-from="dark-green"
         data-h-color-to="white">
</section>
```

```javascript
// ScrollTrigger 驱动颜色插值
ScrollTrigger.create({
  trigger: section,
  start: 'top bottom',
  end: 'bottom top',
  scrub: true,
  onUpdate: (self) => {
    const p = self.progress
    // 解析 data-gl-change-from / data-gl-change-to
    // 对每个颜色对做 lerpColor()，更新 CSS 变量
    // 同时传递到子元素的 Rive color inputs
  }
})
```

设计库 `--color-*` 变量通过设计师 token 命名（如 `dark-green`, `white`, `dark-green-tint-1-low`），在 CSS 中以 CSS 自定义属性定义，运行时根据滚动进度插值切换。
