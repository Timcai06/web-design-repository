# Framework Adaptation Guide

本仓库所有效果的原版实现分散在零框架（vanilla JS）、Vue、Webflow、Astro 中。以下指南说明如何将每个核心模式移植到主流框架。

---

## 一、通用原则

### 动效生命周期

```
Component Mount → 注册 ScrollTrigger / Observer / Event
         ↓
Component Update → re-calc 触发器位置 (layout shift 后)
         ↓
Component Unmount → 清理 ScrollTrigger / RAF / Observer
```

**框架无关的核心约束：**
1. 动效必须在 mount 后注册（DOM 已渲染）
2. 必须在 unmount 前清理（防止内存泄漏和 ghost triggers）
3. 如果内容异步加载，动效要在内容加载完成后注册

### ScrollTrigger 批量清理

```javascript
// GSAP 建议：为每个页面/组件分配一个 ID，统一清理
ScrollTrigger.getAll().forEach(st => {
  if (st.vars.id === 'my-component') st.kill()
})
```

---

## 二、React (Next.js)

### 模式 1: 通用 Scroll Reveal Hook

```tsx
// hooks/useScrollReveal.ts
'use client'
import { useEffect, useRef } from 'react'

export function useScrollReveal<T extends HTMLElement>(
  options?: { threshold?: number; rootMargin?: string }
) {
  const ref = useRef<T>(null)

  useEffect(() => {
    const el = ref.current
    if (!el) return

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          el.classList.add('revealed')
          observer.unobserve(el)
        }
      },
      { threshold: options?.threshold ?? 0.2,
        rootMargin: options?.rootMargin ?? '0px' }
    )
    observer.observe(el)
    return () => observer.disconnect()
  }, [])

  return ref
}
```

**使用：**
```tsx
function Section() {
  const ref = useScrollReveal<HTMLDivElement>({ threshold: 0.3 })
  return <div ref={ref} className="opacity-0 translate-y-8 transition-all
    duration-700 data-[revealed]:opacity-100 data-[revealed]:translate-y-0" />
}
```

### 模式 2: Perspective Tilt Card

```tsx
// components/TiltCard.tsx
'use client'
import { useRef, useCallback } from 'react'

export function TiltCard({ children }: { children: React.ReactNode }) {
  const cardRef = useRef<HTMLDivElement>(null)
  const state = useRef({ targetX: 0, targetY: 0, currentX: 0, currentY: 0 })
  const rafId = useRef<number>()

  const animate = useCallback(() => {
    const s = state.current
    s.currentX += (s.targetX - s.currentX) * 0.12
    s.currentY += (s.targetY - s.currentY) * 0.12
    if (cardRef.current) {
      cardRef.current.style.transform =
        `perspective(800px) rotateX(${s.currentX}deg) rotateY(${s.currentY}deg)`
    }
    rafId.current = requestAnimationFrame(animate)
  }, [])

  const handleMouse = (e: React.MouseEvent) => {
    const rect = cardRef.current!.getBoundingClientRect()
    const x = (e.clientX - rect.left) / rect.width
    const y = (e.clientY - rect.top) / rect.height
    state.current.targetY = (x - 0.5) * 16
    state.current.targetX = (0.5 - y) * 10
  }

  const handleLeave = () => {
    state.current.targetX = 0
    state.current.targetY = 0
  }

  return (
    <div ref={cardRef}
      onMouseMove={handleMouse}
      onMouseLeave={handleLeave}
      style={{ transformStyle: 'preserve-3d', cursor: 'pointer' }}>
      {children}
    </div>
  )
}
```

### 模式 3: Scroll-Driven GSAP (Next.js 客户端)

```tsx
// components/ScrollReveal.tsx
'use client'
import { useEffect, useRef } from 'react'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

export function ScrollReveal({ children, className }: {
  children: React.ReactNode
  className?: string
}) {
  const ref = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const el = ref.current
    if (!el) return

    const ctx = gsap.context(() => {
      gsap.from(el.children, {
        y: 60,
        opacity: 0,
        stagger: 0.1,
        duration: 0.8,
        ease: 'power3.out',
        scrollTrigger: {
          trigger: el,
          start: 'top 85%',
          toggleActions: 'play none none none',
          id: 'ScrollReveal', // 方便清理
        },
      })
    })

    return () => ctx.revert() // GSAP context 自动清理
  }, [])

  return <div ref={ref} className={className}>{children}</div>
}
```

---

## 三、Vue (Nuxt 3)

### 模式 1: useScrollReveal Composable

```typescript
// composables/useScrollReveal.ts
export function useScrollReveal(threshold = 0.2) {
  const el = ref<HTMLElement | null>(null)
  const isRevealed = ref(false)

  onMounted(() => {
    if (!el.value) return
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          isRevealed.value = true
          observer.unobserve(entry.target)
        }
      },
      { threshold }
    )
    observer.observe(el.value)
    onUnmounted(() => observer.disconnect())
  })

  return { el, isRevealed }
}
```

**使用 (类似 Zentry 的 composable 模式):**
```vue
<script setup>
const { el, isRevealed } = useScrollReveal(0.3)
</script>

<template>
  <div ref="el" :class="['reveal-section', { revealed: isRevealed }]">
    <slot />
  </div>
</template>

<style scoped>
.reveal-section { opacity: 0; transform: translateY(40px); transition: all 0.8s ease; }
.reveal-section.revealed { opacity: 1; transform: translateY(0); }
</style>
```

