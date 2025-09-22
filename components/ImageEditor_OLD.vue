<script setup>
import { ref, onMounted } from 'vue'

const canvas = ref(null)
const ctx = ref(null)
const image = ref(null)
const angle = ref(0)

onMounted(() => {
  // Aquí sí estamos en el navegador, se puede usar Image()
  image.value = new Image()
})

function onFileChange(e) {
  const file = e.target.files[0]
  if (!file || !image.value) return

  const reader = new FileReader()
  reader.onload = () => {
    image.value.src = reader.result
    image.value.onload = () => drawImage()
  }
  reader.readAsDataURL(file)
}

function drawImage() {
  if (!ctx.value) {
    ctx.value = canvas.value.getContext('2d')
  }

  const c = canvas.value
  const size = Math.max(image.value.width, image.value.height)
  c.width = size
  c.height = size

  ctx.value.clearRect(0, 0, c.width, c.height)
  ctx.value.save()
  ctx.value.translate(c.width / 2, c.height / 2)
  ctx.value.rotate((angle.value * Math.PI) / 180)
  ctx.value.drawImage(
    image.value,
    -image.value.width / 2,
    -image.value.height / 2
  )
  ctx.value.restore()
}

function rotate() {
  if (!image.value) return
  angle.value = (angle.value + 90) % 360
  drawImage()
}
</script>

<template>
  <div class="p-4">
    <input type="file" accept="image/*" @change="onFileChange" />
    <div class="mt-4">
      <canvas ref="canvas" class="border" />
    </div>
    <button
      class="mt-4 px-4 py-2 bg-blue-500 text-white rounded"
      @click="rotate"
    >
      Rotate 90°
    </button>
  </div>
</template>
