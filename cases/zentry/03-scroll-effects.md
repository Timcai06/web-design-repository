# Scroll-Driven 效果集 — 文字拆分 / 计数器 / Spline / Marquee

---

## 一、文字拆分动画系统

### useSplitText

```javascript
// composables/useSplitText.js
export function useSplitText(el, options = {}) {
  const { type = 'chars', stagger = 0.02 } = options

  onMounted(() => {
    const text = el.value.textContent.trim()
    el.value.textContent = ''

    if (type === 'chars') {
      // 拆分为字符
      el.value.innerHTML = text.split('').map((c, i) =>
        `<span class="split-char" data-i="${i}"
              style="display:inline-block">${c === ' ' ? '\u00A0' : c}</span>`
      ).join('')
    } else if (type === 'words') {
      // 拆分为单词
      el.value.innerHTML = text.split(/\s+/).map((w, i) =>
        `<span class="split-word" data-i="${i}"
              style="display:inline-block">${w}</span>`
      ).join(' ')
    }
  })

  // 返回字符/单词引用与动画函数
  const items = ref([])

  function animateIn(animationOpts = {}) {
    const {
      from = { y: 40, opacity: 0, rotateX: 30 },
      to = { y: 0, opacity: 1, rotateX: 0 },
      duration = 0.6,
      stagger: s = stagger,
      ease = 'power3.out',
      scrollTrigger = null,
    } = animationOpts

    gsap.fromTo(
      el.value.querySelectorAll('.split-char, .split-word'),
      from,
      { ...to, duration, stagger: s, ease, scrollTrigger }
    )
  }

  return { items, animateIn }
}
```

### 使用

```vue
<template>
  <h2 ref="titleRef">Welcome to Zentry</h2>
</template>

<script setup>
const titleRef = ref(null)
const { animateIn } = useSplitText(titleRef, { type: 'chars' })

onMounted(() => {
  animateIn({
    scrollTrigger: {
      trigger: titleRef.value,
      start: 'top 80%',
    },
  })
})
</script>
```

---

## 二、Clip-Path 文字揭示

Resn 大量使用 `clip-path: inset()` 实现文字从下往上的擦除出场：

```vue
<template>
  <div class="text-reveal" ref="revealRef">
    <div class="text-reveal__inner">
      <h2>{{ text }}</h2>
    </div>
  </div>
</template>

<script setup>
const revealRef = ref(null)

onMounted(() => {
  gsap.fromTo(
    revealRef.value,
    { clipPath: 'inset(0 0 100% 0)' },
    {
      clipPath: 'inset(0 0 0% 0)',
      duration: 1.2,
      ease: 'power3.inOut',
      scrollTrigger: {
        trigger: revealRef.value,
        start: 'top 85%',
        end: 'top 40%',
        scrub: true,
      },
    }
  )
})
</script>
```

```css
.text-reveal {
  clip-path: inset(0 0 100% 0);
  will-change: clip-path;
}

.text-reveal__inner {
  display: inline-block;
}
```

变体：**对角线揭示**

```css
clip-path: polygon(0 100%, 0 100%, 0 100%, 0 100%);
/* 动画到: polygon(0 0, 100% 0, 100% 100%, 0 100%) */
```

---

## 三、数字计数器动画

ZENT Token 区域和 "Who We Are" 区域使用 GSAP 数字动画：

```vue
<template>
  <div class="stat" ref="statRef">
    <span class="stat__number" ref="numRef">0</span>
    <span class="stat__label">{{ label }}</span>
  </div>
</template>

<script setup>
const props = defineProps({
  target: Number,     // 目标数字
  suffix: String,     // 后缀（如 "M", "K"）
  label: String,
})

const numRef = ref(null)
const statRef = ref(null)

onMounted(() => {
  const obj = { val: 0 }

  ScrollTrigger.create({
    trigger: statRef.value,
    start: 'top 85%',
    onEnter: () => {
      gsap.to(obj, {
        val: props.target,
        duration: 2,
        ease: 'power3.out',
        onUpdate: () => {
          numRef.value.textContent = Math.round(obj.val) + (props.suffix || '')
        },
      })
    },
    once: true,   // 只执行一次
  })
})
</script>
```

---

## 四、Spline 3D 元素

