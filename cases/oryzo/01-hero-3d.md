# Three.js Hero 3D 杯垫 + Preloader

---

## 一、Render Tree

```html
<body>
  <canvas id="canvas">           <!-- 全屏 Three.js canvas，z-index 底层 -->
  <div id="ui">
    <div id="site-content">
      <div id="home" class="page">
        <section id="hero">
          <div class="section__inner">
            <div class="hero-tagline">Made for mugs. Built for tables.</div>
            <div class="hero-logo">ORYZO</div>
            <div class="hero-video">
              <img src="video_thumb.webp" />
              <button>PLAY</button>
            </div>
          </div>
        </section>
        <!-- 后续 sections... -->
      </div>
    </div>
  </div>
  <div id="preloader">
    <canvas id="preloader-canvas" />   <!-- 全屏 WebGL preloader -->
  </div>
</body>
```

关键结构：

```
#canvas (Three.js 全屏)          → z-index: 0 (背景层)
  ├─ 杯垫 3D 模型 (OBJ/GLTF)
  ├─ 环境光照 (HDR env map)
  └─ 粒子/装饰

#ui (所有 HTML 内容)              → z-index: 1 (覆盖层)
  ├─ #site-content (滚动内容)
  ├─ #site-header (导航)
  ├─ #scroll-indicator (进度条)
  └─ #preloader (启动画面)
```

---

## 二、Three.js 场景设置

### 场景初始化

```javascript
import * as THREE from 'three'
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

const canvas = document.getElementById('canvas')
const scene = new THREE.Scene()

const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100)
camera.position.set(0, 0.5, 3)

const renderer = new THREE.WebGLRenderer({
  canvas,
  alpha: true,
  antialias: true,
})
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
renderer.setSize(window.innerWidth, window.innerHeight)
renderer.toneMapping = THREE.ACESFilmicToneMapping
renderer.toneMappingExposure = 1.2
```

### 杯垫模型加载

```javascript
const loader = new GLTFLoader()
let coaster

loader.load('/models/oryzo-coaster.glb', (gltf) => {
  coaster = gltf.scene

  // 材质调整
  coaster.traverse(child => {
    if (child.isMesh) {
      // cork 材质 - 粗糙表面
      if (child.name.includes('cork')) {
        child.material.roughness = 0.9
        child.material.metalness = 0
      }
      // 标签/印刷 - 光滑
      if (child.name.includes('label')) {
        child.material.roughness = 0.2
        child.material.metalness = 0.1
      }
    }
  })

  scene.add(coaster)
})
```

### 光照

```javascript
// HDR 环境贴图
const pmremGenerator = new THREE.PMREMGenerator(renderer)
const hdrLoader = new THREE.RGBELoader()
hdrLoader.load('/env/studio.hdr', (texture) => {
  const envMap = pmremGenerator.fromEquirectangular(texture).texture
  scene.environment = envMap
  texture.dispose()
  pmremGenerator.dispose()
})

// 主光
const mainLight = new THREE.DirectionalLight('#ffffff', 2)
mainLight.position.set(2, 3, 4)
scene.add(mainLight)

// 补光
const fillLight = new THREE.DirectionalLight('#ffd58c', 0.5)
fillLight.position.set(-2, 1, 0)
scene.add(fillLight)

// 边缘光 (rim light)
const rimLight = new THREE.DirectionalLight('#ffffff', 0.8)
rimLight.position.set(0, -1, -3)
scene.add(rimLight)
```

---

## 三、Scroll-Driven 3D 交互

核心：**Three.js 模型的状态随滚动进度变化**，而非 camera 轨道控制。

```javascript
// Lenis 平滑滚动
const lenis = new Lenis()
lenis.on('scroll', (e) => {
  const progress = e.animatedScroll / (document.documentElement.scrollHeight - window.innerHeight)

  if (coaster) {
    // 1. 模型绕 Y 轴旋转 (0° → 360°)
    coaster.rotation.y = progress * Math.PI * 2

    // 2. 模型轻微上下浮动
    coaster.position.y = Math.sin(progress * Math.PI * 4) * 0.05

    // 3. 模型缩放 (开场时从 0.8 → 1.0)
    const scaleProgress = Math.min(progress * 3, 1)  // 前 33% 滚动完成缩放
    coaster.scale.setScalar(0.8 + scaleProgress * 0.2)
  }

  // 4. Camera 轻微推拉
  camera.position.z = 3 - progress * 0.5  // 从 z=3 到 z=2.5
  camera.lookAt(0, 0, 0)
})

// 动画循环
function animate(time) {
  lenis.raf(time)
  renderer.render(scene, camera)
  requestAnimationFrame(animate)
}
animate()
```

