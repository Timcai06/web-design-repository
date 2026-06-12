# 无缝过渡系统 — Preloader / 片头视频 / WebGL 场景

Persepolis 最令人印象深刻的技术成就：**三种不同技术（CSS preloader → After Effects 视频 → WebGL 实时场景）之间的完美无缝过渡**。

---

## 一、过渡总览

```
Preloader (CSS/Canvas)       片头视频 (After Effects)       WebGL 场景 (实时)
┌──────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│                  │    │                      │    │                      │
│  Loading...      │──→ │ Cinematic Title      │──→ │  3D Persepolis       │
│  80%             │    │ Sequence              │    │  Gate of All Nations │
│                  │    │  (渲染好的视频)        │    │  (实时操作)          │
│                  │    │                      │    │                      │
└──────────────────┘    └──────────────────────┘    └──────────────────────┘
       CSS fade              Chroma Key Shader              WebGL
```

---

## 二、Preloader → 片头视频

```html
<div id="loader">
  <canvas id="loader-canvas" />  <!-- WebGL 加载动画 -->
  <div class="loader__progress">
    <span class="loader__bar" ref="loaderBar" />
    <span class="loader__text">Loading...</span>
  </div>
</div>

<div id="title-sequence" style="display:none">
  <video id="title-video" muted playsinline />
</div>
```

```javascript
// 资源加载进度
async function loadAssets(onProgress) {
  const assets = [
    fetchJSON('/data/persepolis.json'),
    loadVideo('/video/title.mp4'),
    loadAudio('/audio/ambient.mp3'),
    // ...
  ]

  let loaded = 0
  for (const asset of assets) {
    await asset
    loaded++
    onProgress(loaded / assets.length)
  }
}

// Preloader 动画
function startLoader() {
  const canvas = document.getElementById('loader-canvas')
  // WebGL 粒子/logo 动画

  loadAssets((progress) => {
    gsap.to(loaderBar, {
      scaleX: progress,
      duration: 0.3,
      ease: 'power2.out',
    })

    if (progress >= 1) {
      // Preloader 淡出 → 片头视频淡入
      gsap.to('#loader', {
        opacity: 0,
        duration: 0.5,
        onComplete: () => {
          document.getElementById('loader').style.display = 'none'
          document.getElementById('title-sequence').style.display = 'block'

          // 视频叠加与 WebGL 相同的 Post-Processing
          applyPostFXToVideo()
        },
      })
    }
  })
}
```

### Preloader WebGL 动画

```javascript
// preloader 使用 WebGL canvas 显示粒子动画或旋转图标
function initLoaderCanvas(canvas) {
  const gl = canvas.getContext('webgl2')
  // 简单的粒子系统或 logo 旋转
}
```

---

## 三、片头视频 → WebGL 场景 (Chroma Key)

这是最核心的技术挑战：**After Effects 渲染的片头视频 → 实时 WebGL 场景，中间没有剪切**。

### 思路

```
片头视频最后一帧:              WebGL 场景第一帧:
┌──────────────────────┐    ┌──────────────────────┐
│                      │    │                      │
│  背景: 波斯波利斯全景    │    │  3D 场景:            │
│  + 标题文字            │    │  Gate of All Nations │
│  + 粒子/光效           │    │                      │
│                      │    │  (通过 Chroma Key     │
│  视频中有一个"纯色区域"   │    │   替换为 3D 渲染)     │
│  作为 Chroma Key 标记   │    │                      │
└──────────────────────┘    └──────────────────────┘
         │                             │
         └──────────融合────────────────┘
              Chroma Key Shader
```

### Chroma Key WebGL Shader

通过自定义 WebGL shader 实现视频的色度键抠像：

```glsl
// fragment shader — Chroma Key
#version 300 es
precision highp float;

uniform sampler2D uVideoFrame;    // 片头视频
uniform sampler2D uSceneFrame;    // WebGL 场景
uniform float uTransition;        // 0 → 1 (全视频 → 全 WebGL)

in vec2 vUV;
out vec4 fragColor;

void main() {
  vec4 videoColor = texture(uVideoFrame, vUV);
  vec4 sceneColor = texture(uSceneFrame, vUV);

  // Chroma Key: 检测纯色背景 (关键色: #00FF00 或特定 RGB 范围)
  vec3 keyColor = vec3(0.0, 1.0, 0.0);  // 绿色
  float diff = distance(videoColor.rgb, keyColor);

  float threshold = 0.15;   // 色差阈值
  float smoothness = 0.05;  // 边缘羽化

  float alpha = smoothstep(threshold - smoothness, threshold + smoothness, diff);

  // 过渡混合: Chroma Key + 全局淡入淡出
  float mixFactor = mix(alpha, 1.0, uTransition);

  vec4 finalColor = mix(keyColor, sceneColor, mixFactor);

  // Post-Processing (与全站一致)
  finalColor = applyBloom(finalColor);
  finalColor = applyVignette(finalColor);
  finalColor = applyGrain(finalColor);

  fragColor = finalColor;
}
```

