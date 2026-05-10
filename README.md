<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PIXEL KAIZEN: ULTIMATE</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; --green: #2eff7b; }
        body { margin: 0; background: #000; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        #game-container { position: relative; width: 800px; height: 400px; margin: 5px auto; border: 4px solid #333; overflow: hidden; background: #0a0a0a; }
        
        .hud { position: absolute; top: 10px; width: 100%; display: flex; justify-content: space-between; padding: 0 20px; box-sizing: border-box; z-index: 50; pointer-events: none;}
        .bar-bg { background: rgba(0,0,0,0.8); height: 14px; border: 2px solid #fff; width: 180px; }
        .hp-fill { background: var(--red); height: 100%; width: 100%; transition: 0.3s; }
        .ce-bg { background: #000; height: 5px; margin-top: 2px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; }

        .menu-screen { position: absolute; inset: 0; background: rgba(0,0,0,0.95); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .menu-btn { margin: 5px; padding: 10px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 220px; text-transform: uppercase; font-size: 12px; }
        .menu-btn:hover { background: #fff; color: #000; }
        
        .swatch-box { display: flex; gap: 8px; margin-bottom: 15px; }
        .swatch { width: 30px; height: 30px; border: 2px solid #444; border-radius: 50%; cursor: pointer; }
        .swatch.active { border-color: #fff; transform: scale(1.2); }

        .controls-layer { position: absolute; bottom: 5px; width: 800px; left: 50%; transform: translateX(-50%); display: flex; justify-content: space-between; pointer-events: none; }
        .ctrl-group { display: flex; gap: 5px; pointer-events: auto; padding: 10px; }
        .btn { width: 50px; height: 50px; background: rgba(50,50,50,0.8); border: 2px solid #666; color: white; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 10px; user-select: none; }
        .btn:active { background: #fff; color: #000; }
        
        #domain-flash { position: absolute; inset: 0; pointer-events: none; z-index: 100; mix-blend-mode: overlay; opacity: 0; transition: 0.5s; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="domain-flash"></div>
    
    <!-- Initial Mode Menu -->
    <div id="mode-menu" class="menu-screen">
        <h1 style="color:var(--blue)">PIXEL KAIZEN</h1>
        <button class="menu-btn" onclick="showDiff('1v1')">VS COMPUTER</button>
        <button class="menu-btn" onclick="setupGame('split')">SPLIT-SCREEN 1V1</button>
        <button class="menu-btn" onclick="showDiff('domain-only')">DOMAIN ONLY (CPU)</button>
        <button class="menu-btn" style="border-color:var(--purple)" onclick="toggleMenu('custom-menu', true)">CUSTOMIZE</button>
    </div>

    <!-- Difficulty Menu -->
    <div id="diff-menu" class="menu-screen" style="display:none">
        <h2>SELECT DIFFICULTY</h2>
        <button class="menu-btn" onclick="setupGame(tempMode, 'easy')">EASY</button>
        <button class="menu-btn" onclick="setupGame(tempMode, 'medium')">MEDIUM</button>
        <button class="menu-btn" onclick="setupGame(tempMode, 'hard')">NIGHTMARE</button>
    </div>

    <!-- Customization Menu -->
    <div id="custom-menu" class="menu-screen" style="display:none">
        <h3>PLAYER COLOR</h3>
        <div class="swatch-box" id="swatches"></div>
        <h3>EQUIPPED DOMAIN</h3>
        <button class="menu-btn" id="dom-btn" onclick="cycleDom()">TYPE: SLASHES</button>
        <button class="menu-btn" onclick="toggleMenu('custom-menu', false)">DONE</button>
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
        <div class="btn" id="p1-j">JMP</div><div class="btn" id="p1-a">HIT</div><div class="btn" id="p1-c">CE</div><div class="btn" id="p1-d">DE</div>
    </div>
    <div class="ctrl-group" id="p2-ctrls" style="display:none">
        <div class="btn" id="p2-d">DE</div><div class="btn" id="p2-c">CE</div><div class="btn" id="p2-a">HIT</div><div class="btn" id="p2-j">JMP</div>
        <div class="btn" id="p2-l">◀</div><div class="btn" id="p2-r">▶</div>
    </div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameActive = false, currentGameMode = '', diff = 'medium', tempMode = '';
let p1Color = '#00d4ff', p2Color = '#ff2e2e', selectedDomIdx = 0;
const domains = ['slashes', 'burst', 'whiteout', 'rain', 'clone'];
let p1, p2, clone = null, domainTimer = 0, lastP1Domain = 0;
let hitSparks = [], particles = [], shake = 0;

// Init Swatches
const colors = ['#00d4ff', '#ffffff', '#ffeb3b', '#e91e63', '#2eff7b', '#ff9800'];
colors.forEach(c => {
    const s = document.createElement('div');
    s.className = 'swatch' + (c === p1Color ? ' active' : '');
    s.style.background = c;
    s.onclick = () => { p1Color = c; document.querySelectorAll('.swatch').forEach(x => x.classList.remove('active')); s.classList.add('active'); };
    document.getElementById('swatches').appendChild(s);
});

function toggleMenu(id, show) { document.getElementById(id).style.display = show ? 'flex' : 'none'; }
function showDiff(mode) { tempMode = mode; toggleMenu('mode-menu', false); toggleMenu('diff-menu', true); }
function cycleDom() { selectedDomIdx = (selectedDomIdx + 1) % domains.length; document.getElementById('dom-btn').innerText = "TYPE: " + domains[selectedDomIdx].toUpperCase(); }

class Fighter {
    constructor(x, color, isBot = false, id = 1) {
        this.id = id; this.isBot = isBot; this.color = color;
        this.pos = { x, y: 280 }; this.vel = { x: 0, y: 0 };
        this.hp = 100; this.ce = 0; this.dir = x > 400 ? -1 : 1;
        this.isAtk = false; this.atkF = 0; this.isCharge = false; this.jumps = 0;
    }
    update() {
        this.pos.x += this.vel.x; this.pos.y += this.vel.y;
        if (this.pos.y > 280) { this.pos.y = 280; this.vel.y = 0; this.jumps = 0; } else { this.vel.y += 0.8; }
        if (this.isCharge) this.ce = Math.min(100, this.ce + 0.25);
        if (this.isAtk) { this.atkF += 0.15; if(this.atkF >= 1) { this.isAtk = false; this.atkF = 0; } }
    }
    draw(c) {
        c.save(); c.translate(this.pos.x + 20, this.pos.y + 45);
        if (this.isCharge) { c.shadowBlur = 20; c.shadowColor = this.color; }
        c.scale(this.dir, 1); c.strokeStyle = this.color; c.lineWidth = 4; c.lineCap = 'round';
        let ext = this.isAtk ? Math.sin(this.atkF * Math.PI) * 30 : 0;
        let walk = (Math.abs(this.vel.x) > 0.1) ? Math.sin(Date.now()*0.01) * 15 : 0;
        c.beginPath(); c.arc(0, -35, 10, 0, Math.PI*2); c.stroke(); // Head
        c.beginPath(); c.moveTo(0, -25); c.lineTo(0, 10); c.stroke(); // Spine
        c.beginPath(); c.moveTo(0, -15); c.lineTo(10+ext, 5); c.stroke(); // Arm
        c.beginPath(); c.moveTo(0, 10); c.lineTo(10+walk, 35); c.stroke(); // Leg
        c.restore();
    }
}

function setupGame(mode, d) {
    currentGameMode = mode; diff = d || 'medium';
    p1 = new Fighter(150, p1Color, false, 1);
    p2 = new Fighter(650, p2Color, mode !== 'split', 2);
    if(mode === 'split') document.getElementById('p2-ctrls').style.display = 'flex';
    toggleMenu('mode-menu', false); toggleMenu('diff-menu', false);
    gameActive = true; loop();
}

function useDomain(caster, target) {
    if (caster.ce < 100 || domainTimer > 0) return;
    caster.ce = 0; domainTimer = 500; // Total effect time
    const type = caster.isBot ? domains[Math.floor(Math.random()*domains.length)] : domains[selectedDomIdx];
    const flash = document.getElementById('domain-flash');
    flash.style.opacity = '1'; flash.style.background = caster.color;
    setTimeout(() => flash.style.opacity = '0', 500);

    if (type === 'clone') {
        clone = new Fighter(caster.pos.x, caster.color + "88", false, 3);
        clone.hp = 50; clone.lifetime = 1200; // 20 seconds at 60fps
    } else if (type === 'whiteout') {
        target.hp -= 40; shake = 30;
    } else if (type === 'slashes') {
        let h = 0; let itv = setInterval(() => {
            target.hp -= 2; createHit(target.pos.x+20, target.pos.y+40);
            if(h++ > 20) clearInterval(itv);
        }, 100);
    } else if (type === 'rain') {
        for(let i=0; i<60; i++) particles.push({x: Math.random()*800, y: -Math.random()*400, target});
    }
}

function createHit(x, y) { hitSparks.push({x, y, life: 10}); }

function loop() {
    if (!gameActive) return;
    ctx.setTransform(1,0,0,1,0,0); ctx.clearRect(0, 0, 800, 400);

    // Camera
    let dist = Math.abs(p1.pos.x - p2.pos.x);
    let scale = Math.max(0.6, Math.min(1.2, 800/(dist+200)));
    ctx.translate(400, 200); ctx.scale(scale, scale);
    ctx.translate(-(p1.pos.x+p2.pos.x)/2 - 20, -250);
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }

    // Logic
    updateFighter(p1); updateFighter(p2);
    if(p2.isBot) updateAI(p2, p1);
    if(clone) {
        updateAI(clone, p2); updateFighter(clone); clone.draw(ctx);
        clone.lifetime--; if(clone.lifetime <= 0 || clone.hp <= 0) clone = null;
    }

    // Punches
    checkHit(p1, p2); checkHit(p2, p1);
    if(clone) checkHit(clone, p2);

    // Rendering
    p1.draw(ctx); p2.draw(ctx);
    particles.forEach((p, i) => {
        p.y += 10; ctx.fillStyle = 'white'; ctx.fillRect(p.x, p.y, 2, 10);
        if(Math.abs(p.x - p.target.pos.x) < 40 && Math.abs(p.y - p.target.pos.y) < 50) { p.target.hp -= 0.5; particles.splice(i,1); }
    });
    hitSparks.forEach((s, i) => {
        ctx.strokeStyle = '#fff'; ctx.beginPath(); ctx.moveTo(s.x-5, s.y-5); ctx.lineTo(s.x+5, s.y+5); ctx.stroke();
        s.life--; if(s.life <= 0) hitSparks.splice(i, 1);
    });

    // UI
    document.getElementById('p1-hp').style.width = p1.hp + "%";
    document.getElementById('p2-hp').style.width = p2.hp + "%";
    document.getElementById('p1-ce').style.width = p1.ce + "%";
    document.getElementById('p2-ce').style.width = p2.ce + "%";
    if(domainTimer > 0) domainTimer--;
    
    requestAnimationFrame(loop);
}

function updateFighter(f) {
    if(f.id === 1) {
        f.vel.x = keys.p1l ? -5 : (keys.p1r ? 5 : 0);
    } else if(f.id === 2 && currentGameMode === 'split') {
        f.vel.x = keys.p2l ? -5 : (keys.p2r ? 5 : 0);
    }
    f.update();
}

function updateAI(bot, target) {
    let d = target.pos.x - bot.pos.x;
    let speed = diff === 'easy' ? 2 : (diff === 'medium' ? 3.5 : 5);
    bot.vel.x = Math.abs(d) > 60 ? (d > 0 ? speed : -speed) : 0;
    bot.dir = d > 0 ? 1 : -1;
    if(Math.random() < 0.02) bot.isAtk = true;
    if(bot.ce >= 100 && Math.random() < 0.01) useDomain(bot, target);
}

function checkHit(a, v) {
    if(a.isAtk && a.atkF > 0.4 && a.atkF < 0.7) {
        let hitX = a.pos.x + (a.dir * 40);
        if(Math.abs(hitX - v.pos.x) < 40 && Math.abs(a.pos.y - v.pos.y) < 50) {
            if(currentGameMode !== 'domain-only') v.hp -= (a.id === 3 ? 0.5 : 1.5);
            createHit(v.pos.x+20, v.pos.y+40);
        }
    }
}

const keys = {};
const bind = (id, k) => {
    const el = document.getElementById(id);
    el.ontouchstart = (e) => { e.preventDefault(); 
        if(id.includes('j')) { if(k === 'p1') { if(p1.jumps < 2) { p1.vel.y = -15; p1.jumps++; } } else { if(p2.jumps < 2) { p2.vel.y = -15; p2.jumps++; } } }
        else if(id.includes('a')) { if(k === 'p1') p1.isAtk = true; else p2.isAtk = true; }
        else if(id.includes('c')) { if(k === 'p1') p1.isCharge = true; else p2.isCharge = true; }
        else if(id.includes('d')) { if(k === 'p1') useDomain(p1, p2); else useDomain(p2, p1); }
        else keys[k] = true;
    };
    el.ontouchend = () => keys[k] = false;
};

bind('p1-l', 'p1l'); bind('p1-r', 'p1r'); bind('p1-j', 'p1'); bind('p1-a', 'p1'); bind('p1-c', 'p1'); bind('p1-d', 'p1');
bind('p2-l', 'p2l'); bind('p2-r', 'p2r'); bind('p2-j', 'p2'); bind('p2-a', 'p2'); bind('p2-c', 'p2'); bind('p2-d', 'p2');
</script>
</body>
</html>
