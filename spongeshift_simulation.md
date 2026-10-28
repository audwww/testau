<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>SpongeShift — Victoria Harbour Floating Greenery Sandbox</title>
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M12 1.5l9 5.25v10.5L12 22.5l-9-5.25V6.75z' fill='%230088cc'/%3E%3C/svg%3E">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --font-sans: 'Helvetica Neue', Helvetica, Arial, sans-serif;
    --text-xs: 12px;
    --text-sm: 14px;
    --text-base: 16px;
    --text-lg: 20px;
    
    --sea-primary: #0077b6;
    --sea-dark: #023e8a;
    --sea-light: #00b4d8;
    --sea-foam: #e0f4fc;
    
    --bg-main: #0a141c;
    --panel-bg: rgba(255, 255, 255, 0.94);
    --panel-border: rgba(180, 200, 215, 0.7);
    --panel-shadow: 0 16px 40px rgba(0, 0, 0, 0.25);
    
    --text-main: #1c2a38;
    --text-muted: #5a6e7f;
    --text-light: #8b9ea8;
    
    --danger: #d9381e;
    --radius: 2px; /* Sharp architectural corners */
    --tr: 200ms cubic-bezier(0.16, 1, 0.3, 1);
  }

  html { -webkit-text-size-adjust: none; text-size-adjust: none; -webkit-font-smoothing: antialiased; }
  html, body { height: 100vh; width: 100vw; overflow: hidden; background: #0a141c; }
  
  #stage {
    position: fixed; inset: 0;
    width: 100vw; height: 100vh;
    overflow: hidden;
  }
  #c {
    display: block;
    width: 100vw; height: 100vh;
    touch-action: none;
    cursor: crosshair;
  }
  #c.dragging { cursor: grabbing; }

  .hud { position: fixed; z-index: 10; }
  .panel {
    background: var(--panel-bg);
    border: 1px solid var(--panel-border);
    border-radius: var(--radius);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    box-shadow: var(--panel-shadow);
  }

  /* ---------- title ---------- */
  #title {
    top: 18px; left: 18px;
    padding: 14px 20px;
  }
  #title h1 {
    font-size: 20px;
    font-weight: 700;
    line-height: 1.1;
    color: var(--text-main);
  }
  #title h1 em { font-style: normal; color: var(--sea-primary); }
  #title p {
    font-size: var(--text-xs);
    color: var(--text-muted);
    margin-top: 4px;
    font-weight: 500;
  }

  /* ---------- stats ---------- */
  #stats {
    top: 18px; right: 18px;
    padding: 16px 20px;
    width: 260px;
  }
  #stats .label {
    font-size: var(--text-xs);
    letter-spacing: 0.1em;
    color: var(--text-muted);
    text-transform: uppercase;
    font-weight: 600;
    margin-bottom: 6px;
  }
  .bigstat {
    font-size: 34px;
    font-weight: 700;
    line-height: 1;
    color: var(--sea-dark);
    font-variant-numeric: tabular-nums lining-nums;
    display: flex; align-items: baseline; gap: 8px;
  }
  .bigstat small { font-size: var(--text-sm); color: var(--text-muted); font-weight: 500; }
  #protBar {
    height: 6px; border-radius: var(--radius);
    background: #e2eaf0;
    margin: 8px 0 12px;
    overflow: hidden;
  }
  #protFill {
    height: 100%; width: 0%;
    background: linear-gradient(90deg, var(--sea-light), var(--sea-primary));
    border-radius: var(--radius);
    transition: width 400ms var(--tr);
  }
  .statrow {
    display: flex; justify-content: space-between; align-items: baseline;
    font-size: var(--text-sm);
    padding: 5px 0;
    color: var(--text-muted);
    border-top: 1px solid rgba(180, 200, 215, 0.35);
  }
  .statrow b { font-variant-numeric: tabular-nums lining-nums; font-weight: 600; color: var(--text-main); }
  .statrow .val-sea { color: var(--sea-primary); }
  .disclaimer {
    margin-top: 10px;
    font-size: 10px;
    color: var(--text-light);
    line-height: 1.3;
  }

  /* ---------- controls ---------- */
  #controls {
    bottom: 20px; left: 50%;
    transform: translateX(-50%);
    padding: 12px 18px;
    display: flex; align-items: center; gap: 14px;
    max-width: calc(100vw - 24px);
    flex-wrap: wrap;
    justify-content: center;
  }
  .type-selector {
    display: flex; background: #e2eaf0; padding: 3px; border-radius: var(--radius); gap: 3px;
    border: 1px solid rgba(180, 200, 215, 0.6);
  }
  .type-btn {
    font-family: var(--font-sans);
    font-size: 12px; font-weight: 600;
    padding: 8px 14px; border: none; background: transparent;
    color: var(--text-muted); border-radius: var(--radius); cursor: pointer; transition: all var(--tr);
    white-space: nowrap;
  }
  .type-btn.active {
    background: #ffffff; color: var(--sea-dark); box-shadow: 0 2px 6px rgba(0,0,0,0.08);
  }
  #tyBtn {
    font-family: var(--font-sans);
    font-size: 13px;
    font-weight: 700;
    padding: 9px 16px;
    border-radius: var(--radius);
    border: 1px solid var(--sea-primary);
    background: linear-gradient(180deg, var(--sea-primary), var(--sea-dark));
    color: #ffffff;
    cursor: pointer;
    box-shadow: 0 4px 12px rgba(0, 119, 182, 0.25);
    transition: all var(--tr);
    white-space: nowrap;
  }
  #tyBtn:hover {
    background: linear-gradient(180deg, var(--sea-light), var(--sea-primary));
  }
  #tyBtn.active {
    border-color: var(--danger);
    background: linear-gradient(180deg, #e63946, #b71c1c);
  }
  .sliderbox { display: flex; flex-direction: column; gap: 3px; min-width: 130px; }
  .sliderbox label {
    font-size: var(--text-xs);
    color: var(--text-muted);
    font-weight: 600;
    display: flex; justify-content: space-between;
  }
  .sliderbox label b { color: var(--sea-primary); }
  input[type='range'] {
    -webkit-appearance: none; appearance: none;
    width: 100%; height: 5px; border-radius: var(--radius);
    background: #d8e4ec; outline: none; cursor: pointer;
  }
  input[type='range']::-webkit-slider-thumb {
    -webkit-appearance: none; appearance: none;
    width: 15px; height: 15px; border-radius: 50%;
    background: #ffffff; border: 2.5px solid var(--sea-primary);
    cursor: grab;
  }
  .ctlbtns { display: flex; gap: 6px; }
  .ctlbtn {
    font-family: var(--font-sans);
    font-size: var(--text-xs);
    font-weight: 600;
    padding: 8px 12px;
    border-radius: var(--radius);
    border: 1px solid var(--panel-border);
    background: #ffffff;
    color: var(--text-main);
    cursor: pointer;
    transition: all var(--tr);
    white-space: nowrap;
  }
  .ctlbtn:hover { background: var(--sea-foam); border-color: var(--sea-light); color: var(--sea-dark); }

  /* ---------- hint ---------- */
  #hint {
    bottom: 96px; left: 50%;
    transform: translateX(-50%);
    font-size: var(--text-xs);
    font-weight: 500;
    color: var(--text-muted);
    background: rgba(255, 255, 255, 0.94);
    border: 1px solid var(--panel-border);
    padding: 6px 16px;
    border-radius: var(--radius);
    backdrop-filter: blur(12px);
    pointer-events: none;
    text-align: center;
    box-shadow: 0 4px 12px rgba(0,0,0,0.06);
  }
  #hint b { color: var(--sea-primary); font-weight: 700; }

  /* ---------- signal badge ---------- */
  #signal {
    top: 18px; left: 50%;
    transform: translateX(-50%);
    display: none;
    align-items: center; gap: 10px;
    padding: 10px 18px;
    border-color: rgba(217, 56, 30, 0.4);
    background: rgba(255, 255, 255, 0.95);
  }
  #signal.on { display: flex; }
  #signal .num { font-size: 22px; font-weight: 800; color: var(--danger); line-height: 1; }
  #signal .txt { font-size: var(--text-xs); text-transform: uppercase; font-weight: 600; color: var(--text-muted); }
  #signal .txt b { display: block; color: var(--text-main); font-size: var(--text-sm); text-transform: none; font-weight: 700; }

  #fps {
    position: fixed; bottom: 8px; left: 12px;
    font-size: 11px; color: var(--text-light); z-index: 11; pointer-events: none;
  }
