<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>FreshScan — Food & Beverage Inventory</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap');

:root {
  --cream:#faf7f2;--paper:#f2ede4;--warm1:#e8dfd0;--warm2:#d4c9b5;
  --ink:#1c1a17;--ink2:#3d3a35;--muted:#8a8278;--faint:#b8b0a4;
  --green:#2d6a4f;--green2:#40916c;--green3:#74c69d;
  --red:#ae2012;--orange:#ca6702;--blue:#1a6fa8;
  --radius:10px;--shadow:0 2px 12px rgba(28,26,23,.1);--shadow2:0 8px 32px rgba(28,26,23,.15);
}
*{box-sizing:border-box;margin:0;padding:0}
body{background:var(--cream);color:var(--ink);font-family:'DM Sans',sans-serif;min-height:100vh;display:flex;flex-direction:column}
body::before{content:'';position:fixed;inset:0;background-image:url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E");pointer-events:none;z-index:0;opacity:.6}

header{position:relative;z-index:1;display:flex;align-items:center;justify-content:space-between;padding:16px 28px;background:var(--ink);border-bottom:3px solid var(--green)}
.logo{display:flex;align-items:baseline;gap:10px}
.logo-word{font-family:'DM Serif Display',serif;font-size:22px;color:var(--cream);letter-spacing:-.01em}
.logo-word em{color:var(--green3);font-style:italic}
.logo-sub{font-size:10px;font-family:'DM Mono',monospace;letter-spacing:.15em;text-transform:uppercase;color:var(--faint)}
.header-right{display:flex;gap:10px;align-items:center}
.count-pill{background:var(--green);color:var(--cream);font-family:'DM Mono',monospace;font-size:11px;padding:4px 12px;border-radius:20px;letter-spacing:.05em}

.app{position:relative;z-index:1;display:grid;grid-template-columns:420px 1fr;flex:1;height:calc(100vh - 61px);overflow:hidden}

/* LEFT */
.left-panel{background:var(--paper);border-right:1px solid var(--warm2);display:flex;flex-direction:column;overflow-y:auto}
.section{padding:16px 20px;border-bottom:1px solid var(--warm1)}
.section-label{font-family:'DM Mono',monospace;font-size:9px;text-transform:uppercase;letter-spacing:.18em;color:var(--muted);margin-bottom:10px}
.key-row{display:flex;gap:8px}
input[type=password],input[type=text],select{flex:1;background:var(--cream);border:1.5px solid var(--warm2);border-radius:7px;padding:8px 12px;font-family:'DM Mono',monospace;font-size:12px;color:var(--ink);outline:none;transition:border-color .2s}
input:focus,select:focus{border-color:var(--green2)}

.video-wrapper{position:relative;border-radius:var(--radius);overflow:hidden;background:var(--ink);aspect-ratio:4/3;box-shadow:var(--shadow)}
video{width:100%;height:100%;object-fit:cover;display:block}
.cam-overlay{position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:10px;background:rgba(28,26,23,.82)}
.cam-overlay p{font-size:12px;color:var(--faint);text-align:center}
.scanner-line{position:absolute;left:0;right:0;height:2px;background:linear-gradient(90deg,transparent,var(--green3),transparent);animation:scan 3s ease-in-out infinite;display:none;box-shadow:0 0 8px var(--green3)}
.live .scanner-line{display:block}
@keyframes scan{0%{top:10%;opacity:0}10%{opacity:1}90%{opacity:1}100%{top:90%;opacity:0}}
.corner-mark{position:absolute;width:18px;height:18px;border-color:var(--green3);border-style:solid;pointer-events:none;opacity:0;transition:opacity .4s}
.live .corner-mark{opacity:1}
.cm-tl{top:8px;left:8px;border-width:2px 0 0 2px}.cm-tr{top:8px;right:8px;border-width:2px 2px 0 0}
.cm-bl{bottom:8px;left:8px;border-width:0 0 2px 2px}.cm-br{bottom:8px;right:8px;border-width:0 2px 2px 0}
.live-badge{position:absolute;top:8px;left:50%;transform:translateX(-50%);display:none;align-items:center;gap:5px;background:rgba(28,26,23,.75);padding:3px 10px;border-radius:20px;font-family:'DM Mono',monospace;font-size:9px;letter-spacing:.12em;color:var(--green3)}
.live .live-badge{display:flex}
.live-dot{width:5px;height:5px;border-radius:50%;background:var(--green3);animation:blink 1.2s infinite}
@keyframes blink{0%,100%{opacity:1}50%{opacity:.2}}
.cam-row{display:flex;gap:8px;margin-bottom:10px}
.cam-row select{font-size:11px}

/* Shot slots */
.shots-grid{display:grid;grid-template-columns:repeat(2,1fr);gap:8px;margin-bottom:12px}
.slot{aspect-ratio:4/3;border-radius:7px;border:1.5px dashed var(--warm2);background:var(--cream);position:relative;overflow:hidden;cursor:pointer;transition:border-color .2s,box-shadow .2s}
.slot:hover{border-color:var(--green2)}
.slot.filled{border-style:solid;border-color:var(--green)}
.slot.active{border-color:var(--orange);border-style:solid;box-shadow:0 0 0 3px rgba(202,103,2,.15)}
.slot img{width:100%;height:100%;object-fit:cover}
.slot-lbl{position:absolute;bottom:0;left:0;right:0;font-family:'DM Mono',monospace;font-size:8px;letter-spacing:.08em;text-align:center;padding:2px 3px;background:rgba(28,26,23,.65);color:#fff}
.slot.active .slot-lbl{background:rgba(202,103,2,.85)}
.slot-ph{display:flex;align-items:center;justify-content:center;height:100%;color:var(--warm2);font-size:20px}
.slot-rm{position:absolute;top:3px;right:3px;width:15px;height:15px;border-radius:50%;background:var(--red);color:#fff;border:none;cursor:pointer;font-size:8px;display:none;align-items:center;justify-content:center}
.slot.filled:hover .slot-rm{display:flex}
.slot-roi-btn{position:absolute;top:3px;left:3px;width:18px;height:18px;border-radius:4px;background:rgba(26,111,168,.85);color:#fff;border:none;cursor:pointer;font-size:9px;display:none;align-items:center;justify-content:center;title:'Draw ROI'}
.slot.filled .slot-roi-btn{display:flex}
.roi-count{position:absolute;top:3px;left:24px;background:rgba(26,111,168,.85);color:#fff;font-family:'DM Mono',monospace;font-size:8px;padding:2px 5px;border-radius:4px;display:none}
.slot.filled .roi-count{display:block}

/* Buttons */
.btn{display:inline-flex;align-items:center;justify-content:center;gap:6px;padding:9px 16px;border-radius:8px;border:none;font-family:'DM Sans',sans-serif;font-size:13px;font-weight:500;cursor:pointer;transition:all .18s;white-space:nowrap}
.btn:disabled{opacity:.38;cursor:not-allowed}
.btn-green{background:var(--green);color:#fff}.btn-green:hover:not(:disabled){background:var(--green2)}
.btn-outline{background:transparent;border:1.5px solid var(--warm2);color:var(--ink2)}.btn-outline:hover:not(:disabled){border-color:var(--green2);color:var(--green)}
.btn-ink{background:var(--ink);color:var(--cream)}.btn-ink:hover:not(:disabled){background:var(--ink2)}
.btn-blue{background:var(--blue);color:#fff}.btn-blue:hover:not(:disabled){background:#1e82c8}
.btn-red{background:var(--red);color:#fff}.btn-red:hover:not(:disabled){background:#c1432e}
.btn-sm{padding:6px 12px;font-size:12px}
.btn-full{width:100%}
.btn-icon{background:var(--cream);border:1.5px solid var(--warm2);color:var(--muted);padding:7px 10px}
.btn-icon:hover:not(:disabled){border-color:var(--green2);color:var(--green)}
.row{display:flex;gap:8px}

canvas{display:none}

/* ── ROI Modal ── */
.roi-modal{position:fixed;inset:0;z-index:100;background:rgba(28,26,23,.92);display:none;flex-direction:column;align-items:center;justify-content:center;gap:12px;padding:20px}
.roi-modal.open{display:flex}
.roi-modal-title{color:var(--cream);font-family:'DM Serif Display',serif;font-size:18px}
.roi-modal-sub{color:var(--faint);font-size:12px;font-family:'DM Mono',monospace;text-align:center;line-height:1.6}
.roi-canvas-wrap{position:relative;max-width:90vw;max-height:65vh;border:2px solid var(--green);border-radius:var(--radius);overflow:hidden;cursor:crosshair}
#roiCanvas{display:block;max-width:100%;max-height:65vh;object-fit:contain}
.roi-toolbar{display:flex;gap:10px;flex-wrap:wrap;justify-content:center}
.roi-field-select{background:var(--ink2);border:1.5px solid var(--green);border-radius:7px;padding:7px 12px;color:var(--cream);font-family:'DM Mono',monospace;font-size:12px;outline:none;cursor:pointer}
.roi-legend{display:flex;gap:10px;flex-wrap:wrap;justify-content:center}
.roi-legend-item{display:flex;align-items:center;gap:5px;font-size:11px;font-family:'DM Mono',monospace;color:var(--faint)}
.roi-legend-swatch{width:12px;height:12px;border-radius:2px;border:1.5px solid rgba(255,255,255,.3)}

/* RIGHT */
.right-panel{display:flex;flex-direction:column;overflow:hidden;background:var(--cream)}
.right-header{padding:14px 24px;border-bottom:1px solid var(--warm1);display:flex;align-items:center;justify-content:space-between;background:var(--paper);flex-wrap:wrap;gap:10px}
.export-row{display:flex;gap:8px}

.analyzing-bar{margin:14px 24px 0;background:var(--paper);border:1.5px solid var(--green3);border-radius:var(--radius);padding:14px 18px;display:none;align-items:center;gap:14px}
.analyzing-bar.on{display:flex}
.spinner{width:26px;height:26px;border:2.5px solid var(--warm1);border-top-color:var(--green);border-radius:50%;animation:spin .75s linear infinite;flex-shrink:0}
@keyframes spin{to{transform:rotate(360deg)}}
.analyzing-text{font-size:13px;color:var(--ink2)}
.analyzing-text strong{display:block;color:var(--green);font-size:14px;margin-bottom:2px}

.review-card{margin:14px 24px 0;background:var(--paper);border:1.5px solid var(--green);border-radius:var(--radius);overflow:hidden;display:none;box-shadow:var(--shadow)}
.review-card.on{display:block}
.rc-head{padding:10px 16px;background:var(--green);display:flex;align-items:center;justify-content:space-between}
.rc-head-title{font-family:'DM Serif Display',serif;font-size:15px;color:#fff;letter-spacing:.01em}
.rc-head-right{display:flex;align-items:center;gap:12px}
.rc-head-sub{font-size:11px;color:rgba(255,255,255,.7);font-family:'DM Mono',monospace}
.sample-id-badge{background:rgba(255,255,255,.15);border:1px solid rgba(255,255,255,.3);border-radius:6px;padding:4px 10px;font-family:'DM Mono',monospace;font-size:14px;font-weight:500;color:#fff;letter-spacing:.1em}
.rc-fields{display:grid;grid-template-columns:repeat(3,1fr)}
.rc-field{padding:12px 16px;border-bottom:1px solid var(--warm1);border-right:1px solid var(--warm1)}
.rc-field:nth-child(3n){border-right:none}
.rc-field:nth-last-child(-n+3){border-bottom:none}
.rc-label{font-family:'DM Mono',monospace;font-size:9px;letter-spacing:.15em;text-transform:uppercase;color:var(--muted);margin-bottom:5px}
.rc-input{width:100%;background:transparent;border:none;border-bottom:1.5px solid transparent;color:var(--ink);font-family:'DM Sans',sans-serif;font-size:13px;padding:2px 0;outline:none;transition:border-color .2s}
.rc-input:focus{border-bottom-color:var(--green2)}
.rc-input::placeholder{color:var(--faint);font-style:italic}
.rc-notes{padding:10px 16px;border-top:1px solid var(--warm1);background:rgba(45,106,79,.04)}
.rc-notes textarea{width:100%;background:transparent;border:none;color:var(--muted);font-family:'DM Mono',monospace;font-size:11px;resize:none;outline:none;line-height:1.7;height:48px}
.rc-actions{padding:10px 16px;border-top:1px solid var(--warm1);display:flex;gap:8px;justify-content:flex-end;background:var(--cream)}

/* Table */
.table-wrap{flex:1;overflow-y:auto;padding:16px 24px 24px}
.empty{text-align:center;padding:60px 20px;color:var(--faint)}
.empty-icon{font-size:40px;margin-bottom:12px;opacity:.5}
.empty p{font-size:14px;color:var(--muted)}
.empty span{font-family:'DM Mono',monospace;font-size:11px;display:block;margin-top:6px}
table{width:100%;border-collapse:collapse;font-size:13px}
thead th{font-family:'DM Mono',monospace;font-size:9px;letter-spacing:.14em;text-transform:uppercase;color:var(--muted);padding:10px 12px;text-align:left;border-bottom:1.5px solid var(--warm2);position:sticky;top:0;background:var(--cream);z-index:1}
tbody tr{border-bottom:1px solid var(--warm1);transition:background .12s}
tbody tr:hover{background:var(--paper)}
tbody tr:last-child{border-bottom:none}
td{padding:10px 12px;vertical-align:middle}
.td-id{font-family:'DM Mono',monospace;font-size:13px;font-weight:600;color:var(--blue);letter-spacing:.08em}
.td-num{font-family:'DM Mono',monospace;font-size:11px;color:var(--faint)}
.td-name{font-weight:500;color:var(--ink)}
.td-exp-ok{font-family:'DM Mono',monospace;font-size:12px;color:var(--green2)}
.td-exp-soon{font-family:'DM Mono',monospace;font-size:12px;color:var(--orange);font-weight:600}
.td-exp-over{font-family:'DM Mono',monospace;font-size:12px;color:var(--red);font-weight:600}
.td-na{color:var(--faint);font-style:italic}
.td-mono{font-family:'DM Mono',monospace;font-size:11px;color:var(--ink2)}
.td-time{font-family:'DM Mono',monospace;font-size:10px;color:var(--faint)}
.exp-tag{display:inline-block;font-size:9px;padding:1px 6px;border-radius:10px;margin-left:6px;font-family:'DM Mono',monospace;letter-spacing:.06em;vertical-align:middle}
.tag-soon{background:rgba(202,103,2,.12);color:var(--orange)}
.tag-over{background:rgba(174,32,18,.12);color:var(--red)}
.del-btn{background:none;border:none;color:var(--faint);cursor:pointer;padding:4px;border-radius:4px;font-size:13px;transition:color .15s}
.del-btn:hover{color:var(--red)}

/* Search bar */
.search-bar{padding:10px 24px;border-bottom:1px solid var(--warm1);display:flex;gap:8px}
.search-bar input{flex:1;background:var(--paper);border:1.5px solid var(--warm2);border-radius:7px;padding:7px 12px;font-family:'DM Mono',monospace;font-size:12px;color:var(--ink);outline:none}
.search-bar input:focus{border-color:var(--green2)}

.toast{position:fixed;bottom:22px;right:22px;background:var(--ink);color:var(--cream);border-left:3px solid var(--green);border-radius:8px;padding:11px 18px;font-size:13px;font-family:'DM Sans',sans-serif;box-shadow:var(--shadow2);transform:translateY(70px);opacity:0;transition:all .3s cubic-bezier(.34,1.56,.64,1);z-index:999;pointer-events:none}
.toast.show{transform:translateY(0);opacity:1}

::-webkit-scrollbar{width:5px}
::-webkit-scrollbar-track{background:transparent}
::-webkit-scrollbar-thumb{background:var(--warm2);border-radius:3px}
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
  <!-- ═══ LEFT ═══ -->
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
        <div class="corner-mark cm-tl"></div><div class="corner-mark cm-tr"></div>
        <div class="corner-mark cm-bl"></div><div class="corner-mark cm-br"></div>
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
      <div class="section-label">
        Capture Shots
        <span id="slotCount" style="color:var(--green2)">0/2</span>
        &nbsp;·&nbsp;
        <span style="color:var(--blue);font-size:9px">✎ = draw ROI on captured shot</span>
      </div>
      <!-- CONFIG: NUM_CAMERAS controls slot count -->
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

  <!-- ═══ RIGHT ═══ -->
  <div class="right-panel">

    <div class="right-header">
      <div style="font-family:'DM Serif Display',serif;font-size:18px;color:var(--ink)">Inventory Log</div>
      <div class="export-row">
        <button class="btn btn-outline btn-sm" onclick="exportCSV()">⬇ CSV</button>
        <button class="btn btn-green btn-sm" onclick="exportXLSX()">⬇ Excel</button>
      </div>
    </div>

    <div class="search-bar">
      <input type="text" id="searchBox" placeholder="Search by ID, name, brand, barcode…" oninput="renderTable()" />
    </div>

    <div class="analyzing-bar" id="analyzeBar">
      <div class="spinner"></div>
      <div class="analyzing-text">
        <strong>Reading product labels…</strong>
        Google Cloud Vision OCR · scanning images in parallel
      </div>
    </div>

    <div class="review-card" id="reviewCard">
      <div class="rc-head">
        <span class="rc-head-title">Review Extracted Data</span>
        <div class="rc-head-right">
          <span class="rc-head-sub">Edit before saving</span>
          <span class="sample-id-badge" id="sampleIdBadge">––––</span>
        </div>
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
            <th>ID</th><th>#</th><th>Product Name</th><th>Brand</th>
            <th>Expiration</th><th>Barcode / SKU</th>
            <th>Size</th><th>Logged At</th><th></th>
          </tr>
        </thead>
        <tbody id="invBody"></tbody>
      </table>
    </div>

  </div>
</div>

<!-- ROI Modal -->
<div class="roi-modal" id="roiModal">
  <div class="roi-modal-title">Draw OCR Regions</div>
  <div class="roi-modal-sub">
    Select a field type, then drag a box over where that info appears on the label.<br>
    Google Vision will only read text inside your drawn boxes.
  </div>
  <div class="roi-toolbar">
    <select class="roi-field-select" id="roiField">
      <option value="name">Product Name</option>
      <option value="exp">Expiration Date</option>
      <option value="sku">Barcode / SKU</option>
      <option value="brand">Brand</option>
      <option value="size">Size / Weight</option>
    </select>
    <button class="btn btn-red btn-sm" onclick="clearRois()">✕ Clear All Boxes</button>
    <button class="btn btn-outline btn-sm" style="color:#fff;border-color:rgba(255,255,255,.3)" onclick="closeRoiModal()">Cancel</button>
    <button class="btn btn-green btn-sm" onclick="saveRois()">✓ Done</button>
  </div>
  <div class="roi-canvas-wrap">
    <canvas id="roiCanvas"></canvas>
  </div>
  <div class="roi-legend" id="roiLegend"></div>
</div>

<div class="toast" id="toast"></div>
<canvas id="captureCanvas"></canvas>

<script>
// ═══════════════════════════════════════════════
// CONFIG
// ═══════════════════════════════════════════════
const NUM_CAMERAS = 2;
const SLOT_LABELS = ['Front', 'Back'];

// ROI field colours
const ROI_COLORS = {
  name:  '#f4a261',
  exp:   '#e63946',
  sku:   '#4cc9f0',
  brand: '#a8dadc',
  size:  '#c77dff'
};
const ROI_LABELS = { name:'Product Name', exp:'Expiration', sku:'Barcode/SKU', brand:'Brand', size:'Size' };
// ═══════════════════════════════════════════════

let stream      = null;
let shots       = Array(NUM_CAMERAS).fill(null);   // base64 dataURL per slot
let rois        = Array(NUM_CAMERAS).fill(null);   // [{field,x,y,w,h}] per slot (normalised 0-1)
let activeSlot  = 0;
let inventory   = [];
let pendingSampleId = null;

// ── ID generator ──
const usedIds = new Set();
function generateId() {
  let id;
  do { id = String(Math.floor(Math.random() * 9000) + 1000); } while (usedIds.has(id));
  usedIds.add(id);
  return id;
}

// ── Grid ──
function buildGrid() {
  const g = document.getElementById('shotsGrid');
  g.innerHTML = '';
  for (let i = 0; i < NUM_CAMERAS; i++) {
    const d = document.createElement('div');
    d.className = 'slot' + (i === 0 ? ' active' : '');
    d.id = `slot-${i}`;
    d.onclick = () => setSlot(i);
    d.innerHTML = `
      <div class="slot-ph">+</div>
      <div class="slot-lbl">${SLOT_LABELS[i] || 'Shot ' + (i + 1)}</div>
      <button class="slot-rm" onclick="rmShot(event,${i})">✕</button>
      <button class="slot-roi-btn" onclick="openRoiModal(event,${i})" title="Draw OCR regions">✎</button>
      <span class="roi-count" id="roi-count-${i}"></span>`;
    g.appendChild(d);
  }
}

function setSlot(i) {
  activeSlot = i;
  document.querySelectorAll('.slot').forEach((s, idx) => s.classList.toggle('active', idx === i));
  document.getElementById('capNum').textContent = i + 1;
}

function rmShot(e, i) {
  e.stopPropagation();
  shots[i] = null;
  rois[i]  = null;
  const s = document.getElementById(`slot-${i}`);
  s.classList.remove('filled');
  s.innerHTML = `
    <div class="slot-ph">+</div>
    <div class="slot-lbl">${SLOT_LABELS[i] || 'Shot ' + (i + 1)}</div>
    <button class="slot-rm" onclick="rmShot(event,${i})">✕</button>
    <button class="slot-roi-btn" onclick="openRoiModal(event,${i})" title="Draw OCR regions">✎</button>
    <span class="roi-count" id="roi-count-${i}"></span>`;
  updUI();
}

function updUI() {
  const n = shots.filter(Boolean).length;
  document.getElementById('slotCount').textContent = `${n}/${NUM_CAMERAS}`;
  document.getElementById('analyzeBtn').disabled = n === 0;
  for (let i = 0; i < NUM_CAMERAS; i++) {
    const el = document.getElementById(`roi-count-${i}`);
    if (el) el.textContent = rois[i]?.length ? `${rois[i].length} ROI` : '';
  }
}

// ── Camera ──
async function requestPermission() {
  try {
    const probe = await navigator.mediaDevices.getUserMedia({ video: true, audio: false });
    probe.getTracks().forEach(t => t.stop());
    return true;
  } catch(e) {
    const msg = e.name === 'NotAllowedError' ? 'Camera permission denied. Allow camera access in browser settings.'
      : e.name === 'NotFoundError' ? 'No camera found on this device.'
      : 'Camera error: ' + e.message;
    toast(msg, true); return false;
  }
}

async function startCam() {
  if (stream) { stream.getTracks().forEach(t => t.stop()); stream = null; }
  const granted = await requestPermission();
  if (!granted) return;
  await fillCams();
  const sel = document.getElementById('camSel').value;
  const videoConstraints = sel
    ? { deviceId: { exact: sel } }
    : { facingMode: { ideal: 'environment' }, width: { ideal: 1280 }, height: { ideal: 720 } };
  try {
    stream = await navigator.mediaDevices.getUserMedia({ video: videoConstraints, audio: false });
    const video = document.getElementById('video');
    video.srcObject = stream;
    try { await video.play(); } catch(_) {}
    document.getElementById('camOverlay').style.display = 'none';
    document.getElementById('vwrap').classList.add('live');
    document.getElementById('capBtn').disabled = false;
    await fillCams();
    const activeTrack = stream.getVideoTracks()[0];
    if (activeTrack) {
      const settings = activeTrack.getSettings();
      const opts = document.getElementById('camSel').options;
      for (let i = 0; i < opts.length; i++) {
        if (opts[i].value === settings.deviceId) { document.getElementById('camSel').selectedIndex = i; break; }
      }
    }
    toast('Camera ready ✓');
  } catch(e) {
    const msg = e.name === 'NotAllowedError' ? 'Permission denied — allow camera access.'
      : e.name === 'NotReadableError' ? 'Camera in use by another app.'
      : e.name === 'OverconstrainedError' ? 'Camera constraint not supported — try a different camera.'
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
      ? cams.map((c, i) => `<option value="${c.deviceId}" ${c.deviceId===cur?'selected':''}>${c.label || 'Camera '+(i+1)}</option>`).join('')
      : '<option value="">No cameras found</option>';
  } catch(e) {}
}

async function switchCam() { if (stream) await startCam(); }

// ── Capture ──
function captureShot() {
  if (!stream) { toast('Start camera first', true); return; }
  const v = document.getElementById('video');
  const c = document.getElementById('captureCanvas');
  c.width = v.videoWidth || 640; c.height = v.videoHeight || 480;
  c.getContext('2d').drawImage(v, 0, 0);
  const url = c.toDataURL('image/jpeg', 0.88);
  shots[activeSlot] = url;
  rois[activeSlot]  = rois[activeSlot] || [];  // keep existing ROIs if re-capturing same slot

  const s = document.getElementById(`slot-${activeSlot}`);
  s.classList.add('filled');
  s.innerHTML = `
    <img src="${url}" alt="Shot ${activeSlot+1}" />
    <div class="slot-lbl">${SLOT_LABELS[activeSlot] || 'Shot '+(activeSlot+1)}</div>
    <button class="slot-rm" onclick="rmShot(event,${activeSlot})">✕</button>
    <button class="slot-roi-btn" onclick="openRoiModal(event,${activeSlot})" title="Draw OCR regions">✎</button>
    <span class="roi-count" id="roi-count-${activeSlot}"></span>`;

  let nxt = -1;
  for (let i = activeSlot+1; i < NUM_CAMERAS; i++) { if (!shots[i]) { nxt = i; break; } }
  if (nxt === -1) for (let i = 0; i < activeSlot; i++) { if (!shots[i]) { nxt = i; break; } }
  if (nxt !== -1) setSlot(nxt);
  updUI();
  toast(`"${SLOT_LABELS[activeSlot]}" captured`);
}

// ════════════════════════════════════════════
// ROI DRAWING
// ════════════════════════════════════════════
let roiSlotIndex = 0;
let roiDrawing   = false;
let roiStart     = { x: 0, y: 0 };
let roiTempRois  = [];   // working copy while modal is open
let roiImg       = null; // HTMLImageElement for the slot being edited

function openRoiModal(e, slotIdx) {
  e.stopPropagation();
  if (!shots[slotIdx]) { toast('Capture a shot first', true); return; }
  roiSlotIndex = slotIdx;
  roiTempRois  = JSON.parse(JSON.stringify(rois[slotIdx] || []));

  const modal = document.getElementById('roiModal');
  modal.classList.add('open');

  roiImg = new Image();
  roiImg.onload = () => { renderRoiCanvas(); };
  roiImg.src = shots[slotIdx];
}

function closeRoiModal() {
  document.getElementById('roiModal').classList.remove('open');
}

function saveRois() {
  rois[roiSlotIndex] = JSON.parse(JSON.stringify(roiTempRois));
  updUI();
  closeRoiModal();
  toast(`${roiTempRois.length} ROI region${roiTempRois.length!==1?'s':''} saved for ${SLOT_LABELS[roiSlotIndex]}`);
}

function clearRois() {
  roiTempRois = [];
  renderRoiCanvas();
}

function renderRoiCanvas() {
  const canvas = document.getElementById('roiCanvas');
  const wrap   = canvas.parentElement;

  // Size canvas to image natural aspect ratio, fitting in the wrap
  const maxW = Math.min(window.innerWidth * 0.88, 900);
  const maxH = window.innerHeight * 0.58;
  const imgW = roiImg.naturalWidth;
  const imgH = roiImg.naturalHeight;
  const scale = Math.min(maxW / imgW, maxH / imgH);
  canvas.width  = Math.round(imgW * scale);
  canvas.height = Math.round(imgH * scale);

  const ctx = canvas.getContext('2d');
  ctx.drawImage(roiImg, 0, 0, canvas.width, canvas.height);

  // Draw saved ROIs
  roiTempRois.forEach(roi => {
    const cx = roi.x * canvas.width;
    const cy = roi.y * canvas.height;
    const cw = roi.w * canvas.width;
    const ch = roi.h * canvas.height;
    ctx.strokeStyle = ROI_COLORS[roi.field] || '#fff';
    ctx.lineWidth = 2;
    ctx.setLineDash([]);
    ctx.strokeRect(cx, cy, cw, ch);
    ctx.fillStyle = (ROI_COLORS[roi.field] || '#fff') + '33';
    ctx.fillRect(cx, cy, cw, ch);
    // Label
    ctx.fillStyle = ROI_COLORS[roi.field] || '#fff';
    ctx.font = 'bold 11px DM Mono, monospace';
    ctx.fillText(ROI_LABELS[roi.field], cx + 4, cy + 13);
  });

  // Update legend
  updateLegend();
}

function updateLegend() {
  const legend = document.getElementById('roiLegend');
  const fields = [...new Set(roiTempRois.map(r => r.field))];
  legend.innerHTML = fields.map(f => `
    <div class="roi-legend-item">
      <div class="roi-legend-swatch" style="background:${ROI_COLORS[f]}"></div>
      ${ROI_LABELS[f]}
    </div>`).join('');
}

// Mouse/touch events on the ROI canvas
function getCanvasPos(canvas, evt) {
  const rect = canvas.getBoundingClientRect();
  const clientX = evt.touches ? evt.touches[0].clientX : evt.clientX;
  const clientY = evt.touches ? evt.touches[0].clientY : evt.clientY;
  return {
    x: (clientX - rect.left) / rect.width,
    y: (clientY - rect.top)  / rect.height
  };
}

document.getElementById('roiCanvas').addEventListener('mousedown',  roiPointerDown);
document.getElementById('roiCanvas').addEventListener('mousemove',  roiPointerMove);
document.getElementById('roiCanvas').addEventListener('mouseup',    roiPointerUp);
document.getElementById('roiCanvas').addEventListener('touchstart', e => { e.preventDefault(); roiPointerDown(e); }, { passive: false });
document.getElementById('roiCanvas').addEventListener('touchmove',  e => { e.preventDefault(); roiPointerMove(e); }, { passive: false });
document.getElementById('roiCanvas').addEventListener('touchend',   e => { e.preventDefault(); roiPointerUp(e); },   { passive: false });

function roiPointerDown(e) {
  roiDrawing = true;
  roiStart = getCanvasPos(document.getElementById('roiCanvas'), e);
}

function roiPointerMove(e) {
  if (!roiDrawing) return;
  const canvas = document.getElementById('roiCanvas');
  const cur = getCanvasPos(canvas, e);
  // Redraw base + saved rects
  renderRoiCanvas();
  // Draw in-progress rect
  const ctx = canvas.getContext('2d');
  const field = document.getElementById('roiField').value;
  const color = ROI_COLORS[field] || '#fff';
  const x = roiStart.x * canvas.width;
  const y = roiStart.y * canvas.height;
  const w = (cur.x - roiStart.x) * canvas.width;
  const h = (cur.y - roiStart.y) * canvas.height;
  ctx.strokeStyle = color;
  ctx.lineWidth = 2;
  ctx.setLineDash([5, 3]);
  ctx.strokeRect(x, y, w, h);
  ctx.fillStyle = color + '22';
  ctx.fillRect(x, y, w, h);
}

function roiPointerUp(e) {
  if (!roiDrawing) return;
  roiDrawing = false;
  const canvas = document.getElementById('roiCanvas');
  const cur = getCanvasPos(canvas, e);
  const field = document.getElementById('roiField').value;

  // Normalise so x,y is always top-left and w,h are positive
  const x = Math.min(roiStart.x, cur.x);
  const y = Math.min(roiStart.y, cur.y);
  const w = Math.abs(cur.x - roiStart.x);
  const h = Math.abs(cur.y - roiStart.y);

  if (w > 0.01 && h > 0.01) {  // ignore tiny accidental taps
    roiTempRois.push({ field, x, y, w, h });
  }
  renderRoiCanvas();
}

// ════════════════════════════════════════════
// GOOGLE VISION OCR
// ════════════════════════════════════════════

// Crop image to an ROI (normalised coords) and return base64 jpeg
function cropToRoi(base64img, roi) {
  return new Promise(resolve => {
    const img = new Image();
    img.onload = () => {
      const c = document.createElement('canvas');
      c.width  = Math.round(roi.w * img.naturalWidth);
      c.height = Math.round(roi.h * img.naturalHeight);
      c.getContext('2d').drawImage(img,
        roi.x * img.naturalWidth, roi.y * img.naturalHeight,
        c.width, c.height, 0, 0, c.width, c.height);
      resolve(c.toDataURL('image/jpeg', 0.9).split(',')[1]);
    };
    img.src = 'data:image/jpeg;base64,' + base64img;
  });
}

async function visionOCR(apiKey, base64img) {
  const body = {
    requests: [{
      image: { content: base64img },
      features: [
        { type: 'TEXT_DETECTION',    maxResults: 1 },
        { type: 'BARCODE_DETECTION', maxResults: 5 }
      ]
    }]
  };
  const resp = await fetch(
    `https://vision.googleapis.com/v1/images:annotate?key=${apiKey}`,
    { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(body) }
  );
  if (!resp.ok) { const e = await resp.json(); throw new Error(e.error?.message || 'Google Vision API error'); }
  const data   = await resp.json();
  const result = data.responses[0];
  return {
    fullText: result.fullTextAnnotation?.text || result.textAnnotations?.[0]?.description || '',
    barcodes: (result.barcodeAnnotations || []).map(b => b.rawValue).filter(Boolean)
  };
}

// ── Smart label parser ──
function parseLabel(allText, allBarcodes) {
  const lines = allText.split(/\n+/).map(l => l.trim()).filter(Boolean);
  const text  = allText;

  const expPatterns = [
    /(?:exp(?:iry|ires?|[. :])|best[ -]?by[: ]|use[ -]?by[: ]|sell[ -]?by[: ]|bb[: ]|bbd[: ])\s*([A-Za-z0-9\/\-\. ]{4,20})/i,
    /\b((?:jan|feb|mar|apr|may|jun|jul|aug|sep|oct|nov|dec)[a-z]*\.?\s+\d{1,2}[,\s]+\d{2,4})\b/i,
    /\b(\d{1,2}[\/\-]\d{1,2}[\/\-]\d{2,4})\b/,
    /\b(\d{4}[\/\-]\d{2}[\/\-]\d{2})\b/,
    /\b(\d{2}[\/\-]\d{4})\b/
  ];
  let expiration = null;
  for (const pat of expPatterns) { const m = text.match(pat); if (m) { expiration = (m[1]||m[0]).trim(); break; } }

  let sku = allBarcodes.length ? allBarcodes[0] : null;
  if (!sku) {
    const skuMatches = [...text.matchAll(/\b(\d{8,14})\b/g)].map(m => m[1]);
    sku = skuMatches.find(s => s.length===12||s.length===13) || skuMatches.find(s => s.length>=8) || null;
  }

  const sizeMatch = text.match(/\b(\d+\.?\d*\s*(?:fl\.?\s*oz|oz|lb|lbs|g|kg|ml|l|liters?|ounces?|pounds?|ct|count|pcs|pieces?|pack))\b/i);
  const size = sizeMatch ? sizeMatch[1].replace(/\s+/g,' ').trim() : null;

  const junk = /^[\d\s\-\/\.\,]+$|www\.|\.com|@|\bllc\b|\binc\b|\bcorp\b|tel:|fax:|distributed|manufactured|contains|ingredients|nutrition|serving|calories|total|sodium|sugar|protein|fat|carb|\bpo box\b/i;
  const candidates = lines.filter(l => l.length > 2 && l.length < 60 && !junk.test(l) && !/^\d{6,}$/.test(l));
  const brand = candidates[0] || null;
  const name  = candidates.slice(0, 3).join(' ') || null;

  const allergenMatch = text.match(/contains?:?\s*([^\.\n]{5,80})/i);
  const flavorMatches = [...new Set((text.match(/\b(original|classic|light|diet|zero|sugar[ -]free|organic|natural|low[ -]fat|whole grain|gluten[ -]free|vegan|vitamin[- ][a-z])\b/gi)||[]).map(s=>s.toLowerCase()))];
  let notes = '';
  if (allergenMatch) notes += 'Contains: ' + allergenMatch[1].trim().slice(0,80);
  if (flavorMatches.length) notes += (notes?' | ':'') + flavorMatches.slice(0,4).join(', ');
  notes = notes.slice(0,180) || null;

  return { brand, name, expiration, sku, size, notes };
}

// ── ROI-aware field text map ──
// Returns { fieldName: combinedText } from ROI-cropped OCR, falls back to full-image parse
async function roiFieldTexts(apiKey, base64img, slotRois) {
  if (!slotRois || slotRois.length === 0) return null;

  // Group ROIs by field
  const byField = {};
  for (const roi of slotRois) {
    if (!byField[roi.field]) byField[roi.field] = [];
    byField[roi.field].push(roi);
  }

  const result = {};
  // For each field, OCR all its ROI crops and concatenate
  for (const [field, fieldRois] of Object.entries(byField)) {
    const texts = [];
    for (const roi of fieldRois) {
      const cropped = await cropToRoi(base64img, roi);
      const { fullText, barcodes } = await visionOCR(apiKey, cropped);
      texts.push(field === 'sku' && barcodes.length ? barcodes.join(' ') : fullText);
    }
    result[field] = texts.join(' ').trim();
  }
  return result;
}

async function analyzeShots() {
  const key = document.getElementById('apiKey').value.trim();
  if (!key) { toast('Enter your Google Vision API key first', true); return; }
  const imgs = shots.map((s, i) => s ? { data: s.split(',')[1], idx: i } : null).filter(Boolean);
  if (!imgs.length) { toast('Capture at least one shot', true); return; }

  document.getElementById('analyzeBar').classList.add('on');
  document.getElementById('reviewCard').classList.remove('on');
  document.getElementById('analyzeBtn').disabled = true;

  try {
    // Determine if any slot has ROIs defined
    const hasAnyRoi = imgs.some(img => rois[img.idx]?.length > 0);

    let parsed;

    if (hasAnyRoi) {
      // ROI mode: run cropped OCR per field, merge across slots
      const fieldTexts = { name:'', brand:'', exp:'', sku:'', size:'' };
      let allBarcodes  = [];

      for (const img of imgs) {
        const slotRois = rois[img.idx];
        if (slotRois?.length > 0) {
          const ft = await roiFieldTexts(key, img.data, slotRois);
          if (ft) {
            for (const [f, t] of Object.entries(ft)) {
              if (f === 'sku' && /^\d{8,}$/.test(t.trim())) allBarcodes.push(t.trim());
              fieldTexts[f] = (fieldTexts[f] + ' ' + t).trim();
            }
          }
        } else {
          // No ROI on this slot — still do full OCR and let parser fill gaps
          const { fullText, barcodes } = await visionOCR(key, img.data);
          allBarcodes = [...allBarcodes, ...barcodes];
          const fp = parseLabel(fullText, barcodes);
          if (!fieldTexts.name  && fp.name)       fieldTexts.name  = fp.name;
          if (!fieldTexts.brand && fp.brand)       fieldTexts.brand = fp.brand;
          if (!fieldTexts.exp   && fp.expiration)  fieldTexts.exp   = fp.expiration;
          if (!fieldTexts.sku   && fp.sku)         fieldTexts.sku   = fp.sku;
          if (!fieldTexts.size  && fp.size)        fieldTexts.size  = fp.size;
        }
      }

      // Clean up each field text with the parser where needed
      const mergedText = Object.values(fieldTexts).join('\n');
      const fallback   = parseLabel(mergedText, allBarcodes);

      parsed = {
        name:       fieldTexts.name  || fallback.name,
        brand:      fieldTexts.brand || fallback.brand,
        expiration: fieldTexts.exp   || fallback.expiration,
        sku:        (allBarcodes[0]  || fieldTexts.sku || fallback.sku),
        size:       fieldTexts.size  || fallback.size,
        notes:      fallback.notes
      };

    } else {
      // No ROIs — full image OCR on all shots, merge and parse
      const results = await Promise.all(imgs.map(img => visionOCR(key, img.data)));
      const allText     = results.map(r => r.fullText).join('\n\n');
      const allBarcodes = [...new Set(results.flatMap(r => r.barcodes))];
      parsed = parseLabel(allText, allBarcodes);
    }

    // Generate sample ID
    pendingSampleId = generateId();
    document.getElementById('sampleIdBadge').textContent = pendingSampleId;

    document.getElementById('rf-name').value  = parsed.name  || '';
    document.getElementById('rf-brand').value = parsed.brand || '';
    document.getElementById('rf-exp').value   = parsed.expiration || '';
    document.getElementById('rf-sku').value   = parsed.sku   || '';
    document.getElementById('rf-size').value  = parsed.size  || '';
    document.getElementById('rf-notes').value = parsed.notes || '';

    document.getElementById('analyzeBar').classList.remove('on');
    document.getElementById('reviewCard').classList.add('on');

    const found = [parsed.name, parsed.expiration, parsed.sku].filter(Boolean).length;
    if (found === 0) toast('Low confidence — try better lighting or draw ROI boxes', true);
    else toast(`Extracted ✓  Sample ID: ${pendingSampleId}`);

  } catch(e) {
    document.getElementById('analyzeBar').classList.remove('on');
    document.getElementById('analyzeBtn').disabled = false;
    toast('Error: ' + e.message, true);
  }
}

// ── Save ──
function saveEntry() {
  const entry = {
    id:         Date.now(),
    sampleId:   pendingSampleId || generateId(),
    name:       document.getElementById('rf-name').value.trim()  || '—',
    brand:      document.getElementById('rf-brand').value.trim() || null,
    expiration: document.getElementById('rf-exp').value.trim()   || null,
    sku:        document.getElementById('rf-sku').value.trim()   || null,
    size:       document.getElementById('rf-size').value.trim()  || null,
    notes:      document.getElementById('rf-notes').value.trim() || null,
    timestamp:  new Date().toLocaleString()
  };
  inventory.push(entry);
  renderTable();
  dismissReview();
  resetShots();
  document.getElementById('countPill').textContent = `${inventory.length} item${inventory.length!==1?'s':''}`;
  toast(`Saved · ID: ${entry.sampleId}`);
}

function dismissReview() {
  document.getElementById('reviewCard').classList.remove('on');
  document.getElementById('analyzeBtn').disabled = shots.filter(Boolean).length === 0;
  pendingSampleId = null;
}

function resetShots() {
  shots = Array(NUM_CAMERAS).fill(null);
  rois  = Array(NUM_CAMERAS).fill(null);
  buildGrid(); setSlot(0); updUI();
}

// ── Expiry helpers ──
function expClass(raw) {
  if (!raw) return 'td-na';
  const d = new Date(raw); if (isNaN(d)) return 'td-exp-ok';
  const diff = (d - Date.now()) / 86400000;
  if (diff < 0)  return 'td-exp-over';
  if (diff < 30) return 'td-exp-soon';
  return 'td-exp-ok';
}
function expTag(raw) {
  if (!raw) return '';
  const d = new Date(raw); if (isNaN(d)) return '';
  const diff = (d - Date.now()) / 86400000;
  if (diff < 0)  return `<span class="exp-tag tag-over">EXPIRED</span>`;
  if (diff < 30) return `<span class="exp-tag tag-soon">SOON</span>`;
  return '';
}

// ── Table ──
function renderTable() {
  const q    = document.getElementById('searchBox')?.value.trim().toLowerCase() || '';
  const body = document.getElementById('invBody');
  const tbl  = document.getElementById('invTable');
  const emp  = document.getElementById('emptyState');

  let rows = inventory;
  if (q) rows = rows.filter(e =>
    [e.sampleId, e.name, e.brand, e.sku, e.expiration].some(v => v?.toLowerCase().includes(q))
  );

  if (!rows.length) {
    tbl.style.display = 'none'; emp.style.display = 'block';
    emp.querySelector('p').textContent = q ? 'No matches found' : 'No samples logged yet';
    emp.querySelector('span').textContent = q ? `Searching "${q}"` : 'Capture → Analyze → Save';
    return;
  }
  tbl.style.display = 'table'; emp.style.display = 'none';
  body.innerHTML = rows.map((e, i) => `
    <tr>
      <td class="td-id">${e.sampleId}</td>
      <td class="td-num">${inventory.indexOf(e)+1}</td>
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
  const entry = inventory.find(e => e.id === id);
  if (entry) usedIds.delete(entry.sampleId);
  inventory = inventory.filter(e => e.id !== id);
  renderTable();
  document.getElementById('countPill').textContent = `${inventory.length} item${inventory.length!==1?'s':''}`;
}

function clearAll() {
  if (!inventory.length) return;
  if (!confirm(`Delete all ${inventory.length} entries?`)) return;
  inventory = []; usedIds.clear(); renderTable();
  document.getElementById('countPill').textContent = '0 items';
}

// ── Exports ──
function buildRows() {
  return inventory.map((e, i) => ({
    'Sample ID': e.sampleId,
    '#': i+1,
    'Product Name': e.name,
    'Brand': e.brand || '',
    'Expiration': e.expiration || '',
    'Barcode/SKU': e.sku || '',
    'Size': e.size || '',
    'Notes': e.notes || '',
    'Logged At': e.timestamp
  }));
}

function exportCSV() {
  if (!inventory.length) { toast('Nothing to export', true); return; }
  const rows = buildRows(); const hdrs = Object.keys(rows[0]);
  const csv = [hdrs, ...rows.map(r => hdrs.map(h => `"${String(r[h]).replace(/"/g,'""')}"`))].map(r => r.join(',')).join('\n');
  const a = document.createElement('a');
  a.href = URL.createObjectURL(new Blob([csv], { type: 'text/csv' }));
  a.download = `inventory-${today()}.csv`; a.click();
  toast(`Exported ${inventory.length} rows as CSV`);
}

function exportXLSX() {
  if (!inventory.length) { toast('Nothing to export', true); return; }
  const ws = XLSX.utils.json_to_sheet(buildRows());
  ws['!cols'] = [{wch:8},{wch:4},{wch:28},{wch:18},{wch:16},{wch:16},{wch:10},{wch:40},{wch:20}];
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, 'Inventory');
  XLSX.writeFile(wb, `inventory-${today()}.xlsx`);
  toast(`Exported ${inventory.length} rows as Excel`);
}

function today() { return new Date().toISOString().slice(0,10); }
function toggleKey() { const i = document.getElementById('apiKey'); i.type = i.type==='password'?'text':'password'; }

let toastT;
function toast(msg, err=false) {
  const t = document.getElementById('toast');
  t.textContent = msg; t.style.borderLeftColor = err ? 'var(--red)' : 'var(--green)';
  t.classList.add('show'); clearTimeout(toastT);
  toastT = setTimeout(() => t.classList.remove('show'), 3000);
}

buildGrid();
</script>

<!-- SBC note (hidden, for reference) -->
<!--
  BUDGET SBC OPTIONS WITH ONBOARD OCR (no API needed):
  1. Raspberry Pi 4 (2GB) ~$45 + camera module ~$25
     → Run Tesseract OCR locally via Python script
     → Or run PaddleOCR (much more accurate than Tesseract for product labels)
     → Serve this web app from the Pi over your local network (python3 -m http.server)
     → Pi handles all OCR — zero API calls, unlimited scans
  2. Orange Pi 5 ~$60 — faster than RPi4, same software stack
  3. Radxa Rock 5B ~$80 — has NPU for accelerated OCR inference
  All run PaddleOCR or EasyOCR well. PaddleOCR is the recommended choice —
  it handles curved/skewed label text much better than Tesseract.
-->
</body>
</html>
