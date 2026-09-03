<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, viewport-fit=cover">
<title>PaceLab 配速實驗室｜跑者配速與目標換算</title>
<meta name="description" content="配速 ⇄ 完賽時間雙向換算、戰術配速策略建議、田徑場圈速轉換器">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="PaceLab">
<meta name="theme-color" content="#0B1F3A">
<link id="manifest-link" rel="manifest">
<link id="apple-icon-link" rel="apple-touch-icon">
<link id="icon-link" rel="icon">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=JetBrains+Mono:wght@500;700&display=swap" rel="stylesheet">
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          navy: {
            950: '#08152A',
            900: '#0B1F3A',
            800: '#12294D',
            700: '#1C3A63',
            600: '#2A4E7E',
            100: '#E7ECF4',
            50:  '#F2F5FA'
          }
        },
        fontFamily: {
          sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'],
          mono: ['"JetBrains Mono"', 'ui-monospace', 'SFMono-Regular', 'monospace']
        }
      }
    }
  }
</script>
<style>
  html, body { -webkit-tap-highlight-color: transparent; overscroll-behavior-y: contain; }
  input[type="text"], input[type="number"] { font-variant-numeric: tabular-nums; }
  .no-scrollbar::-webkit-scrollbar { display: none; }
  .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
  .tap { transition: transform .08s ease, background-color .12s ease, color .12s ease, border-color .12s ease; }
  .tap:active { transform: scale(0.96); }
  .view { animation: fadein .18s ease; }
  @keyframes fadein { from { opacity: 0; transform: translateY(4px); } to { opacity: 1; transform: translateY(0); } }
  @media (prefers-reduced-motion: reduce) {
    .tap, .view { animation: none !important; transition: none !important; }
  }
