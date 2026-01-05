<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>HG KEY SYSTEM</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;500;700&display=swap');

* { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Poppins', sans-serif; }
html,body { height: 100%; }
body {
    background: #000;
    color: #fff;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
}

.background {
    position: absolute;
    width: 200%;
    height: 200%;
    background: radial-gradient(circle at center, #7b00ff33, transparent 60%);
    animation: pulse 6s infinite alternate;
    z-index: 0;
}

@keyframes pulse { from { transform: scale(1); } to { transform: scale(1.2); } }

.container {
    position: relative;
    background: rgba(10,10,10,0.88);
    border-radius: 18px;
    padding: 30px;
    width: 380px;
    box-shadow: 0 0 45px #7b00ff55;
    text-align: center;
    z-index: 1;
}

h1 { font-size: 24px; margin-bottom: 8px; color: #b266ff; }
p { font-size: 13px; color: #aaa; margin-bottom: 18px; }

.key-box {
    background: #111;
    padding: 14px;
    border-radius: 10px;
    margin-bottom: 12px;
    word-break: break-all;
    border: 1px solid #7b00ff55;
    font-size: 14px;
    min-height: 44px;
    display:flex;
    align-items:center;
    justify-content:center;
}

.buttons {
    display: flex;
    gap: 10px;
}

button {
    flex: 1;
    padding: 12px;
    background: linear-gradient(135deg, #7b00ff, #b266ff);
    border: none;
    border-radius: 10px;
    color: #fff;
    font-size: 14px;
    cursor: pointer;
    transition: 0.25s;
}
button[disabled] { opacity: 0.55; cursor: not-allowed; transform: none; box-shadow: none; }
button:hover:not([disabled]) { transform: scale(1.05); box-shadow: 0 0 18px #7b00ffaa; }

.timer { margin-top: 15px; font-size: 14px; }
.valid { color: #00ff88; }
.invalid { color: #ff4444; }

.copied { color: #00ffcc; font-size: 12px; margin-top: 6px; min-height: 18px; }
</style>
</head>

<body>

<div class="background" aria-hidden="true"></div>

<div class="container" role="region" aria-label="Gerador de chave">
    <h1>HG KEY SYSTEM</h1>
    <p>Key válida por 30 minutos</p>

    <div class="key-box" id="keyBox" aria-live="polite">Nenhuma key gerada</div>

    <div class="buttons" role="group" aria-label="Controles">
        <button id="generateBtn" onclick="generateKey()">GERAR KEY</button>
        <button id="copyBtn" onclick="copyKey()">COPIAR</button>
    </div>

    <div class="timer" id="timerStatus">
        Status: <span class="invalid">INVÁLIDA</span>
    </div>

    <div class="copied" id="copyMsg" aria-live="polite"></div>
</div>

<script>
const KEY_TIME = 30 * 60 * 1000; // 30 minutos
const PREFIX = "Hg'x.";

function randomKey() {
    const chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    let key = PREFIX;
    for (let i = 0; i < 15; i++) {
        key += chars.charAt(Math.floor(Math.random() * chars.length));
        if ((i + 1) % 5 === 0 && i !== 15) key += "-";
    }
    return key;
}

function storeKey(key, expire) {
    localStorage.setItem("hg_mod_key", key);
    localStorage.setItem("hg_mod_key_expire", String(expire));
}

function clearStoredKey() {
    localStorage.removeItem("hg_mod_key");
    localStorage.removeItem("hg_mod_key_expire");
}

function setKeyBox(text) {
    document.getElementById("keyBox").innerText = text || "Nenhuma key gerada";
}

function setCopyMsg(text, timeout = 2000) {
    const el = document.getElementById("copyMsg");
    el.innerText = text || "";
    if (timeout > 0) {
        clearTimeout(el._t);
        el._t = setTimeout(() => el.innerText = "", timeout);
    }
}

function enableControls(enabled) {
    document.getElementById("generateBtn").disabled = !enabled;
    document.getElementById("copyBtn").disabled = !enabled;
}

function generateKey(force = false) {
    const existing = localStorage.getItem("hg_mod_key");
    const expireRaw = localStorage.getItem("hg_mod_key_expire");
    const expireNum = expireRaw ? parseInt(expireRaw, 10) : 0;

    if (!force && existing && Date.now() <= expireNum) {
        setCopyMsg("Já existe uma key válida. Regerar para substituir.", 2500);
        return;
    }

    const key = randomKey();
    const expire = Date.now() + KEY_TIME;
    storeKey(key, expire);
    setKeyBox(key);
    setCopyMsg("Key gerada com sucesso.", 1500);
    updateStatus();
}

function copyKey() {
    const key = localStorage.getItem("hg_mod_key");
    const expireRaw = localStorage.getItem("hg_mod_key_expire");
    const expireNum = expireRaw ? parseInt(expireRaw, 10) : 0;

    if (!key || Date.now() > expireNum) {
        setCopyMsg("Nenhuma key válida para copiar.", 2000);
        return;
    }

    if (navigator.clipboard && window.isSecureContext) {
        navigator.clipboard.writeText(key).then(() => {
            setCopyMsg("✔ Key copiada com sucesso", 1800);
        }).catch(() => {
            fallbackCopy(key);
        });
    } else {
        fallbackCopy(key);
    }
}

function fallbackCopy(text) {
    const ta = document.createElement("textarea");
    ta.value = text;
    ta.style.position = "fixed";
    ta.style.left = "-9999px";
    document.body.appendChild(ta);
    ta.select();
    try {
        document.execCommand('copy');
        setCopyMsg("✔ Key copiada (fallback)", 1800);
    } catch (e) {
        setCopyMsg("Falha ao copiar", 1800);
    } finally {
        document.body.removeChild(ta);
    }
}

function formatRemaining(ms) {
    if (ms <= 0) return "0s";
    const s = Math.floor(ms / 1000);
    const m = Math.floor(s / 60);
    const sec = s % 60;
    if (m > 0) return `${m}m ${sec}s`;
    return `${sec}s`;
}

function updateStatus() {
    const key = localStorage.getItem("hg_mod_key");
    const expireRaw = localStorage.getItem("hg_mod_key_expire");
    const expireNum = expireRaw ? parseInt(expireRaw, 10) : 0;

    if (!key || Date.now() > expireNum) {
        // expired or none
        clearStoredKey();
        setKeyBox("Nenhuma key gerada");
        document.getElementById("timerStatus").innerHTML =
            'Status: <span class="invalid">INVÁLIDA</span>';
        enableControls(true);
        return;
    }

    // valid
    setKeyBox(key);
    const remaining = expireNum - Date.now();
    document.getElementById("timerStatus").innerHTML =
        `Status: <span class="valid">VÁLIDA</span> | Expira em ${formatRemaining(remaining)}`;
    enableControls(true); // allow copying; generating is allowed but could be disabled if desired
}

// Atualiza a UI imediatamente ao carregar (mostra key existente)
(function init() {
    const key = localStorage.getItem("hg_mod_key");
    const expireRaw = localStorage.getItem("hg_mod_key_expire");
    const expireNum = expireRaw ? parseInt(expireRaw, 10) : 0;

    if (key && Date.now() <= expireNum) {
        setKeyBox(key);
        // keep controls enabled: allow copy; if you prefer, disable generate: document.getElementById("generateBtn").disabled = true;
    } else {
        // cleanup if expired
        clearStoredKey();
        setKeyBox("Nenhuma key gerada");
    }

    updateStatus();
    // Atualiza a cada segundo
    setInterval(updateStatus, 1000);
})();
</script>

</body>
</html>
