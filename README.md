<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>FreshScan — Food & Beverage Inventory (Google Vision)</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap');

:root {
  --cream:   #faf7f2;
  --paper:   #f2ede4;
  --warm1:   #e8dfd0;
  --warm2:   #d4c9b5;
  --ink:     #1c1a17;
  --ink2:    #3d3a35;
  --muted:   #8a8278;
  --faint:   #b8b0a4;
  --green:   #2d6a4f;
  --green2:  #40916c;
  --green3:  #74c69d;
  --red:     #ae2012;
  --orange:  #ca6702;
  --radius:  10px;
  --shadow:  0 2px 12px rgba(28,26,23,0.1);
  --shadow2: 0 8px 32px rgba(28,26,23,0.15);
}

* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  background: var(--cream);
  color: var(--ink);
  font-family: 'DM Sans', sans-serif;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

body::before {
  content: '';
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E");
  pointer-events: none;
  z-index: 0;
  opacity: 0.6;
}

header {
  position: relative; z-index: 1;
  display: flex; align-items: center; justify-content: space-between;
  padding: 16px 28px;
  background: var(--ink);
  border-bottom: 3px solid var(--green);
}
.logo { display: flex; align-items: baseline; gap: 10px; }
.logo-word { font-family: 'DM Serif Display', serif; font-size: 22px; color: var(--cream); letter-spacing: -0.01em; }
.logo-word em { color: var(--green3); font-style: italic; }
.logo-sub { font-size: 10px; font-family: 'DM Mono', monospace; letter-spacing: 0.15em; text-transform: uppercase; color: var(--faint); }
.header-right { display: flex; gap: 10px; align-items: center; }
.count-pill { background: var(--green); color: var(--cream); font-family: 'DM Mono', monospace; font-size: 11px; padding: 4px 12px; border-radius: 20px; letter-spacing: 0.05em; }

.app {
  position: relative; z-index: 1;
  display: grid; grid-template-columns: 400px 1fr;
  flex: 1; height: calc(100vh - 61px); overflow: hidden;
}

/* LEFT */
.left-panel { background: var(--paper); border-right: 1px solid var(--warm2); display: flex; flex-direction: column; overflow-y: auto; }
.section { padding: 16px 20px; border-bottom: 1px solid var(--warm1); }
.section-label { font-family: 'DM Mono', monospace; font-size: 9px; text-transform: uppercase; letter-spacing: 0.18em; color: var(--muted); margin-bottom: 10px; }
.key-row { display: flex; gap: 8px; }

input[type=password], input[type=text], select {
  flex: 1; background: var(--cream); border: 1.5px solid var(--warm2); border-radius: 7px;
  padding: 8px 12px; font-family: 'DM Mono', monospace; font-size: 12px; color: var(--ink); outline: none; transition: border-color 0.2s;
}
input:focus, select:focus { border-color: var(--green2); }

.video-wrapper { position: relative; border-radius: var(--radius); overflow: hidden; background: var(--ink); aspect-ratio: 4/3; box-shadow: var(--shadow); }
video { width: 100%; height: 100%; object-fit: cover; display: block; }
.cam-overlay { position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 10px; background: rgba(28,26,23,0.82); }
.cam-overlay p { font-size: 12px; color: var(--faint); text-align: center; }

.scanner-line { position: absolute; left: 0; right: 0; height: 2px; background: linear-gradient(90deg, transparent, var(--green3), transparent); animation: scan 3s ease-in-out infinite; display: none; box-shadow: 0 0 8px var(--green3); }
.live .scanner-line { display: block; }
@keyframes scan { 0%{top:10%;opacity:0}10%{opacity:1}90%{opacity:1}100%{top:90%;opacity:0} }

.corner-mark { position: absolute; width: 18px; height: 18px; border-color: var(--green3); border-style: solid; pointer-events: none; opacity: 0; transition: opacity 0.4s; }
.live .corner-mark { opacity: 1; }
.cm-tl { top:8px;left:8px;border-width:2px 0 0 2px; }
.cm-tr { top:8px;right:8px;border-width:2px 2px 0 0; }
.cm-bl { bottom:8px;left:8px;border-width:0 0 2px 2px; }
.cm-br { bottom:8px;right:8px;border-width:0 2px 2px 0; }

.live-badge { position: absolute; top:8px; left:50%; transform:translateX(-50%); display:none; align-items:center; gap:5px; background:rgba(28,26,23,0.75); padding:3px 10px; border-radius:20px; font-family:'DM Mono',monospace; font-size:9px; letter-spacing:0.12em; color:var(--green3); }
.live .live-badge { display:flex; }
.live-dot { width:5px;height:5px;border-radius:50%;background:var(--green3);animation:blink 1.2s infinite; }
@keyframes blink { 0%,100%{opacity:1}50%{opacity:0.2} }

