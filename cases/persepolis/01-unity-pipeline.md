# Unity 3D 管线 — 场景搭建 / 光照 / Instancing / 导出

Persepolis 的 3D 资产管线是典型的 **离线烘焙 + 实时渲染** 模式：在 Unity 中搭建好完整的场景（光照、材质、模型放置），然后通过自定义导出器转换为 WebGL 可用的格式。

---

## 一、模型资产管线

```
Maya (高模)
  │
  ├─ 减面优化 (Low-poly baking)
  │
  ▼
Substance Painter (PBR 贴图)
  ├─ Albedo (漫反射)
  ├─ Normal (法线)
  ├─ Roughness (粗糙度)
  ├─ Metalness (金属度)
  └─ AO (环境光遮蔽)
      │
      ▼
Unity (场景装配 + 光照烘焙)
  │
  ▼
Custom JSON Exporter → WebGL
```

### 模型来源

Getty 提供了基于考古研究的初始 3D 模型，Media.Monks 与 Getty 策展人 + UCLA 教授协作不断修正：

- 建筑尺寸、布局、房间功能
- 雕像和浮雕的精确复原
- 材质和颜色的历史考证（石灰岩、泥砖墙、雪松木屋顶）

---

## 二、Unity 场景搭建

### 光照烘焙 (Lightmapping)

Unity 的原生 lightmapping 功能用于预计算静态光照：

```csharp
// Unity 场景中设置
LightmapSettings.lightmapsMode = LightmapsMode.NonDirectional

// 每个网格设置:
//   - Static: true (参与光照烘焙)
//   - Lightmap Scale: 根据重要性调整 (重要建筑更高)
```

收益：
- 运行时不需要实时光照计算
- 可与 instancing 兼容（Unity 默认支持）
- 提高移动端性能

### Instancing

利用 WebGL 2.0 的 instancing 功能批量渲染重复几何体：

```csharp
// Unity 中所有相同的柱子、墙壁共享同一个 Mesh
// 运行时通过 instancing 一次 draw call 渲染所有实例

// 每个实例的变换矩阵 (position, rotation, scale) 存储在 buffer 中
// GPU 端: gl_InstanceID 读取对应矩阵

struct InstanceData {
  mat4 transform;
  vec4 color;        // 轻微颜色变化增加视觉丰富度
  float lightmapUV;  // 光照贴图 UV
};
```

**效果**：Persepolis 有大量重复结构（柱子、台阶、城墙），Instancing 使 draw call 从数万降到数百。

---

## 三、固定相机路径

这是关键的优化决策：**不要自由漫游，使用固定相机路径**。

### Unity 中的相机动画

```csharp
// Unity Timeline 中定义相机路径
// 每个"房间"是一条独立的 Timeline 片段

[RequireComponent(typeof(CinemachineVirtualCamera))]
public class CameraPathController : MonoBehaviour {
  public List<Transform> waypoints;    // 路径点
  public float duration = 30f;         // 走完全程时间
  public AnimationCurve speedCurve;    // 速度曲线 (进/出缓动)

  public Vector3 GetPosition(float t) {
    // Catmull-Rom 插值
    // t: 0 → 1 映射到路径进度
  }
}
```

### 固定路径的优势

| 优势 | 说明 |
|---|---|
| **反射优化** | 可手动指定反射中可见对象，避免渲染整个场景两次 |
| **Post-Process 固定** | Bloom 等效果可按场景手动调优 |
| **LOD 固定** | 知道相机在每帧看哪里，LOD 切换可以预设 |
| **遮挡剔除** | 已知视角方向，提前剔除不可见物体 |
| **性能可预测** | 每帧渲染负载稳定 |

### Unity Timeline 控制

```
Timeline (整个体验)
  │
  ├─ Gate of All Nations     → Camera Track + Post-Process Profile
  ├─ Apadana Palace          → Camera Track + Post-Process Profile
  ├─ Throne Hall             → Camera Track + Post-Process Profile
  ├─ Tripylon                → Camera Track + Post-Process Profile
  ├─ Palace of Xerxes        → Camera Track + Post-Process Profile
  └─ Harem / Museum          → Camera Track + Post-Process Profile
      │
      └─ 每个片段:
          ├─ Cinemachine Virtual Camera (路径动画)
          ├─ Post-Process Volume (Bloom 强度、色调)
          ├─ Lightmap Swap (房间专属光照)
          └─ Event Markers (文物揭示触发点)
```

---

## 四、自定义 Unity → JSON Exporter

Media.Monks 开发了自定义导出器，将 Unity 场景序列化为 JSON：

