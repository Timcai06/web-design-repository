# Persepolis Reimagined — Technical Case Study

**URL:** https://persepolis.getty.edu  
**Studio:** [Media.Monks](https://media.monks.com/) (Netherlands)  
**Client:** Getty Villa Museum  
**Type:** Virtual museum tour / immersive WebGL experience  
**Date Analyzed:** 2026-05-29

---

## 项目概览

Persepolis Reimagined 是 Getty Villa 博物馆实体展览 "Persia: Ancient Iran and the Classical World" 的数字化补充体验——在浏览器中**以 WebGL 实时渲染一座 2500 年前古城的精确 3D 重建**。

与前述 4 个案例（都是 scroll-driven 营销站点）不同，这是一个**3D 虚拟导览**——用户在 Persepolis 的 6 个宫殿/广场中自由穿行，发现 19 件历史文物的 3D 复原。

### 体验流程

```
进入站点
  │
  ├─ Preloader (WebGL 加载进度)
  │
  ├─ 片头视频 (Cinematic Title Sequence)
  │   └─ 无缝过渡到 WebGL 场景 (Chroma Key Shader)
  │
  ├─ 鸟瞰视图 — 整个 Persepolis 城
  │   ├─ 滚动/缩放查看全城
  │   └─ 选择入口 (Gate of All Nations)
  │
  ├─ 6 个房间探索:
  │   ├─ Gate of All Nations (万国门)
  │   ├─ Apadana Palace (阿帕达纳宫)
  │   ├─ Throne Hall (百柱殿)
  │   ├─ Tripylon / Council Hall
  │   ├─ Palace of Xerxes (薛西斯宫)
  │   └─ Harem / Museum
  │   │
  │   ├─ 固定相机路径穿行
  │   ├─ 雾化转场 (mist transition)
  │   ├─ 发现文物 → 弹出交互面板
  │   └─ "Past & Present" 古今对比
  │
  └─ 退出体验
```

### 技术栈

| 技术 | 用途 |
|---|---|
| **Unity** | 3D 场景搭建、光照烘焙、Instancing、相机动画 |
| **WebGL 2.0** | 浏览器端实时 3D 渲染（含 Instancing） |
| **Media.Monks 自研 WebGL 框架** | 自定义渲染管线 |
| **Vue.js** | 前端 UI 层框架 |
| **GSAP** | 过渡/动画 |
| **Custom Unity → JSON Exporter** | Unity 场景导出为自定义 JSON 格式 |
| **Clarisse (Isotropix)** | 场景合成、Look Development、贴图烘焙 |
| **Maya** | 3D 建模 |
| **Substance Painter** | PBR 材质贴图 |
| **After Effects** | 片头视频合成 |
| **Chroma Key WebGL Shader** | 视频 → WebGL 无缝过渡 |

---

## 架构总览

```
Media.Monks 3D Pipeline (Offline)         浏览器运行时 (Web)
┌──────────────────────────────┐          ┌────────────────────────┐
│  Maya (建模)                  │          │  WebGL 2.0              │
│    ↓                         │          │  ├─ Custom Framework    │
│  Substance Painter (贴图)     │          │  ├─ Instanced Rendering │
│    ↓                         │          │  ├─ Post-Processing     │
│  Unity (场景搭建 + 光照)      │──JSON──→│  │  (vignette/bloom/grain)│
│    ↓                         │          │  ├─ Chroma Key Shader   │
│  Custom Exporter (自定义导出)  │          │  └─ Fixed Camera Path  │
│    ↓                         │          │                        │
│  Clarisse (Look Dev)         │          │  Vue.js (UI Overlay)    │
│    ↓                         │          │  GSAP (Transitions)     │
│  After Effects (片头视频)     │          │                        │
└──────────────────────────────┘          └────────────────────────┘
```

---

## 文件索引

| 文件 | 内容 |
|---|---|
| `01-unity-pipeline.md` | Unity 场景搭建、光照烘焙、Instancing、自定义 JSON Exporter |
| `02-webgl-renderer.md` | 自定义 WebGL 框架、固定相机路径、Post-Processing、性能优化 |
| `03-transitions.md` | Preloader、片头视频、Chroma Key Shader、雾化转场 |
| `04-narrative-ux.md` | 6 房间导航、19 文物交互、Past & Present、叙事设计 |
| `05-tech-stack.md` | 完整管线、工具链、Media.Monks vs 其他工作室对比 |
