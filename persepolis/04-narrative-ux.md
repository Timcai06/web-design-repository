# 房间导航 / 文物交互 / "Past & Present"

---

## 一、6 房间布局

```
                   ┌─────────────────────┐
                   │   鸟瞰视图/全城       │
                   │   (入口选择界面)      │
                   └──────────┬──────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
    ┌─────────────────┐ ┌────────────┐ ┌──────────────┐
    │ Gate of All     │ │  Apadana   │ │  Throne Hall │
    │ Nations (万国门) │ │  Palace    │ │  (百柱殿)     │
    │                 │ │  (阿帕达纳)  │ │              │
    │  0-30s          │ │  30-60s    │ │  60-90s      │
    └────────┬────────┘ └─────┬──────┘ └──────┬───────┘
             │                │               │
             ▼                ▼               ▼
    ┌─────────────────┐ ┌────────────┐ ┌──────────────┐
    │ Tripylon /      │ │ Palace of  │ │ Harem /      │
    │ Council Hall    │ │ Xerxes     │ │ Museum       │
    │                 │ │ (薛西斯宫)   │ │              │
    │  90-120s        │ │  120-150s  │ │  150-180s    │
    └─────────────────┘ └────────────┘ └──────────────┘
```

### 导航 UI

```html
<div id="navigation">
  <!-- 地图视图 (WebGL 叠加) -->
  <div id="mini-map" ref="miniMap">
    <canvas id="map-canvas" />          <!-- 2D 俯视图 -->
    <div class="map__marker" v-for="room in rooms"
         :style="{ left: room.mapX + '%', top: room.mapY + '%' }"
         @click="goToRoom(room.id)" />
  </div>

  <!-- 底部进度条 -->
  <div id="progress-bar" ref="progressBar">
    <div class="progress-bar__inner" :style="{ width: progress + '%' }" />
    <div class="progress-bar__labels">
      <span v-for="room in rooms" :key="room.id"
            :class="{ active: currentRoom === room.id }">
        {{ room.name }}
      </span>
    </div>
  </div>

  <!-- 导航控制按钮 -->
  <button class="nav-btn prev" @click="stepBackward">‹</button>
  <button class="nav-btn next" @click="stepForward">›</button>
</div>
```

### Vue 状态管理

```javascript
// Vue.js 控制全局导航状态
const app = new Vue({
  data: {
    currentRoom: 'gate',
    progress: 0,
    rooms: [
      { id: 'gate', name: 'Gate of All Nations', range: [0, 30] },
      { id: 'apadana', name: 'Apadana Palace', range: [30, 60] },
      { id: 'throne', name: 'Throne Hall', range: [60, 90] },
      { id: 'tripylon', name: 'Tripylon', range: [90, 120] },
      { id: 'xerxes', name: 'Palace of Xerxes', range: [120, 150] },
      { id: 'harem', name: 'Harem / Museum', range: [150, 180] },
    ],
    activeArtifact: null,
  },

  methods: {
    // 导航到指定房间
    goToRoom(roomId) {
      const room = this.rooms.find(r => r.id === roomId)
      const midTime = (room.range[0] + room.range[1]) / 2
      camera.setProgress(midTime / camera.duration)
      transitionToRoom(roomId)
    },

    // 前进/后退
    stepForward() { camera.stepForward(2) },
    stepBackward() { camera.stepBackward(2) },

    // 文物交互面板
    showArtifact(artifact) {
      this.activeArtifact = artifact
      pauseExperience()   // 暂停场景（相机停在当前位置）
    },

    closeArtifact() {
      this.activeArtifact = null
      resumeExperience()
    },
  },
})
```

---

## 二、文物交互系统

### 19 件文物

文物按交互方式分为三类：

| 类别 | 格式 | 示例 |
|---|---|---|
| **3D 文物** | WebGL 模型 | 金碗、浮雕、雕像（可在场景中旋转查看） |
| **图像文物** | 高清照片 | 铭文板、陶器碎片 |
| **音频文物** | 旁白 + 音效 | 历史解说、古波斯音乐复原 |

### 发现流程

```
用户沿相机路径前进
      │
      ▼
到达文物触发点 (由 Unity Timeline 中的 Event Marker 定义)
      │
      ▼
UI 提示 "发现文物" (+ 屏幕闪烁/音效)
      │
      ▼
用户点击查看 ──→ 弹出面板
      │              │
      │              ├─ 文物 3D 模型 (可旋转/缩放)
      │              ├─ 文字介绍
      │              ├─ 音频播放按钮
      │              └─ "Past & Present" 切换
      │
      ▼
用户关闭面板 → 体验继续
```