</style>
</head>
<body>

<div id="stage"><canvas id="c" aria-label="SpongeShift simulation"></canvas></div>

<header id="title" class="hud panel">
  <h1>Sponge<em>Shift</em></h1>
  <p>Victoria Harbour 3D floating greenery sandbox</p>
</header>

<div id="signal" class="hud panel" role="status">
  <span class="num" id="sigNum">8</span>
  <span class="txt">Simulated warning<b id="sigName">Signal No. 8</b></span>
</div>

<section id="stats" class="hud panel" aria-label="Simulation statistics">
  <div class="colA">
    <div class="label">Shore protection</div>
    <div class="bigstat"><span id="prot">0%</span><small>calmer</small></div>
    <div id="protBar"><div id="protFill"></div></div>
  </div>
  <div class="colB" style="flex:1">
    <div class="statrow"><span>Modules</span><b id="stModules">0</b></div>
    <div class="statrow"><span>Largest reef</span><b class="val-sea" id="stReef">0</b></div>
    <div class="statrow"><span>Water absorbed</span><b id="stWater">0 L</b></div>
    <div class="statrow"><span>Wind</span><b id="stWind">12 km/h</b></div>
  </div>
  <p class="disclaimer">Illustrative sandbox — wave heights, uptake litres and protection % are simulated estimates.</p>
</section>

<div id="hint" class="hud">
  <span>Click open water to plant selected module · <b>drag</b> to move · <b>right-click</b> or <b>double-click</b> to remove</span>
</div>

<footer id="controls" class="hud panel">
  <div class="type-selector" aria-label="Module Type">
    <button class="type-btn active" data-type="breakwater">Breakwater</button>
    <button class="type-btn" data-type="plant">Plant</button>
    <button class="type-btn" data-type="filtration">Filtration</button>
  </div>
  <button id="tyBtn" aria-pressed="false">Summon typhoon</button>
  <div class="sliderbox">
    <label for="tySlider"><span>Intensity</span><b id="tyVal">5</b></label>
    <input type="range" id="tySlider" min="1" max="10" step="1" value="5">
  </div>
  <div class="ctlbtns">
    <button class="ctlbtn" id="btnDemo">Demo reef</button>
    <button class="ctlbtn" id="btnClear">Clear</button>
    <button class="ctlbtn" id="btnSound">Sound off</button>
  </div>
</footer>

<div id="fps">60 fps</div>

<script>
'use strict';
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');