Resn 使用 **Spline**（3D 设计工具）创建交互式 3D 元素，通过 `@splinetool/runtime` 或 `@splinetool/viewer` 嵌入：

```vue
<template>
  <SplineComponent
    class="who-we-are__spline"
    :scene="splineSceneUrl"
    @load="onSplineLoad"
  />
</template>

<script setup>
import SplineComponent from './SplineComponent.vue'

const splineSceneUrl = 'https://prod.spline.design/.../scene.splinecode'

function onSplineLoad(splineApp) {
  // Spline 应用加载完成
  // 可以通过 splineApp 控制 3D 对象
  // 如: splineApp.emitEvent('mouseDown', 'Cube')
}
</script>
```

### SplineComponent (封装)

```vue
<template>
  <canvas ref="splineCanvas" />
</template>

<script setup>
import { Application } from '@splinetool/runtime'

const props = defineProps({
  scene: String,
})

const emit = defineEmits(['load'])
const splineCanvas = ref(null)

let app

onMounted(() => {
  app = new Application(splineCanvas.value)
  app.load(props.scene).then(() => {
    emit('load', app)
  })
})

onUnmounted(() => {
  if (app) app.dispose()
})
</script>
```

### 为什么用 Spline 而不是直接 Three.js？

| 方式 | 优点 | 缺点 |
|---|---|---|
| **Spline** | 设计师可独立产出 3D；快速迭代；内置交互 | 运行时较大；定制性有限 |
| **Three.js 自建** | 完全控制；极致性能 | 开发成本高；需要 3D 工程师 |
| **Rive** | 矢量动画性能好 | 不能处理 3D 模型 |

Zentry 在 "Who We Are" 区域使用 Spline 嵌入 3D 装饰元素，而**核心视觉效果（Portal Mask、卡片效果）全部使用 DOM 技术**。

---

## 五、Partners Logo Marquee

```vue
<template>
  <div class="partners" ref="partnersRef">
    <div class="partners__track">
      <div class="partners__track-inner" ref="trackRef">
        <img v-for="logo in logos" :key="logo.id"
             :src="logo.src" :alt="logo.name"
             class="partners__logo" />
        <!-- 复制一份用于无缝滚动 -->
        <img v-for="logo in logos" :key="'dup-' + logo.id"
             :src="logo.src" :alt="logo.name"
             class="partners__logo" aria-hidden="true" />
      </div>
    </div>
  </div>
</template>

<script setup>
const trackRef = ref(null)

onMounted(() => {
  // GSAP 水平无限滚动
  gsap.to(trackRef.value, {
    xPercent: -50,
    duration: 30,
    ease: 'none',
    repeat: -1,
  })
})

// 暂停/恢复 on hover
function onPartnersHover(entering) {
  gsap.to(trackRef.value, {
    timeScale: entering ? 0 : 1,
    duration: 0.3,
  })
}
</script>
```

```css
.partners__track {
  overflow: hidden;
  mask-image: linear-gradient(
    to right,
    transparent 0%,
    black 5%,
    black 95%,
    transparent 100%
  );
}

.partners__track-inner {
  display: flex;
  gap: 4rem;
  width: fit-content;
  will-change: transform;
}
```

---

## 六、Cumulative Scroll Effects 总览

| 效果 | 技术 | 交互方式 |
|---|---|---|
| Portal Mask | CSS `clip-path` + GSAP | 滚动驱动菱形展开 |
| 视频背景 | ScrollyVideo | 滚动驱动帧进度 |
| 标题字符弹入 | `useSplitText` + GSAP stagger | 滚动触发 |
| 文字 clip-path 揭示 | `clip-path: inset()` | 滚动擦除 |
| Bento 卡片入场 | GSAP from (y, opacity, scale) | 滚动触发 + stagger |
| Perspective tilt | `usePerspectiveTransform` + `useDamp` | 鼠标移动 |
| 卡片 hover reveal | `clip-path: ellipse()` | Hover |
| 数字计数器 | GSAP `onUpdate` | 滚动触发 |
| Partners Marquee | GSAP `xPercent` loop | 自动 + hover 暂停 |
| HUD 导航 | Lenis velocity | 滚动方向检测 |
| 音频可视化器 | `useRaf` | 常驻动画 |
| Spline 3D | `@splinetool/runtime` | 自动/鼠标交互 |
