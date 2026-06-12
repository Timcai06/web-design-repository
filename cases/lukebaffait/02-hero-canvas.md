# WebGL Hero + 帧序列穿越

源码: `hero-project.js` + `core-renderer.js` + `index.js:55-58,262-400`

---

## 一、WebGL CoreRenderer 引擎

### 启动入口

`index.js:55-58`:

```js
function startShader() {
  const projectData = window._heroProjectData;   // 来自 hero-project.js
  const blob = new Blob([JSON.stringify(projectData)], { type: 'application/json' });
  const blobUrl = URL.createObjectURL(blob);
  const container = document.getElementById('hero-canvas');
  container.setAttribute('data-cr-project-src', blobUrl);
  CoreRenderer.init().then(() => {
    URL.revokeObjectURL(blobUrl);
    // 初始化后模拟鼠标事件
    const e = new MouseEvent('mousemove', { clientX: lastMouseX, clientY: lastMouseY });
    window.dispatchEvent(e);
  });
}
```

**数据加载方式**：项目数据通过 `Blob URL` 注入 DOM 属性，`CoreRenderer` 读取后编译着色器。

### 4 层着色器管线

来自 `hero-project.js` 的 `window._heroProjectData.layers`:

| 层 | `layerId` | `type` | 功能 |
|---|---|---|---|
| 1 | `gradient` | `effect/gradient` | 背景渐变 + UV 旋转，`mouseMomentum: 0` |
| 2 | `image` | `image` | 背景图 + **Vertex Shader 视差倾斜** |
| 3 | `flow_field` | `effect/flowField` | **Perlin 噪声流场扭曲**，`mouseMomentum: 1` |
| 4 | `blur` | `effect/blur` | **高斯模糊** (36 tap, 4 passes + deband) |

#### 层 2: 图片视差倾斜 (Vertex Shader)

来自 `hero-project.js` 的 `compiledVertexShaders[1]`:

```glsl
#version 300 es
in vec3 aVertexPosition;
in vec2 aTextureCoord;
uniform vec2 uMousePos;
out vec2 vTextureCoord;
out vec3 vVertexPosition;

void main() {
  float angleX = uMousePos.y * 0.5 - 0.25;            // 鼠标 Y 控制 X 轴旋转
  float angleY = (1.0 - uMousePos.x) * 0.5 - 0.25;    // 鼠标 X 控制 Y 轴旋转
  mat4 rotateX = mat4(...);
  mat4 rotateY = mat4(...);
  mat4 rotationMatrix = rotateX * rotateY;
  gl_Position = uPMatrix * uMVMatrix * vec4(aVertexPosition, 1.0);
  vVertexPosition = (rotationMatrix * vec4(aVertexPosition, 1.0)).xyz;
  vTextureCoord = vec4(aTextureCoord, 0.0, 1.0).xy;
}
```

**原理**：鼠标位置 `[0,1]` 映射到旋转角度 `[-0.25, 0.25]` 弧度。顶点经过旋转矩阵后再计算位置，产生 3D 视差倾斜效果。

#### 层 3: Perlin 噪声流场 (Fragment Shader)

```glsl
// 核心函数
vec2 distortUV(vec2 uv) {
  // ... 基于鼠标位置计算影响区域 ...
  float dist = ease(1, max(0., 1. - length(st * aspectVec - mPos * aspectVec) * 4.));
  float amt = (1.0000 * 2.0) * 0.01 * dist;
  
  for (int i = 0; i < MAX_ITERATIONS; i++) {    // 8 次迭代
    float perlin = perlin_noise(vec3((scaled - 0.5) * freq, t)) - 0.5;
    float ang = perlin * rad;                     // 噪声值 → 偏移角度
    st += vec2(cos(ang), sin(ang)) * amt;         // 按角度偏移 UV
  }
  return mix(uv, clamp(st, 0., 1.), 0.5100);
}
```

**实现细节**：
- `perlin_noise` 使用 3D 哈希函数计算值噪声
- 鼠标附近影响更大 (`dist` 衰减)
- 8 次迭代累积偏移，产生流体扭曲效果
- `uTime * 0.0166` 驱动持续动画

#### 层 4: 高斯模糊 (4 Passes)

- 36 tap 高斯核（预计算权重存于 switch-case）
- Pass 1: 水平模糊 (downSample: 0.25)
- Pass 2: 垂直模糊 (downSample: 0.5)
- Pass 3: 水平模糊 (downSample: 0.5)
- Pass 4: 垂直模糊 + **deband 抖动**
- `mouseMomentum: 1` — 模糊半径随鼠标位置变化（径向模糊效果）

**deband 函数**：
```glsl
// PCG 随机数生成器
uvec2 pcg2d(uvec2 v) {
  v = v * 1664525u + 1013904223u;
  v.x += v.y * v.y * 1664525u + 1013904223u;
  v.y += v.x * v.x * 1664525u + 1013904223u;
  return v;
}
float deband() {
  return (randFibo(gl_FragCoord.xy) - 0.5) / 255.0;
}
```
每像素加 ±1/255 随机值，消除 8-bit 量化导致的色带。

