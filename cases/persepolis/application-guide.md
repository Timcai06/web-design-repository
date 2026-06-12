# Persepolis Reimagined — Design Rationale & Application Guide

Persepolis 与其他 4 个案例不同——它不是 scroll-driven 营销站，而是**虚拟博物馆导览**。设计哲学：**让 2500 年前的古城在浏览器中"复活"，而不只是"展示"它。**

---

## 一、设计语言分析

### 视觉 DNA

```
照片级真实 3D 重建（Unity + WebGL） ← 核心
     ↓
电影级片头（After Effects + Chroma Key 过渡）
     ↓
学术性 UI（文物面板、时间线、地图）
     ↓
沉浸式音频氛围
```

**为什么这套语言有效：**

1. **照片级真实** — 对于"古城重建"，用户的期待是"真实感"。风格化（low-poly、卡通渲染）会削弱"这是真实历史"的可信度。Media.Monks 选择 PBR 材质 + 物理光照烘焙，目标就是让用户感觉"我站在 Persepolis 的真实废墟中"。

2. **固定相机路径** — 这不是"能自由走动的游戏"，而是"沿着精心设计的路线参观"。固定路径意味着每个镜头都是精心构图的——不会出现"用户看向地面"或"卡在墙里"的体验问题。对于博物馆导览，"被引导"比"自由探索"更好——因为用户不是来玩的，是来学习的。

3. **电影片头 + Chroma Key 过渡** — 从视频到 WebGL 的无缝过渡（Chroma Key shader 去除视频的绿色背景，叠加到 WebGL 场景上），制造"从电影走进现实"的 illusion。

4. **Past & Present 滑块** — 左右拖动滑块：左边是"现在的废墟"（实拍照片），右边是"2500 年前"（3D 重建）。这是 Persepolis 的叙事核心——"时间的力量"。

---

## 二、每个效果的 Design Rationale + Application Guide

### 1. Fixed Camera Path（固定相机路径）

**为什么好：** 虚拟导览的第一法则是"不要让用户迷路"。固定路径意味着：
- 每个镜头都是"最佳视角"——设计团队精心选择了最能展示建筑的角度
- 不存在"用户看向地面"或"盯着墙壁 10 秒"的体验问题
- Path 上的缓动曲线控制节奏——快的地方加速，重要的文物前减速

**适合放在：** 博物馆、教育、虚拟 tours、房地产 VR 看房。

**改编方案 — 通用 Camera Path：**
```typescript
interface CameraKeyframe {
  position: [number, number, number]
  target: [number, number, number]
  timestamp: number  // 0-1 along path
}

function animateCamera(keyframes: CameraKeyframe[], progress: number) {
  // 按 progress 插值相邻两个 keyframe
  // position = lerp(A.position, B.position, localProgress)
  // target = lerp(A.target, B.target, localProgress)
  camera.position.copy(position)
  camera.lookAt(target)
}
```

### 2. Preloader（WebGL 加载进度）

**为什么好：** 不是"进度条+logo"— 而是显示 WebGL 资源加载状态 + 一个简短的开场动画。让等待成为体验的一部分。

**适合放在：** 任何需要加载大量 3D 内容的网站。

### 3. Chroma Key Shader（视频 → WebGL 过渡）

**为什么好：** 片头视频（After Effects 合成）通过 Chroma Key 去除绿色背景，叠加到 WebGL 3D 场景上。视频淡出后，WebGL 场景接管——用户感觉"走进了视频里的世界"。

**技术原理：**
```glsl
// Fragment Shader: Chroma Key
vec3 videoColor = texture2D(videoTexture, uv).rgb;
float key = smoothstep(0.1, 0.5, length(videoColor - vec3(0.0, 1.0, 0.0)));
// key = 0 for green, 1 for non-green
gl_FragColor = mix(webglColor, vec4(videoColor, 1.0), key);
```

