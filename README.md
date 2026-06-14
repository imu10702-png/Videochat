# Videochat
Random video chat
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>RandomLink — Global Video Chat</title>
<script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box}
:root{
  --bg:#08080f;--surface:#111118;--surface2:#1a1a26;
  --bdr:rgba(255,255,255,0.09);
  --acc:#7c6fff;--acc2:#00e5c3;
  --tx:#eeeeff;--mt:rgba(238,238,255,0.45);
  --red:#ff4d6d;--gr:#00e5c3;
}
body{background:var(--bg);color:var(--tx);font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;height:100dvh;overflow:hidden;display:flex;flex-direction:column}
#stage{flex:1;position:relative;overflow:hidden;background:#000}
#remoteVideo{width:100%;height:100%;object-fit:cover;display:block}
#localVideo{position:absolute;bottom:16px;right:16px;width:clamp(88px,20vw,155px);aspect-ratio:3/4;border-radius:12px;object-fit:cover;border:2px solid rgba(255,255,255,.18);box-shadow:0 6px 28px rgba(0,0,0,.7);z-index:10;cursor:pointer;display:none}
.screen{position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:18px;background:rgba(8,8,15,.94);backdrop-filter:blur(10px);z-index:20;transition:opacity .25s}
.screen.hidden{opacity:0;pointer-events:none}
.logo{font-size:clamp(28px,5vw,40px);font-weight:800;letter-spacing:-1.5px;background:linear-gradient(130deg,var(--acc),var(--acc2));-webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text}
.tag{font-size:14px;color:var(--mt);text-align:center;max-width:260px;line-height:1.65}
.globe-wrap{width:76px;height:76px;position:relative;margin-bottom:4px}
.globe{width:76px;height:76px;border-radius:50%;border:2px solid var(--acc);animation:sy 9s linear infinite;box-shadow:0 0 28px rgba(124,111,255,.3),inset 0 0 24px rgba(0,229,195,.08)}
.globe::after{content:'';position:absolute;top:50%;left:-2px;right:-2px;height:1px;background:var(--acc2);opacity:.4}
@keyframes sy{from{transform:rotateY(0)}to{transform:rotateY(360deg)}}
.pr{position:absolute;inset:-10px;border-radius:50%;border:1.5px solid var(--acc2);animation:pp 2s ease-out infinite;opacity:0}
.pr:nth-child(2){animation-delay:.7s}.pr:nth-child(3){animation-delay:1.4s}
@keyframes pp{0%{transform:scale(.85);opacity:.55}100%{transform:scale(1.55);opacity:0}}
.dots{display:flex;gap:7px}
.dot{width:9px;height:9px;border-radius:50%;background:var(--acc);animation:bb 1.2s ease-in-out infinite}
.dot:nth-child(2){animation-delay:.2s;background:var(--acc2)}.dot:nth-child(3){animation-delay:.4s}
@keyframes bb{0%,80%,100%{transform:translateY(0);opacity:.4}40%{transform:translateY(-11px);opacity:1}}
.btn{display:inline-flex;align-items:center;justify-content:center;gap:8px;padding:13px 26px;border-radius:50px;border:none;font-size:15px;font-weight:600;cursor:pointer;transition:transform .15s,opacity .15s;min-width:150px}
.btn:active{transform:scale(.96)}
.btn-p{background:var(--acc);color:#fff;box-shadow:0 4px 22px rgba(124,111,255,.4)}
.btn-p:hover{opacity:.88}
.btn-g{background:rgba(255,255,255,.07);color:var(--tx);border:1px solid var(--bdr)}
.btn-g:hover{background:rgba(255,255,255,.13)}
#permBox{background:rgba(255,77,109,.1);border:1px solid rgba(255,77,109,.3);border-radius:12px;padding:14px 18px;max-width:290px;font-size:13px;color:#ffb3c1;line-height:1.6;text-align:center;display:none}
#sb{position:absolute;top:14px;left:50%;transform:translateX(-50%);background:rgba(0,0,0,.62);border:1px solid var(--bdr);border-radius:20px;padding:5px 14px;font-size:12px;color:var(--mt);backdrop-filter:blur(6px);z-index:15;white-space:nowrap;transition:opacity .3s}
#sb.ok{color:var(--gr);border-color:rgba(0,229,195,.3)}
#rph{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;flex-direction:column;gap:14px;background:var(--surface);z-index:5;color:var(--mt);font-size:14px}
.avph{width:68px;height:68px;border-radius:50%;background:var(--surface2);border:2px solid var(--bdr);display:flex;align-items:center;justify-content:center;font-size:26px}
#controls{display:flex;align-items:center;justify-content:center;gap:10px;padding:14px 16px;background:var(--surface);border-top:1px solid var(--bdr);flex-shrink:0}
.ctrl{width:50px;height:50px;border-radius:50%;border:1px solid var(--bdr);background:var(--surface2);color:var(--tx);font-size:20px;cursor:pointer;display:flex;align-items:center;justify-content:center;transition:all .15s}
.ctrl:hover{background:rgba(255,255,255,.1)}.ctrl:active{transform:scale(.9)}
.ctrl.off{background:rgba(255,77,109,.18);border-color:var(--red);color:var(--red)}
.ctrl.fb{background:rgba(0,229,195,.12);border-color:var(--gr)}
#nextBtn{padding:0 22px;height:50px;border-radius:25px;background:var(--acc);color:#fff;border:none;font-size:14px;font-weight:700;cursor:pointer;letter-spacing:.5px;transition:all .15s;box-shadow:0 4px 18px rgba(124,111,255,.35)}
#nextBtn:hover{opacity:.88}#nextBtn:active{transform:scale(.95)}
#sTimer{font-size:13px;color:var(--mt);font-variant-numeric:tabular-nums}
#fbNote{font-size:11px;color:var(--mt);text-align:center;max-width:280px;line-height:1.6;padding:8px 16px;background:rgba(124,111,255,.08);border:1px solid rgba(124,111,255,.2);border-radius:10px;display:none}
@media(max-width:460px){#localVideo{width:84px}.ctrl{width:44px;height:44px;font-size:18px}#nextBtn{padding:0 14px;font-size:13px}#controls{gap:7px;padding:12px 10px}}
</style>
</head>
<body>

<div id="stage">
  <video id="remoteVideo" autoplay playsinline></video>
  <div id="rph"><div class="avph">👤</div><span>Waiting for a stranger…</span></div>
  <video id="localVideo" autoplay playsinline muted></video>
  <div id="sb">–</div>

  <!-- WELCOME -->
  <div class="screen" id="scW">
    <div class="globe-wrap">
      <div class="globe"></div>
      <div class="pr"></div><div class="pr"></div><div class="pr"></div>
    </div>
    <div class="logo">RandomLink</div>
    <div class="tag">Instant face-to-face with strangers worldwide.<br>No signup · No text · Just video.</div>
    <div id="permBox"></div>
    <div id="fbNote"></div>
    <button class="btn btn-p" id="startBtn" onclick="startApp()">▶ &nbsp;Allow Camera &amp; Start</button>
  </div>

  <!-- SEARCHING -->
  <div class="screen hidden" id="scS">
    <div class="dots"><div class="dot"></div><div class="dot"></div><div class="dot"></div></div>
    <div style="font-size:19px;font-weight:700">Finding someone…</div>
    <div id="sTimer">Searching 0s</div>
    <div style="font-size:12px;color:var(--mt);max-width:240px;text-align:center;line-height:1.5">Share this page link with a friend to connect instantly, or wait for a random match.</div>
    <button class="btn btn-g" onclick="cancelSearch()" style="margin-top:4px">Cancel</button>
  </div>

  <!-- DISCONNECTED -->
  <div class="screen hidden" id="scD">
    <div style="font-size:44px">👋</div>
    <div style="font-size:20px;font-weight:700">Stranger left</div>
    <div class="tag">They disconnected. Find someone new?</div>
    <div style="display:flex;gap:12px;flex-wrap:wrap;justify-content:center;margin-top:4px">
      <button class="btn btn-p" onclick="findNext()">Next →</button>
      <button class="btn btn-g" onclick="goHome()">Home</button>
    </div>
  </div>
</div>

<div id="controls">
  <button class="ctrl" id="muteBtn"  onclick="toggleMute()" title="Mute">🎙️</button>
  <button class="ctrl" id="flipBtn"  onclick="flipCamera()" title="Flip camera">🔄</button>
  <button id="nextBtn" onclick="findNext()">NEXT →</button>
  <button class="ctrl" id="vidBtn"   onclick="toggleVideo()" title="Video off">📹</button>
  <button class="ctrl" onclick="goHome()" style="background:rgba(255,77,109,.15);border-color:var(--red);color:var(--red)" title="End">✕</button>
</div>

<script>
// ══════════════════════════════════════════════════════
//  Firebase Realtime Database — FREE public lobby
//  Uses Firebase's free Spark plan. Replace databaseURL
//  with your own project for production use.
//  HOW TO GET YOUR OWN (free, 2 min):
//  1. Go to console.firebase.google.com
//  2. Create project → Build → Realtime Database
//  3. Start in test mode (rules allow read/write)
//  4. Copy your databaseURL here
// ══════════════════════════════════════════════════════
const FB_CFG = {
  apiKey: "AIzaSyDemo00000000000000000000000000000",
  databaseURL: "https://randomlink-lobby-default-rtdb.firebaseio.com"
};
// NOTE: The above is a placeholder. Without a real Firebase project,
// only same-browser-tab matchmaking works (BroadcastChannel).
// See setup instructions in the NEXT →  button area below.

let db = null;
let myLobbyRef = null;
let lobbyPollTimer = null;
let bc = null;

let localStream = null;
let peer = null;
let myPeerId = null;
let currentCall = null;
let isSearching = false;
let searchSec = 0, searchTimer = null;
let callSec = 0, callTimer = null;
let isMuted = false, isVidOff = false, usingFront = true;

const $ = id => document.getElementById(id);
const showScreen = id => ['scW','scS','scD'].forEach(s => $(s).classList.toggle('hidden', s!==id));
const setStatus = (t,c='') => { const b=$('sb'); b.textContent=t; b.className=c; };

// ── Firebase init ─────────────────────────────────────
function initFirebase() {
  try {
    if (!firebase.apps.length) firebase.initializeApp(FB_CFG);
    db = firebase.database();
    // Quick connectivity test
    db.ref('.info/connected').once('value', snap => {
      if (!snap.val()) db = null;
    });
  } catch(e) { db = null; }
}

// ── START (must be user click for camera permission) ──
async function startApp() {
  const btn = $('startBtn');
  btn.textContent = '⏳ Requesting camera…';
  btn.disabled = true;
  $('permBox').style.display = 'none';

  // Camera permission — triggered by click ✓
  try {
    localStream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode:'user', width:{ideal:1280}, height:{ideal:720} },
      audio: { echoCancellation:true, noiseSuppression:true, sampleRate:48000 }
    });
  } catch(e) {
    btn.disabled = false;
    btn.textContent = '▶ Try Again';
    const pb = $('permBox');
    pb.style.display = 'block';
    if (e.name==='NotAllowedError'||e.name==='PermissionDeniedError') {
      pb.innerHTML = '🔒 <strong>Camera blocked.</strong><br>Click the 🔒 icon in your browser address bar → allow Camera &amp; Microphone → refresh.';
    } else if (e.name==='NotFoundError') {
      pb.textContent = '📷 No camera detected. Please connect a camera and try again.';
    } else {
      pb.textContent = '⚠️ Camera error: ' + (e.message||e.name);
    }
    return;
  }

  // Show local preview
  const lv = $('localVideo');
  lv.srcObject = localStream;
  lv.style.display = 'block';

  // Init PeerJS (WebRTC broker)
  setStatus('⏳ Connecting…');
  await initPeer();

  // Init Firebase matchmaking
  initFirebase();

  showScreen(null);
  $('rph').style.display = 'flex';
  setStatus('✅ Ready');

  // Auto-search
  setTimeout(findNext, 400);
}

