<template>
  <div class="grid-wrapper">
    <div class="grid-container">
      <div
        v-for="slot in slots"
        :key="slot.name"
        :class="['slot', slot.name.toLowerCase(), { selected: selectedSlot === slot.name }]"
        @click="selectSlot(slot.name)"
      >
        <canvas
          :ref="el => (slotRefs[slot.name] = el)"
          class="slot-canvas"
        ></canvas>
        <span class="label">{{ slot.name }}</span>
      </div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <button
        class="btn"
        :disabled="!selectedSlot"
        @click="triggerFileInput"
      >
        Load image into {{ selectedSlot || 'slot' }}
      </button>

      <input
        ref="fileInput"
        type="file"
        accept="image/*"
        class="hidden"
        @change="handleFileForSelected"
      />

      <button
        class="btn save"
        :disabled="!hasImages"
        @click="saveAll"
      >
        Save All (JPG)
      </button>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed, onMounted } from 'vue'

/* --- configuration of slots in logical order --- */
const slots = [
  { name: 'Face' }, // left, spans two rows
  { name: 'Maxi' }, // center top
  { name: 'Side' }, // right, spans two rows
  { name: 'Mand' }, // center mid
  { name: 'Rite' }, // bottom left
  { name: 'Fore' }, // bottom center
  { name: 'Left' }  // bottom right
]

/* reactive refs */
const slotRefs = reactive({})
const images = reactive({})
const selectedSlot = ref(null)
const fileInput = ref(null)

const hasImages = computed(() => Object.keys(images).length > 0)

/* --- selection logic --- */
function selectSlot(name) {
  selectedSlot.value = selectedSlot.value === name ? null : name
}

/* --- file input helpers --- */
function triggerFileInput() {
  if (!selectedSlot.value) return
  fileInput.value.value = ''
  fileInput.value.click()
}

function handleFileForSelected(event) {
  const file = event.target.files && event.target.files[0]
  if (!file || !selectedSlot.value) return

  const reader = new FileReader()
  reader.onload = e => {
    const img = new Image()
    img.onload = () => {
      images[selectedSlot.value] = img
      drawImageToCanvas(selectedSlot.value)
    }
    img.src = e.target.result
  }
  reader.readAsDataURL(file)
}

/* --- drawing helper --- */
function drawImageToCanvas(slotName) {
  const canvas = slotRefs[slotName]
  const img = images[slotName]
  if (!canvas || !img) return

  const width = canvas.clientWidth
  const height = canvas.clientHeight
  canvas.width = Math.round(width)
  canvas.height = Math.round(height)

  const ctx = canvas.getContext('2d')
  ctx.clearRect(0, 0, canvas.width, canvas.height)

  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  const scale = Math.min(canvas.width / img.width, canvas.height / img.height)
  const drawW = Math.round(img.width * scale)
  const drawH = Math.round(img.height * scale)
  const offsetX = Math.round((canvas.width - drawW) / 2)
  const offsetY = Math.round((canvas.height - drawH) / 2)

  ctx.drawImage(img, offsetX, offsetY, drawW, drawH)
}

/* --- Save all canvases as JPG --- */
function saveAll() {
  slots.forEach(slot => {
    const canvas = slotRefs[slot.name]
    if (!canvas) return
    const link = document.createElement('a')
    link.download = `${slot.name}.jpg`
    link.href = canvas.toDataURL('image/jpeg', 0.92)
    link.click()
  })
}

/* --- Resize handling: only in browser --- */
onMounted(() => {
  window.addEventListener('resize', () => {
    Object.keys(images).forEach(name => {
      setTimeout(() => drawImageToCanvas(name), 50)
    })
  })
})
</script>

<style scoped>
/* wrapper */
.grid-wrapper {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* grid */
.grid-container {
  width: 100%;
  max-width: 1400px;
  padding: 18px;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 360px 360px 260px;
  gap: 18px;
  grid-template-areas:
    "face maxi side"
    "face mand side"
    "rite fore left";
}

.slot {
  position: relative;
  overflow: hidden;
  border-radius: 8px;
  background: #fff;
  border: 2px solid #e0e0e0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.slot-canvas {
  width: 100%;
  height: 100%;
  display: block;
}

.label {
  position: absolute;
  bottom: 6px;
  right: 8px;
  background: rgba(0,0,0,0.55);
  color: #fff;
  font-size: 0.8rem;
  padding: 3px 6px;
  border-radius: 4px;
}

.face { grid-area: face; grid-row: 1 / span 2; }
.maxi { grid-area: maxi; }
.mand { grid-area: mand; }
.side { grid-area: side; grid-row: 1 / span 2; }
.rite { grid-area: rite; }
.fore { grid-area: fore; }
.left { grid-area: left; }

.slot.selected {
  border: 3px solid #007bff;
  box-shadow: 0 0 10px rgba(0,123,255,0.45);
  z-index: 2;
}

.controls {
  margin-top: 14px;
  display: flex;
  gap: 12px;
  align-items: center;
}
.btn {
  padding: 8px 12px;
  border-radius: 6px;
  background: #2d9f4a;
  color: white;
  border: none;
  cursor: pointer;
}
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
.save { background: #1463d1; }

@media (max-width: 900px) {
  .grid-container {
    grid-template-columns: 1fr;
    grid-template-rows:
      320px 320px 320px 220px 220px 220px 220px;
    grid-template-areas:
      "face"
      "maxi"
      "mand"
      "side"
      "rite"
      "fore"
      "left";
    gap: 12px;
    padding: 12px;
  }
  .slot { min-height: 160px; }
}
</style>