let VIEW = { W: window.innerWidth, H: window.innerHeight, shoreY: window.innerHeight * 0.22 };

function resizeCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  VIEW.W = window.innerWidth;
  VIEW.H = window.innerHeight;
  VIEW.shoreY = window.innerHeight * 0.22;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

/* Full screen oblique aerial projection filling 100% viewport */
const PERSP = { pow: 1.3, sx0: 0.15, sx1: 1.45 };
function perspD(y) { return Math.pow(clamp((y - VIEW.shoreY) / (VIEW.H - VIEW.shoreY), 0, 1), PERSP.pow); }
function perspS(d) { return PERSP.sx0 + (PERSP.sx1 - PERSP.sx0) * d; }
function mapPt(x, y) {
  const d = perspD(y);
  return { x: VIEW.W / 2 + (x - VIEW.W / 2) * perspS(d), y: VIEW.shoreY + (VIEW.H - VIEW.shoreY) * d, s: perspS(d) };
}
function unmapPt(sx, sy) {
  const d = clamp((sy - VIEW.shoreY) / (VIEW.H - VIEW.shoreY), 0, 1);
  const dd = Math.pow(d, 1 / PERSP.pow);
  return { x: VIEW.W / 2 + (sx - VIEW.W / 2) / perspS(d), y: VIEW.shoreY + dd * (VIEW.H - VIEW.shoreY) };
}
function localVScale(wy) { return Math.max(mapPt(0, wy + 1).y - mapPt(0, wy).y, 0.02); }

function projHexVerts(wcx, wcy, R, rot, tilt) {
  const vs = new Array(6);
  const ct = Math.cos(tilt || 0), st = Math.sin(tilt || 0);
  for (let i = 0; i < 6; i++) {
    const a = rot + Math.PI / 180 * (60 * i - 30);
    let vx = R * Math.cos(a), vy = R * Math.sin(a);
    if (tilt) { const nx = vx * ct - vy * st, ny = vx * st + vy * ct; vx = nx; vy = ny; }
    vs[i] = mapPt(wcx + vx, wcy + vy);
  }
  return vs;
}
function traceVerts(vs, ox, oy) {
  ctx.beginPath();
  for (let i = 0; i < 6; i++) { const p = vs[i]; i ? ctx.lineTo(p.x + ox, p.y + oy) : ctx.moveTo(p.x + ox, p.y + oy); }
  ctx.closePath();
}
function hexProjPath(wcx, wcy, R, rot) { traceVerts(projHexVerts(wcx, wcy, R, rot, 0), 0, 0); }

const CELL = 12;
let GW = Math.ceil(VIEW.W / CELL);
let GH = Math.ceil((VIEW.H - VIEW.shoreY) / CELL);
const HEXR = 26;
const SQRT3 = Math.sqrt(3);
const MAX_MODULES = 120;
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
const lerp = (a, b, t) => a + (b - a) * t;

const DIRS = [[1,0],[-1,0],[0,1],[0,-1],[1,-1],[-1,1]];
function axialToPixel(q, r) { return { x: VIEW.W / 2 + HEXR * SQRT3 * (q + r / 2), y: VIEW.shoreY + 120 + HEXR * 1.5 * r }; }
function pixelToAxial(x, y) {
  const dx = x - VIEW.W / 2;
  const dy = y - (VIEW.shoreY + 120);
  const q = (SQRT3 / 3 * dx - dy / 3) / HEXR;
  const r = (2 / 3 * dy) / HEXR;
  let cx = q, cz = r, cy = -cx - cz;
  let rx = Math.round(cx), ry = Math.round(cy), rz = Math.round(cz);
  const ddx = Math.abs(rx - cx), ddy = Math.abs(ry - cy), ddz = Math.abs(rz - cz);
  if (ddx > ddy && ddx > ddz) rx = -ry - rz; else if (ddy > ddz) ry = -rx - rz; else rz = -rx - ry;
  return { q: rx, r: rz };
}
function hexValid(q, r) {
  const p = axialToPixel(q, r);
  return p.x > -50 && p.x < VIEW.W + 50 && p.y > VIEW.shoreY + 30 && p.y < VIEW.H - 40;
}

let selectedModuleType = 'breakwater'; // 'breakwater', 'plant', 'filtration'
let moduleSeq = 0;
const modules = [];
const occ = new Map();
function key(q, r) { return q + ',' + r; }

function addModule(q, r, type = selectedModuleType) {
  if (modules.length >= MAX_MODULES || occ.has(key(q, r))) return null;
  const seed = Math.random() * 1000;
  const vr = mulberry((Math.floor(seed * 1000) ^ 0x9e3779b9) | 0);
  const m = {
    id: moduleSeq++,
    q, r, type,
    bob: 0, tilt: 0, bobV: 0,
    size: 0.88 + vr() * 0.12,
    hull: type === 'breakwater' ? 0.95 : 0.75,
    absorb: 0, swamped: 0, seed,
    comp: -1, neighbors: [],
  };
  modules.push(m);
  occ.set(key(q, r), m);
  onModulesChanged();
  return m;
}

function removeModule(m) {
  const i = modules.indexOf(m);
  if (i < 0) return;
  modules.splice(i, 1);
  occ.delete(key(m.q, m.r));
  onModulesChanged();
}