// ── PeerJS ────────────────────────────────────────────
function initPeer() {
  return new Promise(res => {
    if (peer) try { peer.destroy(); } catch(e){}
    peer = new Peer({ debug:0 });
    peer.on('open', id => { myPeerId=id; res(); });
    peer.on('call', inc => { if (!currentCall) answerCall(inc); });
    peer.on('error', err => {
      if (err.type !== 'peer-unavailable') setStatus('⚠ Network issue');
    });
    setTimeout(res, 5000); // don't block forever
  });
}

// ── Matchmaking ───────────────────────────────────────
function findNext() {
  endCurrentCall();
  if (!myPeerId) { setTimeout(findNext, 600); return; }
  startSearch();
}

function startSearch() {
  isSearching = true; searchSec = 0;
  showScreen('scS');
  $('rph').style.display = 'flex';
  $('remoteVideo').srcObject = null;
  setStatus('🔍 Searching…');
  $('sTimer').textContent = 'Searching 0s';
  searchTimer = setInterval(()=>{ searchSec++; $('sTimer').textContent=`Searching ${searchSec}s`; }, 1000);

  if (db) {
    matchFirebase();
  } else {
    matchBroadcast();  // Same-browser tab fallback
  }
}

function cancelSearch() {
  isSearching = false;
  clearInterval(searchTimer);
  cleanupLobby();
  showScreen(null);
  $('rph').style.display = 'flex';
  setStatus('Ready — tap NEXT');
}