### Unity Timeline Event Markers → JSON

```csharp
// Unity 中在 Timeline 上放置 Event Marker
// 导出为 JSON 中的 artifact triggers

[System.Serializable]
public class ArtifactTrigger {
  public string id;           // "foundation_plate"
  public string roomId;       // "gate"
  public float triggerTime;   // 15.0 (seconds)
  public string type;         // "3d" | "image" | "audio"
  public string modelUrl;     // "/artifacts/foundation_plate.glb"
  public string imageUrl;     // "/artifacts/plate_photo.jpg"
  public string audioUrl;     // "/audio/plate_narration.mp3"
  public float duration;      // 30 (暂停秒数)
}
```

### WebGL 端检测

```javascript
// 渲染循环中检测文物触发
function updateArtifacts(currentTime) {
  currentRoom.artifacts.forEach(artifact => {
    const timeDiff = Math.abs(currentTime - artifact.triggerTime)

    if (timeDiff < 0.5 && !artifact.triggered) {
      artifact.triggered = true
      showArtifactNotification(artifact)
    }
  })
}
```

---

## 三、"Past & Present" 古今对比

这是交互设计中的亮点功能：**滑动切换文物"古代复原状态"vs"现代遗留状态"**。

### 交互

```html
<div class="artifact-compare" v-if="activeArtifact">
  <div class="compare__image-wrapper" ref="compareWrapper">
    <!-- 现代照片 (底层) -->
    <img :src="activeArtifact.presentImage" class="compare__present" />
    <!-- 古代复原 (上层, clip-path 控制) -->
    <img :src="activeArtifact.pastImage" class="compare__past"
         :style="{ clipPath: `inset(0 ${100 - sliderPos}% 0 0)` }" />
    <!-- 分隔滑块 -->
    <div class="compare__slider" :style="{ left: sliderPos + '%' }"
         @mousedown="startDrag" />
  </div>

  <div class="compare__caption">
    <span class="is-active" v-if="sliderPos > 50">Reconstruction</span>
    <span class="is-active" v-else>Present Day</span>
  </div>
</div>
```

### JS

```javascript
data() {
  return {
    sliderPos: 50,
    isDragging: false,
  }
},

methods: {
  startDrag(e) {
    this.isDragging = true
    const wrapper = this.$refs.compareWrapper
    const rect = wrapper.getBoundingClientRect()

    const onMove = (e) => {
      const x = e.clientX - rect.left
      this.sliderPos = Math.max(0, Math.min(100, (x / rect.width) * 100))
    }

    const onUp = () => {
      this.isDragging = false
      window.removeEventListener('mousemove', onMove)
      window.removeEventListener('mouseup', onUp)
    }

    window.addEventListener('mousemove', onMove)
    window.addEventListener('mouseup', onUp)
  },
},
```

### CSS

```css
.compare__image-wrapper {
  position: relative;
  overflow: hidden;
  cursor: col-resize;
}

.compare__past {
  position: absolute;
  inset: 0;
  clip-path: inset(0 50% 0 0);   /* 从右向左剪裁 */
  will-change: clip-path;
}

.compare__slider {
  position: absolute;
  top: 0; bottom: 0;
  width: 3px;
  background: white;
  cursor: col-resize;
  transform: translateX(-50%);
  z-index: 2;
}

.compare__slider::before {
  content: '⟷';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  border-radius: 50%;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
}
```

---

## 四、叙事设计决策

| 决策 | 原因 |
|---|---|
| **固定相机路径** | 控制叙事节奏，确保用户看到每个关键场景 |
| **非自由漫游** | 不需要 WASD 操作，降低门槛，适合博物馆观众 |
| **雾化转场** | 保持沉浸感，掩盖场景加载延迟 |
| **自动暂停 + 文物弹出** | 避免用户错过重要历史信息 |
| **可选自由探索** | 地图视图允许跳转，兼顾控制派和自由派用户 |

### 用户类型适配

```
被动观众 (默认路径):                         主动探索者 (可选):
  自动前进 ─── 发现文物 ─── 自动前进            地图选房间 ─── 自由浏览
       │                          │                │
       └── 也可点击查看 ──────────┘                └── 每件文物都可查看
```