### 滚动映射总结

```
滚动进度     0%                33%                66%                100%
          ┌──────┐          ┌──────┐           ┌──────┐           ┌──────┐
旋转      │ 0°   │───────→  │ 120° │──────────→│ 240° │──────────→│ 360° │
缩放      │ 0.8  │═══════→  │ 1.0  │           │ 1.0  │           │ 1.0  │
          │      │ (前33%   │      │           │      │           │      │
          │      │  完成)   │      │           │      │           │      │
Camera Z  │ 3.0  │────────→ │ 2.8  │──────────→│ 2.6  │──────────→│ 2.5  │
悬浮      │ 0    │  ～～～  │ ～   │  ～～～～～ │ ～   │  ～～～～～ │ 0    │
```

---

## 四、Preloader

### 结构

```html
<div id="preloader">
  <canvas id="preloader-canvas" />
  <div class="preloader__text">LOADING</div>
</div>
```

```css
#preloader {
  position: fixed;
  inset: 0;
  z-index: 99999;
  background: #0a0a0a;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.8s ease;
}

#preloader.is-hidden {
  opacity: 0;
  pointer-events: none;
}
```

### 逻辑

```javascript
// 预加载关键资源
async function preload() {
  const preloader = document.getElementById('preloader')
  const canvas = document.getElementById('preloader-canvas')

  // 启动 preloader WebGL 动画 (粒子/logo 动画)
  initPreloaderAnimation(canvas)

  // 并行加载以下资源:
  await Promise.all([
    loadGLTF('/models/oryzo-coaster.glb'),
    loadHDR('/env/studio.hdr'),
    loadVideo('/video/product.mp4'),
    loadImages(),  // 所有 <picture> 中的图片
  ])

  // 最小展示时间 (确保 preloader 至少 1.5s)
  await new Promise(r => setTimeout(r, 1500))

  // 隐藏 preloader
  preloader.classList.add('is-hidden')
  setTimeout(() => preloader.remove(), 800)

  // 启动 Three.js 主场景
  initScene()
}
```

### Preloader 3D 动画

```javascript
function initPreloaderAnimation(canvas) {
  const gl = canvas.getContext('webgl') || canvas.getContext('webgl2')
  // 使用自定义 shader 画粒子/旋转环
  // 或者用 Three.js 的轻量场景
  const preloaderScene = new THREE.Scene()
  // ... 简单几何体旋转动画
}
```

---

## 五、Canvas 与 HTML 内容的层叠关系

```css
/* Three.js 背景 canvas */
#canvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 0;
  pointer-events: none;   /* 鼠标事件穿透到 HTML UI */
}

/* HTML UI 层 */
#ui {
  position: relative;
  z-index: 1;
  pointer-events: none;   /* 容器穿透, 子元素根据需要覆盖 */
}

#ui button,
#ui a,
#ui input,
#ui select {
  pointer-events: auto;   /* 交互元素可点击 */
}
```

---

## 六、视频 Overlay

```html
<div id="video-overlay" style="display:none">
  <div id="video-overlay__vimeo-video" />   <!-- Vimeo Player API -->
  <button id="video-overlay__mute-btn" />
  <div id="video-overlay-cursor" />          <!-- 自定义光标 -->
</div>
```

```javascript
import Player from '@vimeo/player'

const overlay = document.getElementById('video-overlay')
const player = new Player('video-overlay__vimeo-video', {
  id: vimeoId,
  autoplay: true,
  controls: false,
  muted: false,
})

document.querySelector('.hero-video button').addEventListener('click', () => {
  overlay.style.display = 'block'
  player.play()
})

overlay.addEventListener('click', () => {
  player.pause()
  overlay.style.display = 'none'
})
```
