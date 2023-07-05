<template>
  <div class="body">
    <div class="toolbar">
      <button class="btn" @click="selectedTool = 'pen'">Карандаш</button>
      <button class="btn" @click="selectedTool = 'felt-pen'">Фломастер</button>
      <button class="btn" @click="selectedTool = 'eraser'">Ластик</button>
      <button class="btn" @click="selectedTool = 'cleanup'">Очистить</button>
      <button class="btn" @click="saveToGallery">Сохранить</button>
      <input type="color" v-model="color" />
      <input type="range" v-model="hardness" min="0" max="100" />
    </div>
  <canvas ref="canvas" @mousedown="startDrawing" @mousemove="continueDrawing" @mouseup="stopDrawing"></canvas>
    <div class="gallery">
      <h2>Галерея</h2>
      <div class="images">
        <div v-for="(image, index) in gallery" :key="index" style="width: 640px; height: 480px">
          <img :src="image" />
          <div class="actions">
            <a :href="image" download>Скачать</a>
            <button @click="deleteImage(index)" class="delete">Удалить</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedTool: 'pen',
      color: '#000000',
      hardness: 50,
      context: null,
      isDrawing: false,
      gallery: []
    }
  },
  mounted() {
    this.context = this.$refs.canvas.getContext('2d')
    this.resizeCanvas()
    window.addEventListener('resize', this.resizeCanvas)
    this.gallery = JSON.parse(localStorage.getItem('gallery')) || []
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.resizeCanvas)
  },
  methods: {
    startDrawing(event) {
      this.isDrawing = true
      this.context.beginPath()
      this.context.moveTo(event.clientX, event.clientY)
    },
    continueDrawing(event) {
      if (!this.isDrawing) return
      this.context.lineTo(event.clientX, event.clientY)
      if (this.selectedTool === 'pen') {
        this.context.strokeStyle = this.color
        this.context.lineWidth = this.hardness /50
        this.context.stroke()
      } else if (this.selectedTool === 'felt-pen') {
        this.context.fillStyle = this.color
        this.context.globalAlpha = this.hardness / 60
        this.context.fillRect(event.clientX, event.clientY, this.hardness, this.hardness)
      } else if (this.selectedTool === 'eraser') {
        this.context.clearRect(event.clientX, event.clientY, this.hardness, this.hardness)
      }
    },
    stopDrawing() {
      this.isDrawing = false
    },
    resizeCanvas() {
      this.$refs.canvas.width = window.innerWidth
      this.$refs.canvas.height = window.innerHeight
    },
    cleanup() {
      this.context.clearRect(0, 0, this.$refs.canvas.width, this.$refs.canvas.height)
    },
    saveToGallery() {
      const image = this.$refs.canvas.toDataURL()
      this.gallery.push(image)
      localStorage.setItem('gallery', JSON.stringify(this.gallery))
    },
    deleteImage(index) {
      this.gallery.splice(index, 1)
      localStorage.setItem('gallery', JSON.stringify(this.gallery))
    }
  },
  watch: {
    selectedTool(newValue) {
      if (newValue === 'cleanup') {
        this.cleanup()
      }
    }
  }
}
</script>

<style>
.toolbar {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.toolbar > * {
  margin-right: 10px;
}
.gallery {
  margin-top: 30px;
}
.gallery h2 {
  margin-bottom: 10px;
}
.images {
  display: flex;
  flex-wrap: wrap;
}
.images {
  width: 200px;
  margin-right: 10px;
  margin-bottom: 10px;
}
</style>