**适合放在：** 任何有"片头"需求的沉浸式 Web 体验——游戏、虚拟导览、品牌电影。

### 4. Room Transitions via Fog（雾化转场）

**为什么好：** 从一个宫殿走到另一个时，场景逐渐被雾气覆盖，然后在新场景中雾气消散。这比"硬切"或"淡入/淡出"更自然——雾是物理世界的自然现象，用户不需要学习就能理解"我走过了很长的路"。

**适合放在：** 任何有"场景切换"的体验，尤其是 3D 环境。

**通用 Fog Transition：**
```javascript
function fadeToBlack(scene, duration = 1.5) {
  // 增加雾密度到极值
  gsap.to(scene.fog, {
    density: 0.1,  // Three.js FogExp2
    duration: duration / 2,
    onComplete: () => {
      // 切换场景内容
      loadNewScene()
      // 让雾消散
      gsap.to(scene.fog, {
        density: 0.001,
        duration: duration / 2
      })
    }
  })
}
```

### 5. Past & Present Slider

**为什么好：** 并排比较是人类最自然的"理解变化"方式。左右拖动的交互方式简单到不需要教程——3 岁小孩都会用。

**适合放在：** 历史/文化/教育体验，"改进/对比"展示。

**改编方案：**
```css
.comparison-container {
  position: relative;
  width: 100%;
  height: 500px;
}
.comparison-past {
  position: absolute;
  inset: 0;
  /* 左侧显示 "past" 内容 */
  clip-path: inset(0 50% 0 0);
}
.comparison-present {
  position: absolute;
  inset: 0;
  /* 右侧显示 "present" 内容 */
}
```

### 6. Interactive Artifact Panels（交互文物面板）

**为什么好：** 每个文物（共 19 件）在用户靠近时弹出交互面板，显示：文物名称、年代、3D 模型、历史描述、Past & Present 对比。

**交互模式：** "靠近即弹出" — 不是点击，不是滚动——是"空间触发"。这符合博物馆体验——参观者在展品前停下时，信息自然出现。

**适合放在：** 教育/博物馆/cultural 体验。

---

## 三、与其他案例的对比

| 维度 | Persepolis | 其他 4 个案例 |
|------|-----------|-------------|
| 交互模式 | 点击/接近触发 | 滚动驱动 |
| 3D 管线 | Unity → 自定义 WebGL 导出 | Three.js / CSS / Rive |
| 叙事方式 | 博物馆导览（线性路径） | 品牌叙事（自由滚动） |
| 内容类型 | 教育/学术 | 营销/品牌 |
| 开发成本 | 极高（Unity 团队 + 3D 美术） | 中-高 |
| 适用客户 | Getty Villa (博物馆) | 品牌/产品/个人 |

**关键决策：** 如果你的项目需要"在浏览器中展示一个真实空间的精确 3D 重建"，Persepolis 的 Unity-to-JSON 管线是正确路径。但如果只需要"看起来 3D"的效果，其他 4 个案例的轻量方案更合适。

---

## 四、整体可复用性评分

| 效果 | 可复用性 | 开发成本 | 视觉冲击 | 推荐优先级 |
|------|---------|---------|---------|-----------|
| Past & Present Slider | ⭐⭐⭐⭐ | 中 | ⭐⭐⭐⭐⭐ | 1 |
| Fog Transitions | ⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐ | 2 |
| Chroma Key 视频过渡 | ⭐⭐⭐ | 中 | ⭐⭐⭐⭐⭐ | 3 |
| Proximity Trigger UI | ⭐⭐⭐ | 中 | ⭐⭐⭐ | 4 |
| Fixed Camera Path | ⭐⭐⭐ | 高 | ⭐⭐⭐⭐ | 5 |
| Unity → WebGL Pipeline | ⭐ | 极高 | ⭐⭐⭐⭐⭐ | 6（仅博物馆/教育） |
| 自定义 JSON Exporter | ⭐ | 极高 | ⭐⭐ | 7 |
