<template>
  <div class="container">
    <div class="sticky-bar">
      <div class="banner">AlmostPerfect</div>
      <div class="controls">
        <div class="counter-container" v-if="images.length">
          <div class="counter">
            {{ images.length }} loaded / {{ scoredCount }} scored
          </div>
          <label class="filter-checkbox">
            <input type="checkbox" v-model="showOnlyUnscored">
            <span>only show unscored</span>
          </label>
        </div>
        <input
          type="file"
          ref="fileInput"
          @change="handleFolderSelect"
          webkitdirectory
          directory
          multiple
          hidden
        />
        <button class="upload-btn" @click="triggerFolderSelect">
          <span><svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#e3e3e3"><path d="M160-160q-33 0-56.5-23.5T80-240v-480q0-33 23.5-56.5T160-800h207q16 0 30.5 6t25.5 17l57 57h320q33 0 56.5 23.5T880-640v400q0 33-23.5 56.5T800-160H160Zm0-80h640v-400H447l-80-80H160v480Zm0 0v-480 480Z"/></svg></span>
        </button>
        <button 
          class="download-btn"
          @click="downloadScores"
          :disabled="!Object.keys(scores).length"
        >
          <span><svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#e3e3e3"><path d="M480-337q-8 0-15-2.5t-13-8.5L308-492q-12-12-11.5-28t11.5-28q12-12 28.5-12.5T365-549l75 75v-286q0-17 11.5-28.5T480-800q17 0 28.5 11.5T520-760v286l75-75q12-12 28.5-11.5T652-548q11 12 11.5 28T652-492L508-348q-6 6-13 8.5t-15 2.5ZM240-160q-33 0-56.5-23.5T160-240v-80q0-17 11.5-28.5T200-360q17 0 28.5 11.5T240-320v80h480v-80q0-17 11.5-28.5T760-360q17 0 28.5 11.5T800-320v80q0 33-23.5 56.5T720-160H240Z"/></svg></span>
        </button>
      </div>
    </div>

    <div v-if="filteredImages.length" class="grid-container">
      <div 
        v-for="(image, index) in filteredImages" 
        :key="index" 
        class="grid-item"
        @click="openLightbox(index)"
      >
        <div class="image-wrapper">
          <img 
            :src="image.url" 
            :alt="image.name"
            class="grid-image"
            loading="lazy"
          />
        </div>
        <div class="score-badge" :class="{ positive: scores[image.name] > 0, negative: scores[image.name] < 0 }" v-if="shouldShowBadge(image)">
          {{ formatScore(image) }}
        </div>
      </div>
    </div>

    <div v-else class="empty-state">
      <div class="empty-content">
        <span><svg xmlns="http://www.w3.org/2000/svg" height="48px" viewBox="0 -960 960 960" width="48px" fill="#e3e3e3"><path d="M180-120q-24 0-42-18t-18-42v-600q0-24 18-42t42-18h600q24 0 42 18t18 42v600q0 24-18 42t-42 18H180Zm0-60h600v-600H180v600Zm0 0v-600 600Zm86-97h429q8.5 0 12.75-8t-.75-16L590-457q-5-6-12-6t-12 6L446-302l-81-111q-5-6-12-6t-12 6l-86 112q-6 8-1.75 16t12.75 8Zm74.12-293q20.88 0 35.38-14.62 14.5-14.62 14.5-35.5 0-20.88-14.62-35.38-14.62-14.5-35.5-14.5-20.88 0-35.38 14.62-14.5 14.62-14.5 35.5 0 20.88 14.62 35.38 14.62 14.5 35.5 14.5Z"/></svg></span>
        <h3>No image</h3>
      </div>
    </div>

    <div v-if="showLightbox" class="lightbox" @click.self="closeLightbox">
      <button class="close-btn" @click="closeLightbox">&times;</button>
      <button class="nav-btn prev" @click="prevImage">&lt;</button>
      <button class="nav-btn next" @click="nextImage">&gt;</button>
      
      <div class="lightbox-content">
        <div class="score-display">
          Score:  {{ tempScore > 0 ? `+${tempScore}` : tempScore }}
          <div class="score-controls">
            <button @click="changeScore(-1)" :disabled="tempScore <= -5"><svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#e3e3e3"><path d="M120-320q-32 0-56-24t-24-56v-80q0-7 2-15t4-15l120-282q9-20 30-34t44-14h440v520L440-82q-15 15-35.5 17.5T365-72q-19-10-28-28t-4-37l45-183H120Zm480-34v-406H240L120-480v80h360l-54 220 174-174Zm200-486q33 0 56.5 23.5T880-760v360q0 33-23.5 56.5T800-320H680v-80h120v-360H680v-80h120Zm-200 80v406-406Z"/></svg></button>
            <button @click="changeScore(1)" :disabled="tempScore >= 5"><svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#e3e3e3"><path d="M840-640q32 0 56 24t24 56v80q0 7-2 15t-4 15L794-168q-9 20-30 34t-44 14H280v-520l240-238q15-15 35.5-17.5T595-888q19 10 28 28t4 37l-45 183h258Zm-480 34v406h360l120-280v-80H480l54-220-174 174ZM160-120q-33 0-56.5-23.5T80-200v-360q0-33 23.5-56.5T160-640h120v80H160v360h120v80H160Zm200-80v-406 406Z"/></svg></button>
          </div>
        </div>
        <img 
          :src="currentImage.url" 
          :alt="currentImage.name"
          class="lightbox-image"
        />
        <div class="image-info">
          {{ currentImage.name }} ({{ currentIndex + 1 }}/{{ filteredImages.length }})
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      images: [],
      showLightbox: false,
      currentIndex: 0,
      scores: {},
      tempScore: 0,
      showOnlyUnscored: false,
    }
  },
  computed: {
    currentImage() {
      return this.filteredImages[this.currentIndex] || {}
    },
    scoredCount() {
      return Object.values(this.scores).length;
    },
    filteredImages() {
      return this.showOnlyUnscored ? this.images.filter(img => !(img.name in this.scores)) : this.images
    },
  },
  methods: {
    shouldShowBadge(image) {
      return this.scores[image.name] !== undefined
    },

    formatScore(image) {
      const score = this.scores[image.name]
      return score > 0 ? `+${score}` : score
    },

    changeScore(delta) {
      this.tempScore = Math.min(Math.max(this.tempScore + delta, -5), 5)
    },

    saveScore() {
      this.scores[this.currentImage.name] = this.tempScore
    },

    downloadScores() {
      const data = JSON.stringify(this.scores)
      const blob = new Blob([data], { type: 'application/json' })
      const link = document.createElement('a')
      link.href = URL.createObjectURL(blob)
      link.download = 'scores.json'
      link.click()
    },

    triggerFolderSelect() {
      this.$refs.fileInput.click()
    },

    async handleFolderSelect(event) {
      const files = Array.from(event.target.files)
      const scoreFile = files.find(f => f.name === 'scores.json')
      
      if (scoreFile) {
        const scoreData = await new Promise(resolve => {
          const reader = new FileReader()
          reader.onload = e => resolve(JSON.parse(e.target.result))
          reader.readAsText(scoreFile)
        })
        this.scores = { ...scoreData }
      } else {
        this.scores = {}
      }

      const imageFiles = files.filter(file => file.type.startsWith('image/'))

      this.images = []

      const promises = imageFiles.map(file => {
        return new Promise((resolve) => {
          const reader = new FileReader()
          reader.onload = (e) => {
            resolve({
              name: file.name,
              url: e.target.result
            })
          }
          reader.readAsDataURL(file)
        })
      })

      this.images = await Promise.all(promises)
    },

    openLightbox(index) {
      this.currentIndex = index
      this.tempScore = this.scores[this.currentImage.name] || 0
      this.showLightbox = true
      document.body.style.overflow = 'hidden'
    },

    closeLightbox() {
      this.showLightbox = false
      document.body.style.overflow = ''
    },

    nextImage() {
      this.saveScore()
      if (!this.filteredImages.length) {
        this.closeLightbox()
        return
      }
      if (!this.showOnlyUnscored) {
        this.currentIndex += 1
      }
      this.currentIndex %= this.filteredImages.length
      this.tempScore = this.scores[this.currentImage.name] || 0
    },

    prevImage() {
      this.saveScore()
      if (!this.filteredImages.length) {
        this.closeLightbox()
        return
      }
      this.currentIndex = (this.currentIndex - 1 + this.filteredImages.length) % this.filteredImages.length
      this.tempScore = this.scores[this.currentImage.name] || 0
    },

    handleKeydown(e) {
      if (this.showLightbox) {
        if (e.key === 'Escape') this.closeLightbox()
        if (e.key === 'ArrowRight') this.nextImage()
        if (e.key === 'ArrowLeft') this.prevImage()
        if (e.key === 'ArrowDown') this.changeScore(-1)
        if (e.key === 'ArrowUp') this.changeScore(1)
      }
    }
  },
  mounted() {
    window.addEventListener('keydown', this.handleKeydown)
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeydown)
  }
}
</script>