### 模式 2: Zentry 式 Portal Mask

```vue
<!-- components/PortalMask.vue -->
<script setup>
const portalRef = ref(null)
const progress = ref(0)

useBaseScrollTrigger(portalRef, {
  start: 'top top',
  end: 'bottom top',
  scrub: true,
  onUpdate: (self) => { progress.value = self.progress },
})

const clipPathStyle = computed(() => {
  // 菱形从中心展开
  const p = progress.value
  const center = 50 - p * 50  // 50 → 0
  return {
    clipPath: `polygon(${center}% ${center}%, ${100 - center}% ${center}%,
                       ${100 - center}% ${100 - center}%, ${center}% ${100 - center}%)`,
  }
})
</script>

<template>
  <div ref="portalRef" class="portal-mask" :style="clipPathStyle">
    <slot />
  </div>
</template>

<style scoped>
.portal-mask {
  position: absolute; inset: 0;
  will-change: clip-path;
  z-index: 2;
}
</style>
```

### 模式 3: Lenis + Nuxt 集成

```typescript
// plugins/lenis.client.ts
import Lenis from 'lenis'

export default defineNuxtPlugin(() => {
  const lenis = new Lenis({
    duration: 1.2,
    easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
    orientation: 'vertical',
    smoothWheel: true,
  })

  function raf(time: number) {
    lenis.raf(time)
    requestAnimationFrame(raf)
  }
  requestAnimationFrame(raf)

  return {
    provide: { lenis }
  }
})
```

---

## 四、Astro

Oryzo 使用 Astro。关键优势：**所有 scroll 效果在客户端 JS 中实现，Astro 只负责构建 HTML。**

```astro
---
// Astro 是静态生成，所有 JS 在客户端组件中
import Lenis from 'lenis'
---
<html>
<body>
  <div id="app" />
  <script>
    // 客户端 JS — 与 landsite 或 vanilla JS 项目写法完全一致
    import { gsap } from 'gsap'
    import { ScrollTrigger } from 'gsap/ScrollTrigger'
    gsap.registerPlugin(ScrollTrigger)

    // ... 所有 vanilla JS 代码
  </script>
</body>
</html>
```

**Astro + Framework 混合 (Oryzo 方式):**
- 用 Astro 做构建 + routing
- 用 `client:load` 或 `client:visible` 加载交互组件
- Three.js / Rive / GSAP 都在客户端组件中运行

---

## 五、Webflow

landonorris 使用 Webflow + 自定义 JS 嵌入。

**Webflow 集成要点:**
1. 在 Webflow Designer 中添加 `data-*` 属性到元素
2. 在页面 `<head>` 或 footer 中嵌入自定义 JS
3. JS 通过 `data-*` 属性查找元素并绑定效果

```html
<!-- Webflow 自定义代码嵌入 -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12/dist/ScrollTrigger.min.js"></script>
<script>
// 与 landonorris 相同模式：data 属性驱动
document.querySelectorAll('[data-reveal]').forEach(el => {
  ScrollTrigger.create({
    trigger: el,
    start: 'top 85%',
    onEnter: () => el.classList.add('is-visible'),
  })
})
</script>
```

---

## 六、GSAP + ScrollTrigger — 框架无关的精简用法

在所有框架中，这 3 种模式覆盖 80% 的需求：

### Scrub（精确跟随）

```javascript
ScrollTrigger.create({
  trigger: '.element',
  start: 'top bottom',
  end: 'bottom top',
  scrub: 1,           // 0.5-2: 数字越大跟随越慢
  onUpdate: (self) => {
    element.style.transform = `scale(${1 + self.progress * 0.3})`
  }
})
```

### Play-on-Enter（触发生效）

```javascript
ScrollTrigger.create({
  trigger: '.element',
  start: 'top 85%',
  toggleActions: 'play none none none',
  onEnter: () => element.classList.add('active'),
})
```

### Toggle（状态切换）

```javascript
ScrollTrigger.create({
  trigger: '.trigger-point',
  start: 'top center',
  onEnter: () => nav.classList.add('light-theme'),
  onLeaveBack: () => nav.classList.remove('light-theme'),
})
```

---

## 七、性能清单（框架无关）

| 注意点 | 原因 | 做法 |
|--------|------|------|
| 清理 ScrollTrigger | SPA 路由切换不清理会导致 ghost triggers | `ScrollTrigger.getAll().filter(...).kill()` |
| 清理 RAF | unmount 后动画继续运行 | `useEffect return` / `onUnmounted` 中 `cancelAnimationFrame` |
| IntersectionObserver 清理 | observer 回调在 unmount 后仍可能触发 | `observer.disconnect()` |
| 图片/视频加载后重新触发 | 动效注册时图片未加载完毕 → 位置算错 | `imagesLoaded` 或 `load` 事件后 `ScrollTrigger.refresh()` |
| 响应式 breakpoints | 桌面和移动端动效参数不同 | `ScrollTrigger.matchMedia()` + GSAP `matchMedia()` |
| Redux/State 变化 | 状态变化新增/删除 DOM 节点 → 动效未注册 | 新增节点后手动调用 init 函数 |
