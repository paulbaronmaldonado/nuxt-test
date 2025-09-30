<template>
  <div class="wrapper">
    <!-- Controls: selector + file input + save all -->
    <div class="controls">
      <label>
        Slot:
        <select v-model="selectedSlot">
          <option disabled value="">-- select slot --</option>
          <option v-for="s in allSlots" :key="s" :value="s">{{ s }}</option>
        </select>
      </label>

      <label>
        Load image:
        <input ref="fileInput" type="file" accept="image/*" @change="handleFileInput" />
      </label>

      <button @click="clearSelected" :disabled="!selectedSlot">Deselect</button>
      <button @click="saveAll" :disabled="!hasAnyImage">Save All (JPG)</button>
    </div>

    <!-- Grid: 3 rows (3 vertical | 2 horizontal | 3 horizontal) -->
    <div class="grid">
      <!-- Row1: Serio, Smiling, Side (vertical) -->
      <div
        class="frame serio"
        :class="{ active: selectedSlot === 'Serio' }"
        @click="selectSlot('Serio')"
      >
        <canvas ref="cSerio"></canvas>
        <div v-if="!images.Serio" class="placeholder">Serio</div>
      </div>

      <div
        class="frame smiling"
        :class="{ active: selectedSlot === 'Smiling' }"
        @click="selectSlot('Smiling')"
      >
        <canvas ref="cSmiling"></canvas>
        <div v-if="!images.Smiling" class="placeholder">Smiling</div>
      </div>

      <div
        class="frame side"
        :class="{ active: selectedSlot === 'Side' }"
        @click="selectSlot('Side')"
      >
        <canvas ref="cSide"></canvas>
        <div v-if="!images.Side" class="placeholder">Side</div>
      </div>

      <!-- Row2: Maxi, Mand (horizontal) + empty cell to preserve grid -->
      <div class="row2">
        <div
          class="frame maxi"
          :class="{ active: selectedSlot === 'Maxi' }"
          @click="selectSlot('Maxi')"
        >
          <canvas ref="cMaxi"></canvas>
          <div v-if="!images.Maxi" class="placeholder">Maxi</div>
        </div>

        <div
          class="frame mand"
          :class="{ active: selectedSlot === 'Mand' }"
          @click="selectSlot('Mand')"
        >
          <canvas ref="cMand"></canvas>
          <div v-if="!images.Mand" class="placeholder">Mand</div>
        </div>
      </div>

      <div class="frame empty"></div> <!-- mantiene estructura de 3 columnas -->

      <!-- Row3: Rite, Fore, Left (horizontal) -->
      <div
        class="frame rite"
        :class="{ active: selectedSlot === 'Rite' }"
        @click="selectSlot('Rite')"
      >
        <canvas ref="cRite"></canvas>
        <div v-if="!images.Rite" class="placeholder">Rite</div>
      </div>

      <div
        class="frame fore"
        :class="{ active: selectedSlot === 'Fore' }"
        @click="selectSlot('Fore')"
      >
        <canvas ref="cFore"></canvas>
        <div v-if="!images.Fore" class="placeholder">Fore</div>
      </div>

      <div
        class="frame left"
        :class="{ active: selectedSlot === 'Left' }"
        @click="selectSlot('Left')"
      >
        <canvas ref="cLeft"></canvas>
        <div v-if="!images.Left" class="placeholder">Left</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onBeforeUnmount, nextTick, computed } from 'vue'

const allSlots = ['Serio','Smiling','Side','Maxi','Mand','Rite','Fore','Left']
const selectedSlot = ref('')          // nombre del slot activo
const images = reactive({})           // map slot -> dataURL
const fileInput = ref(null)

// canvas refs
const cSerio = ref(null)
const cSmiling = ref(null)
const cSide = ref(null)
const cMaxi = ref(null)
const cMand = ref(null)
const cRite = ref(null)
const cFore = ref(null)
const cLeft = ref(null)

const canvasMap = {
  Serio: cSerio,
  Smiling: cSmiling,
  Side: cSide,
  Maxi: cMaxi,
  Mand: cMand,
  Rite: cRite,
  Fore: cFore,
  Left: cLeft
}

const hasAnyImage = computed(() => Object.keys(images).length > 0)

function selectSlot(name) {
  selectedSlot.value = (selectedSlot.value === name) ? '' : name
  // set the select dropdown too
}

function clearSelected() {
  selectedSlot.value = ''
}

