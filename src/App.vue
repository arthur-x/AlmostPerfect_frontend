<template>
  <div class="container">

    <div class="upload-area">
      <input
        type="file"
        ref="fileInput"
        @change="handleFolderSelect"
        webkitdirectory
        directory
        multiple
        hidden
      />
      <button class="upload-btn" @click="triggerFolderSelect">Choose folder</button>
      <button class="download-btn" @click="downloadScores" :disabled="!images.length">Download scores</button>
      <div v-if="images.length" class="counter">{{ images.length }} loaded / {{ scoredCount }} scored</div>
    </div>

    <div v-if="images.length" class="grid-container">
      <div 
        v-for="(image, index) in images" 
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
        <span class="icon">🖼️</span>
        <h3>No image loaded</h3>
        <p>Click the button above to choose a folder</p>
      </div>
    </div>

    <div v-if="showLightbox" class="lightbox" @click.self="closeLightbox">
      <button class="close-btn" @click="closeLightbox">&times;</button>
      <button class="nav-btn prev" @click="prevImage">&lt;</button>
      <button class="nav-btn next" @click="nextImage">&gt;</button>
      
      <div class="lightbox-content">
        <div class="score-display">
          Score:  {{ currentScore > 0 ? `+${currentScore}` : currentScore }}
          <div class="score-controls">
            <button @click="changeScore(-1)" :disabled="currentScore <= -5">👎</button>
            <button @click="changeScore(1)" :disabled="currentScore >= 5">👍</button>
          </div>
        </div>
        <img 
          :src="currentImage.url" 
          :alt="currentImage.name"
          class="lightbox-image"
        />
        <div class="image-info">
          {{ currentImage.name }} ({{ currentIndex + 1 }}/{{ images.length }})
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
      hoverIndex: -1,
      scores: {},
    }
  },
  computed: {
    currentImage() {
      return this.images[this.currentIndex] || {}
    },
    currentScore() {
      return this.scores[this.currentImage.name] || 0
    },
    scoredCount() {
      return Object.values(this.scores).length;
    }
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
      const current = this.scores[this.currentImage.name] || 0
      const newScore = Math.min(Math.max(current + delta, -5), 5)
      this.scores[this.currentImage.name] = newScore
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
      this.showLightbox = true
      document.body.style.overflow = 'hidden'
    },

    closeLightbox() {
      this.showLightbox = false
      document.body.style.overflow = ''
    },

    nextImage() {
      this.changeScore(0)
      this.currentIndex = (this.currentIndex + 1) % this.images.length
    },

    prevImage() {
      this.changeScore(0)
      this.currentIndex = (this.currentIndex - 1 + this.images.length) % this.images.length
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
  padding: 2rem;
  width: 100%;
}

.upload-area {
  text-align: center;
  margin-bottom: 3rem;
  
  .counter {
    margin: 2rem 0;
  }

  .upload-btn {
    background: #ed9f22;
    color: white;
    padding: 1rem 2rem;
    margin: 0 1rem;
    border-radius: 50px;
    border: none;
    font-size: 1.1rem;
    transition: all 0.2s ease;
    cursor: pointer;
  }

  .download-btn {
    background-color: #2196F3;
    color: white;
    padding: 1rem 2rem;
    margin: 0 1rem;
    border-radius: 50px;
    border: none;
    font-size: 1.1rem;
    transition: all 0.2s ease;
    cursor: pointer;
    &:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
  }
}

.grid-container {
  display: grid;
  gap: 1rem;
  padding: 0;
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
  top: 2rem;
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
  
  .icon {
    font-size: 4rem;
    margin-bottom: 1rem;
  }
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
  padding: 8px 16px;
  margin: 0 0.5rem;
  border-radius: 20px;
  cursor: pointer;
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
</style>