function findFreeHexNear(q, r) {
  if (!occ.has(key(q, r)) && hexValid(q, r)) return { q, r };
  for (let ring = 1; ring < 10; ring++) {
    for (let dq = -ring; dq <= ring; dq++) {
      for (let dr = Math.max(-ring, -ring - dq); dr <= Math.min(ring, ring - dq); dr++) {
        if (Math.max(Math.abs(dq), Math.abs(dr), Math.abs(dq + dr)) !== ring) continue;
        const nq = q + dq, nr = r + dr;
        if (!occ.has(key(nq, nr)) && hexValid(nq, nr)) return { q: nq, r: nr };
      }
    }
  }
  return null;
}

function recomputeComponents() {
  modules.forEach(m => { m.comp = -1; m.neighbors = []; });
  for (const m of modules) {
    for (const [dq, dr] of DIRS) {
      const b = occ.get(key(m.q + dq, m.r + dr));
      if (b) m.neighbors.push(b);
    }
  }
  let comps = 0; const sizes = [];
  for (const m of modules) {
    if (m.comp !== -1) continue;
    const stack = [m]; m.comp = comps; let size = 0;
    while (stack.length) {
      const a = stack.pop(); size++;
      for (const b of a.neighbors) if (b.comp === -1) { b.comp = comps; stack.push(b); }
    }
    sizes.push(size); comps++;
  }
  return sizes.length ? Math.max(...sizes) : 0;
}

/* Wave Simulation Field */
const MAX_GW = 200, MAX_GH = 120;
const hCur = new Float32Array(MAX_GW * MAX_GH), hPrev = new Float32Array(MAX_GW * MAX_GH);
const rCur = new Float32Array(MAX_GW * MAX_GH), rPrev = new Float32Array(MAX_GW * MAX_GH);
const mask = new Uint16Array(MAX_GW * MAX_GH);
const SPONGE = [0.35, 0.6, 0.78, 0.9, 0.965];
let hRef = hCur, hPrevRef = hPrev, rRef = rCur, rPrevRef = rPrev;
const colPhase = new Float32Array(MAX_GW);
const colNoise = new Float32Array(MAX_GW);
for (let x = 0; x < MAX_GW; x++) { colPhase[x] = Math.random() * Math.PI * 2; colNoise[x] = 0.72 + Math.random() * 0.28; }

function stepField(cur, prev, visc, applyModules) {
  GW = Math.min(MAX_GW, Math.ceil(VIEW.W / CELL));
  GH = Math.min(MAX_GH, Math.ceil((VIEW.H - VIEW.shoreY) / CELL));
  const C2 = 0.42;
  for (let y = 1; y < GH - 1; y++) {
    const row = y * GW;
    for (let x = 1; x < GW - 1; x++) {
      const i = row + x;
      const lap = cur[i - 1] + cur[i + 1] + cur[i - GW] + cur[i + GW] - 4 * cur[i];
      const v = 2 * cur[i] - prev[i] + C2 * lap;
      prev[i] = cur[i] + (v - cur[i]) * visc;
    }
  }
  const t = cur; cur = prev; prev = t;
  for (let x = 0; x < GW; x++) {
    const s = sourceAt(x);
    const iB = (GH - 1) * GW + x, iB2 = (GH - 2) * GW + x;
    cur[iB] = s; cur[iB2] = lerp(cur[iB2], s, 0.5);
  }
  for (let y = 0; y < SPONGE.length; y++)
    for (let x = 0; x < GW; x++) cur[y * GW + x] *= SPONGE[y];
  for (let y = 0; y < GH; y++) { cur[y * GW] = cur[y * GW + 1]; cur[y * GW + GW - 1] = cur[y * GW + GW - 2]; }

  if (applyModules) {
    for (let i = 0; i < GW * GH; i++) {
      const id = mask[i];
      if (id) {
        const m = modulesById[id - 1];
        if (m && m.swamped <= 0) {
          const n = m.neighbors.length;
          const powerFactor = m.type === 'breakwater' ? 0.45 : m.type === 'filtration' ? 0.60 : 0.72;
          const f = n === 0 ? powerFactor : Math.max(0.08, powerFactor - 0.12 * n);
          const before = cur[i];
          cur[i] *= f;
          m.absorb = Math.min(1, m.absorb + Math.abs(before) * 0.015);
          litersAccum += Math.abs(before) * (1 - f) * 0.005;
        }
      }
    }
  }
  return [cur, prev];
}

const modulesById = [];
function onModulesChanged() {
  mask.fill(0);
  modulesById.length = 0;
  for (const m of modules) {
    modulesById[m.id] = m;
    if (m.dragging) continue;
    const c = axialToPixel(m.q, m.r);
    const x0 = Math.max(0, Math.floor((c.x - HEXR * 1.5) / CELL)), x1 = Math.min(GW - 1, Math.ceil((c.x + HEXR * 1.5) / CELL));
    const y0 = Math.max(0, Math.floor((c.y - HEXR * 1.5 - VIEW.shoreY) / CELL)), y1 = Math.min(GH - 1, Math.ceil((c.y + HEXR * 1.5 - VIEW.shoreY) / CELL));
    for (let gy = y0; gy <= y1; gy++) {
      for (let gx = x0; gx <= x1; gx++) {
        const px = gx * CELL + CELL / 2, py = VIEW.shoreY + gy * CELL + CELL / 2;
        if (Math.hypot(px - c.x, py - c.y) < HEXR * 0.95) mask[gy * GW + gx] = m.id + 1;
      }
    }
  }
  largestReef = recomputeComponents();
}