// Firebase lobby: write our ID → read for partner → call them
function matchFirebase() {
  const lobby = db.ref('lobby');

  // First: look for someone already waiting
  lobby.orderByChild('ts').limitToFirst(10).once('value', snap => {
    if (!isSearching) return;
    let partner = null;
    snap.forEach(c => {
      const d = c.val();
      if (d.peerId && d.peerId !== myPeerId && !partner) {
        partner = { key: c.key, peerId: d.peerId };
      }
    });

    if (partner) {
      // Claim them (remove from lobby) and call
      lobby.child(partner.key).remove();
      stopSearch();
      callPeer(partner.peerId);
    } else {
      // Add ourselves and wait
      myLobbyRef = lobby.push({ peerId: myPeerId, ts: firebase.database.ServerValue.TIMESTAMP });
      myLobbyRef.onDisconnect().remove();

      // Poll every 2.5s for new arrivals
      lobbyPollTimer = setInterval(() => {
        if (!isSearching) { clearInterval(lobbyPollTimer); return; }
        lobby.orderByChild('ts').limitToFirst(10).once('value', snap2 => {
          if (!isSearching) return;
          snap2.forEach(c => {
            const d = c.val();
            if (d.peerId && d.peerId !== myPeerId) {
              lobby.child(c.key).remove();
              stopSearch();
              callPeer(d.peerId);
            }
          });
        });
      }, 2500);
    }
  });
}

// BroadcastChannel: works across tabs in same browser
function matchBroadcast() {
  try {
    bc = new BroadcastChannel('rl_v2');
    bc.onmessage = e => {
      const m = e.data;
      if (!isSearching) return;
      if (m.type==='hello' && m.from && m.from!==myPeerId) {
        bc.postMessage({ type:'match', from:myPeerId, to:m.from });
        stopSearch();
        callPeer(m.from);
      }
      if (m.type==='match' && m.to===myPeerId) {
        stopSearch();
      }
    };
    bc.postMessage({ type:'hello', from:myPeerId });
  } catch(e) {
    setStatus('⚠ Open 2 tabs to test locally');
  }
}

