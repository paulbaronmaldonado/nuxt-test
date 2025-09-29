<template>
  <div class="grid-wrapper">
    <div class="grid-container">
      <div class="slot face">Face</div>

      <div class="slot maxi">Maxi</div>
      <div class="slot side">Side</div>

      <div class="slot mand">Mand</div>

      <!-- bottom row -->
      <div class="slot rite">Rite</div>
      <div class="slot fore">Fore</div>
      <div class="slot left">Left</div>
    </div>
  </div>
</template>

<script setup>
// Versión de grid con columnas del mismo ancho
</script>

<style scoped>
.grid-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
}

.grid-container {
  width: 100%;
  max-width: 1400px;
  padding: 18px;
  box-sizing: border-box;

  display: grid;
  grid-template-columns: 1fr 1fr 1fr; /* todas iguales */
  grid-template-rows: minmax(260px, 1fr) minmax(260px, 1fr) 260px;
  gap: 18px;

  grid-template-areas:
    "face maxi side"
    "face mand side"
    "rite fore left";
}

.slot {
  border: 2px solid #ddd;
  border-radius: 8px;
  background: #fbfbfb;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #222;
  font-weight: 700;
  font-size: 1.2rem;
  box-shadow: 0 1px 0 rgba(0,0,0,0.03) inset;
  min-height: 120px;
  padding: 8px;
}

.face { grid-area: face; }
.maxi { grid-area: maxi; }
.mand { grid-area: mand; }
.side { grid-area: side; }
.rite { grid-area: rite; }
.fore { grid-area: fore; }
.left { grid-area: left; }

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
  .slot { font-size: 1rem; min-height: 160px; }
}
</style>
