# Zentry — 技术栈深度分析

---

## 一、构建管线

### Nuxt 3 + Vue 3

```
zentry.com
├── .nuxt/                    # Nuxt 构建缓存
├── pages/
│   ├── index.vue             # 首页
│   ├── about.vue             # About 页
│   └── contact.vue           # Contact 页
├── components/
│   ├── Hero.vue
│   ├── NavBar.vue
│   ├── Bento.vue
│   ├── BentoCard.vue
│   ├── SplineComponent.vue
│   ├── ScrollyVideo.vue
│   ├── Intro.vue
│   ├── Nexus.vue
│   ├── Partners.vue
│   ├── ProductGrid.vue
│   ├── WhoWeAre.vue
│   ├── Sting.vue
│   ├── InkTrapText.vue
│   ├── Caption.vue
│   ├── Title.vue
│   ├── PageTransiton.vue
│   ├── BtnMain.vue
│   ├── BtnRound.vue
│   ├── Footer.vue
│   └── ThemeTrigger.vue
├── composables/
│   ├── useSplitText.js
│   ├── useCssClipMask.js
│   ├── useDamp.js
│   ├── usePerspectiveTransform.js
│   ├── usePerspectiveFadeEntry.js
│   ├── useBaseScrollTrigger.js
│   ├── useTheme.js
│   ├── usePointer.js
│   ├── usePointerInElement.js
│   ├── useWindowPointer.js
│   ├── useWindowPointerNorm.js
│   ├── useOnScroll.js
│   ├── useRaf.js
│   ├── useRafBool.js
│   ├── useThresholdTrigger.js
│   ├── usePropEffects.js
│   └── useMediaMeta.js
├── layouts/
│   └── default.vue
├── plugins/
│   ├── gsap.client.js         # GSAP 注册
│   ├── lenis.client.js        # Lenis 平滑滚动
│   └── storyblok.js           # Storyblok CMS
├── stores/                    # Pinia (或 useState)
├── public/
│   ├── videos/
│   ├── images/
│   └── favicon/
├── nuxt.config.ts
└── package.json
```

### 构建产物 (Nuxt 3)

```
/_nuxt/
├── entry.DOO_LDJZ.js          # 入口 bundle
├── default.DYn5k_Ji.js       # 默认 layout
├── three.module.fMA-iDlM.js  # Three.js (代码分割)
├── lenis.CYDLGOJa.js         # Lenis (代码分割)
├── Hero.tu0dqTIj.js          # Hero 组件 (懒加载)
├── Bento.CgYh9Gem.js         # Bento 组件
├── SplineComponent.BpKTdwBe.js
├── ScrollyVideo.CuZK9059.js
├── ...
└── *.css                     # 每个组件独立的 CSS
```

**注意**：Nuxt 3 自动代码分割，Three.js/Spline 只在包含它们的页面/组件中加载。

---

## 二、与 Storyblok CMS 集成

Zentry 使用 **Storyblok** 作为 headless CMS：

```javascript
// plugins/storyblok.js
import StoryblokVue from '@storyblok/vue'

export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.use(StoryblokVue, {
    accessToken: process.env.STORYBLOK_TOKEN,
    bridge: process.dev,
    apiOptions: {
      region: 'us',
    },
  })
})
```

内容由 Storyblok 管理，通过 Nuxt 的 `useStoryblok` composable 获取：

```vue
<script setup>
const story = await useStoryblok('home', { version: 'draft' })
</script>
```

---

## 三、Composables 系统详解

Resn 建立了一套可复用的 composables 库，这是他们"proprietary framework"的核心：

### 输入类

| Composable | 输入 | 输出 |
|---|---|---|
| `useWindowPointer` | 鼠标事件 | `{ x, y }` (viewport px) |
| `useWindowPointerNorm` | 鼠标事件 | `{ x, y }` (-1 to 1, 相对元素) |
| `usePointerInElement` | 鼠标 + 元素 ref | `{ isInside, x, y }` |
| `useOnScroll` | Lenis 事件 | `{ scrollY, velocity, isVisible, direction }` |
| `useViewportResize` | resize 事件 | `{ width, height }` |
| `useMediaMeta` | 媒体文件 | `{ duration, dimensions }` |

### 动画类

