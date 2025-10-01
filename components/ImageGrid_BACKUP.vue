<template>
  <!-- root con variables CSS reactivas vía :style -->
  <div class="app-root" :style="rootStyle">
    <!-- IZQUIERDA: grid con 3 filas (fila1 verticales, fila2 2 centrados, fila3 horizontales) -->
    <div class="left-column" ref="leftCol">
      <!-- FILA 1 -->
      <div class="row row-1">
        <div class="frame frame-vertical" data-slot="Serio">
          <canvas v-if="images.Serio" ref="cSerio"></canvas>
          <div v-else class="placeholder">Serio</div>
        </div>

        <div class="frame frame-vertical" data-slot="Face">
          <canvas v-if="images.Face" ref="cFace"></canvas>
          <div v-else class="placeholder">Face</div>
        </div>

        <div class="frame frame-vertical" data-slot="Side">
          <canvas v-if="images.Side" ref="cSide"></canvas>
          <div v-else class="placeholder">Side</div>
        </div>
      </div>

      <!-- FILA 2 (dos frames centrados, mismos tamaños que fila 3) -->
      <div class="row row-2">
        <div class="row2-inner">
          <div class="frame frame-horizontal" data-slot="Maxi">
            <canvas v-if="images.Maxi" ref="cMaxi"></canvas>
            <div v-else class="placeholder">Maxi</div>
          </div>

          <div class="frame frame-horizontal" data-slot="Mand">
            <canvas v-if="images.Mand" ref="cMand"></canvas>
            <div v-else class="placeholder">Mand</div>
          </div>
        </div>
      </div>

      <!-- FILA 3 -->
      <div class="row row-3">
        <div class="frame frame-horizontal" data-slot="Rite">
          <canvas v-if="images.Rite" ref="cRite"></canvas>
          <div v-else class="placeholder">Rite</div>
        </div>

        <div class="frame frame-horizontal" data-slot="Fore">
          <canvas v-if="images.Fore" ref="cFore"></canvas>
          <div v-else class="placeholder">Fore</div>
        </div>

        <div class="frame frame-horizontal" data-slot="Left">
          <canvas v-if="images.Left" ref="cLeft"></canvas>
          <div v-else class="placeholder">Left</div>
        </div>
      </div>
    </div>

    <!-- DERECHA: panel de controles -->
    <aside class="right-panel">
      <section class="panel-section">
        <h3>1. Selección de slot</h3>
        <p class="muted">Selecciona el directorio con imágenes.</p>

        <label class="label-inline">
          <input type="file" webkitdirectory directory @change="handleDirectory" />
        </label>

        <!-- listado de archivos -->
        <div class="file-list">
          <table>
            <thead>
              <tr><th>Prev</th><th>Archivo</th></tr>
            </thead>
            <tbody>
              <tr v-for="(f, idx) in files" :key="idx">
                <td class="thumb-col">
                  <img v-if="f.thumb" :src="f.thumb" class="thumb"/>
                </td>
                <td class="name-col">{{ f.name }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <section class="panel-section">
        <h3>2. Controles</h3>
        <p class="muted">Controles básicos de prueba.</p>
        <button @click="clearSelected" :disabled="!selectedSlot" class="btn">Deseleccionar</button>
        <button @click="saveAll" :disabled="!hasAnyImage" class="btn primary">Save All (JPG)</button>
      </section>

      <section class="panel-section grow">
        <h3>3. Vista ↔ Archivo</h3>
        <p class="muted">Se mostrará el mapping cuando se use "USAR ESTOS".</p>

        <table class="map-table">
          <thead><tr><th>Vista</th><th>Archivo</th></tr></thead>
          <tbody>
            <tr v-for="s in slots" :key="s">
              <td class="col-vista">{{ s }}</td>
              <td class="col-file">{{ fileNames[s] || '---' }}</td>
            </tr>
          </tbody>
        </table>
      </section>
    </aside>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onBeforeUnmount, nextTick, computed } from 'vue'

/* ---------- Config ---------- */
const slots = ['Serio','Face','Side','Maxi','Mand','Rite','Fore','Left']

