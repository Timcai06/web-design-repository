# Rive + ScrollTrigger — 核心创新

landonorris.com 的核心技术创新：将 Rive 动画进度（`scroll` input）绑定到 GSAP ScrollTrigger，实现**滚动驱动的矢量动画**，以及**滚动入场播放**。

---

## 一、完整架构

```
HTML canvas[data-rive-object]
  │
  ├─ data-rive-file="reef"              → B4()
  ├─ data-rive-file="signature"          → E4()
  ├─ data-rive-file="circuits"           → I4()
  ├─ data-rive-file="btn-ui"             → C4() / iR()
  └─ data-rive-file="phrases"            → K4()
       │
       ▼
  kI() 预加载 .riv → DB[]
       │
       ▼
  根据 state machine 后缀分流：
  ┌─────────────┬───────────────┬──────────────────────┐
  │  后缀        │ 函数          │ 行为                  │
  ├─────────────┼───────────────┼──────────────────────┤
  │ _scroll     │ oR()          │ scroll input 驱动     │
  │ _play       │ CH()          │ 滚动入场播放一次       │
  │ (无后缀)    │ —             │ 立即自动播放           │
  └─────────────┴───────────────┴──────────────────────┘
```

---

## 二、两条核心函数

### oR() — Scroll-Driven（`_scroll` 后缀）

将 Rive 状态机的 `scroll` input（0–1000）映射到 ScrollTrigger 进度（0–1），实现精确的 scrub 驱动。

```javascript
function oR(A, Q, B) {
  let E = A.getAttribute("data-rive-state-machine"),
      I = null,
      C = Q.stateMachineInputs(E);
  if (C) I = C.find(K => K.name === "scroll");

  let K = A.getAttribute("data-rive-scrolltrigger-start"),
      J = A.getAttribute("data-rive-scrolltrigger-end"),
      U = m.timeline({
        paused: true,
        onUpdate: function() {
          if (I) I.value = m.utils.clamp(0, 1, this.progress()) * 1000;
        }
      });
  U.to({}, { duration: 1 });

  ScrollTrigger.create({
    trigger: B,
    start: K || "top bottom",
    end: J || "bottom top",
    scrub: 0.5,
    onUpdate: H => {
      U.progress(H.progress);
      if (!Q.isPlaying) Q.play();
    },
    onEnter: () => {
      Q.play();
      if (I) I.value = 0;
    },
    onLeave: () => {
      if (I) I.value = 1000;
      setTimeout(() => Q.pause(), 50);
    },
    onEnterBack: () => { Q.play(); },
    onLeaveBack: () => {
      if (I) I.value = 0;
      setTimeout(() => Q.pause(), 50);
    }
  });
}
```

### CH() — Play-on-Scroll-Enter（`_play` 后缀）

等滚动到指定位置才播放动画，播放一次即止。

```javascript
function CH(A, Q, B, E = "top 80%") {
  A.style.opacity = "0";
  ScrollTrigger.create({
    trigger: B,
    start: E,
    onEnter: () => {
      A.style.opacity = "1";
      Q.play();
    },
    once: true
  });
}
```

---

## 三、HTML 数据属性模式

```html
<!-- oR: scroll-driven，状态机后缀 _scroll -->
<canvas
  data-rive-object
  data-rive-file="reef"
  data-rive-artboard="helmet-reef"
  data-rive-state-machine="helmet-reef_scroll"
  data-rive-scrolltrigger-start="top 80%"
  data-rive-scrolltrigger-end="bottom 20%"
/>

<!-- CH: 入场播放，状态机后缀 _play -->
<canvas
  data-rive-object
  data-rive-file="reef"
  data-rive-artboard="helmet-reef"
  data-rive-state-machine="helmet-reef_play"
/>

<!-- 无后缀：立即 autoplay -->
<canvas
  data-rive-object
  data-rive-file="circuits"
  data-rive-artboard="circuits"
  data-rive-state-machine="circuits"
/>
```

---

## 四、各 Rive 对象的初始化函数

| 函数 | 匹配 `data-rive-file` | 功能 |
|------|----------------------|------|
| **B4()** | `"reef"` | 检测状态机后缀 → `_scroll` 走 `oR()`，`_play` 走 `CH()`，无后缀 autoplay。额外处理 `off-icons` |
| **E4()** | `"signature"` | `signature_scroll` → `oR()` 绑定 `hero-rive-tracker`；`signature_play` → `CH()` |
| **I4()** | `"circuits"` | 处理 `scroll` input、`data-rive-input-track`（电路选择）、`data-rive-circuit-hover`、`data-rive-input-color` / `data-rive-input-weight`（boolean） |
| **C4()** | `"btn-ui"` | 箭头动画，查找 `hover` boolean input，绑定 `data-btn-rive-hover` 元素的 mouseenter/mouseleave/touch |
| **iR()** | `"btn-ui"` | 汉堡菜单，artboard `hamburger`，inputs: `hover`/`close`/`color-transparent`/`color-white` |
| **K4()** | `"phrases"` | 检查 `data-rive-scrolltrigger="true"` 决定是否滚动触发 |
| **nR()** | `"ln4"` | Logo 动画，artboard `logo`，inputs: `logo-active`/`hover`/`color_dark-green`/`color_white`/`color_lime` |

---

## 五、预加载机制

```javascript
const p6 = {
  reef: "https://cdn.landonorris.com/assets/rive/reef.riv",
  signature: "https://cdn.landonorris.com/assets/rive/signature.riv",
  circuits: "https://cdn.landonorris.com/assets/rive/circuits.riv",
  "btn-ui": "https://cdn.landonorris.com/assets/rive/btn-ui.riv",
  phrases: "https://cdn.landonorris.com/assets/rive/phrases.riv",
  ln4: "https://cdn.landonorris.com/assets/rive/ln4.riv"
};

let JI = 0;    // 加载计数器
const DB = {};  // 存储已加载文件：DB["reef"] → Rive instance

function kI() {
  // 遍历 p6，加载 .riv 文件到 DB
  // 每加载完一个 JI++，全部完成发送 allriveloaded 事件
}

// 各 init 函数等待 allriveloaded，若 kI() 未启动则调用之
```

---

## 关键设计原则

1. **Rive 作渲染引擎，GSAP 作进度引擎** — Rive 不负责 timeline，只负责渲染单帧；进度由 ScrollTrigger scrub 控制
2. **状态机约定 > 配置** — 用 `_scroll` / `_play` 后缀名决定绑定方式，减少手工配置
3. **`scroll` input 范围 0–1000** — 映射 GSAP progress 0–1 × 1000，实现高精度
4. **事件防御** — `onLeave`/`onLeaveBack` 用 `setTimeout` 延迟 pause，避免帧闪烁
5. **CDN + 预加载** — 所有 `.riv` 文件通过 `kI()` 统一预加载到 `DB`，所有 init 函数只从 `DB` 取实例