<style lang="scss" scoped>

.container {
  max-width: none;
  margin: 0;
  width: 100%;
}

.grid-container {
  display: grid;
  gap: 1rem;
  padding: 2rem;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  width: 100%;
}

.grid-item {
  position: relative;
  min-height: 160px;
  aspect-ratio: 1;
  border-radius: 8px;
  overflow: hidden;
  transition: transform var(--transition-fast);
  cursor: zoom-in;
  width: 100%;
  
  &:hover {
    transform: scale(1.03);
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
  }
}

.grid-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: inherit;
}

.lightbox {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.9);
  display: grid;
  place-items: center;
  z-index: 1000;
  backdrop-filter: blur(10px);
  
  &-content {
    max-width: 90vw;
    max-height: 90vh;
    position: relative;
  }
  
  &-image {
    max-height: 80vh;
    object-fit: contain;
    border-radius: 8px;
    animation: zoomIn 0.3s ease;
  }
}

@keyframes zoomIn {
  from { transform: scale(0.9); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.close-btn {
  position: absolute;
  top: 1.5rem;
  right: 2rem;
  background: none;
  border: none;
  color: white;
  font-size: 3rem;
  cursor: pointer;
  transition: transform 0.2s ease;
  
  &:hover {
    transform: rotate(90deg);
  }
}

.nav-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
  font-size: 2rem;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  
  &:hover {
    background: rgba(255, 255, 255, 0.3);
  }
  
  &.prev { left: 2rem; }
  &.next { right: 2rem; }
}

.image-info {
  position: absolute;
  bottom: -2rem;
  width: 100%;
  text-align: center;
  color: white;
  font-size: 1.2rem;
}

.empty-state {
  display: grid;
  place-items: center;
  min-height: 60vh;
  text-align: center;
  color: var(--text-dark);
}

.score-display {
  font-size: 1.2rem;
  position: absolute;
  top: -3rem;
  left: 0;
  width: 100%;
  color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.score-controls button {
  background: rgba(255,255,255,0.2);
  border: none;
  color: white;
  padding: 5px 10px;
  margin: 0 0.5rem;
  border-radius: 10px;
  cursor: pointer;

  &:hover {
    background: rgba(255, 255, 255, 0.3);
  }
}

.score-controls button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.score-badge {
  position: absolute;
  top: 8px;
  right: 8px;
  background: rgba(25, 25, 25, 0.9);
  color: #fff;
  padding: 6px 12px;
  border-radius: 16px;
  font-size: 14px;
  font-weight: bold;
  z-index: 2;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: opacity 0.3s ease;
}

.score-badge.positive {
  background: #4CAF50;
}

.score-badge.negative {
  background: #f44336;
}

.image-wrapper {
  position: relative;
}

.filter-checkbox {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  user-select: none;
  color: gray;
}

.filter-checkbox input {
  width: 16px;
  height: 16px;
  accent-color: darkgray;
}

.sticky-bar {
  position: sticky;
  top: 0;
  z-index: 1000;
  background: rgba(33, 33, 33, 0.95);
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
  padding: 12px 0;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.banner {
  margin: 0 2rem;
  font-size: x-large;
  font-style: italic;
  color: pink;
}

.controls {
  margin: 0 2rem;
  display: flex;
  align-items: center;
  gap: 20px;
}

.upload-btn {
  background: rgba(128, 128, 128, 0.2) !important;
  padding: 10px 24px !important;
  border-radius: 8px !important;
  border: none;
  font-size: 1rem !important;
  cursor: pointer;
}

.counter-container {
  display: flex;
  gap: 2rem;
}

.counter {
  color: gray;
  white-space: nowrap;
}

.download-btn {
  background: rgba(128, 128, 128, 0.2) !important;
  padding: 10px 24px !important;
  border-radius: 8px !important;
  border: none;
  font-size: 1rem !important;
  cursor: pointer;
}

.download-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>