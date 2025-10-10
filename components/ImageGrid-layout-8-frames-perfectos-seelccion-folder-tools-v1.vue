<template>
  <!-- root -->
  <div class="app-root" :style="rootStyle">
    <!-- LEFT: grid with frames (sin cambios a tu versión buena 01) -->
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

      <!-- SECTION 3: Transformations -->
      <section class="panel-section">
        <h3>3. Transformations</h3>
        <p class="muted">Apply edits to the selected slot (rotate, flip, zoom, move).</p>

        <div class="transform-grid">
          <div class="row-btns">
            <button class="btn small" @click="applyTransform('rotateLeft')" :disabled="!canTransform">⟲ Rotate Left</button>
            <button class="btn small" @click="applyTransform('rotateRight')" :disabled="!canTransform">Rotate Right ⟳</button>
            <button class="btn small" @click="applyTransform('flipH')" :disabled="!canTransform">Flip H ↔</button>
            <button class="btn small" @click="applyTransform('flipV')" :disabled="!canTransform">Flip V ↕</button>
          </div>

          <div class="row-btns" style="margin-top:8px;">
            <button class="btn small" @click="applyTransform('zoomIn')" :disabled="!canTransform">Zoom +</button>
            <button class="btn small" @click="applyTransform('zoomOut')" :disabled="!canTransform">Zoom −</button>
            <button class="btn small" @click="applyTransform('reset')" :disabled="!canTransform">Reset</button>
          </div>

          <div class="nudge-block" style="margin-top:8px;">
            <div class="nudge-row">
              <button class="btn tiny" @click="nudgeTransform(0,-10)" :disabled="!canTransform">▲</button>
            </div>
            <div class="nudge-row" style="display:flex; gap:6px; justify-content:center; margin-top:6px;">
              <button class="btn tiny" @click="nudgeTransform(-10,0)" :disabled="!canTransform">◀</button>
              <button class="btn tiny" @click="nudgeTransform(10,0)" :disabled="!canTransform">▶</button>
            </div>
            <div class="nudge-row" style="display:flex; gap:6px; justify-content:center; margin-top:6px;">
              <button class="btn tiny" @click="nudgeTransform(0,10)" :disabled="!canTransform">▼</button>
            </div>
            <div class="nudge-hint muted" style="margin-top:6px; font-size:12px;">Use nudges to reposition image when zoomed.</div>
          </div>
        </div>
      </section>

      <!-- SECTION 4: Mapping -->
      <section class="panel-section grow">
        <h3>4. View ↔ File</h3>
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
import { ref, reactive, onMounted, onBeforeUnmount, nextTick, computed } from 'vue'

/* ---------- Config ---------- */
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

/* ---------- Estado ---------- */
const images = reactive(Object.fromEntries(slots.map(s => [s, null])))
const fileNames = reactive(Object.fromEntries(slots.map(s => [s, null])))

/* refs a canvases */
const cSerio = ref(null), cFace = ref(null), cSide = ref(null),
      cMaxl = ref(null), cMand = ref(null), cRite = ref(null),
      cFore = ref(null), cLeft = ref(null)
const refsMap = { Serio: cSerio, Face: cFace, Side: cSide, Maxl: cMaxl, Mand: cMand, Rite: cRite, Fore: cFore, Left: cLeft }

/* archivos / selección */
const allFiles = ref([])
const selectedFile = ref(null)
const selectedSlot = ref(null)
const selectedCollection = ref(null)

/* CSS vars */
const rootStyleReactive = reactive({ '--frame-w': '220px', '--frame-h': '165px' })
const rootStyle = rootStyleReactive

/* URLs previas para revocar */
let previousObjectUrls = []

/* computed: orden alfabético para mostrar */
const sortedFiles = computed(() => {
  return [...allFiles.value].sort((a,b) => a.name.localeCompare(b.name))
})

/* saber si hay alguna imagen cargada */
const hasAnyImage = computed(() => slots.some(s => !!images[s]))

/* ------------------ TRANSFORMACIONES ------------------ */
const transforms = reactive(Object.fromEntries(slots.map(s => [s, {
  rotate: 0,
  flipH: false,
  flipV: false,
  zoom: 1,
  offsetX: 0,
  offsetY: 0
}])))

const canTransform = computed(() => !!(selectedSlot.value && images[selectedSlot.value]))