function stopSearch() {
  isSearching = false;
  clearInterval(searchTimer);
  clearInterval(lobbyPollTimer);
  cleanupLobby();
}

function cleanupLobby() {
  if (myLobbyRef) { myLobbyRef.remove(); myLobbyRef=null; }
  clearInterval(lobbyPollTimer);
  if (bc) { try{bc.close();}catch(e){} bc=null; }
}

// ── Calling ───────────────────────────────────────────
function callPeer(peerId) {
  if (!peer || !localStream || !peerId) return;
  showScreen(null);
  setStatus('📡 Ringing…');
  const call = peer.call(peerId, localStream);
  setupCall(call);
}

function answerCall(call) {
  stopSearch();
  cleanupLobby();
  showScreen(null);
  setStatus('📡 Connecting…');
  call.answer(localStream);
  setupCall(call);
}

function setupCall(call) {
  currentCall = call;
  let gotStream = false;

  call.on('stream', remote => {
    gotStream = true;
    $('remoteVideo').srcObject = remote;
    $('rph').style.display = 'none';
    showScreen(null);
    setStatus('🟢 Connected','ok');
    startCallTimer();
  });

  call.on('close', onDisconnect);
  call.on('error', onDisconnect);

  // Timeout if peer never sends stream
  setTimeout(() => {
    if (!gotStream && currentCall===call) onDisconnect();
  }, 12000);
}

function onDisconnect() {
  stopCallTimer();
  currentCall = null;
  $('remoteVideo').srcObject = null;
  $('rph').style.display = 'flex';
  showScreen('scD');
  setStatus('');
}

function endCurrentCall() {
  stopSearch();
  stopCallTimer();
  cleanupLobby();
  if (currentCall) { try{currentCall.close();}catch(e){} currentCall=null; }
  $('remoteVideo').srcObject = null;
}

function goHome() {
  endCurrentCall();
  showScreen(null);
  $('rph').style.display = 'flex';
  setStatus('Ready — tap NEXT to find someone');
}

// ── Media controls ────────────────────────────────────
function toggleMute() {
  isMuted = !isMuted;
  localStream?.getAudioTracks().forEach(t => t.enabled=!isMuted);
  const b=$('muteBtn'); b.textContent=isMuted?'🔇':'🎙️'; b.classList.toggle('off',isMuted);
}

function toggleVideo() {
  isVidOff = !isVidOff;
  localStream?.getVideoTracks().forEach(t => t.enabled=!isVidOff);
  const b=$('vidBtn'); b.textContent=isVidOff?'🚫':'📹'; b.classList.toggle('off',isVidOff);
}

async function flipCamera() {
  usingFront = !usingFront;
  $('flipBtn').style.opacity='.35';
  try {
    const ns = await navigator.mediaDevices.getUserMedia({
      video:{ facingMode: usingFront?'user':'environment' }, audio:false
    });
    const nv = ns.getVideoTracks()[0];
    const ov = localStream.getVideoTracks()[0];
    localStream.removeTrack(ov); ov.stop();
    localStream.addTrack(nv);
    $('localVideo').srcObject = null;
    $('localVideo').srcObject = localStream;
    if (currentCall) {
      const s = currentCall.peerConnection?.getSenders?.()?.find(s=>s.track?.kind==='video');
      if (s) await s.replaceTrack(nv);
    }
    $('flipBtn').classList.toggle('fb', !usingFront);
  } catch(e) { usingFront=!usingFront; }
  $('flipBtn').style.opacity='1';
}

// Tap PiP to bring it front
$('localVideo').addEventListener('click', () => {
  const lv=$('localVideo'), rv=$('remoteVideo');
  [lv.style.zIndex, rv.style.zIndex] = lv.style.zIndex==='20' ? ['10',''] : ['20',''];
  lv.style.inset = lv.style.zIndex==='20' ? '0' : '';
  lv.style.width  = lv.style.zIndex==='20' ? '100%' : '';
  lv.style.height = lv.style.zIndex==='20' ? '100%' : '';
  lv.style.borderRadius = lv.style.zIndex==='20' ? '0' : '';
  lv.style.bottom = lv.style.zIndex==='20' ? '0' : '16px';
  lv.style.right  = lv.style.zIndex==='20' ? '0' : '16px';
});

// ── Timers ────────────────────────────────────────────
function startCallTimer() {
  callSec=0; stopCallTimer();
  callTimer = setInterval(()=>{
    callSec++;
    const m=String(Math.floor(callSec/60)).padStart(2,'0');
    const s=String(callSec%60).padStart(2,'0');
    setStatus(`🟢 ${m}:${s}`,'ok');
  },1000);
}
function stopCallTimer(){ clearInterval(callTimer); callTimer=null; }