const ty = { active: false, intensity: 5, level: 0 };
let sourceAmp = 1.3;
function sourceAt(x) {
  const t = simTime;
  const lv = ty.level / 10;
  let v = Math.sin(t * 3.5 + x * 0.05 + colPhase[x]) + 0.5 * Math.sin(t * 5.5 + x * 0.11 + colPhase[x] * 0.7);
  v += lv * (Math.sin(t * 7.0 - x * 0.06 + colPhase[x] * 0.3) * 0.9 + Math.sin(t * 9.5 + x * 0.1) * 0.6);
  return v * sourceAmp * colNoise[x];
}
function signalFor(level) {
  if (level < 0.5) return null;
  const i = Math.round(level);
  if (i <= 3) return { n: 3, name: 'Signal No. 3 — strong winds' };
  if (i <= 7) return { n: 8, name: 'Signal No. 8 — gale or storm' };
  return { n: 10, name: 'Signal No. 10 — hurricane force' };
}

let largestReef = 0, litersAccum = 0;
let shoreEnergy = 0, shoreEnergyRef = 0, shoreEnergyS = 0, shoreEnergyRefS = 0;
let protection = 0;

function bandEnergy(field, y0, y1) {
  let s = 0, n = 0;
  for (let y = y0; y <= y1; y++) for (let x = 4; x < GW - 4; x++) { s += Math.abs(field[y * GW + x]); n++; }
  return s / Math.max(1, n);
}

/* Pointer & Interaction */
const pointer = { x: 0, y: 0, down: false, drag: null, lastTapT: 0, lastTapM: null };
function toCanvas(e) {
  const r = canvas.getBoundingClientRect();
  return { x: e.clientX - r.left, y: e.clientY - r.top };
}
function toWorld(e) { const c = toCanvas(e); return unmapPt(c.x, c.y); }
function moduleAt(x, y) {
  let best = null, bd = HEXR * 1.15;
  for (const m of modules) {
    const c = axialToPixel(m.q, m.r);
    const d = Math.hypot(x - c.x, y - c.y);
    if (d < bd) { bd = d; best = m; }
  }
  return best;
}

canvas.addEventListener('contextmenu', e => {
  e.preventDefault();
  const cp = toCanvas(e); const p = unmapPt(cp.x, cp.y);
  const m = moduleAt(p.x, p.y);
  if (m) removeModule(m);
});
canvas.addEventListener('pointerdown', e => {
  if (e.button === 2) return;
  const cp = toCanvas(e); const p = unmapPt(cp.x, cp.y);
  pointer.x = p.x; pointer.y = p.y; pointer.down = true;
  let m = moduleAt(p.x, p.y);
  const now = performance.now();
  if (m && now - pointer.lastTapT < 320 && pointer.lastTapM === m) { removeModule(m); pointer.down = false; return; }
  pointer.lastTapT = now; pointer.lastTapM = m;
  if (!m && p.y > VIEW.shoreY + 20 && p.x >= 0 && p.x <= VIEW.W && p.y <= VIEW.H) {
    const a = pixelToAxial(p.x, p.y);
    const spot = findFreeHexNear(a.q, a.r);
    if (spot) { addModule(spot.q, spot.r); }
  }
  if (m) {
    pointer.drag = { m, dx: p.x - axialToPixel(m.q, m.r).x, dy: p.y - axialToPixel(m.q, m.r).y, oq: m.q, or: m.r };
    occ.delete(key(m.q, m.r));
    m.dragX = p.x - pointer.drag.dx; m.dragY = p.y - pointer.drag.dy; m.dragging = true;
    canvas.classList.add('dragging');
    canvas.setPointerCapture(e.pointerId);
    onModulesChanged();
  }
});
canvas.addEventListener('pointermove', e => {
  const p = toWorld(e); pointer.x = p.x; pointer.y = p.y;
  if (pointer.down && pointer.drag) {
    pointer.drag.m.dragX = p.x - pointer.drag.dx;
    pointer.drag.m.dragY = p.y - pointer.drag.dy;
  }
});
function endDrag() {
  if (pointer.drag) {
    const m = pointer.drag.m; m.dragging = false;
    const a = pixelToAxial(m.dragX, m.dragY);
    const spot = findFreeHexNear(a.q, a.r) || { q: pointer.drag.oq, r: pointer.drag.or };
    m.q = spot.q; m.r = spot.r;
    occ.set(key(m.q, m.r), m);
    onModulesChanged();
    pointer.drag = null;
    canvas.classList.remove('dragging');
  }
  pointer.down = false;
}
canvas.addEventListener('pointerup', endDrag);
canvas.addEventListener('pointercancel', endDrag);

/* UI Controls */
const $ = id => document.getElementById(id);
document.querySelectorAll('.type-btn').forEach(btn => {
  btn.addEventListener('click', e => {
    document.querySelectorAll('.type-btn').forEach(b => b.classList.remove('active'));
    e.target.classList.add('active');
    selectedModuleType = e.target.dataset.type;
  });
});
$('tyBtn').addEventListener('click', () => {
  ty.active = !ty.active;
  $('tyBtn').textContent = ty.active ? 'Stop typhoon' : 'Summon typhoon';
  $('tyBtn').classList.toggle('active', ty.active);
});
$('tySlider').addEventListener('input', e => { ty.intensity = +e.target.value; $('tyVal').textContent = ty.intensity; });
$('btnClear').addEventListener('click', () => { while (modules.length) removeModule(modules[0]); });
$('btnDemo').addEventListener('click', buildDemoReef);

function buildDemoReef() {
  while (modules.length) removeModule(modules[0]);
  for (let i = -12; i <= 12; i++) {
    const r = Math.round(i * 0.4);
    const type = i % 3 === 0 ? 'plant' : i % 3 === 1 ? 'filtration' : 'breakwater';
    addModule(i, r + 4, type);
  }
}

