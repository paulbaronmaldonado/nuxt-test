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
      <!-- toolbar -->
      <div class="toolbar">
        <h3 class="toolbar-title">Controls</h3>
        <button class="quick-save" @click="saveAll" title="Quick save">💾</button>
      </div>

      <!-- 1. Select folder -->
      <section class="panel-section">
        <h4>1. Select folder</h4>
        <p class="muted">Choose the folder containing the images.</p>
        <input type="file" webkitdirectory multiple @change="handleFolderSelect" />

        <div class="file-list" v-if="sortedFiles.length">
          <table>
            <tbody>
              <tr v-for="(file, idx) in sortedFiles" :key="idx"
                  :class="{ selected: selectedFile && selectedFile.name === file.name }"
                  @click="handleFileClick(file)">
                <td class="thumb"><img :src="file.url" alt="thumb" /></td>
                <td class="fname">{{ file.name }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- 2. Actions -->
      <section class="panel-section">
        <h4>2. Actions</h4>
        <div class="actions-row">
          <button class="btn" @click="clearAll">Clear all</button>
          <button class="btn primary" :disabled="!selectedFile" @click="reloadWithThese">Load images</button>
          <button class="btn" :disabled="!canUndo" @click="undoTransform">Undo</button>
        </div>
      </section>

      <!-- 3. View ↔ File -->
      <section class="panel-section">
        <h4>3. View ↔ File</h4>
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

      <!-- 4. Transformations -->
      <section class="panel-section">
        <h4>4. Transformations</h4>
        <div class="transform-grid">
          <div class="row-btns icon-row">
            <button class="btn small icon-btn" @click="applyTransform('rotateLeft')" :disabled="!canTransform" title="Rotate Left">⟲</button>
            <button class="btn small icon-btn" @click="applyTransform('rotateRight')" :disabled="!canTransform" title="Rotate Right">⟳</button>
            <button class="btn small icon-btn" @click="applyTransform('flipH')" :disabled="!canTransform" title="Flip Horizontal">⇋</button>
            <button class="btn small icon-btn" @click="applyTransform('flipV')" :disabled="!canTransform" title="Flip Vertical">⇵</button>
            <button class="btn small icon-btn" @click="applyTransform('reset')" :disabled="!canTransform" title="Reset">⟳0</button>
          </div>

          <div class="row-btns icon-row" style="margin-top:8px;">
            <button class="btn small icon-btn" @click="applyTransform('zoomIn')" :disabled="!canTransform">＋</button>
            <button class="btn small icon-btn" @click="applyTransform('zoomOut')" :disabled="!canTransform">－</button>
            <div class="nudge-inline">
              <button class="btn tiny icon-btn" @click="nudgeTransform(0,-10)" :disabled="!canTransform">▲</button>
              <div style="display:flex; gap:6px; align-items:center; margin-top:6px;">
                <button class="btn tiny icon-btn" @click="nudgeTransform(-10,0)" :disabled="!canTransform">◀</button>
                <button class="btn tiny icon-btn" @click="nudgeTransform(10,0)" :disabled="!canTransform">▶</button>
              </div>
              <button class="btn tiny icon-btn" @click="nudgeTransform(0,10)" :disabled="!canTransform" style="margin-top:6px">▼</button>
            </div>
          </div>
        </div>
      </section>

      <!-- Sticky Save -->
      <div class="sticky-save">
        <button class="btn primary full" @click="saveAll" :disabled="!hasAnyImage">Save All</button>
      </div>
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
const transforms = reactive(Object.fromEntries(slots.map(s => [s, { rotate:0, flipH:false, flipV:false, zoom:1, offsetX:0, offsetY:0 }])))

/* Undo stacks */
const undoStacks = reactive(Object.fromEntries(slots.map(s => [s, []])))
const UNDO_LIMIT = 10

function pushUndo(slot) {
  if (!slot) return
  const t = JSON.parse(JSON.stringify(transforms[slot]))
  undoStacks[slot].push(t)
  if (undoStacks[slot].length > UNDO_LIMIT) undoStacks[slot].shift()
}
function undoTransform() {
  const slot = selectedSlot.value
  if (!slot || undoStacks[slot].length === 0) return
  const last = undoStacks[slot].pop()
  Object.assign(transforms[slot], last)
  nextTick(() => drawSlot(slot))
}
const canUndo = computed(() => selectedSlot.value && undoStacks[selectedSlot.value]?.length > 0)

/* refs */
const cSerio = ref(null), cFace = ref(null), cSide = ref(null),
      cMaxl = ref(null), cMand = ref(null), cRite = ref(null),
      cFore = ref(null), cLeft = ref(null)
const refsMap = { Serio: cSerio, Face: cFace, Side: cSide, Maxl: cMaxl, Mand: cMand, Rite: cRite, Fore: cFore, Left: cLeft }

/* archivos */
const allFiles = ref([])
const selectedFile = ref(null)
const selectedSlot = ref(null)
const selectedCollection = ref(null)

/* estilo */
const rootStyleReactive = reactive({ '--frame-w': '220px', '--frame-h': '165px' })
const rootStyle = rootStyleReactive
let previousObjectUrls = []

/* computed */
const sortedFiles = computed(() => [...allFiles.value].sort((a,b)=>a.name.localeCompare(b.name)))
const hasAnyImage = computed(() => slots.some(s => !!images[s]))
const canTransform = computed(() => !!(selectedSlot.value && images[selectedSlot.value]))

function applyTransform(action) {
  const slot = selectedSlot.value
  if (!slot) return
  pushUndo(slot)
  const t = transforms[slot]
  switch (action) {
    case 'rotateLeft': t.rotate = (t.rotate - 90) % 360; break
    case 'rotateRight': t.rotate = (t.rotate + 90) % 360; break
    case 'flipH': t.flipH = !t.flipH; break
    case 'flipV': t.flipV = !t.flipV; break
    case 'zoomIn': t.zoom = Math.min(3, +(t.zoom + 0.1).toFixed(2)); break
    case 'zoomOut': t.zoom = Math.max(0.5, +(t.zoom - 0.1).toFixed(2)); break
    case 'reset': Object.assign(t,{rotate:0,flipH:false,flipV:false,zoom:1,offsetX:0,offsetY:0}); break
  }
  nextTick(()=>drawSlot(slot))
}

function nudgeTransform(dx,dy){
  const slot=selectedSlot.value
  if(!slot)return
  pushUndo(slot)
  transforms[slot].offsetX+=dx
  transforms[slot].offsetY+=dy
  nextTick(()=>drawSlot(slot))
}

/* carpeta */
function handleFolderSelect(e) {
  previousObjectUrls.forEach(url => URL.revokeObjectURL(url))
  previousObjectUrls = []
  allFiles.value = []
  const files = Array.from(e.target.files||[])
  const allowedRE = /\.(jpe?g|png|gif|bmp|webp|tiff?|tif)$/i
  files.forEach(f=>{
    if(!f||!f.name)return
    if(f.name.startsWith('.'))return
    if(!allowedRE.test(f.name))return
    const url=URL.createObjectURL(f)
    previousObjectUrls.push(url)
    allFiles.value.push({file:f,name:f.name,url})
  })
  allFiles.value.sort((a,b)=>a.name.localeCompare(b.name))
  selectedFile.value=null
  selectedCollection.value=null
}

function handleFileClick(file){
  selectedFile.value=file
  parseAndLoadSingle(file)
}

function onFrameClick(slotName){
  selectedSlot.value=slotName
  const fname=fileNames[slotName]
  if(!fname){selectedFile.value=null;return}
  const found=allFiles.value.find(f=>f.name===fname)
  selectedFile.value=found||null
}

/* filename parsing */
function parseFilename(fname){
  const base=fname.replace(/\.[^.]+$/,'')
  const tokens=base.split('_')
  const patientId=tokens[0]||null
  const collectionId=tokens.length===5?tokens[3]:null
  const viewToken=tokens[tokens.length-1]?tokens[tokens.length-1].toLowerCase():''
  return {patientId,collectionId,viewToken}
}
function mapView(viewToken){
  for(const slot in viewMap){
    if(viewMap[slot].includes(viewToken))return slot
  }
  return null
}

/* carga */
function parseAndLoadSingle(fileObj){
  clearAll()
  selectedFile.value=fileObj
  const {viewToken}=parseFilename(fileObj.name)
  const slot=mapView(viewToken)
  if(!slot)return
  loadImageIntoSlot(fileObj,slot)
  selectedSlot.value=slot
}
function reloadWithThese(){
  if(!selectedFile.value)return
  const fname=selectedFile.value.name
  const isTokenized=fname.includes('_')
  if(isTokenized){
    const {patientId,collectionId}=parseFilename(fname)
    selectedCollection.value={patientId,collectionId}
    loadFullCollection()
  }else{
    clearAll()
    const sorted=sortedFiles.value
    const startIdx=sorted.findIndex(f=>f.name===fname)
    if(startIdx<0)return
    const sequenceMap=['Fore','Rite','Left','Mand','Maxl','Serio','Face','Side']
    for(let i=0;i<sequenceMap.length;i++){
      const f=sorted[startIdx+i]
      if(!f)break
      const slot=sequenceMap[i]
      if(slot)loadImageIntoSlot(f,slot)
    }
  }
}
function loadFullCollection(){
  if(!selectedCollection.value)return
  const {patientId,collectionId}=selectedCollection.value
  clearAll()
  allFiles.value.forEach(f=>{
    const {patientId:pid,collectionId:cid,viewToken}=parseFilename(f.name)
    if(pid!==patientId)return
    if(collectionId){
      if(cid!==collectionId)return
    }else{
      const tokens=f.name.replace(/\.[^.]+$/,'').split('_')
      if(tokens.length!==4)return
    }
    const slot=mapView(viewToken)
    if(slot)loadImageIntoSlot(f,slot)
  })
}
function loadImageIntoSlot(fileObj,slot){
  const img=new Image()
  img.onload=async()=>{
    images[slot]=img
    fileNames[slot]=fileObj.name
    Object.assign(transforms[slot],{rotate:0,flipH:false,flipV:false,zoom:1,offsetX:0,offsetY:0})
    undoStacks[slot]=[]
    await nextTick()
    drawSlot(slot)
  }
  img.src=fileObj.url
}

/* dibujo */
function drawSlot(slot){
  const cref=refsMap[slot]
  const canvas=cref&&cref.value
  const img=images[slot]
  if(!canvas||!img)return
  const ctx=canvas.getContext('2d')
  const w=Math.round(canvas.clientWidth)
  const h=Math.round(canvas.clientHeight)
  canvas.width=w;canvas.height=h
  ctx.clearRect(0,0,w,h)
  ctx.fillStyle='#fff';ctx.fillRect(0,0,w,h)
  const t=transforms[slot]
  const baseScale=Math.min(w/img.width,h/img.height)||1
  const totalScale=baseScale*(t.zoom||1)
  ctx.save()
  ctx.translate(w/2,h/2)
  ctx.rotate((t.rotate||0)*Math.PI/180)
  ctx.scale((t.flipH?-1:1)*totalScale,(t.flipV?-1:1)*totalScale)
  const offsetXAdj=(t.offsetX||0)/(totalScale||1)
  const offsetYAdj=(t.offsetY||0)/(totalScale||1)
  ctx.drawImage(img,-img.width/2+offsetXAdj,-img.height/2+offsetYAdj,img.width,img.height)
  ctx.restore()
}
function redrawAll(){slots.forEach(s=>{if(images[s])drawSlot(s)})}

/* clear & save */
function clearAll(){
  slots.forEach(s=>{
    images[s]=null
    fileNames[s]=null
    Object.assign(transforms[s],{rotate:0,flipH:false,flipV:false,zoom:1,offsetX:0,offsetY:0})
    undoStacks[s]=[]
  })
  selectedFile.value=null
  selectedSlot.value=null
  selectedCollection.value=null
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
  // actualizamos rightPanelWidth a 440 para coincidir con UI
  const rightPanelWidth = 440
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
/* Base layout (igual a la versión buena) */
.app-root { display:flex; height:100vh; background:#f6f7f8; font-family:sans-serif; box-sizing:border-box; }

/* LEFT (frames) */
.left-column { flex:1; display:grid; grid-template-rows:auto auto auto; gap:12px; padding:12px; box-sizing:border-box; justify-items:center; overflow-y:auto; }
.row { width:100%; display:flex; justify-content:center; }
.row-1, .row-3 { display:grid; grid-template-columns:repeat(3,var(--frame-w)); gap:12px; justify-content:center; }
.row2-inner { display:flex; gap:12px; justify-content:center; }

/* FRAME */
.frame { background:#fff; border:1.5px solid #e2e6ea; border-radius:8px; box-shadow:0 1px 4px rgba(20,20,20,0.03); display:flex; align-items:center; justify-content:center; overflow:hidden; position:relative; }
.frame-horizontal { width:var(--frame-w); height:var(--frame-h); min-width:120px; min-height:90px; }
.frame-vertical { width:var(--frame-w); height:calc(var(--frame-w)*1.3333); min-width:120px; min-height:160px; }
.placeholder { color:#6b7280; font-weight:700; }
.frame canvas { width:100%; height:100%; display:block; }
.frame.selected { border:2px solid #0b63d6; box-shadow:0 6px 18px rgba(11,99,214,0.12); }

/* RIGHT panel (ancho aumentado a 440px) */
.right-panel { width:440px; box-sizing:border-box; border-left:1px solid #e6e8ea; background:#fff; display:flex; flex-direction:column; overflow:auto; padding:14px; }

/* toolbar */
.toolbar { display:flex; align-items:center; justify-content:space-between; margin-bottom:10px; }
.toolbar-title { margin:0; font-size:16px; font-weight:600; }
.quick-save { background:transparent; border:none; cursor:pointer; font-size:18px; }

/* secciones */
.panel-section { padding:10px 0; border-bottom:1px solid #f1f3f4; }
.panel-section h4 { margin:0 0 6px 0; font-size:14px; }
.muted { color:#6b7280; font-size:13px; margin-bottom:6px; }

/* botones */
.btn { display:inline-block; padding:8px 10px; border-radius:6px; border:1px solid #cbd5e1; background:#f8fafc; cursor:pointer; margin-right:8px; }
.btn:disabled { opacity:0.5; cursor:not-allowed; }
.btn.primary { background:#0b63d6; color:#fff; border-color:#0b63d6; }
.btn.full { width:100%; }

/* acciones */
.actions-row { display:flex; gap:8px; }

/* file-list reducido a 200px */
.file-list { max-height:200px; overflow:auto; margin-top:8px; border:1px solid #e5e7eb; background:#fff; border-radius:6px; }
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

/* transform controls */
.transform-grid .row-btns { display:flex; gap:6px; flex-wrap:wrap; }
.btn.small { padding:6px 8px; font-size:13px; }
.btn.tiny { padding:4px 6px; font-size:12px; }
.nudge-block { display:flex; flex-direction:column; align-items:center; }
.nudge-hint { color:#6b7280; font-size:12px; }

/* sticky save */
.sticky-save { position:sticky; bottom:0; background:#fff; padding-top:10px; padding-bottom:6px; border-top:1px solid #e6e8ea; }

/* responsive */
@media (max-width:920px) {
  .app-root { flex-direction:column; }
  .right-panel { width:100%; border-left:none; border-top:1px solid #e6e8ea; }
  :root { --frame-w: 140px; --frame-h: 105px; }
}

/* ICONS & NUDGES - ajustes estéticos */
.icon-row {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  justify-content: flex-start;
  align-items: center;
}
.icon-btn {
  width: 36px;
  height: 36px;
  padding: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
}
.icon {
  width: 18px;
  height: 18px;
  stroke: #374151;
  fill: none;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
}
.icon-btn:hover .icon { stroke: #0b63d6; }
.nudge-inline {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-left: 10px;
}
.nudge-inline .btn.tiny {
  min-width: 34px;
  padding: 4px 6px;
  text-align: center;
  border-radius: 6px;
}
</style>