window.addEventListener('beforeunload', ()=>{
  endCurrentCall();
  if(peer) peer.destroy();
  localStream?.getTracks().forEach(t=>t.stop());
});
</script>
</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>RandomLink — Global Video Chat</title>
<script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box}
:root{
  --bg:#08080f;--surface:#111118;--surface2:#1a1a26;
  --bdr:rgba(255,255,255,0.09);
  --acc:#7c6fff;--acc2:#00e5c3;
  --tx:#eeeeff;--mt:rgba(238,238,255,0.45);
  --red:#ff4d6d;--gr:#00e5c3;
}
body{background:var(--bg);color:var(--tx);font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;height:100dvh;overflow:hidden;display:flex;flex-direction:column}
#stage{flex:1;position:relative;overflow:hidden;background:#000}
#remoteVideo{width:100%;height:100%;object-fit:cover;display:block}
#localVideo{position:absolute;bottom:16px;right:16px;width:clamp(88px,20vw,155px);aspect-ratio:3/4;border-radius:12px;object-fit:cover;border:2px solid rgba(255,255,255,.18);box-shadow:0 6px 28px rgba(0,0,0,.7);z-index:10;cursor:pointer;display:none}
.screen{position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:18px;background:rgba(8,8,15,.94);backdrop-filter:blur(10px);z-index:20;transition:opacity .25s}
.screen.hidden{opacity:0;pointer-events:none}
.logo{font-size:clamp(28px,5vw,40px);font-weight:800;letter-spacing:-1.5px;background:linear-gradient(130deg,var(--acc),var(--acc2));-webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text}
.tag{font-size:14px;color:var(--mt);text-align:center;max-width:260px;line-height:1.65}
.globe-wrap{width:76px;height:76px;position:relative;margin-bottom:4px}
.globe{width:76px;height:76px;border-radius:50%;border:2px solid var(--acc);animation:sy 9s linear infinite;box-shadow:0 0 28px rgba(124,111,255,.3),inset 0 0 24px rgba(0,229,195,.08)}
.globe::after{content:'';position:absolute;top:50%;left:-2px;right:-2px;height:1px;background:var(--acc2);opacity:.4}
@keyframes sy{from{transform:rotateY(0)}to{transform:rotateY(360deg)}}
.pr{position:absolute;inset:-10px;border-radius:50%;border:1.5px solid var(--acc2);animation:pp 2s ease-out infinite;opacity:0}
.pr:nth-child(2){animation-delay:.7s}.pr:nth-child(3){animation-delay:1.4s}
@keyframes pp{0%{transform:scale(.85);opacity:.55}100%{transform:scale(1.55);opacity:0}}
.dots{display:flex;gap:7px}
.dot{width:9px;height:9px;border-radius:50%;background:var(--acc);animation:bb 1.2s ease-in-out infinite}
.dot:nth-child(2){animation-delay:.2s;background:var(--acc2)}.dot:nth-child(3){animation-delay:.4s}
@keyframes bb{0%,80%,100%{transform:translateY(0);opacity:.4}40%{transform:translateY(-11px);opacity:1}}
.btn{display:inline-flex;align-items:center;justify-content:center;gap:8px;padding:13px 26px;border-radius:50px;border:none;font-size:15px;font-weight:600;cursor:pointer;transition:transform .15s,opacity .15s;min-width:150px}
.btn:active{transform:scale(.96)}
.btn-p{background:var(--acc);color:#fff;box-shadow:0 4px 22px rgba(124,111,255,.4)}
.btn-p:hover{opacity:.88}
.btn-g{background:rgba(255,255,255,.07);color:var(--tx);border:1px solid var(--bdr)}
.btn-g:hover{background:rgba(255,255,255,.13)}
#permBox{background:rgba(255,77,109,.1);border:1px solid rgba(255,77,109,.3);border-radius:12px;padding:14px 18px;max-width:290px;font-size:13px;color:#ffb3c1;line-height:1.6;text-align:center;display:none}
#sb{position:absolute;top:14px;left:50%;transform:translateX(-50%);background:rgba(0,0,0,.62);border:1px solid var(--bdr);border-radius:20px;padding:5px 14px;font-size:12px;color:var(--mt);backdrop-filter:blur(6px);z-index:15;white-space:nowrap;transition:opacity .3s}
#sb.ok{color:var(--gr);border-color:rgba(0,229,195,.3)}
#rph{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;flex-direction:column;gap:14px;background:var(--surface);z-index:5;color:var(--mt);font-size:14px}
.avph{width:68px;height:68px;border-radius:50%;background:var(--surface2);border:2px solid var(--bdr);display:flex;align-items:center;justify-content:center;font-size:26px}
#controls{display:flex;align-items:center;justify-content:center;gap:10px;padding:14px 16px;background:var(--surface);border-top:1px solid var(--bdr);flex-shrink:0}
.ctrl{width:50px;height:50px;border-radius:50%;border:1px solid var(--bdr);background:var(--surface2);color:var(--tx);font-size:20px;cursor:pointer;display:flex;align-items:center;justify-content:center;transition:all .15s}
.ctrl:hover{background:rgba(255,255,255,.1)}.ctrl:active{transform:scale(.9)}
.ctrl.off{background:rgba(255,77,109,.18);border-color:var(--red);color:var(--red)}
.ctrl.fb{background:rgba(0,229,195,.12);border-color:var(--gr)}
#nextBtn{padding:0 22px;height:50px;border-radius:25px;background:var(--acc);color:#fff;border:none;font-size:14px;font-weight:700;cursor:pointer;letter-spacing:.5px;transition:all .15s;box-shadow:0 4px 18px rgba(124,111,255,.35)}
#nextBtn:hover{opacity:.88}#nextBtn:active{transform:scale(.95)}
#sTimer{font-size:13px;color:var(--mt);font-variant-numeric:tabular-nums}
#fbNote{font-size:11px;color:var(--mt);text-align:center;max-width:280px;line-height:1.6;padding:8px 16px;background:rgba(124,111,255,.08);border:1px solid rgba(124,111,255,.2);border-radius:10px;display:none}
@media(max-width:460px){#localVideo{width:84px}.ctrl{width:44px;height:44px;font-size:18px}#nextBtn{padding:0 14px;font-size:13px}#controls{gap:7px;padding:12px 10px}}
</style>
</head>
<body>

<div id="stage">
  <video id="remoteVideo" autoplay playsinline></video>
  <div id="rph"><div class="avph">👤</div><span>Waiting for a stranger…</span></div>
  <video id="localVideo" autoplay playsinline muted></video>
  <div id="sb">–</div>

  <!-- WELCOME -->
  <div class="screen" id="scW">
    <div class="globe-wrap">
      <div class="globe"></div>
      <div class="pr"></div><div class="pr"></div><div class="pr"></div>
    </div>
    <div class="logo">RandomLink</div>
    <div class="tag">Instant face-to-face with strangers worldwide.<br>No signup · No text · Just video.</div>
    <div id="permBox"></div>
    <div id="fbNote"></div>
    <button class="btn btn-p" id="startBtn" onclick="startApp()">▶ &nbsp;Allow Camera &amp; Start</button>
  </div>

  <!-- SEARCHING -->
  <div class="screen hidden" id="scS">
    <div class="dots"><div class="dot"></div><div class="dot"></div><div class="dot"></div></div>
    <div style="font-size:19px;font-weight:700">Finding someone…</div>
    <div id="sTimer">Searching 0s</div>
    <div style="font-size:12px;color:var(--mt);max-width:240px;text-align:center;line-height:1.5">Share this page link with a friend to connect instantly, or wait for a random match.</div>
    <button class="btn btn-g" onclick="cancelSearch()" style="margin-top:4px">Cancel</button>
  </div>

  <!-- DISCONNECTED -->
  <div class="screen hidden" id="scD">
    <div style="font-size:44px">👋</div>
    <div style="font-size:20px;font-weight:700">Stranger left</div>
    <div class="tag">They disconnected. Find someone new?</div>
    <div style="display:flex;gap:12px;flex-wrap:wrap;justify-content:center;margin-top:4px">
      <button class="btn btn-p" onclick="findNext()">Next →</button>
      <button class="btn btn-g" onclick="goHome()">Home</button>
    </div>
  </div>
</div>

<div id="controls">
  <button class="ctrl" id="muteBtn"  onclick="toggleMute()" title="Mute">🎙️</button>
  <button class="ctrl" id="flipBtn"  onclick="flipCamera()" title="Flip camera">🔄</button>
  <button id="nextBtn" onclick="findNext()">NEXT →</button>
  <button class="ctrl" id="vidBtn"   onclick="toggleVideo()" title="Video off">📹</button>
  <button class="ctrl" onclick="goHome()" style="background:rgba(255,77,109,.15);border-color:var(--red);color:var(--red)" title="End">✕</button>
</div>

<script>
// ══════════════════════════════════════════════════════
//  Firebase Realtime Database — FREE public lobby
//  Uses Firebase's free Spark plan. Replace databaseURL
//  with your own project for production use.
//  HOW TO GET YOUR OWN (free, 2 min):
//  1. Go to console.firebase.google.com
//  2. Create project → Build → Realtime Database
//  3. Start in test mode (rules allow read/write)
//  4. Copy your databaseURL here
// ══════════════════════════════════════════════════════
const FB_CFG = {
  apiKey: "AIzaSyDemo00000000000000000000000000000",
  databaseURL: "https://randomlink-lobby-default-rtdb.firebaseio.com"
};
// NOTE: The above is a placeholder. Without a real Firebase project,
// only same-browser-tab matchmaking works (BroadcastChannel).
// See setup instructions in the NEXT →  button area below.

let db = null;
let myLobbyRef = null;
let lobbyPollTimer = null;
let bc = null;

let localStream = null;
let peer = null;
let myPeerId = null;
let currentCall = null;
let isSearching = false;
let searchSec = 0, searchTimer = null;
let callSec = 0, callTimer = null;
let isMuted = false, isVidOff = false, usingFront = true;

const $ = id => document.getElementById(id);
const showScreen = id => ['scW','scS','scD'].forEach(s => $(s).classList.toggle('hidden', s!==id));
const setStatus = (t,c='') => { const b=$('sb'); b.textContent=t; b.className=c; };

// ── Firebase init ─────────────────────────────────────
function initFirebase() {
  try {
    if (!firebase.apps.length) firebase.initializeApp(FB_CFG);
    db = firebase.database();
    // Quick connectivity test
    db.ref('.info/connected').once('value', snap => {
      if (!snap.val()) db = null;
    });
  } catch(e) { db = null; }
}

// ── START (must be user click for camera permission) ──
async function startApp() {
  const btn = $('startBtn');
  btn.textContent = '⏳ Requesting camera…';
  btn.disabled = true;
  $('permBox').style.display = 'none';

  // Camera permission — triggered by click ✓
  try {
    localStream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode:'user', width:{ideal:1280}, height:{ideal:720} },
      audio: { echoCancellation:true, noiseSuppression:true, sampleRate:48000 }
    });
  } catch(e) {
    btn.disabled = false;
    btn.textContent = '▶ Try Again';
    const pb = $('permBox');
    pb.style.display = 'block';
    if (e.name==='NotAllowedError'||e.name==='PermissionDeniedError') {
      pb.innerHTML = '🔒 <strong>Camera blocked.</strong><br>Click the 🔒 icon in your browser address bar → allow Camera &amp; Microphone → refresh.';
    } else if (e.name==='NotFoundError') {
      pb.textContent = '📷 No camera detected. Please connect a camera and try again.';
    } else {
      pb.textContent = '⚠️ Camera error: ' + (e.message||e.name);
    }
    return;
  }

  // Show local preview
  const lv = $('localVideo');
  lv.srcObject = localStream;
  lv.style.display = 'block';

  // Init PeerJS (WebRTC broker)
  setStatus('⏳ Connecting…');
  await initPeer();

  // Init Firebase matchmaking
  initFirebase();

  showScreen(null);
  $('rph').style.display = 'flex';
  setStatus('✅ Ready');

  // Auto-search
  setTimeout(findNext, 400);
}