/* Water rendering buffer */
const waterOff = document.createElement('canvas');
waterOff.width = MAX_GW; waterOff.height = MAX_GH;
const wctx = waterOff.getContext('2d');
const waterImg = wctx.createImageData(MAX_GW, MAX_GH);
const CDEEP = [0, 105, 148], CMID = [0, 150, 199], CCREST = [72, 202, 228], CWHITE = [240, 248, 255];

function mulberry(seed) { return function () { seed |= 0; seed = (seed + 0x6D2B79F5) | 0; let t = Math.imul(seed ^ (seed >>> 15), 1 | seed); t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t; return ((t ^ (t >>> 14)) >>> 0) / 4294967296; }; }

const buildings = [
  [-200, 70, 85], [-120, 50, 130], [-60, 80, 105], [30, 45, 160], [90, 60, 115], [160, 75, 95],
  [250, 40, 190], [300, 60, 135], [380, 50, 105], [440, 80, 80], [530, 45, 220], [590, 55, 125],
  [660, 65, 95], [740, 45, 250], [800, 60, 140], [880, 50, 110], [940, 80, 85], [1030, 45, 230],
  [1090, 60, 135], [1170, 50, 100], [1240, 70, 150], [1330, 55, 105], [1400, 80, 120]
];

function drawSkyline(t) {
  const lv = clamp(ty.level / 10, 0, 1);
  const base = VIEW.shoreY - 10;
  ctx.fillStyle = rgb(mix([190, 205, 218], [65, 80, 92], lv));
  ctx.fillRect(0, 0, VIEW.W, base);

  const bFace = mix([245, 248, 252], [105, 118, 128], lv);
  const bSide = mix([210, 218, 225], [85, 98, 108], lv);
  const bLine = rgb(mix([140, 155, 165], [45, 55, 65], lv));
  ctx.strokeStyle = bLine; ctx.lineWidth = 1;

  for (const [x, w, h] of buildings) {
    const dx = 10, dy = -7;
    ctx.fillStyle = rgb(bSide);
    ctx.beginPath();
    ctx.moveTo(x + w, base - h); ctx.lineTo(x + w + dx, base - h + dy);
    ctx.lineTo(x + w + dx, base + dy); ctx.lineTo(x + w, base); ctx.closePath(); ctx.fill();
    ctx.fillStyle = rgb(bFace);
    ctx.fillRect(x, base - h, w, h);
    ctx.strokeRect(x + 0.5, base - h + 0.5, w - 1, h - 1);
  }
}

function mix(a, b, t) { return [lerp(a[0], b[0], t), lerp(a[1], b[1], t), lerp(a[2], b[2], t)]; }
function rgb(c) { return `rgb(${Math.round(c[0])},${Math.round(c[1])},${Math.round(c[2])})`; }

function drawWater() {
  const d = waterImg.data;
  const lv = clamp(ty.level / 10, 0, 1);
  const foamT = 3.4 - lv * 1.9;
  const deep = mix(CDEEP, [35, 60, 82], lv * 0.62), mid = mix(CMID, [45, 78, 105], lv * 0.62),
        crest = mix(CCREST, [85, 128, 158], lv * 0.62);
  for (let i = 0; i < GW * GH; i++) {
    const v = hRef[i];
    const tt = clamp(v / 6, -1, 1);
    let col = tt >= 0 ? mix(mid, crest, tt) : mix(mid, deep, -tt);
    const f = clamp((v - foamT) / 4.5, 0, 1) * (0.35 + lv * 0.5);
    if (f > 0) col = mix(col, CWHITE, f);
    const j = i * 4;
    d[j] = col[0]; d[j + 1] = col[1]; d[j + 2] = col[2]; d[j + 3] = 255;
  }
  wctx.putImageData(waterImg, 0, 0);
  ctx.imageSmoothingEnabled = true;
  for (let r = 0; r < GH; r++) {
    const y0 = VIEW.shoreY + r * CELL;
    const d0 = perspD(y0), d1 = perspD(y0 + CELL);
    const s = perspS((d0 + d1) / 2);
    const left = VIEW.W / 2 - (VIEW.W / 2) * s;
    ctx.drawImage(waterOff, 0, r, GW, 1, left, VIEW.shoreY + (VIEW.H - VIEW.shoreY) * d0, VIEW.W * s, (VIEW.H - VIEW.shoreY) * (d1 - d0) + 0.9);
  }
}

