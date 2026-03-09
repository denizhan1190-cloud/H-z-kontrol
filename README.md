<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="theme-color" content="#020617">
<meta name="description" content="HızKontrol - Akıllı Sürüş Asistanı. EDS uyarıları, hız limiti takibi, ticari/normal araç ayrımı.">
<title>HızKontrol - Akıllı Sürüş Asistanı</title>
<link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiSMSxektvbnRyb2wiLCJzaG9ydF9uYW1lIjoiSMSxektvbnRyb2wiLCJzdGFydF91cmwiOiIuIiwiZGlzcGxheSI6InN0YW5kYWxvbmUiLCJiYWNrZ3JvdW5kX2NvbG9yIjoiIzAyMDYxNyIsInRoZW1lX2NvbG9yIjoiIzAyMDYxNyJ9">
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🚗</text></svg>">
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=JetBrains+Mono:wght@400;600;700&display=swap" rel="stylesheet">
<style>
*{box-sizing:border-box;margin:0;padding:0}
html,body{height:100%;overflow:hidden}
body{font-family:'DM Sans',sans-serif;background:#020617;color:#e2e8f0}
#app{height:100%;overflow-y:auto;overflow-x:hidden;-webkit-overflow-scrolling:touch}
::-webkit-scrollbar{width:4px;height:4px}
::-webkit-scrollbar-thumb{background:#1e293b;border-radius:4px}
@keyframes pulseRing{0%{transform:scale(1);opacity:.6}100%{transform:scale(1.35);opacity:0}}
@keyframes slideDown{0%{transform:translateY(-16px);opacity:0}100%{transform:translateY(0);opacity:1}}
@keyframes flashBorder{0%,100%{border-color:#ef4444}50%{border-color:#ef444444}}
@keyframes blink{0%,100%{opacity:1}50%{opacity:.3}}
@keyframes pulseShadow{0%,100%{box-shadow:0 0 20px rgba(239,68,68,.5)}50%{box-shadow:0 0 40px rgba(239,68,68,.8)}}
@keyframes fadeIn{0%{opacity:0;transform:translateY(10px)}100%{opacity:1;transform:translateY(0)}}
@keyframes spin{to{transform:rotate(360deg)}}
.btn{cursor:pointer;transition:all .2s;font-family:'DM Sans',sans-serif;outline:none;-webkit-tap-highlight-color:transparent}
.btn:active{transform:scale(.97)!important}
</style>
</head>
<body>
<div id="app"></div>
<script>
// PWA Service Worker Kaydı
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('./sw.js')
      .then(reg => console.log('Service Worker kaydedildi.', reg.scope))
      .catch(err => console.log('Service Worker kayıt hatası:', err));
  });
}

// ═══════════════════════════════════════
// HızKontrol v5.1 — Standalone PWA + WakeLock + Offline
// ═══════════════════════════════════════

const ROAD_SEGMENTS=[
{id:"s1",name:"Ankara Blv. Kızılay",signLimit:50,normalLimit:50,ticariLimit:50,roadType:"sehirIci",city:"Ankara",lat:39.9208,lng:32.8541},
{id:"s2",name:"Bağdat Cad. Kadıköy",signLimit:50,normalLimit:50,ticariLimit:50,roadType:"sehirIci",city:"İstanbul",lat:40.9632,lng:29.0642},
{id:"s3",name:"İstiklal Caddesi",signLimit:30,normalLimit:30,ticariLimit:30,roadType:"sehirIci",city:"İstanbul",lat:41.0335,lng:28.977},
{id:"s4",name:"Kordon Boyu",signLimit:50,normalLimit:50,ticariLimit:50,roadType:"sehirIci",city:"İzmir",lat:38.435,lng:27.142},
{id:"s5",name:"Küçükçekmece Sahil",signLimit:40,normalLimit:40,ticariLimit:40,roadType:"sehirIci",city:"İstanbul",lat:41.0048,lng:28.7734},
{id:"s6",name:"Tunalı Hilmi Cad.",signLimit:50,normalLimit:50,ticariLimit:50,roadType:"sehirIci",city:"Ankara",lat:39.9042,lng:32.861},
{id:"s7",name:"Atatürk Bulvarı",signLimit:70,normalLimit:70,ticariLimit:60,roadType:"sehirIci",city:"Antalya",lat:36.887,lng:30.7035},
{id:"d1",name:"E-5 Bakırköy",signLimit:82,normalLimit:82,ticariLimit:75,roadType:"sehirDisi",city:"İstanbul",lat:40.9808,lng:28.8772},
{id:"d2",name:"D-100 Pendik",signLimit:70,normalLimit:70,ticariLimit:65,roadType:"sehirDisi",city:"İstanbul",lat:40.8781,lng:29.2322},
{id:"d3",name:"Konya Yolu Gölbaşı",signLimit:90,normalLimit:90,ticariLimit:80,roadType:"sehirDisi",city:"Ankara",lat:39.7881,lng:32.7991},
{id:"d4",name:"Antalya Lara Yolu",signLimit:70,normalLimit:70,ticariLimit:65,roadType:"sehirDisi",city:"Antalya",lat:36.8529,lng:30.7278},
{id:"d5",name:"Eskişehir Çevreyolu",signLimit:82,normalLimit:82,ticariLimit:72,roadType:"sehirDisi",city:"Eskişehir",lat:39.7767,lng:30.5206},
{id:"d6",name:"D-550 Balıkesir",signLimit:90,normalLimit:90,ticariLimit:80,roadType:"sehirDisi",city:"Balıkesir",lat:39.6484,lng:27.8826},
{id:"d7",name:"Trabzon Sahil",signLimit:60,normalLimit:60,ticariLimit:55,roadType:"sehirDisi",city:"Trabzon",lat:41.0015,lng:39.7178},
{id:"o1",name:"TEM Otoyol Gebze",signLimit:120,normalLimit:120,ticariLimit:90,roadType:"otoyol",city:"Kocaeli",lat:40.8027,lng:29.4314},
{id:"o2",name:"FSM Köprüsü",signLimit:90,normalLimit:90,ticariLimit:80,roadType:"otoyol",city:"İstanbul",lat:41.0905,lng:29.0592},
{id:"o3",name:"O-4 Otoyol Bolu",signLimit:120,normalLimit:120,ticariLimit:90,roadType:"otoyol",city:"Bolu",lat:40.7356,lng:31.6089},
{id:"o4",name:"İzmit Körfez Geçişi",signLimit:110,normalLimit:110,ticariLimit:85,roadType:"otoyol",city:"Kocaeli",lat:40.7654,lng:29.9408},
{id:"o5",name:"Osmangazi Köprüsü",signLimit:110,normalLimit:110,ticariLimit:85,roadType:"otoyol",city:"Bursa",lat:40.586,lng:29.0438},
{id:"o6",name:"O-52 Gaziantep",signLimit:120,normalLimit:120,ticariLimit:90,roadType:"otoyol",city:"Gaziantep",lat:37.0662,lng:37.3833},
{id:"o7",name:"Mersin Otoyol",signLimit:120,normalLimit:120,ticariLimit:90,roadType:"otoyol",city:"Mersin",lat:36.8121,lng:34.6415},
{id:"o8",name:"Ankara-İstanbul O-4",signLimit:140,normalLimit:140,ticariLimit:95,roadType:"otoyol",city:"Ankara",lat:40.15,lng:30.5},
];

const EDS_DATABASE=[
{id:1,name:"Ankara Blv. EDS",lat:39.921,lng:32.8545,speedLimit:50,type:"eds",city:"Ankara",segmentId:"s1"},
{id:2,name:"E-5 Bakırköy Radar",lat:40.9812,lng:28.8775,speedLimit:82,type:"radar",city:"İstanbul",segmentId:"d1"},
{id:3,name:"TEM Gebze EDS",lat:40.803,lng:29.4318,speedLimit:120,type:"eds",city:"Kocaeli",segmentId:"o1"},
{id:4,name:"FSM Köprüsü EDS",lat:41.0908,lng:29.0595,speedLimit:90,type:"eds",city:"İstanbul",segmentId:"o2"},
{id:5,name:"D-100 Pendik Radar",lat:40.8784,lng:29.2325,speedLimit:70,type:"radar",city:"İstanbul",segmentId:"d2"},
{id:6,name:"Bağdat Cad. EDS",lat:40.9635,lng:29.0645,speedLimit:50,type:"eds",city:"İstanbul",segmentId:"s2"},
{id:7,name:"O-4 Bolu Radar",lat:40.736,lng:31.6092,speedLimit:120,type:"radar",city:"Bolu",segmentId:"o3"},
{id:8,name:"Mecidiyeköy EDS",lat:41.0676,lng:28.9922,speedLimit:50,type:"eds",city:"İstanbul",segmentId:"s2"},
{id:9,name:"Körfez Geçişi Radar",lat:40.7658,lng:29.9412,speedLimit:110,type:"radar",city:"Kocaeli",segmentId:"o4"},
{id:10,name:"Küçükçekmece EDS",lat:41.0052,lng:28.7738,speedLimit:60,type:"eds",city:"İstanbul",segmentId:"s5"},
{id:11,name:"Konya Yolu Radar",lat:39.7885,lng:32.7995,speedLimit:82,type:"radar",city:"Ankara",segmentId:"d3"},
{id:12,name:"Mersin Otoyol EDS",lat:36.8125,lng:34.6418,speedLimit:120,type:"eds",city:"Mersin",segmentId:"o7"},
{id:13,name:"İzmir Konak EDS",lat:38.4195,lng:27.129,speedLimit:50,type:"eds",city:"İzmir",segmentId:"s4"},
{id:14,name:"Lara Yolu Radar",lat:36.8532,lng:30.7282,speedLimit:70,type:"radar",city:"Antalya",segmentId:"d4"},
{id:15,name:"Osmangazi EDS",lat:40.5863,lng:29.0442,speedLimit:110,type:"eds",city:"Bursa",segmentId:"o5"},
];

const CITIES=[
{name:"İstanbul",lat:41.0082,lng:28.9784},{name:"Ankara",lat:39.9334,lng:32.8597},
{name:"İzmir",lat:38.4237,lng:27.1428},{name:"Antalya",lat:36.8969,lng:30.7133},
{name:"Bursa",lat:40.1885,lng:29.061},{name:"Trabzon",lat:41.0027,lng:39.7168},
{name:"Kocaeli",lat:40.7654,lng:29.9408},{name:"Eskişehir",lat:39.7767,lng:30.5206},
{name:"Gaziantep",lat:37.0662,lng:37.3833},{name:"Mersin",lat:36.8121,lng:34.6415},
];

const ROAD_TYPES=[{id:"sehirIci",label:"Şehir İçi",icon:"🏙️"},{id:"sehirDisi",label:"Şehir Dışı",icon:"🛣️"},{id:"otoyol",label:"Otoyol",icon:"🛤️"}];

function haversine(a,b,c,d){const R=6371,x=((c-a)*Math.PI)/180,y=((d-b)*Math.PI)/180,z=Math.sin(x/2)**2+Math.cos((a*Math.PI)/180)*Math.cos((c*Math.PI)/180)*Math.sin(y/2)**2;return R*2*Math.atan2(Math.sqrt(z),Math.sqrt(1-z))}

function speak(t,on){if(!on)return;try{const s=speechSynthesis;s.cancel();const u=new SpeechSynthesisUtterance(t);u.lang="tr-TR";u.rate=1.1;const v=s.getVoices().find(v=>v.lang.startsWith("tr"));if(v)u.voice=v;s.speak(u)}catch(e){}}

// ═══ EKRAN UYANIK TUTMA (WAKE LOCK) ═══
let wakeLock = null;
async function requestWakeLock() {
  try {
    if ('wakeLock' in navigator) {
      wakeLock = await navigator.wakeLock.request('screen');
      console.log('Ekran uyanık tutma aktif.');
    }
  } catch (err) {
    console.error(`Wake Lock Hatası: ${err.name}, ${err.message}`);
  }
}
function releaseWakeLock() {
  if (wakeLock !== null) {
    wakeLock.release().then(() => { wakeLock = null; console.log('Ekran uyanık tutma devredışı.'); });
  }
}
// Uygulama alta alınıp geri açıldığında ekran kilidini yenile
document.addEventListener('visibilitychange', async () => {
  if (wakeLock !== null && document.visibilityState === 'visible' && S.simulating) {
    await requestWakeLock();
  }
});


// ═══ APP STATE ═══
const S={
  screen:"select",vehicleType:null,roadType:"sehirIci",speed:0,simulating:false,
  alerts:[],traveledKm:0,theme:"dark",voice:true,gpsStatus:"inactive",gpsError:null,
  userLoc:null,city:null,maxSpeed:0,avgSum:0,avgCount:0,edsDist:{},
  segment:null,showGPS:false,alertId:0
};

let simLoc={lat:41.0082,lng:28.9784};
let gpsWatch=null;
let simInterval=null;
let edsInterval=null;
let segInterval=null;
let spokenEds=new Set();
let spokenOver=false;
let spokenSign=null;
let mapCanvas=null;
let mapCtx=null;
let mapOffset={x:0,y:0};
let mapTrail=[];
let mapFrame=null;

function getTheme(){
  const dk={bg:"#020617",bg2:"#0a1628",card:"#0c1322",cardB:"#111d33",border:"#1e293b",text:"#e2e8f0",text2:"#94a3b8",text3:"#64748b",gauge:"#0f172a",acc:"#3b82f6",accL:"#60a5fa",mapBg:"#070e1a",mapRd:"#1a2744",mapRdS:"#253a5c",mapGr:"#0d1929",mapW:"#0a1930",rdSurf:"#1a2a42",rdDash:"#3a5070",bldg:"#0f1d30",glassA:"rgba(2,6,23,.88)",glassB:"rgba(10,22,40,.92)",glassC:"rgba(2,6,23,.96)",glassBdr:"rgba(30,41,59,.6)",glassSh:"rgba(0,0,0,.4)",limBg:"rgba(0,0,0,.3)",inBg:"#0f1d30",inBdr:"#1e3a5f"};
  const lt={bg:"#f0f4f8",bg2:"#e2e8f0",card:"#ffffff",cardB:"#e2e8f0",border:"#cbd5e1",text:"#1e293b",text2:"#475569",text3:"#94a3b8",gauge:"#e2e8f0",acc:"#2563eb",accL:"#3b82f6",mapBg:"#e8eef6",mapRd:"#c8d4e4",mapRdS:"#b0bed2",mapGr:"#dce4f0",mapW:"#c4d8f0",rdSurf:"#b8c8dc",rdDash:"#90a4be",bldg:"#c8d4e4",glassA:"rgba(240,244,248,.88)",glassB:"rgba(226,232,240,.92)",glassC:"rgba(240,244,248,.96)",glassBdr:"rgba(203,213,225,.6)",glassSh:"rgba(0,0,0,.1)",limBg:"rgba(255,255,255,.5)",inBg:"#f8fafc",inBdr:"#cbd5e1"};
  return S.theme==="dark"?dk:lt;
}

// ═══ RENDER ═══
function render(){
  const app=document.getElementById("app");
  const T=getTheme();
  app.style.background=`linear-gradient(180deg,${T.bg} 0%,${T.bg2} 50%,${T.bg} 100%)`;

  if(S.screen==="select"){renderSelect(app,T);return;}
  renderDashboard(app,T);
}

function renderSelect(app,T){
  app.innerHTML=`
  <div style="min-height:100vh;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:40px 20px;gap:24px;animation:fadeIn .5s ease">
    <button class="btn" onclick="toggleTheme()" style="position:fixed;top:16px;right:16px;padding:8px 16px;border-radius:24px;font-size:13px;font-weight:600;background:${T.card};border:1px solid ${T.border};color:${T.text};z-index:10">${S.theme==="dark"?"☀️ Aydınlık":"🌙 Karanlık"}</button>
    <div style="display:flex;flex-direction:column;align-items:center;gap:10px">
      <div style="width:80px;height:80px;border-radius:50%;background:${T.acc}18;border:2px solid ${T.acc}55;display:flex;align-items:center;justify-content:center">
        <svg width="44" height="44" viewBox="0 0 24 24" fill="none" stroke="${T.accL}" stroke-width="1.5"><circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/></svg>
      </div>
      <h1 style="font-size:30px;font-weight:700;background:linear-gradient(135deg,${T.accL},#a78bfa);-webkit-background-clip:text;-webkit-text-fill-color:transparent">HızKontrol</h1>
      <p style="color:${T.text3};font-size:12px;letter-spacing:.08em;text-transform:uppercase">Akıllı Sürüş Asistanı v5.1</p>
    </div>
    <div style="display:flex;gap:6px;flex-wrap:wrap;justify-content:center;max-width:380px">
      ${["📍 GPS","🔊 Sesli Uyarı","📸 EDS","🌓 Tema","🚧 Tabela","🗺️ Harita"].map(f=>`<span style="padding:4px 11px;border-radius:20px;background:${T.card};border:1px solid ${T.cardB};font-size:11px;color:${T.text2}">${f}</span>`).join("")}
    </div>
    <p style="font-size:15px;font-weight:600;color:${T.text2}">Araç Sınıfınızı Seçin</p>
    <div style="display:flex;gap:14px;width:100%;max-width:400px">
      <button class="btn" onclick="selectVehicle('normal')" style="flex:1;background:${T.card};border:2px solid ${T.border};border-radius:18px;padding:22px 12px 16px;display:flex;flex-direction:column;align-items:center;gap:6px;color:${T.text}">
        <div style="font-size:44px">🚗</div>
        <div style="font-size:15px;font-weight:700">Normal Araç</div>
        <div style="font-size:10px;color:${T.text3}">Binek, SUV, sedan</div>
        <div style="margin-top:4px;padding:4px 10px;border-radius:10px;background:#3b82f615;border:1px solid #3b82f633;font-size:10px;color:#3b82f6;font-weight:500">Tabela 120 → Siz 120</div>
      </button>
      <button class="btn" onclick="selectVehicle('ticari')" style="flex:1;background:${T.card};border:2px solid ${T.border};border-radius:18px;padding:22px 12px 16px;display:flex;flex-direction:column;align-items:center;gap:6px;color:${T.text}">
        <div style="font-size:44px">🚛</div>
        <div style="font-size:15px;font-weight:700">Ticari Araç</div>
        <div style="font-size:10px;color:${T.text3}">Kamyon, minibüs, otobüs</div>
        <div style="margin-top:4px;padding:4px 10px;border-radius:10px;background:#f59e0b15;border:1px solid #f59e0b33;font-size:10px;color:#f59e0b;font-weight:500">Tabela 120 → Siz 90</div>
      </button>
    </div>
  </div>`;
}

function renderDashboard(app,T){
  const segs=ROAD_SEGMENTS.filter(s=>s.roadType===S.roadType);
  const seg=S.segment||segs[0];
  const signLim=seg?seg.signLimit:50;
  const yourLim=seg?(S.vehicleType==="ticari"?seg.ticariLimit:seg.normalLimit):50;
  const ratio=yourLim>0?S.speed/yourLim:0;
  const over=S.speed>yourLim;
  const sCol=ratio<.7?"#10b981":ratio<.95?"#f59e0b":"#ef4444";
  const avgSpd=S.avgCount>0?Math.round(S.avgSum/S.avgCount):0;
  const eds=EDS_DATABASE.map(p=>({...p,distance:S.edsDist[p.id]!==undefined?S.edsDist[p.id]:99})).sort((a,b)=>a.distance-b.distance).slice(0,5);
  const gpsCol={inactive:"#64748b",searching:"#f59e0b",active:"#10b981",error:"#ef4444",manual:"#a78bfa",simulated:"#a78bfa"}[S.gpsStatus];
  const gpsLbl={inactive:"GPS Bağla →",searching:"Aranıyor...",active:"GPS Aktif ✓",error:"GPS Hatası ✕",manual:"📍 "+(S.city||"Manuel"),simulated:"Simülasyon"}[S.gpsStatus];

  let html=`
  <div style="display:flex;align-items:center;justify-content:space-between;padding:12px 20px;border-bottom:1px solid ${T.border}22">
    <button class="btn" onclick="goBack()" style="background:none;border:none;color:${T.accL};font-size:14px;font-weight:600">← Geri</button>
    <div style="display:flex;align-items:center;gap:6px">
      <span style="font-size:16px">${S.vehicleType==="ticari"?"🚛":"🚗"}</span>
      <span style="font-size:14px;font-weight:600;color:${T.text}">${S.vehicleType==="ticari"?"Ticari":"Normal"}</span>
    </div>
    <div style="display:flex;gap:8px;align-items:center">
      <button class="btn" onclick="toggleVoice()" style="background:none;border:none;font-size:18px;opacity:${S.voice?1:.4}">🔊</button>
      <button class="btn" onclick="toggleTheme()" style="background:none;border:none;font-size:16px">${S.theme==="dark"?"☀️":"🌙"}</button>
    </div>
  </div>

  <div style="padding:8px 20px 0;display:flex;justify-content:center">
    <button class="btn" onclick="toggleGPSPanel()" style="display:flex;align-items:center;gap:8px;padding:6px 14px;border-radius:12px;background:${T.card};border:1.5px solid ${S.gpsStatus==="active"?"#10b981":S.gpsStatus==="error"?"#ef4444":T.cardB};color:${T.text}">
      <div style="width:8px;height:8px;border-radius:50%;background:${gpsCol};${S.gpsStatus==="searching"?"animation:blink 1s infinite":""}"></div>
      <span style="font-size:12px;font-weight:600;color:${S.gpsStatus==="active"?"#10b981":S.gpsStatus==="error"?"#ef4444":T.text2}">${gpsLbl}</span>
      <span style="font-size:10px;color:${T.text3}">${S.showGPS?"▲":"▼"}</span>
    </button>
  </div>`;

  // GPS Panel
  if(S.showGPS){
    html+=`<div style="margin:8px 20px 0;padding:16px;background:${T.card};border:1.5px solid ${T.cardB};border-radius:16px;animation:slideDown .3s ease">
      <div style="display:flex;align-items:center;gap:10px;margin-bottom:12px">
        <div style="width:12px;height:12px;border-radius:50%;background:${gpsCol};box-shadow:0 0 8px ${gpsCol}88"></div>
        <span style="font-size:14px;font-weight:700;color:${T.text}">${{inactive:"GPS Bağlı Değil",searching:"GPS Aranıyor...",active:"GPS Bağlandı ✓",error:"GPS Hatası",manual:"Manuel Konum",simulated:"Simülasyon Modu"}[S.gpsStatus]}</span>
      </div>
      ${S.gpsError?`<div style="padding:8px 12px;border-radius:10px;background:rgba(239,68,68,.1);border:1px solid rgba(239,68,68,.3);margin-bottom:10px;font-size:12px;color:#ef4444">⚠️ ${S.gpsError}</div>`:""}
      ${S.userLoc?`<div style="padding:6px 10px;border-radius:8px;background:rgba(16,185,129,.1);border:1px solid rgba(16,185,129,.3);margin-bottom:10px;font-size:11px;color:#10b981;font-family:monospace">📍 ${S.userLoc.lat.toFixed(4)}, ${S.userLoc.lng.toFixed(4)} ${S.city?`(${S.city})`:""}</div>`:""}
      <button class="btn" onclick="requestGPS()" style="width:100%;padding:12px;border-radius:12px;border:none;background:${S.gpsStatus==="active"?"rgba(16,185,129,.15)":"linear-gradient(135deg,#2563eb,#3b82f6)"};color:${S.gpsStatus==="active"?"#10b981":"#fff"};font-size:14px;font-weight:700;margin-bottom:10px">
        ${S.gpsStatus==="searching"?"⏳ Bağlanıyor...":S.gpsStatus==="active"?"✅ GPS Bağlı":"📍 GPS İzni Ver ve Bağlan"}
      </button>
      <div style="text-align:center;font-size:11px;color:${T.text3};margin-bottom:10px">── veya şehir seçin ──</div>
      <div style="display:flex;flex-wrap:wrap;gap:6px;margin-bottom:10px">
        ${CITIES.map(c=>`<button class="btn" onclick="selectCity('${c.name}',${c.lat},${c.lng})" style="padding:6px 12px;border-radius:20px;border:1.5px solid ${S.city===c.name?T.acc:T.border};background:${S.city===c.name?T.acc+"22":"transparent"};color:${S.city===c.name?T.accL:T.text2};font-size:11px;font-weight:600">${c.name}</button>`).join("")}
      </div>
      <div style="padding:8px 12px;border-radius:10px;background:${T.acc}0a;border:1px solid ${T.acc}22;font-size:11px;color:${T.text3};line-height:1.5">💡 <b>İpucu:</b> Tam GPS için uygulamayı telefon tarayıcısında açın veya şehir seçerek başlayın.</div>
    </div>`;
  }

  // Road Type
  html+=`<div style="display:flex;gap:8px;padding:8px 20px;justify-content:center">
    ${ROAD_TYPES.map(r=>`<button class="btn" onclick="setRoad('${r.id}')" style="display:flex;flex-direction:column;align-items:center;gap:3px;padding:8px 18px;border-radius:12px;border:1.5px solid ${S.roadType===r.id?T.acc:T.border};background:${S.roadType===r.id?T.acc+"22":"transparent"};color:${S.roadType===r.id?T.accL:T.text3}"><span style="font-size:16px">${r.icon}</span><span style="font-size:11px;font-weight:600">${r.label}</span></button>`).join("")}
  </div>`;

  // Alerts
  if(S.alerts.length){
    html+=`<div style="display:flex;flex-direction:column;gap:8px;padding:0 20px;max-height:160px;overflow:hidden">
      ${S.alerts.map(a=>{const bg=a.type==="danger"?"rgba(239,68,68,.15)":a.type==="warning"?"rgba(245,158,11,.15)":"rgba(16,185,129,.15)";const bc=a.type==="danger"?"#ef4444":a.type==="warning"?"#f59e0b":"#10b981";const ic=a.type==="danger"?"🚨":a.type==="warning"?"⚠️":"✅";return`<div onclick="removeAlert(${a.id})" style="background:${bg};border:1px solid ${bc};border-radius:14px;padding:12px 16px;display:flex;align-items:center;gap:10px;animation:slideDown .4s ease;cursor:pointer"><span style="font-size:20px">${ic}</span><span style="color:${T.text};font-size:12px;flex:1">${a.message}</span><span style="color:${T.text3};font-size:10px">✕</span></div>`}).join("")}
    </div>`;
  }

  // Speedometer + Map
  const circ=2*Math.PI*95;
  const dashOff=circ-(Math.min(ratio,1.3)/1.3)*circ;
  html+=`<div style="display:flex;justify-content:center;padding:10px 0 4px">
    <div style="position:relative;width:300px;height:300px;display:flex;align-items:center;justify-content:center">
      <canvas id="minimap" width="300" height="300" style="position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);border-radius:50%;opacity:.5;width:300px;height:300px"></canvas>
      <div style="position:absolute;width:230px;height:230px;border-radius:50%;background:radial-gradient(circle,${T.glassA} 0%,${T.glassB} 70%,${T.glassC} 100%);backdrop-filter:blur(6px);border:2px solid ${T.glassBdr};box-shadow:0 0 40px ${T.glassSh};z-index:2"></div>
      <svg width="230" height="230" viewBox="0 0 230 230" style="position:absolute;z-index:3">
        <circle cx="115" cy="115" r="95" fill="none" stroke="${T.gauge}" stroke-width="10" opacity=".5"/>
        <circle cx="115" cy="115" r="95" fill="none" stroke="${sCol}" stroke-width="10" stroke-linecap="round" stroke-dasharray="${circ}" stroke-dashoffset="${dashOff}" transform="rotate(-90 115 115)" style="transition:stroke-dashoffset .3s,stroke .3s;filter:drop-shadow(0 0 10px ${sCol}66)"/>
      </svg>
      <div style="position:relative;z-index:4;display:flex;flex-direction:column;align-items:center">
        <div style="font-size:60px;font-weight:700;font-family:'JetBrains Mono',monospace;line-height:1;color:${sCol};text-shadow:0 0 20px ${sCol}33;transition:color .3s">${S.speed}</div>
        <div style="font-size:12px;color:${T.text3}">km/s</div>
        <div style="margin-top:5px;padding:3px 10px;border-radius:20px;border:1.5px solid ${sCol}44;font-size:11px;font-weight:600;color:${T.text2};font-family:monospace;background:${T.limBg}">Limit: ${yourLim}</div>
      </div>
      ${over?`<div style="position:absolute;inset:-8px;border-radius:50%;border:2px solid #ef4444;opacity:0;animation:pulseRing 2s ease-out infinite;z-index:5"></div><div style="position:absolute;inset:-8px;border-radius:50%;border:2px solid #ef4444;opacity:0;animation:pulseRing 2s ease-out .7s infinite;z-index:5"></div>`:""}
      <div style="position:absolute;bottom:4px;left:50%;transform:translateX(-50%);z-index:5;font-size:8px;color:${T.text3};background:${T.card}cc;padding:2px 8px;border-radius:6px;font-weight:600">🗺️ CANLI HARİTA</div>
    </div>
  </div>`;

  // Speed Sign
  if(seg){
    const diff=signLim!==yourLim;
    html+=`<div style="display:flex;align-items:center;gap:14px;padding:12px 16px;background:${T.card};border:1px solid ${T.cardB};border-radius:16px;margin:4px 20px 0">
      <div style="position:relative;flex-shrink:0">
        <div style="width:68px;height:68px;border-radius:50%;border:5px solid #ef4444;background:#fff;display:flex;flex-direction:column;align-items:center;justify-content:center;box-shadow:${over?"0 0 20px rgba(239,68,68,.5)":"0 2px 12px rgba(0,0,0,.2)"};${over?"animation:pulseShadow 1.5s infinite":""}">
          <span style="font-size:22px;font-weight:800;color:#1a1a1a;font-family:monospace;line-height:1">${signLim}</span>
          <span style="font-size:7px;color:#555;font-weight:600">km/s</span>
        </div>
        <div style="position:absolute;bottom:-4px;right:-4px;width:22px;height:22px;border-radius:50%;background:${S.vehicleType==="ticari"?"#f59e0b":"#3b82f6"};display:flex;align-items:center;justify-content:center;font-size:11px;border:2px solid ${T.card}">${S.vehicleType==="ticari"?"🚛":"🚗"}</div>
      </div>
      <div style="flex:1">
        <div style="font-size:10px;color:${T.text3};font-weight:600;text-transform:uppercase;margin-bottom:2px">Yol Tabelası</div>
        <div style="font-size:13px;color:${T.text};font-weight:600;margin-bottom:4px">${seg.name} • ${seg.city}</div>
        <div style="display:flex;gap:8px;flex-wrap:wrap">
          <span style="font-size:11px;color:${T.text2}">Tabela: <b>${signLim}</b></span>
          ${diff?`<span style="font-size:11px;color:${S.vehicleType==="ticari"?"#f59e0b":"#3b82f6"};font-weight:700">Siz: <b>${yourLim}</b></span>`:`<span style="font-size:11px;color:#10b981">Aynı limit</span>`}
        </div>
        ${diff&&S.vehicleType==="ticari"?`<div style="margin-top:4px;padding:3px 8px;border-radius:6px;background:rgba(245,158,11,.12);font-size:10px;color:#f59e0b">⚠ Ticari: ${signLim-yourLim} km/s daha yavaş</div>`:""}
      </div>
    </div>`;

    // Comparison
    html+=`<div style="display:flex;gap:6px;padding:6px 20px 0">
      <div style="flex:1;padding:8px;border-radius:12px;text-align:center;background:${S.vehicleType==="normal"?"rgba(59,130,246,.12)":T.card};border:1.5px solid ${S.vehicleType==="normal"?"#3b82f6":T.cardB}">
        <div style="font-size:9px;color:${T.text3};font-weight:600;text-transform:uppercase">🚗 Normal</div>
        <div style="font-size:20px;font-weight:800;font-family:monospace;color:${S.vehicleType==="normal"?"#3b82f6":T.text2}">${seg.normalLimit}</div>
      </div>
      <div style="display:flex;align-items:center"><span style="font-size:10px;color:${T.text3};font-weight:700">VS</span></div>
      <div style="flex:1;padding:8px;border-radius:12px;text-align:center;background:${S.vehicleType==="ticari"?"rgba(245,158,11,.12)":T.card};border:1.5px solid ${S.vehicleType==="ticari"?"#f59e0b":T.cardB}">
        <div style="font-size:9px;color:${T.text3};font-weight:600;text-transform:uppercase">🚛 Ticari</div>
        <div style="font-size:20px;font-weight:800;font-family:monospace;color:${S.vehicleType==="ticari"?"#f59e0b":T.text2}">${seg.ticariLimit}</div>
      </div>
    </div>`;
  }

  // Stats
  html+=`<div style="display:flex;gap:6px;padding:6px 20px 0">
    ${[{i:"📏",v:S.traveledKm.toFixed(1),l:"km"},{i:"⚡",v:S.maxSpeed,l:"Maks",c:S.maxSpeed>yourLim?"#ef4444":"#10b981"},{i:"📊",v:avgSpd,l:"Ort."},{i:"📡",v:eds.filter(e=>e.distance<30).length,l:"EDS"}].map(s=>`<div style="flex:1;background:${T.card};border:1px solid ${T.cardB};border-radius:12px;padding:8px 4px;display:flex;flex-direction:column;align-items:center;gap:2px"><span style="font-size:13px">${s.i}</span><span style="font-size:14px;font-weight:700;font-family:monospace;color:${s.c||T.text}">${s.v}</span><span style="font-size:8px;color:${T.text3};text-transform:uppercase">${s.l}</span></div>`).join("")}
  </div>`;

  // Road Segments
  html+=`<div style="padding:8px 20px 4px"><div style="font-size:11px;font-weight:700;color:${T.text2};text-transform:uppercase;margin-bottom:6px">Yol Bölgeleri</div>
    <div style="display:flex;gap:6px;overflow-x:auto;padding-bottom:6px;-webkit-overflow-scrolling:touch">
      ${segs.map(s=>{const isA=seg&&seg.id===s.id;const lim=S.vehicleType==="ticari"?s.ticariLimit:s.normalLimit;return`<div style="min-width:105px;padding:8px 10px;border-radius:12px;flex-shrink:0;background:${isA?(S.vehicleType==="ticari"?"rgba(245,158,11,.12)":"rgba(59,130,246,.12)"):T.card};border:1.5px solid ${isA?(S.vehicleType==="ticari"?"#f59e0b":"#3b82f6"):T.cardB}"><div style="font-size:9px;color:${T.text3};font-weight:600">${s.city}</div><div style="font-size:10px;color:${T.text};font-weight:600;margin-bottom:4px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis">${s.name}</div><div style="display:flex;justify-content:space-between;align-items:center"><div style="width:26px;height:26px;border-radius:50%;border:2.5px solid #ef4444;background:#fff;display:flex;align-items:center;justify-content:center"><span style="font-size:9px;font-weight:800;color:#1a1a1a;font-family:monospace">${s.signLimit}</span></div><div style="text-align:right"><div style="font-size:7px;color:${T.text3}">Siz</div><div style="font-size:13px;font-weight:800;font-family:monospace;color:${lim<s.signLimit?"#f59e0b":"#10b981"}">${lim}</div></div></div></div>`}).join("")}
    </div>
  </div>`;

  // EDS
  const nearby=eds.filter(e=>e.distance<30).slice(0,4);
  if(nearby.length){
    html+=`<div style="padding:4px 20px"><div style="font-size:11px;font-weight:700;color:${T.text2};text-transform:uppercase;margin-bottom:6px">EDS / Radar</div>
      <div style="display:flex;flex-direction:column;gap:6px">
        ${nearby.map(p=>{const ic=p.distance<2,iv=p.distance<.5;const sg=ROAD_SEGMENTS.find(s=>s.id===p.segmentId);const el=sg?(S.vehicleType==="ticari"?sg.ticariLimit:sg.normalLimit):p.speedLimit;return`<div style="display:flex;align-items:center;justify-content:space-between;padding:10px 12px;border-radius:12px;border:1.5px solid ${iv?"#ef4444":ic?"#f59e0b":T.border};background:${iv?"rgba(239,68,68,.08)":ic?"rgba(245,158,11,.05)":T.card};${iv?"animation:flashBorder 1s infinite":""}"><div style="display:flex;align-items:center;gap:8px"><span style="font-size:18px">${p.type==="eds"?"📸":"📡"}</span><div><div style="font-size:11px;font-weight:600;color:${T.text}">${p.name}</div><div style="font-size:9px;color:${T.text3}">${p.city}</div></div></div><div style="display:flex;align-items:center;gap:10px"><span style="font-size:12px;font-weight:700;font-family:monospace;color:${iv?"#ef4444":ic?"#f59e0b":T.text2}">${p.distance<1?(p.distance*1000).toFixed(0)+"m":p.distance.toFixed(1)+"km"}</span><div style="display:flex;flex-direction:column;align-items:center;background:rgba(239,68,68,.1);border:1.5px solid #ef444433;border-radius:8px;padding:2px 8px;min-width:38px"><span style="font-size:8px;color:${T.text3}">Limit</span><span style="font-size:13px;font-weight:700;font-family:monospace;color:${T.text}">${el}</span></div></div></div>`}).join("")}
      </div>
    </div>`;
  }

  // Control
  html+=`<div style="padding:12px 20px 32px">
    ${!S.simulating?
      `<button class="btn" onclick="startDrive()" style="width:100%;padding:16px;border-radius:14px;border:none;background:linear-gradient(135deg,#2563eb,${T.acc});color:#fff;font-size:16px;font-weight:700;box-shadow:0 4px 24px ${T.acc}44">▶ Sürüşe Başla</button>`:
      `<button class="btn" onclick="stopDrive()" style="width:100%;padding:16px;border-radius:14px;border:2px solid #ef4444;background:rgba(239,68,68,.1);color:#ef4444;font-size:16px;font-weight:700">■ Sürüşü Durdur</button>`
    }
  </div>`;

  app.innerHTML=html;
  // Init minimap after DOM
  setTimeout(()=>{mapCanvas=document.getElementById("minimap");if(mapCanvas)mapCtx=mapCanvas.getContext("2d");startMapLoop();},50);
}

// ═══ MAP LOOP ═══
function startMapLoop(){
  if(mapFrame)cancelAnimationFrame(mapFrame);
  function drawMap(){
    if(!mapCtx||!mapCanvas){mapFrame=requestAnimationFrame(drawMap);return;}
    const T=getTheme();
    const W=mapCanvas.width,H=mapCanvas.height,cx=W/2;
    mapCtx.clearRect(0,0,W,H);
    mapCtx.fillStyle=T.mapBg;mapCtx.fillRect(0,0,W,H);
    if(S.simulating)mapOffset.y+=(S.speed/120)*1.5;
    const oy=mapOffset.y;
    const gs=S.roadType==="otoyol"?80:S.roadType==="sehirDisi"?60:40;
    mapCtx.strokeStyle=T.mapGr;mapCtx.lineWidth=.5;
    for(let x=-gs;x<W+gs;x+=gs){mapCtx.beginPath();mapCtx.moveTo(x-(mapOffset.x%gs),0);mapCtx.lineTo(x-(mapOffset.x%gs),H);mapCtx.stroke();}
    for(let y=-gs;y<H+gs;y+=gs){mapCtx.beginPath();mapCtx.moveTo(0,y+(oy%gs));mapCtx.lineTo(W,y+(oy%gs));mapCtx.stroke();}
    mapCtx.fillStyle=T.mapW;mapCtx.beginPath();mapCtx.ellipse(W*.85,((H*.3+oy*.15)%(H+100))-50,35,20,.3,0,Math.PI*2);mapCtx.fill();
    mapCtx.strokeStyle=T.mapRd;mapCtx.lineWidth=2;
    for(let i=0;i<4;i++){const rx=30+i*(W/3.5),by=((i*200+oy*.6)%(H+200))-100;mapCtx.beginPath();mapCtx.moveTo(rx-60,by);mapCtx.lineTo(rx+80,by);mapCtx.stroke();}
    for(let i=0;i<3;i++){const ry=((i*180+oy*.4)%(H+160))-80;mapCtx.beginPath();mapCtx.moveTo(0,ry);mapCtx.lineTo(W,ry);mapCtx.stroke();}
    const mw=S.roadType==="otoyol"?32:S.roadType==="sehirDisi"?24:18;
    mapCtx.fillStyle=T.mapRdS;mapCtx.fillRect(cx-mw/2-2,0,mw+4,H);
    mapCtx.fillStyle=T.rdSurf;mapCtx.fillRect(cx-mw/2,0,mw,H);
    mapCtx.strokeStyle=T.rdDash;mapCtx.lineWidth=1.5;mapCtx.setLineDash([14,10]);mapCtx.lineDashOffset=oy*2;
    mapCtx.beginPath();mapCtx.moveTo(cx,0);mapCtx.lineTo(cx,H);mapCtx.stroke();mapCtx.setLineDash([]);
    if(S.roadType==="sehirIci")for(let i=0;i<8;i++){const bx=(i%2===0)?cx-mw/2-18-Math.random()*30:cx+mw/2+8+Math.random()*30;const by=((i*75+oy*.5)%(H+60))-30;mapCtx.fillStyle=T.bldg;mapCtx.fillRect(bx,by,10+Math.random()*14,10+Math.random()*14);}
    const edsArr=EDS_DATABASE.map(p=>({...p,distance:S.edsDist[p.id]!==undefined?S.edsDist[p.id]:99})).sort((a,b)=>a.distance-b.distance).slice(0,5);
    edsArr.forEach((e,i)=>{if(e.distance>20)return;const my=H/2-(e.distance/8)*H;const mx=cx+(i%2===0?-1:1)*(mw/2+8+(i%3)*10);if(my>-20&&my<H+20){const g=mapCtx.createRadialGradient(mx,my,0,mx,my,12);g.addColorStop(0,e.distance<2?"rgba(239,68,68,.4)":"rgba(245,158,11,.3)");g.addColorStop(1,"rgba(0,0,0,0)");mapCtx.fillStyle=g;mapCtx.beginPath();mapCtx.arc(mx,my,12,0,Math.PI*2);mapCtx.fill();mapCtx.fillStyle=e.distance<2?"#ef4444":"#f59e0b";mapCtx.beginPath();mapCtx.arc(mx,my,4,0,Math.PI*2);mapCtx.fill();}});
    if(S.simulating){mapTrail.push({x:cx,y:H/2+30});if(mapTrail.length>15)mapTrail.shift();mapTrail.forEach((p,i)=>{mapCtx.fillStyle=`rgba(59,130,246,${(i/mapTrail.length)*.3})`;mapCtx.beginPath();mapCtx.arc(p.x,p.y+(mapTrail.length-i)*3,3,0,Math.PI*2);mapCtx.fill();});}
    const cg=mapCtx.createRadialGradient(cx,H/2+30,0,cx,H/2+30,20);cg.addColorStop(0,"rgba(59,130,246,.5)");cg.addColorStop(1,"rgba(59,130,246,0)");mapCtx.fillStyle=cg;mapCtx.beginPath();mapCtx.arc(cx,H/2+30,20,0,Math.PI*2);mapCtx.fill();
    mapCtx.fillStyle="#3b82f6";mapCtx.beginPath();mapCtx.moveTo(cx,H/2+20);mapCtx.lineTo(cx-6,H/2+34);mapCtx.lineTo(cx+6,H/2+34);mapCtx.closePath();mapCtx.fill();
    mapCtx.fillStyle="#fff";mapCtx.beginPath();mapCtx.arc(cx,H/2+28,2.5,0,Math.PI*2);mapCtx.fill();
    mapFrame=requestAnimationFrame(drawMap);
  }
  drawMap();
}

// ═══ ACTIONS ═══
function selectVehicle(type){S.vehicleType=type;S.screen="dashboard";render();}
function goBack(){stopDrive();S.screen="select";S.vehicleType=null;render();}
function toggleTheme(){S.theme=S.theme==="dark"?"light":"dark";document.querySelector('meta[name="theme-color"]').content=S.theme==="dark"?"#020617":"#f0f4f8";render();}
function toggleVoice(){S.voice=!S.voice;render();}
function toggleGPSPanel(){S.showGPS=!S.showGPS;render();}
function setRoad(id){S.roadType=id;S.segment=null;spokenSign=null;render();}
function removeAlert(id){S.alerts=S.alerts.filter(a=>a.id!==id);render();}

function addAlert(msg,type){
  S.alertId++;
  S.alerts=[{id:S.alertId,message:msg,type},...S.alerts].slice(0,5);
  render();
  setTimeout(()=>{S.alerts=S.alerts.filter(a=>a.id!==S.alertId);render();},6000);
}

function requestGPS(){
  S.gpsError=null;
  if(!navigator.geolocation){S.gpsError="Bu cihaz GPS desteklemiyor.";S.gpsStatus="error";render();return;}
  S.gpsStatus="searching";render();
  navigator.geolocation.getCurrentPosition(pos=>{
    S.gpsStatus="active";S.gpsError=null;
    S.userLoc={lat:pos.coords.latitude,lng:pos.coords.longitude};
    simLoc={...S.userLoc};
    let minD=Infinity,nc=null;CITIES.forEach(c=>{const d=haversine(S.userLoc.lat,S.userLoc.lng,c.lat,c.lng);if(d<minD){minD=d;nc=c.name;}});S.city=nc;
    let ms=Infinity,ns=null;ROAD_SEGMENTS.forEach(s=>{const d=haversine(S.userLoc.lat,S.userLoc.lng,s.lat,s.lng);if(d<ms){ms=d;ns=s;}});if(ns)S.segment=ns;
    addAlert(`✅ GPS bağlandı! ${nc||""}`,`success`);
    speak(`GPS bağlandı. Konumunuz ${nc||"tespit edildi"}.`,S.voice);
    gpsWatch=navigator.geolocation.watchPosition(p=>{S.userLoc={lat:p.coords.latitude,lng:p.coords.longitude};simLoc={...S.userLoc};if(p.coords.speed!=null&&p.coords.speed>=0)S.speed=Math.round(p.coords.speed*3.6);},()=>{},{enableHighAccuracy:true,maximumAge:2000,timeout:15000});
    render();
  },err=>{
    const msgs={1:"GPS izni reddedildi. Tarayıcı ayarlarından konum iznini açın.",2:"Konum bilgisi alınamadı.",3:"GPS zaman aşımı. Tekrar deneyin."};
    S.gpsError=msgs[err.code]||err.message;S.gpsStatus="error";render();
  },{enableHighAccuracy:true,maximumAge:0,timeout:15000});
}

function selectCity(name,lat,lng){
  S.userLoc={lat,lng};simLoc={lat,lng};S.city=name;S.gpsStatus="manual";S.gpsError=null;
  let ms=Infinity,ns=null;ROAD_SEGMENTS.forEach(s=>{const d=haversine(lat,lng,s.lat,s.lng);if(d<ms){ms=d;ns=s;}});if(ns)S.segment=ns;
  addAlert(`📍 ${name} seçildi.`,"success");speak(`${name} konumu seçildi.`,S.voice);render();
}

function startDrive(){
  S.simulating=true;spokenEds.clear();spokenOver=false;spokenSign=null;
  if(S.gpsStatus==="inactive")S.gpsStatus="simulated";
  speak("Sürüş başlatıldı. İyi yolculuklar!",S.voice);
  
  // WAKE LOCK AKTİF ET
  requestWakeLock();

  const segs=ROAD_SEGMENTS.filter(s=>s.roadType===S.roadType);
  let segIdx=0;if(segs.length)S.segment=segs[0];

  // Segment cycling
  segInterval=setInterval(()=>{
    const sg=ROAD_SEGMENTS.filter(s=>s.roadType===S.roadType);
    if(!sg.length)return;
    segIdx=(segIdx+1)%sg.length;
    S.segment=sg[segIdx];
    const lim=S.vehicleType==="ticari"?sg[segIdx].ticariLimit:sg[segIdx].normalLimit;
    if(spokenSign!==sg[segIdx].id){
      spokenSign=sg[segIdx].id;
      addAlert(`🚧 ${sg[segIdx].name} — Tabela: ${sg[segIdx].signLimit} | Siz: ${lim}`,"warning");
      speak(sg[segIdx].signLimit!==lim?`Yeni bölge: ${sg[segIdx].name}. Tabela ${sg[segIdx].signLimit}, limitiniz ${lim}.`:`Yeni bölge: ${sg[segIdx].name}. Limit ${lim}.`,S.voice);
    }
    render();
  },8000);

  // Speed simulation
  simInterval=setInterval(()=>{
    if(S.gpsStatus!=="active"){
      const seg=S.segment||ROAD_SEGMENTS[0];
      const lim=S.vehicleType==="ticari"?seg.ticariLimit:seg.normalLimit;
      const tgt=Math.round(lim*(.7+Math.random()*.45));
      const d=tgt-S.speed;
      S.speed=Math.max(0,Math.round(S.speed+Math.sign(d)*Math.min(Math.abs(d),2+Math.random()*2)));
    }
    S.traveledKm+=.02;
    if(S.speed>0){S.maxSpeed=Math.max(S.maxSpeed,S.speed);S.avgSum+=S.speed;S.avgCount++;}

    // Check overspeed
    const seg=S.segment||ROAD_SEGMENTS[0];
    const yl=S.vehicleType==="ticari"?seg.ticariLimit:seg.normalLimit;
    if(S.speed>yl&&!spokenOver){spokenOver=true;addAlert(`⚡ Hız aşımı! Limit: ${yl} — Hız: ${S.speed}`,"danger");speak(`Hız limitini aşıyorsunuz! Limit ${yl}. Yavaşlayın!`,S.voice);}
    if(S.speed<=yl)spokenOver=false;
    render();
  },300);

  // EDS simulation
  edsInterval=setInterval(()=>{
    EDS_DATABASE.forEach(p=>{
      const c=S.edsDist[p.id]!==undefined?S.edsDist[p.id]:5+Math.random()*20;
      S.edsDist[p.id]=Math.max(0,+(c-.03-Math.random()*.04).toFixed(2));
      if(S.edsDist[p.id]<=0){S.edsDist[p.id]=+(10+Math.random()*20).toFixed(1);spokenEds.delete(p.id+"_2km");spokenEds.delete(p.id+"_500m");}
      const d=S.edsDist[p.id];
      const sg=ROAD_SEGMENTS.find(s=>s.id===p.segmentId);
      const lim=sg?(S.vehicleType==="ticari"?sg.ticariLimit:sg.normalLimit):p.speedLimit;
      if(d<2&&d>1.5&&!spokenEds.has(p.id+"_2km")){spokenEds.add(p.id+"_2km");addAlert(`📍 ${p.name} — ${d.toFixed(1)} km | Limit: ${lim}`,"warning");speak(`Dikkat! ${p.name}, ${d.toFixed(1)} kilometre. Limitiniz ${lim}.`,S.voice);}
      if(d<.5&&d>.2&&!spokenEds.has(p.id+"_500m")){spokenEds.add(p.id+"_500m");addAlert(`🚨 ${p.name} — ÇOK YAKIN!`,"danger");speak(`Uyarı! ${p.name} çok yakın!`,S.voice);}
    });
  },600);

  render();
}

function stopDrive(){
  S.simulating=false;S.speed=0;S.traveledKm=0;S.maxSpeed=0;S.avgSum=0;S.avgCount=0;S.edsDist={};S.alerts=[];S.segment=null;
  spokenEds.clear();spokenSign=null;mapTrail=[];mapOffset={x:0,y:0};
  clearInterval(simInterval);clearInterval(edsInterval);clearInterval(segInterval);
  if(gpsWatch!==null){navigator.geolocation.clearWatch(gpsWatch);gpsWatch=null;}
  
  // WAKE LOCK KAPAT
  releaseWakeLock();
  
  render();
}

// ═══ INIT ═══
render();

// Prevent zoom on double tap
document.addEventListener('touchstart',function(e){if(e.touches.length>1)e.preventDefault();},{passive:false});
let lastTouch=0;
document.addEventListener('touchend',function(e){const now=Date.now();if(now-lastTouch<=300)e.preventDefault();lastTouch=now;},false);
</script>
</body>
</html>