function applyTransform(action) {
  const slot = selectedSlot.value
  if (!slot) return
  const t = transforms[slot]
  switch (action) {
    case 'rotateLeft': t.rotate = (t.rotate - 90) % 360; break
    case 'rotateRight': t.rotate = (t.rotate + 90) % 360; break
    case 'flipH': t.flipH = !t.flipH; break
    case 'flipV': t.flipV = !t.flipV; break
    case 'zoomIn': t.zoom = Math.min(3, +(t.zoom + 0.1).toFixed(2)); break
    case 'zoomOut': t.zoom = Math.max(0.5, +(t.zoom - 0.1).toFixed(2)); break
    case 'reset':
      transforms[slot].rotate = 0
      transforms[slot].flipH = false
      transforms[slot].flipV = false
      transforms[slot].zoom = 1
      transforms[slot].offsetX = 0
      transforms[slot].offsetY = 0
      break
  }
  nextTick(() => drawSlot(slot))
}

function nudgeTransform(dx, dy) {
  const slot = selectedSlot.value
  if (!slot) return
  transforms[slot].offsetX = (transforms[slot].offsetX || 0) + dx
  transforms[slot].offsetY = (transforms[slot].offsetY || 0) + dy
  nextTick(() => drawSlot(slot))
}

/* ------------------ Manejo de carpeta (filtrado) ------------------ */
function handleFolderSelect(e) {
  previousObjectUrls.forEach(url => URL.revokeObjectURL(url))
  previousObjectUrls = []
  allFiles.value = []

  const files = Array.from(e.target.files || [])
  const allowedRE = /\.(jpe?g|png|gif|bmp|webp|tiff?|tif)$/i

  files.forEach(f => {
    if (!f || typeof f.name !== 'string') return
    if (f.name.startsWith('.')) return
    if (!allowedRE.test(f.name)) return
    const url = URL.createObjectURL(f)
    previousObjectUrls.push(url)
    allFiles.value.push({ file: f, name: f.name, url })
  })

  allFiles.value.sort((a,b) => a.name.localeCompare(b.name))
  selectedFile.value = null
  selectedCollection.value = null
}

/* ------------------ Selección y sincronización ------------------ */
function handleFileClick(file) {
  selectedFile.value = file
  parseAndLoadSingle(file)
}

function onFrameClick(slotName) {
  selectedSlot.value = slotName
  const fname = fileNames[slotName]
  if (!fname) { selectedFile.value = null; return }
  const found = allFiles.value.find(f => f.name === fname)
  selectedFile.value = found || null
}

/* ------------------ Parsing e inference ------------------ */
function parseFilename(fname) {
  const base = fname.replace(/\.[^.]+$/, '')
  const tokens = base.split('_')
  const patientId = tokens[0] || null
  const collectionId = tokens.length === 5 ? tokens[3] : null
  const viewToken = tokens[tokens.length - 1] ? tokens[tokens.length - 1].toLowerCase() : ''
  return { patientId, collectionId, viewToken }
}

function mapView(viewToken) {
  for (const slot in viewMap) {
    if (viewMap[slot].includes(viewToken)) return slot
  }
  return null
}

/* ------------------ CARGA DE IMÁGENES ------------------ */
function parseAndLoadSingle(fileObj) {
  clearAll()
  selectedFile.value = fileObj
  const { viewToken } = parseFilename(fileObj.name)
  const slot = mapView(viewToken)
  if (!slot) return
  loadImageIntoSlot(fileObj, slot)
  selectedSlot.value = slot
}

function reloadWithThese() {
  if (!selectedFile.value) return
  const { patientId, collectionId } = parseFilename(selectedFile.value.name)
  selectedCollection.value = { patientId, collectionId }
  loadFullCollection()
}

function loadFullCollection() {
  if (!selectedCollection.value) return
  const { patientId, collectionId } = selectedCollection.value
  clearAll()
  allFiles.value.forEach(f => {
    const { patientId: pid, collectionId: cid, viewToken } = parseFilename(f.name)
    if (pid !== patientId) return
    if (collectionId) {
      if (cid !== collectionId) return
    } else {
      const tokens = f.name.replace(/\.[^.]+$/, '').split('_')
      if (tokens.length !== 4) return
    }
    const slot = mapView(viewToken)
    if (slot) loadImageIntoSlot(f, slot)
  })
}

function loadImageIntoSlot(fileObj, slot) {
  const img = new Image()
  img.onload = async () => {
    images[slot] = img
    fileNames[slot] = fileObj.name
    transforms[slot].rotate = 0
    transforms[slot].flipH = false
    transforms[slot].flipV = false
    transforms[slot].zoom = 1
    transforms[slot].offsetX = 0
    transforms[slot].offsetY = 0
    await nextTick()
    drawSlot(slot)
  }
  img.src = fileObj.url
}

