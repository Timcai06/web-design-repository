# Bento Grid — 产品卡片 + Perspective Tilt + Mask Reveal

---

## 一、布局结构

```
┌──────────────────────────────────────────────────────────┐
│                    Nexus / Products Section               │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   zDATA       │  │   zAI        │  │   zTERMINAL  │   │
│  │               │  │              │  │              │   │
│  │  Data layer   │  │ Agentic AI   │  │ Terminal     │   │
│  │  turning      │  │ layer power  │  │ for the      │   │
│  │  human data   │  │ a playable   │  │ open web     │   │
│  │  into meta-   │  │ agent society│  │              │   │
│  │  contextual   │  │              │  │              │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   RADIANT     │  │   ZIGMA      │  │   (CTA)      │   │
│  │               │  │              │  │              │   │
│  │  Game-of-     │  │  NFT +       │  │  Enter Vault │   │
│  │  games portal │  │  agentic AI  │  │              │   │
│  │               │  │  + community │  │              │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└──────────────────────────────────────────────────────────┘
```

### HTML 结构

```vue
<template>
  <Bento class="nexus">
    <div class="bento__grid">
      <BentoCard v-for="(item, i) in products" :key="i"
                 :data="item"
                 :index="i"
                 class="bento__card" />
    </div>
    <!-- 背景装饰元素 -->
    <Sting class="bento__sting" />
  </Bento>
</template>
```

---

## 二、Bento Card 组件

### Props

```vue
<script setup>
const props = defineProps({
  data: Object,    // { title, description, media, link, theme }
  index: Number,
})

const cardRef = ref(null)
</script>
```

### 渐进入场（ScrollTrigger）

```vue
<script setup>
onMounted(() => {
  // Bento 卡片延迟交错入场
  gsap.from(cardRef.value, {
    y: 60,
    opacity: 0,
    scale: 0.95,
    duration: 0.8,
    delay: props.index * 0.1,           // 每张卡延迟 100ms
    ease: 'power3.out',
    scrollTrigger: {
      trigger: cardRef.value,
      start: 'top 85%',
      toggleActions: 'play none none reverse',
    },
  })
})
</script>
```

---

## 三、Perspective Tilt — 鼠标驱动的 3D 倾斜

### usePerspectiveTransform

```vue
<script setup>
const cardRef = ref(null)

// 自定义 composable
const { rotateX, rotateY } = usePerspectiveTransform(cardRef, {
  maxAngle: 8,           // 最大倾斜角度 ±8°
  sensitivity: 0.1,      // 跟随速度（阻尼系数）
  perspective: 800,      // CSS perspective 值
})

// 输出 CSS transform
const cardStyle = computed(() => ({
  transform: `perspective(800px) rotateX(${rotateX.value}deg) rotateY(${rotateY.value}deg)`,
  transition: 'transform 0.1s ease-out',
}))
</script>
```

### usePerspectiveTransform 实现

```javascript
// composables/usePerspectiveTransform.js
export function usePerspectiveTransform(el, options = {}) {
  const { maxAngle = 8, sensitivity = 0.1, perspective = 800 } = options

  const rotateX = ref(0)
  const rotateY = ref(0)

  // 鼠标归一化坐标（-1 到 1）
  const { x, y } = useWindowPointerNorm(el)

  // 阻尼跟随
  const dampedX = useDamp(x, { speed: sensitivity })
  const dampedY = useDamp(y, { speed: sensitivity })

  // 监听变化
  watch([dampedX, dampedY], ([nx, ny]) => {
    rotateY.value = nx * maxAngle    // X 轴移动控制 Y 轴旋转
    rotateX.value = -ny * maxAngle   // Y 轴移动控制 X 轴旋转
  })

  return { rotateX, rotateY }
}
```

### useWindowPointerNorm

```javascript
// composables/useWindowPointerNorm.js
export function useWindowPointerNorm(el) {
  const x = ref(0)
  const y = ref(0)

  function onPointer(e) {
    const rect = el.value.getBoundingClientRect()
    x.value = ((e.clientX - rect.left) / rect.width - 0.5) * 2
    y.value = ((e.clientY - rect.top) / rect.height - 0.5) * 2
  }

  onMounted(() => window.addEventListener('pointermove', onPointer))
  onUnmounted(() => window.removeEventListener('pointermove', onPointer))

  return { x, y }
}
```

### useDamp (Lag/Smoothing)

```javascript
// composables/useDamp.js
export function useDamp(source, options = {}) {
  const { speed = 0.1 } = options
  const damped = ref(source.value)

  useRaf(() => {
    damped.value += (source.value - damped.value) * speed
  })

  return damped
}
```

---

## 四、Card Hover — Clip-Path Mask Reveal

卡片 hover 时，用 clip-path 椭圆展开揭示媒体内容：

```vue
<template>
  <div class="bento__card"
       @mouseenter="onHover(true)"
       @mouseleave="onHover(false)"
       :style="cardStyle">
    <!-- 背景层 (始终可见) -->
    <div class="bento__card-bg">
      <img :src="data.bgImage" />
    </div>
    <!-- 揭示层 (hover 时 clip-path 展开) -->
    <div class="bento__card-reveal" ref="revealLayer">
      <video :src="data.hoverVideo" muted loop />
    </div>
    <!-- 内容 (标题 + 描述) -->
    <div class="bento__card-content">
      <h3>{{ data.title }}</h3>
      <p>{{ data.description }}</p>
    </div>
  </div>
</template>

<script setup>
const revealLayer = ref(null)

function onHover(entering) {
  gsap.to(revealLayer.value, {
    clipPath: entering
      ? 'ellipse(100% 100% at 50% 50%)'   // 展开
      : 'ellipse(0% 0% at 50% 0%)',       // 收缩到顶部
    duration: 0.5,
    ease: 'power3.out',
  })
}
</script>
```

### CSS

```css
.bento__card-reveal {
  position: absolute;
  inset: 0;
  clip-path: ellipse(0% 0% at 50% 0%);
  transition: clip-path 0.5s var(--cubic-default);
  will-change: clip-path;
}

.bento__card:hover .bento__card-reveal {
  clip-path: ellipse(100% 100% at 50% 50%);
}
```

---

## 五、Sting 装饰背景

Bento 区域包含一个名为 `Sting` 的装饰性背景组件（波浪曲线/网格图形）：

```vue
<template>
  <div class="sting" ref="stingRef">
    <svg viewBox="0 0 1440 900" fill="none">
      <path d="M...C..." class="sting__path" />
    </svg>
  </div>
</template>

<script setup>
// useCssClipMask 控制 sting 的可见性
const { clipPath } = useCssClipMask(stingRef, {
  initialPath: 'inset(0 0 100% 0)',
  finalPath: 'inset(0 0 0% 0)',
})

useBaseScrollTrigger(stingRef, {
  start: 'top 70%',
  end: 'top 20%',
  scrub: true,
  onUpdate: ({ progress }) => clipPath.value = progress,
})
</script>
```
