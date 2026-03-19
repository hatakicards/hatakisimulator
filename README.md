<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <title>HATAKI TCG - Simulator Pro</title>
    <style>
        :root {
            --common: #7f8c8d; --rare: #2980b9; --epic: #8e44ad;
            --legendary: #f1c40f; --special: #e74c3c;
            --unique: #00ffff;
            --hataki: #b8860b;
            --bg-default: radial-gradient(circle, #222 0%, #000 100%);
            --bg-epic: radial-gradient(circle, #4b0082 0%, #000 100%);
            --bg-legendary: radial-gradient(circle, #b8860b 0%, #000 100%);
            --bg-unique: radial-gradient(circle, #001a1a 0%, #000 100%);
        }

        body { background: #0a0a0a; color: white; font-family: 'Segoe UI', sans-serif; margin: 0; overflow: hidden; }
        
        .page { display: none; width: 100%; height: 100vh; overflow-y: auto; transition: background 1s ease; }
        .page.active { display: block; }

        #opening { background: var(--bg-default); overflow: hidden; position: relative; transition: background 1s ease-in-out; }
        #opening.bg-epic { background: var(--bg-epic) !important; }
        #opening.bg-legendary { background: var(--bg-legendary) !important; }
        #opening.bg-unique { 
            background: radial-gradient(circle, #003333 0%, #000 100%) !important;
            box-shadow: inset 0 0 100px #00ffff55;
        }
        #opening.bg-special { 
            background: linear-gradient(45deg, #ff0000, #ff7f00, #ffff00, #00ff00, #0000ff, #4b0082, #8b00ff) !important;
            background-size: 400% 400% !important; animation: rainbow-bg 5s ease infinite !important;
        }
        @keyframes rainbow-bg { 0% { background-position: 0% 50%; } 50% { background-position: 100% 50%; } 100% { background-position: 0% 50%; } }

        #loader { position: fixed; inset: 0; background: #000; z-index: 9999; display: none; flex-direction: column; justify-content: center; align-items: center; }
        .spinner { width: 50px; height: 50px; border: 5px solid #333; border-top-color: var(--special); border-radius: 50%; animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }

        .header { padding: 30px; text-align: center; background: #151515; border-bottom: 4px solid var(--special); position: relative; }
        
        #logo-clicker {
            font-size: 3em; font-weight: 900; color: var(--legendary);
            text-shadow: 0 0 15px var(--hataki); cursor: pointer; user-select: none;
            display: inline-block; transition: 0.1s; margin: 0;
        }
        #logo-clicker:active { transform: scale(0.95); filter: brightness(1.5); }
        
        .floating-coin {
            position: fixed; color: #f1c40f; font-weight: bold; font-size: 28px;
            pointer-events: none; z-index: 9999; animation: floatUp 1s ease-out forwards;
            text-shadow: 0 0 5px #000, 0 0 10px #f1c40f;
        }
        @keyframes floatUp {
            0% { transform: translateY(0) scale(1); opacity: 1; }
            100% { transform: translateY(-70px) scale(1.5); opacity: 0; }
        }

        .stats-toggle-btn { background: var(--special); color: white; border: none; padding: 10px 25px; border-radius: 8px; cursor: pointer; font-weight: bold; margin: 20px auto; display: block; transition: 0.3s; }
        .stats-toggle-btn:hover { filter: brightness(1.2); }

        .stats-box { background: #111; border: 2px solid #333; border-radius: 15px; margin: 20px auto; padding: 20px; max-width: 800px; display: none; flex-wrap: wrap; justify-content: space-around; gap: 15px; animation: fadeIn 0.5s ease; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }

        .stats-box h3 { width: 100%; text-align: center; margin-top: 0; color: var(--special); text-transform: uppercase; font-size: 14px; letter-spacing: 2px; }
        .stat-item { text-align: center; }
        .stat-item span { display: block; font-size: 12px; color: #888; text-transform: uppercase; }
        .stat-item b { font-size: 18px; color: white; }

        .txt-epic { color: #bf00ff !important; text-shadow: 0 0 5px #bf00ff; }
        .txt-legendary { color: #f1c40f !important; text-shadow: 0 0 5px #f1c40f; }
        .txt-unique { color: var(--unique) !important; text-shadow: 0 0 10px var(--unique); font-weight: bold; }
        .txt-special-anim { background: linear-gradient(to right, #ef5350, #f48fb1, #7e57c2, #2196f3, #26c6da, #43a047, #eeff41, #f9a825, #ff5722); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-size: 400% 400%; animation: rainbow-bg 3s linear infinite; font-weight: 900; }
        
        .pack-grid { display: flex; justify-content: center; gap: 25px; padding: 0; flex-wrap: wrap; flex: 1; }
        .pack-box { cursor: pointer; width: 190px; transition: 0.3s; text-align: center; background: #1a1a1a; padding: 15px; border-radius: 15px; border: 2px solid #333; }
        .pack-box:hover { transform: translateY(-10px); border-color: var(--legendary); }
        .pack-box img { width: 100%; border-radius: 8px; margin-bottom: 10px; background: #222; min-height: 250px; object-fit: cover; }
        .pack-cost { font-weight: bold; color: #f1c40f; font-size: 1.1em; display: none; align-items: center; justify-content: center; gap: 5px; }

        #stage { width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; perspective: 1500px; }
        .card { width: 275px; height: 395px; position: absolute; opacity: 0; transform: scale(0); transition: 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); cursor: pointer; }
        .card.show { opacity: 1; transform: scale(1); }
        .card.flipped { transform: scale(1.15) !important; z-index: 999 !important; }
        .card-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
        .card.flipped .card-inner { transform: rotateY(180deg); }
        .card-face { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; border-radius: 15px; border: 5px solid #333; overflow: hidden; background-color: #222; }
        .card-back { background: #222 url('retro.png') center/cover; z-index: 2; }
        .card-front { transform: rotateY(180deg); background-size: cover; background-position: center; z-index: 1; }

        .card-price-tag { position: absolute; bottom: 5px; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.8); color: #2ecc71; padding: 3px 10px; border-radius: 20px; font-weight: bold; border: 1px solid #2ecc71; font-size: 12px; z-index: 10; display: flex; align-items: center; gap: 4px; }
        
        .new-badge { position: absolute; top: 10px; right: 10px; background: #e74c3c; color: white; padding: 5px 10px; font-weight: bold; border-radius: 8px; z-index: 20; border: 2px solid white; box-shadow: 0 0 10px rgba(0,0,0,0.5); animation: pulse 1s infinite; letter-spacing: 1px; font-size: 12px; }
        @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.1); } 100% { transform: scale(1); } }

        @keyframes rumble {
            0% { transform: translate(0,0) rotate(0); }
            10% { transform: translate(-4px, -4px) rotate(-1.5deg); }
            20% { transform: translate(4px, 0px) rotate(1.5deg); }
            30% { transform: translate(0px, 4px) rotate(0); }
            40% { transform: translate(4px, 4px) rotate(1.5deg); }
            50% { transform: translate(-4px, 0px) rotate(-1.5deg); }
            60% { transform: translate(0px, -4px) rotate(0); }
            100% { transform: translate(0,0) rotate(0); }
        }
        .rumbling { animation: rumble 0.1s infinite !important; z-index: 1000 !important; }

        /* Nuova Animazione Particelle a Puntini */
        .particle-container {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 100vw; height: 100vh; pointer-events: none; z-index: -1; overflow: hidden;
        }
        .dot-particle {
            position: absolute; width: 6px; height: 6px; border-radius: 50%; opacity: 0;
        }
        .card.rumbling .dot-particle { animation: particleIn 0.8s ease-in infinite; }

        @keyframes particleIn {
            0% { transform: translate(var(--dx), var(--dy)) scale(1.5); opacity: 0; }
            20% { opacity: 1; }
            100% { transform: translate(0, 0) scale(0); opacity: 0; }
        }

        .star-burst { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 400px; height: 400px; z-index: 1100; pointer-events: none; clip-path: polygon(50% 0%, 61% 35%, 100% 50%, 61% 65%, 50% 100%, 39% 65%, 0% 50%, 39% 35%); opacity: 0; }
        .star-legendary { background: radial-gradient(circle, #fff, var(--legendary)); box-shadow: 0 0 100px var(--legendary); animation: starPop 1.3s ease-out forwards; }
        .star-special { background: linear-gradient(45deg, #f00, #ff0, #0f0, #0ff, #00f, #f0f, #f00); background-size: 400% 400%; animation: starPop 1.3s ease-out forwards, rainbow-bg 2s linear infinite; }
        .star-unique { background: radial-gradient(circle, #fff, #00ffff); box-shadow: 0 0 150px #00ffff; animation: starPop 1.6s ease-out forwards; }
        @keyframes starPop { 0% { transform: translate(-50%, -50%) scale(0) rotate(0deg); opacity: 1; } 50% { opacity: 1; } 100% { transform: translate(-50%, -50%) scale(5) rotate(180deg); opacity: 0; } }

        #home-btn-container { position: absolute; bottom: 30px; width: 100%; display: flex; justify-content: center; z-index: 1000; }
        .nav-btn { display: none; padding: 15px 40px; background: var(--special); border: none; color: white; cursor: pointer; border-radius: 8px; font-weight: bold; }

        .rarity-EPICO { border-color: var(--epic) !important; }
        .rarity-LEGGENDARIO { border-color: var(--legendary) !important; box-shadow: inset 0 0 20px var(--legendary); }
        .rarity-SPECIALE { border-color: var(--special) !important; box-shadow: inset 0 0 30px var(--special); }
        .rarity-UNIQUE { 
            border-color: #00ffff !important; 
            box-shadow: 0 0 15px #00ffff, inset 0 0 10px #00ffff, 0 0 25px #00ffff;
            animation: unique-flame 1.5s infinite alternate ease-in-out;
        }

        @keyframes unique-flame {
            0% { border-color: #00ffff; box-shadow: 0 0 10px #00ffff, 0 0 20px #00e6e6; }
            50% { border-color: #e0ffff; box-shadow: 0 0 30px #00ffff, 0 0 50px #b0fbff; }
            100% { border-color: #00ffff; box-shadow: 0 0 12px #00ffff, 0 0 22px #00cccc; }
        }

        #collection-grid { display: flex; flex-wrap: wrap; justify-content: center; gap: 8px; padding: 20px; }
        .c-item { width: 105px; height: 145px; border-radius: 8px; border: 2px solid #444; background-size: cover; background-position: center; position: relative; overflow: hidden; cursor: pointer; transition: 0.2s; }
        .dup-badge { position: absolute; top: 5px; right: 5px; background: var(--special); color: white; font-size: 10px; font-weight: bold; padding: 2px 6px; border-radius: 10px; border: 1px solid white; z-index: 5; }

        .sell-all-btn { background: #27ae60; color: white; border: none; padding: 10px 20px; border-radius: 8px; cursor: pointer; font-weight: bold; margin: 10px auto; display: none; }

        .pack-wrapper { width: 280px; height: 400px; position: relative; cursor: pointer; z-index: 100; }
        .pack-part { position: absolute; width: 100%; height: 50%; left: 0; background-size: 280px 400px; background-repeat: no-repeat; transition: 0.8s cubic-bezier(0.5, 0, 0.5, 1); }
        .p-top { top: 0; background-position: top; border-radius: 15px 15px 0 0; }
        .p-bottom { bottom: 0; background-position: bottom; border-radius: 0 0 15px 15px; }
        .pack-wrapper.ripped .p-top { transform: translateY(-150%) rotate(-20deg); opacity: 0; }
        .pack-wrapper.ripped .p-bottom { transform: translateY(150%) rotate(20deg); opacity: 0; }

        #viewer { position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 2000; display: none; justify-content: center; align-items: center; cursor: zoom-out; backdrop-filter: blur(8px); }
        #viewer .big-card { width: 320px; height: 460px; border-radius: 20px; border: 6px solid #444; background-size: cover; background-position: center; position: relative; }
        #viewer .ui-overlay { position: absolute; bottom: -80px; width: 100%; display: flex; justify-content: center; gap: 20px; }
        .action-btn { padding: 10px 20px; border: none; border-radius: 5px; font-weight: bold; cursor: pointer; color: white; }
        .sell-btn { background: #27ae60; display: flex; align-items: center; gap: 5px; }

        .currency-display { position: absolute; top: 30px; right: 30px; background: rgba(0,0,0,0.8); border: 2px solid var(--hataki); padding: 10px 20px; border-radius: 12px; display: flex; align-items: center; gap: 15px; }
        .wallet-coins { color: #f1c40f; font-size: 1.4em; font-weight: bold; display: flex; align-items: center; gap: 8px; }

        #dup-summary { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: #151515; border: 3px solid var(--hataki); border-radius: 20px; padding: 30px; z-index: 2000; text-align: center; width: 80%; max-width: 500px; display: none; }
        #dup-grid { display: flex; flex-wrap: wrap; gap: 10px; justify-content: center; margin-bottom: 20px; max-height: 250px; overflow-y: auto; background: #0a0a0a; padding: 15px; border-radius: 10px; }
        .dup-img { width: 60px; height: 85px; object-fit: cover; border-radius: 4px; }

        #quest-trigger { position: fixed; left: 20px; bottom: 20px; width: 60px; height: 60px; background: var(--legendary); border-radius: 12px; display: flex; justify-content: center; align-items: center; font-size: 30px; cursor: pointer; z-index: 1000; border: 2px solid #fff; }
        #quest-panel { position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 5001; display: none; justify-content: center; align-items: center; }
        .quest-box { background: #151515; width: 90%; max-width: 450px; border-radius: 20px; border: 3px solid var(--legendary); padding: 25px; position: relative; }
        .quest-list { list-style: none; padding: 0; margin-top: 20px; }
        .quest-item { background: #222; margin-bottom: 12px; padding: 15px; border-radius: 10px; border-left: 5px solid #444; }
        .quest-item.completed { border-left-color: #f1c40f; background: #2a2a1a; cursor: pointer; }
        .quest-item.claimed { opacity: 0.5; border-left-color: #27ae60; }
        
        .holo { position: absolute; inset: -100%; background: linear-gradient(115deg, rgba(255,0,0,0.2) 0%, rgba(255,255,0,0.2) 20%, rgba(0,255,0,0.2) 40%, rgba(0,255,255,0.2) 60%, rgba(0,0,255,0.2) 80%, rgba(255,0,255,0.2) 100%); animation: h-shift 3s infinite linear; pointer-events: none; z-index: 6; }
        @keyframes h-shift { 0% { transform: translateX(-30%) translateY(-30%); } 100% { transform: translateX(30%) translateY(30%); } }
        .glass { position: absolute; inset: 0; background: linear-gradient(135deg, rgba(255,255,255,0.4) 2px, transparent 2px); background-size: 50px 50px; pointer-events: none; z-index: 7; }
        .holo-unique { position: absolute; inset: 0; background: repeating-linear-gradient(45deg, transparent, transparent 10px, rgba(0,255,255,0.1) 10px); animation: rainbow-holo 2s linear infinite; z-index: 8; }
        @keyframes rainbow-holo { 0% { filter: hue-rotate(0deg); } 100% { filter: hue-rotate(360deg); } }
        .icon-coin { width: 18px; height: 18px; border-radius: 50%; vertical-align: middle; }
    </style>
</head>
<body>

<div id="loader"><div class="spinner"></div><p>CARICAMENTO CARTE...</p></div>

<div id="quest-trigger" onclick="toggleQuests()">📓</div>

<div id="quest-panel">
    <div class="quest-box">
        <span onclick="toggleQuests()" style="position:absolute; top:15px; right:15px; font-size:24px; cursor:pointer;">&times;</span>
        <h2 style="text-align:center; color:var(--legendary);">MISSIONI GIORNALIERE</h2>
        <div class="quest-list" id="quest-list-ui"></div>
    </div>
</div>

<div id="custom-popup" style="position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 5000; display: none; justify-content: center; align-items: center;">
    <div style="background: #151515; padding: 40px; border-radius: 20px; border: 3px solid var(--hataki); text-align: center; max-width: 400px;">
        <img id="pop-img" src="" style="width: 120px; border-radius: 10px; margin-bottom: 20px;">
        <p id="pop-text" style="font-size: 1.2em;"></p>
        <div style="display: flex; gap: 20px; justify-content: center; margin-top: 30px;">
            <button id="pop-confirm" style="padding: 12px 30px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; background: var(--legendary); color: black;">CONFERMA</button>
            <button id="pop-cancel" onclick="closePopup()" style="padding: 12px 30px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; background: #444; color: white;">ANNULLA</button>
        </div>
    </div>
</div>

<div id="viewer" onclick="closeViewer()">
    <div style="position: relative;" onclick="event.stopPropagation()">
        <div id="viewer-content" class="big-card"></div>
        <div class="ui-overlay">
            <button class="action-btn sell-btn" id="sell-action-btn">VENDI PER <span id="sell-value">0</span> <img src="moneta.jpg" class="icon-coin"></button>
            <button class="action-btn" onclick="closeViewer()" style="background: var(--special);">CHIUDI</button>
        </div>
    </div>
</div>

<div id="home" class="page active">
    <div class="header">
        <div id="logo-clicker" onclick="scratchLogo(event)">HATAKI SIMULATOR</div>
        <div class="currency-display">
            <span class="wallet-coins"><img src="moneta.jpg" class="icon-coin"> <b id="h-coins">0</b></span>
        </div>
    </div>
    <button class="stats-toggle-btn" onclick="toggleStats()">VISUALIZZA STATISTICHE</button>
    <div class="stats-box" id="main-stats-box">
        <h3>Statistiche Generali</h3>
        <div class="stat-item"><span>Pacchetti Aperti</span><b id="s-packs">0</b></div>
        <div class="stat-item"><span>Valore Euro (Statistica)</span><b id="s-wallet-euro" style="color: #2ecc71;">0.00€</b></div>
        <div class="stat-item"><span>Click Effettuati</span><b id="s-clicks" style="color: #f1c40f;">0</b></div>
        <div class="stat-item"><span>Valore Collezione (Euro)</span><b id="s-coll-val-euro" style="color: #2ecc71;">0.00€</b></div>
        <div class="stat-item"><span class="txt-epic">Epiche</span><b id="s-e">0</b></div>
        <div class="stat-item"><span class="txt-legendary">Leggendarie</span><b id="s-l">0</b></div>
        <div class="stat-item" id="stat-unique-box" style="display: none;"><span class="txt-unique">Gold</span><b id="s-u">0</b></div>
        <div class="stat-item"><span class="txt-special-anim">Speciali</span><b id="s-s">0</b></div>
    </div>
    
    <div style="display: flex; justify-content: center; flex-wrap: wrap; gap: 40px; max-width: 1200px; margin: 40px auto 0 auto;">
        <div class="pack-grid" style="padding: 0; margin-top: 0; flex: 1; display: flex; justify-content: center; gap: 25px; flex-wrap: wrap;">
            <div class="pack-box" onclick="preOpen('base pack.png', 'base', 40)"><img src="base pack.png"><p>BASE PACK</p></div>
            <div class="pack-box" onclick="preOpen('super pack.png', 'super', 100)"><img src="super pack.png"><p>SUPER PACK</p></div>
            <div class="pack-box" onclick="preOpen('legendary pack.png', 'legendary', 180)"><img src="legendary pack.png"><p>LEGENDARY PACK</p></div>
        </div>
        
        <div style="text-align: center; border: 2px dashed var(--legendary); padding: 20px; border-radius: 15px; background: #111; min-width: 160px; height: fit-content;">
            <h3 style="color: var(--legendary); margin-top: 0; margin-bottom: 15px; text-transform: uppercase; font-size: 14px; letter-spacing: 2px;">RANDOM CARDS</h3>
            <div style="display: flex; flex-direction: column; gap: 15px; align-items: center;">
                <div class="pack-box" style="width: 120px; padding: 10px; border-color: var(--legendary);" onclick="preOpen('random1.jpg', 'random1', 250)">
                    <img src="random1.jpg" style="min-height: 160px;">
                </div>
                <div class="pack-box" style="width: 120px; padding: 10px;" onclick="preOpen('random2.jpg', 'random2', 20)">
                    <img src="random2.jpg" style="min-height: 160px;">
                </div>
            </div>
        </div>
    </div>

    <div style="text-align:center; padding-bottom: 50px; margin-top: 50px;">
        <h3>LA TUA COLLEZIONE (<span id="coll-count">0</span>)</h3>
        <button id="btn-sell-all-dups" class="sell-all-btn" onclick="sellAllDuplicates()">VENDI TUTTI I DOPPIONI</button>
        <div id="collection-grid"></div>
    </div>
</div>

<div id="opening" class="page">
    <div id="stage"></div>
    <div id="dup-summary">
        <h3 style="color:var(--special);">DOPPIONI TROVATI</h3>
        <div id="dup-grid"></div>
        <p>Valore Netto: <b id="dup-net" style="color:#f1c40f;">0</b> <img src="moneta.jpg" class="icon-coin"></p>
        <div style="display:flex; gap:15px; justify-content:center; margin-top: 20px;">
            <button id="btn-sell-dups" style="padding:10px 20px; background:#27ae60; color:white; border:none; border-radius:8px; cursor:pointer; font-weight:bold;">VENDI TUTTO</button>
            <button style="padding:10px 20px; background:#444; color:white; border:none; border-radius:8px; cursor:pointer;" onclick="closeDupSummary()">IGNORA</button>
        </div>
    </div>
    <div id="home-btn-container"><button id="home-btn" class="nav-btn" onclick="goHome()">TORNA ALLA HOME</button></div>
</div>

<script>
const DB = {
    COMUNE: ["creep.png", "fire monster.png", "fukushimo.png", "ice.png", "kaito.png", "kean retsu prototipo.png", "murasaki.png","nemuru.png", "raizen.png", "ren.png", "richiri.png", "rock.png", "SAKURA.png", "shimoto.png", "shirabe.png", "sir.png", "tadashi.png", "timuru.png", "wild monster.png", "wind golem.png", "gurumaru.png", "kaori.png"], 
    RARO: ["akira sato fuoco 2.png", "cristals.png", "haruto.png", "Immagine1.png", "kaori 2.png", "niturne.png", "RAIZEN 2.png", "RETSU 2.png", "shimoto 2.png", "shirabe 2.png", "spikes.png", "TADASHI 2.png"],
    EPICO: ["akira fuoco 3.png", "akira ghiaccio 3.png", "alien.png", "aniki.png", "creep king.png", "element.png", "HAKARI 2.png", "hakari.png", "HARUTO 2.png", "HARUTO 3.png", "iron.png", "kage.png", "KAORI 3.png", "LOGTH.png", "MAGIC.png", "maze.png", "mountain.png", "oshimoto.png", "rae.png", "RAIZEN 3.png", "retsu 3.png", "rin.png", "shadow.png", "SHIRABE 3.png", "TADASHI 3.png", "yoiichi.png"],
    LEGGENDARIO: ["fire god.png", "GOD.png", "HAKARI 3.png", "ice god.png", "KAGE 2.png", "maze 2.png", "NITURNE 3.png", "RAE 2.png", "retsu 4.png", "rin 2.png", "rock god.png", "wild god.png", "YOIICHI 2.png"],
    SPECIALE: ["fire phoenix.png", "GALAXY.png", "mother nature.png", "RIMACI 3.png", "YOIICHI 3.png"],
    UNIQUE: ["golden kage+.jpg", "golden maze.jpg", "golden retsu.png", "golden yoiichi.png"]
};

let stats = { balanceEuro: 0, coins: 500, clicks: 0, packsOpened: 0, e:0, l:0, s:0, u:0, lastDaily: 0 };
let collectionData = [];
let questData = {
    sbustatore: { progress: 0, target: 5, claimed: false },
    collezionista: { progress: 0, target: 2, claimed: false },
    schizzinoso: { progress: 0, target: 30, claimed: false }
};
let currentPackDuplicates = [];
let isAnimatingUnique = false;
let isOpeningPack = false;

function getCardValueCoins(rarity, img, isHolo) {
    if(img === 'bonus.png') return 500;
    if(rarity === 'COMUNE') return 1; 
    if(rarity === 'RARO') return 5;
    if(rarity === 'EPICO') return isHolo ? 50 : 15;
    if(rarity === 'LEGGENDARIO') return 100;
    if(rarity === 'SPECIALE') return 500;
    if(rarity === 'UNIQUE') return 1000; 
    return 0;
}

function getCardValueEuro(rarity, img, isHolo) {
    if(img === 'bonus.png') return 200.00;
    if(rarity === 'COMUNE') return 0.01; if(rarity === 'RARO') return 0.05;
    if(rarity === 'EPICO') return isHolo ? 1.50 : 0.20;
    if(rarity === 'LEGGENDARIO') return img.toUpperCase().includes('GOD') ? 12.00 : 7.00;
    if(rarity === 'SPECIALE') return (img.includes('fire phoenix') || img.includes('mother nature')) ? 40.00 : 25.00;
    if(rarity === 'UNIQUE') return img.includes('yoiichi') ? 100.00 : 70.00; return 0;
}

function updateStatsUI() {
    document.getElementById('h-coins').innerText = Math.floor(stats.coins);
    document.getElementById('s-wallet-euro').innerText = stats.balanceEuro.toFixed(2) + '€';
    document.getElementById('s-wallet-euro').style.color = stats.balanceEuro >= 0 ? '#2ecc71' : '#e74c3c';
    document.getElementById('s-clicks').innerText = stats.clicks;
    document.getElementById('s-packs').innerText = stats.packsOpened;
    let collVal = collectionData.reduce((t, c) => t + (getCardValueEuro(c.rarity, c.img, c.isHolo) * c.count), 0);
    document.getElementById('s-coll-val-euro').innerText = collVal.toFixed(2) + '€';
    document.getElementById('s-e').innerText = stats.e; 
    document.getElementById('s-l').innerText = stats.l;
    document.getElementById('s-s').innerText = stats.s; 
    document.getElementById('s-u').innerText = stats.u;
    if (stats.u > 0) document.getElementById('stat-unique-box').style.display = 'block';
    document.getElementById('btn-sell-all-dups').style.display = collectionData.some(c => c.count > 1) ? 'block' : 'none';
    updateQuestUI();
}

function saveGame() { localStorage.setItem('hataki_save_v9', JSON.stringify({ stats, collection: collectionData, quests: questData })); }

function checkDailyBonus() {
    const now = new Date();
    const last = new Date(stats.lastDaily || 0);
    if (now.toDateString() !== last.toDateString()) {
        stats.coins += 500;
        stats.lastDaily = now.getTime();
        showPopup('moneta.jpg', "BENVENUTO! Hai ricevuto il bonus giornaliero di 500 monete!", false, closePopup);
        saveGame();
        updateStatsUI();
    }
}

function loadGame() {
    const saved = localStorage.getItem('hataki_save_v9');
    if (saved) { const d = JSON.parse(saved); stats = d.stats; collectionData = d.collection; if(d.quests) questData = d.quests; }
    if(stats.balanceEuro === undefined) stats.balanceEuro = 0;
    if(stats.coins === undefined) stats.coins = 500;
    if(stats.clicks === undefined) stats.clicks = 0;
    updateStatsUI(); renderCollection();
    checkDailyBonus();
}

function scratchLogo(event) {
    stats.coins += 1;
    stats.clicks++;
    updateStatsUI();
    saveGame();
    const floatingText = document.createElement('div');
    floatingText.innerText = '+1';
    floatingText.className = 'floating-coin';
    floatingText.style.left = (event.clientX - 10) + 'px';
    floatingText.style.top = (event.clientY - 20) + 'px';
    document.body.appendChild(floatingText);
    setTimeout(() => floatingText.remove(), 1000);
}

function preOpen(img, mode, costCoins) {
    if(isOpeningPack) return; 
    if (stats.coins < costCoins) { showPopup(img, "Monete insufficienti!", false); return; }
    showPopup(img, `Acquistare per ${costCoins} monete?`, true, () => {
        stats.coins -= costCoins;
        let costEuro = 0;
        if(mode === 'base') costEuro = 3;
        else if(mode === 'super') costEuro = 5;
        else if(mode === 'legendary') costEuro = 8;
        else if(mode === 'random1') costEuro = 5;
        else if(mode === 'random2') costEuro = 0.5;
        stats.balanceEuro -= costEuro;
        stats.packsOpened++;
        if(!questData.sbustatore.claimed) questData.sbustatore.progress = Math.min(5, questData.sbustatore.progress + 1);
        closePopup(); initOpening(img, mode);
    });
}

function initOpening(img, mode) {
    isOpeningPack = true; 
    currentPackDuplicates = []; 
    document.getElementById('opening').className = 'page active';
    document.getElementById('opening').style.background = 'var(--bg-default)';
    document.getElementById('home').classList.remove('active');
    document.getElementById('home-btn').style.display = 'none';
    document.getElementById('stage').innerHTML = `<div class="pack-wrapper" onclick="openPack(this, '${mode}')"><div class="pack-part p-top" style="background-image:url('${img}')"></div><div class="pack-part p-bottom" style="background-image:url('${img}')"></div></div>`;
}

function openPack(el, mode) { 
    if(el.classList.contains('ripped')) return;
    el.classList.add('ripped'); 
    setTimeout(() => { el.style.display = 'none'; spawnCards(mode); }, 800); 
}

function updateBackground(rarity) {
    const op = document.getElementById('opening');
    op.classList.remove('bg-epic', 'bg-legendary', 'bg-unique', 'bg-special');
    if(rarity === 'EPICO') op.classList.add('bg-epic');
    if(rarity === 'LEGGENDARIO') op.classList.add('bg-legendary');
    if(rarity === 'UNIQUE') op.classList.add('bg-unique');
    if(rarity === 'SPECIALE') op.classList.add('bg-special');
}

function spawnCards(mode) {
    let rarities = [];
    let isGodPack = Math.random() < 0.0001; 
    if (isGodPack) {
        rarities = ['LEGGENDARIO', 'LEGGENDARIO', 'SPECIALE', 'SPECIALE', 'SPECIALE', 'SPECIALE', 'UNIQUE', 'UNIQUE'];
    } else if (mode === 'random1') {
        let r = Math.random() * 100;
        if (r < 0.5) rarities.push('UNIQUE');
        else if (r < 30.5) rarities.push('SPECIALE'); 
        else rarities.push('LEGGENDARIO'); 
    } else if (mode === 'random2') {
        let r = Math.random() * 100;
        if (r < 0.5) rarities.push('UNIQUE');
        else if (r < 5.0) rarities.push('SPECIALE');
        else if (r < 25.0) rarities.push('LEGGENDARIO'); 
        else if (r < 50.0) rarities.push('EPICO'); 
        else if (r < 75.0) rarities.push('RARO'); 
        else rarities.push('COMUNE'); 
    } else {
        if (mode === 'super') {
            for(let i=0; i<3; i++) rarities.push('COMUNE');
            for(let i=0; i<3; i++) rarities.push('RARO');
            rarities.push('EPICO'); 
            let upgradeRoll = Math.random();
            if (upgradeRoll < 0.05) rarities.push('SPECIALE');
            else if (upgradeRoll < 0.35) rarities.push('LEGGENDARIO'); 
            else rarities.push('EPICO');
        } else if (mode === 'base') {
            for(let i=0; i<5; i++) rarities.push('COMUNE');
            for(let i=0; i<2; i++) rarities.push('RARO');
            let thirdSlot = Math.random();
            if (thirdSlot < 0.005) rarities.push('SPECIALE');
            else if (thirdSlot < 0.055) rarities.push('LEGGENDARIO'); 
            else if (thirdSlot < 0.505) rarities.push('EPICO'); 
            else if (thirdSlot < 1.005) rarities.push('RARO'); 
        } else if (mode === 'legendary') {
            for(let i=0; i<2; i++) rarities.push('RARO');
            for(let i=0; i<5; i++) rarities.push('EPICO');
            rarities.push(Math.random() < 0.2 ? 'SPECIALE' : 'LEGGENDARIO');
        }
    }

    if (mode !== 'random1' && mode !== 'random2') {
        rarities = rarities.map(r => (r === 'COMUNE' && Math.random() < 0.001) ? 'BONUS' : r);
        if (!isGodPack && Math.random() < 0.001) rarities[7] = 'UNIQUE';
    }

    let activeCards = rarities.length;
    rarities.forEach((rarity, i) => {
        let imgName;
        let finalRarity = rarity;
        if (rarity === 'BONUS') { imgName = 'bonus.png'; finalRarity = 'COMUNE'; }
        else { let pool = DB[rarity]; imgName = pool[Math.floor(Math.random() * pool.length)]; }

        let isHolo = (finalRarity === 'UNIQUE' || finalRarity === 'SPECIALE' || finalRarity === 'LEGGENDARIO' || (finalRarity === 'EPICO' && Math.random() < 0.2));
        let effect = isHolo ? '<div class="holo"></div>' : '';
        if(finalRarity === 'UNIQUE') effect += '<div class="holo-unique"></div>';
        
        // Creazione contenitore particelle a puntini
        let particleHtml = '';
        if (finalRarity === 'UNIQUE' || finalRarity === 'SPECIALE') {
            particleHtml = '<div class="particle-container">';
            let pColor = finalRarity === 'UNIQUE' ? 'var(--unique)' : 'var(--special)';
            for(let p=0; p<40; p++){
                let dx = (Math.random() * 200 - 100) + 'vw';
                let dy = (Math.random() * 200 - 100) + 'vh';
                let delay = (Math.random() * 0.8) + 's';
                particleHtml += `<div class="dot-particle" style="background:${pColor}; --dx:${dx}; --dy:${dy}; animation-delay:${delay}; left:50%; top:50%;"></div>`;
            }
            particleHtml += '</div>';
        }

        let isNew = !collectionData.some(c => c.img === imgName && c.isHolo === isHolo);
        let newBadge = isNew ? '<div class="new-badge">NUOVO!</div>' : '';

        const card = document.createElement('div'); card.className = 'card';
        card.style.zIndex = 100 - i;
        card.innerHTML = `${particleHtml}<div class="card-inner"><div class="card-face card-back"></div><div class="card-front card-face rarity-${finalRarity}" style="background-image:url('${imgName}')">${effect}${newBadge}<div class="card-price-tag">${getCardValueEuro(finalRarity, imgName, isHolo).toFixed(2)}€</div></div></div>`;
        
        card.onclick = function() {
            if(isAnimatingUnique || this.classList.contains('removing') || this.classList.contains('rumbling')) return;
            if(!this.classList.contains('flipped')) {
                let rumbleTime = 0; let starType = '';
                if(finalRarity === 'UNIQUE' || imgName === 'bonus.png') { rumbleTime = 5000; starType = 'star-unique'; }
                else if(finalRarity === 'SPECIALE') { rumbleTime = 3000; starType = 'star-special'; }
                else if(finalRarity === 'LEGGENDARIO') { rumbleTime = 500; starType = 'star-legendary'; }
                
                if(rumbleTime > 0) {
                    isAnimatingUnique = true; this.classList.add('rumbling');
                    setTimeout(() => { if(starType) triggerStar(starType); }, rumbleTime - 400);
                    setTimeout(() => {
                        updateBackground(finalRarity);
                        this.classList.remove('rumbling'); 
                        this.classList.add('flipped'); 
                        isAnimatingUnique = false; 
                        const pCont = this.querySelector('.particle-container');
                        if(pCont) pCont.remove(); // Rimuove particelle a fine animazione
                        if(finalRarity === 'UNIQUE') stats.u++;
                        else if(finalRarity === 'LEGGENDARIO') stats.l++;
                        else if(finalRarity === 'SPECIALE') {
                            stats.s++;
                            if(!questData.collezionista.claimed) questData.collezionista.progress = Math.min(2, questData.collezionista.progress + 1);
                        }
                        updateStatsUI();
                    }, rumbleTime); 
                } else {
                    updateBackground(finalRarity);
                    this.classList.add('flipped');
                    if(finalRarity === 'EPICO') stats.e++;
                    updateStatsUI();
                }
                if(collectionData.some(c => c.img === imgName && c.isHolo === isHolo)) currentPackDuplicates.push({ rarity: finalRarity, img: imgName, isHolo });
            } else {
                this.style.pointerEvents = 'none';
                this.classList.add('removing'); this.style.opacity = '0';
                saveToCollection(finalRarity, imgName, effect, isHolo);
                setTimeout(() => { 
                    this.remove(); activeCards--;
                    if(activeCards === 0) showPackEnd();
                }, 300);
            }
        };
        document.getElementById('stage').appendChild(card);
        setTimeout(() => card.classList.add('show'), i * 150);
    });
}

function showPackEnd() {
    isOpeningPack = false; 
    document.getElementById('btn-sell-dups').disabled = false;
    if(currentPackDuplicates.length > 0) {
        const grid = document.getElementById('dup-grid'); grid.innerHTML = '';
        let totalValCoins = 0;
        currentPackDuplicates.forEach(d => {
            const img = document.createElement('img'); img.src = d.img; img.className = 'dup-img';
            grid.appendChild(img);
            totalValCoins += getCardValueCoins(d.rarity, d.img, d.isHolo);
        });
        document.getElementById('dup-net').innerText = totalValCoins;
        document.getElementById('dup-summary').style.display = 'block';
        document.getElementById('btn-sell-dups').onclick = () => {
            document.getElementById('btn-sell-dups').disabled = true;
            let totalEuroRecovered = 0;
            currentPackDuplicates.forEach(d => {
                const idx = collectionData.findIndex(c => c.img === d.img && c.isHolo === d.isHolo && c.count > 1);
                if(idx !== -1) {
                    collectionData[idx].count--;
                    totalEuroRecovered += getCardValueEuro(d.rarity, d.img, d.isHolo);
                }
            });
            stats.coins += totalValCoins; stats.balanceEuro += totalEuroRecovered;
            closeDupSummary(); renderCollection(); updateStatsUI(); saveGame();
        };
    } else {
        document.getElementById('home-btn').style.display = 'block';
    }
}

function saveToCollection(rarity, img, effect, isHolo) {
    const existing = collectionData.find(c => c.img === img && c.isHolo === isHolo);
    if(existing) existing.count++;
    else collectionData.push({ rarity, img, effect, isHolo, count: 1 });
    renderCollection(); updateStatsUI(); saveGame();
}

function renderCollection() {
    const grid = document.getElementById('collection-grid'); grid.innerHTML = '';
    const order = { 'UNIQUE': 0, 'SPECIALE': 1, 'LEGGENDARIO': 2, 'EPICO': 3, 'RARO': 4, 'COMUNE': 5 };
    let sorted = [...collectionData].sort((a, b) => {
        if (order[a.rarity] !== order[b.rarity]) return order[a.rarity] - order[b.rarity];
        if (a.isHolo !== b.isHolo) return b.isHolo - a.isHolo;
        return a.img.localeCompare(b.img);
    });
    sorted.forEach((c) => {
        const d = document.createElement('div'); d.className = `c-item rarity-${c.rarity}`;
        d.style.backgroundImage = `url('${c.img}')`; d.innerHTML = c.effect;
        if(c.count > 1) d.innerHTML += `<div class="dup-badge">x${c.count}</div>`;
        const originalIndex = collectionData.indexOf(c);
        d.onclick = () => openViewer(originalIndex); grid.appendChild(d);
    });
    document.getElementById('coll-count').innerText = collectionData.reduce((t,c) => t + c.count, 0);
}

function openViewer(i) {
    const c = collectionData[i]; const v = document.getElementById('viewer');
    const content = document.getElementById('viewer-content');
    content.className = `big-card rarity-${c.rarity}`; content.style.backgroundImage = `url('${c.img}')`;
    content.innerHTML = c.effect;
    const sBtn = document.getElementById('sell-action-btn');
    sBtn.style.display = 'inline-block';
    const valCoins = getCardValueCoins(c.rarity, c.img, c.isHolo);
    document.getElementById('sell-value').innerText = valCoins;
    sBtn.onclick = () => sellCard(i);
    v.style.display = 'flex';
}

function sellCard(i) {
    const c = collectionData[i];
    const valCoins = getCardValueCoins(c.rarity, c.img, c.isHolo);
    const valEuro = getCardValueEuro(c.rarity, c.img, c.isHolo);
    stats.coins += valCoins; stats.balanceEuro += valEuro;
    c.count--;
    if(c.count <= 0) collectionData.splice(i, 1);
    if(!questData.schizzinoso.claimed) questData.schizzinoso.progress = Math.min(30, questData.schizzinoso.progress + 1);
    closeViewer(); renderCollection(); updateStatsUI(); saveGame();
}

function sellAllDuplicates() {
    collectionData.forEach((c) => {
        if(c.count > 1 && c.rarity !== 'UNIQUE') {
            const copies = c.count - 1;
            const valCoins = getCardValueCoins(c.rarity, c.img, c.isHolo) * copies;
            const valEuro = getCardValueEuro(c.rarity, c.img, c.isHolo) * copies;
            stats.coins += valCoins; stats.balanceEuro += valEuro;
            c.count = 1;
            if(!questData.schizzinoso.claimed) questData.schizzinoso.progress = Math.min(30, questData.schizzinoso.progress + copies);
        }
    });
    renderCollection(); updateStatsUI(); saveGame();
}

function toggleQuests() { const p = document.getElementById('quest-panel'); p.style.display = (p.style.display === 'flex') ? 'none' : 'flex'; }
function updateQuestUI() {
    const list = document.getElementById('quest-list-ui'); list.innerHTML = '';
    const quests = [{ id: 'sbustatore', title: 'SBUSATORE', desc: 'Apri 5 pacchetti', target: 5, reward: 50 }, { id: 'collezionista', title: 'COLLEZIONISTA', desc: 'Trova 2 Speciali', target: 2, reward: 200 }, { id: 'schizzinoso', title: 'SCHIZZINOSO', desc: 'Vendi 30 carte', target: 30, reward: 100 }];
    quests.forEach(q => {
        const data = questData[q.id]; const isDone = data.progress >= q.target;
        const div = document.createElement('div');
        div.className = `quest-item ${isDone ? 'completed' : ''} ${data.claimed ? 'claimed' : ''}`;
        div.innerHTML = `<strong>${q.title}</strong> (${data.progress}/${q.target})<br><small>${q.desc}</small>`;
        if(isDone && !data.claimed) div.onclick = () => { stats.coins += q.reward; data.claimed = true; updateStatsUI(); saveGame(); };
        list.appendChild(div);
    });
}

function triggerStar(t) { const s = document.createElement('div'); s.className = `star-burst ${t}`; document.body.appendChild(s); setTimeout(() => s.remove(), 1600); }
function toggleStats() { const b = document.getElementById('main-stats-box'); b.style.display = (b.style.display === 'flex') ? 'none' : 'flex'; }
function closePopup() { document.getElementById('custom-popup').style.display = 'none'; }
function showPopup(img, text, showConfirm, action) {
    document.getElementById('custom-popup').style.display = 'flex';
    document.getElementById('pop-img').src = img || '';
    document.getElementById('pop-text').innerText = text;
    document.getElementById('pop-confirm').onclick = action;
    document.getElementById('pop-cancel').style.display = showConfirm ? 'block' : 'none';
}
function closeViewer() { document.getElementById('viewer').style.display = 'none'; }
function closeDupSummary() { document.getElementById('dup-summary').style.display = 'none'; document.getElementById('home-btn').style.display = 'block'; }
function goHome() { isOpeningPack = false; document.getElementById('opening').className = 'page'; document.getElementById('home').classList.add('active'); }
window.onload = loadGame;
</script>
</body>
</html>