/* estado de imágenes (obj Image) y nombres de archivo */
const images = reactive(Object.fromEntries(slots.map(s => [s, null])))
const fileNames = reactive(Object.fromEntries(slots.map(s => [s, null])))

/* listado de archivos de directorio */
const files = ref([])

/* refs a canvases y contenedor */
const cSerio = ref(null), cFace = ref(null), cSide = ref(null),
      cMaxi = ref(null), cMand = ref(null), cRite = ref(null),
      cFore = ref(null), cLeft = ref(null)
const refsMap = { Serio: cSerio, Face: cFace, Side: cSide,
                  Maxi: cMaxi, Mand: cMand, Rite: cRite,
                  Fore: cFore, Left: cLeft }

const leftCol = ref(null)
const fileInput = ref(null)
const selectedSlot = ref('')

/* reactive style root: variables CSS que actualizamos desde JS */
const rootStyleReactive = reactive({
  '--frame-w': '220px',
  '--frame-h': '165px'
})
const rootStyle = rootStyleReactive

/* computed para saber si hay alguna imagen */
const hasAnyImage = computed(() => slots.some(s => !!images[s]))

/* ---------- Directorio ---------- */
function handleDirectory(event) {
  const fileList = Array.from(event.target.files || [])
  files.value = fileList.map(f => {
    let thumb = null
    if (f.type.startsWith('image/')) {
      thumb = URL.createObjectURL(f)
    }
    return { name: f.name, thumb }
  })
}

/* ---------- Dibujo en canvas ---------- */
function drawSlot(slot) {
  const cref = refsMap[slot]
  const canvas = cref && cref.value
  const img = images[slot]
  if (!canvas || !img) return

  const ctx = canvas.getContext('2d')
  const w = Math.max(1, Math.round(canvas.clientWidth))
  const h = Math.max(1, Math.round(canvas.clientHeight))
  canvas.width = w
  canvas.height = h

  ctx.clearRect(0,0,w,h)
  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0,0,w,h)

  const scale = Math.min(w / img.width, h / img.height)
  const dw = Math.round(img.width * scale)
  const dh = Math.round(img.height * scale)
  const dx = Math.round((w - dw) / 2)
  const dy = Math.round((h - dh) / 2)
  ctx.drawImage(img, dx, dy, dw, dh)
}

function redrawAll() { slots.forEach(s => { if (images[s]) drawSlot(s) }) }

/* deselect, save */
function clearSelected() { selectedSlot.value = '' }

function saveAll() {
  slots.forEach(s => {
    const c = refsMap[s].value
    if (!c) return
    const link = document.createElement('a')
    link.href = c.toDataURL('image/jpeg', 0.92)
    link.download = `${s}.jpg`
    link.click()
  })
}

/* ---------- Cálculo dinámico de tamaños ---------- */
let resizeTimer = null
function computeSizes() {
  const rightPanelWidth = 360
  const horizontalGap = 12
  const verticalGapsTotal = 2 * 12

  const vw = window.innerWidth
  let leftW = Math.max(200, vw - rightPanelWidth - 32)

  let tentativeFrameW = (leftW - (2 * horizontalGap)) / 3

  let hHorizontal = tentativeFrameW * 3 / 4
  let hVertical = tentativeFrameW * 4 / 3

  let totalNeeded = hVertical + hHorizontal + hHorizontal + verticalGapsTotal + 24
  const vh = window.innerHeight

  if (totalNeeded > vh) {
    const usable = Math.max(120, vh - verticalGapsTotal - 24)
    const verticalFactor = 4/3
    const horizontalFactor = 3/4
    const denom = verticalFactor + 2*horizontalFactor
    const adjustedFrameW = Math.max(120, usable / denom)
    tentativeFrameW = Math.min(tentativeFrameW, adjustedFrameW)
    hHorizontal = tentativeFrameW * 3/4
    hVertical = tentativeFrameW * 4/3
  }

  const maxFrameW = 420
  tentativeFrameW = Math.min(maxFrameW, Math.max(120, tentativeFrameW))
  hHorizontal = tentativeFrameW * 3/4
  hVertical = tentativeFrameW * 4/3

  rootStyleReactive['--frame-w'] = `${Math.round(tentativeFrameW)}px`
  rootStyleReactive['--frame-h'] = `${Math.round(hHorizontal)}px`

  nextTick(redrawAll)
}

