# 自定义 WebGL 渲染器 + 性能优化

---

## 一、Media.Monks 自研 WebGL 框架

Media.Monks 使用自行研发的 WebGL 渲染框架（非 Three.js、非 Babylon.js），支持：

- WebGL 2.0 (利用 VAO、UBO、Instancing)
- Unity JSON 数据加载
- 光照贴图渲染
- Instanced 几何体批量渲染
- Post-Processing 管线（Bloom、Vignette、Film Grain）

### 初始化流程

```javascript
// WebGL 运行时入口
class PersepolisRenderer {
  constructor(canvasId) {
    this.canvas = document.getElementById(canvasId)
    this.gl = this.canvas.getContext('webgl2', {
      alpha: false,
      antialias: true,
      powerPreference: 'high-performance',
    })

    this.sceneData = null
    this.camera = new FixedPathCamera()
    this.postFX = new PostProcessing(this.gl)
  }

  async load(url) {
    // 1. 加载 Unity 导出的 JSON
    const response = await fetch(url)
    this.sceneData = await response.json()

    // 2. 创建几何体 buffers (VBO/IBO)
    this.meshes = this.sceneData.meshes.map(m => new MeshBuffer(this.gl, m))

    // 3. 创建实例化数据 buffers
    this.instanceBuffers = this.sceneData.instances.map(i =>
      new InstanceBuffer(this.gl, i)
    )

    // 4. 加载光照贴图纹理
    this.lightmaps = this.sceneData.lightmaps.map(lm =>
      textureFromBase64(this.gl, lm)
    )

    // 5. 初始化 Post-Processing
    this.postFX.init(this.sceneData.postProcess)

    // 6. 启动渲染循环
    this.animate()
  }
}
```

---

## 二、固定相机路径系统

### 路径插值

```javascript
class FixedPathCamera {
  constructor(data) {
    this.keyframes = data.keyframes    // [{time, position, rotation}]
    this.duration = data.duration
    this.currentTime = 0
  }

  // 用户交互控制进度
  setProgress(t) {         // t: 0 → 1
    this.currentTime = t * this.duration
    this.updateTransform()
  }

  // 键盘/点击增量控制
  stepForward(delta) {
    this.currentTime = Math.min(this.currentTime + delta, this.duration)
    this.updateTransform()
  }

  stepBackward(delta) {
    this.currentTime = Math.max(this.currentTime - delta, 0)
    this.updateTransform()
  }

  updateTransform() {
    // 在 keyframes 间插值
    const t = this.currentTime
    const a = this.keyframes.findLast(k => k.time <= t)
    const b = this.keyframes.find(k => k.time > t)

    if (!a || !b) return

    const localT = (t - a.time) / (b.time - a.time)
    const smoothT = this.smoothstep(localT)

    // lerp position
    this.position = a.position.map((v, i) =>
      v + (b.position[i] - v) * smoothT
    )

    // slerp rotation (quaternion)
    this.quaternion = this.slerp(a.rotation, b.rotation, smoothT)

    // 更新 view matrix
    this.viewMatrix = mat4.fromRotationTranslation(
      mat4.create(), this.quaternion, this.position
    )
  }

  smoothstep(t) {
    return t * t * (3 - 2 * t)
  }

  slerp(a, b, t) {
    // 四元数球面插值
    let dot = a[0]*b[0] + a[1]*b[1] + a[2]*b[2] + a[3]*b[3]
    if (dot < 0) { b = b.map(v => -v); dot = -dot }
    const theta = Math.acos(Math.min(dot, 1))
    if (theta < 1e-6) return a
    const st = Math.sin(theta)
    const wa = Math.sin((1-t) * theta) / st
    const wb = Math.sin(t * theta) / st
    return a.map((v, i) => v * wa + b[i] * wb)
  }
}
```

### 用户交互

```javascript
// 键盘控制
document.addEventListener('keydown', (e) => {
  switch(e.key) {
    case 'ArrowRight':
    case 'd':
      camera.stepForward(0.5)
      break
    case 'ArrowLeft':
    case 'a':
      camera.stepBackward(0.5)
      break
  }
})

// 鼠标点击前进
canvas.addEventListener('click', () => {
  camera.stepForward(1.0)  // 前进 1 秒
})

// 底部进度条控制
progressBar.addEventListener('input', (e) => {
  camera.setProgress(e.target.value / 100)
})
```

---

## 三、Instanced 渲染

```glsl
// vertex shader (GLSL 300 es)
#version 300 es

layout(location = 0) in vec3 aPosition;
layout(location = 1) in vec3 aNormal;
layout(location = 2) in vec2 aUV;
layout(location = 3) in mat4 aInstanceTransform;  // Instanced attribute
layout(location = 7) in vec4 aLightmapUV;          // Instanced attribute

uniform mat4 uViewProj;

out vec2 vUV;
out vec2 vLightmapUV;

void main() {
  vUV = aUV;
  vLightmapUV = aLightmapUV.xy;

  vec4 worldPos = aInstanceTransform * vec4(aPosition, 1.0);
  gl_Position = uViewProj * worldPos;
}
```

