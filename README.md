<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Trova karaokes</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Arizonia&family=Baloo+2:wght@700&family=Caprasimo&family=Caveat+Brush&family=Chewy&family=Francois+One&family=Molle&family=Special+Elite&family=Black+Ops+One&family=Playfair+Display:wght@900&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0f0f14; --panel: #1a1a24; --panel2: #24243a;
    --accent: #ff3b7f; --accent2: #4fc3ff; --text: #eee;
    --muted: #888; --ok: #4ade80; --warn: #fbbf24;
  }
  * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  html, body { overflow-y: auto !important; overflow-x: hidden; }
  body {
    margin: 0; padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    background: var(--bg); color: var(--text);
    padding-bottom: 30px;
  }
  header {
    background: linear-gradient(90deg, var(--accent), var(--accent2));
    padding: 8px 16px 4px; text-align: center;
  }
  header h1 { margin: 0; font-size: 18px; font-weight: bold; color: white; }
  header .dev { font-size: 9px; color: rgba(255,255,255,0.85); margin-top: 1px; }

  #bigTimer {
    background: var(--panel); padding: 10px; text-align: center;
    font-size: 36px; font-weight: bold; font-family: 'Courier New', monospace;
    color: var(--accent2); border-bottom: 1px solid var(--panel2);
  }

  #previewWrap {
    width: 100%; aspect-ratio: 16/9; background: #000;
    border-bottom: 1px solid var(--panel2);
  }
  #previewCanvas { width: 100%; height: 100%; display: block; }

  .mini-player {
    background: var(--panel2); padding: 6px 10px;
    display: flex; align-items: center; gap: 6px; flex-wrap: wrap;
    border-bottom: 1px solid var(--panel2);
  }
  .mini-player button { padding: 5px 10px; font-size: 11px; }
  .mini-player .time-display {
    flex: 1; text-align: center; font-size: 11px;
    color: var(--accent2); font-family: 'Courier New', monospace;
    min-width: 120px;
  }
  #progressSlider { width: 100%; height: 18px; }

  #timelineWrap {
    position: relative; width: 100%; height: 240px;
    background: var(--panel2); border-bottom: 1px solid #000;
    overflow-y: auto; overflow-x: hidden;
    user-select: none;
  }
  #timelineWrap::-webkit-scrollbar { width: 8px; }
  #timelineWrap::-webkit-scrollbar-thumb { background: var(--accent); border-radius: 4px; }
  #timelineCanvas { width: 100%; display: block; }

  .split-bar {
    background: var(--panel); padding: 8px;
    border-bottom: 1px solid var(--panel2);
  }
  .split-bar button {
    width: 100%; background: var(--warn); color: #000;
    padding: 10px; font-size: 14px; font-weight: bold;
    border: none; border-radius: 6px; cursor: pointer;
  }

  .tabs {
    display: flex; background: var(--panel);
    border-bottom: 2px solid var(--panel2);
    overflow-x: auto; position: sticky; top: 0; z-index: 50;
  }
  .tab {
    flex: 1; min-width: 80px; padding: 10px 6px;
    text-align: center; cursor: pointer;
    font-size: 11px; font-weight: bold;
    color: var(--muted); border-bottom: 3px solid transparent;
    white-space: nowrap;
  }
  .tab.active {
    color: var(--accent); border-bottom-color: var(--accent);
    background: var(--panel2);
  }

  .tab-content {
    display: none; padding: 10px;
    background: var(--panel);
    min-height: 150px;
    border-bottom: 1px solid var(--panel2);
  }
  .tab-content.active { display: block; }

  button {
    background: var(--accent); color: white; border: none;
    padding: 8px 12px; border-radius: 6px; font-weight: bold;
    cursor: pointer; margin: 3px 2px; font-size: 12px;
  }
  button.secondary { background: var(--panel2); }
  button.ok { background: var(--ok); color: #000; }
  button.warn { background: var(--warn); color: #000; }
  button.danger { background: #ef4444; }
  button:active { transform: scale(0.95); }

  textarea, select, input[type="text"], input[type="number"] {
    width: 100%; background: var(--panel2); color: var(--text);
    border: 1px solid #333; border-radius: 6px; padding: 8px;
    font-size: 12px; margin: 4px 0;
  }
  textarea { min-height: 80px; resize: vertical; font-family: inherit; }
  label { display: block; margin: 6px 0 3px; color: var(--muted); font-size: 11px; }
  input[type="color"], input[type="range"] { width: 100%; background: transparent; }
  input[type="range"] { height: 24px; }

  .controls { display: flex; flex-wrap: wrap; gap: 4px; justify-content: center; margin: 6px 0; }
  .tap-btn {
    background: linear-gradient(135deg, var(--accent), #ff6b9f);
    font-size: 16px; padding: 14px 20px; border-radius: 40px;
    flex: 1; min-width: 140px;
  }
  .mode-toggle { display: flex; background: var(--panel2); border-radius: 20px; overflow: hidden; margin: 8px 0; }
  .mode-btn { flex: 1; padding: 8px; text-align: center; cursor: pointer; font-size: 12px; font-weight: bold; }
  .mode-btn.active { background: var(--accent); }

  .line-nav { display: flex; gap: 4px; margin: 8px 0; }
  .line-nav button { flex: 1; }

  .lines-list { max-height: 180px; overflow-y: auto; background: var(--panel2); border-radius: 6px; padding: 6px; }
  .line-item {
    padding: 5px 8px; margin: 2px 0; background: var(--panel);
    border-radius: 4px; font-size: 11px; display: flex;
    justify-content: space-between; align-items: center;
  }
  .line-item.active { background: var(--accent2); color: #000; }
  .line-item.synced { border-left: 3px solid var(--ok); }
  .line-item.empty { opacity: 0.3; font-style: italic; }

  .words-container {
    display: flex; flex-wrap: wrap; gap: 4px; padding: 6px;
    background: var(--panel2); border-radius: 6px;
    min-height: 40px; max-height: 120px; overflow-y: auto;
  }
  .word-chip {
    background: var(--panel); color: var(--text);
    padding: 3px 7px; border-radius: 8px; font-size: 11px;
    cursor: pointer; user-select: none; border: 1px solid #444;
  }
  .word-chip.synced { background: var(--ok); color: #000; }
  .word-chip.current { background: var(--accent2); color: #000; }
  .word-chip .scissors { margin-left: 3px; color: var(--warn); font-weight: bold; }

  .custom-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 6px; }
  .custom-grid .full { grid-column: 1 / -1; }

  #status {
    position: fixed; bottom: 0; left: 0; right: 0;
    background: var(--panel); padding: 6px; text-align: center;
    font-size: 10px; color: var(--muted); border-top: 1px solid #333; z-index: 50;
  }
  .help-box {
    background: rgba(79,195,255,0.1); border-left: 3px solid var(--accent2);
    padding: 5px 8px; font-size: 10px; color: var(--text);
    border-radius: 4px; margin: 4px 0;
  }

  .nav-float {
    position: fixed; right: 10px; bottom: 40px; z-index: 200;
    display: flex; flex-direction: column; gap: 6px;
  }
  .nav-float button {
    width: 44px; height: 44px; border-radius: 50%;
    font-size: 20px; padding: 0; margin: 0;
    background: var(--accent2); color: #000;
    box-shadow: 0 2px 8px rgba(0,0,0,0.5);
  }

  .btn-row { display: flex; flex-wrap: wrap; gap: 6px; margin: 4px 0; }
  .btn-row button { flex: 1; min-width: 140px; }
  .folio-badge {
    display: inline-block; background: var(--panel2); border: 1px solid var(--accent);
    color: var(--accent2); font-family: 'Courier New', monospace;
    padding: 3px 10px; border-radius: 6px; font-weight: bold; letter-spacing: 2px; font-size: 13px;
  }

  @media (min-width: 768px) {
    #previewWrap { max-height: 40vh; }
    #bigTimer { font-size: 28px; }
  }
</style>
</head>
<body>

<header>
  <h1>Trova karaokes</h1>
  <div class="dev">Desarrollador: Pedro Antonio Jiménez Hernández</div>
</header>

<div id="bigTimer">0:00.00</div>

<div id="previewWrap">
  <canvas id="previewCanvas" width="1280" height="720"></canvas>
</div>

<div class="mini-player">
  <button id="playPauseBtn" onclick="togglePlayPause()">▶️</button>
  <button class="secondary" onclick="audioPlayer.currentTime = Math.max(0, audioPlayer.currentTime - 1)">-1s</button>
  <div class="time-display" id="miniTimeDisplay">0:00.00 / 0:00</div>
  <button class="secondary" onclick="audioPlayer.currentTime += 1">+1s</button>
  <input type="range" id="progressSlider" min="0" max="100" value="0" step="0.1">
</div>

<div id="timelineWrap"><canvas id="timelineCanvas"></canvas></div>

<div class="split-bar">
  <button onclick="splitAtCursor()">✂️ Dividir bloque en el cursor rojo (cada parte con sus handles)</button>
</div>

<div class="tabs">
  <div class="tab" data-tab="audio">🎵 Audio</div>
  <div class="tab" data-tab="sync">🎤 Sincro</div>
  <div class="tab" data-tab="editor">✏️ Editor</div>
  <div class="tab" data-tab="styles">🎨 Estilos</div>
  <div class="tab" data-tab="project">💾 Proyecto</div>
  <div class="tab" data-tab="export">🎬 Exportar</div>
</div>

<div class="tab-content" id="tab-audio">
  <label>🎵 Cargar audio (MP3/WAV):</label>
  <input type="file" id="audioFile" accept="audio/*,.mp3,.wav">
  <audio id="audioPlayer" controls style="width:100%; margin-top:6px;"></audio>
  <label>📝 Letra (líneas vacías = puentes musicales):</label>
  <textarea id="lyricsInput" placeholder="Pega la letra..."></textarea>
  <div class="controls">
    <button onclick="processLyrics()">Procesar letra</button>
    <button class="secondary" onclick="document.getElementById('lyricsFile').click()">📄 Importar .txt</button>
  </div>
  <input type="file" id="lyricsFile" accept=".txt,text/plain" style="display:none;">
</div>

<div class="tab-content" id="tab-sync">
  <div class="mode-toggle">
    <div class="mode-btn active" id="modeBtnLines" onclick="setMode('lines')">📝 Líneas</div>
    <div class="mode-btn" id="modeBtnWords" onclick="setMode('words')">🔤 Palabras</div>
  </div>
  <div class="help-box" id="modeHelp">Modo Líneas: sincroniza renglón completo</div>
  <div class="controls">
    <button onclick="audioPlayer.play()">▶️ Play</button>
    <button onclick="audioPlayer.pause()">⏸️ Pausa</button>
    <button class="secondary" onclick="audioPlayer.currentTime = Math.max(0, audioPlayer.currentTime - 3)">⏪ -3s</button>
    <button class="secondary" onclick="audioPlayer.currentTime += 3">⏩ +3s</button>
  </div>
  <div class="controls">
    <button class="secondary" onclick="setSpeed(0.5)">0.5x</button>
    <button class="secondary" onclick="setSpeed(0.75)">0.75x</button>
    <button class="secondary" onclick="setSpeed(1)">1x</button>
    <button class="secondary" onclick="setSpeed(1.25)">1.25x</button>
  </div>
  <button class="tap-btn" onclick="tapSync()">🎤 TAP (Espacio = TAP, doble = Play/Pausa)</button>
  <div class="line-nav">
    <button class="secondary" onclick="prevLine()">◀ Anterior</button>
    <button class="secondary" onclick="nextLine()">Siguiente ▶</button>
    <button class="warn" onclick="undoLast()">↩️ Deshacer</button>
  </div>
  <p style="text-align:center; font-size:11px; color:var(--muted);">
    Línea: <span id="currentLineIdx">-</span>/<span id="totalLines">0</span>
  </p>
</div>

<div class="tab-content" id="tab-editor">
  <label>Líneas:</label>
  <div class="lines-list" id="linesList"></div>
  <label style="margin-top:8px;">Palabras (línea actual):</label>
  <div class="words-container" id="wordsContainer"></div>
</div>

<div class="tab-content" id="tab-styles">
  <div class="custom-grid">
    <div class="full">
      <label>📄 Renglones visibles (grupo fijo en la pantalla y en la línea de tiempo):</label>
      <select id="visibleLines">
        <option value="1">1 renglón</option>
        <option value="2">2 renglones</option>
        <option value="3">3 renglones</option>
        <option value="4" selected>4 renglones</option>
        <option value="5">5 renglones</option>
        <option value="6">6 renglones</option>
      </select>
      <div class="help-box">Con 4 elegido, la pantalla y los bloques verdes siempre se organizan de 4 en 4, verticales uno tras otro; nunca se muestran 2, 3, 5 u 6 renglones.</div>
    </div>
    <div class="full">
      <label>🔤 Fuente:</label>
      <select id="fontFamily">
        <optgroup label="Fuentes del sistema">
          <option value="Arial">Arial</option>
          <option value="Arial Black">Arial Black</option>
          <option value="Impact">Impact</option>
          <option value="Georgia">Georgia</option>
          <option value="Courier New">Courier New</option>
          <option value="Verdana">Verdana</option>
          <option value="Playbill">Playbill</option>
          <option value="Times New Roman">Times New Roman</option>
        </optgroup>
        <optgroup label="Fuentes Google">
          <option value="Black Ops One">Black Ops One</option>
          <option value="Arizonia">Arizonia</option>
          <option value="Baloo 2">Baloo 2</option>
          <option value="Caprasimo">Caprasimo</option>
          <option value="Caveat Brush">Caveat Brush</option>
          <option value="Chewy">Chewy</option>
          <option value="Francois One">Francois One</option>
          <option value="Molle">Molle</option>
          <option value="Playfair Display">Playfair Display Black</option>
          <option value="Special Elite">Special Elite (Máquina)</option>
        </optgroup>
      </select>
    </div>
    <div>
      <label>Tamaño: <span id="fontSizeVal">60</span>px</label>
      <input type="range" id="fontSize" min="20" max="150" value="60">
    </div>
    <div>
      <label>Posición Y: <span id="posYVal">50</span>%</label>
      <input type="range" id="posY" min="10" max="90" value="50">
    </div>
    <div>
      <label>Color letra:</label>
      <input type="color" id="textColor" value="#ffffff">
    </div>
    <div>
      <label>Color sync:</label>
      <input type="color" id="syncColor" value="#ff3b7f">
    </div>
    <div class="full">
      <label>🎨 Fondo:</label>
      <select id="bgType">
        <option value="color">Color sólido</option>
        <option value="gradient">Gradiente</option>
        <option value="image">Imagen</option>
        <option value="transparent">🎬 Transparente (sin fondo)</option>
      </select>
    </div>
    <div id="bgColorWrap">
      <label>Color:</label>
      <input type="color" id="bgColor" value="#000000">
    </div>
    <div id="bgColor2Wrap" style="display:none;">
      <label>Color 2:</label>
      <input type="color" id="bgColor2" value="#1a1a2e">
    </div>
    <div id="gradientControls" style="display:none;" class="full">
      <label>Ángulo gradiente: <span id="gradAngleVal">135</span>°</label>
      <input type="range" id="gradientAngle" min="0" max="360" value="135">
      <label>Posición color 1: <span id="gradPos1Val">0</span>%</label>
      <input type="range" id="gradientPos1" min="0" max="100" value="0">
      <label>Posición color 2: <span id="gradPos2Val">100</span>%</label>
      <input type="range" id="gradientPos2" min="0" max="100" value="100">
    </div>
    <div id="bgImageWrap" style="display:none;" class="full">
      <label>Imagen:</label>
      <input type="file" id="bgImage" accept="image/*">
      <select id="bgFit" style="margin-top:4px;">
        <option value="cover">Cubrir</option>
        <option value="contain">Contener</option>
        <option value="fill">Estirar</option>
      </select>
    </div>
    <div class="full">
      <label>Efectos:</label>
      <div style="display:flex; flex-wrap:wrap; gap:4px; font-size:11px;">
        <label><input type="checkbox" id="fxColor" checked> Color</label>
        <label><input type="checkbox" id="fxFill"> Relleno</label>
        <label><input type="checkbox" id="fxBounce"> Rebote</label>
        <label><input type="checkbox" id="fxGlow"> Brillo</label>
        <label><input type="checkbox" id="fxUnderline"> Subrayado</label>
        <label><input type="checkbox" id="fxShadow"> Sombra</label>
        <label><input type="checkbox" id="fxTypewriter"> Máquina</label>
      </div>
    </div>
  </div>
</div>

<div class="tab-content" id="tab-project">
  <label>Nombre del karaoke (se llena solo con el título del audio o del .txt):</label>
  <input type="text" id="projectName" placeholder="Ej. La de la mala suerte" autocomplete="off">
  <label>Folio automático (0001 → 1000 y se reinicia a 0001): <span class="folio-badge" id="folioBadge">0001</span></label>

  <div class="btn-row">
    <button class="ok" onclick="saveToFolder(false)">📁 Guardar en carpeta (ruta manual)</button>
    <button class="secondary" onclick="downloadProject">⬇️ Descargar proyecto (.json)</button>
  </div>
  <div class="btn-row">
    <button class="warn" onclick="exportFinal">✅ Exportar karaoke FINAL (folio + nombre)</button>
    <button class="secondary" onclick="document.getElementById('projectFileInput').click()">📂 Abrir proyecto (.json)</button>
  </div>
  <div class="btn-row">
    <button class="secondary" onclick="loadProject">Cargar respaldo del navegador</button>
    <button class="danger" onclick="resetProject()">🔄 Reiniciar todo</button>
  </div>
  <input type="file" id="projectFileInput" accept=".json,application/json" style="display:none;">
  <div class="help-box">
    Los guardados de prueba suman folio automáticamente (0001, 0002…). "Exportar karaoke FINAL" conserva el folio actual y el nombre del karaoke para el archivo final exportable.<br>
    "Guardar en carpeta" abre el selector manual de carpeta (Chrome/Edge de escritorio). En móvil o Safari el archivo se descarga automáticamente a la carpeta de descargas.
  </div>
</div>

<div class="tab-content" id="tab-export">
  <div class="help-box">
    🎬 Exporta tu karaoke como video. <b>Si elegiste fondo transparente</b>, el video tendrá fondo negro que puedes reemplazar en Filmora con chroma key.
  </div>
  <button class="ok" onclick="exportVideo()" style="width:100%; padding:14px; font-size:14px;">
    🎬 Crear video MP4/WebM
  </button>
  <p id="exportStatus" style="font-size:11px; color:var(--muted); margin-top:8px; text-align:center;"></p>
</div>

<div id="status">Listo</div>

<div class="nav-float">
  <button onclick="window.scrollTo({top:0, behavior:'smooth'})">⬆️</button>
  <button onclick="window.scrollTo({top:document.body.scrollHeight, behavior:'smooth'})">⬇️</button>
</div>

<script>
const state = {
  lines: [], currentLineIdx: -1, currentWordIdx: -1, mode: 'lines',
  audioDuration: 0, zoom: 1, scrollX: 0, bgImage: null,
  bgImageDataUrl: null, syncHistory: [], projectName: ''
};

const audioPlayer = document.getElementById('audioPlayer');
const previewCanvas = document.getElementById('previewCanvas');
const previewCtx = previewCanvas.getContext('2d', { alpha: true });
const timelineCanvas = document.getElementById('timelineCanvas');
const timelineCtx = timelineCanvas.getContext('2d');

// Geometría fija de la línea de tiempo: grupos verticales de N renglones
const TL = { BLOCK_H: 22, GAP: 3, BAND_GAP: 12, HEADER: 16, FOOTER: 48, HANDLE_W: 12 };
let dirHandle = null;

// ============ FOLIO 0001 → 1000 (se reinicia) ============
const FOLIO_KEY = 'karaokeFolio';
const MAX_FOLIO = 1000;
function getFolio(){ const n = parseInt(localStorage.getItem(FOLIO_KEY), 10); return (n >= 1 && n <= MAX_FOLIO) ? n : 1; }
function setFolio(n){ localStorage.setItem(FOLIO_KEY, String(n)); }
function nextFolio(){ let n = getFolio(); n = (n >= MAX_FOLIO) ? 1 : n + 1; setFolio(n); return n; }
function folioText(n){ return String(n).padStart(4, '0'); }
function safeFileName(name){ return (name || 'karaoke').replace(/[\\/:*?"<>|]+/g, '-').trim().slice(0, 60) || 'karaoke'; }
function refreshFolioBadge(){ const el = document.getElementById('folioBadge'); if (el) el.textContent = folioText(getFolio()); }

// ============ PESTAÑAS TOGGLE ============
let tabActiva = null;
document.querySelectorAll('.tab').forEach(function(tab) {
  tab.addEventListener('click', function() {
    var id = this.getAttribute('data-tab');
    var contenido = document.getElementById('tab-' + id);
    if (!contenido) return;
    if (tabActiva === id) {
      this.classList.remove('active');
      contenido.classList.remove('active');
      tabActiva = null;
    } else {
      document.querySelectorAll('.tab').forEach(function(t) { t.classList.remove('active'); });
      document.querySelectorAll('.tab-content').forEach(function(c) { c.classList.remove('active'); });
      this.classList.add('active');
      contenido.classList.add('active');
      tabActiva = id;
    }
  });
});

function togglePlayPause() {
  if (audioPlayer.paused) audioPlayer.play();
  else audioPlayer.pause();
}
audioPlayer.addEventListener('play', () => { document.getElementById('playPauseBtn').textContent = '⏸️'; });
audioPlayer.addEventListener('pause', () => { document.getElementById('playPauseBtn').textContent = '▶️'; });

const progressSlider = document.getElementById('progressSlider');
progressSlider.addEventListener('input', (e) => {
  if (state.audioDuration > 0) audioPlayer.currentTime = (e.target.value / 100) * state.audioDuration;
});
audioPlayer.addEventListener('timeupdate', () => {
  if (state.audioDuration > 0) progressSlider.value = (audioPlayer.currentTime / state.audioDuration) * 100;
});

function updateTimer() {
  const t = audioPlayer.currentTime || 0;
  const mins = Math.floor(t / 60);
  const secs = Math.floor(t % 60);
  const ms = Math.floor((t % 1) * 100);
  document.getElementById('bigTimer').textContent = `${mins}:${secs.toString().padStart(2,'0')}.${ms.toString().padStart(2,'0')}`;
  const totalMins = Math.floor(state.audioDuration / 60);
  const totalSecs = Math.floor(state.audioDuration % 60);
  document.getElementById('miniTimeDisplay').textContent = `${mins}:${secs.toString().padStart(2,'0')}.${ms.toString().padStart(2,'0')} / ${totalMins}:${totalSecs.toString().padStart(2,'0')}`;
}

function setSpeed(s) { audioPlayer.playbackRate = s; setStatus(`Velocidad: ${s}x`); }

function setMode(m) {
  state.mode = m;
  document.getElementById('modeBtnLines').classList.toggle('active', m === 'lines');
  document.getElementById('modeBtnWords').classList.toggle('active', m === 'words');
  document.getElementById('modeHelp').textContent = m === 'lines' ? 'Modo Líneas: sincroniza renglón completo' : 'Modo Palabras: divide palabras para edición';
  renderLines(); renderWords(); drawTimeline();
}

function defaultNameFromFile(name){
  const base = name.replace(/\.[^.]+$/, '').replace(/[-_]+/g, ' ').trim();
  if (base && !state.projectName) { state.projectName = base; document.getElementById('projectName').value = base; }
}

document.getElementById('audioFile').addEventListener('change', (e) => {
  const file = e.target.files[0];
  if (!file) return;
  defaultNameFromFile(file.name);
  const url = URL.createObjectURL(file);
  audioPlayer.src = url;
  audioPlayer.addEventListener('loadedmetadata', () => {
    state.audioDuration = audioPlayer.duration;
    setStatus(`✅ Audio: ${file.name} (${state.audioDuration.toFixed(1)}s)`);
    drawTimeline();
  }, { once: true });
});

document.getElementById('lyricsFile').addEventListener('change', (e) => {
  const file = e.target.files[0];
  if (!file) return;
  defaultNameFromFile(file.name);
  const reader = new FileReader();
  reader.onload = (ev) => {
    document.getElementById('lyricsInput').value = ev.target.result;
    setStatus(`✅ Letra importada: ${file.name}. Pulsa "Procesar letra".`);
  };
  reader.readAsText(file, 'UTF-8');
  e.target.value = '';
});

document.getElementById('projectName').addEventListener('input', (e) => {
  state.projectName = e.target.value.trim();
  saveProject();
});

function processLyrics() {
  const text = document.getElementById('lyricsInput').value;
  const linesText = text.split('\n');
  state.lines = linesText.map(line => {
    const trimmed = line.trim();
    const words = trimmed ? trimmed.split(/\s+/).filter(w => w) : [];
    return { text: trimmed, time: null, duration: null, isEmpty: trimmed === '', words: words.map(w => ({ text: w, time: null, duration: null })) };
  });
  state.currentLineIdx = -1; state.currentWordIdx = -1; state.syncHistory = [];
  renderLines(); renderWords(); drawTimeline();
  const totalWords = state.lines.reduce((s, l) => s + l.words.length, 0);
  const nonEmpty = state.lines.filter(l => !l.isEmpty).length;
  document.getElementById('totalLines').textContent = nonEmpty;
  setStatus(`${nonEmpty} líneas con texto, ${totalWords} palabras`);
  saveProject();
}

function renderLines() {
  const container = document.getElementById('linesList');
  container.innerHTML = '';
  state.lines.forEach((line, i) => {
    const div = document.createElement('div');
    div.className = 'line-item';
    if (i === state.currentLineIdx) div.classList.add('active');
    if (line.time !== null) div.classList.add('synced');
    if (line.isEmpty) div.classList.add('empty');
    div.innerHTML = `<span>${i+1}. ${line.isEmpty ? '(vacío)' : line.text}</span><span style="font-size:10px; color:var(--muted);">${line.time !== null ? line.time.toFixed(2)+'s' : '-'}</span>`;
    container.appendChild(div);
  });
}

function renderWords() {
  const container = document.getElementById('wordsContainer');
  container.innerHTML = '';
  if (state.currentLineIdx < 0 || state.currentLineIdx >= state.lines.length) return;
  const line = state.lines[state.currentLineIdx];
  if (line.isEmpty) { container.innerHTML = '<span style="color:var(--muted);">(vacío)</span>'; return; }
  if (state.mode === 'words') {
    line.words.forEach((w, i) => {
      const chip = document.createElement('span');
      chip.className = 'word-chip';
      if (w.time !== null) chip.classList.add('synced');
      chip.innerHTML = `${w.text}<span class="scissors">✂</span>`;
      chip.querySelector('.scissors').onclick = (e) => { e.stopPropagation(); splitWord(i); };
      container.appendChild(chip);
    });
  } else {
    const chip = document.createElement('span');
    chip.className = 'word-chip';
    if (line.time !== null) chip.classList.add('synced');
    chip.textContent = line.text;
    container.appendChild(chip);
  }
}

// Divisor manual de palabra (tijeras del chip): deja dos palabras con tiempo propio
function splitWord(wordIdx) {
  const line = state.lines[state.currentLineIdx];
  if (!line || !line.words) return;
  const w = line.words[wordIdx];
  if (!w || w.text.length < 2) { setStatus('⚠️ Palabra muy corta para dividir'); return; }
  const cut = Math.floor(w.text.length / 2);
  const half = Math.max(0.08, (w.duration || 0.3) / 2);
  const base = w.time ?? 0;
  const w1 = { text: w.text.slice(0, cut), time: w.time, duration: half };
  const w2 = { text: w.text.slice(cut), time: base + half, duration: half };
  line.words.splice(wordIdx, 1, w1, w2);
  setStatus(`✂️ "${w1.text}" | "${w2.text}" (cada una con handles)`);
  renderWords(); drawTimeline(); saveProject();
}

// Divide el bloque bajo el cursor rojo; cada parte conserva su propio tiempo y handles
function splitAtCursor() {
  const t = audioPlayer.currentTime;
  if (state.lines.length === 0) { setStatus('⚠️ Primero procesa la letra'); return; }

  if (state.mode === 'words') {
    let found = null;
    state.lines.forEach((line, li) => {
      if (line.isEmpty || !line.words) return;
      line.words.forEach((w, wi) => {
        if (w.time === null) return;
        const end = w.time + (w.duration || 0.3);
        if (t >= w.time && t <= end) found = { li, wi, w, end };
      });
    });
    if (!found) { setStatus('⚠️ No hay bloque de palabra bajo el cursor rojo'); return; }
    const { li, wi, w, end } = found;
    if (t - w.time < 0.08 || end - t < 0.08) { setStatus('⚠️ Cursor muy cerca del borde; muévelo un poco'); return; }
    const frac = (t - w.time) / Math.max(0.01, end - w.time);
    const cut = Math.max(1, Math.min(w.text.length - 1, Math.round(w.text.length * frac)));
    const w1 = { text: w.text.slice(0, cut), time: w.time, duration: Math.max(0.08, t - w.time) };
    const w2 = { text: w.text.slice(cut), time: t, duration: Math.max(0.08, end - t) };
    state.lines[li].words.splice(wi, 1, w1, w2);
    state.currentLineIdx = li; state.currentWordIdx = wi + 1;
    setStatus(`✂️ "${w1.text}" | "${w2.text}" con handles propios en ${t.toFixed(2)}s`);
    renderWords(); drawTimeline(); saveProject();
    return;
  }

  // Modo líneas: divide el renglón en dos bloques independientes, cada uno con handles
  let target = -1, targetEnd = 0, targetDist = Infinity;
  state.lines.forEach((line, li) => {
    if (line.isEmpty || line.time === null) return;
    const end = line.time + (line.duration || 3);
    if (t >= line.time && t <= end) { target = li; targetEnd = end; }
    else {
      const d = Math.min(Math.abs(t - line.time), Math.abs(t - end));
      if (d < targetDist) { targetDist = d; target = li; targetEnd = end; }
    }
  });
  if (target === -1) { setStatus('⚠️ No hay renglones sincronizados cerca del cursor'); return; }
  const line = state.lines[target];
  if (line.time === null) { line.time = t; line.duration = 3; targetEnd = t + 3; }
  if (t - line.time < 0.08 || targetEnd - t < 0.08) { setStatus('⚠️ Cursor muy cerca del borde del bloque'); return; }

  const frac = (t - line.time) / Math.max(0.01, targetEnd - line.time);
  const words = line.text.split(/\s+/).filter(Boolean);
  if (words.length < 2) { setStatus('⚠️ Renglón muy corto para partir en dos bloques'); return; }
  const cutWords = Math.max(1, Math.min(words.length - 1, Math.round(words.length * frac)));
  const leftWords = words.slice(0, cutWords);
  const rightWords = words.slice(cutWords);
  const leftText = leftWords.join(' ');
  const rightText = rightWords.join(' ');

  line.text = leftText;
  line.duration = Math.max(0.08, t - line.time);
  line.words = leftWords.map(w => ({ text: w, time: null, duration: null }));

  const newLine = {
    text: rightText, time: t, duration: Math.max(0.08, targetEnd - t),
    isEmpty: false,
    words: rightWords.map(w => ({ text: w, time: null, duration: null }))
  };
  state.lines.splice(target + 1, 0, newLine);
  state.currentLineIdx = target + 1;
  state.currentWordIdx = 0;
  syncLyricsTextarea();
  setStatus(`✂️ Bloque dividido en ${t.toFixed(2)}s: "${leftText}" | "${rightText}" (handles propios)`);
  renderLines(); renderWords(); drawTimeline(); saveProject();
}

function syncLyricsTextarea() {
  document.getElementById('lyricsInput').value = state.lines.map(l => (l && l.text) || '').join('\n');
}

function tapSync() {
  if (state.lines.length === 0) { alert('Procesa la letra primero'); return; }
  if (audioPlayer.paused) { alert('Reproduce primero'); return; }
  const t = audioPlayer.currentTime;
  state.syncHistory.push({ line: state.currentLineIdx, word: state.currentWordIdx });
  function advance() {
    state.currentLineIdx++;
    while (state.currentLineIdx < state.lines.length && state.lines[state.currentLineIdx].isEmpty) state.currentLineIdx++;
    if (state.currentLineIdx >= state.lines.length) { state.currentLineIdx = state.lines.length - 1; setStatus('✅ Completo'); return false; }
    return true;
  }
  if (state.currentLineIdx >= 0 && state.currentLineIdx < state.lines.length) {
    const prev = state.lines[state.currentLineIdx];
    if (prev.time !== null && !prev.isEmpty) prev.duration = t - prev.time;
  }
  if (advance()) state.lines[state.currentLineIdx].time = t;
  document.getElementById('currentLineIdx').textContent = state.currentLineIdx + 1;
  renderLines(); renderWords(); drawTimeline(); saveProject();
}

function prevLine() {
  if (state.currentLineIdx > 0) {
    state.currentLineIdx--;
    while (state.currentLineIdx > 0 && state.lines[state.currentLineIdx].isEmpty) state.currentLineIdx--;
    state.currentWordIdx = 0;
    renderLines(); renderWords();
  }
}
function nextLine() {
  if (state.currentLineIdx < state.lines.length - 1) {
    state.currentLineIdx++;
    while (state.currentLineIdx < state.lines.length - 1 && state.lines[state.currentLineIdx].isEmpty) state.currentLineIdx++;
    state.currentWordIdx = 0;
    renderLines(); renderWords();
  }
}
function undoLast() {
  if (state.syncHistory.length === 0) return;
  const last = state.syncHistory.pop();
  state.currentLineIdx = last.line;
  state.currentWordIdx = last.word;
  renderLines(); renderWords(); drawTimeline();
  setStatus('↩️ Deshacer');
}

function setupTimeline() {
  const wrap = document.getElementById('timelineWrap');
  const rect = wrap.getBoundingClientRect();
  if (rect.width === 0) return;
  timelineCanvas.width = rect.width * devicePixelRatio;
  state.timelineW = rect.width;
}

function drawHandle(ctx, x, y, h, side) {
  const hw = TL.HANDLE_W; const hh = h * 0.7; const hy = y + (h - hh) / 2;
  ctx.fillStyle = '#ffffff'; ctx.strokeStyle = '#333'; ctx.lineWidth = 1;
  ctx.beginPath();
  if (ctx.roundRect) ctx.roundRect(x - hw/2, hy, hw, hh, 4);
  else { ctx.rect(x - hw/2, hy, hw, hh); }
  ctx.fill(); ctx.stroke();
  ctx.fillStyle = '#333'; ctx.font = 'bold 9px sans-serif'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
  ctx.fillText(side === 'left' ? '◀' : '▶', x, hy + hh/2);
}

function roundRectFill(ctx, x, y, w, h, r) {
  ctx.beginPath();
  if (ctx.roundRect) ctx.roundRect(x, y, w, h, r); else ctx.rect(x, y, w, h);
  ctx.fill();
}

// Layout compartido por dibujo y detección de toques: grupos verticales de N renglones
function timelineLayout() {
  const N = parseInt(document.getElementById('visibleLines').value, 10) || 4;
  const pps = 40 * state.zoom;
  const rows = [];
  state.lines.forEach((line, li) => {
    const band = Math.floor(li / N);
    const r = li % N;
    const rowY = TL.HEADER + band * (N * (TL.BLOCK_H + TL.GAP) + TL.BAND_GAP) + r * (TL.BLOCK_H + TL.GAP);
    rows.push({ li, rowY, band, r });
  });
  const bandCount = Math.max(1, Math.ceil(state.lines.length / N));
  const totalH = TL.HEADER + bandCount * (N * (TL.BLOCK_H + TL.GAP) + TL.BAND_GAP) + TL.FOOTER;
  return { N, pps, rows, totalH, bandCount };
}

// ============ TIMELINE: GRUPOS VERTICALES DE N + REGLA NUMÉRICA ============
function drawTimeline() {
  if (!state.timelineW) setupTimeline();
  if (!state.timelineW) return;
  const ctx = timelineCtx;
  const W = state.timelineW;
  const L = timelineLayout();
  const N = L.N, pps = L.pps, rows = L.rows, totalH = L.totalH;

  timelineCanvas.height = totalH * devicePixelRatio;
  timelineCtx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
  state.timelineH = totalH;
  ctx.clearRect(0, 0, W, totalH);
  ctx.fillStyle = '#171722';
  ctx.fillRect(0, 0, W, totalH);

  const rulerY = totalH - TL.FOOTER + 16;
  const startT = state.scrollX / pps;
  const endT = startT + W / pps;

  // Rejilla vertical por segundo
  ctx.lineWidth = 1;
  for (let t = Math.ceil(startT); t <= endT; t++) {
    const x = t * pps - state.scrollX;
    ctx.strokeStyle = 'rgba(255,255,255,0.05)';
    ctx.beginPath(); ctx.moveTo(x, TL.HEADER); ctx.lineTo(x, rulerY); ctx.stroke();
  }

  // Etiquetas de grupo y separadores entre grupos de N
  if (rows.length > 0) for (let b = 0; b < L.bandCount; b++) {
    const bandY = TL.HEADER + b * (N * (TL.BLOCK_H + TL.GAP) + TL.BAND_GAP);
    ctx.fillStyle = 'rgba(255,255,255,0.4)';
    ctx.font = 'bold 10px sans-serif'; ctx.textAlign = 'left'; ctx.textBaseline = 'middle';
    const first = b * N + 1;
    const last = Math.min(state.lines.length, (b + 1) * N);
    ctx.fillText(`Grupo ${b + 1} · renglones ${first}-${last}`, 6, bandY - 6);
    if (b > 0) {
      ctx.strokeStyle = '#ff3b7f'; ctx.setLineDash([5, 4]); ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.moveTo(0, bandY - TL.BAND_GAP / 2);
      ctx.lineTo(W, bandY - TL.BAND_GAP / 2);
      ctx.stroke();
      ctx.setLineDash([]);
    }
  }

  // Filas fijas (cada renglón tiene su lugar; los vacíos se reservan atenuados)
  rows.forEach(row => {
    const line = state.lines[row.li];
    const y = row.rowY;
    ctx.fillStyle = line.isEmpty ? 'rgba(255,255,255,0.03)' : 'rgba(255,255,255,0.06)';
    ctx.fillRect(0, y, W, TL.BLOCK_H);
    ctx.fillStyle = 'rgba(255,255,255,0.28)';
    ctx.font = '9px monospace'; ctx.textAlign = 'left'; ctx.textBaseline = 'middle';
    ctx.fillText(String(row.li + 1).padStart(2, '0'), 4, y + TL.BLOCK_H / 2);

    if (line.isEmpty) return;

    if (state.mode === 'lines') {
      if (line.time === null) return;
      const x = line.time * pps - state.scrollX;
      const dur = line.duration || 2;
      const wPx = Math.max(dur * pps, 24);
      const currentGroup = Math.floor(state.currentLineIdx / N);
      if (row.li === state.currentLineIdx) ctx.fillStyle = '#4fc3ff';
      else if (row.band === currentGroup) ctx.fillStyle = '#34d17a';
      else ctx.fillStyle = '#1f8a55';
      roundRectFill(ctx, x, y, wPx, TL.BLOCK_H, 4);
      ctx.fillStyle = '#06210f'; ctx.font = 'bold 9px sans-serif'; ctx.textAlign = 'left';
      const maxChars = Math.max(1, Math.floor((wPx - 18) / 6));
      ctx.fillText(line.text.slice(0, maxChars), x + 14, y + TL.BLOCK_H / 2 + 0.5);
      drawHandle(ctx, x, y, TL.BLOCK_H, 'left');
      drawHandle(ctx, x + wPx, y, TL.BLOCK_H, 'right');
    } else {
      line.words.forEach((w, wi) => {
        if (w.time === null) return;
        const x = w.time * pps - state.scrollX;
        const dur = w.duration || 0.3;
        const wPx = Math.max(dur * pps, 16);
        ctx.fillStyle = (row.li === state.currentLineIdx && wi === state.currentWordIdx) ? '#4fc3ff' : '#34d17a';
        roundRectFill(ctx, x, y, wPx, TL.BLOCK_H, 3);
        ctx.fillStyle = '#06210f'; ctx.font = 'bold 8px sans-serif'; ctx.textAlign = 'left';
        const maxChars = Math.max(1, Math.floor((wPx - 14) / 5));
        ctx.fillText(w.text.slice(0, maxChars), x + 10, y + TL.BLOCK_H / 2 + 0.5);
        drawHandle(ctx, x, y, TL.BLOCK_H, 'left');
        drawHandle(ctx, x + wPx, y, TL.BLOCK_H, 'right');
      });
    }
  });

  // Regla numérica de tiempo (segundos y minutos)
  ctx.fillStyle = '#101018';
  ctx.fillRect(0, rulerY - 8, W, TL.FOOTER);
  ctx.strokeStyle = '#5a6a80'; ctx.lineWidth = 1;
  ctx.beginPath(); ctx.moveTo(0, rulerY); ctx.lineTo(W, rulerY); ctx.stroke();
  for (let t = Math.floor(startT * 2) / 2; t <= endT; t += 0.5) {
    const x = t * pps - state.scrollX;
    if (x < 0 || x > W) continue;
    const whole = Math.abs(t - Math.round(t)) < 0.01;
    const big = Math.round(t) % 5 === 0;
    ctx.strokeStyle = whole ? (big ? '#9fb3c8' : '#5a6a80') : '#3a4658';
    const h = big ? 16 : (whole ? 11 : 6);
    ctx.beginPath(); ctx.moveTo(x, rulerY); ctx.lineTo(x, rulerY + h); ctx.stroke();
    if (big) {
      const m = Math.floor(t / 60), s = Math.floor(t % 60);
      ctx.fillStyle = '#e8eef6'; ctx.font = 'bold 10px monospace'; ctx.textAlign = 'center'; ctx.textBaseline = 'alphabetic';
      ctx.fillText(`${m}:${String(s).padStart(2, '0')}`, x, rulerY + h + 12);
    }
  }

  // Cursor rojo con etiqueta numérica de tiempo
  if (state.audioDuration && isFinite(state.audioDuration)) {
    const px = audioPlayer.currentTime * pps - state.scrollX;
    if (px >= 0 && px <= W) {
      ctx.strokeStyle = '#ff3b7f'; ctx.lineWidth = 2;
      ctx.beginPath(); ctx.moveTo(px, TL.HEADER); ctx.lineTo(px, rulerY); ctx.stroke();
      ctx.fillStyle = '#ff3b7f';
      ctx.beginPath(); ctx.moveTo(px - 5, rulerY); ctx.lineTo(px + 5, rulerY); ctx.lineTo(px, rulerY + 8); ctx.closePath(); ctx.fill();
      const t = audioPlayer.currentTime;
      const txt = `${Math.floor(t / 60)}:${String(Math.floor(t % 60)).padStart(2, '0')}.${String(Math.floor((t % 1) * 100)).padStart(2, '0')}`;
      ctx.font = 'bold 10px monospace';
      const tw = ctx.measureText(txt).width + 10;
      const bx = Math.max(2, Math.min(W - tw - 2, px - tw / 2));
      ctx.fillStyle = '#ff3b7f'; ctx.fillRect(bx, 2, tw, 14);
      ctx.fillStyle = '#fff'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
      ctx.fillText(txt, bx + tw / 2, 9);
    }
  }
}

function autoScrollTimeline() {
  if (!state.audioDuration || !state.timelineW) return;
  if (audioPlayer.paused) return;
  const pps = 40 * state.zoom; const W = state.timelineW;
  const playheadX = audioPlayer.currentTime * pps - state.scrollX;
  const targetX = W * 0.3;
  if (playheadX > W * 0.7 || playheadX < W * 0.1) {
    state.scrollX = audioPlayer.currentTime * pps - targetX;
    if (state.scrollX < 0) state.scrollX = 0;
  }
}

function hitTestTimeline(x, y) {
  const L = timelineLayout();
  const pps = L.pps;
  for (const row of L.rows) {
    const line = state.lines[row.li];
    if (y < row.rowY - 2 || y > row.rowY + TL.BLOCK_H + 2) continue;
    if (line.isEmpty) return { kind: 'slot', li: row.li };
    if (state.mode === 'lines') {
      if (line.time === null) return { kind: 'slot', li: row.li };
      const bx = line.time * pps - state.scrollX;
      const bw = Math.max((line.duration || 2) * pps, 24);
      const endX = bx + bw;
      if (Math.abs(x - bx) <= TL.HANDLE_W) return { kind: 'handle', side: 'left', li: row.li, origTime: line.time, origDuration: line.duration || 2 };
      if (Math.abs(x - endX) <= TL.HANDLE_W) return { kind: 'handle', side: 'right', li: row.li, origTime: line.time, origDuration: line.duration || 2 };
      if (x >= bx && x <= endX) return { kind: 'block', li: row.li, origTime: line.time };
      return { kind: 'slot', li: row.li };
    }
    for (let wi = 0; wi < line.words.length; wi++) {
      const w = line.words[wi];
      if (w.time === null) continue;
      const bx = w.time * pps - state.scrollX;
      const bw = Math.max((w.duration || 0.3) * pps, 16);
      const endX = bx + bw;
      if (Math.abs(x - bx) <= TL.HANDLE_W) return { kind: 'handle', side: 'left', li: row.li, wi, origTime: w.time, origDuration: w.duration || 0.3 };
      if (Math.abs(x - endX) <= TL.HANDLE_W) return { kind: 'handle', side: 'right', li: row.li, wi, origTime: w.time, origDuration: w.duration || 0.3 };
      if (x >= bx && x <= endX) return { kind: 'block', li: row.li, wi, origTime: w.time };
    }
    return { kind: 'slot', li: row.li };
  }
  return null;
}

let tlTouch = null;
const tlWrap = document.getElementById('timelineWrap');
tlWrap.addEventListener('touchstart', handleTlStart, { passive: false });
tlWrap.addEventListener('touchmove', handleTlMove, { passive: false });
tlWrap.addEventListener('touchend', handleTlEnd);
tlWrap.addEventListener('mousedown', handleTlStart);
window.addEventListener('mousemove', handleTlMove);
window.addEventListener('mouseup', handleTlEnd);

function handleTlStart(e) {
  const touch = e.touches ? e.touches[0] : e;
  const rect = tlWrap.getBoundingClientRect();
  const x = touch.clientX - rect.left;
  const y = touch.clientY - rect.top + tlWrap.scrollTop;
  const hit = hitTestTimeline(x, y);

  if (hit) {
    if (hit.kind === 'handle') { tlTouch = { mode: 'handle', handle: hit, startX: touch.clientX }; }
    else if (hit.kind === 'block') { tlTouch = { mode: 'drag', startX: touch.clientX, block: hit, origTime: hit.origTime }; }
    else {
      state.currentLineIdx = hit.li; state.currentWordIdx = 0;
      const line = state.lines[hit.li];
      if (line && line.time !== null) audioPlayer.currentTime = line.time;
      renderLines(); renderWords();
      tlTouch = null;
      e.preventDefault();
      return;
    }
  } else if (e.touches && e.touches.length === 2) {
    const dx = e.touches[0].clientX - e.touches[1].clientX;
    const dy = e.touches[0].clientY - e.touches[1].clientY;
    tlTouch = { mode: 'zoom', startDist: Math.hypot(dx, dy), startZoom: state.zoom };
  } else {
    const L = timelineLayout();
    const tTime = (x + state.scrollX) / L.pps;
    if (state.audioDuration) audioPlayer.currentTime = Math.max(0, Math.min(state.audioDuration, tTime));
    tlTouch = { mode: 'scrub', startX: touch.clientX, startScroll: state.scrollX };
  }
  e.preventDefault();
}

function handleTlMove(e) {
  if (!tlTouch) return;
  const touch = e.touches ? e.touches[0] : e;
  const L = timelineLayout();
  const pps = L.pps;
  if (tlTouch.mode === 'handle') {
    const dx = touch.clientX - tlTouch.startX;
    const dt = dx / pps;
    const steps = Math.round(dt / 0.25);
    const sdt = steps * 0.25;
    const h = tlTouch.handle;
    if (h.side === 'left') {
      const newTime = Math.max(0, h.origTime + sdt);
      if (h.wi !== undefined) {
        const w = state.lines[h.li].words[h.wi];
        const end = w.time + (w.duration || 0.3);
        if (newTime < end - 0.05) { w.duration = end - newTime; w.time = newTime; }
      } else {
        const line = state.lines[h.li];
        const end = line.time + (line.duration || 2);
        if (newTime < end - 0.05) { line.duration = end - newTime; line.time = newTime; }
      }
    } else {
      const newEnd = h.origTime + h.origDuration + sdt;
      if (h.wi !== undefined) { const w = state.lines[h.li].words[h.wi]; w.duration = Math.max(0.08, newEnd - w.time); }
      else { const line = state.lines[h.li]; line.duration = Math.max(0.08, newEnd - line.time); }
    }
    drawTimeline();
  } else if (tlTouch.mode === 'drag') {
    const dx = touch.clientX - tlTouch.startX;
    const dt = dx / pps;
    const steps = Math.round(dt / 0.25);
    const sdt = steps * 0.25;
    if (tlTouch.block.wi !== undefined) {
      const w = state.lines[tlTouch.block.li].words[tlTouch.block.wi];
      w.time = Math.max(0, tlTouch.block.origTime + sdt);
    } else {
      state.lines[tlTouch.block.li].time = Math.max(0, tlTouch.block.origTime + sdt);
    }
    drawTimeline();
  } else if (tlTouch.mode === 'scrub') {
    const dx = touch.clientX - tlTouch.startX;
    state.scrollX = Math.max(0, tlTouch.startScroll - dx);
    drawTimeline();
  } else if (tlTouch.mode === 'zoom' && e.touches) {
    const dx = e.touches[0].clientX - e.touches[1].clientX;
    const dy = e.touches[0].clientY - e.touches[1].clientY;
    const dist = Math.hypot(dx, dy);
    state.zoom = Math.max(0.2, Math.min(10, tlTouch.startZoom * (dist / tlTouch.startDist)));
    drawTimeline();
  }
  e.preventDefault();
}

function handleTlEnd() { if (tlTouch && (tlTouch.mode === 'drag' || tlTouch.mode === 'handle')) saveProject(); tlTouch = null; }

function getSettings() {
  return {
    fontFamily: document.getElementById('fontFamily').value,
    fontSize: parseInt(document.getElementById('fontSize').value),
    posY: parseInt(document.getElementById('posY').value) / 100,
    visibleLines: parseInt(document.getElementById('visibleLines').value),
    textColor: document.getElementById('textColor').value,
    syncColor: document.getElementById('syncColor').value,
    bgType: document.getElementById('bgType').value,
    bgColor: document.getElementById('bgColor').value,
    bgColor2: document.getElementById('bgColor2').value,
    gradientAngle: parseInt(document.getElementById('gradientAngle').value),
    gradientPos1: parseInt(document.getElementById('gradientPos1').value),
    gradientPos2: parseInt(document.getElementById('gradientPos2').value),
    bgFit: document.getElementById('bgFit').value,
    fx: { color: document.getElementById('fxColor').checked, fill: document.getElementById('fxFill').checked, bounce: document.getElementById('fxBounce').checked, glow: document.getElementById('fxGlow').checked, underline: document.getElementById('fxUnderline').checked, shadow: document.getElementById('fxShadow').checked, typewriter: document.getElementById('fxTypewriter').checked }
  };
}

function drawOneLine(ctx, line, y, s, isSynced, isCurrent, t, W, H) {
  if (line.isEmpty) return;
  ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
  const text = line.text;
  ctx.fillStyle = isSynced ? s.syncColor : s.textColor;
  const progress = isCurrent ? Math.min(1, Math.max(0, (t - (line.time||0)) / (line.duration || 3))) : 0;
  if (isCurrent && s.fx.typewriter) {
    const chars = Math.floor(text.length * progress);
    const shown = text.slice(0, chars); const hidden = text.slice(chars);
    const totalW = ctx.measureText(text).width; const startX = W/2 - totalW/2;
    ctx.textAlign = 'left'; ctx.fillStyle = s.syncColor; ctx.fillText(shown, startX, y);
    ctx.fillStyle = s.textColor; const shownW = ctx.measureText(shown).width; ctx.fillText(hidden, startX + shownW, y);
    ctx.textAlign = 'center';
  } else if (isCurrent && s.fx.fill) {
    const fullW = ctx.measureText(text).width; const fillW = fullW * progress;
    ctx.fillStyle = s.textColor; ctx.fillText(text, W/2, y);
    ctx.save(); ctx.beginPath(); ctx.rect(W/2 - fullW/2, y - s.fontSize, fillW, s.fontSize * 2); ctx.clip();
    ctx.fillStyle = s.syncColor; ctx.fillText(text, W/2, y); ctx.restore();
  } else {
    ctx.fillText(text, W/2, y);
  }
}

// ============ PREVIEW: SIEMPRE exactamente N renglones (filas vacías con lugar) ============
function drawPreview() {
  const s = getSettings();
  const ctx = previewCtx;
  const W = previewCanvas.width;
  const H = previewCanvas.height;

  ctx.clearRect(0, 0, W, H);

  if (s.bgType === 'transparent') {
    const checkSize = 20;
    for (let y = 0; y < H; y += checkSize) {
      for (let x = 0; x < W; x += checkSize) {
        ctx.fillStyle = ((Math.floor(x/checkSize) + Math.floor(y/checkSize)) % 2 === 0) ? '#2a2a2a' : '#3a3a3a';
        ctx.fillRect(x, y, checkSize, checkSize);
      }
    }
  } else if (s.bgType === 'color') {
    ctx.fillStyle = s.bgColor; ctx.fillRect(0, 0, W, H);
  } else if (s.bgType === 'gradient') {
    const angle = s.gradientAngle * Math.PI / 180;
    const cx = W/2, cy = H/2; const len = Math.max(W, H);
    const x1 = cx - Math.cos(angle) * len/2; const y1 = cy - Math.sin(angle) * len/2;
    const x2 = cx + Math.cos(angle) * len/2; const y2 = cy + Math.sin(angle) * len/2;
    const g = ctx.createLinearGradient(x1, y1, x2, y2);
    g.addColorStop(s.gradientPos1/100, s.bgColor); g.addColorStop(s.gradientPos2/100, s.bgColor2);
    ctx.fillStyle = g; ctx.fillRect(0, 0, W, H);
  } else if (s.bgType === 'image' && state.bgImage) {
    const img = state.bgImage; let dw, dh, dx, dy;
    const ar = W / H; const iar = img.width / img.height;
    if (s.bgFit === 'cover') { if (iar > ar) { dh = H; dw = H * iar; } else { dw = W; dh = W / iar; } }
    else if (s.bgFit === 'contain') { if (iar > ar) { dw = W; dh = W / iar; } else { dh = H; dw = H * iar; } }
    else { dw = W; dh = H; }
    dx = (W - dw) / 2; dy = (H - dh) / 2;
    ctx.drawImage(img, dx, dy, dw, dh);
  } else {
    ctx.fillStyle = '#000'; ctx.fillRect(0, 0, W, H);
  }

  if (state.lines.length === 0) {
    ctx.fillStyle = '#666'; ctx.font = 'bold 30px Arial';
    ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
    ctx.fillText('Agrega letra y sincroniza con TAP', W/2, H/2);
    return;
  }

  const t = audioPlayer.currentTime;

  let currentLineIdx = -1;
  for (let i = 0; i < state.lines.length; i++) {
    const line = state.lines[i];
    if (line.isEmpty || line.time === null) continue;
    const end = line.time + (line.duration || 3);
    if (t >= line.time && t < end) { currentLineIdx = i; break; }
  }
  if (currentLineIdx === -1) {
    let closestIdx = -1; let closestDist = Infinity;
    for (let i = 0; i < state.lines.length; i++) {
      const line = state.lines[i];
      if (line.isEmpty || line.time === null) continue;
      const dist = Math.abs(t - line.time);
      if (dist < closestDist) { closestDist = dist; closestIdx = i; }
    }
    currentLineIdx = closestIdx;
  }
  if (currentLineIdx === -1) currentLineIdx = 0;

  const N = s.visibleLines; // número fijo de renglones en pantalla: nunca 2,3,5,6 si es 4
  let fontSize = s.fontSize;
  let lineHeight = fontSize * 1.4;
  if (lineHeight * N > H * 0.92) fontSize = Math.max(16, Math.floor((H * 0.92) / (N * 1.4)));
  lineHeight = fontSize * 1.4;
  const blockIndex = Math.floor(currentLineIdx / N);
  const startLineIdx = blockIndex * N;
  ctx.font = `bold ${fontSize}px ${s.fontFamily}`;
  const startY = H * s.posY - (lineHeight * N) / 2 + lineHeight / 2;

  for (let k = 0; k < N; k++) {
    const y = startY + k * lineHeight;
    const li = startLineIdx + k;
    const line = (li < state.lines.length) ? state.lines[li] : null;
    const valid = line && !line.isEmpty && line.text && line.text.trim().length > 0;

    if (!valid) {
      // Fila vacía que respeta el lugar exacto, atenuada, como si hubiera texto
      ctx.save();
      ctx.globalAlpha = 0.18;
      ctx.strokeStyle = s.textColor;
      ctx.setLineDash([6, 6]); ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.moveTo(W * 0.18, y); ctx.lineTo(W * 0.82, y);
      ctx.stroke(); ctx.setLineDash([]);
      ctx.globalAlpha = 0.30;
      ctx.fillStyle = s.textColor;
      ctx.font = `${Math.max(10, Math.floor(fontSize * 0.35))}px monospace`;
      ctx.textAlign = 'left'; ctx.textBaseline = 'middle';
      ctx.fillText(li != null && li < state.lines.length ? `${li + 1}` : '·', W * 0.18 - 26, y);
      ctx.restore();
      continue;
    }

    const lineEnd = line.time !== null ? line.time + (line.duration || 3) : 0;
    let isSynced = false, isCurrent = false;
    if (line.time !== null) {
      if (t >= line.time && t < lineEnd) { isCurrent = true; isSynced = true; }
      else if (t >= lineEnd) { isSynced = true; }
    }
    drawOneLine(ctx, line, y, s, isSynced, isCurrent, t, W, H);
  }
}

function loop() {
  try { updateTimer(); autoScrollTimeline(); drawPreview(); drawTimeline(); }
  catch (e) { console.warn('Loop:', e); }
  requestAnimationFrame(loop);
}

['fontFamily','fontSize','posY','visibleLines','textColor','syncColor','bgType','bgColor','bgColor2','gradientAngle','gradientPos1','gradientPos2','bgFit','fxColor','fxFill','fxBounce','fxGlow','fxUnderline','fxShadow','fxTypewriter'].forEach(id => {
  const el = document.getElementById(id);
  if (!el) return;
  el.addEventListener('input', () => {
    if (id === 'fontSize') document.getElementById('fontSizeVal').textContent = el.value;
    if (id === 'posY') document.getElementById('posYVal').textContent = el.value;
    if (id === 'gradientAngle') document.getElementById('gradAngleVal').textContent = el.value;
    if (id === 'gradientPos1') document.getElementById('gradPos1Val').textContent = el.value;
    if (id === 'gradientPos2') document.getElementById('gradPos2Val').textContent = el.value;
    if (id === 'bgType') {
      document.getElementById('bgColorWrap').style.display = (el.value === 'color' || el.value === 'gradient') ? '' : 'none';
      document.getElementById('bgColor2Wrap').style.display = el.value === 'gradient' ? '' : 'none';
      document.getElementById('gradientControls').style.display = el.value === 'gradient' ? '' : 'none';
      document.getElementById('bgImageWrap').style.display = el.value === 'image' ? '' : 'none';
    }
    if (id === 'visibleLines') drawTimeline();
    saveProject();
  });
});

document.getElementById('bgImage').addEventListener('change', (e) => {
  const file = e.target.files[0]; if (!file) return;
  const img = new Image(); img.onload = () => { state.bgImage = img; };
  img.src = URL.createObjectURL(file);
  const reader = new FileReader(); reader.onload = () => { state.bgImageDataUrl = reader.result; saveProject(); };
  reader.readAsDataURL(file);
});

let spaceTimer = null;
document.addEventListener('keydown', (e) => {
  if (e.code === 'Space' && document.activeElement.tagName !== 'TEXTAREA' && document.activeElement.tagName !== 'INPUT') {
    e.preventDefault();
    if (spaceTimer) { clearTimeout(spaceTimer); spaceTimer = null; togglePlayPause(); }
    else { spaceTimer = setTimeout(() => { spaceTimer = null; tapSync(); }, 200); }
  }
  if (e.key === 'ArrowDown') window.scrollBy({ top: 200, behavior: 'smooth' });
  if (e.key === 'ArrowUp') window.scrollBy({ top: -200, behavior: 'smooth' });
});

async function exportVideo() {
  const status = document.getElementById('exportStatus');
  status.textContent = '⏳ Preparando...';
  if (!audioPlayer.src) { alert('Carga un audio primero'); return; }
  if (state.lines.length === 0) { alert('Procesa la letra primero'); return; }
  audioPlayer.currentTime = 0;
  try {
    await new Promise(resolve => {
      if (audioPlayer.readyState >= 2) resolve();
      else audioPlayer.addEventListener('canplay', resolve, { once: true });
    });
    const canvasStream = previewCanvas.captureStream(30);
    const AudioContextClass = window.AudioContext || window.webkitAudioContext;
    const audioCtx = new AudioContextClass();
    if (audioCtx.state === 'suspended') await audioCtx.resume();
    const source = audioCtx.createMediaElementSource(audioPlayer);
    const dest = audioCtx.createMediaStreamDestination();
    source.connect(dest); source.connect(audioCtx.destination);
    const combinedStream = new MediaStream([...canvasStream.getVideoTracks(), ...dest.stream.getAudioTracks()]);
    const mimeCandidates = ['video/webm;codecs=vp9,opus', 'video/webm;codecs=vp8,opus', 'video/webm'];
    let mime = '';
    for (const m of mimeCandidates) { if (MediaRecorder.isTypeSupported(m)) { mime = m; break; } }
    if (!mime) { alert('Tu navegador no soporta grabación de video'); return; }
    const recorder = new MediaRecorder(combinedStream, { mimeType: mime, videoBitsPerSecond: 4000000, audioBitsPerSecond: 128000 });
    const chunks = [];
    recorder.ondataavailable = e => { if (e.data.size > 0) chunks.push(e.data); };
    recorder.onstop = () => {
      const blob = new Blob(chunks, { type: mime });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href = url;
      const ext = mime.includes('mp4') ? 'mp4' : 'webm';
      const fname = `${folioText(getFolio())} - ${safeFileName(state.projectName)}.${ext}`;
      a.download = fname;
      document.body.appendChild(a); a.click(); document.body.removeChild(a);
      status.textContent = '✅ Video exportado: ' + fname;
      audioCtx.close();
    };
    recorder.onerror = (err) => { status.textContent = '❌ Error: ' + err.error; audioCtx.close(); };
    recorder.start(100);
    await audioPlayer.play();
    status.textContent = '🔴 Grabando... No pauses la reproducción';
    audioPlayer.onended = () => { setTimeout(() => recorder.stop(), 500); };
  } catch (err) {
    status.textContent = '❌ Error: ' + err.message;
    console.error(err);
  }
}

// ============ GUARDADO: FOLIO + NOMBRE, CARPETA MANUAL / DESCARGA / EXPORTACIÓN FINAL ============
function buildProjectData(isFinal) {
  const folio = getFolio();
  return {
    app: 'trova-karaokes', version: 2,
    meta: { folio, folioText: folioText(folio), name: state.projectName || '', final: !!isFinal, savedAt: new Date().toISOString() },
    lines: state.lines,
    settings: getSettings(),
    lyrics: document.getElementById('lyricsInput').value,
    mode: state.mode,
    bgImageDataUrl: state.bgImageDataUrl
  };
}

function downloadBlob(blob, fname) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = fname;
  document.body.appendChild(a); a.click();
  setTimeout(() => { URL.revokeObjectURL(url); a.remove(); }, 400);
}

function projectFileName(data) {
  return `${data.meta.folioText} - ${safeFileName(data.meta.name)}.json`;
}

async function saveToFolder(isFinal) {
  if (state.lines.length === 0) { setStatus('⚠️ Primero procesa la letra'); return; }
  if (!isFinal) nextFolio(); // las pruebas consumen folio; el final conserva el actual
  const data = buildProjectData(isFinal);
  const fname = projectFileName(data);
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });

  if (window.showDirectoryPicker) {
    try {
      if (!dirHandle) dirHandle = await window.showDirectoryPicker({ mode: 'readwrite' });
      const fileHandle = await dirHandle.getFileHandle(fname, { create: true });
      const writable = await fileHandle.createWritable();
      await writable.write(blob);
      await writable.close();
      setStatus(`📁 Guardado en carpeta: ${fname}${isFinal ? ' · EXPORTACIÓN FINAL' : ''}`);
      refreshFolioBadge(); saveProject();
      return;
    } catch (err) {
      if (err && err.name === 'AbortError') { setStatus('Selección de carpeta cancelada'); return; }
      setStatus('No se pudo escribir en la carpeta; se descargará el archivo.');
    }
  }
  downloadBlob(blob, fname);
  setStatus(`⬇️ Descargado: ${fname}${isFinal ? ' · FINAL' : ''}`);
  refreshFolioBadge(); saveProject();
}

function downloadProject() {
  if (state.lines.length === 0) { setStatus('⚠️ Primero procesa la letra'); return; }
  nextFolio();
  const data = buildProjectData(false);
  const fname = projectFileName(data);
  downloadBlob(new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' }), fname);
  setStatus(`⬇️ Descargado: ${fname}`);
  refreshFolioBadge(); saveProject();
}

function exportFinal() {
  if (state.lines.length === 0) { setStatus('⚠️ Primero procesa la letra'); return; }
  const nombre = state.projectName || 'karaoke';
  if (!confirm(`Exportar karaoke FINAL con folio ${folioText(getFolio())} y nombre "${nombre}"?`)) return;
  saveToFolder(true);
}

document.getElementById('projectFileInput').addEventListener('change', (e) => {
  const file = e.target.files[0]; if (!file) return;
  const reader = new FileReader();
  reader.onload = (ev) => {
    try {
      const data = JSON.parse(ev.target.result);
      applyProjectData(data);
      setStatus(`📂 Proyecto abierto: ${file.name}`);
    } catch (err) { setStatus('❌ Error al leer el proyecto: ' + err.message); }
  };
  reader.readAsText(file, 'UTF-8');
  e.target.value = '';
});

function applySettings(s) {
  if (!s) return;
  Object.entries(s).forEach(([k, v]) => {
    if (k === 'fx') {
      Object.entries(v).forEach(([fk, fv]) => {
        const el = document.getElementById('fx' + fk.charAt(0).toUpperCase() + fk.slice(1));
        if (el) el.checked = fv;
      });
    } else {
      const el = document.getElementById(k);
      if (el) el.value = v;
    }
  });
  const map = { fontSize: 'fontSizeVal', posY: 'posYVal', gradientAngle: 'gradAngleVal', gradientPos1: 'gradPos1Val', gradientPos2: 'gradPos2Val' };
  Object.entries(map).forEach(([id, label]) => {
    const el = document.getElementById(id);
    if (el) document.getElementById(label).textContent = el.value;
  });
  const bgType = document.getElementById('bgType');
  document.getElementById('bgColorWrap').style.display = (bgType.value === 'color' || bgType.value === 'gradient') ? '' : 'none';
  document.getElementById('bgColor2Wrap').style.display = bgType.value === 'gradient' ? '' : 'none';
  document.getElementById('gradientControls').style.display = bgType.value === 'gradient' ? '' : 'none';
  document.getElementById('bgImageWrap').style.display = bgType.value === 'image' ? '' : 'none';
}

function applyProjectData(data) {
  if (!data) return;
  if (data.meta && data.meta.folio) setFolio(data.meta.folio);
  if (data.meta && typeof data.meta.name === 'string') {
    state.projectName = data.meta.name;
    document.getElementById('projectName').value = data.meta.name;
  }
  if (data.lyrics) document.getElementById('lyricsInput').value = data.lyrics;
  if (data.lines) {
    state.lines = data.lines;
    state.currentLineIdx = -1; state.currentWordIdx = -1;
    renderLines(); renderWords();
  }
  applySettings(data.settings);
  if (data.bgImageDataUrl) {
    state.bgImageDataUrl = data.bgImageDataUrl;
    const img = new Image(); img.onload = () => { state.bgImage = img; };
    img.src = data.bgImageDataUrl;
  }
  if (data.mode) setMode(data.mode);
  refreshFolioBadge();
  drawTimeline();
}

function saveProject() {
  const data = buildProjectData(false);
  try { localStorage.setItem('karaokeProject', JSON.stringify(data)); } catch (e) {}
}

function loadProject() {
  const raw = localStorage.getItem('karaokeProject');
  if (!raw) { setStatus('No hay respaldo guardado en este navegador'); return; }
  try {
    applyProjectData(JSON.parse(raw));
    setStatus('📂 Respaldo del navegador cargado');
  } catch (e) { setStatus('❌ Respaldo dañado'); }
}

function resetProject() {
  if (!confirm('¿Reiniciar todo? Se borrará el proyecto y el folio del navegador.')) return;
  localStorage.removeItem('karaokeProject');
  localStorage.removeItem(FOLIO_KEY);
  location.reload();
}
function setStatus(msg) { document.getElementById('status').textContent = msg; }

window.addEventListener('load', () => {
  refreshFolioBadge();
  setupTimeline();
  loadProject();
  loop();
});
window.addEventListener('resize', () => { setupTimeline(); drawTimeline(); });
</script>

</body>
</html>