function handleResize() {
  if (resizeTimer) clearTimeout(resizeTimer)
  resizeTimer = setTimeout(() => {
    computeSizes()
  }, 120)
}

onMounted(() => {
  computeSizes()
  window.addEventListener('resize', handleResize)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.app-root {
  display: flex;
  height: 100vh;
  box-sizing: border-box;
  background: #f6f7f8;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
}

.left-column {
  flex: 1 1 0%;
  display: grid;
  grid-template-rows: auto auto auto;
  gap: 12px;
  padding: 12px;
  box-sizing: border-box;
  align-items: start;
  justify-items: center;
  overflow-y: auto;
}

.row { width: 100%; box-sizing: border-box; display: flex; justify-content: center; }
.row-1 { display: grid; grid-template-columns: repeat(3, var(--frame-w)); gap: 12px; justify-content: center; }
.row-2 { display: flex; justify-content: center; }
.row2-inner { display: flex; gap: 12px; justify-content: center; }
.row-3 { display: grid; grid-template-columns: repeat(3, var(--frame-w)); gap: 12px; justify-content: center; }

.frame {
  background: #fff;
  border: 1.5px solid #e2e6ea;
  border-radius: 8px;
  box-shadow: 0 1px 4px rgba(20,20,20,0.03);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.frame-horizontal {
  width: var(--frame-w);
  height: var(--frame-h);
  min-width: 120px;
  min-height: 90px;
}

.frame-vertical {
  width: var(--frame-w);
  height: calc(var(--frame-w) * 1.3333333);
  min-width: 120px;
  min-height: 160px;
}

.placeholder { color: #6b7280; font-weight: 700; pointer-events: none; }
.frame canvas { width: 100%; height: 100%; display: block; object-fit: contain; }

.right-panel {
  width: 360px;
  box-sizing: border-box;
  border-left: 1px solid #e6e8ea;
  background: #fff;
  display: flex;
  flex-direction: column;
  overflow: auto;
}

.panel-section { padding: 14px; border-bottom: 1px solid #f1f3f4; }
.muted { color: #6b7280; margin-bottom: 8px; font-size: 13px; }

.label-inline { display:flex; gap:8px; align-items:center; margin-bottom:10px; }
.label-inline select, .label-inline input[type="file"] { font-size:13px; }

.btn { display:inline-block; padding:8px 10px; border-radius:6px; border:1px solid #cbd5e1; background:#f8fafc; cursor:pointer; }
.btn.primary { background:#0b63d6; color:#fff; border-color:#0b63d6; }

/* listado archivos */
.file-list {
  border: 1px solid #e2e6ea;
  border-radius: 6px;
  overflow-y: auto;
  max-height: 220px; /* más compacto: solo una fila extra */
}

.file-list table { width: 100%; border-collapse: collapse; font-size: 13px; }
.file-list th, .file-list td { border-bottom: 1px solid #f1f1f1; padding: 4px 6px; text-align: left; }

/* columna thumbnail más angosta */
.thumb-col { width: 40px; }
.thumb { width: 32px; height: 32px; object-fit: cover; border-radius: 4px; }

.name-col { max-width: 240px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

/* mapping table */
.map-table { width:100%; border-collapse: collapse; font-size: 13px; }
.map-table th, .map-table td { border:1px solid #e6e6e6; padding:6px 8px; text-align:left; }
.col-vista { width:40%; font-weight:700; }

@media (max-width: 920px) {
  .app-root { flex-direction: column; }
  .right-panel { width: 100%; border-left: none; border-top: 1px solid #e6e8ea; }
  .left-column { padding: 8px; }
  :root {
    --frame-w: 140px;
    --frame-h: 105px;
  }
}
</style>
