<template>
  <!-- root -->
  <div class="app-root" :style="rootStyle">
    <!-- LEFT: grid with frames -->
    <div class="left-column" ref="leftCol">
      <!-- ROW 1 -->
      <div class="row row-1">
        <div class="frame frame-vertical" data-slot="Serio" :class="{ selected: selectedSlot === 'Serio' }" @click="onFrameClick('Serio')">
          <canvas v-if="images.Serio" ref="cSerio"></canvas>
          <div v-else class="placeholder">Serio</div>
        </div>

        <div class="frame frame-vertical" data-slot="Face" :class="{ selected: selectedSlot === 'Face' }" @click="onFrameClick('Face')">
          <canvas v-if="images.Face" ref="cFace"></canvas>
          <div v-else class="placeholder">Face</div>
        </div>

        <div class="frame frame-vertical" data-slot="Side" :class="{ selected: selectedSlot === 'Side' }" @click="onFrameClick('Side')">
          <canvas v-if="images.Side" ref="cSide"></canvas>
          <div v-else class="placeholder">Side</div>
        </div>
      </div>

      <!-- ROW 2 -->
      <div class="row row-2">
        <div class="row2-inner">
          <div class="frame frame-horizontal" data-slot="Maxl" :class="{ selected: selectedSlot === 'Maxl' }" @click="onFrameClick('Maxl')">
            <canvas v-if="images.Maxl" ref="cMaxl"></canvas>
            <div v-else class="placeholder">Maxl</div>
          </div>

          <div class="frame frame-horizontal" data-slot="Mand" :class="{ selected: selectedSlot === 'Mand' }" @click="onFrameClick('Mand')">
            <canvas v-if="images.Mand" ref="cMand"></canvas>
            <div v-else class="placeholder">Mand</div>
          </div>
        </div>
      </div>

      <!-- ROW 3 -->
      <div class="row row-3">
        <div class="frame frame-horizontal" data-slot="Rite" :class="{ selected: selectedSlot === 'Rite' }" @click="onFrameClick('Rite')">
          <canvas v-if="images.Rite" ref="cRite"></canvas>
          <div v-else class="placeholder">Rite</div>
        </div>

        <div class="frame frame-horizontal" data-slot="Fore" :class="{ selected: selectedSlot === 'Fore' }" @click="onFrameClick('Fore')">
          <canvas v-if="images.Fore" ref="cFore"></canvas>
          <div v-else class="placeholder">Fore</div>
        </div>

        <div class="frame frame-horizontal" data-slot="Left" :class="{ selected: selectedSlot === 'Left' }" @click="onFrameClick('Left')">
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
        <!-- user selects folder; input keeps using webkitdirectory (works in Chrome/Edge) -->
        <input
          type="file"
          webkitdirectory
          multiple
          @change="handleFolderSelect"
        />

        <div class="file-list" v-if="sortedFiles.length">
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
        <button @click="saveAll" class="btn" :disabled="!hasAnyImage">Save All (JPG)</button>
      </section>

      <!-- SECTION 3: Mapping -->
      <section class="panel-section grow">
        <h3>3. View ↔ File</h3>
        <table class="map-table">
          <thead><tr><th>View</th><th>File</th></tr></thead>
          <tbody>
            <tr v-for="s in slots" :key="s" :class="{ selected: selectedSlot === s }" @click="onFrameClick(s)">
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
  ImageGrid.vue (actualizado)
  - Basado en el código que pegaste como referencia.
  - Comentarios en español; UI en inglés.
  - Añadido:
     * filtro para listar solo imágenes al seleccionar carpeta
     * selección sincronizada (archivo <-> slot)
     * Save All (JPG) para descargar desde canvases
     * revoke URLs previos para evitar fugas
*/

import { ref, reactive, onMounted, onBeforeUnmount, nextTick, computed } from 'vue'

/* --- Configuración de slots y mapeo de vistas --- */
const slots = ['Serio','Face','Side','Maxl','Mand','Rite','Fore','Left']

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

/* --- Estado reactivo --- */
const images = reactive(Object.fromEntries(slots.map(s => [s, null])))
const fileNames = reactive(Object.fromEntries(slots.map(s => [s, null])))

/* refs a canvases (mismos nombres que en template) */
const cSerio = ref(null), cFace = ref(null), cSide = ref(null),
      cMaxl = ref(null), cMand = ref(null), cRite = ref(null),
      cFore = ref(null), cLeft = ref(null)