.cam-row { display:flex;gap:8px;margin-bottom:10px; }
.cam-row select { font-size:11px; }

.shots-grid { display:grid; grid-template-columns:repeat(2,1fr); gap:8px; margin-bottom:12px; }
.slot { aspect-ratio:4/3; border-radius:7px; border:1.5px dashed var(--warm2); background:var(--cream); position:relative; overflow:hidden; cursor:pointer; transition:border-color 0.2s,box-shadow 0.2s; }
.slot:hover { border-color:var(--green2); }
.slot.filled { border-style:solid;border-color:var(--green); }
.slot.active { border-color:var(--orange);border-style:solid;box-shadow:0 0 0 3px rgba(202,103,2,0.15); }
.slot img { width:100%;height:100%;object-fit:cover; }
.slot-lbl { position:absolute;bottom:0;left:0;right:0;font-family:'DM Mono',monospace;font-size:8px;letter-spacing:0.08em;text-align:center;padding:2px 3px;background:rgba(28,26,23,0.65);color:#fff; }
.slot.active .slot-lbl { background:rgba(202,103,2,0.85); }
.slot-ph { display:flex;align-items:center;justify-content:center;height:100%;color:var(--warm2);font-size:20px; }
.slot-rm { position:absolute;top:3px;right:3px;width:15px;height:15px;border-radius:50%;background:var(--red);color:#fff;border:none;cursor:pointer;font-size:8px;display:none;align-items:center;justify-content:center; }
.slot.filled:hover .slot-rm { display:flex; }

.btn { display:inline-flex;align-items:center;justify-content:center;gap:6px;padding:9px 16px;border-radius:8px;border:none;font-family:'DM Sans',sans-serif;font-size:13px;font-weight:500;cursor:pointer;transition:all 0.18s;white-space:nowrap; }
.btn:disabled { opacity:0.38;cursor:not-allowed; }
.btn-green { background:var(--green);color:#fff; }
.btn-green:hover:not(:disabled) { background:var(--green2); }
.btn-outline { background:transparent;border:1.5px solid var(--warm2);color:var(--ink2); }
.btn-outline:hover:not(:disabled) { border-color:var(--green2);color:var(--green); }
.btn-ink { background:var(--ink);color:var(--cream); }
.btn-ink:hover:not(:disabled) { background:var(--ink2); }
.btn-red { background:var(--red);color:#fff; }
.btn-red:hover:not(:disabled) { background:#c1432e; }
.btn-sm { padding:6px 12px;font-size:12px; }
.btn-full { width:100%; }
.btn-icon { background:var(--cream);border:1.5px solid var(--warm2);color:var(--muted);padding:7px 10px; }
.btn-icon:hover:not(:disabled) { border-color:var(--green2);color:var(--green); }
.row { display:flex;gap:8px; }

canvas { display:none; }

/* RIGHT */
.right-panel { display:flex;flex-direction:column;overflow:hidden;background:var(--cream); }
.right-header { padding:14px 24px;border-bottom:1px solid var(--warm1);display:flex;align-items:center;justify-content:space-between;background:var(--paper);flex-wrap:wrap;gap:10px; }
.export-row { display:flex;gap:8px; }

.analyzing-bar { margin:14px 24px 0;background:var(--paper);border:1.5px solid var(--green3);border-radius:var(--radius);padding:14px 18px;display:none;align-items:center;gap:14px; }
.analyzing-bar.on { display:flex; }
.spinner { width:26px;height:26px;border:2.5px solid var(--warm1);border-top-color:var(--green);border-radius:50%;animation:spin 0.75s linear infinite;flex-shrink:0; }
@keyframes spin { to{transform:rotate(360deg)} }
.analyzing-text { font-size:13px;color:var(--ink2); }
.analyzing-text strong { display:block;color:var(--green);font-size:14px;margin-bottom:2px; }

.review-card { margin:14px 24px 0;background:var(--paper);border:1.5px solid var(--green);border-radius:var(--radius);overflow:hidden;display:none;box-shadow:var(--shadow); }
.review-card.on { display:block; }
.rc-head { padding:10px 16px;background:var(--green);display:flex;align-items:center;justify-content:space-between; }
.rc-head-title { font-family:'DM Serif Display',serif;font-size:15px;color:#fff;letter-spacing:0.01em; }
.rc-head-sub { font-size:11px;color:rgba(255,255,255,0.7);font-family:'DM Mono',monospace; }
.rc-fields { display:grid;grid-template-columns:repeat(3,1fr); }
.rc-field { padding:12px 16px;border-bottom:1px solid var(--warm1);border-right:1px solid var(--warm1); }
.rc-field:nth-child(3n) { border-right:none; }
.rc-field:nth-last-child(-n+3) { border-bottom:none; }
.rc-label { font-family:'DM Mono',monospace;font-size:9px;letter-spacing:0.15em;text-transform:uppercase;color:var(--muted);margin-bottom:5px; }
.rc-input { width:100%;background:transparent;border:none;border-bottom:1.5px solid transparent;color:var(--ink);font-family:'DM Sans',sans-serif;font-size:13px;padding:2px 0;outline:none;transition:border-color 0.2s; }
.rc-input:focus { border-bottom-color:var(--green2); }
.rc-input::placeholder { color:var(--faint);font-style:italic; }
.rc-notes { padding:10px 16px;border-top:1px solid var(--warm1);background:rgba(45,106,79,0.04); }
.rc-notes textarea { width:100%;background:transparent;border:none;color:var(--muted);font-family:'DM Mono',monospace;font-size:11px;resize:none;outline:none;line-height:1.7;height:48px; }
.rc-actions { padding:10px 16px;border-top:1px solid var(--warm1);display:flex;gap:8px;justify-content:flex-end;background:var(--cream); }

.table-wrap { flex:1;overflow-y:auto;padding:16px 24px 24px; }
.empty { text-align:center;padding:60px 20px;color:var(--faint); }
.empty-icon { font-size:40px;margin-bottom:12px;opacity:0.5; }
.empty p { font-size:14px;color:var(--muted); }
.empty span { font-family:'DM Mono',monospace;font-size:11px;display:block;margin-top:6px; }

table { width:100%;border-collapse:collapse;font-size:13px; }
thead th { font-family:'DM Mono',monospace;font-size:9px;letter-spacing:0.14em;text-transform:uppercase;color:var(--muted);padding:10px 12px;text-align:left;border-bottom:1.5px solid var(--warm2);position:sticky;top:0;background:var(--cream);z-index:1; }
tbody tr { border-bottom:1px solid var(--warm1);transition:background 0.12s; }
tbody tr:hover { background:var(--paper); }
tbody tr:last-child { border-bottom:none; }
td { padding:10px 12px;vertical-align:middle; }

.td-num  { font-family:'DM Mono',monospace;font-size:11px;color:var(--faint); }
.td-name { font-weight:500;color:var(--ink); }
.td-exp-ok   { font-family:'DM Mono',monospace;font-size:12px;color:var(--green2); }
.td-exp-soon { font-family:'DM Mono',monospace;font-size:12px;color:var(--orange);font-weight:600; }
.td-exp-over { font-family:'DM Mono',monospace;font-size:12px;color:var(--red);font-weight:600; }
.td-na  { color:var(--faint);font-style:italic; }
.td-mono { font-family:'DM Mono',monospace;font-size:11px;color:var(--ink2); }
.td-time { font-family:'DM Mono',monospace;font-size:10px;color:var(--faint); }

.exp-tag { display:inline-block;font-size:9px;padding:1px 6px;border-radius:10px;margin-left:6px;font-family:'DM Mono',monospace;letter-spacing:0.06em;vertical-align:middle; }
.tag-soon { background:rgba(202,103,2,0.12);color:var(--orange); }
.tag-over { background:rgba(174,32,18,0.12);color:var(--red); }

.del-btn { background:none;border:none;color:var(--faint);cursor:pointer;padding:4px;border-radius:4px;font-size:13px;transition:color 0.15s; }
.del-btn:hover { color:var(--red); }

.toast { position:fixed;bottom:22px;right:22px;background:var(--ink);color:var(--cream);border-left:3px solid var(--green);border-radius:8px;padding:11px 18px;font-size:13px;font-family:'DM Sans',sans-serif;box-shadow:var(--shadow2);transform:translateY(70px);opacity:0;transition:all 0.3s cubic-bezier(0.34,1.56,0.64,1);z-index:999;pointer-events:none; }
.toast.show { transform:translateY(0);opacity:1; }

::-webkit-scrollbar { width:5px; }
::-webkit-scrollbar-track { background:transparent; }
::-webkit-scrollbar-thumb { background:var(--warm2);border-radius:3px; }
</style>
</head>
<body>

<header>
  <div class="logo">
    <div><span class="logo-word"><em>Fresh</em>Scan</span></div>
    <span class="logo-sub">Food &amp; Bev · Inventory</span>
  </div>
  <div class="header-right">
    <span class="count-pill" id="countPill">0 items</span>
    <button class="btn btn-red btn-sm" onclick="clearAll()">✕ Clear All</button>
  </div>
</header>

<div class="app">

  <!-- LEFT PANEL -->
  <div class="left-panel">

    <div class="section">
      <div class="section-label">Google Cloud Vision API Key</div>
      <div class="key-row">
        <input type="password" id="apiKey" placeholder="AIza…" />
        <button class="btn btn-icon btn-sm" onclick="toggleKey()">👁</button>
      </div>
      <div style="margin-top:7px;font-size:11px;color:var(--muted);font-family:'DM Mono',monospace;line-height:1.6">
        Free: 1,000 req/month &nbsp;·&nbsp;
        <a href="https://console.cloud.google.com" target="_blank" style="color:var(--green2)">Get key ↗</a>
      </div>
    </div>

    <div class="section">
      <div class="section-label">Camera</div>
      <div class="cam-row">
        <select id="camSel" onchange="switchCam()"><option value="">Select camera…</option></select>
        <button class="btn btn-icon btn-sm" onclick="startCam()">⟳</button>
      </div>
      <div class="video-wrapper" id="vwrap">
        <video id="video" autoplay playsinline muted></video>
        <div class="scanner-line"></div>
        <div class="corner-mark cm-tl"></div>
        <div class="corner-mark cm-tr"></div>
        <div class="corner-mark cm-bl"></div>
        <div class="corner-mark cm-br"></div>
        <div class="live-badge"><div class="live-dot"></div>LIVE</div>
        <div class="cam-overlay" id="camOverlay">
          <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="#8a8278" stroke-width="1.5">
            <path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"/>
            <circle cx="12" cy="13" r="4"/>
          </svg>
          <p>Click Start Camera</p>
        </div>
      </div>
    </div>

    <div class="section">
      <div class="section-label">Capture Shots <span id="slotCount" style="color:var(--green2)">0/4</span></div>
      <!-- CONFIG: NUM_CAMERAS at top of script controls slot count -->
      <div class="shots-grid" id="shotsGrid"></div>
      <div class="row" style="margin-bottom:8px">
        <button class="btn btn-outline btn-sm" onclick="startCam()">▶ Start Camera</button>
        <button class="btn btn-ink" onclick="captureShot()" id="capBtn" disabled style="flex:1">
          📷 Capture Shot <span id="capNum">1</span>
        </button>
      </div>
      <button class="btn btn-green btn-full" onclick="analyzeShots()" id="analyzeBtn" disabled>
        🔍 Analyze with AI
      </button>
    </div>

  </div>

  <!-- RIGHT PANEL -->
  <div class="right-panel">

    <div class="right-header">
      <div style="font-family:'DM Serif Display',serif;font-size:18px;color:var(--ink)">Inventory Log</div>
      <div class="export-row">
        <button class="btn btn-outline btn-sm" onclick="exportCSV()">⬇ CSV</button>
        <button class="btn btn-green btn-sm" onclick="exportXLSX()">⬇ Excel</button>
      </div>
    </div>

    <div class="analyzing-bar" id="analyzeBar">
      <div class="spinner"></div>
      <div class="analyzing-text">
        <strong>Reading product labels…</strong>
        Google Cloud Vision OCR · scanning all images in parallel
      </div>
    </div>

    <div class="review-card" id="reviewCard">
      <div class="rc-head">
        <span class="rc-head-title">Review Extracted Data</span>
        <span class="rc-head-sub">Edit any field before saving</span>
      </div>
      <div class="rc-fields">
        <div class="rc-field" style="grid-column:1/3">
          <div class="rc-label">Product / Sample Name</div>
          <input class="rc-input" id="rf-name" placeholder="Unknown product" />
        </div>
        <div class="rc-field">
          <div class="rc-label">Brand</div>
          <input class="rc-input" id="rf-brand" placeholder="Not found" />
        </div>
        <div class="rc-field">
          <div class="rc-label">Expiration Date</div>
          <input class="rc-input" id="rf-exp" placeholder="Not found" />
        </div>
        <div class="rc-field">
          <div class="rc-label">Barcode / UPC / SKU</div>
          <input class="rc-input" id="rf-sku" placeholder="Not found" />
        </div>
        <div class="rc-field">
          <div class="rc-label">Net Weight / Volume</div>
          <input class="rc-input" id="rf-size" placeholder="Not found" />
        </div>
      </div>
      <div class="rc-notes">
        <div class="rc-label" style="margin-bottom:4px">Additional Notes (AI)</div>
        <textarea id="rf-notes" readonly></textarea>
      </div>
      <div class="rc-actions">
        <button class="btn btn-outline btn-sm" onclick="dismissReview()">✕ Discard</button>
        <button class="btn btn-green btn-sm" onclick="saveEntry()">✓ Save to Inventory</button>
      </div>
    </div>

    <div class="table-wrap">
      <div class="empty" id="emptyState">
        <div class="empty-icon">🥫</div>
        <p>No samples logged yet</p>
        <span>Capture → Analyze → Save</span>
      </div>
      <table id="invTable" style="display:none">
        <thead>
          <tr>
            <th>#</th><th>Product Name</th><th>Brand</th>
            <th>Expiration</th><th>Barcode / SKU</th>
            <th>Size</th><th>Logged At</th><th></th>
          </tr>
        </thead>
        <tbody id="invBody"></tbody>
      </table>
    </div>

  </div>
</div>

<div class="toast" id="toast"></div>
<canvas id="canvas"></canvas>

<script>
// ═══════════════════════════════════════════════
// CONFIG — adjust when adding physical cameras
// ═══════════════════════════════════════════════
const NUM_CAMERAS = 2;
const SLOT_LABELS = ['Front', 'Back'];
// Multi-cam future: const CAM_MAP = { 0:'deviceId-1', 1:'deviceId-2', ... }
// ═══════════════════════════════════════════════

let stream     = null;
let shots      = Array(NUM_CAMERAS).fill(null);
let activeSlot = 0;
let inventory  = [];

function buildGrid() {
  const g = document.getElementById('shotsGrid');
  g.innerHTML = '';
  for (let i = 0; i < NUM_CAMERAS; i++) {
    const d = document.createElement('div');
    d.className = 'slot' + (i === 0 ? ' active' : '');
    d.id = `slot-${i}`;
    d.onclick = () => setSlot(i);
    d.innerHTML = `<div class="slot-ph">+</div>
      <div class="slot-lbl">${SLOT_LABELS[i]||'Shot '+(i+1)}</div>
      <button class="slot-rm" onclick="rmShot(event,${i})">✕</button>`;
    g.appendChild(d);
  }
}

function setSlot(i) {
  activeSlot = i;
  document.querySelectorAll('.slot').forEach((s,idx) => s.classList.toggle('active', idx===i));
  document.getElementById('capNum').textContent = i+1;
}

function rmShot(e, i) {
  e.stopPropagation();
  shots[i] = null;
  const s = document.getElementById(`slot-${i}`);
  s.classList.remove('filled');
  s.innerHTML = `<div class="slot-ph">+</div>
    <div class="slot-lbl">${SLOT_LABELS[i]||'Shot '+(i+1)}</div>
    <button class="slot-rm" onclick="rmShot(event,${i})">✕</button>`;
  updUI();
}

function updUI() {
  const n = shots.filter(Boolean).length;
  document.getElementById('slotCount').textContent = `${n}/${NUM_CAMERAS}`;
  document.getElementById('analyzeBtn').disabled = n === 0;
}

async function requestPermission() {
  // Explicitly request permission with minimal constraints first.
  // This is the step Android requires before enumerateDevices returns labels.
  try {
    const probe = await navigator.mediaDevices.getUserMedia({ video: true, audio: false });
    probe.getTracks().forEach(t => t.stop()); // release immediately — just needed the permission grant
    return true;
  } catch(e) {
    const msg = e.name === 'NotAllowedError'
      ? 'Camera permission denied. Please allow camera access in your browser settings and try again.'
      : e.name === 'NotFoundError'
      ? 'No camera found on this device.'
      : 'Camera error: ' + e.message;
    toast(msg, true);
    return false;
  }
}

async function startCam() {
  // Stop any existing stream
  if (stream) { stream.getTracks().forEach(t => t.stop()); stream = null; }

  // Request permission first if we don't already have a stream
  const granted = await requestPermission();
  if (!granted) return;

  // Now enumerate so we get real device labels on Android
  await fillCams();

  const sel = document.getElementById('camSel').value;

  // Build constraints — prefer back camera on mobile when no specific device selected
  let videoConstraints;
  if (sel) {
    videoConstraints = { deviceId: { exact: sel } };
  } else {
    videoConstraints = {
      facingMode: { ideal: 'environment' }, // back camera preferred, not required
      width:  { ideal: 1280 },
      height: { ideal: 720  }
    };
  }

  try {
    stream = await navigator.mediaDevices.getUserMedia({ video: videoConstraints, audio: false });
    const video = document.getElementById('video');
    video.srcObject = stream;

    // On Android, play() may need to be called explicitly
    try { await video.play(); } catch(_) {}

    document.getElementById('camOverlay').style.display = 'none';
    document.getElementById('vwrap').classList.add('live');
    document.getElementById('capBtn').disabled = false;

    // Refresh cam list now that we have permission and labels
    await fillCams();

    // Show which camera is active in the selector
    const activeTrack = stream.getVideoTracks()[0];
    if (activeTrack) {
      const settings = activeTrack.getSettings();
      const opts = document.getElementById('camSel').options;
      for (let i = 0; i < opts.length; i++) {
        if (opts[i].value === settings.deviceId) {
          document.getElementById('camSel').selectedIndex = i;
          break;
        }
      }
    }

    toast('Camera ready ✓');
  } catch(e) {
    const msg = e.name === 'NotAllowedError'
      ? 'Permission denied — please allow camera access.'
      : e.name === 'NotReadableError'
      ? 'Camera is in use by another app.'
      : e.name === 'OverconstrainedError'
      ? 'Camera constraint not supported — try selecting a different camera.'
      : 'Camera error: ' + e.message;
    toast(msg, true);
  }
}

async function fillCams() {
  try {
    const devs = await navigator.mediaDevices.enumerateDevices();
    const cams = devs.filter(d => d.kind === 'videoinput');
    const sel  = document.getElementById('camSel');
    const cur  = sel.value;
    sel.innerHTML = cams.length
      ? cams.map((c, i) =>
          `<option value="${c.deviceId}" ${c.deviceId===cur?'selected':''}>${c.label || 'Camera '+(i+1)}</option>`
        ).join('')
      : '<option value="">No cameras found</option>';
  } catch(e) { /* enumerateDevices can fail silently before permission */ }
}

async function switchCam() { if (stream) await startCam(); }

function captureShot() {
  if (!stream) { toast('Start camera first', true); return; }
  const v = document.getElementById('video');
  const c = document.getElementById('canvas');
  c.width = v.videoWidth||640; c.height = v.videoHeight||480;
  c.getContext('2d').drawImage(v,0,0);
  const url = c.toDataURL('image/jpeg', 0.88);
  shots[activeSlot] = url;
  const s = document.getElementById(`slot-${activeSlot}`);
  s.classList.add('filled');
  s.innerHTML = `<img src="${url}" alt="Shot ${activeSlot+1}" />
    <div class="slot-lbl">${SLOT_LABELS[activeSlot]||'Shot '+(activeSlot+1)}</div>
    <button class="slot-rm" onclick="rmShot(event,${activeSlot})">✕</button>`;
  let nxt=-1;
  for(let i=activeSlot+1;i<NUM_CAMERAS;i++){if(!shots[i]){nxt=i;break;}}
  if(nxt===-1)for(let i=0;i<activeSlot;i++){if(!shots[i]){nxt=i;break;}}
  if(nxt!==-1)setSlot(nxt);
  updUI();
  toast(`"${SLOT_LABELS[activeSlot]}" captured`);
}

// ── Google Vision OCR: send one image, return full text string ──
async function visionOCR(apiKey, base64img) {
  const body = {
    requests: [{
      image: { content: base64img },
      features: [
        { type: 'TEXT_DETECTION',     maxResults: 1 },
        { type: 'BARCODE_DETECTION',  maxResults: 5 }
      ]
    }]
  };
  const resp = await fetch(
    `https://vision.googleapis.com/v1/images:annotate?key=${apiKey}`,
    { method:'POST', headers:{'Content-Type':'application/json'}, body:JSON.stringify(body) }
  );
  if (!resp.ok) {
    const e = await resp.json();
    throw new Error(e.error?.message || 'Google Vision API error');
  }
  const data = await resp.json();
  const result = data.responses[0];

  // Full text block
  const fullText = result.fullTextAnnotation?.text
    || result.textAnnotations?.[0]?.description
    || '';

  // Barcodes detected natively by Vision
  const barcodes = (result.barcodeAnnotations || [])
    .map(b => b.rawValue).filter(Boolean);

  return { fullText, barcodes };
}

// ── Smart parser: extract structured fields from raw OCR text ──
function parseLabel(allText, allBarcodes) {
  const lines = allText.split(/\n+/).map(l => l.trim()).filter(Boolean);
  const text  = allText;

  // ── Expiration date ──
  // Matches: EXP 01/2026  |  Best By: Mar 15 2025  |  BB 2025-06-30  |  Use By 06/15/25
  const expPatterns = [
    /(?:exp(?:iry|ires?|[. :])|best[ -]?by[: ]|use[ -]?by[: ]|sell[ -]?by[: ]|bb[: ]|bbd[: ])\s*([A-Za-z0-9\/\-\. ]{4,20})/i,
    /\b((?:jan|feb|mar|apr|may|jun|jul|aug|sep|oct|nov|dec)[a-z]*\.?\s+\d{1,2}[,\s]+\d{2,4})\b/i,
    /\b(\d{1,2}[\/\-]\d{1,2}[\/\-]\d{2,4})\b/,
    /\b(\d{4}[\/\-]\d{2}[\/\-]\d{2})\b/,
    /\b(\d{2}[\/\-]\d{4})\b/   // MM/YYYY
  ];
  let expiration = null;
  for (const pat of expPatterns) {
    const m = text.match(pat);
    if (m) { expiration = (m[1]||m[0]).trim(); break; }
  }

  // ── Barcode / SKU ──
  // Prefer Vision-detected barcodes, then look for UPC/EAN patterns in text
  let sku = allBarcodes.length ? allBarcodes[0] : null;
  if (!sku) {
    const skuPat = /\b(\d{8,14})\b/g;
    const skuMatches = [...text.matchAll(skuPat)].map(m => m[1]);
    // Prefer 12-digit UPC-A or 13-digit EAN-13
    sku = skuMatches.find(s => s.length===12||s.length===13)
       || skuMatches.find(s => s.length>=8)
       || null;
  }

  // ── Net weight / volume / size ──
  const sizePat = /\b(\d+\.?\d*\s*(?:fl\.?\s*oz|oz|lb|lbs|g|kg|ml|l|liters?|ounces?|pounds?|ct|count|pcs|pieces?|pack))\b/i;
  const sizeMatch = text.match(sizePat);
  const size = sizeMatch ? sizeMatch[1].replace(/\s+/g,' ').trim() : null;

  // ── Brand & name heuristic ──
  // Strategy: first 1-3 non-trivial lines are usually brand + product name.
  // Filter out lines that look like addresses, URLs, barcodes, weights, or dates.
  const junkPattern = /^[\d\s\-\/\.\,]+$|www\.|\.com|@|\bllc\b|\binc\b|\bcorp\b|tel:|fax:|distributed|manufactured|contains|ingredients|nutrition|serving|calories|total|sodium|sugar|protein|fat|carb|\bpo box\b/i;
  const candidateLines = lines.filter(l =>
    l.length > 2 &&
    l.length < 60 &&
    !junkPattern.test(l) &&
    !/^\d{6,}$/.test(l)     // not a pure long number
  );

  // The largest-font text is usually first in Vision results (it sorts by position, top-to-bottom)
  const brand = candidateLines[0] || null;
  const name  = candidateLines.slice(0,3).join(' ') || null;

  // ── Notes: allergens + flavor ──
  const allergenPat = /contains?:?\s*([^\.\n]{5,80})/i;
  const flavorPat   = /\b(original|classic|light|diet|zero|sugar[ -]free|organic|natural|low[ -]fat|whole grain|gluten[ -]free|vegan|vitamin[- ][a-z])\b/gi;
  const allergenMatch = text.match(allergenPat);
  const flavorMatches = [...new Set((text.match(flavorPat)||[]).map(s=>s.toLowerCase()))];
  let notes = '';
  if (allergenMatch) notes += 'Contains: ' + allergenMatch[1].trim().slice(0,80);
  if (flavorMatches.length) notes += (notes?' | ':'') + flavorMatches.slice(0,4).join(', ');
  notes = notes.slice(0,180) || null;

  return { brand, name, expiration, sku, size, notes };
}

async function analyzeShots() {
  const key = document.getElementById('apiKey').value.trim();
  if (!key) { toast('Enter your Google Vision API key first', true); return; }
  const imgs = shots.filter(Boolean);
  if (!imgs.length) { toast('Capture at least one shot', true); return; }

  document.getElementById('analyzeBar').classList.add('on');
  document.getElementById('reviewCard').classList.remove('on');
  document.getElementById('analyzeBtn').disabled = true;

  try {
    // Send all captured shots to Vision in parallel (each costs 1 request unit)
    const results = await Promise.all(
      imgs.map(img => visionOCR(key, img.split(',')[1]))
    );

    // Merge all OCR text and barcodes across images
    const allText     = results.map(r => r.fullText).join('\n\n');
    const allBarcodes = [...new Set(results.flatMap(r => r.barcodes))];

    // Parse structured fields from combined text
    const parsed = parseLabel(allText, allBarcodes);

    document.getElementById('rf-name').value  = parsed.name  || '';
    document.getElementById('rf-brand').value = parsed.brand || '';
    document.getElementById('rf-exp').value   = parsed.expiration || '';
    document.getElementById('rf-sku').value   = parsed.sku   || '';
    document.getElementById('rf-size').value  = parsed.size  || '';
    document.getElementById('rf-notes').value = parsed.notes || '';

    document.getElementById('analyzeBar').classList.remove('on');
    document.getElementById('reviewCard').classList.add('on');

    // Friendly hint if very little was found
    const found = [parsed.name, parsed.expiration, parsed.sku].filter(Boolean).length;
    if (found === 0) toast('Low confidence — try better lighting or a closer shot', true);
    else toast(`Extracted from ${imgs.length} image${imgs.length>1?'s':''} ✓`);

  } catch(e) {
    document.getElementById('analyzeBar').classList.remove('on');
    document.getElementById('analyzeBtn').disabled = false;
    toast('Error: ' + e.message, true);
  }
}

function saveEntry() {
  const entry = {
    id:         Date.now(),
    name:       document.getElementById('rf-name').value.trim()||'—',
    brand:      document.getElementById('rf-brand').value.trim()||null,
    expiration: document.getElementById('rf-exp').value.trim()||null,
    sku:        document.getElementById('rf-sku').value.trim()||null,
    size:       document.getElementById('rf-size').value.trim()||null,
    notes:      document.getElementById('rf-notes').value.trim()||null,
    timestamp:  new Date().toLocaleString()
  };
  inventory.push(entry);
  renderTable();
  dismissReview();
  resetShots();
  document.getElementById('countPill').textContent = `${inventory.length} item${inventory.length!==1?'s':''}`;
  toast('Saved ✓');
}

function dismissReview() {
  document.getElementById('reviewCard').classList.remove('on');
  document.getElementById('analyzeBtn').disabled = shots.filter(Boolean).length===0;
}

function resetShots() {
  shots = Array(NUM_CAMERAS).fill(null);
  buildGrid(); setSlot(0); updUI();
}

function expClass(raw) {
  if (!raw) return 'td-na';
  const d = new Date(raw);
  if (isNaN(d)) return 'td-exp-ok';
  const diff = (d-Date.now())/86400000;
  if (diff<0) return 'td-exp-over';
  if (diff<30) return 'td-exp-soon';
  return 'td-exp-ok';
}
function expTag(raw) {
  if (!raw) return '';
  const d = new Date(raw); if (isNaN(d)) return '';
  const diff = (d-Date.now())/86400000;
  if (diff<0) return `<span class="exp-tag tag-over">EXPIRED</span>`;
  if (diff<30) return `<span class="exp-tag tag-soon">SOON</span>`;
  return '';
}

function renderTable() {
  const body=document.getElementById('invBody');
  const tbl =document.getElementById('invTable');
  const emp =document.getElementById('emptyState');
  if(!inventory.length){tbl.style.display='none';emp.style.display='block';return;}
  tbl.style.display='table'; emp.style.display='none';
  body.innerHTML=inventory.map((e,i)=>`
    <tr>
      <td class="td-num">${i+1}</td>
      <td class="td-name">${e.name}</td>
      <td class="${e.brand?'td-mono':'td-na'}">${e.brand||'—'}</td>
      <td class="${expClass(e.expiration)}">${e.expiration||'—'}${expTag(e.expiration)}</td>
      <td class="${e.sku?'td-mono':'td-na'}">${e.sku||'—'}</td>
      <td class="${e.size?'td-mono':'td-na'}">${e.size||'—'}</td>
      <td class="td-time">${e.timestamp}</td>
      <td><button class="del-btn" onclick="deleteEntry(${e.id})">✕</button></td>
    </tr>`).join('');
}

function deleteEntry(id) {
  inventory=inventory.filter(e=>e.id!==id);
  renderTable();
  document.getElementById('countPill').textContent=`${inventory.length} item${inventory.length!==1?'s':''}`;
}

function clearAll() {
  if(!inventory.length) return;
  if(!confirm(`Delete all ${inventory.length} entries?`)) return;
  inventory=[]; renderTable();
  document.getElementById('countPill').textContent='0 items';
}

function buildRows() {
  return inventory.map((e,i)=>({
    '#':i+1,'Product Name':e.name,'Brand':e.brand||'',
    'Expiration':e.expiration||'','Barcode/SKU':e.sku||'',
    'Size':e.size||'','Notes':e.notes||'','Logged At':e.timestamp
  }));
}

function exportCSV() {
  if(!inventory.length){toast('Nothing to export',true);return;}
  const rows=buildRows(); const hdrs=Object.keys(rows[0]);
  const csv=[hdrs,...rows.map(r=>hdrs.map(h=>`"${String(r[h]).replace(/"/g,'""')}"`))].map(r=>r.join(',')).join('\n');
  const a=document.createElement('a');
  a.href=URL.createObjectURL(new Blob([csv],{type:'text/csv'}));
  a.download=`inventory-${today()}.csv`; a.click();
  toast(`Exported ${inventory.length} rows as CSV`);
}

function exportXLSX() {
  if(!inventory.length){toast('Nothing to export',true);return;}
  const ws=XLSX.utils.json_to_sheet(buildRows());
  ws['!cols']=[{wch:4},{wch:28},{wch:18},{wch:16},{wch:16},{wch:10},{wch:40},{wch:20}];
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,'Inventory');
  XLSX.writeFile(wb,`inventory-${today()}.xlsx`);
  toast(`Exported ${inventory.length} rows as Excel`);
}

function today(){return new Date().toISOString().slice(0,10);}

function toggleKey(){const i=document.getElementById('apiKey');i.type=i.type==='password'?'text':'password';}

let toastT;
function toast(msg,err=false){
  const t=document.getElementById('toast');
  t.textContent=msg; t.style.borderLeftColor=err?'var(--red)':'var(--green)';
  t.classList.add('show'); clearTimeout(toastT);
  toastT=setTimeout(()=>t.classList.remove('show'),2800);
}

buildGrid();
// Don't enumerate cameras on load — Android requires permission first.
// Camera list populates automatically when the user clicks Start Camera.
</script>
</body>
</html>
