<template>
  <!-- root -->
  <div class="app-root" :style="rootStyle">
    <!-- LEFT: grid with frames -->
    <div class="left-column" ref="leftCol">
      <!-- ROW 1 -->
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

      <!-- ROW 2 -->
      <div class="row row-2">
        <div class="row2-inner">
          <div class="frame frame-horizontal" data-slot="Maxl">
            <canvas v-if="images.Maxl" ref="cMaxl"></canvas>
            <div v-else class="placeholder">Maxl</div>
          </div>

          <div class="frame frame-horizontal" data-slot="Mand">
            <canvas v-if="images.Mand" ref="cMand"></canvas>
            <div v-else class="placeholder">Mand</div>
          </div>
        </div>
      </div>

      <!-- ROW 3 -->
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

    <!-- RIGHT: control panel -->
    <aside class="right-panel">
      <!-- SECTION 1: File listing -->
      <section class="panel-section">
        <h3>1. Select folder</h3>
        <p class="muted">Choose the folder containing the images.</p>
        <input
          type="file"
          webkitdirectory
          multiple
          @change="handleFolderSelect"
        />

        <div class="file-list">
          <table>
            <tbody>
              <tr
                v-for="(file, idx) in sortedFiles"
                :key="idx"
                :class="{ selected: selectedFile && selectedFile.name === file.name }"
                @click="handleFileClick(file)"
              >
                <td class="thumb">
                  <img :src="file.url" alt="thumb" />
                </td>
                <td class="fname">{{ file.name }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- SECTION 2: Controls -->
      <section class="panel-section">
        <h3>2. Controls</h3>
        <p class="muted">Basic test controls.</p>
        <button @click="clearAll" class="btn">Clear all</button>
        <button @click="reloadWithThese" class="btn primary" :disabled="!selectedFile">
          Reload with these images
        </button>
      </section>

      <!-- SECTION 3: Mapping -->
      <section class="panel-section grow">
        <h3>3. View ↔ File</h3>
        <table class="map-table">
          <thead><tr><th>View</th><th>File</th></tr></thead>
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
/* 
   ImageGrid.vue con:
   - listado de archivos con thumbnails
   - resaltado de archivo seleccionado
   - carga inmediata de la imagen seleccionada
   - botón "Reload with these images" que carga toda la colección
*/

import { ref, reactive, onMounted, onBeforeUnmount, nextTick, computed } from 'vue'

/* slots */
const slots = ['Serio','Face','Side','Maxl','Mand','Rite','Fore','Left']

/* estados */
const images = reactive(Object.fromEntries(slots.map(s => [s, null])))
const fileNames = reactive(Object.fromEntries(slots.map(s => [s, null])))

const allFiles = ref([]) // todos los archivos leídos de la carpeta
const selectedFile = ref(null) // archivo actualmente seleccionado
const selectedCollection = ref(null) // info de la colección

/* refs a canvases */
const cSerio = ref(null), cFace = ref(null), cSide = ref(null),
      cMaxl = ref(null), cMand = ref(null), cRite = ref(null),
      cFore = ref(null), cLeft = ref(null)
const refsMap = { Serio: cSerio, Face: cFace, Side: cSide,
                  Maxl: cMaxl, Mand: cMand, Rite: cRite,
                  Fore: cFore, Left: cLeft }

/* estilos */
const rootStyleReactive = reactive({
  '--frame-w': '220px',
  '--frame-h': '165px'
})
const rootStyle = rootStyleReactive

/* ordenar archivos alfabéticamente */
const sortedFiles = computed(() => {
  return [...allFiles.value].sort((a,b) => a.name.localeCompare(b.name))
})

/* ====== HANDLERS ====== */
function handleFolderSelect(e) {
  const files = Array.from(e.target.files)
  allFiles.value = files.map(f => ({
    file: f,
    name: f.name,
    url: URL.createObjectURL(f)
  }))
}

/* click en un archivo */
function handleFileClick(file) {
  selectedFile.value = file
  parseAndLoadSingle(file)
}

/* botón Reload */
function reloadWithThese() {
  if (!selectedFile.value) return
  const { patientId, collectionId } = parseFilename(selectedFile.value.name)
  selectedCollection.value = { patientId, collectionId }
  loadFullCollection()
}

/* clear */
function clearAll() {
  slots.forEach(s => { images[s] = null; fileNames[s] = null })
  selectedFile.value = null
  selectedCollection.value = null
}

/* ====== PARSING DEL NOMBRE ====== */
function parseFilename(fname) {
  const base = fname.replace(/\.[^.]+$/, '')
  const tokens = base.split('_')
  let patientId = tokens[0]
  let collectionId = tokens.length === 5 ? tokens[3] : null
  let viewToken = tokens[tokens.length - 1].toLowerCase()
  return { patientId, collectionId, viewToken }
}

/* mapping de vistas */
const viewMap = {
  Serio: ['serio','serious'],
  Face: ['face','smile'],
  Side: ['side','profile'],
  Maxl: ['maxl','upper','upper jaw','max'],
  Mand: ['mand','lower','lower jaw'],
  Rite: ['rite','right','right buccal'],
  Fore: ['fore','anterior','front'],
  Left: ['left','left buccal']
}

function mapView(viewToken) {
  for (const slot in viewMap) {
    if (viewMap[slot].includes(viewToken)) return slot
  }
  return null
}

/* ====== CARGA DE IMÁGENES ====== */
function parseAndLoadSingle(file) {
  const { viewToken } = parseFilename(file.name)
  const slot = mapView(viewToken)
  if (!slot) return
  loadImageIntoSlot(file, slot)
}

function loadFullCollection() {
  if (!selectedCollection.value) return
  const { patientId, collectionId } = selectedCollection.value
  allFiles.value.forEach(file => {
    const { patientId: pid, collectionId: cid, viewToken } = parseFilename(file.name)
    if (pid === patientId && cid === collectionId) {
      const slot = mapView(viewToken)
      if (slot) loadImageIntoSlot(file, slot)
    }
  })
}

function loadImageIntoSlot(file, slot) {
  const img = new Image()
  img.onload = async () => {
    images[slot] = img
    fileNames[slot] = file.name
    await nextTick()
    drawSlot(slot)
  }
  img.src = file.url
}

/* ====== CANVAS DRAW ====== */
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
  ctx.fillStyle = '#fff'
  ctx.fillRect(0,0,w,h)
  const scale = Math.min(w / img.width, h / img.height)
  const dw = Math.round(img.width * scale)
  const dh = Math.round(img.height * scale)
  const dx = Math.round((w - dw) / 2)
  const dy = Math.round((h - dh) / 2)
  ctx.drawImage(img, dx, dy, dw, dh)
}

