# Hydra 引擎 + WebGL 管线

Active Theory 的核心技术资产：**Hydra**，一个自研的 WebGL 3D 引擎 + 可视化 GUI 编辑器。

---

## 一、为什么自研引擎

### 时间线

```
2012: Active Theory 成立，使用 Three.js
2015: 开始自研工具链（Aura: WebGL Native 运行环境）
2017: Hydra 3D Engine 第一版（基于 Three.js 设计范式）
2019: Hydra 完全脱离 Three.js，自研渲染管线
2024: ATv6 — Hydra 成熟版 + Hydra GUI
```

### 决策逻辑

| 维度 | Three.js | Hydra | 
|------|---------|-------|
| **渲染管线** | 通用（处理所有场景） | 专用（针对已知场景优化） |
| **CPU 开销** | 高（大量 JS 抽象） | 低（精简调用链） |
| **GPU 吞吐** | 受限于通用性 | 可精确控制 |
| **工具链** | 无官方 GUI | Hydra GUI（可视化编辑器） |
| **跨平台** | 仅 Web | Web + iOS + Android + Desktop + VR |
| **学习曲线** | 中 | 高（自研，无社区） |

### 什么情况下你应该考虑自研引擎

- **团队 >10 人** — 有资源维护引擎
- **项目类型高度重复** — 引擎的 ROI 取决于使用频率
- **有特定渲染需求** — Three.js 的通用 API 成为瓶颈
- **需要设计师参与 3D** — GUI 编辑器降低设计师门槛

**对于个人开发者 / 小团队：Three.js 仍然是最正确的选择。** Active Theory 在成立 5 年后才开始自研引擎。

---

## 二、Hydra 渲染管线

### 架构

```
JavaScript (State-based Functional)
       │
       ▼
  Scene Graph (节点树)
       │
       ├── Geometry (Draco 压缩网格)
       ├── Shader (自定义 GLSL)
       └── Transform (位置 / 旋转 / 缩放)
       │
       ▼
  Renderer (自研)
       │
       ├── WebGL 2.0 (首选)
       │   └── WebGL 1.0 (fallback)
       │
       ▼
  Post-Processing
       │
       └── GPU (最终输出)
```

### 与 Three.js 的关键区别

```javascript
// Three.js 方式
const geometry = new THREE.BoxGeometry(1, 1, 1)
const material = new THREE.MeshStandardMaterial({ color: 0xff0000 })
const mesh = new THREE.Mesh(geometry, material)
scene.add(mesh)

// Hydra 方式
const mesh = hydra.createMesh({
  geometry: 'box',
  shader: 'standard',
  uniforms: { color: [1, 0, 0] }
})
scene.add(mesh)
```

核心差异：
1. **Shader 取代 Material** — Three.js 有大量 Material 子类（MeshStandardMaterial, MeshPhongMaterial 等），每一个都包含复杂的 PBR 计算。Hydra 直接用着色器，减少了抽象层。
2. **渲染管线精确控制** — Three.js 需要处理所有可能的渲染场景（透明物体排序、阴影映射等）。Hydra 只处理它知道的场景——因此调用链更短、CPU 开销更低。
3. **Instancing 默认启用** — Three.js 需要显式使用 `InstancedMesh`。Hydra 自动检测重复几何并启用 instancing。

---

## 三、性能优化

### Draco 网格压缩

```
原始 GLTF:    ~15MB (3D 场景文件)
Draco 压缩:   ~3-4MB (压缩率 75-80%)
加载后解码:   添加 ~50ms 解码时间
```

**ATv6 的 LCP ~1.3s** — 这在 WebGL 重度站点中极其优秀（Oryzo 约 3-5s，Persepolis 约 5-10s）。

### WebWorker 并行化

```javascript
// 架构: 所有重量级任务移至 WebWorker
const worker = new Worker('geometry-worker.js')

worker.postMessage({ type: 'loadGeometry', url: 'model.drc' })
worker.onmessage = (e) => {
  // 主线程只做渲染
  hydra.addGeometry(e.data.geometry)
}
```

在 WebWorker 中执行的任务：
- 几何加载 + Draco 解码
- 粒子系统生成
- 物理碰撞计算

### 渲染管线优化

```
CPU 主线程: 场景更新 + 状态管理
     │
     ▼
GPU (WebGL 2.0): 顶点处理 + 光栅化 + 后处理
     │
WebWorker: 几何加载 + 粒子生成 + 物理
```

---

## 四、ActiveFrame — 自研视频格式

Active Theory 构建了 `ActiveFrame`：基于 WebCodecs API 的自定义逐帧视频播放器。

### 为什么不用 `<video>` 标签

```html
<!-- 标准视频：无法逐帧精确控制 -->
<video src="hero.mp4" />

<!-- ActiveFrame：逐帧控制，与 3D 场景同步 -->
<canvas id="activeframe" />
<script>
const af = new ActiveFrame({
  src: 'hero.aframe',
  canvas: document.getElementById('activeframe'),
})
// 精确跳转到第 127 帧
af.seek(127)
// 帧级同步到 3D 场景时间线
scene.on('timelineUpdate', (frame) => af.seek(frame))
```
```

### 适用场景

ActiveFrame 适合：
- 需要帧级精确控制的视频（与 3D 场景同步）
- 不能有 `<video>` 标签额外开销的 WebGL 场景
- 自定义编解码需求

**对于不需要帧级控制的视频，标准 `<video>` 标签仍然更简单高效。**

---

## 五、Hydra GUI — 设计师友好工具

```
┌────────────────────────────────────────────────────┐
│  Hydra GUI (开发者用)                               │
│  ┌────────────┐ ┌────────────┐ ┌────────────────┐  │
│  │ Scene Tree │ │ Properties │ │  3D Viewport    │  │
│  │            │ │  Position  │ │                 │  │
│  │ ├─ Lights  │ │  Rotation  │ │  (实时预览)      │  │
│  │ ├─ Camera  │ │  Scale     │ │                 │  │
│  │ ├─ Models  │ │  Shader    │ │                 │  │
│  │ └─ FX      │ │  Params    │ │                 │  │
│  └────────────┘ └────────────┘ └────────────────┘  │
│  ┌────────────────────────────────────────────────┐ │
│  │  GUI 数据 ↔ Firebase ←→ 多协作者实时同步          │ │
│  └────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘
```

**核心理念：** "Unity meets Photoshop" — 设计师在 GUI 中拖拽构建 3D 场景，开发者通过 JS 访问每个节点添加交互逻辑。场景数据通过 Firebase 实时同步，多团队成员可以同时工作。

这与传统管线（设计师出 Figma → 开发者写 Three.js 代码 → 反复沟通调整）完全不同——设计师和开发者操作同一个场景文件。
