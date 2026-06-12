# 文字拆分动画系统

landonorris.com 使用 **GSAP SplitText 3.13.0** 插件（非手写 `splitToChars`），插件内嵌于 `lando.OFF+BRAND.gold-android-fix-03.js`（~450KB 主包）中，通过 `E1.registerPlugin(TA, u6, CI)` 注册（`CI` 为 SplitText）。

---

## 一、HTML 属性驱动

| 属性 | 值 | 效果 |
|---|---|---|
| `split-text` | `chars` | 拆分为 `<span class="char">` |
| `split-text` | `lines` | 行包裹 `<div class="line">` |
| `split-text` | `lines,chars` | 同时拆分行和字符 |
| `data-anim-high` | `right, lime` | 从右侧入场 + lime 色 |
| `data-anim-high` | `left, dark-green-tint-1, 200` | 左侧入场 + 200px 偏移 |
| `data-anim-high` | `right, lime-off` | 右侧入场 + lime-off 色 |

---

## 二、GSAP SplitText 核心用法

```javascript
// 拆分字符 + 行
const split = new SplitText(target, {
  type: 'chars,lines',
  // charsClass: 'char',      // 默认 .char
  // linesClass: 'line',      // 默认 .line
  // wordsClass: 'word',
  smartWrap: true,             // 保留单词边界
  onSplit: (els) => {
    els.chars.forEach(el => el.setAttribute('aria-hidden', 'true'))
    target.setAttribute('aria-label', target.textContent)
  }
})
```

**实际注册代码示意**（反编译简化）：

```javascript
// 位于主 bundle 内，定义 CI 类
class CI {
  constructor(e, t) {
    this.init(e, t)
  }
  // type: 'chars,words,lines'
  // 内部基于 getBoundingClientRect Y 变化检测换行
}

// 注册
E1.registerPlugin(TA, u6, CI)
```

---

## 三、CSS 管线

```css
/* 行容器：clip-path 裁剪实现从下至上 reveal */
[split-text]:not([data-oval-scroll]) .line {
  position: relative;
  clip-path: polygon(0 -2%, 0 94%, 100% 94%, 100% -2%);
  white-space: nowrap;
}

/* 字符：inline-block 确保形变定位 */
[split-text] .char {
  display: inline-block;
}

/* data-anim-high：行自适应宽度 */
[data-anim-high] .line {
  width: auto !important;
  display: inline-block !important;
}

/* 保留换行符 */
[split-text] {
  white-space: pre-line;
}
```

`clip-path: polygon()` 的底部 94% 截断让行在动画前只有顶部 4% 可见，配合 GSAP tween 将 clip-path 过渡到 `polygon(0 -2%, 0 102%, 100% 102%, 100% -2%)` 实现从下至上 reveal。

---

## 四、动画执行（`data-anim-high` 模式）

```javascript
// 从属性解析方向/颜色/偏移
gsap.fromTo(lines, {
  y: dir === 'left' ? offset || 80 : dir === 'right' ? -80 : 0,
  x: dir === 'left' ? -80 : dir === 'right' ? 80 : 0,
  opacity: 0,
  clipPath: 'polygon(0 94%, 0 94%, 100% 94%, 100% 94%)',
}, {
  y: 0, x: 0, opacity: 1,
  clipPath: 'polygon(0 -2%, 0 102%, 100% 102%, 100% -2%)',
  duration: 1.2,
  stagger: { each: 0.08, from: 'start' },
  ease: 'power4.out',
})
```

---

## 五、运用位置

| 区域 | split-text | data-anim-high |
|---|---|---|
| 导航栏链接 | `chars` | — |
| Next Race 标题 | `lines` | `right, lime` |
| "Message from Lando" | `chars` | `data-hero-anim="msg"` |
| Impact 加粗段落 | `lines` | Brier 字体 |
| On Track / Off Track | `lines` | ✓ |
| 头盔区标题 | `lines` | — |
| 底部 enquiries / 社交 | `chars` | — |