```javascript
// JavaScript 绑定 instanced attributes
const instanceVBO = gl.createBuffer()
gl.bindBuffer(gl.ARRAY_BUFFER, instanceVBO)

// 每个实例: transform (4 vec4) + lightmapUV (1 vec4) = 5 × 4 floats
gl.bufferData(gl.ARRAY_BUFFER, instanceData, gl.STATIC_DRAW)

// 绑定为 instanced attribute
const stride = 5 * 4 * 4  // 5 vec4 × 4 components × 4 bytes
for (let i = 0; i < 4; i++) {
  const loc = 3 + i  // aInstanceTransform 占用 location 3-6
  gl.enableVertexAttribArray(loc)
  gl.vertexAttribPointer(loc, 4, gl.FLOAT, false, stride, i * 16)
  gl.vertexAttribDivisor(loc, 1)  // 每个实例一次
}
// lightmapUV at location 7
gl.enableVertexAttribArray(7)
gl.vertexAttribPointer(7, 4, gl.FLOAT, false, stride, 4 * 16)
gl.vertexAttribDivisor(7, 1)

// 一次 draw call 渲染所有实例
gl.drawElementsInstanced(gl.TRIANGLES, indexCount, gl.UNSIGNED_SHORT, 0, instanceCount)
```

---

## 四、Post-Processing 管线

所有场景（Preloader / 片头视频 / WebGL 城市场景）共用同一套 Post-Processing，确保视觉连贯性：

```javascript
class PostProcessing {
  constructor(gl) {
    this.bloom = new BloomPass(gl)
    this.vignette = new VignettePass(gl)
    this.grain = new FilmGrainPass(gl)

    // 使用 FrameBuffer 链
    this.renderTargetA = gl.createTexture()
    this.renderTargetB = gl.createTexture()
    // ...
  }

  render(sceneTexture) {
    // Scene → Bloom → Vignette → Grain → Screen
    let input = sceneTexture

    if (this.bloom.enabled) {
      input = this.bloom.render(input, this.bloomIntensity)
    }

    if (this.vignette.enabled) {
      input = this.vignette.render(input, this.vignetteIntensity)
    }

    if (this.grain.enabled) {
      input = this.grain.render(input, this.grainIntensity)
    }

    // 最终输出到屏幕
    this.blitToScreen(input)
  }
}
```

### Post-Process 参数 (按场景调优)

```
                        Bloom     Vignette    Grain
Gate of All Nations     0.6       0.3         0.03
Apadana Palace          0.8       0.25        0.04
Throne Hall             0.7       0.35        0.05
Tripylon                0.5       0.2         0.03
Palace of Xerxes        0.7       0.3         0.04
Harem / Museum          0.4       0.15        0.02
```

这些值在 Unity Timeline 中预设，导出到 JSON 后由 WebGL 运行时读取。

---

## 五、性能优化策略

### 1. 固定相机 → 反射半渲染

```javascript
// 由于相机路径固定，可以手动指定反射中可见的对象
const REFLECTION_VISIBLE_OBJECTS = [
  'floor_gate', 'columns_gate', 'statues_gate'
  // 不包含屋顶、远处城墙等不会反射的对象
]

// 只渲染这些对象的反射，而非全场景
function renderReflection() {
  gl.bindFramebuffer(gl.FRAMEBUFFER, reflectionFBO)
  gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT)

  REFLECTION_VISIBLE_OBJECTS.forEach(id => {
    const mesh = sceneData.meshes.find(m => m.id === id)
    if (mesh) renderMesh(mesh, reflectionCamera)
  })
}
```

### 2. LOD (Level of Detail)

根据固定相机路径预计算每帧的 LOD 需求：

```javascript
// 沿相机路径采样，预计算每个物体的最大可见距离
function precomputeLOD(cameraPath, meshes) {
  const sampleCount = 100
  for (let i = 0; i < sampleCount; i++) {
    const t = i / sampleCount
    const camPos = cameraPath.getPosition(t)

    meshes.forEach(mesh => {
      const dist = distance(camPos, mesh.center)
      mesh.maxLOD = Math.max(mesh.maxLOD, getLODLevel(dist))
    })
  }
}
```

### 3. 内存管理

```javascript
// 切换房间时只加载当前房间的 assets
function loadRoom(roomId) {
  // 卸载上一个房间的资源
  unloadCurrentRoom()

  // 加载新房间的实例数据 (几何体已预加载)
  const room = sceneData.rooms.find(r => r.id === roomId)
  this.activeInstances = room.instanceIds.map(id =>
    this.instanceData[id]
  )

  // 加载房间专属光照贴图
  this.postFX.updateProfile(room.postProcessProfile)

  // 预加载房间文物的 3D/图片资源
  room.artifacts.forEach(a => preloadArtifact(a))
}

function unloadCurrentRoom() {
  // 释放纹理
  this.lightmaps.forEach(lm => gl.deleteTexture(lm))
  // 释放文物模型
  this.artifactMeshes.forEach(m => m.dispose())
  // 清空实例 buffer
  this.activeInstances = []
}
```

### 4. WebGL 2.0 独家功能

| 功能 | 用途 |
|---|---|
| **Instancing** | 批量渲染重复几何体 |
| **VAO (Vertex Array Object)** | 快速切换顶点配置 |
| **UBO (Uniform Buffer Object)** | 共享相机矩阵、光照参数 |
| **Mipmap** | 纹理远距离降采样 |
| **sRGB 纹理** | 正确的颜色空间 |

### 性能目标

| 指标 | 目标 |
|---|---|
| 帧率 | 60fps (桌面) / 30fps (移动) |
| 三角形数 | ~500K (单帧) |
| Draw Calls | ~200 (含 instancing) |
| 纹理内存 | < 256MB |
| JSON 体积 | ~5MB (gzip) |
