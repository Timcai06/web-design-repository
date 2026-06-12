# Hero — 3D Portal Mask + 视频 HUD + 导航

Zentry 最标志性的效果：**用 CSS `clip-path` 实现的 3D Portal Mask**，这是 Resn 刻意不使用 WebGL 的技术宣言。

---

## 一、Render Tree

```html
<template>
  <section class="hero">
    <div class="hero__media">
      <div class="hero__media-inner">
        <!-- 视频背景 (ScrollyVideo) -->
        <ScrollyVideo class="hero__video" :src="heroVideoSrc" />
        <!-- Portal Mask 叠加层 -->
        <div class="hero__portal-mask" ref="portalMask">
          <video class="hero__portal-video" src="portal.mp4" muted loop />
        </div>
      </div>
    </div>
    <div class="hero__content">
      <h1 class="hero__title">
        <span class="hero__title-line" ref="titleLine1">REDEFINE</span>
        <span class="hero__title-line" ref="titleLine2">REALITY</span>
      </h1>
      <p class="hero__subtitle">Enter the Human-Agentic OS</p>
    </div>
    <!-- HUD 导航 (浮动, fixed 定位) -->
    <NavBar class="hero__navbar" />
  </section>
</template>
```

---

## 二、3D Portal Mask (纯 CSS/clip-path)

这是全站最核心的技术创新：**一个看起来像 3D 传送门的视觉，完全由 CSS clip-path 实现**，零 WebGL。

### 原理

```
视频 1 (背景)                   视频 2 (Portal)
┌──────────────────────┐       ┌──────────────────────┐
│                      │       │                      │
│  背景场景 (持续播放)   │       │    传送门内容         │
│                      │       │    (由 clip-path      │
│                      │       │     裁剪为菱形)        │
│                      │       │                      │
└──────────────────────┘       └──────────────────────┘
         │                              │
         │                              │
         ▼                              ▼
   ┌──────────────────────────────────────────┐
   │           最终渲染                        │
   │  ┌──────────────────────────────────┐    │
   │  │  背景视频                         │    │
   │  │       ┌──────────┐              │    │
   │  │       │ Portal   │              │    │
   │  │       │ (clip-   │              │    │
   │  │       │  path    │              │    │
   │  │       │ 菱形)     │              │    │
   │  │       └──────────┘              │    │
   │  └──────────────────────────────────┘    │
   └──────────────────────────────────────────┘
```

### 实现

```vue
<script setup>
const portalMask = ref(null)

// useCssClipMask — 自定义 composable 管理 clip-path 动画
const { clipPath, updateProgress } = useCssClipMask(portalMask, {
  // 路径定义：菱形 → 展开/收缩
  initialPath: 'polygon(50% 50%, 50% 50%, 50% 50%, 50% 50%)',
  finalPath: 'polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%)',
  // 或更复杂的 portal 形状
})

// ScrollTrigger 控制进度
useBaseScrollTrigger(portalMask, {
  start: 'top top',
  end: 'bottom top',
  scrub: true,
  onUpdate: (self) => {
    updateProgress(self.progress)
  },
})
</script>
```

### CSS

```css
.hero__portal-mask {
  position: absolute;
  inset: 0;
  clip-path: polygon(50% 50%, 50% 50%, 50% 50%, 50% 50%);
  /*                  左上      右上      右下      左下    */
  /* 初始收缩到中心点，滚动中展开 */
  transition: clip-path 0.1s linear; /* GSAP 驱动，transition 仅作后备 */
  will-change: clip-path;
  z-index: 2;
}

.hero__portal-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

### ScrollTrigger 动画

```javascript
// 滚动时 clip-path 从中心菱形 → 全屏展开
onMounted(() => {
  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: '.hero',
      start: 'top top',
      end: 'bottom top',
      scrub: 1.5,
    },
  })

  tl.to(portalMask.value, {
    clipPath: 'polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%)',
    ease: 'power2.inOut',
  }, 0)
})
```

---

## 三、ScrollyVideo — 滚动驱动视频

Hero 背景使用 `ScrollyVideo` 库实现**滚动控制视频进度**：

```vue
<template>
  <ScrollyVideo
    class="hero__video"
    :src="heroVideoSrc"
    :frameRate="30"
    :sticky="false"
    :cover="true"
  />
</template>

<script setup>
import ScrollyVideo from 'scrolly-video'

const heroVideoSrc = 'https://cdn.zentry.com/videos/hero-bg.mp4'
</script>
```

ScrollyVideo 的工作原理：
- 将视频帧映射到滚动进度
- 滚动时跳到对应帧（类似逐帧视频）
- `sticky: false` 表示视频不固定在视口

---

## 四、HUD 导航

浮动 navigation，带有滚动方向感知和音频可视化器。

### 方向感知

```vue
<script setup>
const nav = ref(null)
const { isVisible } = useOnScroll({
  threshold: 100,   // 滚动 100px 后触发
})

// 滚动方向检测
const scrollDirection = computed(() => {
  // Lenis 的 velocity > 0 → 向下滚动
  // lenis.velocity < 0 → 向上滚动
  return velocity.value > 0 ? 'down' : 'up'
})

// 向下滚动隐藏 nav，向上滚动显示
watch(scrollDirection, (dir) => {
  gsap.to(nav.value, {
    y: dir === 'down' ? '-100%' : '0%',
    duration: 0.4,
    ease: 'power3.out',
  })
})
</script>
```

### 音频可视化器

```vue
<template>
  <div class="nav__audio-visualizer" ref="audioViz">
    <span v-for="i in 5" :key="i" class="viz-bar"
          :style="{ height: barHeights[i-1] + 'px' }" />
  </div>
</template>

<script setup>
const barHeights = ref([4, 8, 12, 8, 4])

// 使用 requestAnimationFrame 驱动伪随机跳动
useRaf(() => {
  if (Math.random() > 0.7) {
    barHeights.value = barHeights.value.map(() =>
      4 + Math.random() * 16
    )
  }
})
</script>
```

## 五、"REDEFINE REALITY" 标题动画

双词渐进揭示，配合视频 Portal 展开：

```vue
<template>
  <h1 class="hero__title">
    <span class="hero__title-line" ref="line1">
      <span v-for="(char, i) in 'REDEFINE'" :key="i"
            class="char" ref="chars1">{{ char }}</span>
    </span>
    <span class="hero__title-line" ref="line2">
      <span v-for="(char, i) in 'REALITY'" :key="i"
            class="char" ref="chars2">{{ char }}</span>
    </span>
  </h1>
</template>

<script setup>
const chars1 = ref([])
const chars2 = ref([])

onMounted(() => {
  // 词 1: REDEFINE — 逐字从下方弹入
  gsap.from(chars1.value, {
    y: 80,
    opacity: 0,
    rotateX: 40,
    duration: 0.6,
    stagger: 0.04,
    ease: 'power4.out',
    scrollTrigger: {
      trigger: '.hero',
      start: 'top 80%',
    },
  })

  // 词 2: REALITY — 延迟出现
  gsap.from(chars2.value, {
    y: 80,
    opacity: 0,
    rotateX: 40,
    duration: 0.6,
    stagger: 0.04,
    delay: 0.3,
    ease: 'power4.out',
    scrollTrigger: {
      trigger: '.hero',
      start: 'top 60%',
    },
  })
})
</script>
```