/* Render Modules matching reference model illustrations */
function drawModule(m, t) {
  let wx, wy;
  if (m.dragging) { wx = m.dragX; wy = m.dragY; }
  else { const p = axialToPixel(m.q, m.r); wx = p.x; wy = p.y; }
  const swamped = m.swamped > 0;
  ctx.save();
  if (swamped) ctx.globalAlpha = 0.55;
  const bob = m.dragging ? 0 : m.bob - (swamped ? 6 : 0);
  const mp = mapPt(wx, wy);
  const s = mp.s, R = HEXR * m.size, heightPx = 7 * m.hull * s;
  const e = bob * s;
  const deck = -e - heightPx;
  const vsc = localVScale(wy);
  const vs = projHexVerts(wx, wy, R, 0, m.tilt * s);

  // Shadow
  ctx.fillStyle = 'rgba(2, 62, 138, 0.22)';
  ctx.beginPath(); ctx.ellipse(mp.x, mp.y - e + 3, R * 1.05 * s, R * 0.5 * vsc, 0, 0, Math.PI * 2); ctx.fill();

  if (m.type === 'breakwater') {
    // BREAKWATER: Heavy concrete capped armor module with studded face ports
    ctx.fillStyle = '#6c757d';
    for (let i = 0; i < heightPx; i += 1.4) { traceVerts(vs, 0, -e - i); ctx.fill(); }
    traceVerts(vs, 0, deck); ctx.fillStyle = '#adb5bd'; ctx.fill();
    ctx.strokeStyle = '#343a40'; ctx.lineWidth = 1.5 * s; ctx.stroke();
    
    // Elevated chamfered central cap
    const capVs = projHexVerts(wx, wy, R * 0.65, 0, m.tilt * s);
    traceVerts(capVs, 0, deck - 4 * s);
    ctx.fillStyle = '#ced4da'; ctx.fill(); ctx.stroke();
    
    // Armor studs on side slopes
    ctx.fillStyle = '#495057';
    for (let i = 0; i < 6; i++) {
      const angle = (i * 60 + 30) * Math.PI / 180;
      const sx = mp.x + Math.cos(angle) * R * 0.5 * s;
      const sy = mp.y + deck + Math.sin(angle) * R * 0.3 * vsc + 2 * s;
      ctx.beginPath(); ctx.arc(sx, sy, 2 * s, 0, Math.PI * 2); ctx.fill();
    }
  } else if (m.type === 'plant') {
    // PLANT: Beveled outer concrete ring with recessed soil bed and mangrove tree
    ctx.fillStyle = '#5d4037';
    for (let i = 0; i < heightPx; i += 1.4) { traceVerts(vs, 0, -e - i); ctx.fill(); }
    traceVerts(vs, 0, deck); ctx.fillStyle = '#8d6e63'; ctx.fill();
    ctx.strokeStyle = '#3e2723'; ctx.lineWidth = 1.5 * s; ctx.stroke();
    
    // Inner green planter soil bed
    const innerVs = projHexVerts(wx, wy, R * 0.62, 0, m.tilt * s);
    traceVerts(innerVs, 0, deck);
    ctx.fillStyle = '#33691e'; ctx.fill(); ctx.stroke();
    
    // Mangrove tree canopy & roots
    ctx.strokeStyle = '#4e342e'; ctx.lineWidth = 2.2 * s;
    ctx.beginPath();
    ctx.moveTo(mp.x, mp.y + deck);
    ctx.lineTo(mp.x, mp.y + deck - 24 * s);
    ctx.stroke();
    
    // Canopy foliage
    ctx.fillStyle = '#558b2f';
    ctx.beginPath(); ctx.arc(mp.x - 3 * s, mp.y + deck - 28 * s, 11 * s, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = '#689f38';
    ctx.beginPath(); ctx.arc(mp.x + 4 * s, mp.y + deck - 26 * s, 9 * s, 0, Math.PI * 2); ctx.fill();
  } else if (m.type === 'filtration') {
    // FILTRATION: Modular eco-basin with water layer and attached cylindrical filters
    ctx.fillStyle = '#37474f';
    for (let i = 0; i < heightPx; i += 1.4) { traceVerts(vs, 0, -e - i); ctx.fill(); }
    traceVerts(vs, 0, deck); ctx.fillStyle = '#607d8b'; ctx.fill();
    ctx.strokeStyle = '#212121'; ctx.lineWidth = 1.5 * s; ctx.stroke();
    
    // Water treatment basin interior
    const basinVs = projHexVerts(wx, wy, R * 0.65, 0, m.tilt * s);
    traceVerts(basinVs, 0, deck);
    ctx.fillStyle = '#00bcd4'; ctx.fill(); ctx.stroke();
    
    // Side intake filter cylinders along outer edges
    ctx.fillStyle = '#eceff1'; ctx.strokeStyle = '#455a64'; ctx.lineWidth = 1 * s;
    ctx.beginPath();
    ctx.ellipse(mp.x - R * 0.7 * s, mp.y + deck + 3 * s, 7 * s, 3.5 * s, 0, 0, Math.PI * 2);
    ctx.fill(); ctx.stroke();
    ctx.beginPath();
    ctx.ellipse(mp.x + R * 0.7 * s, mp.y + deck + 3 * s, 7 * s, 3.5 * s, 0, 0, Math.PI * 2);
    ctx.fill(); ctx.stroke();
  }

  if (m.absorb > 0.03) {
    ctx.strokeStyle = 'rgba(0, 229, 255, 0.9)';
    ctx.lineWidth = 3 * s; ctx.lineCap = 'round';
    ctx.beginPath();
    ctx.ellipse(mp.x, mp.y + deck, R * 0.8 * s, R * 0.8 * vsc, 0, -Math.PI / 2, -Math.PI / 2 + m.absorb * Math.PI * 2);
    ctx.stroke(); ctx.lineCap = 'butt';
  }
  ctx.restore();
  m.absorb *= Math.pow(0.995, 1);
}

function drawConnections() {
  ctx.save(); ctx.lineCap = 'round';
  for (const m of modules) {
    for (const b of m.neighbors) {
      if (b.id < m.id || m.dragging || b.dragging) continue;
      const a1 = axialToPixel(m.q, m.r), a2 = axialToPixel(b.q, b.r);
      const p1 = mapPt(a1.x, a1.y), p2 = mapPt(a2.x, a2.y);
      p1.y -= (m.bob + 7 * m.hull) * p1.s; p2.y -= (b.bob + 7 * b.hull) * p2.s;
      const bs = (p1.s + p2.s) / 2;
      ctx.strokeStyle = 'rgba(0, 180, 216, 0.5)'; ctx.lineWidth = 4 * bs;
      ctx.beginPath(); ctx.moveTo(p1.x, p1.y); ctx.lineTo(p2.x, p2.y); ctx.stroke();
      ctx.strokeStyle = 'rgba(255, 255, 255, 0.9)'; ctx.lineWidth = 1.2 * bs;
      ctx.beginPath(); ctx.moveTo(p1.x, p1.y); ctx.lineTo(p2.x, p2.y); ctx.stroke();
    }
  }
  ctx.restore();
}

function drawHexGrid() {
  ctx.save(); ctx.strokeStyle = 'rgba(180, 205, 220, 0.22)'; ctx.lineWidth = 1;
  for (let r = -8; r < 25; r++) {
    for (let q = -25; q < 25; q++) {
      const p = axialToPixel(q, r);
      if (p.y < VIEW.shoreY + 15 || p.y > VIEW.H + 50) continue;
      hexProjPath(p.x, p.y, HEXR, 0); ctx.stroke();
    }
  }
  ctx.restore();
}

/* Main Loop */
let simTime = 0, dt = 0, lastFrame = performance.now(), acc = 0;
const TICK = 1000 / 60;
let frames = 0, fpsT = 0;

function update() {
  simTime += TICK / 1000;
  const target = ty.active ? ty.intensity : 0;
  ty.level += (target - ty.level) * 0.02;
  sourceAmp = 1.3 + (ty.level / 10) * 6.2;

  const out = stepField(hRef, hPrevRef, 0.998, true);
  hRef = out[0]; hPrevRef = out[1];
  const out2 = stepField(rRef, rPrevRef, 0.998, false);
  rRef = out2[0]; rPrevRef = out2[1];

  shoreEnergy = bandEnergy(hRef, 0, 3);
  shoreEnergyRef = bandEnergy(rRef, 0, 3);
  shoreEnergyS += (shoreEnergy - shoreEnergyS) * 0.02;
  shoreEnergyRefS += (shoreEnergyRef - shoreEnergyRefS) * 0.02;
  const protRaw = clamp(1 - (shoreEnergyS + 0.001) / (shoreEnergyRefS + 0.001), 0, 1);
  protection += (protRaw - protection) * 0.06;

  for (const m of modules) {
    const c = m.dragging ? { x: m.dragX, y: m.dragY } : axialToPixel(m.q, m.r);
    const gx = clamp(Math.round((c.x - 5) / CELL), 0, GW - 1);
    const gy = clamp(Math.round((c.y - VIEW.shoreY) / CELL), 0, GH - 1);
    const hFace = hRef[clamp(gy + 3, 0, GH - 1) * GW + gx];
    const mag = Math.pow(Math.abs(hFace), 0.7) * 1.35;
    const targetBob = hFace >= 0 ? Math.min(mag, 9) : Math.max(-mag, -9);
    m.bobV += (targetBob - m.bob) * 0.25; m.bobV *= 0.72; m.bob += m.bobV;
  }
}

function render() {
  const t = simTime;
  drawSkyline(t);
  drawWater();
  drawHexGrid();
  drawConnections();

  const sorted = [...modules].sort((a, b) => {
    const pa = a.dragging ? a.dragY : axialToPixel(a.q, a.r).y;
    const pb = b.dragging ? b.dragY : axialToPixel(b.q, b.r).y;
    return pa - pb;
  });
  for (const m of sorted) drawModule(m, t);

  if (pointer.drag) {
    const a = pixelToAxial(pointer.drag.m.dragX, pointer.drag.m.dragY);
    const p = axialToPixel(a.q, a.r);
    const free = !occ.has(key(a.q, a.r)) && hexValid(a.q, a.r);
    ctx.strokeStyle = free ? 'rgba(0, 180, 216, 0.9)' : 'rgba(217, 56, 30, 0.9)';
    ctx.lineWidth = 2; hexProjPath(p.x, p.y, HEXR * pointer.drag.m.size, 0); ctx.stroke();
  }
}

function updateHUD() {
  const protPct = Math.round(protection * 100);
  $('prot').textContent = protPct + '%';
  $('protFill').style.width = protPct + '%';
  $('stModules').textContent = modules.length;
  $('stReef').textContent = largestReef + (largestReef === 1 ? ' hex' : ' hexes');
  $('stWater').textContent = Math.round(litersAccum) + ' L';
  $('stWind').textContent = Math.round(12 + (ty.level / 10) * 245) + ' km/h';
  const sig = signalFor(ty.level);
  if (sig && ty.level > 0.4) {
    $('signal').classList.add('on');
    $('sigNum').textContent = sig.n; $('sigName').textContent = sig.name;
  } else $('signal').classList.remove('on');
}

function loop(now) {
  const raw = now - lastFrame; lastFrame = now;
  dt = Math.min(raw / 1000, 0.05);
  acc += Math.min(raw, 100);
  while (acc >= TICK) { update(); acc -= TICK; }
  render();
  frames++; fpsT += raw;
  if (fpsT >= 500) { $('fps').textContent = Math.round(frames / (fpsT / 1000)) + ' fps'; frames = 0; fpsT = 0; updateHUD(); }
  requestAnimationFrame(loop);
}

addModule(-1, 2, 'breakwater'); addModule(0, 2, 'plant'); addModule(1, 2, 'filtration');
updateHUD();
requestAnimationFrame(loop);
</script>
</body>
</html>