// ── PeerJS ────────────────────────────────────────────
function initPeer() {
  return new Promise(res => {
    if (peer) try { peer.destroy(); } catch(e){}
    peer = new Peer({ debug:0 });
    peer.on('open', id => { myPeerId=id; res(); });
    peer.on('call', inc => { if (!currentCall) answerCall(inc); });
    peer.on('error', err => {
      if (err.type !== 'peer-unavailable') setStatus('⚠ Network issue');
    });
    setTimeout(res, 5000); // don't block forever
  });
}

// ── Matchmaking ───────────────────────────────────────
function findNext() {
  endCurrentCall();
  if (!myPeerId) { setTimeout(findNext, 600); return; }
  startSearch();
}

function startSearch() {
  isSearching = true; searchSec = 0;
  showScreen('scS');
  $('rph').style.display = 'flex';
  $('remoteVideo').srcObject = null;
  setStatus('🔍 Searching…');
  $('sTimer').textContent = 'Searching 0s';
  searchTimer = setInterval(()=>{ searchSec++; $('sTimer').textContent=`Searching ${searchSec}s`; }, 1000);

  if (db) {
    matchFirebase();
  } else {
    matchBroadcast();  // Same-browser tab fallback
  }
}

function cancelSearch() {
  isSearching = false;
  clearInterval(searchTimer);
  cleanupLobby();
  showScreen(null);
  $('rph').style.display = 'flex';
  setStatus('Ready — tap NEXT');
}

