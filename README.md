<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>拼贴动图制作工具</title>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,600;1,400&family=Noto+Serif+SC:wght@300;400;600&family=Caveat:wght@400;600&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #4a4438;
    --panel-bg: #2e2a24;
    --accent: #c8a96e;
    --text: #f0ebe3;
    --muted: #8a8070;
    --border: #5a5248;
    --canvas-bg: #4a4438;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--panel-bg);
    color: var(--text);
    font-family: 'Noto Serif SC', serif;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  header {
    padding: 16px 28px;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    gap: 16px;
    background: var(--panel-bg);
  }

  header h1 {
    font-family: 'Playfair Display', serif;
    font-size: 20px;
    font-weight: 400;
    color: var(--accent);
    letter-spacing: 0.05em;
  }

  header span {
    font-size: 12px;
    color: var(--muted);
    font-family: 'Caveat', cursive;
    letter-spacing: 0.08em;
  }

  .workspace {
    display: flex;
    flex: 1;
    overflow: hidden;
  }

  /* ── LEFT PANEL ── */
  .left-panel {
    width: 280px;
    flex-shrink: 0;
    border-right: 1px solid var(--border);
    overflow-y: auto;
    padding: 20px 16px;
    display: flex;
    flex-direction: column;
    gap: 20px;
    background: #252019;
  }

  .section-title {
    font-size: 11px;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    color: var(--muted);
    margin-bottom: 10px;
    font-family: 'Playfair Display', serif;
  }

  /* Upload zone */
  .upload-zone {
    border: 1.5px dashed var(--border);
    border-radius: 8px;
    padding: 18px 12px;
    text-align: center;
    cursor: pointer;
    transition: border-color 0.2s, background 0.2s;
  }
  .upload-zone:hover { border-color: var(--accent); background: rgba(200,169,110,0.06); }
  .upload-zone .icon { font-size: 24px; margin-bottom: 6px; }
  .upload-zone p { font-size: 12px; color: var(--muted); line-height: 1.5; }
  .upload-zone input { display: none; }

  /* Thumbnail list */
  .thumb-list {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
  }
  .thumb-item {
    position: relative;
    border-radius: 6px;
    overflow: hidden;
    aspect-ratio: 1;
    border: 1.5px solid transparent;
    cursor: pointer;
    transition: border-color 0.2s;
  }
  .thumb-item:hover { border-color: var(--accent); }
  .thumb-item img { width: 100%; height: 100%; object-fit: cover; display: block; }
  .thumb-item .remove-btn {
    position: absolute;
    top: 3px; right: 3px;
    width: 18px; height: 18px;
    background: rgba(0,0,0,0.7);
    border: none; color: #fff;
    font-size: 11px; border-radius: 50%;
    cursor: pointer; display: flex;
    align-items: center; justify-content: center;
    opacity: 0; transition: opacity 0.2s;
  }
  .thumb-item:hover .remove-btn { opacity: 1; }
  .thumb-item .tag-input {
    position: absolute;
    bottom: 0; left: 0; right: 0;
    background: rgba(0,0,0,0.65);
    border: none;
    color: #fff;
    font-size: 10px;
    padding: 4px 6px;
    font-family: 'Caveat', cursive;
    width: 100%;
    outline: none;
  }
  .thumb-item .tag-input::placeholder { color: rgba(255,255,255,0.45); }

  /* Color palette */
  .palette-row {
    display: flex;
    gap: 6px;
    flex-wrap: wrap;
    margin-bottom: 8px;
  }
  .color-swatch {
    width: 28px; height: 28px;
    border-radius: 50%;
    cursor: pointer;
    border: 2px solid transparent;
    transition: transform 0.15s, border-color 0.15s;
    flex-shrink: 0;
  }
  .color-swatch:hover { transform: scale(1.15); }
  .color-swatch.active { border-color: white; }

  /* Controls */
  .ctrl-row {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 8px;
  }
  .ctrl-row label { font-size: 11px; color: var(--muted); flex-shrink: 0; width: 70px; }
  .ctrl-row input[type=color] {
    width: 32px; height: 26px;
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 1px;
    background: none;
    cursor: pointer;
  }
  .ctrl-row input[type=range] {
    flex: 1;
    accent-color: var(--accent);
  }
  .ctrl-row select {
    flex: 1;
    background: var(--panel-bg);
    border: 1px solid var(--border);
    color: var(--text);
    padding: 5px 8px;
    border-radius: 4px;
    font-size: 12px;
    outline: none;
  }

  /* Text items */
  .text-item-row {
    display: flex;
    gap: 6px;
    align-items: center;
    margin-bottom: 6px;
  }
  .text-item-row input[type=text] {
    flex: 1;
    background: var(--panel-bg);
    border: 1px solid var(--border);
    color: var(--text);
    padding: 6px 9px;
    border-radius: 4px;
    font-size: 12px;
    font-family: 'Caveat', cursive;
    outline: none;
  }
  .text-item-row input[type=text]:focus { border-color: var(--accent); }
  .text-item-row .del-btn {
    background: none; border: none;
    color: var(--muted); cursor: pointer; font-size: 14px;
    padding: 0 4px;
  }
  .text-item-row .del-btn:hover { color: #e07070; }

  .btn {
    display: inline-flex; align-items: center; justify-content: center;
    gap: 6px;
    padding: 8px 14px;
    border-radius: 6px;
    font-size: 12px;
    cursor: pointer;
    transition: background 0.2s, transform 0.1s;
    border: none;
    font-family: 'Noto Serif SC', serif;
  }
  .btn:active { transform: scale(0.97); }
  .btn-primary { background: var(--accent); color: #1a1610; font-weight: 600; }
  .btn-primary:hover { background: #d9be85; }
  .btn-ghost { background: transparent; border: 1px solid var(--border); color: var(--text); }
  .btn-ghost:hover { border-color: var(--accent); color: var(--accent); }
  .btn-full { width: 100%; }

  /* ── CENTER CANVAS ── */
  .canvas-area {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #1c1814;
    overflow: hidden;
    position: relative;
  }

  #canvas-wrap {
    position: relative;
    overflow: hidden;
    border-radius: 4px;
    box-shadow: 0 8px 48px rgba(0,0,0,0.6);
    background: var(--canvas-bg);
    transition: background 0.4s;
  }

  .collage-item {
    position: absolute;
    cursor: grab;
    user-select: none;
    transition: box-shadow 0.2s;
  }
  .collage-item:active { cursor: grabbing; }
  .collage-item.selected { outline: 2px solid var(--accent); outline-offset: 2px; }
  .collage-item img {
    display: block;
    width: 100%; height: 100%;
    object-fit: cover;
    pointer-events: none;
  }

  .collage-text {
    position: absolute;
    cursor: grab;
    user-select: none;
    font-family: 'Caveat', cursive;
    font-size: 18px;
    color: #f0ebe3;
    white-space: nowrap;
    text-shadow: 1px 1px 3px rgba(0,0,0,0.3);
    padding: 2px 4px;
    border-radius: 3px;
    transition: outline 0.15s;
  }
  .collage-text.selected { outline: 1.5px dashed var(--accent); outline-offset: 3px; }
  .collage-text:active { cursor: grabbing; }

  /* Decorative dots */
  .deco-dot {
    position: absolute;
    pointer-events: none;
    opacity: 0.55;
  }

  /* corner brackets */
  .corner-bracket {
    position: absolute;
    pointer-events: none;
    stroke: rgba(240,235,227,0.35);
    stroke-width: 2;
    fill: none;
  }

  /* ── RIGHT PANEL ── */
  .right-panel {
    width: 220px;
    flex-shrink: 0;
    border-left: 1px solid var(--border);
    overflow-y: auto;
    padding: 20px 14px;
    background: #252019;
    display: flex;
    flex-direction: column;
    gap: 18px;
  }

  /* Animation controls */
  .anim-btn-row {
    display: flex; gap: 8px;
  }

  .timeline-bar {
    height: 4px;
    background: var(--border);
    border-radius: 2px;
    margin-top: 8px;
    overflow: hidden;
  }
  .timeline-fill {
    height: 100%;
    background: var(--accent);
    border-radius: 2px;
    width: 0%;
    transition: width 0.1s linear;
  }

  /* resize handle */
  .resize-handle {
    position: absolute;
    bottom: -4px; right: -4px;
    width: 12px; height: 12px;
    background: var(--accent);
    border-radius: 2px;
    cursor: se-resize;
    opacity: 0;
    transition: opacity 0.15s;
  }
  .collage-item.selected .resize-handle { opacity: 1; }

  /* Anim overlay */
  .anim-overlay {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }

  @keyframes fadeSlideIn {
    from { opacity: 0; transform: translateY(12px); }
    to { opacity: 1; transform: translateY(0); }
  }
  @keyframes fadeIn {
    from { opacity: 0; } to { opacity: 1; }
  }

  .playing .collage-item, .playing .collage-text {
    opacity: 0;
  }

  /* scrollbar */
  ::-webkit-scrollbar { width: 4px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

  .divider { height: 1px; background: var(--border); margin: 2px 0; }

  .auto-bg-row {
    display: flex; gap: 6px; flex-wrap: wrap; margin-top: 6px;
  }
  .auto-swatch {
    width: 26px; height: 26px;
    border-radius: 4px;
    cursor: pointer;
    border: 2px solid transparent;
    transition: transform 0.15s, border-color 0.15s;
  }
  .auto-swatch:hover { transform: scale(1.1); border-color: white; }
  .auto-swatch.active { border-color: var(--accent); }

  .tag-badge {
    display: inline-block;
    font-family: 'Caveat', cursive;
    font-size: 11px;
    color: var(--muted);
    padding: 2px 6px;
    background: rgba(255,255,255,0.05);
    border-radius: 4px;
    margin: 2px;
  }
</style>
</head>
<body>

<header>
  <h1>moments · collage</h1>
  <span>拼贴动图制作工具</span>
</header>

<div class="workspace">

  <!-- LEFT: Upload & Settings -->
  <div class="left-panel">

    <div>
      <div class="section-title">上传图片</div>
      <div class="upload-zone" id="uploadZone">
        <div class="icon">🖼</div>
        <p>点击上传图片<br><span style="font-size:10px;color:var(--muted)">支持多张，可拖拽排序</span></p>
        <input type="file" id="fileInput" multiple accept="image/*">
      </div>
      <div class="thumb-list" id="thumbList" style="margin-top:10px"></div>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">背景颜色</div>
      <div style="font-size:10px;color:var(--muted);margin-bottom:6px;">自动从图片提取 ↓</div>
      <div class="auto-bg-row" id="autoBgRow">
        <div style="font-size:11px;color:var(--muted);font-style:italic;">上传图片后自动生成</div>
      </div>
      <div class="ctrl-row" style="margin-top:10px;">
        <label>自定义</label>
        <input type="color" id="bgColorPicker" value="#4a4438">
        <input type="range" id="bgBrightness" min="30" max="110" value="80" title="亮度">
      </div>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">文字样式</div>
      <div class="ctrl-row">
        <label>颜色</label>
        <input type="color" id="textColorPicker" value="#f0ebe3">
      </div>
      <div class="ctrl-row">
        <label>字体</label>
        <select id="fontSelect">
          <option value="'Caveat', cursive">手写体 Caveat</option>
          <option value="'Playfair Display', serif">衬线 Playfair</option>
          <option value="'Noto Serif SC', serif">宋体风格</option>
          <option value="monospace">等宽 Mono</option>
        </select>
      </div>
      <div class="ctrl-row">
        <label>大小</label>
        <input type="range" id="fontSizeSlider" min="10" max="48" value="18">
        <span id="fontSizeVal" style="font-size:11px;color:var(--muted);width:24px;text-align:right">18</span>
      </div>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">文案标签</div>
      <div id="textList"></div>
      <button class="btn btn-ghost btn-full" onclick="addTextItem()" style="margin-top:6px;font-size:11px;">
        + 添加文字
      </button>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">装饰元素</div>
      <div class="ctrl-row">
        <label>角标框</label>
        <input type="range" id="cornerOpacity" min="0" max="100" value="40">
      </div>
      <div class="ctrl-row">
        <label>散点装饰</label>
        <input type="range" id="dotsOpacity" min="0" max="100" value="50">
      </div>
    </div>

  </div>

  <!-- CENTER: Canvas -->
  <div class="canvas-area">
    <div id="canvas-wrap" style="width:420px;height:620px;">
      <svg class="corner-bracket" id="cornerSvg" style="position:absolute;inset:0;width:100%;height:100%;z-index:10;pointer-events:none">
        <path id="tlBracket" d="M 30 8 L 8 8 L 8 30" />
        <path id="trBracket" d="M 390 8 L 412 8 L 412 30" />
        <path id="blBracket" d="M 30 612 L 8 612 L 8 590" />
        <path id="brBracket" d="M 390 612 L 412 612 L 412 590" />
      </svg>
      <canvas id="dotsCanvas" style="position:absolute;inset:0;width:100%;height:100%;pointer-events:none;z-index:1;opacity:0.5"></canvas>
      <!-- collage items injected here -->
      <div id="animOverlay" class="anim-overlay" style="z-index:50"></div>
    </div>
  </div>

  <!-- RIGHT: Animation & Export -->
  <div class="right-panel">

    <div>
      <div class="section-title">画布尺寸</div>
      <div class="ctrl-row">
        <label>宽</label>
        <input type="range" id="canvasW" min="300" max="600" value="420">
        <span id="canvasWVal" style="font-size:10px;color:var(--muted);width:30px">420</span>
      </div>
      <div class="ctrl-row">
        <label>高</label>
        <input type="range" id="canvasH" min="400" max="900" value="620">
        <span id="canvasHVal" style="font-size:10px;color:var(--muted);width:30px">620</span>
      </div>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">图片间距</div>
      <div class="ctrl-row">
        <label>大小</label>
        <input type="range" id="imgSizeSlider" min="60" max="220" value="130">
      </div>
      <div class="ctrl-row">
        <label>旋转随机</label>
        <input type="range" id="rotateRange" min="0" max="15" value="4">
      </div>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">动画预览</div>
      <div class="anim-btn-row">
        <button class="btn btn-primary" onclick="playAnimation()" style="flex:1">▶ 播放</button>
        <button class="btn btn-ghost" onclick="resetAnimation()" style="flex:1">↺ 重置</button>
      </div>
      <div class="ctrl-row" style="margin-top:10px;">
        <label>速度</label>
        <input type="range" id="animSpeed" min="200" max="1500" value="600">
      </div>
      <div class="timeline-bar"><div class="timeline-fill" id="timelineFill"></div></div>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">动画样式</div>
      <select id="animStyle" style="width:100%;background:var(--panel-bg);border:1px solid var(--border);color:var(--text);padding:6px 8px;border-radius:4px;font-size:12px;outline:none;margin-bottom:8px;">
        <option value="fadeSlide">淡入上滑</option>
        <option value="pop">弹出缩放</option>
        <option value="drift">漂移淡入</option>
        <option value="rotate">旋转淡入</option>
      </select>
      <div class="ctrl-row">
        <label>间隔(ms)</label>
        <input type="range" id="animDelay" min="100" max="800" value="350">
        <span id="animDelayVal" style="font-size:10px;color:var(--muted);width:32px">350</span>
      </div>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">重新布局</div>
      <button class="btn btn-ghost btn-full" onclick="randomLayout()" style="font-size:11px;">🎲 随机排列</button>
      <button class="btn btn-ghost btn-full" onclick="gridLayout()" style="margin-top:6px;font-size:11px;">⊞ 网格排列</button>
    </div>

    <div class="divider"></div>

    <div>
      <div class="section-title">导出</div>
      <button class="btn btn-primary btn-full" onclick="exportPNG()" style="margin-bottom:6px">📷 导出当前帧</button>
      <button class="btn btn-ghost btn-full" onclick="exportGIF()" style="font-size:11px;">🎞 导出动图 GIF</button>
      <div style="font-size:10px;color:var(--muted);margin-top:6px;line-height:1.5;">GIF导出需稍等片刻，帧数越多体积越大</div>
    </div>

  </div>
</div>

<!-- GIF.js from CDN -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.js"></script>
<!-- html2canvas -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>

<script>
// ─────────────────────────────────────────
// STATE
// ─────────────────────────────────────────
const state = {
  images: [],      // {id, src, x, y, w, h, rotation, tag, zIndex}
  texts: [],       // {id, text, x, y, color, font, fontSize}
  bgColor: '#4a4438',
  textColor: '#f0ebe3',
  font: "'Caveat', cursive",
  fontSize: 18,
  canvasW: 420,
  canvasH: 620,
  selectedId: null,
  isPlaying: false,
  animTimeout: null,
};

let dragState = null;
let resizeState = null;
let idCounter = 0;
const newId = () => `item_${++idCounter}`;

// ─────────────────────────────────────────
// DOM refs
// ─────────────────────────────────────────
const canvasWrap = document.getElementById('canvas-wrap');
const thumbList = document.getElementById('thumbList');
const textListEl = document.getElementById('textList');
const bgPicker = document.getElementById('bgColorPicker');
const textColorPicker = document.getElementById('textColorPicker');
const fontSelect = document.getElementById('fontSelect');
const fontSizeSlider = document.getElementById('fontSizeSlider');
const fontSizeVal = document.getElementById('fontSizeVal');
const canvasWSlider = document.getElementById('canvasW');
const canvasHSlider = document.getElementById('canvasH');
const bgBrightness = document.getElementById('bgBrightness');
const cornerOpacitySlider = document.getElementById('cornerOpacity');
const dotsOpacitySlider = document.getElementById('dotsOpacity');
const animDelaySlider = document.getElementById('animDelay');
const animDelayVal = document.getElementById('animDelayVal');

// ─────────────────────────────────────────
// UPLOAD
// ─────────────────────────────────────────
document.getElementById('uploadZone').onclick = () => document.getElementById('fileInput').click();
document.getElementById('fileInput').onchange = e => loadFiles(e.target.files);

document.getElementById('uploadZone').ondragover = e => { e.preventDefault(); e.currentTarget.style.borderColor='var(--accent)'; };
document.getElementById('uploadZone').ondragleave = e => { e.currentTarget.style.borderColor='var(--border)'; };
document.getElementById('uploadZone').ondrop = e => {
  e.preventDefault();
  e.currentTarget.style.borderColor='var(--border)';
  loadFiles(e.dataTransfer.files);
};

function loadFiles(files) {
  Array.from(files).forEach(file => {
    if (!file.type.startsWith('image/')) return;
    const reader = new FileReader();
    reader.onload = ev => {
      const src = ev.target.result;
      const id = newId();
      const w = 120 + Math.random() * 80;
      const h = w * (0.7 + Math.random() * 0.6);
      const x = 20 + Math.random() * (state.canvasW - w - 40);
      const y = 20 + Math.random() * (state.canvasH - h - 40);
      const rotation = (Math.random() - 0.5) * 8 * (parseInt(document.getElementById('rotateRange').value) / 4);
      state.images.push({ id, src, x, y, w, h, rotation, tag: '', zIndex: state.images.length + 1 });
      renderAll();
      buildThumbs();
      extractColors();
    };
    reader.readAsDataURL(file);
  });
}

// ─────────────────────────────────────────
// COLOR EXTRACTION (simple canvas sampling)
// ─────────────────────────────────────────
function extractColors() {
  if (state.images.length === 0) return;
  const palettes = [];
  let done = 0;
  state.images.forEach(img => {
    const image = new Image();
    image.onload = () => {
      const c = document.createElement('canvas');
      c.width = 40; c.height = 40;
      const ctx = c.getContext('2d');
      ctx.drawImage(image, 0, 0, 40, 40);
      const d = ctx.getImageData(0, 0, 40, 40).data;
      let r=0,g=0,b=0,n=0;
      for(let i=0;i<d.length;i+=16){ r+=d[i];g+=d[i+1];b+=d[i+2];n++; }
      r=Math.round(r/n); g=Math.round(g/n); b=Math.round(b/n);
      // Darken for background
      const darken = v => Math.max(0, Math.round(v * 0.45));
      const warm = (r2,g2,b2) => [Math.min(255,r2+10), Math.max(0,g2-5), Math.max(0,b2-15)];
      const dc = warm(darken(r),darken(g),darken(b));
      palettes.push(toHex(...dc));
      done++;
      if(done === state.images.length) showAutoPalette(palettes);
    };
    image.src = img.src;
  });
}

function toHex(r,g,b){
  return '#'+[r,g,b].map(v=>v.toString(16).padStart(2,'0')).join('');
}

function showAutoPalette(colors) {
  const row = document.getElementById('autoBgRow');
  row.innerHTML = '';
  const unique = [...new Set(colors)];
  // Add some variations
  const extras = unique.map(c => {
    const [r,g,b] = hexToRgb(c);
    return toHex(Math.min(255,r+20),Math.min(255,g+20),Math.min(255,b+20));
  });
  const all = [...unique, ...extras].slice(0,8);
  all.forEach(color => {
    const sw = document.createElement('div');
    sw.className = 'auto-swatch';
    sw.style.background = color;
    sw.title = color;
    sw.onclick = () => {
      document.querySelectorAll('.auto-swatch').forEach(s=>s.classList.remove('active'));
      sw.classList.add('active');
      state.bgColor = color;
      bgPicker.value = color;
      applyBg();
    };
    row.appendChild(sw);
  });
}

function hexToRgb(hex) {
  const r = parseInt(hex.slice(1,3),16);
  const g = parseInt(hex.slice(3,5),16);
  const b = parseInt(hex.slice(5,7),16);
  return [r,g,b];
}

// ─────────────────────────────────────────
// BACKGROUND
// ─────────────────────────────────────────
bgPicker.oninput = () => { state.bgColor = bgPicker.value; applyBg(); };
bgBrightness.oninput = () => applyBg();

function applyBg() {
  const brightness = parseInt(bgBrightness.value) / 80;
  const [r,g,b] = hexToRgb(state.bgColor);
  const adj = v => Math.min(255, Math.round(v * brightness));
  const finalColor = toHex(adj(r),adj(g),adj(b));
  canvasWrap.style.background = finalColor;
  drawDecorations();
}

// ─────────────────────────────────────────
// THUMBNAILS
// ─────────────────────────────────────────
function buildThumbs() {
  thumbList.innerHTML = '';
  state.images.forEach(img => {
    const div = document.createElement('div');
    div.className = 'thumb-item';
    div.innerHTML = `<img src="${img.src}">
      <button class="remove-btn" onclick="removeImage('${img.id}')">×</button>
      <input class="tag-input" type="text" placeholder="标签..." value="${img.tag}" oninput="updateTag('${img.id}',this.value)">`;
    div.onclick = e => { if(e.target.tagName!=='INPUT'&&e.target.tagName!=='BUTTON') selectItem(img.id); };
    thumbList.appendChild(div);
  });
}

function removeImage(id) {
  state.images = state.images.filter(i => i.id !== id);
  buildThumbs();
  renderAll();
  extractColors();
}

function updateTag(id, val) {
  const img = state.images.find(i => i.id === id);
  if(img) { img.tag = val; renderAll(); }
}

// ─────────────────────────────────────────
// TEXT ITEMS
// ─────────────────────────────────────────
function addTextItem(text='(moments)') {
  const id = newId();
  state.texts.push({
    id, text,
    x: 20 + Math.random() * (state.canvasW - 100),
    y: 20 + Math.random() * (state.canvasH - 30),
    color: state.textColor,
    font: state.font,
    fontSize: state.fontSize,
  });
  renderTexts();
  renderTextList();
}

function renderTextList() {
  textListEl.innerHTML = '';
  state.texts.forEach(t => {
    const row = document.createElement('div');
    row.className = 'text-item-row';
    row.innerHTML = `<input type="text" value="${t.text}" oninput="updateText('${t.id}',this.value)">
      <button class="del-btn" onclick="removeText('${t.id}')">×</button>`;
    textListEl.appendChild(row);
  });
}

function updateText(id, val) {
  const t = state.texts.find(x => x.id === id);
  if(t) { t.text = val; renderTexts(); }
}

function removeText(id) {
  state.texts = state.texts.filter(t => t.id !== id);
  renderTexts();
  renderTextList();
}

// ─────────────────────────────────────────
// FONT / TEXT CONTROLS
// ─────────────────────────────────────────
textColorPicker.oninput = () => {
  state.textColor = textColorPicker.value;
  if(state.selectedId) {
    const t = state.texts.find(x => x.id === state.selectedId);
    if(t) { t.color = state.textColor; renderTexts(); }
  }
};
fontSelect.onchange = () => {
  state.font = fontSelect.value;
  if(state.selectedId) {
    const t = state.texts.find(x => x.id === state.selectedId);
    if(t) { t.font = state.font; renderTexts(); }
  }
};
fontSizeSlider.oninput = () => {
  state.fontSize = parseInt(fontSizeSlider.value);
  fontSizeVal.textContent = state.fontSize;
  if(state.selectedId) {
    const t = state.texts.find(x => x.id === state.selectedId);
    if(t) { t.fontSize = state.fontSize; renderTexts(); }
  }
};

// ─────────────────────────────────────────
// CANVAS SIZE
// ─────────────────────────────────────────
canvasWSlider.oninput = () => {
  state.canvasW = parseInt(canvasWSlider.value);
  document.getElementById('canvasWVal').textContent = state.canvasW;
  canvasWrap.style.width = state.canvasW + 'px';
  updateCorners();
  drawDecorations();
};
canvasHSlider.oninput = () => {
  state.canvasH = parseInt(canvasHSlider.value);
  document.getElementById('canvasHVal').textContent = state.canvasH;
  canvasWrap.style.height = state.canvasH + 'px';
  updateCorners();
  drawDecorations();
};

function updateCorners() {
  const W = state.canvasW, H = state.canvasH;
  document.getElementById('trBracket').setAttribute('d', `M ${W-30} 8 L ${W-8} 8 L ${W-8} 30`);
  document.getElementById('blBracket').setAttribute('d', `M 30 ${H-8} L 8 ${H-8} L 8 ${H-30}`);
  document.getElementById('brBracket').setAttribute('d', `M ${W-30} ${H-8} L ${W-8} ${H-8} L ${W-8} ${H-30}`);
}

// ─────────────────────────────────────────
// DECORATIONS
// ─────────────────────────────────────────
cornerOpacitySlider.oninput = () => {
  document.getElementById('cornerSvg').style.opacity = parseInt(cornerOpacitySlider.value) / 100;
};
dotsOpacitySlider.oninput = () => {
  document.getElementById('dotsCanvas').style.opacity = parseInt(dotsOpacitySlider.value) / 100;
};

function drawDecorations() {
  const canvas = document.getElementById('dotsCanvas');
  canvas.width = state.canvasW;
  canvas.height = state.canvasH;
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0,0,state.canvasW,state.canvasH);
  // Small scattered squares / dots
  const positions = [
    [0.05,0.12],[0.92,0.08],[0.08,0.85],[0.88,0.78],
    [0.15,0.45],[0.82,0.35],[0.5,0.05],[0.3,0.92],
    [0.65,0.88],[0.02,0.55],[0.95,0.5]
  ];
  positions.forEach(([px,py],i) => {
    const x = px * state.canvasW;
    const y = py * state.canvasH;
    const s = 4 + (i%3)*3;
    ctx.fillStyle = `rgba(240,235,227,0.3)`;
    if(i%2===0) {
      ctx.fillRect(x, y, s, s);
    } else {
      ctx.beginPath();
      ctx.arc(x, y, s/2, 0, Math.PI*2);
      ctx.fill();
    }
  });
}

// ─────────────────────────────────────────
// RENDER
// ─────────────────────────────────────────
function renderAll() {
  renderImages();
  renderTexts();
  drawDecorations();
}

function renderImages() {
  document.querySelectorAll('.collage-item').forEach(el => el.remove());
  state.images.forEach(img => {
    const el = document.createElement('div');
    el.className = 'collage-item';
    el.id = img.id;
    el.style.cssText = `left:${img.x}px;top:${img.y}px;width:${img.w}px;height:${img.h}px;
      transform:rotate(${img.rotation}deg);z-index:${img.zIndex};
      box-shadow:3px 3px 12px rgba(0,0,0,0.4);`;

    const image = document.createElement('img');
    image.src = img.src;
    el.appendChild(image);

    // Tag label
    if(img.tag) {
      const tagEl = document.createElement('div');
      tagEl.style.cssText = `position:absolute;bottom:4px;left:6px;font-family:'Caveat',cursive;
        font-size:12px;color:rgba(255,255,255,0.85);text-shadow:0 1px 3px rgba(0,0,0,0.5);
        pointer-events:none;`;
      tagEl.textContent = '(' + img.tag + ')';
      el.appendChild(tagEl);
    }

    // Resize handle
    const rh = document.createElement('div');
    rh.className = 'resize-handle';
    el.appendChild(rh);

    attachDrag(el, img);
    attachResize(rh, img);

    el.onclick = e => { e.stopPropagation(); selectItem(img.id); };
    el.oncontextmenu = e => { e.preventDefault(); bringToFront(img.id); };

    canvasWrap.insertBefore(el, document.getElementById('animOverlay'));
  });
}

function renderTexts() {
  document.querySelectorAll('.collage-text').forEach(el => el.remove());
  state.texts.forEach(t => {
    const el = document.createElement('div');
    el.className = 'collage-text';
    el.id = t.id;
    el.textContent = t.text;
    el.style.cssText = `left:${t.x}px;top:${t.y}px;color:${t.color};
      font-family:${t.font};font-size:${t.fontSize}px;z-index:20;`;
    attachDragText(el, t);
    el.onclick = e => { e.stopPropagation(); selectItem(t.id); };
    canvasWrap.insertBefore(el, document.getElementById('animOverlay'));
  });
}

// ─────────────────────────────────────────
// DRAG & DROP
// ─────────────────────────────────────────
function attachDrag(el, img) {
  el.addEventListener('mousedown', e => {
    if(e.target.classList.contains('resize-handle')) return;
    e.preventDefault();
    dragState = { el, img, startX: e.clientX - img.x, startY: e.clientY - img.y, type:'img' };
    selectItem(img.id);
  });
}

function attachDragText(el, t) {
  el.addEventListener('mousedown', e => {
    e.preventDefault();
    dragState = { el, item: t, startX: e.clientX - t.x, startY: e.clientY - t.y, type:'text' };
    selectItem(t.id);
  });
}

function attachResize(handle, img) {
  handle.addEventListener('mousedown', e => {
    e.preventDefault(); e.stopPropagation();
    const rect = canvasWrap.getBoundingClientRect();
    resizeState = { img, startW: img.w, startH: img.h, startX: e.clientX, startY: e.clientY };
  });
}

document.addEventListener('mousemove', e => {
  if(dragState) {
    if(dragState.type === 'img') {
      dragState.img.x = e.clientX - dragState.startX;
      dragState.img.y = e.clientY - dragState.startY;
      dragState.el.style.left = dragState.img.x + 'px';
      dragState.el.style.top = dragState.img.y + 'px';
    } else {
      dragState.item.x = e.clientX - dragState.startX;
      dragState.item.y = e.clientY - dragState.startY;
      dragState.el.style.left = dragState.item.x + 'px';
      dragState.el.style.top = dragState.item.y + 'px';
    }
  }
  if(resizeState) {
    const dx = e.clientX - resizeState.startX;
    const dy = e.clientY - resizeState.startY;
    resizeState.img.w = Math.max(40, resizeState.startW + dx);
    resizeState.img.h = Math.max(40, resizeState.startH + dy);
    const el = document.getElementById(resizeState.img.id);
    if(el) { el.style.width = resizeState.img.w+'px'; el.style.height = resizeState.img.h+'px'; }
  }
});

document.addEventListener('mouseup', () => { dragState = null; resizeState = null; });

canvasWrap.addEventListener('click', e => {
  if(e.target === canvasWrap) deselectAll();
});

// ─────────────────────────────────────────
// SELECTION
// ─────────────────────────────────────────
function selectItem(id) {
  deselectAll();
  state.selectedId = id;
  const el = document.getElementById(id);
  if(el) el.classList.add('selected');
  // Update controls for selected text
  const t = state.texts.find(x => x.id === id);
  if(t) {
    textColorPicker.value = t.color;
    fontSizeSlider.value = t.fontSize;
    fontSizeVal.textContent = t.fontSize;
  }
}

function deselectAll() {
  state.selectedId = null;
  document.querySelectorAll('.selected').forEach(el => el.classList.remove('selected'));
}

function bringToFront(id) {
  const img = state.images.find(i => i.id === id);
  if(img) {
    const maxZ = Math.max(...state.images.map(i => i.zIndex));
    img.zIndex = maxZ + 1;
    const el = document.getElementById(id);
    if(el) el.style.zIndex = img.zIndex;
  }
}

// ─────────────────────────────────────────
// LAYOUTS
// ─────────────────────────────────────────
function randomLayout() {
  const rotRange = parseInt(document.getElementById('rotateRange').value);
  state.images.forEach(img => {
    const sz = parseInt(document.getElementById('imgSizeSlider').value);
    img.w = sz * (0.7 + Math.random()*0.7);
    img.h = img.w * (0.7 + Math.random()*0.6);
    img.x = 10 + Math.random() * Math.max(10, state.canvasW - img.w - 20);
    img.y = 10 + Math.random() * Math.max(10, state.canvasH - img.h - 20);
    img.rotation = (Math.random()-0.5) * 2 * rotRange;
  });
  renderImages();
}

function gridLayout() {
  const n = state.images.length;
  if(n === 0) return;
  const cols = Math.ceil(Math.sqrt(n));
  const rows = Math.ceil(n / cols);
  const padX = 20, padY = 20, gapX = 10, gapY = 10;
  const cellW = (state.canvasW - padX*2 - gapX*(cols-1)) / cols;
  const cellH = (state.canvasH - padY*2 - gapY*(rows-1)) / rows;
  state.images.forEach((img, i) => {
    const col = i % cols, row = Math.floor(i / cols);
    img.x = padX + col * (cellW + gapX);
    img.y = padY + row * (cellH + gapY);
    img.w = cellW; img.h = cellH;
    img.rotation = 0;
  });
  renderImages();
}

// ─────────────────────────────────────────
// ANIMATION
// ─────────────────────────────────────────
const allElements = () => [...document.querySelectorAll('.collage-item, .collage-text')];

function resetAnimation() {
  clearTimeout(state.animTimeout);
  state.isPlaying = false;
  document.getElementById('timelineFill').style.width = '0%';
  allElements().forEach(el => { el.style.opacity='1'; el.style.transform = el.dataset.origTransform || el.style.transform; });
}

function playAnimation() {
  if(state.isPlaying) { resetAnimation(); return; }
  state.isPlaying = true;

  const els = allElements();
  // save original transforms
  els.forEach(el => { el.dataset.origTransform = el.style.transform || ''; });

  // hide all
  els.forEach(el => {
    el.style.transition = 'none';
    el.style.opacity = '0';
    if(el.classList.contains('collage-item')) {
      el.style.transform = (el.dataset.origTransform||'') + ' scale(0.85)';
    }
  });

  const speed = parseInt(document.getElementById('animSpeed').value);
  const delay = parseInt(document.getElementById('animDelay').value);
  const style = document.getElementById('animStyle').value;
  const total = els.length;

  els.forEach((el, i) => {
    state.animTimeout = setTimeout(() => {
      el.style.transition = `opacity ${speed}ms cubic-bezier(0.25,0.1,0.25,1), transform ${speed}ms cubic-bezier(0.25,0.1,0.25,1)`;
      el.style.opacity = '1';
      const orig = el.dataset.origTransform || '';
      switch(style) {
        case 'fadeSlide':
          el.style.transform = orig; break;
        case 'pop':
          el.style.transform = orig + ' scale(1)'; break;
        case 'drift':
          el.style.transform = orig; break;
        case 'rotate':
          el.style.transform = orig; break;
      }
      document.getElementById('timelineFill').style.width = ((i+1)/total*100)+'%';
      if(i === total-1) {
        setTimeout(() => {
          state.isPlaying = false;
          document.getElementById('timelineFill').style.width = '0%';
        }, speed + 200);
      }
    }, i * delay);
  });
}

animDelaySlider.oninput = () => { animDelayVal.textContent = animDelaySlider.value; };

// ─────────────────────────────────────────
// EXPORT
// ─────────────────────────────────────────
async function exportPNG() {
  deselectAll();
  const canvas = await html2canvas(canvasWrap, { useCORS: true, scale: 2, backgroundColor: null });
  const link = document.createElement('a');
  link.download = 'moments-collage.png';
  link.href = canvas.toDataURL('image/png');
  link.click();
}

async function exportGIF() {
  deselectAll();
  // Reset then capture frames
  const els = allElements();
  els.forEach(el => { el.style.opacity = '0'; });

  const frames = [];
  const delay = parseInt(document.getElementById('animDelay').value);
  const speed = parseInt(document.getElementById('animSpeed').value);

  // Capture blank frame
  const f0 = await html2canvas(canvasWrap, { useCORS:true, scale:1, backgroundColor:null });
  frames.push({ canvas: f0, delay: 300 });

  for(let i=0; i<els.length; i++) {
    els[i].style.opacity = '1';
    await new Promise(r => setTimeout(r, 80));
    const fc = await html2canvas(canvasWrap, { useCORS:true, scale:1, backgroundColor:null });
    frames.push({ canvas: fc, delay: delay });
  }
  // Hold last frame
  const last = await html2canvas(canvasWrap, { useCORS:true, scale:1, backgroundColor:null });
  frames.push({ canvas: last, delay: 1200 });

  // Restore
  els.forEach(el => el.style.opacity = '1');

  if(typeof GIF === 'undefined') {
    alert('GIF库加载失败，请导出PNG格式');
    return;
  }

  const gif = new GIF({
    workers: 2, quality: 10,
    width: canvasWrap.offsetWidth,
    height: canvasWrap.offsetHeight,
    workerScript: 'https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.worker.js'
  });

  frames.forEach(f => gif.addFrame(f.canvas, { delay: f.delay }));

  gif.on('finished', blob => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = 'moments-collage.gif';
    a.click();
  });

  gif.render();
}

// ─────────────────────────────────────────
// INIT
// ─────────────────────────────────────────
drawDecorations();
updateCorners();

// Default texts
addTextItem('(moments)');
addTextItem('we live in the present');
renderTextList();
</script>
</body>
</html>
