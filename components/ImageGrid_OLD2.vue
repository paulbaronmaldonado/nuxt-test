<template>
  <div>
    <input type="file" multiple accept="image/*" @change="handleFiles" />
    <div class="grid">
      <div
        v-for="(image, index) in images"
        :key="index"
        :class="['image-frame', { selected: selectedImage === index }]"
        @click="toggleSelection(index)"
      >
        <canvas
          :ref="el => canvasRefs[index] = el"
          class="canvas"
        ></canvas>
      </div>
    </div>

    <div class="controls">
      <button @click="rotate" :disabled="selectedImage === null">Rotate</button>
      <button @click="flipH" :disabled="selectedImage === null">Flip H</button>
      <button @click="flipV" :disabled="selectedImage === null">Flip V</button>
      <button @click="zoomIn" :disabled="selectedImage === null">Zoom In</button>
      <button @click="zoomOut" :disabled="selectedImage === null">Zoom Out</button>
      <button @click="saveAll" :disabled="images.length === 0">Save All</button>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'

const images = ref([])
const selectedImage = ref(null)
const canvasRefs = []
const transforms = ref([])

const handleFiles = async (event) => {
  const files = event.target.files
  images.value = []
  transforms.value = []

  for (const file of files) {
    const img = new Image()
    img.src = URL.createObjectURL(file)
    await new Promise(resolve => { img.onload = resolve })

    images.value.push(img)
    transforms.value.push({ rotate: 0, scaleX: 1, scaleY: 1, zoom: 1 })
  }

  await nextTick()
  images.value.forEach((img, index) => drawImage(index))
}

const toggleSelection = (index) => {
  selectedImage.value = selectedImage.value === index ? null : index
}

const drawImage = (index) => {
  const canvas = canvasRefs[index]
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const { rotate, scaleX, scaleY, zoom } = transforms.value[index]

  canvas.width = 200
  canvas.height = 200
  ctx.clearRect(0, 0, canvas.width, canvas.height)

  ctx.save()
  ctx.translate(canvas.width / 2, canvas.height / 2)
  ctx.rotate((rotate * Math.PI) / 180)
  ctx.scale(scaleX * zoom, scaleY * zoom)

  const img = images.value[index]
  const scale = Math.min(canvas.width / img.width, canvas.height / img.height)
  const newWidth = img.width * scale
  const newHeight = img.height * scale

  ctx.drawImage(img, -newWidth / 2, -newHeight / 2, newWidth, newHeight)
  ctx.restore()
}

const rotate = () => {
  if (selectedImage.value === null) return
  transforms.value[selectedImage.value].rotate += 90
  drawImage(selectedImage.value)
}
const flipH = () => {
  if (selectedImage.value === null) return
  transforms.value[selectedImage.value].scaleX *= -1
  drawImage(selectedImage.value)
}
const flipV = () => {
  if (selectedImage.value === null) return
  transforms.value[selectedImage.value].scaleY *= -1
  drawImage(selectedImage.value)
}
const zoomIn = () => {
  if (selectedImage.value === null) return
  transforms.value[selectedImage.value].zoom *= 1.2
  drawImage(selectedImage.value)
}
const zoomOut = () => {
  if (selectedImage.value === null) return
  transforms.value[selectedImage.value].zoom *= 0.8
  drawImage(selectedImage.value)
}

const saveAll = () => {
  images.value.forEach((_, index) => {
    const canvas = canvasRefs[index]
    const link = document.createElement('a')
    link.download = `image-${index + 1}.jpg`
    link.href = canvas.toDataURL('image/jpeg')
    link.click()
  })
}
</script>

<style scoped>
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, 200px);
  gap: 10px;
  margin-top: 10px;
}
.image-frame {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #ccc;
  overflow: hidden;
  cursor: pointer;
}
.image-frame.selected {
  outline: 4px solid #007BFF !important;
  box-shadow: 0 0 12px rgba(0, 123, 255, 0.8) !important;
}
.canvas {
  max-width: 100%;
  max-height: 100%;
}
.controls {
  margin-top: 20px;
  display: flex;
  gap: 10px;
}
</style>