// Firebase lobby: write our ID → read for partner → call them
function matchFirebase() {
  const lobby = db.ref('lobby');

  // First: look for someone already waiting
  lobby.orderByChild('ts').limitToFirst(10).once('value', snap => {
    if (!isSearching) return;
    let partner = null;
    snap.forEach(c => {
      const d = c.val();
      if (d.peerId && d.peerId !== myPeerId && !partner) {
        partner = { key: c.key, peerId: d.peerId };
      }
    });

    if (partner) {
      // Claim them (remove from lobby) and call
      lobby.child(partner.key).remove();
      stopSearch();
      callPeer(partner.peerId);
    } else {
      // Add ourselves and wait
      myLobbyRef = lobby.push({ peerId: myPeerId, ts: firebase.database.ServerValue.TIMESTAMP });
      myLobbyRef.onDisconnect().remove();

      // Poll every 2.5s for new arrivals
      lobbyPollTimer = setInterval(() => {
        if (!isSearching) { clearInterval(lobbyPollTimer); return; }
        lobby.orderByChild('ts').limitToFirst(10).once('value', snap2 => {
          if (!isSearching) return;
          snap2.forEach(c => {
            const d = c.val();
            if (d.peerId && d.peerId !== myPeerId) {
              lobby.child(c.key).remove();
              stopSearch();
              callPeer(d.peerId);
            }
          });
        });
      }, 2500);
    }
  });
}

// BroadcastChannel: works across tabs in same browser
function matchBroadcast() {
  try {
    bc = new BroadcastChannel('rl_v2');
    bc.onmessage = e => {
      const m = e.data;
      if (!isSearching) return;
      if (m.type==='hello' && m.from && m.from!==myPeerId) {
        bc.postMessage({ type:'match', from:myPeerId, to:m.from });
        stopSearch();
        callPeer(m.from);
      }
      if (m.type==='match' && m.to===myPeerId) {
        stopSearch();
      }
    };
    bc.postMessage({ type:'hello', from:myPeerId });
  } catch(e) {
    setStatus('⚠ Open 2 tabs to test locally');
  }
}

