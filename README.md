<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PIXEL KAIZEN 6.1 | CINEMATIC</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; --green: #2eff7b; }
        body { margin: 0; background: #000; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        #game-container { position: relative; width: 800px; height: 400px; margin: 5px auto; border: 4px solid #333; overflow: hidden; background: #0a0a0a; transition: border 0.3s; }
        
        .hud { position: absolute; top: 10px; width: 100%; display: flex; justify-content: space-between; padding: 0 20px; box-sizing: border-box; z-index: 50; pointer-events: none;}
        .bar-bg { background: rgba(0,0,0,0.8); height: 16px; border: 2px solid #fff; width: 220px; }
        .hp-fill { background: var(--red); height: 100%; width: 100%; transition: width 0.2s; }
        .ce-bg { background: #000; height: 6px; margin-top: 2px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; }

        .menu-screen { position: absolute; inset: 0; background: rgba(0,0,0,0.95); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .menu-btn { margin: 4px; padding: 10px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 220px; text-transform: uppercase; font-size: 11px; }
        .menu-btn:hover { background: #fff; color: #000; }
        
        .custom-row { display: flex; gap: 30px; margin-bottom: 10px; }
        .custom-col { display: flex; flex-direction: column; align-items: center; width: 250px; border: 1px solid #444; padding: 10px; border-radius: 10px; }
        .swatch-box { display: flex; gap: 6px; margin: 8px 0; }
        .swatch { width: 25px; height: 25px; border: 2px solid #444; border-radius: 50%; cursor: pointer; }
        .swatch.active { border-color: #fff; transform: scale(1.2); }

        /* Domain Text UI */
        #domain-announcement { position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 150; pointer-events: none; opacity: 0; transform: scale(0.5); transition: 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        #domain-announcement.active { opacity: 1; transform: scale(1.1); }
        .de-subtext { font-size: 14px; color: #fff; letter-spacing: 4px; text-shadow: 0 0 10px #000; }
        .de-maintext { font-size: 40px; color: var(--purple); text-shadow: 0 0 20px #fff; -webkit-text-stroke: 1px white; }

        .controls-layer { position: absolute; bottom: 5px; width: 800px; left: 50%; transform: translateX(-50%); display: flex; justify-content: space-between; pointer-events: none; }
        .ctrl-group { display: flex; gap: 6px; pointer-events: auto; padding: 10px; }
        .btn { width: 68px; height: 68px; background: rgba(40,40,40,0.9); border: 2px solid #777; color: white; border-radius: 50%; display: flex; flex-direction: column; align-items: center; justify-content: center; font-size: 10px; user-select: none; }
        .btn:active { background: #fff; color: #000; border-color: #fff; }
        .cd-txt { font-size: 8px; color: var(--red); font-weight: bold; }
        
        #domain-flash { position: absolute; inset: 0; pointer-events: none; z-index: 100; mix-blend-mode: overlay; opacity: 0; transition: 0.4s; }
        #stage-ui { position: absolute; top: 50px; left: 50%; transform: translateX(-50%); color: var(--green); font-size: 18px; z-index: 40; display: none; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="domain-flash"></div>
    <div id="domain-announcement">
        <div class="de-subtext">DOMAIN EXPANSION</div>
        <div id="de-name" class="de-maintext">VOID</div>
    </div>
    <div id="stage-ui">STAGE 1</div>
    
    <div id="mode-menu" class="menu-screen">
        <h1 style="color:var(--blue); letter-spacing: 4px;">KAIZEN 6.1</h1>
        <button class="menu-btn" onclick="showDiff('1v1')">VS COMPUTER</button>
        <button class="menu-btn" onclick="setupGame('survival', 'medium')">SURVIVAL GAUNTLET</button>
        <button class="menu-btn" onclick="setupGame('split')">SPLIT-SCREEN 1V1</button>
        <button class="menu-btn" style="border-color:var(--purple)" onclick="toggleMenu('custom-menu', true)">CUSTOMIZE FIGHTERS</button>
    </div>

    <div id="diff-menu" class="menu-screen" style="display:none">
        <h2>SELECT DIFFICULTY</h2>
        <button class="menu-btn" onclick="setupGame(tempMode, 'easy')">EASY</button>
        <button class="menu-btn" onclick="setupGame(tempMode, 'medium')">NORMAL</button>
        <button class="menu-btn" onclick="setupGame(tempMode, 'hard')">NIGHTMARE</button>
    </div>

    <div id="custom-menu" class="menu-screen" style="display:none">
        <div class="custom-row">
            <div class="custom-col">
                <h3 style="color:var(--blue)">PLAYER 1</h3>
                <div class="swatch-box" id="swatches-p1"></div>
                <button class="menu-btn" id="shape-p1" onclick="toggleShape(1)">SHAPE: STICKMAN</button>
                <button class="menu-btn" id="dom-p1" onclick="cycleDom(1)">DE: SLASHES</button>
            </div>
            <div class="custom-col">
                <h3 style="color:var(--red)">PLAYER 2</h3>
                <div class="swatch-box" id="swatches-p2"></div>
                <button class="menu-btn" id="shape-p2" onclick="toggleShape(2)">SHAPE: STICKMAN</button>
                <button class="menu-btn" id="dom-p2" onclick="cycleDom(2)">DE: SLASHES</button>
            </div>
        </div>
        <button class="menu-btn" style="width: 530px;" onclick="toggleMenu('custom-menu', false)">SAVE SETTINGS</button>
    </div>

    <div id="ko-screen" class="menu-screen" style="display:none; background:rgba(255,0,0,0.3)">
        <h1 id="ko-text" style="font-size:60px">K.O.</h1>
        <button class="menu-btn" onclick="location.reload()">MAIN MENU</button>
    </div>

    <div class="hud">
        <div id="p1-ui">
            <div class="bar-bg"><div id="p1-hp" class="hp-fill"></div></div>
            <div class="ce-bg"><div id="p1-ce" class="ce-fill"></div></div>
        </div>
        <div id="p2-ui" style="text-align:right">
            <div class="bar-bg" style="margin-left:auto"><div id="p2-hp" class="hp-fill"></div></div>
            <div class="ce-bg" style="margin-left:auto"><div id="p2-ce" class="ce-fill"></div></div>
        </div>
    </div>

    <canvas id="gameCanvas" width="800" height="400"></canvas>
</div>

<div class="controls-layer">
    <div class="ctrl-group" id="p1-ctrls">
        <div class="btn" id="p1-l">◀</div><div class="btn" id="p1-r">▶</div>
        <div class="btn" id="p1-j">JUMP</div><div class="btn" id="p1-a">HIT</div><div class="btn" id="p1-c">AURA</div>
        <div class="btn" id="p1-d">DE <span id="p1-cd" class="cd-txt"></span></div>
    </div>
    <div class="ctrl-group" id="p2-ctrls" style="display:none">
        <div class="btn" id="p2-d">DE <span id="p2-cd" class="cd-txt"></span></div><div class="btn" id="p2-c">AURA</div><div class="btn" id="p2-a">HIT</div><div class="btn" id="p2-j">JUMP</div>
        <div class="btn" id="p2-l">◀</div><div class="btn" id="p2-r">▶</div>
    </div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameActive = false, currentGameMode = '', diff = 'medium', tempMode = '', gameOver = false;
let p1Config = { color: '#00d4ff', shape: 'stickman', domain: 'slashes', lastDE: 0 };
let p2Config = { color: '#ff2e2e', shape: 'stickman', domain: 'slashes', lastDE: 0 };
const domains = ['slashes', 'burst', 'whiteout', 'rain', 'clone'];
let p1, p2, clone = null, stage = 1, shake = 0, cooldown = 15000;
let hitSparks = [], particles = [];

// Init UI Swatches
const colors = ['#00d4ff', '#ffffff', '#ffeb3b', '#e91e63', '#2eff7b', '#ff9800'];
function buildSwatches(containerId, playerNum) {
    const box = document.getElementById(containerId);
    colors.forEach(c => {
        const s = document.createElement('div');
        s.className = 'swatch' + (c === (playerNum === 1 ? p1Config.color : p2Config.color) ? ' active' : '');
        s.style.background = c;
        s.onclick = () => {
            if(playerNum === 1) p1Config.color = c; else p2Config.color = c;
            box.querySelectorAll('.swatch').forEach(x => x.classList.remove('active'));
            s.classList.add('active');
        };
        box.appendChild(s);
    });
}
buildSwatches('swatches-p1', 1); buildSwatches('swatches-p2', 2);

function toggleMenu(id, show) { document.getElementById(id).style.display = show ? 'flex' : 'none'; }
function showDiff(mode) { tempMode = mode; toggleMenu('mode-menu', false); toggleMenu('diff-menu', true); }
function toggleShape(p) {
    let cfg = p === 1 ? p1Config : p2Config;
    cfg.shape = cfg.shape === 'stickman' ? 'box' : 'stickman';
    document.getElementById('shape-p'+p).innerText = "SHAPE: " + cfg.shape.toUpperCase();
}
function cycleDom(p) {
    let cfg = p === 1 ? p1Config : p2Config;
    let idx = (domains.indexOf(cfg.domain) + 1) % domains.length;
    cfg.domain = domains[idx];
    document.getElementById('dom-p'+p).innerText = "DE: " + cfg.domain.toUpperCase();
}

class Fighter {
    constructor(x, config, isBot = false, id = 1) {
        this.id = id; this.isBot = isBot; this.config = config;
        this.reset(x);
    }
    reset(x) {
        this.pos = { x, y: 280 }; this.vel = { x: 0, y: 0 };
        this.maxHp = (this.isBot && currentGameMode === 'survival') ? 100 + (stage*15) : 100;
        this.hp = this.maxHp; this.ce = 0; this.dir = x > 400 ? -1 : 1;
        this.isAtk = false; this.atkF = 0; this.isCharge = false; this.jumps = 0;
    }
    update() {
        this.pos.x += this.vel.x; this.pos.y += this.vel.y;
        if (this.pos.y > 280) { this.pos.y = 280; this.vel.y = 0; this.jumps = 0; } else { this.vel.y += 0.8; }
        if (this.isCharge) this.ce = Math.min(100, this.ce + 0.35);
        if (this.isAtk) { this.atkF += 0.18; if(this.atkF >= 1) { this.isAtk = false; this.atkF = 0; } }
    }
    draw(c) {
        c.save(); c.translate(this.pos.x + 20, this.pos.y + 45);
        if (this.isCharge) { c.shadowBlur = 25; c.shadowColor = this.config.color; }
        c.scale(this.dir, 1); c.strokeStyle = this.config.color; c.fillStyle = this.config.color; c.lineWidth = 5; c.lineCap = 'round';
        
        let ext = this.isAtk ? Math.sin(this.atkF * Math.PI) * 35 : 0;
        let walk = (Math.abs(this.vel.x) > 0.1) ? Math.sin(Date.now()*0.015) * 15 : 0;
        let tuck = (this.pos.y < 280) ? 15 : 0;

        if (this.config.shape === 'box') {
            c.fillRect(-15, -40, 30, 75);
            c.strokeStyle = '#fff'; c.lineWidth = 2; c.strokeRect(-15, -40, 30, 75);
            if(this.isAtk) c.fillRect(15, -15, 20 + ext, 12);
        } else {
            c.beginPath(); c.arc(0, -45, 12, 0, Math.PI*2); c.stroke(); // Head
            c.beginPath(); c.moveTo(0, -33); c.lineTo(0, 10); c.stroke(); // Spine
            c.beginPath(); c.moveTo(0, -25); c.lineTo(15 + ext, -10); c.stroke();
            c.beginPath(); c.moveTo(0, -25); c.lineTo(-15, 0); c.stroke();
            c.beginPath(); c.moveTo(0, 10); c.lineTo(15 + walk, 40 - tuck); c.stroke();
            c.beginPath(); c.moveTo(0, 10); ctx.lineTo(-15 - walk, 40 - tuck); c.stroke();
        }
        c.restore();
    }
}

function setupGame(mode, d) {
    currentGameMode = mode; diff = d || 'medium'; gameOver = false;
    p1 = new Fighter(150, p1Config, false, 1);
    p2 = new Fighter(650, p2Config, mode !== 'split', 2);
    if(mode === 'split') document.getElementById('p2-ctrls').style.display = 'flex';
    if(mode === 'survival') { stage = 1; document.getElementById('stage-ui').style.display = 'block'; }
    toggleMenu('mode-menu', false); toggleMenu('diff-menu', false);
    gameActive = true; loop();
}

function useDomain(caster, target) {
    let now = Date.now();
    if (caster.ce < 100 || now - caster.config.lastDE < cooldown) return;
    
    caster.config.lastDE = now; caster.ce = 0;
    const type = caster.config.domain;
    
    // Announcement UI
    const announce = document.getElementById('domain-announcement');
    const nameTxt = document.getElementById('de-name');
    nameTxt.innerText = type.toUpperCase();
    nameTxt.style.color = caster.config.color;
    announce.classList.add('active');
    
    const flash = document.getElementById('domain-flash');
    flash.style.opacity = '1'; flash.style.background = caster.config.color;
    document.getElementById('game-container').style.borderColor = caster.config.color;
    
    setTimeout(() => { 
        announce.classList.remove('active');
        flash.style.opacity = '0'; 
        document.getElementById('game-container').style.borderColor = '#333'; 
    }, 1500);

    if (type === 'clone') {
        clone = new Fighter(caster.pos.x, { ...caster.config, color: caster.config.color+'88' }, false, 3);
        clone.hp = 50; clone.lifetime = 1200; 
    } else if (type === 'whiteout') {
        target.hp -= 45; shake = 40;
    } else if (type === 'slashes') {
        let h = 0; let itv = setInterval(() => {
            target.hp -= 2; hitSparks.push({x: target.pos.x+20, y: target.pos.y+40, life: 8});
            if(h++ > 25) clearInterval(itv);
        }, 100);
    } else if (type === 'rain') {
        for(let i=0; i<60; i++) particles.push({x: Math.random()*800, y: -Math.random()*600, target});
    } else if (type === 'burst') {
        target.hp -= 50; shake = 50;
        particles.push({x: target.pos.x, y: 0, type: 'burst'}); setTimeout(() => particles = [], 500);
    }
}

function checkHit(a, v) {
    if(a.isAtk && a.atkF > 0.4 && a.atkF < 0.75) {
        let hitX = a.pos.x + (a.dir * 45);
        if(Math.abs(hitX - v.pos.x) < 45 && Math.abs(a.pos.y - v.pos.y) < 65) {
            v.hp -= 2; shake = 6;
            hitSparks.push({x: v.pos.x+20, y: v.pos.y+40, life: 10});
        }
    }
}

function loop() {
    if (!gameActive || gameOver) return;
    ctx.setTransform(1,0,0,1,0,0); ctx.clearRect(0, 0, 800, 400);

    let dist = Math.abs(p1.pos.x - p2.pos.x);
    let scale = Math.max(0.65, Math.min(1.15, 800/(dist+280)));
    ctx.translate(400, 200); ctx.scale(scale, scale);
    ctx.translate(-(p1.pos.x+p2.pos.x)/2 - 20, -260);
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }

    p1.vel.x = keys.p1l ? -6 : (keys.p1r ? 6 : 0);
    if(currentGameMode === 'split') p2.vel.x = keys.p2l ? -6 : (keys.p2r ? 6 : 0);
    else {
        let d = p1.pos.x - p2.pos.x;
        let speed = diff === 'easy' ? 2.5 : (diff === 'medium' ? 4.5 : 6.5);
        p2.vel.x = Math.abs(d) > 75 ? (d > 0 ? speed : -speed) : 0;
        p2.dir = d > 0 ? 1 : -1;
        if(Math.random() < 0.03) p2.isAtk = true;
        if(p2.ce >= 100 && Math.random() < 0.01) useDomain(p2, p1);
    }
    
    if(clone) {
        let d = p2.pos.x - clone.pos.x;
        clone.vel.x = d > 0 ? 3 : -3; clone.dir = d > 0 ? 1 : -1;
        if(Math.abs(d) < 60 && Math.random() < 0.05) clone.isAtk = true;
        checkHit(clone, p2);
        clone.update(); clone.draw(ctx);
        clone.lifetime--; if(clone.lifetime <= 0 || clone.hp <= 0) clone = null;
    }

    p1.update(); p2.update();
    checkHit(p1, p2); checkHit(p2, p1);

    p1.draw(ctx); p2.draw(ctx);
    particles.forEach((p, i) => {
        if(p.type === 'burst') { ctx.fillStyle = 'white'; ctx.fillRect(p.x-30, 0, 100, 400); }
        else {
            p.y += 15; ctx.fillStyle = '#fff'; ctx.fillRect(p.x, p.y, 3, 15);
            if(Math.abs(p.x - p.target.pos.x) < 40 && Math.abs(p.y - p.target.pos.y) < 60) { p.target.hp -= 0.6; particles.splice(i,1); }
        }
    });
    hitSparks.forEach((s, i) => {
        ctx.strokeStyle = '#fff'; ctx.beginPath(); ctx.moveTo(s.x-10, s.y-10); ctx.lineTo(s.x+10, s.y+10); ctx.stroke();
        s.life--; if(s.life <= 0) hitSparks.splice(i, 1);
    });

    if (p1.hp <= 0) { gameOver = true; toggleMenu('ko-screen', true); }
    if (p2.hp <= 0) {
        if(currentGameMode === 'survival') {
            stage++; p1.hp = Math.min(100, p1.hp + 45);
            p2.reset(650); document.getElementById('stage-ui').innerText = "STAGE " + stage;
        } else { gameOver = true; toggleMenu('ko-screen', true); }
    }

    document.getElementById('p1-hp').style.width = p1.hp + "%";
    document.getElementById('p2-hp').style.width = (p2.hp / (p2.isBot ? (100 + stage*15)/100 : 1)) + "%";
    document.getElementById('p1-ce').style.width = p1.ce + "%";
    document.getElementById('p2-ce').style.width = p2.ce + "%";
    
    let now = Date.now();
    let p1cd = Math.max(0, Math.ceil((cooldown - (now - p1Config.lastDE))/1000));
    let p2cd = Math.max(0, Math.ceil((cooldown - (now - p2Config.lastDE))/1000));
    document.getElementById('p1-cd').innerText = p1cd > 0 ? p1cd + "S" : "";
    document.getElementById('p2-cd').innerText = p2cd > 0 ? p2cd + "S" : "";
    
    requestAnimationFrame(loop);
}

const keys = {};
const bind = (id, k) => {
    const el = document.getElementById(id);
    el.ontouchstart = (e) => { e.preventDefault(); 
        if(id.includes('j')) { let t = k === 'p1' ? p1 : p2; if(t.jumps < 2) { t.vel.y = -17; t.jumps++; } }
        else if(id.includes('a')) { if(k === 'p1') p1.isAtk = true; else p2.isAtk = true; }
        else if(id.includes('c')) { if(k === 'p1') p1.isCharge = true; else p2.isCharge = true; }
        else if(id.includes('d')) { if(k === 'p1') useDomain(p1, p2); else useDomain(p2, p1); }
        else keys[k] = true;
    };
    
    // FIXED: Release logic to stop Aura charging
    const release = (e) => {
        if(e) e.preventDefault();
        if(id.includes('c')) { if(k === 'p1') p1.isCharge = false; else p2.isCharge = false; }
        else keys[k] = false;
    };
    el.ontouchend = el.onmouseleave = release;
};

bind('p1-l', 'p1l'); bind('p1-r', 'p1r'); bind('p1-j', 'p1'); bind('p1-a', 'p1'); bind('p1-c', 'p1'); bind('p1-d', 'p1');
bind('p2-l', 'p2l'); bind('p2-r', 'p2r'); bind('p2-j', 'p2'); bind('p2-a', 'p2'); bind('p2-c', 'p2'); bind('p2-d', 'p2');
</script>
</body>
</html>
