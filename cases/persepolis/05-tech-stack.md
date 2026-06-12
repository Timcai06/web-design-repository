# Persepolis — 技术栈与管线深度分析

---

## 一、完整工具链

### 离线管线 (Content Creation)

| 工具 | 用途 | 输出 |
|---|---|---|
| **Maya** | 高模/低模建模、UV 展开 | `.fbx` 模型 |
| **Substance Painter** | PBR 贴图（Albedo/Normal/Roughness/Metalness/AO） | `.tga` / `.png` 贴图 |
| **Unity** | 场景装配、光照烘焙 (Lightmapping)、Timeline 相机动画、Instancing | `.unity` 场景 |
| **Clarisse (Isotropix)** | 场景合成、Look Development、360 穹顶环境、植被散布 | `.exr` HDR / 贴图 |
| **Custom Unity Exporter** | Media.Monks 自研 Editor Script，导出场景为 JSON | `persepolis.json` |
| **After Effects** | 片头视频合成、特效、调色 | `.mp4` 视频 |
| **Substance Designer** | 程序化纹理生成（砖墙、地面等） | `.sbsar` |

### 运行时 (Browser)

| 技术 | 用途 | 来源 |
|---|---|---|
| **WebGL 2.0** | 底层 3D 渲染 API (VAO/UBO/Instancing) | 原生 |
| **Media.Monks Custom Framework** | 自研渲染引擎（非 Three.js） | 自研 |
| **Vue.js** | UI 层（导航、文物面板、进度条、地图） | CDN / NPM |
| **GSAP** | 过渡动画（preloader、文物面板、雾效） | NPM |
| **Web Audio API** | 环境音效、旁白播放 | 原生 |

### JSON 数据流

```
Unity Scene
  │
  ▼
Custom C# Editor Script
  ├─ Mesh → Vertex Buffers (flat float arrays)
  ├─ Instance → Transform Matrices + Lightmap UVs
  ├─ Lightmap → Base64 PNG
  ├─ Camera Path → Keyframes (position + quaternion)
  ├─ Post-Process → Config JSON
  └─ Room Artifacts → Timeline Event Data
      │
      ▼
persepolis.json (gzipped ~5MB)
      │
      ▼
WebGL Runtime (JavaScript fetch → parse → render)
```

---

## 二、文件体积分析

| 资源 | 体积 (gzip) | 说明 |
|---|---|---|
| `persepolis.json` | ~5MB | 网格、实例、光照贴图、相机路径 |
| 光照贴图纹理 | ~2MB | 内嵌在 JSON 中（base64） |
| 片头视频 | ~3MB | After Effects 渲染 |
| 文物 3D 模型 (×19) | ~1MB | 按需加载 |
| 文物图片 | ~2MB | 高清照片 |
| 音频 (旁白+音效) | ~3MB | 按需加载 |
| 字体/CSS/JS | ~500KB | Vue + GSAP + 自研框架 |
| **总计** | **~16.5MB** | |

---

## 三、Persepolis vs 其他案例

| 维度 | Persepolis | landonorris/lukebaffait/oryzo | zentry |
|---|---|---|---|
| **类型** | WebGL 虚拟导览 | Scroll-driven 营销站点 | 品牌官网 |
| **核心 3D** | Unity → WebGL 自定义引擎 | Three.js / Rive | DOM-only (CSS clip-path) |
| **导航** | 固定相机路径 + 地图跳转 | 垂直滚动 | 垂直滚动 |
| **框架** | Vue.js | 无 / Astro | Nuxt 3 + Vue 3 |
| **CMS** | 无 (静态 JSON 数据) | 无 / Webflow | Storyblok |
| **音频** | 环境音 + 旁白 | 无 | 无 |
| **视频** | 片头 + Chroma Key 融合 | Vimeo overlay | ScrollyVideo |
| **移动端** | 降级帧率 | Canvas DPR 限制 | CSS will-change |
| **开发周期** | 9 个月 | 数周 | 数周 |

### 为什么不用 Three.js？

Media.Monks 选择自研 WebGL 框架而非 Three.js：

| 原因 | 说明 |
|---|---|
| **Unity 兼容性** | 自研框架直接对应 Unity 导出格式，无需中间转换 |
| **Instancing 优先** | 核心性能策略依赖 Instancing，Three.js 原生支持有限 |
| **Post-Processing** | 需要 Chroma Key、Bloom、Vignette、Grain 统一管线 |
| **控制力** | 固定相机路径 + 反射优化，需要底层 GL 控制 |
| **历史积累** | Media.Monks 已有自研框架积累，复用成本低 |

---

## 四、获奖记录

| 奖项 | 类别 | 年份 |
|---|---|---|
| **FWA of the Year** | Site of the Year | 2023 |
| **FWA People's Choice Award** | People's Choice | 2023 |
| **Awwwards SOTD** | Site of the Day | 2022 |
| **Dutch Digital Design** | Featured Case | 2022 |

---

## 五、团队

- **Studio**: Media.Monks (Netherlands)
- **Client**: Getty Villa Museum
- **Team Size**: ~15 人
- **Timeline**: 9 个月
- **Collaborators**: Getty 策展人、UCLA 历史学教授、古波斯音乐专家

---

## 六、相较其他四个案例的关键差异

```
                    Scroll-driven 营销          Unity → WebGL 虚拟导览
                    ─────────────────        ──────────────────────────
交互模式:            ↓ 滚动 / hover           → 点击前进 / 拖动进度条
3D 渲染:            Three.js / Rive          Unity 烘焙 + WebGL 2.0
动画引擎:            GSAP + Lenis            Unity Timeline + 自定义路径
内容:               营销文案                   历史教育 + 文物展示
核心挑战:           性能 ✕ 视觉效果           历史准确性 ✕ 浏览器性能
工作室:              Lusion / OFF+BRAND / Resn  Media.Monks
```

Persepolis 是所有五个案例中**唯一不是 scroll-driven**、**唯一使用 Unity 游戏引擎**、**唯一带旁白音频**、**唯一有文物教育内容**的项目。
