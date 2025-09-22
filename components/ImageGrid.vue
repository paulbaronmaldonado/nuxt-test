<template>
  <div class="p-4">
    <!-- Input para cargar múltiples imágenes -->
    <input type="file" multiple accept="image/*" @change="onFilesSelected" />

    <!-- Grid fijo para mostrar las imágenes -->
    <div class="grid grid-cols-3 gap-4 mt-4">
      <!-- Dentro del grid -->
      <div
        v-for="(img, index) in images"
        :key="index"
        class="relative cursor-pointer transition-all flex items-center justify-center
               rounded border-4 w-48 h-48 m-2"
        :class="selectedIndex === index
          ? 'border-blue-500 shadow-lg shadow-blue-300'
          : 'border-gray-300'"
        @click="selectImage(index)"
      >
        <canvas
          :ref="setCanvasRef(index)"
          class="max-w-full max-h-full object-contain"
        ></canvas>
      </div>

    </div>

    <!-- Botones de transformación -->
    <div class="flex gap-2 mt-4">
      <button
        class="px-3 py-1 rounded bg-gray-200 disabled:opacity-50"
        :disabled="selectedIndex === null"
        @click="rotateLeft"
      >Rotate Left</button>

      <button
        class="px-3 py-1 rounded bg-gray-200 disabled:opacity-50"
        :disabled="selectedIndex === null"
        @click="rotateRight"
      >Rotate Right</button>

      <button
        class="px-3 py-1 rounded bg-gray-200 disabled:opacity-50"
        :disabled="selectedIndex === null"
        @click="flipHorizontal"
      >Flip H</button>

      <button
        class="px-3 py-1 rounded bg-gray-200 disabled:opacity-50"
        :disabled="selectedIndex === null"
        @click="flipVertical"
      >Flip V</button>
    </div>

    <!-- Botón para guardar todas las imágenes -->
    <div class="mt-4">
      <button
        class="px-4 py-2 rounded bg-green-500 text-white disabled:opacity-50"
        :disabled="images.length === 0"
        @click="saveAll"
      >Save All (JPG)</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const images = ref([])
const canvases = ref([])
const selectedIndex = ref(null)

// Manejo de canvas refs dinámicos
const setCanvasRef = (index) => (el) => {
  canvases.value[index] = el
}

// Seleccionar una imagen del grid
const selectImage = (index) => {
  if (selectedIndex.value === index) {
    // Si clicas la misma, se deselecciona
    selectedIndex.value = null
  } else {
    selectedIndex.value = index
  }
}

// Dibujar imagen en canvas (escalada al frame)
function drawImageOnCanvas(canvas, imageData) {
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const img = new Image()
  img.onload = () => {
    // Ajuste del canvas al tamaño del contenedor
    canvas.width = 200
    canvas.height = 200
    ctx.clearRect(0, 0, canvas.width, canvas.height)

    // Calcular escala para que entre en el frame
    const scale = Math.min(canvas.width / img.width, canvas.height / img.height)
    const x = (canvas.width / 2) - (img.width / 2) * scale
    const y = (canvas.height / 2) - (img.height / 2) * scale
    ctx.setTransform(scale, 0, 0, scale, x, y)
    ctx.drawImage(img, 0, 0)
  }
  img.src = imageData
}

// Cargar imágenes seleccionadas
const onFilesSelected = (event) => {
  const files = event.target.files
  if (!files.length) return

  images.value = []
  canvases.value = []
  selectedIndex.value = null

  Array.from(files).forEach((file, index) => {
    const reader = new FileReader()
    reader.onload = (e) => {
      images.value.push({ src: e.target.result, rotation: 0, flipH: false, flipV: false })
      setTimeout(() => {
        drawTransformed(index)
      }, 50)
    }
    reader.readAsDataURL(file)
  })
}

// Dibujar con transformaciones aplicadas
function drawTransformed(index) {
  const canvas = canvases.value[index]
  const ctx = canvas?.getContext('2d')
  if (!canvas || !ctx) return

  const img = new Image()
  const data = images.value[index]
  img.onload = () => {
    canvas.width = 200
    canvas.height = 200
    ctx.clearRect(0, 0, canvas.width, canvas.height)

    const scale = Math.min(canvas.width / img.width, canvas.height / img.height)
    const x = canvas.width / 2
    const y = canvas.height / 2

    ctx.save()
    ctx.translate(x, y)
    ctx.rotate((data.rotation * Math.PI) / 180)
    ctx.scale(data.flipH ? -scale : scale, data.flipV ? -scale : scale)
    ctx.drawImage(img, -img.width / 2, -img.height / 2)
    ctx.restore()
  }
  img.src = data.src
}

// Transformaciones
const rotateLeft = () => {
  if (selectedIndex.value === null) return
  images.value[selectedIndex.value].rotation -= 90
  drawTransformed(selectedIndex.value)
}

const rotateRight = () => {
  if (selectedIndex.value === null) return
  images.value[selectedIndex.value].rotation += 90
  drawTransformed(selectedIndex.value)
}

const flipHorizontal = () => {
  if (selectedIndex.value === null) return
  images.value[selectedIndex.value].flipH = !images.value[selectedIndex.value].flipH
  drawTransformed(selectedIndex.value)
}

const flipVertical = () => {
  if (selectedIndex.value === null) return
  images.value[selectedIndex.value].flipV = !images.value[selectedIndex.value].flipV
  drawTransformed(selectedIndex.value)
}

// Guardar todas las imágenes como JPG
const saveAll = () => {
  images.value.forEach((img, index) => {
    const canvas = canvases.value[index]
    if (!canvas) return
    const link = document.createElement('a')
    link.download = `image_${index + 1}.jpg`
    link.href = canvas.toDataURL('image/jpeg', 0.9)
    link.click()
  })
}
</script>