/* ------------------ DIBUJO EN CANVAS (aplica transforms) ------------------ */
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

  const t = transforms[slot] || { rotate:0, flipH:false, flipV:false, zoom:1, offsetX:0, offsetY:0 }
  const baseScale = Math.min(w / img.width, h / img.height) || 1
  const totalScale = baseScale * (t.zoom || 1)

  ctx.save()
  ctx.translate(w/2, h/2)
  ctx.rotate((t.rotate || 0) * Math.PI / 180)
  const sx = (t.flipH ? -1 : 1) * totalScale
  const sy = (t.flipV ? -1 : 1) * totalScale
  ctx.scale(sx, sy)
  const offsetXAdj = (t.offsetX || 0) / (totalScale || 1)
  const offsetYAdj = (t.offsetY || 0) / (totalScale || 1)
  ctx.drawImage(img, -img.width/2 + offsetXAdj, -img.height/2 + offsetYAdj, img.width, img.height)
  ctx.restore()
}

/* redibujar todas */
function redrawAll() { slots.forEach(s => { if (images[s]) drawSlot(s) }) }

/* ------------------ Clear / Save ------------------ */
function clearAll() {
  slots.forEach(s => {
    images[s] = null
    fileNames[s] = null
    transforms[s].rotate = 0
    transforms[s].flipH = false
    transforms[s].flipV = false
    transforms[s].zoom = 1
    transforms[s].offsetX = 0
    transforms[s].offsetY = 0
  })
  selectedFile.value = null
  selectedSlot.value = null
  selectedCollection.value = null
}

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

/* ------------------ Resize logic ------------------ */
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
  previousObjectUrls.forEach(url => URL.revokeObjectURL(url))
  previousObjectUrls = []
})
</script>

<style scoped>
.app-root { display:flex; height:100vh; background:#f6f7f8; font-family:sans-serif; box-sizing:border-box; }
.left-column { flex:1; display:grid; grid-template-rows:auto auto auto; gap:12px; padding:12px; justify-items:center; overflow-y:auto; }
.row { width:100%; display:flex; justify-content:center; }
.row-1, .row-3 { display:grid; grid-template-columns:repeat(3,var(--frame-w)); gap:12px; justify-content:center; }
.row2-inner { display:flex; gap:12px; justify-content:center; }

.frame { background:#fff; border:1.5px solid #e2e6ea; border-radius:8px; box-shadow:0 1px 4px rgba(20,20,20,0.03); display:flex; align-items:center; justify-content:center; overflow:hidden; position:relative; }
.frame-horizontal { width:var(--frame-w); height:var(--frame-h); min-width:120px; min-height:90px; }
.frame-vertical { width:var(--frame-w); height:calc(var(--frame-w)*1.3333); min-width:120px; min-height:160px; }
.placeholder { color:#6b7280; font-weight:700; }
.frame canvas { width:100%; height:100%; display:block; }

.frame.selected { border:2px solid #0b63d6; box-shadow:0 6px 18px rgba(11,99,214,0.12); }

.right-panel { width:360px; border-left:1px solid #e6e8ea; background:#fff; display:flex; flex-direction:column; overflow:auto; box-sizing:border-box; }
.panel-section { padding:14px; border-bottom:1px solid #f1f3f4; }
.muted { color:#6b7280; font-size:13px; margin-bottom:8px; }
.btn { padding:8px 10px; border-radius:6px; border:1px solid #cbd5e1; background:#f8fafc; cursor:pointer; margin-right:6px; }
.btn.primary { background:#0b63d6; color:#fff; border-color:#0b63d6; }

.transform-grid .row-btns { display:flex; gap:6px; flex-wrap:wrap; }
.btn.small { padding:6px 8px; font-size:13px; }
.btn.tiny { padding:4px 6px; font-size:12px; }

.nudge-block { display:flex; flex-direction:column; align-items:center; }

.file-list { max-height:260px; overflow:auto; margin-top:8px; border:1px solid #e5e7eb; background:#fff; }
.file-list table { width:100%; border-collapse:collapse; font-size:13px; }
.file-list tr { cursor:pointer; }
.file-list tr.selected { background:#e6f0ff; }
.thumb { width:45px; }
.thumb img { width:40px; height:30px; object-fit:cover; }
.fname { padding-left:6px; }

.map-table { width:100%; border-collapse:collapse; font-size:13px; }
.map-table th, .map-table td { border:1px solid #e6e6e6; padding:6px 8px; }
.col-vista { width:40%; font-weight:700; }

@media (max-width:920px) {
  .app-root { flex-direction:column; }
  .right-panel { width:100%; border-left:none; border-top:1px solid #e6e8ea; }
}
</style>