function redrawAll() { slots.forEach(s => { if (images[s]) drawSlot(s) }) }

/* ====== RESIZE LOGIC ====== */
let resizeTimer = null
function computeSizes() {
  const rightPanelWidth = 360
  const horizontalGap = 12
  const verticalGapsTotal = 2*12
  const vw = window.innerWidth
  let leftW = Math.max(200, vw - rightPanelWidth - 32)
  let tentativeFrameW = (leftW - (2*horizontalGap)) / 3
  let hHorizontal = tentativeFrameW * 3/4
  let hVertical = tentativeFrameW * 4/3
  const vh = window.innerHeight
  let totalNeeded = hVertical + hHorizontal + hHorizontal + verticalGapsTotal + 24
  if (totalNeeded > vh) {
    const usable = Math.max(120, vh - verticalGapsTotal - 24)
    const denom = (4/3) + 2*(3/4)
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
  resizeTimer = setTimeout(() => computeSizes(), 120)
}
onMounted(() => { computeSizes(); window.addEventListener('resize', handleResize) })
onBeforeUnmount(() => window.removeEventListener('resize', handleResize))
</script>

<style scoped>
.app-root { display:flex; height:100vh; background:#f6f7f8; font-family:sans-serif; }

/* LEFT COLUMN */
.left-column { flex:1; display:grid; grid-template-rows:auto auto auto; gap:12px; padding:12px; justify-items:center; overflow-y:auto; }
.row { width:100%; display:flex; justify-content:center; }
.row-1, .row-3 { display:grid; grid-template-columns:repeat(3,var(--frame-w)); gap:12px; justify-content:center; }
.row2-inner { display:flex; gap:12px; justify-content:center; }

.frame { background:#fff; border:1.5px solid #e2e6ea; border-radius:8px; box-shadow:0 1px 4px rgba(20,20,20,0.03); display:flex; align-items:center; justify-content:center; }
.frame-horizontal { width:var(--frame-w); height:var(--frame-h); }
.frame-vertical { width:var(--frame-w); height:calc(var(--frame-w)*1.3333); }
.placeholder { color:#6b7280; font-weight:700; }
.frame canvas { width:100%; height:100%; }

/* RIGHT PANEL */
.right-panel { width:360px; border-left:1px solid #e6e8ea; background:#fff; display:flex; flex-direction:column; overflow:auto; }
.panel-section { padding:14px; border-bottom:1px solid #f1f3f4; }
.muted { color:#6b7280; font-size:13px; margin-bottom:8px; }
.btn { padding:8px 10px; border-radius:6px; border:1px solid #cbd5e1; background:#f8fafc; cursor:pointer; margin-right:6px; }
.btn.primary { background:#0b63d6; color:#fff; border-color:#0b63d6; }

/* file list */
.file-list { max-height:260px; overflow:auto; margin-top:8px; border:1px solid #e5e7eb; }
.file-list table { width:100%; border-collapse:collapse; font-size:13px; }
.file-list tr { cursor:pointer; }
.file-list tr.selected { background:#e6f0ff; }
.thumb { width:45px; }
.thumb img { width:40px; height:30px; object-fit:cover; }
.fname { padding-left:6px; }

/* mapping table */
.map-table { width:100%; border-collapse:collapse; font-size:13px; }
.map-table th, .map-table td { border:1px solid #e6e6e6; padding:6px 8px; }
.col-vista { width:40%; font-weight:700; }

@media (max-width:920px){ .app-root{flex-direction:column;} .right-panel{width:100%; border-left:none; border-top:1px solid #e6e8ea;} }
</style>