</style>
</head>
<body class="bg-navy-50 text-navy-950 font-sans min-h-screen pb-24 selection:bg-navy-900 selection:text-white">

  <!-- Header -->
  <header class="sticky top-0 z-30 bg-navy-900 text-white shadow-md shadow-navy-900/10">
    <div class="max-w-md mx-auto px-5 pt-[calc(env(safe-area-inset-top)+14px)] pb-4">
      <div class="flex items-center gap-3">
        <div class="w-10 h-10 shrink-0 rounded-xl bg-white/10 border border-white/15 flex items-center justify-center">
          <svg width="22" height="22" viewBox="0 0 24 24" fill="none">
            <circle cx="12" cy="12" r="9" stroke="white" stroke-width="1.6"/>
            <path d="M12 7v5l3.2 2" stroke="white" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
        <div>
          <h1 class="text-lg font-bold leading-tight tracking-tight">PaceLab 配速實驗室</h1>
          <p class="text-[13px] text-white/60 leading-tight">配速換算・戰術策略・田徑場圈速</p>
        </div>
      </div>
    </div>
  </header>

  <main class="max-w-md mx-auto px-4 pt-5 space-y-5">

    <!-- ============ VIEW: 距離配速換算 ============ -->
    <section id="view-pace" class="view space-y-5">

      <!-- Mode segmented control -->
      <div class="grid grid-cols-2 gap-2 bg-navy-100 p-1.5 rounded-2xl">
        <button data-mode="A" class="mode-btn tap h-12 rounded-xl text-[15px] font-semibold">配速 → 時間</button>
        <button data-mode="B" class="mode-btn tap h-12 rounded-xl text-[15px] font-semibold">時間 → 配速</button>
      </div>

      <!-- Mode A: 配速輸入 -->
      <div id="modeA" class="space-y-4">
        <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100">
          <label class="block text-[13px] font-semibold text-navy-700 mb-2">預計公里配速（分:秒 / 公里）</label>
          <div class="flex items-center gap-2">
            <input id="paceInput" type="text" inputmode="numeric" placeholder="5:00"
              class="flex-1 h-14 px-4 rounded-xl border-2 border-navy-100 focus:border-navy-700 outline-none text-2xl font-mono font-bold text-navy-950 tracking-wide">
            <span class="text-sm text-navy-500 font-medium shrink-0">/ 公里</span>
          </div>
          <p id="paceInputError" class="text-xs text-rose-600 mt-1.5 hidden">請輸入正確格式，例如 5:00</p>
          <div class="flex flex-wrap gap-2 mt-3" id="paceQuickRow"></div>
        </div>

        <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100">
          <h2 class="text-[13px] font-semibold text-navy-700 mb-3">預計完賽時間</h2>
          <div id="paceResults" class="divide-y divide-navy-100"></div>
        </div>
      </div>

      <!-- Mode B: 目標時間輸入 -->
      <div id="modeB" class="space-y-4 hidden">
        <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100">
          <label class="block text-[13px] font-semibold text-navy-700 mb-2">選擇距離</label>
          <div class="grid grid-cols-4 gap-2" id="distChipsB"></div>
        </div>

        <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100">
          <label class="block text-[13px] font-semibold text-navy-700 mb-2">目標完賽時間（時:分:秒）</label>
          <input id="timeInput" type="text" inputmode="numeric" placeholder="1:45:00"
            class="w-full h-14 px-4 rounded-xl border-2 border-navy-100 focus:border-navy-700 outline-none text-2xl font-mono font-bold text-navy-950 tracking-wide">
          <p id="timeInputError" class="text-xs text-rose-600 mt-1.5 hidden">請輸入正確格式，例如 1:45:00 或 45:00</p>
          <div class="flex flex-wrap gap-2 mt-3" id="timeQuickRow"></div>
        </div>

        <div class="bg-navy-900 rounded-2xl p-5 shadow-sm text-white">
          <p class="text-[13px] font-semibold text-white/60 mb-1">所需平均配速</p>
          <p id="requiredPace" class="text-4xl font-mono font-extrabold tracking-wide">—</p>
          <p class="text-[13px] text-white/50 mt-1">分:秒 / 公里</p>
        </div>
      </div>

      <!-- Virtual Coach -->
      <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100 space-y-4">
        <div class="flex items-center justify-between">
          <h2 class="text-[15px] font-bold text-navy-900">教練戰術建議</h2>
          <span class="text-[11px] text-navy-500 bg-navy-100 px-2 py-1 rounded-full">Virtual Coach</span>
        </div>

        <div>
          <p class="text-[13px] font-semibold text-navy-700 mb-2">套用於哪個距離？</p>
          <div class="grid grid-cols-4 gap-2" id="distChipsCoach"></div>
        </div>

        <div class="grid grid-cols-3 gap-2 bg-navy-100 p-1.5 rounded-2xl">
          <button data-strategy="even" class="strategy-btn tap h-11 rounded-xl text-[13px] font-semibold">均速</button>
          <button data-strategy="negative" class="strategy-btn tap h-11 rounded-xl text-[13px] font-semibold">負分段</button>
          <button data-strategy="positive" class="strategy-btn tap h-11 rounded-xl text-[13px] font-semibold">前快後穩</button>
        </div>

        <div>
          <p class="text-[12px] font-medium text-navy-500 mb-2">分段配速差距</p>
          <div class="grid grid-cols-3 gap-2" id="deltaChips"></div>
        </div>

        <div id="coachOutput" class="space-y-3"></div>
      </div>
    </section>

    <!-- ============ VIEW: 田徑場圈速 ============ -->
    <section id="view-track" class="view space-y-5 hidden">

      <div class="grid grid-cols-2 gap-2 bg-navy-100 p-1.5 rounded-2xl">
        <button data-tdir="lapToPace" class="tdir-btn tap h-12 rounded-xl text-[14px] font-semibold leading-tight">400m 圈速<br>→ 公里配速</button>
        <button data-tdir="paceToLap" class="tdir-btn tap h-12 rounded-xl text-[14px] font-semibold leading-tight">公里配速<br>→ 400m 圈速</button>
      </div>

      <!-- 400m -> pace -->
      <div id="trackLapToPace" class="space-y-4">
        <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100">
          <label class="block text-[13px] font-semibold text-navy-700 mb-2">田徑場一圈（400m）時間（秒）</label>
          <div class="flex items-center gap-2">
            <input id="lapInput" type="text" inputmode="numeric" placeholder="110"
              class="flex-1 h-14 px-4 rounded-xl border-2 border-navy-100 focus:border-navy-700 outline-none text-2xl font-mono font-bold text-navy-950 tracking-wide">
            <span class="text-sm text-navy-500 font-medium shrink-0">秒</span>
          </div>
          <p id="lapInputError" class="text-xs text-rose-600 mt-1.5 hidden">請輸入正確的秒數，例如 110</p>
          <div class="flex flex-wrap gap-2 mt-3" id="lapQuickRow"></div>
        </div>

        <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100">
          <h2 class="text-[13px] font-semibold text-navy-700 mb-3">換算結果</h2>
          <div id="lapResults" class="divide-y divide-navy-100"></div>
        </div>
      </div>

      <!-- pace -> 400m -->
      <div id="trackPaceToLap" class="space-y-4 hidden">
        <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100">
          <label class="block text-[13px] font-semibold text-navy-700 mb-2">公里配速（分:秒 / 公里）</label>
          <div class="flex items-center gap-2">
            <input id="paceInput2" type="text" inputmode="numeric" placeholder="5:00"
              class="flex-1 h-14 px-4 rounded-xl border-2 border-navy-100 focus:border-navy-700 outline-none text-2xl font-mono font-bold text-navy-950 tracking-wide">
            <span class="text-sm text-navy-500 font-medium shrink-0">/ 公里</span>
          </div>
          <p id="paceInput2Error" class="text-xs text-rose-600 mt-1.5 hidden">請輸入正確格式，例如 5:00</p>
          <div class="flex flex-wrap gap-2 mt-3" id="paceQuickRow2"></div>
        </div>

        <div class="bg-white rounded-2xl p-4 shadow-sm border border-navy-100">
          <h2 class="text-[13px] font-semibold text-navy-700 mb-3">換算結果</h2>
          <div id="lapResults2" class="divide-y divide-navy-100"></div>
        </div>
      </div>
    </section>
  </main>

  <!-- Bottom nav -->
  <nav class="fixed bottom-0 inset-x-0 z-30 bg-white border-t border-navy-100 pb-[env(safe-area-inset-bottom)]">
    <div class="max-w-md mx-auto grid grid-cols-2">
      <button data-view="pace" class="nav-btn tap flex flex-col items-center justify-center gap-1 py-2.5">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" class="nav-icon">
          <path d="M4 19h16M6 19V9m4 10V5m4 14v-7m4 7v-3" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
        </svg>
        <span class="text-[12px] font-semibold">距離配速</span>
      </button>
      <button data-view="track" class="nav-btn tap flex flex-col items-center justify-center gap-1 py-2.5">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" class="nav-icon">
          <ellipse cx="12" cy="12" rx="9" ry="6.5" stroke="currentColor" stroke-width="1.8"/>
          <ellipse cx="12" cy="12" rx="4.2" ry="6.5" stroke="currentColor" stroke-width="1.8"/>
        </svg>
        <span class="text-[12px] font-semibold">田徑場圈速</span>
      </button>
    </div>
  </nav>