function stopSearch() {
  isSearching = false;
  clearInterval(searchTimer);
  clearInterval(lobbyPollTimer);
  cleanupLobby();
}

function cleanupLobby() {
  if (myLobbyRef) { myLobbyRef.remove(); myLobbyRef=null; }
  clearInterval(lobbyPollTimer);
  if (bc) { try{bc.close();}catch(e){} bc=null; }
}

// ── Calling ───────────────────────────────────────────
function callPeer(peerId) {
  if (!peer || !localStream || !peerId) return;
  showScreen(null);
  setStatus('📡 Ringing…');
  const call = peer.call(peerId, localStream);
  setupCall(call);
}

function answerCall(call) {
  stopSearch();
  cleanupLobby();
  showScreen(null);
  setStatus('📡 Connecting…');
  call.answer(localStream);
  setupCall(call);
}

function setupCall(call) {
  currentCall = call;
  let gotStream = false;

  call.on('stream', remote => {
    gotStream = true;
    $('remoteVideo').srcObject = remote;
    $('rph').style.display = 'none';
    showScreen(null);
    setStatus('🟢 Connected','ok');
    startCallTimer();
  });

  call.on('close', onDisconnect);
  call.on('error', onDisconnect);

  // Timeout if peer never sends stream
  setTimeout(() => {
    if (!gotStream && currentCall===call) onDisconnect();
  }, 12000);
}

function onDisconnect() {
  stopCallTimer();
  currentCall = null;
  $('remoteVideo').srcObject = null;
  $('rph').style.display = 'flex';
  showScreen('scD');
  setStatus('');
}

function endCurrentCall() {
  stopSearch();
  stopCallTimer();
  cleanupLobby();
  if (currentCall) { try{currentCall.close();}catch(e){} currentCall=null; }
  $('remoteVideo').srcObject = null;
}

function goHome() {
  endCurrentCall();
  showScreen(null);
  $('rph').style.display = 'flex';
  setStatus('Ready — tap NEXT to find someone');
}

// ── Media controls ────────────────────────────────────
function toggleMute() {
  isMuted = !isMuted;
  localStream?.getAudioTracks().forEach(t => t.enabled=!isMuted);
  const b=$('muteBtn'); b.textContent=isMuted?'🔇':'🎙️'; b.classList.toggle('off',isMuted);
}

function toggleVideo() {
  isVidOff = !isVidOff;
  localStream?.getVideoTracks().forEach(t => t.enabled=!isVidOff);
  const b=$('vidBtn'); b.textContent=isVidOff?'🚫':'📹'; b.classList.toggle('off',isVidOff);
}

async function flipCamera() {
  usingFront = !usingFront;
  $('flipBtn').style.opacity='.35';
  try {
    const ns = await navigator.mediaDevices.getUserMedia({
      video:{ facingMode: usingFront?'user':'environment' }, audio:false
    });
    const nv = ns.getVideoTracks()[0];
    const ov = localStream.getVideoTracks()[0];
    localStream.removeTrack(ov); ov.stop();
    localStream.addTrack(nv);
    $('localVideo').srcObject = null;
    $('localVideo').srcObject = localStream;
    if (currentCall) {
      const s = currentCall.peerConnection?.getSenders?.()?.find(s=>s.track?.kind==='video');
      if (s) await s.replaceTrack(nv);
    }
    $('flipBtn').classList.toggle('fb', !usingFront);
  } catch(e) { usingFront=!usingFront; }
  $('flipBtn').style.opacity='1';
}

// Tap PiP to bring it front
$('localVideo').addEventListener('click', () => {
  const lv=$('localVideo'), rv=$('remoteVideo');
  [lv.style.zIndex, rv.style.zIndex] = lv.style.zIndex==='20' ? ['10',''] : ['20',''];
  lv.style.inset = lv.style.zIndex==='20' ? '0' : '';
  lv.style.width  = lv.style.zIndex==='20' ? '100%' : '';
  lv.style.height = lv.style.zIndex==='20' ? '100%' : '';
  lv.style.borderRadius = lv.style.zIndex==='20' ? '0' : '';
  lv.style.bottom = lv.style.zIndex==='20' ? '0' : '16px';
  lv.style.right  = lv.style.zIndex==='20' ? '0' : '16px';
});

// ── Timers ────────────────────────────────────────────
function startCallTimer() {
  callSec=0; stopCallTimer();
  callTimer = setInterval(()=>{
    callSec++;
    const m=String(Math.floor(callSec/60)).padStart(2,'0');
    const s=String(callSec%60).padStart(2,'0');
    setStatus(`🟢 ${m}:${s}`,'ok');
  },1000);
}
function stopCallTimer(){ clearInterval(callTimer); callTimer=null; }

window.addEventListener('beforeunload', ()=>{
  endCurrentCall();
  if(peer) peer.destroy();
  localStream?.getTracks().forEach(t=>t.stop());
});
</script>
</body>
</html>