// file load from top control
function handleFileInput(e) {
  const file = e.target.files && e.target.files[0]
  if (!file) return
  if (!selectedSlot.value) {
    alert('Select a slot first (click a frame or choose from the dropdown).')
    fileInput.value.value = ''
    return
  }

  const reader = new FileReader()
  reader.onload = async (ev) => {
    images[selectedSlot.value] = ev.target.result
    await nextTick()
    drawSlot(selectedSlot.value)
    fileInput.value.value = ''
  }
  reader.readAsDataURL(file)
}

// central draw function (scale+center)
function drawSlot(slotName) {
  const canvasRef = canvasMap[slotName]
  const canvas = canvasRef?.value
  const dataUrl = images[slotName]
  if (!canvas || !dataUrl) return

  const ctx = canvas.getContext('2d')
  const img = new Image()
  img.onload = () => {
    // set internal pixel size to match CSS size
    canvas.width = Math.round(canvas.clientWidth)
    canvas.height = Math.round(canvas.clientHeight)

    ctx.clearRect(0, 0, canvas.width, canvas.height)
    // white background
    ctx.fillStyle = '#ffffff'
    ctx.fillRect(0, 0, canvas.width, canvas.height)

    const scale = Math.min(canvas.width / img.width, canvas.height / img.height)
    const w = Math.round(img.width * scale)
    const h = Math.round(img.height * scale)
    const x = Math.round((canvas.width - w) / 2)
    const y = Math.round((canvas.height - h) / 2)

    ctx.drawImage(img, x, y, w, h)
  }
  img.src = dataUrl
}

// redraw all (used on resize)
function redrawAll() {
  Object.keys(images).forEach(name => {
    drawSlot(name)
  })
}

// Save all canvases that have content
function saveAll() {
  allSlots.forEach(name => {
    const canvas = canvasMap[name].value
    if (!canvas) return
    // ensure at least something drawn - we accept blank white if no image
    const link = document.createElement('a')
    link.download = `${name}.jpg`
    link.href = canvas.toDataURL('image/jpeg', 0.92)
    link.click()
  })
}

let resizeHandler = null
onMounted(() => {
  // draw existing images if any (rare)
  nextTick(() => {
    Object.keys(images).forEach(drawSlot)
  })
  resizeHandler = () => {
    // small debounce
    setTimeout(redrawAll, 80)
  }
  window.addEventListener('resize', resizeHandler)
})
onBeforeUnmount(() => {
  if (resizeHandler) window.removeEventListener('resize', resizeHandler)
})
</script>

<style scoped>
/* outer wrapper */
.wrapper {
  padding: 12px;
  max-width: 1200px;
  margin: 0 auto;
  box-sizing: border-box;
}

/* controls line */
.controls {
  display: flex;
  gap: 10px;
  align-items: center;
  margin-bottom: 12px;
}

/* main grid 3 columns by 3 rows */
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 420px 220px 220px; /* row heights: tall, medium, medium */
  gap: 12px;
}

/* frame visual */
.frame {
  position: relative;
  border: 2px solid #ddd;
  border-radius: 8px;
  background: #fafafa;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* la fila 2 ocupa las 3 columnas y centra los 2 frames */
.row2 {
  grid-column: 1 / span 3;
  grid-row: 2;
  display: flex;
  justify-content: center;
  gap: 12px; /* espacio entre Maxi y Mand */
}
.row2 .frame {
  flex: 0 0 30%; /* cada uno ocupa ~30% del ancho total */
  max-width: 360px; /* opcional, para no crecer demasiado */
}


/* canvas fill the frame */
.frame canvas {
  width: 100%;
  height: 100%;
  display: block;
}

/* placeholder label when no image */
.placeholder {
  position: absolute;
  font-weight: 600;
  color: #666;
  pointer-events: none;
}

/* active selection highlight */
.frame.active {
  outline: 4px solid #007bff;
  box-shadow: 0 6px 18px rgba(0,123,255,0.12);
}

/* assign grid areas by class */
.serio { grid-column: 1; grid-row: 1; }
.smiling { grid-column: 2; grid-row: 1; }
.side { grid-column: 3; grid-row: 1; }

.maxi { grid-column: 1; grid-row: 2; }
.mand { grid-column: 2; grid-row: 2; }
/* right cell 2,3 is empty, so we keep .empty at grid-column:3,row:2 */

.rite { grid-column: 1; grid-row: 3; }
.fore { grid-column: 2; grid-row: 3; }
.left { grid-column: 3; grid-row: 3; }

/* responsive: on narrow screens stack rows */
@media (max-width: 900px) {
  .grid {
    grid-template-columns: 1fr;
    grid-template-rows: repeat(8, 220px);
  }
  .serio, .smiling, .side, .maxi, .mand, .rite, .fore, .left {
    grid-column: 1 !important;
  }
}
</style>