```csharp
// Editor/Exporters/SceneExporter.cs
public class SceneExporter : EditorWindow {
  [MenuItem("Media.Monks/Export Scene to WebGL")]
  static void Export() {
    var sceneData = new SceneData();

    // 1. 导出几何体 (Mesh → vertex buffers)
    foreach (var meshFilter in FindObjectsOfType<MeshFilter>()) {
      var mesh = meshFilter.sharedMesh;
      sceneData.meshes.Add(new MeshData {
        vertices = mesh.vertices,
        uvs = mesh.uv,
        normals = mesh.normals,
        triangles = mesh.triangles,
        // 不导出 tangent 以减少体积
      });
    }

    // 2. 导出实例数据 (transform 矩阵 + lightmap UV)
    foreach (var instance in FindObjectsOfType<InstancedMesh>()) {
      sceneData.instances.Add(new InstanceData {
        meshId = instance.meshId,
        transform = instance.transform.localToWorldMatrix,
        lightmapIndex = instance.lightmapIndex,
        lightmapScaleOffset = instance.lightmapScaleOffset,
      });
    }

    // 3. 导出光照贴图
    foreach (var lm in LightmapSettings.lightmaps) {
      sceneData.lightmaps.Add(EncodeToBase64(lm.lightmapColor));
    }

    // 4. 导出相机路径 (Timeline → keyframes)
    sceneData.cameraPath = ExportCameraTimeline();

    // 5. 导出 Post-Process 配置
    sceneData.postProcess = ExportPostProcessProfile();

    // 6. 序列化为 JSON
    File.WriteAllText("Assets/Export/persepolis.json",
      JsonUtility.ToJson(sceneData));
  }
}
```

### 导出 JSON 结构

```json
{
  "version": "2.0",
  "meshes": [
    {
      "id": "column_01",
      "vertexCount": 1024,
      "indexCount": 6144,
      "positions": [...],
      "normals": [...],
      "uvs": [...],
      "indices": [...]
    }
  ],
  "instances": [
    {
      "meshId": "column_01",
      "transform": [1,0,0,0, 0,1,0,0, 0,0,1,0, 10,0,5,1],
      "lightmapUV": [0.125, 0.25],
      "lightmapIndex": 0
    }
  ],
  "lightmaps": ["base64...", "base64..."],
  "cameraPath": {
    "duration": 180,
    "keyframes": [
      { "time": 0, "position": [0,1.7,0], "rotation": [0,0,0,1] },
      { "time": 30, "position": [5,1.7,2], "rotation": [0,0.707,0,0.707] }
    ]
  },
  "postProcess": {
    "bloom": { "intensity": 0.8, "threshold": 0.6 },
    "vignette": { "intensity": 0.3 },
    "grain": { "intensity": 0.05 }
  },
  "rooms": [
    {
      "id": "gate_of_all_nations",
      "cameraRange": [0, 30],
      "artifacts": [
        { "id": "foundation_plate", "triggerTime": 15 }
      ]
    }
  ]
}
```

---

## 五、Clarisse (Isotropix) 场景合成

对于 WebGL 使用的贴图和 360 度穹顶环境，使用 **Clarisse** 进行 Look Development：

```
Maya 模型 → Clarisse
  ├─ 场景装配（植被、人群、道具）
  ├─ 贴图烘焙（diffuse/specular/normal）
  ├─ 360 度穹顶环境渲染 (用于背景)
  └─ 动画镜头输出 (用于片头)
```

Clarisse 的优势：处理大量几何体时保持交互帧率，支持程序化散布（植被、人群）。

---

## 六、导出管线总览

```
                    Offline Pipeline
  ┌──────────────────────────────────────────────┐
  │                                               │
  │  Maya ─→ Substance ─→ Unity ─→ Clarisse      │
  │    │           │         │         │          │
  │    ▼           ▼         ▼         ▼          │
  │  .fbx       .tga     .unity    .exr/tif       │
  │                                               │
  │  ┌──────────────────────────────────────┐     │
  │  │     Custom Scene Exporter Script      │     │
  │  │  (Unity Editor Extension)            │     │
  │  │                                       │     │
  │  │  ┌─────────────────────────────────┐  │     │
  │  │  │        persepolis.json           │  │     │
  │  │  │  ├─ meshes (vertex buffers)      │  │     │
  │  │  │  ├─ instances (matrix data)      │  │     │
  │  │  │  ├─ lightmaps (base64)           │  │     │
  │  │  │  ├─ cameraPath (keyframes)       │  │     │
  │  │  │  ├─ postProcess (config)         │  │     │
  │  │  │  └─ rooms (artifact triggers)    │  │     │
  │  │  └─────────────────────────────────┘  │     │
  │  └──────────────────────────────────────┘     │
  │                    │                           │
  └────────────────────┼──────────────────────────┘
                       │ HTTP
                       ▼
                 Browser Runtime
              (WebGL 2.0 + Vue.js)
```