| Composable | 用途 |
|---|---|
| `useDamp` | 滞后/平滑跟随（lerp） |
| `useRaf` | requestAnimationFrame 循环管理 |
| `useRafBool` | RAF 布尔值（节流） |
| `useBaseScrollTrigger` | GSAP ScrollTrigger 封装 |
| `usePropEffects` | 响应式属性驱动的 GSAP 动画 |
| `useThresholdTrigger` | 阈值触发的回调 |
| `usePerspectiveFadeEntry` | 带 perspective 的渐入 |
| `usePerspectiveTransform` | 3D 透视倾斜 |

### 效果类

| Composable | 用途 |
|---|---|
| `useSplitText` | 字符串拆分为字符/单词 span |
| `useCssClipMask` | CSS clip-path 进度动画 |
| `useTheme` | 亮/暗主题切换 |
| `usePageState` | 页面过渡状态 |

### 框架

```javascript
// composables/useBaseScrollTrigger.js — 核心 ScrollTrigger 封装
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

export function useBaseScrollTrigger(el, options = {}) {
  let st = null

  onMounted(() => {
    st = ScrollTrigger.create({
      trigger: el.value,
      ...options,
    })
  })

  onUnmounted(() => {
    if (st) st.kill()
  })

  return st
}
```

---

## 四、性能策略

### 1. 无 WebGL 核心渲染

Resn 刻意选择 **DOM-only** 方案，原因：

> "Zentry wanted to avoid using it for rendering complex visual effects. Instead of relying on external 3-D graphic libraries, we focused on using memory-efficient DOM technologies to manipulate elements directly, ensuring fast load times and smooth transitions."

### 2. GPU 高效动画

```css
/* 只使用 transform + opacity，保证 GPU 合成 */
.bento__card {
  will-change: transform, opacity;
  transform: translateZ(0);   /* 提升为合成层 */
}
```

### 3. 内存管理

```javascript
// 页面切换时销毁 unused 元素
onUnmounted(() => {
  // ScrollTrigger 自动销毁
  // 视频元素释放
  if (videoRef.value) {
    videoRef.value.pause()
    videoRef.value.src = ''
  }
  // Three.js/Spline 释放
  if (splineApp) splineApp.dispose()
})
```

### 4. CSS clip-path 性能

> "Using the CSS clip-path property for masking effects proved highly performant and efficient."

CSS `clip-path` 在支持 GPU 加速的浏览器上通过合成器线程处理，不触发 layout/paint。

### 5. 代码分割

Nuxt 3 自动代码分割 + `definePageMeta` 中的懒加载：

```javascript
definePageMeta({
  loading: false,   // 不显示加载指示器
})
```

---

## 五、Resn vs Lusion — 设计哲学对比

| 维度 | Resn (Zentry) | Lusion (oryzo / lukebaffait) |
|---|---|---|
| **3D 路线** | **DOM-only** (CSS clip-path + JS) | **WebGL-heavy** (Three.js / CoreRenderer) |
| **框架** | Nuxt 3 + Vue 3 | Astro (oryzo) / 纯 HTML (lukebaffait) |
| **CMS** | Storyblok | 无 / Webflow |
| **动画** | GSAP + ScrollTrigger | GSAP + ScrollTrigger |
| **平滑滚动** | Lenis | Lenis |
| **3D 工具** | Spline (嵌入式) | Three.js (自建)、Houdini、CoreRenderer |
| **视频** | ScrollyVideo (帧映射) | Vimeo Player API |
| **文字动画** | `useSplitText` + GSAP | 自定义 splitToChars |
| **字体** | InkTrapText (自定义) | 系统字体 + Primer CSS |
| **性能核心** | CSS clip-path + transform | Canvas + will-change |
| **优势** | 更轻量、更快加载、更低内存 | 更丰富的视觉效果、无限创意可能 |
| **适合场景** | 品牌官网、营销页面 | 作品集、创意展示、游戏化体验 |

### 技术选择背后的原因

- **Resn** 的客户（Zentry 是 Web3 品牌）需要更广泛的设备兼容性、更快的加载速度、更好的 SEO
- **Lusion** 的客户（或自项目）追求视觉震撼，愿意为此承受更高的技术成本和性能开销
- **共同点**：GSAP + ScrollTrigger + Lenis 是两家都依赖的核心组合