### CoreRenderer 内部结构

从 `core-renderer.js` 解构出这些核心类：

| 类 | 功能 |
|---|---|
| `class G` | 主渲染器，WebGL 上下文管理 |
| `class Q` | 帧缓冲管理 (FBO) |
| `class Z` | 着色器程序编译与链接 |
| `class ee` | 纹理管理 (sourceImage, bgTexture) |
| `class ie` | 几何体 (quad 顶点 + UV) |
| `class Y` | 均匀变量绑定系统 |

---

## 二、341 帧 Canvas 序列

### 帧系统初始化

`index.js:262-345`:

```js
const FRAME_DIR = 'assets/images/hero%20sequence/';
const FRAME_EXT = '.jpg';
const FRAME_PAD = 4;
const TOTAL_FRAMES = 341;
const FRAME_CACHE_KEY = '20260416-r2';    // 缓存版本控制
const frameUrl = n => `${FRAME_DIR}${String(n).padStart(FRAME_PAD, '0')}${FRAME_EXT}?v=${FRAME_CACHE_KEY}`;
// => "assets/images/hero%20sequence/0001.jpg?v=20260416-r2"
```

### 分批加载策略

```js
async function loadFirstBatch() {
  // 1. 首先加载第 1 帧（必须有）
  const first = await probeWithRetry(1, 3);
  frames[0] = first;
  resizeCanvas();
  drawFrame(0);

  // 2. 并行加载 2-11 帧（测速用）
  const SPEED_BATCH = 10;
  await Promise.all(batchNums.map(async n => {
    const img = await probeWithRetry(n, 2);
    if (img) frames[n - 1] = img;
  }));

  // 3. 根据加载耗时决定帧采样率
  const elapsed = performance.now() - t0;
  return elapsed > 4000 ? 3 : elapsed > 2000 ? 2 : 1;
}

async function loadRemainingFrames(skip) {
  // 按 skip 间隔加载后续帧（慢设备跳帧）
  // 并发数：慢设备 2，快设备 6
  const CONCURRENCY = isSlowHardware ? 2 : 6;
}
```

### Canvas 绘制

```js
function drawFrame(i) {
  if (i === drawnIdx) return;
  if (isSlowHardware) {
    const now = performance.now();
    if (now - _lastDrawMs < 32) return;    // ~30fps 上限
    _lastDrawMs = now;
  }
  const img = frames[i];
  const cw = canvas.width, ch = canvas.height;
  const iw = img.naturalWidth, ih = img.naturalHeight;
  const s = Math.max(cw / iw, ch / ih);     // cover 模式
  const dw = iw * s, dh = ih * s;
  ctx.clearRect(0, 0, cw, ch);
  ctx.drawImage(img, (cw - dw) * 0.5, (ch - dh) * 0.5, dw, dh);
  drawnIdx = i;
}
```

### Scroll → Frame 映射

`index.js:355-400`:

```js
ScrollTrigger.create({
  trigger: '#scroll-wrap',           // 400vh
  start: 'top top',
  end: 'bottom bottom',
  scrub: 0.5,
  animation: scrollTl,
  onUpdate: (self) => {
    const p = self.progress;
    // Phase 1: 0% → 30% (名字淡出，图片放大)
    if (p < REVEAL_PHASE_START) { drawFrameAtProgress(0); return; }
    // Phase 2: 30% → 100% (帧序列 0% → 82%)
    const phase2 = Math.min(1, Math.max(0, (p - REVEAL_PHASE_START) / REVEAL_PHASE_DURATION));
    drawFrameAtProgress(phase2 * FRAME_PROGRESS_AT_EXIT_START);
  },
});

// 退出阶段（进入 about 区域）
ScrollTrigger.create({
  trigger: '#section-after',
  start: 'top bottom',
  end: 'top top',
  scrub: true,
  animation: exitTl,
  onUpdate: (self) => {
    const exitFrameProgress = FRAME_PROGRESS_AT_EXIT_START + (self.progress * (1 - FRAME_PROGRESS_AT_EXIT_START));
    drawFrameAtProgress(exitFrameProgress);
  },
});
```

**三个阶段**：
1. **名字退出** (0-30%)：`exitLeft: -55vw` / `exitRight: 55vw` 向两侧分离消散
2. **帧序列展示** (30%-100%)：341 帧的 0%→82% 映射到滚动 30%-100%
3. **Overlay 退出** (进入 About)：帧序列 82%→100% + `blur(16px)` + 短语文字 stagger 淡出

### 短语文字逐字揭示

```js
const phraseEl = document.getElementById('reveal-phrase');
phraseEl.innerHTML = [...phraseEl.textContent].map(ch =>
  `<span class="rp-char" style="display:inline-block;">${ch === ' ' ? '\u00a0' : ch}</span>`
).join('');

scrollTl.to(phraseChars, {
  opacity: 1,
  filter: isMobile ? undefined : 'blur(0px)',
  duration: 0.06,
  ease: 'none',
  stagger: { each: 0.007, from: 'start' },    // 每字 7ms 间隔
}, 0.62);
```
