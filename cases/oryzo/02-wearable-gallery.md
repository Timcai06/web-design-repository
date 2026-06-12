# Wearable Gallery — 水平滚动画廊

---

## 一、结构

```html
<section id="wearable" class="section">
  <div class="section__inner">
    <div id="wearable-gallery">
      <div id="wearable-gallery-preview">
        <div class="wearable-gallery-preview__image">
          <img id="wearable-preview-img" />
        </div>
      </div>
      <div id="wearable-gallery-move-container">
        <div class="wearable-gallery-thumbs-wrapper">
          <div class="wearable-gallery-thumbs-move-container">
            <!-- 由 JS 生成 -->
            <div class="wearable-gallery-item"
                 data-id="intro"
                 data-type="intro"
                 data-file="intro.webp">
              <div class="wearable-gallery-item__inner">
                <img src="intro.webp" />
              </div>
            </div>
            <div class="wearable-gallery-item"
                 data-id="yoga"
                 data-type="image"
                 data-file="yoga.webp">...</div>
            <div class="wearable-gallery-item"
                 data-id="shoulder"
                 data-type="image"
                 data-file="shoulder.webp">...</div>
            <div class="wearable-gallery-item"
                 data-id="bikini"
                 data-type="image"
                 data-file="bikini.webp">...</div>
            <div class="wearable-gallery-item"
                 data-id="glasses"
                 data-type="image"
                 data-file="glasses.webp">...</div>
            <div class="wearable-gallery-item"
                 data-id="bite"
                 data-type="image"
                 data-file="bite.webp">...</div>
            <div class="wearable-gallery-item"
                 data-id="pocket"
                 data-type="image"
                 data-file="pocket.webp">...</div>
            <div class="wearable-gallery-item"
                 data-id="outro"
                 data-type="outro"
                 data-file="outro.webp">...</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>
```

---

## 二、布局

```
┌────────────────────────────────────────────────────────┐
│                    #wearable section                     │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │           #wearable-gallery-preview               │   │
│  │           (大图预览, 固定区域, 100vh)              │   │
│  │                                                   │   │
│  │              ┌──────────────┐                    │   │
│  │              │              │                    │   │
│  │              │   大图图片    │                    │   │
│  │              │              │                    │   │
│  │              └──────────────┘                    │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │        #wearable-gallery-move-container            │   │
│  │        (水平可滚动, 固定在底部)                    │   │
│  │                                                   │   │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐    │   │
│  │  │intro │ │ yoga │ │should│ │bikini│ │glasses│ →  │   │
│  │  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘    │   │
│  └──────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────┘
```

---

## 三、交互机制

### 自动滚动 + 拖拽

```javascript
class WearableGallery {
  constructor() {
    this.container = document.getElementById('wearable-gallery-move-container')
    this.track = this.container.querySelector('.wearable-gallery-thumbs-move-container')
    this.preview = document.getElementById('wearable-preview-img')

    this.items = [...this.track.querySelectorAll('.wearable-gallery-item')]
    this.currentIndex = 0
    this.isDragging = false
    this.startX = 0
    this.scrollLeft = 0

    // 自动播放 (每 3s 前进一帧)
    this.autoPlayInterval = setInterval(() => this.next(), 3000)

    this.initDrag()
    this.initScroll()
  }

  initDrag() {
    this.container.addEventListener('mousedown', (e) => {
      this.isDragging = true
      this.startX = e.pageX - this.container.offsetLeft
      this.scrollLeft = this.track.scrollLeft
      clearInterval(this.autoPlayInterval)  // 拖拽时暂停自动播放
    })

    window.addEventListener('mousemove', (e) => {
      if (!this.isDragging) return
      e.preventDefault()
      const x = e.pageX - this.container.offsetLeft
      const walk = (x - this.startX) * 1.5
      this.track.scrollLeft = this.scrollLeft - walk
      this.updateActiveFromScroll()
    })

    window.addEventListener('mouseup', () => {
      this.isDragging = false
      this.snapToNearest()
    })
  }
}
```

### 选中项 → 更新大图

```javascript
  selectItem(index) {
    this.currentIndex = index
    const item = this.items[index]

    // 高亮 thumb
    this.items.forEach(el => el.classList.remove('is-active'))
    item.classList.add('is-active')

    // 更新大图 (渐入)
    const imgSrc = item.dataset.file
    const newImg = new Image()
    newImg.onload = () => {
      this.preview.style.opacity = '0'
      this.preview.src = imgSrc
      // 触发 reflow 后渐入
      requestAnimationFrame(() => {
        this.preview.style.opacity = '1'
      })
    }
    newImg.src = `/images/wearable/${imgSrc}`

    // 滚动 thumb 到可见区域
    item.scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' })
  }
```

### Magic Scroll 集成

```javascript
// 在 Lenis scroll 中，当 wearable section 可见时启用水平滚动
const wearableSection = document.getElementById('wearable')

ScrollTrigger.create({
  trigger: wearableSection,
  start: 'top bottom',
  end: 'bottom top',
  onEnter: () => { this.isActive = true },
  onLeave: () => { this.isActive = false },
  onEnterBack: () => { this.isActive = true },
  onLeaveBack: () => { this.isActive = false },
})

// 覆写鼠标滚轮事件，在 gallery 区域切换水平滚动
wearableSection.addEventListener('wheel', (e) => {
  if (!this.isActive) return
  if (Math.abs(e.deltaY) > Math.abs(e.deltaX)) {
    e.preventDefault()
    this.track.scrollLeft += e.deltaY
    this.updateActiveFromScroll()
  }
}, { passive: false })
```

---

## 四、图片预加载策略

```javascript
// 所有 gallery 图片提前加载
function preloadGalleryImages() {
  const files = document.querySelectorAll('[data-file]')
  files.forEach(el => {
    const img = new Image()
    img.src = `/images/wearable/${el.dataset.file}`
  })
}

preloadGalleryImages()
```

---

## 五、性能注意

- 所有 wearable 图片使用 WebP 格式 (intro.webp, yoga.webp 等)
- 大图预览用 CSS `transition: opacity 0.4s ease` 实现交叉淡入，避免闪烁
- 拖拽使用 `requestAnimationFrame` 不做额外计算
- 自动播放在用户交互时暂停 (clearInterval on drag)
