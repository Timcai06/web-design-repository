# 完整技术栈 + 行业对比

## 一、完整工具链

### 前端渲染

| 技术 | 用途 | 备注 |
|------|------|------|
| Hydra 3D Engine | 自研 WebGL 渲染 | 从 Three.js 迁移，持续迭代 7 年+ |
| WebGL 2.0 | 底层图形 API | WebGL 1.0 作为降级方案 |
| GLSL 300 es | 着色器语言 | 自研着色器库 |
| ActiveFrame | 逐帧视频播放 | 基于 WebCodecs API |
| Draco 3D | 网格压缩 | Google 开源库，压缩率 75-80% |
| WebWorker | 后台并行 | 几何加载/粒子生成/物理计算 |

### 后端 / 基础设施

| 技术 | 用途 |
|------|------|
| Node.js | 后端运行时 |
| Google Cloud Platform (GCP) | 云基础设施 |
| Firebase | 实时数据库、认证、云函数 |
| WebSocket | 多用户实时光标同步 |
| Sanity.io / Contentful | Headless CMS |

### 工具链

| 工具 | 用途 |
|------|------|
| Hydra GUI | 可视化 3D 场景编辑器 |
| Houdini | 程序化几何生成 |
| Cinema 4D / Blender | 3D 建模 |
| Figma | UI 设计 |

---

## 二、从 Three.js 到 Hydra 的迁移决策

### 迁移 Timeline

```
Phase 1 (2015): 在 Three.js 上层封装工具函数，标准化工作流
Phase 2 (2017): 替换 Material 层为 Shader 层（仍然是 Three.js 渲染管线）
Phase 3 (2019): 完全替换渲染管线，使用自研 WebGL 抽象
Phase 4 (2021): Hydra GUI 第一版
Phase 5 (2024): ATv6 — Hydra + GUI 成熟版
```

### 迁移中保留的 Three.js 设计范式

```javascript
// Three.js 范式
new THREE.Mesh(geometry, material)

// Hydra 保留了这个范式但替换了实现
hydra.createMesh({ geometry: 'box', shader: 'standard' })
```

### 迁移中丢弃的

1. **Material 系统** → 直接操作 Shader
2. **通用渲染管线** → 针对已知场景优化
3. **Math 库** → 自研（只包含需要的操作）
4. **Loader 系统** → 自定义加载器（集成 Draco）

---

## 三、性能指标

| 指标 | ATv6 | 典型 Three.js 站点 |
|------|------|-------------------|
| LCP | ~1.3s | ~3-5s |
| 首屏 JS | ~200KB | ~500KB+（含 Three.js） |
| 3D 模型加载 | Draco 压缩 | 原始 GLTF |
| 渲染帧率 | 60fps (桌面) | 30-60fps |
| 移动端 | ~30fps | ~15-30fps |

**关键优化手段：**
1. Draco 压缩 — 减少 75% 的网格传输量
2. WebWorker 并行 — 不阻塞主线程
3. 懒加载视频 + 3D 资产 — 首屏只加载必要资源
4. 精简单帧 JS 执行 — state-based functional 代码比 OOP 更少开销

---

## 四、ATv6 在 6 个案例中的定位

```
                  渲染复杂度
                      ↑
               Hydra  │ ATv6
               Three.js │ Oryzo
              自研 WebGL │ Luke, Persepolis
                      │
               CSS 3D  │ Zentry
               Rive    │ landonorris
                      │
                      └──────────────→ 交互复杂度
                        低          高
                             多用户
                             AI 导航
```

### 每家的"独门武器"

| 项目 | 别人做不了/不会做的 |
|------|-------------------|
| **ATv6** | 自研引擎 + 多用户 + AI 导航 |
| **Zentry** | 零 WebGL 实现 3D 效果 |
| **landonorris** | Rive 作为主力动效引擎 |
| **Oryzo** | 讽刺叙事 + 全 3D 产品展示 |
| **Luke** | 纯手写无框架，GLSL 专家 |
| **Persepolis** | Unity 级 3D 在浏览器中 |