<script>
(function () {
  "use strict";

  /* ---------------- PWA manifest / icon (inline, single-file) ---------------- */
  const iconSvg = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 192 192">
    <rect width="192" height="192" rx="40" fill="#0B1F3A"/>
    <circle cx="96" cy="96" r="62" fill="none" stroke="#ffffff" stroke-width="9"/>
    <path d="M96 58v40l28 16" fill="none" stroke="#ffffff" stroke-width="9" stroke-linecap="round" stroke-linejoin="round"/>
  </svg>`;
  const iconUrl = "data:image/svg+xml," + encodeURIComponent(iconSvg);
  document.getElementById('apple-icon-link').href = iconUrl;
  document.getElementById('icon-link').href = iconUrl;

  const manifest = {
    name: "PaceLab 配速實驗室",
    short_name: "PaceLab",
    start_url: ".",
    display: "standalone",
    background_color: "#0B1F3A",
    theme_color: "#0B1F3A",
    icons: [{ src: iconUrl, sizes: "192x192", type: "image/svg+xml", purpose: "any" }]
  };
  const manifestBlob = new Blob([JSON.stringify(manifest)], { type: "application/manifest+json" });
  document.getElementById('manifest-link').href = URL.createObjectURL(manifestBlob);

  /* ---------------- Utilities ---------------- */
  const pad2 = n => (n < 10 ? "0" + n : "" + n);

  // Parse "ss" | "mm:ss" | "hh:mm:ss" -> total seconds (float) or NaN
  function parseFlexibleTime(str) {
    if (typeof str !== "string") return NaN;
    str = str.trim();
    if (!str) return NaN;
    if (!/^[0-9:.]+$/.test(str)) return NaN;
    const parts = str.split(":");
    if (parts.length > 3) return NaN;
    const nums = parts.map(p => (p === "" ? NaN : Number(p)));
    if (nums.some(n => isNaN(n) || n < 0)) return NaN;
    let sec;
    if (nums.length === 1) sec = nums[0];
    else if (nums.length === 2) sec = nums[0] * 60 + nums[1];
    else sec = nums[0] * 3600 + nums[1] * 60 + nums[2];
    return sec > 0 ? sec : NaN;
  }

  // seconds -> "m:ss" (pace style, minutes can exceed 59)
  function fmtPace(sec) {
    sec = Math.round(sec);
    const m = Math.floor(sec / 60);
    const s = sec % 60;
    return `${m}:${pad2(s)}`;
  }

  // seconds -> "h:mm:ss" or "m:ss"
  function fmtTime(sec) {
    sec = Math.round(sec);
    const h = Math.floor(sec / 3600);
    const m = Math.floor((sec % 3600) / 60);
    const s = sec % 60;
    if (h > 0) return `${h}:${pad2(m)}:${pad2(s)}`;
    return `${m}:${pad2(s)}`;
  }

  const DISTANCES = {
    "5k":   { km: 5,       label: "5 公里" },
    "10k":  { km: 10,      label: "10 公里" },
    "half": { km: 21.0975, label: "半程馬拉松" },
    "full": { km: 42.195,  label: "全程馬拉松" }
  };

  const TIME_PRESETS = {
    "5k":   ["20:00", "25:00", "30:00", "35:00"],
    "10k":  ["45:00", "50:00", "1:00:00", "1:10:00"],
    "half": ["1:45:00", "2:00:00", "2:15:00", "2:30:00"],
    "full": ["3:30:00", "4:00:00", "4:30:00", "5:00:00"]
  };

  const PACE_PRESETS = ["3:30", "4:00", "4:30", "5:00", "5:30", "6:00", "6:30", "7:00"];
  const LAP_PRESETS = [70, 75, 80, 85, 90, 95, 100, 110, 120, 130];

  function chipBtn(label, extraClass) {
    return `<button type="button" class="chip tap ${extraClass || ""} h-11 px-3 rounded-xl bg-navy-50 border border-navy-100 text-navy-800 text-[13px] font-semibold font-mono active:bg-navy-100">${label}</button>`;
  }

  /* ---------------- State ---------------- */
  const state = {
    view: "pace",
    mode: "A",
    coachDist: "half",
    strategy: "even",
    delta: 4,
    trackDir: "lapToPace"
  };

  /* ---------------- Nav ---------------- */
  function renderNav() {
    document.querySelectorAll(".nav-btn").forEach(btn => {
      const active = btn.dataset.view === state.view;
      btn.classList.toggle("text-navy-900", active);
      btn.classList.toggle("text-navy-400", !active);
      const icon = btn.querySelector(".nav-icon");
      icon.classList.toggle("text-navy-900", active);
    });
    document.getElementById("view-pace").classList.toggle("hidden", state.view !== "pace");
    document.getElementById("view-track").classList.toggle("hidden", state.view !== "track");
  }
  document.querySelectorAll(".nav-btn").forEach(btn => {
    btn.addEventListener("click", () => { state.view = btn.dataset.view; renderNav(); });
  });

  /* ---------------- Mode A / B toggle ---------------- */
  function renderModeButtons() {
    document.querySelectorAll(".mode-btn").forEach(btn => {
      const active = btn.dataset.mode === state.mode;
      btn.classList.toggle("bg-navy-900", active);
      btn.classList.toggle("text-white", active);
      btn.classList.toggle("shadow", active);
      btn.classList.toggle("text-navy-500", !active);
    });
    document.getElementById("modeA").classList.toggle("hidden", state.mode !== "A");
    document.getElementById("modeB").classList.toggle("hidden", state.mode !== "B");
  }
  document.querySelectorAll(".mode-btn").forEach(btn => {
    btn.addEventListener("click", () => {
      state.mode = btn.dataset.mode;
      renderModeButtons();
      syncCoachFromMode();
    });
  });

  /* ---------------- Mode A: 配速 -> 時間 ---------------- */
  const paceQuickRow = document.getElementById("paceQuickRow");
  paceQuickRow.innerHTML = PACE_PRESETS.map(p => chipBtn(p)).join("");
  paceQuickRow.querySelectorAll(".chip").forEach((btn, i) => {
    btn.addEventListener("click", () => {
      document.getElementById("paceInput").value = PACE_PRESETS[i];
      updateModeA();
    });
  });

  const paceInput = document.getElementById("paceInput");
  const paceInputError = document.getElementById("paceInputError");
  const paceResults = document.getElementById("paceResults");

  function currentModeAPace() {
    const sec = parseFlexibleTime(paceInput.value);
    return sec;
  }

  function updateModeA() {
    const sec = currentModeAPace();
    const valid = !isNaN(sec) && sec > 0 && sec < 3600;
    paceInputError.classList.toggle("hidden", valid || paceInput.value.trim() === "");
    if (!valid) {
      paceResults.innerHTML = `<p class="text-sm text-navy-400 py-3">請輸入配速以查看完賽時間</p>`;
      renderCoach();
      return;
    }
    paceResults.innerHTML = Object.entries(DISTANCES).map(([key, d]) => {
      const t = sec * d.km;
      return `<div class="flex items-center justify-between py-3">
        <span class="text-[14px] font-medium text-navy-700">${d.label}</span>
        <span class="text-lg font-mono font-bold text-navy-950">${fmtTime(t)}</span>
      </div>`;
    }).join("");
    renderCoach();
  }
  paceInput.addEventListener("input", updateModeA);

  /* ---------------- Mode B: 時間 -> 配速 ---------------- */
  const distChipsB = document.getElementById("distChipsB");
  const timeInput = document.getElementById("timeInput");
  const timeInputError = document.getElementById("timeInputError");
  const timeQuickRow = document.getElementById("timeQuickRow");
  const requiredPaceEl = document.getElementById("requiredPace");
  let modeBDist = "half";

  function renderDistChipsB() {
    distChipsB.innerHTML = Object.entries(DISTANCES).map(([key, d]) => {
      const active = key === modeBDist;
      return `<button type="button" data-key="${key}" class="dist-chip-b tap h-12 rounded-xl text-[12px] font-bold leading-tight border-2 ${active ? "bg-navy-900 text-white border-navy-900" : "bg-white text-navy-700 border-navy-100"}">${d.label.replace('公里','K').replace('程馬拉松','馬')}</button>`;
    }).join("");
    distChipsB.querySelectorAll(".dist-chip-b").forEach(btn => {
      btn.addEventListener("click", () => {
        modeBDist = btn.dataset.key;
        state.coachDist = modeBDist;
        renderDistChipsB();
        renderTimeQuickRow();
        renderDistChipsCoach();
        updateModeB();
      });
    });
  }

  function renderTimeQuickRow() {
    const presets = TIME_PRESETS[modeBDist];
    timeQuickRow.innerHTML = presets.map(p => chipBtn(p)).join("");
    timeQuickRow.querySelectorAll(".chip").forEach((btn, i) => {
      btn.addEventListener("click", () => {
        timeInput.value = presets[i];
        updateModeB();
      });
    });
  }

  function updateModeB() {
    const sec = parseFlexibleTime(timeInput.value);
    const valid = !isNaN(sec) && sec > 0;
    timeInputError.classList.toggle("hidden", valid || timeInput.value.trim() === "");
    if (!valid) {
      requiredPaceEl.textContent = "—";
      renderCoach();
      return;
    }
    const km = DISTANCES[modeBDist].km;
    const paceSec = sec / km;
    requiredPaceEl.textContent = fmtPace(paceSec) + " /km";
    renderCoach();
  }
  timeInput.addEventListener("input", updateModeB);

  function currentModeBPace() {
    const sec = parseFlexibleTime(timeInput.value);
    if (isNaN(sec) || sec <= 0) return NaN;
    return sec / DISTANCES[modeBDist].km;
  }

  /* ---------------- Virtual Coach ---------------- */
  const distChipsCoach = document.getElementById("distChipsCoach");
  const deltaChips = document.getElementById("deltaChips");
  const coachOutput = document.getElementById("coachOutput");

  function renderDistChipsCoach() {
    distChipsCoach.innerHTML = Object.entries(DISTANCES).map(([key, d]) => {
      const active = key === state.coachDist;
      return `<button type="button" data-key="${key}" class="dist-chip-coach tap h-11 rounded-xl text-[12px] font-bold leading-tight border-2 ${active ? "bg-navy-700 text-white border-navy-700" : "bg-white text-navy-700 border-navy-100"}">${d.label.replace('公里','K').replace('程馬拉松','馬')}</button>`;
    }).join("");
    distChipsCoach.querySelectorAll(".dist-chip-coach").forEach(btn => {
      btn.addEventListener("click", () => {
        state.coachDist = btn.dataset.key;
        renderDistChipsCoach();
        renderCoach();
      });
    });
  }

  function renderStrategyButtons() {
    document.querySelectorAll(".strategy-btn").forEach(btn => {
      const active = btn.dataset.strategy === state.strategy;
      btn.classList.toggle("bg-navy-900", active);
      btn.classList.toggle("text-white", active);
      btn.classList.toggle("text-navy-500", !active);
    });
  }
  document.querySelectorAll(".strategy-btn").forEach(btn => {
    btn.addEventListener("click", () => {
      state.strategy = btn.dataset.strategy;
      renderStrategyButtons();
      renderCoach();
    });
  });

  function renderDeltaChips() {
    deltaChips.innerHTML = [3, 4, 5].map(d => {
      const active = d === state.delta;
      return `<button type="button" data-d="${d}" class="delta-chip tap h-10 rounded-xl text-[13px] font-semibold border-2 ${active ? "bg-navy-100 border-navy-700 text-navy-900" : "bg-white border-navy-100 text-navy-600"}">±${d} 秒/km</button>`;
    }).join("");
    deltaChips.querySelectorAll(".delta-chip").forEach(btn => {
      btn.addEventListener("click", () => {
        state.delta = Number(btn.dataset.d);
        renderDeltaChips();
        renderCoach();
      });
    });
  }

  function syncCoachFromMode() {
    if (state.mode === "B") state.coachDist = modeBDist;
    renderDistChipsCoach();
    renderCoach();
  }

  function activePaceForCoach() {
    return state.mode === "A" ? currentModeAPace() : currentModeBPace();
  }

  const STRATEGY_TIPS = {
    even: "全程維持穩定配速，體感強度平均分配，適合經驗尚淺或以完賽為目標的跑者。",
    negative: "前半段刻意保守、保留體力，後半段逐步加速衝刺。需要良好的配速紀律，適合追求 PB 的進階跑者。",
    positive: "前段稍快建立時間緩衝，後段轉為保守巡航。風險是後段容易掉速，建議搭配充足的體能儲備與心率監控。"
  };

  function renderCoach() {
    renderStrategyButtons();
    renderDeltaChips();
    const pace = activePaceForCoach();
    if (isNaN(pace) || pace <= 0) {
      coachOutput.innerHTML = `<p class="text-sm text-navy-400 py-2">請先在上方輸入配速或目標時間</p>`;
      return;
    }
    const km = DISTANCES[state.coachDist].km;
    const half = km / 2;
    const d = state.delta;
    let firstPace, secondPace;
    if (state.strategy === "even") { firstPace = pace; secondPace = pace; }
    else if (state.strategy === "negative") { firstPace = pace + d; secondPace = pace - d; }
    else { firstPace = pace - d; secondPace = pace + d; }

    const firstTime = half * firstPace;
    const secondTime = half * secondPace;
    const total = firstTime + secondTime;

    coachOutput.innerHTML = `
      <div class="grid grid-cols-2 gap-3">
        <div class="rounded-xl bg-navy-50 border border-navy-100 p-3">
          <p class="text-[11px] font-semibold text-navy-500 mb-1">前半程 ${half.toFixed(2)} km</p>
          <p class="text-xl font-mono font-bold text-navy-950">${fmtPace(firstPace)}<span class="text-xs font-sans font-medium text-navy-500">/km</span></p>
          <p class="text-[12px] text-navy-500 mt-1">分段時間 ${fmtTime(firstTime)}</p>
        </div>
        <div class="rounded-xl bg-navy-50 border border-navy-100 p-3">
          <p class="text-[11px] font-semibold text-navy-500 mb-1">後半程 ${half.toFixed(2)} km</p>
          <p class="text-xl font-mono font-bold text-navy-950">${fmtPace(secondPace)}<span class="text-xs font-sans font-medium text-navy-500">/km</span></p>
          <p class="text-[12px] text-navy-500 mt-1">分段時間 ${fmtTime(secondTime)}</p>
        </div>
      </div>
      <div class="rounded-xl bg-navy-900 text-white p-3 flex items-center justify-between">
        <span class="text-[13px] font-medium text-white/70">預估總時間</span>
        <span class="text-lg font-mono font-bold">${fmtTime(total)}</span>
      </div>
      <p class="text-[12.5px] leading-relaxed text-navy-600">${STRATEGY_TIPS[state.strategy]}</p>
    `;
  }

  /* ---------------- 田徑場圈速 ---------------- */
  document.querySelectorAll(".tdir-btn").forEach(btn => {
    btn.addEventListener("click", () => {
      state.trackDir = btn.dataset.tdir;
      renderTrackDir();
    });
  });
  function renderTrackDir() {
    document.querySelectorAll(".tdir-btn").forEach(btn => {
      const active = btn.dataset.tdir === state.trackDir;
      btn.classList.toggle("bg-navy-900", active);
      btn.classList.toggle("text-white", active);
      btn.classList.toggle("shadow", active);
      btn.classList.toggle("text-navy-500", !active);
    });
    document.getElementById("trackLapToPace").classList.toggle("hidden", state.trackDir !== "lapToPace");
    document.getElementById("trackPaceToLap").classList.toggle("hidden", state.trackDir !== "paceToLap");
  }

  // 400m -> pace
  const lapQuickRow = document.getElementById("lapQuickRow");
  lapQuickRow.innerHTML = LAP_PRESETS.map(p => chipBtn(p + "秒")).join("");
  lapQuickRow.querySelectorAll(".chip").forEach((btn, i) => {
    btn.addEventListener("click", () => {
      document.getElementById("lapInput").value = LAP_PRESETS[i];
      updateLapToPace();
    });
  });
  const lapInput = document.getElementById("lapInput");
  const lapInputError = document.getElementById("lapInputError");
  const lapResults = document.getElementById("lapResults");

  function updateLapToPace() {
    const sec = parseFlexibleTime(lapInput.value);
    const valid = !isNaN(sec) && sec > 0 && sec < 600;
    lapInputError.classList.toggle("hidden", valid || lapInput.value.trim() === "");
    if (!valid) {
      lapResults.innerHTML = `<p class="text-sm text-navy-400 py-3">請輸入 400m 圈速秒數</p>`;
      return;
    }
    const pacePerKm = sec * 2.5; // sec / 0.4km
    const t200 = sec / 2;
    const t800 = sec * 2;
    lapResults.innerHTML = `
      <div class="flex items-center justify-between py-3">
        <span class="text-[14px] font-medium text-navy-700">公里配速</span>
        <span class="text-lg font-mono font-bold text-navy-950">${fmtPace(pacePerKm)} /km</span>
      </div>
      <div class="flex items-center justify-between py-3">
        <span class="text-[14px] font-medium text-navy-700">200m 分段</span>
        <span class="text-lg font-mono font-bold text-navy-950">${fmtTime(t200)}</span>
      </div>
      <div class="flex items-center justify-between py-3">
        <span class="text-[14px] font-medium text-navy-700">800m 分段</span>
        <span class="text-lg font-mono font-bold text-navy-950">${fmtTime(t800)}</span>
      </div>
    `;
  }
  lapInput.addEventListener("input", updateLapToPace);

  // pace -> 400m
  const paceQuickRow2 = document.getElementById("paceQuickRow2");
  paceQuickRow2.innerHTML = PACE_PRESETS.map(p => chipBtn(p)).join("");
  paceQuickRow2.querySelectorAll(".chip").forEach((btn, i) => {
    btn.addEventListener("click", () => {
      document.getElementById("paceInput2").value = PACE_PRESETS[i];
      updatePaceToLap();
    });
  });
  const paceInput2 = document.getElementById("paceInput2");
  const paceInput2Error = document.getElementById("paceInput2Error");
  const lapResults2 = document.getElementById("lapResults2");

  function updatePaceToLap() {
    const sec = parseFlexibleTime(paceInput2.value);
    const valid = !isNaN(sec) && sec > 0 && sec < 3600;
    paceInput2Error.classList.toggle("hidden", valid || paceInput2.value.trim() === "");
    if (!valid) {
      lapResults2.innerHTML = `<p class="text-sm text-navy-400 py-3">請輸入公里配速</p>`;
      return;
    }
    const lapTime = sec * 0.4;
    const t200 = lapTime / 2;
    const t800 = lapTime * 2;
    lapResults2.innerHTML = `
      <div class="flex items-center justify-between py-3">
        <span class="text-[14px] font-medium text-navy-700">400m 一圈</span>
        <span class="text-lg font-mono font-bold text-navy-950">${fmtTime(lapTime)}</span>
      </div>
      <div class="flex items-center justify-between py-3">
        <span class="text-[14px] font-medium text-navy-700">200m 分段</span>
        <span class="text-lg font-mono font-bold text-navy-950">${fmtTime(t200)}</span>
      </div>
      <div class="flex items-center justify-between py-3">
        <span class="text-[14px] font-medium text-navy-700">800m 分段</span>
        <span class="text-lg font-mono font-bold text-navy-950">${fmtTime(t800)}</span>
      </div>
    `;
  }
  paceInput2.addEventListener("input", updatePaceToLap);

  /* ---------------- Init ---------------- */
  renderNav();
  renderModeButtons();
  renderDistChipsB();
  renderTimeQuickRow();
  renderDistChipsCoach();
  renderTrackDir();

  paceInput.value = "5:00";
  timeInput.value = TIME_PRESETS[modeBDist][1];
  lapInput.value = "110";
  paceInput2.value = "5:00";

  updateModeA();
  updateModeB();
  updateLapToPace();
  updatePaceToLap();
})();
</script>
</body>
</html>