const refsMap = { Serio: cSerio, Face: cFace, Side: cSide,
                  Maxl: cMaxl, Mand: cMand, Rite: cRite,
                  Fore: cFore, Left: cLeft }

/* demás estados */
const allFiles = ref([])        // array de { file, name, url }
const selectedFile = ref(null)  // objeto de allFiles seleccionado
const selectedSlot = ref(null)  // slot seleccionado
const selectedCollection = ref(null) // { patientId, collectionId } cuando se recarga colección

/* CSS variables calculadas desde JS (mantengo como en base) */
const rootStyleReactive = reactive({ '--frame-w': '220px', '--frame-h': '165px' })
const rootStyle = rootStyleReactive

/* storage de URLs creadas para revocarlas después */
let previousObjectUrls = []

/* computed: orden alfabético para mostrar */
const sortedFiles = computed(() => {
  return [...allFiles.value].sort((a,b) => a.name.localeCompare(b.name))
})

/* saber si hay al menos una imagen cargada en canvases */
const hasAnyImage = computed(() => slots.some(s => !!images[s]))

/* ------------------ Manejo de carpeta (filtrado) ------------------ */

/* Filtrar solo imágenes y crear object URLs; revocar previas */
function handleFolderSelect(e) {
  // revocar las URL previas
  previousObjectUrls.forEach(url => URL.revokeObjectURL(url))
  previousObjectUrls = []
  allFiles.value = []

  const files = Array.from(e.target.files || [])
  const allowedRE = /\.(jpe?g|png|gif|bmp|webp|tiff?|tif)$/i

  files.forEach(f => {
    if (!f || typeof f.name !== 'string') return
    // ignorar ficheros ocultos o no-imagenes
    if (f.name.startsWith('.')) return
    if (!allowedRE.test(f.name)) return
    const url = URL.createObjectURL(f)
    previousObjectUrls.push(url)
    allFiles.value.push({ file: f, name: f.name, url })
  })

  // ordenar
  allFiles.value.sort((a,b) => a.name.localeCompare(b.name))
  selectedFile.value = null
  selectedCollection.value = null
}

/* ------------------ Selección y sincronización ------------------ */

/* click en fila de listado -> cargar solo esa imagen y limpiar el resto */
function handleFileClick(file) {
  selectedFile.value = file
  parseAndLoadSingle(file)
}

/* click en un frame o en la tabla -> seleccionar slot y, si existe, elegir el archivo correspondiente */
function onFrameClick(slotName) {
  selectedSlot.value = slotName
  // buscar si ese slot tiene un fileName asociado y seleccionar la fila correspondiente
  const fname = fileNames[slotName]
  if (!fname) {
    selectedFile.value = null
    return
  }
  const found = allFiles.value.find(f => f.name === fname)
  selectedFile.value = found || null
}

/* ------------------ Parsing de nombres e inferencia de slot ------------------ */

/* Dado un filename -> extrae patientId, collectionId (si), viewToken */
function parseFilename(fname) {
  const base = fname.replace(/\.[^.]+$/, '')
  const tokens = base.split('_')
  const patientId = tokens[0] || null
  const collectionId = tokens.length === 5 ? tokens[3] : null
  const viewToken = tokens[tokens.length - 1] ? tokens[tokens.length - 1].toLowerCase() : ''
  return { patientId, collectionId, viewToken }
}

/* mapear token de vista a slot */
function mapView(viewToken) {
  for (const slot in viewMap) {
    if (viewMap[slot].includes(viewToken)) return slot
  }
  return null
}

/* ------------------ Carga de una sola imagen (al hacer click) ------------------ */

/* cuando se hace click en un archivo del listado: limpiar todo y mostrar solo esa */
function parseAndLoadSingle(fileObj) {
  clearAll()
  selectedFile.value = fileObj
  const { viewToken } = parseFilename(fileObj.name)
  const slot = mapView(viewToken)
  if (!slot) return
  // cargar (fileObj tiene .url para usar en Image.src)
  loadImageIntoSlot(fileObj, slot)
  selectedSlot.value = slot
}

/* ------------------ Cargar colección completa (Reload) ------------------ */

function reloadWithThese() {
  if (!selectedFile.value) return
  const { patientId, collectionId } = parseFilename(selectedFile.value.name)
  selectedCollection.value = { patientId, collectionId }
  loadFullCollection()
}