### 过渡时间线

```javascript
// 片头视频 → WebGL 过渡控制
const titleVideo = document.getElementById('title-video')
const transition = { value: 0 }

// 视频播放时同步渲染 WebGL 场景
function animateTransition() {
  const shader = chromaKeyShader

  // 视频播放时 transition = 0 (全视频)
  // 视频最后 2 秒：transition 0 → 1

  const videoDuration = titleVideo.duration
  const transitionStart = videoDuration - 2  // 最后 2 秒开始过渡

  titleVideo.addEventListener('timeupdate', () => {
    if (titleVideo.currentTime >= transitionStart) {
      // 计算 transition 进度
      transition.value = (titleVideo.currentTime - transitionStart) / 2
      shader.setUniform('uTransition', transition.value)

      // 同时开始渲染 WebGL 场景
      if (!sceneStarted) {
        startWebGLScene()
        sceneStarted = true
      }
    }
  })

  // 视频结束 → 完全切换到 WebGL
  titleVideo.addEventListener('ended', () => {
    document.getElementById('title-sequence').style.display = 'none'
    transition.value = 1
    shader.setUniform('uTransition', 1)
    enableUserControl()  // 启用用户交互
  })
}
```

---

## 四、Post-Processing 统一

关键设计决策：**所有场景（Preloader、片头视频、WebGL 实时场景）使用相同的 Post-Processing 效果**。

```javascript
// 将 Post-Processing 应用到视频
function applyPostFXToVideo() {
  // 使用 WebGL 将视频帧通过 Post-Processing 管线渲染
  // 而不是直接显示 <video> 元素

  function videoRenderLoop() {
    // 1. 将视频帧上传到纹理
    gl.bindTexture(gl.TEXTURE_2D, videoTexture)
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, titleVideo)

    // 2. 应用 Post-Processing
    postFX.render(videoTexture)

    // 3. Chroma Key 融合
    chromaKeyShader.render(videoTexture, sceneTexture, transition.value)

    requestAnimationFrame(videoRenderLoop)
  }
}

// WebGL 场景也走同一套 Post-Processing
function startWebGLScene() {
  function sceneRenderLoop() {
    renderScene()
    postFX.render(sceneTexture)

    // 如果还在过渡期，需要与视频融合
    if (transition.value < 1) {
      chromaKeyShader.render(videoTexture, sceneTexture, transition.value)
    }

    requestAnimationFrame(sceneRenderLoop)
  }
}
```

统一 Post-Processing 的效果：

| 效果 | 作用 |
|---|---|
| **Vignette** | 暗角让视线聚焦中心 |
| **Bloom** | 辉光给场景电影感 |
| **Film Grain** | 胶片颗粒统一"材质"感 |

**视觉结果**：用户完全感觉不到"视频"和"3D"的边界——三者看起来是同一个画面。

---

## 五、房间转场 — 雾化过渡

房间之间的切换使用**渐隐雾效**保持沉浸感：

```javascript
function transitionToRoom(roomId) {
  const duration = 1.5  // 1.5s 过渡

  // 1. 雾效渐浓 (视觉遮挡)
  gsap.to(fogUniforms, {
    density: 0.8,
    duration: duration / 2,
    ease: 'power2.in',
    onComplete: () => {
      // 2. 切换房间数据
      loadRoom(roomId)

      // 3. 雾效渐消
      gsap.to(fogUniforms, {
        density: 0.0,
        duration: duration / 2,
        ease: 'power2.out',
      })
    },
  })
}
```

```glsl
// fragment shader — 雾效混合
uniform float uFogDensity;
uniform vec3 uFogColor;       // 与场景主色调匹配

void main() {
  vec4 color = texture(uScene, vUV);
  float depth = gl_FragCoord.z / gl_FragCoord.w;
  float fogFactor = 1.0 - exp(-uFogDensity * uFogDensity * depth * depth);
  fragColor = mix(color, vec4(uFogColor, 1.0), fogFactor);
}
```

---

## 六、过渡系统架构图

```
                   时间轴
         ├──────┬──────┼──────┬──────┤
         │  ①   │  ②   │  ③   │  ④   │
         │      │      │      │      │
  ① Preloader (CSS/WebGL)
         │
         └── CSS opacity: 0 ──→
         │
  ② 片头视频 (After Effects)
         │                     Chroma Key 开始
         └─────────────────────┼────────────────
                               │
  ③ 视频 + WebGL 共存的过渡期
                               │
                               └──── Chroma Key 完成
                                     │
  ④ WebGL 场景 (实时渲染)               │
                                     │
                                     ⑤ 房间切换 (雾效)
```