function loadFullCollection() {
  if (!selectedCollection.value) return
  const { patientId, collectionId } = selectedCollection.value
  // limpiar antes de cargar
  clearAll()
  // buscar en allFiles los archivos que pertenezcan a patientId y collectionId (o sin collection si no aplica)
  allFiles.value.forEach(f => {
    const { patientId: pid, collectionId: cid, viewToken } = parseFilename(f.name)
    // coincide el paciente
    if (pid !== patientId) return
    // si collectionId existe, debe coincidir; si no existe, aceptar solo archivos sin collection (4 tokens)
    if (collectionId) {
      if (cid !== collectionId) return
    } else {
      // si collectionId no está en la referencia, aceptar solo archivos con 4 tokens
      const tokens = f.name.replace(/\.[^.]+$/, '').split('_')
      if (tokens.length !== 4) return
    }
    const slot = mapView(viewToken)
    if (slot) loadImageIntoSlot(f, slot)
  })
}

/* ------------------ Operaciones canvas y helper de carga ------------------ */

function loadImageIntoSlot(fileObj, slot) {
  // fileObj: { file, name, url } o similar
  const img = new Image()
  img.onload = async () => {
    images[slot] = img
    fileNames[slot] = fileObj.name
    await nextTick()
    drawSlot(slot)
  }
  img.src = fileObj.url
}

/* Dibuja respetando centrado y escala proporcional */
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

function redrawAll() {
  slots.forEach(s => {
    if (images[s]) drawSlot(s)
  })
}

/* ------------------ Clear / Save ------------------ */

function clearAll() {
  slots.forEach(s => {
    images[s] = null
    fileNames[s] = null
  })
  selectedFile.value = null
  selectedSlot.value = null
  selectedCollection.value = null
}

/* Guardar todas las canvases como JPG (nombre por slot) */
function saveAll() {
  slots.forEach(s => {
    const cref = refsMap[s]
    const canvas = cref && cref.value
    if (!canvas) return
    const data = canvas.toDataURL('image/jpeg', 0.92)
    const link = document.createElement('a')
    link.href = data
    link.download = `${s}.jpg`
    link.click()
  })
}

/* ------------------ Resize logic (igual que antes) ------------------ */
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

/* lifecycle */
onMounted(() => {
  computeSizes()
  window.addEventListener('resize', handleResize)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  // revocar URLs creadas
  previousObjectUrls.forEach(url => URL.revokeObjectURL(url))
  previousObjectUrls = []
})
</script>

<style scoped>
/* igual que antes pero con clases para selección */
.app-root { display:flex; height:100vh; background:#f6f7f8; font-family:sans-serif; box-sizing:border-box; }

/* LEFT */
.left-column { flex:1; display:grid; grid-template-rows:auto auto auto; gap:12px; padding:12px; justify-items:center; overflow-y:auto; }
.row { width:100%; display:flex; justify-content:center; }
.row-1, .row-3 { display:grid; grid-template-columns:repeat(3,var(--frame-w)); gap:12px; justify-content:center; }
.row2-inner { display:flex; gap:12px; justify-content:center; }

.frame { background:#fff; border:1.5px solid #e2e6ea; border-radius:8px; box-shadow:0 1px 4px rgba(20,20,20,0.03); display:flex; align-items:center; justify-content:center; overflow:hidden; position:relative; }
.frame-horizontal { width:var(--frame-w); height:var(--frame-h); min-width:120px; min-height:90px; }
.frame-vertical { width:var(--frame-w); height:calc(var(--frame-w)*1.3333); min-width:120px; min-height:160px; }
.placeholder { color:#6b7280; font-weight:700; }
.frame canvas { width:100%; height:100%; display:block; }

/* selección visual en frame */
.frame.selected { border:2px solid #0b63d6; box-shadow:0 6px 18px rgba(11,99,214,0.12); }

/* RIGHT */
.right-panel { width:360px; border-left:1px solid #e6e8ea; background:#fff; display:flex; flex-direction:column; overflow:auto; box-sizing:border-box; }
.panel-section { padding:14px; border-bottom:1px solid #f1f3f4; }
.muted { color:#6b7280; font-size:13px; margin-bottom:8px; }
.btn { padding:8px 10px; border-radius:6px; border:1px solid #cbd5e1; background:#f8fafc; cursor:pointer; margin-right:6px; }
.btn.primary { background:#0b63d6; color:#fff; border-color:#0b63d6; }

/* file list */
.file-list { max-height:260px; overflow:auto; margin-top:8px; border:1px solid #e5e7eb; background:#fff; }
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

/* responsive */
@media (max-width:920px) {
  .app-root { flex-direction:column; }
  .right-panel { width:100%; border-left:none; border-top:1px solid #e6e8ea; }
}
</style>
