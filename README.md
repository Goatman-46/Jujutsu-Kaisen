<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>STICKMAN KAIZEN | DYNAMIC SPRITES</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; --green: #2eff7b; }
        body { margin: 0; background: #050505; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        #game-container { position: relative; width: 800px; height: 400px; margin: 10px auto; outline: 4px solid #222; overflow: hidden; background: #111; }
        .domain-active-bg { background: radial-gradient(circle, #1a0026 0%, #000 100%) !important; outline-color: var(--purple) !important; }
        #impact-flash { position: absolute; inset: 0; background: white; opacity: 0; z-index: 101; pointer-events: none; }
        #domain-ui { position: absolute; inset: 0; opacity: 0; display: flex; align-items: center; justify-content: center; z-index: 100; pointer-events: none; transition: 0.3s; }
        #domain-ui.show { opacity: 1; transform: scale(1.1); }
        .hud { position: absolute; top: 15px; width: 100%; display: flex; justify-content: space-between; padding: 0 30px; box-sizing: border-box; z-index: 10; }
        .bar-bg { background: #000; height: 20px; border: 2px solid #fff; width: 300px; }
        .hp-fill { background: linear-gradient(90deg, var(--red), #ff7b7b); height: 100%; width: 100%; transition: 0.1s; }
        .ce-bg { background: #000; height: 6px; margin-top: 4px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; box-shadow: 0 0 8px var(--blue); }
        #menu, #mode-menu, #custom-menu { position: absolute; inset: 0; background: rgba(0,0,0,0.95); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .menu-btn { margin: 8px; padding: 12px 30px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 280px; }
        .menu-btn:hover { background: white; color: black; }
        .swatch-container { display: flex; gap: 10px; margin: 15px 0; }
        .swatch { width: 35px; height: 35px; border: 3px solid #444; cursor: pointer; border-radius: 50%; }
        .swatch.active { border-color: white; transform: scale(1.2); }
        .mobile-btns { display: flex; justify-content: space-between; width: 800px; margin: 5px auto; max-width: 95vw; }
        .btn { width: 65px; height: 65px; background: #222; border: 2px solid #444; color: white; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="mode-menu">
        <h1 style="color: var(--blue); letter-spacing: 5px;">STICKMAN KAIZEN</h1>
        <button class="menu-btn" onclick="selectMode('1v1')">1v1 BATTLE</button>
        <button class="menu-btn" onclick="selectMode('survival')">SURVIVAL MODE</button>
        <button class="menu-btn" style="border-color: var(--purple)" onclick="openCustom()">CUSTOMIZE STICKMAN</button>
    </div>

    <div id="custom-menu" style="display:none">
        <h2>SKIN COLOR</h2>
        <div class="swatch-container" id="swatch-box"></div>
        <h2>ULTIMATE TYPE</h2>
        <button class="menu-btn" id="dom-toggle" onclick="toggleDomainType()">TYPE: SLASHES</button>
        <button class="menu-btn" onclick="closeCustom()">READY</button>
    </div>

    <div id="menu" style="display:none">
        <h2>SELECT DIFFICULTY</h2>
        <button class="menu-btn" onclick="startGame('easy')">EASY</button>
        <button class="menu-btn" onclick="startGame('medium')">NORMAL</button>
        <button class="menu-btn" onclick="startGame('hard')">NIGHTMARE</button>
    </div>

    <div id="impact-flash"></div>
    <div id="domain-ui"><h1>DOMAIN EXPANSION</h1></div>
    
    <div id="ko-screen" style="display:none; position:absolute; inset:0; background:rgba(0,0,0,0.8); z-index:200; flex-direction:column; align-items:center; justify-content:center;">
        <h1 style="font-size:60px; color:var(--red)">K.O.</h1>
        <button class="menu-btn" onclick="location.reload()">RETRY</button>
    </div>

    <div class="hud">
        <div><div class="bar-bg"><div id="p1-hp" class="hp-fill"></div></div><div class="ce-bg"><div id="p1-ce" class="ce-fill"></div></div></div>
        <div style="text-align: right;"><div class="bar-bg"><div id="p2-hp" class="hp-fill"></div></div><div class="ce-bg"><div id="p2-ce" class="ce-fill"></div></div></div>
    </div>
    <canvas id="gameCanvas" width="800" height="400"></canvas>
</div>

<div class="mobile-btns">
    <div style="display:flex; gap:10px;"><div class="btn" id="btn-left">◀</div><div class="btn" id="btn-right">▶</div><div class="btn" id="btn-charge">CE</div></div>
    <div style="display:flex; gap:10px;"><div class="btn" id="btn-up">UP</div><div class="btn" id="btn-atk">HIT</div><div class="btn" id="btn-dom" style="color:var(--purple)">DE</div></div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameOver = false, shake = 0, gameActive = false, domainInEffect = false;
let p1Color = '#ffffff', selectedDomain = 'slashes', bursts = [];
let p1LastDomain = 0, domainCooldown = 10000;

const colors = ['#ffffff', '#00f2ff', '#ffeb3b', '#e91e63', '#2eff7b', '#ff9800'];
const swatchBox = document.getElementById('swatch-box');
colors.forEach(c => {
    const s = document.createElement('div');
    s.className = 'swatch' + (c === p1Color ? ' active' : '');
    s.style.backgroundColor = c;
    s.onclick = () => { p1Color = c; document.querySelectorAll('.swatch').forEach(el => el.classList.remove('active')); s.classList.add('active'); };
    swatchBox.appendChild(s);
});

class Fighter {
    constructor(x, color, isBot = false) {
        this.pos = { x, y: 280 };
        this.vel = { x: 0, y: 0 };
        this.w = 40; this.h = 90;
        this.color = color;
        this.hp = 100; this.ce = 0;
        this.isBot = isBot;
        this.dir = isBot ? -1 : 1;
        this.anim = 0; // Animation timer
        this.isAtk = false; this.isCharge = false; this.jumps = 0;
    }

    draw() {
        ctx.save();
        ctx.translate(this.pos.x + 20, this.pos.y + 45);
        ctx.scale(this.dir, 1);
        ctx.strokeStyle = this.isCharge ? 'white' : this.color;
        ctx.lineWidth = 5;
        ctx.lineCap = 'round';

        // Animate based on state
        this.anim += 0.15;
        let walkCycle = Math.sin(this.anim) * 20;
        let breath = Math.sin(this.anim * 0.5) * 2;

        if (this.isAtk) { // Attack Pose
            this.drawStickman(0, -10, 20, -10, 30, 0, -10, 20, 10, 20);
        } else if (Math.abs(this.vel.x) > 0.1) { // Walk Pose
            this.drawStickman(0, breath, walkCycle, -walkCycle, -walkCycle, walkCycle, walkCycle, 35, -walkCycle, 35);
        } else { // Idle Pose
            this.drawStickman(0, breath, 10, 10, -10, -10, 10, 35, -10, 35);
        }
        
        ctx.restore();
    }

    drawStickman(hx, hy, rax, ray, lax, lay, rlx, rly, llx, lly) {
        // Head
        ctx.beginPath(); ctx.arc(0, -35 + hy, 10, 0, Math.PI*2); ctx.stroke();
        // Spine
        ctx.beginPath(); ctx.moveTo(0, -25 + hy); ctx.lineTo(0, 10); ctx.stroke();
        // Arms
        ctx.beginPath(); ctx.moveTo(0, -15); ctx.lineTo(rax, ray); ctx.stroke(); // Right
        ctx.beginPath(); ctx.moveTo(0, -15); ctx.lineTo(lax, lay); ctx.stroke(); // Left
        // Legs
        ctx.beginPath(); ctx.moveTo(0, 10); ctx.lineTo(rlx, rly); ctx.stroke(); // Right
        ctx.beginPath(); ctx.moveTo(0, 10); ctx.lineTo(llx, lly); ctx.stroke(); // Left
        
        if(this.isCharge) {
            ctx.shadowBlur = 15; ctx.shadowColor = this.color;
            ctx.stroke();
        }
    }

    update() {
        this.pos.x += this.vel.x; this.pos.y += this.vel.y;
        if (this.pos.y > 280) { this.pos.y = 280; this.vel.y = 0; this.jumps = 0; }
        else { this.vel.y += 0.8; }
        if (this.isCharge) this.ce = Math.min(100, this.ce + 0.5);
        this.draw();
    }
}

const p1 = new Fighter(100, p1Color);
const bot = new Fighter(650, '#ff2e2e', true);

function useDomain(c, t) {
    if (c.ce < 100 || domainInEffect) return;
    c.ce = 0; domainInEffect = true;
    document.getElementById('domain-ui').classList.add('show');
    document.getElementById('game-container').classList.add('domain-active-bg');
    
    setTimeout(() => {
        if (selectedDomain === 'slashes') {
            let h = 0; const itv = setInterval(() => {
                if (h++ > 20) { clearInterval(itv); endDomain(); }
                else { t.hp -= 3; shake = 10; }
            }, 100);
        } else {
            let count = 0; const itv = setInterval(() => {
                if (count++ >= 3) { clearInterval(itv); endDomain(); }
                else { t.hp -= 20; shake = 30; bursts.push(t.pos.x); setTimeout(() => bursts = [], 200); }
            }, 600);
        }
    }, 1000);
}

function endDomain() {
    domainInEffect = false;
    document.getElementById('domain-ui').classList.remove('show');
    document.getElementById('game-container').classList.remove('domain-active-bg');
}

function selectMode(m) { document.getElementById('mode-menu').style.display='none'; document.getElementById('menu').style.display='flex'; }
function openCustom() { document.getElementById('mode-menu').style.display='none'; document.getElementById('custom-menu').style.display='flex'; }
function closeCustom() { document.getElementById('custom-menu').style.display='none'; document.getElementById('mode-menu').style.display='flex'; p1.color = p1Color; }
function toggleDomainType() { selectedDomain = selectedDomain === 'slashes' ? 'burst' : 'slashes'; document.getElementById('dom-toggle').innerText = `TYPE: ${selectedDomain.toUpperCase()}`; }
function startGame() { document.getElementById('menu').style.display='none'; gameActive = true; animate(); }

const keys = { a: false, d: false };
const bind = (id, s, e) => {
    const el = document.getElementById(id);
    el.onmousedown = el.ontouchstart = (ev) => { ev.preventDefault(); s(); };
    el.onmouseup = el.onmouseleave = el.ontouchend = (ev) => { ev.preventDefault(); if(e) e(); };
};
bind('btn-left', () => {keys.a = true; p1.dir = -1;}, () => keys.a = false);
bind('btn-right', () => {keys.d = true; p1.dir = 1;}, () => keys.d = false);
bind('btn-up', () => { if(p1.jumps < 2) { p1.vel.y = -15; p1.jumps++; } });
bind('btn-charge', () => p1.isCharge = true, () => p1.isCharge = false);
bind('btn-atk', () => { p1.isAtk = true; setTimeout(()=>p1.isAtk=false, 200); });
bind('btn-dom', () => useDomain(p1, bot));

function animate() {
    if (!gameActive) return;
    ctx.setTransform(1,0,0,1,0,0); ctx.clearRect(0,0,800,400);
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }

    if (!gameOver) {
        if (!domainInEffect) {
            p1.vel.x = keys.a ? -5 : (keys.d ? 5 : 0);
            let dist = p1.pos.x - bot.pos.x;
            bot.vel.x = Math.abs(dist) > 60 ? (dist > 0 ? 3 : -3) : 0;
            if (Math.random() < 0.02) { bot.isAtk = true; setTimeout(()=>bot.isAtk=false, 200); }
            if (p1.isAtk && Math.abs(p1.pos.x - bot.pos.x) < 50) bot.hp -= 0.5;
            if (bot.isAtk && Math.abs(p1.pos.x - bot.pos.x) < 50) p1.hp -= 0.5;
        }
        if (p1.hp <= 0 || bot.hp <= 0) { gameOver = true; document.getElementById('ko-screen').style.display='flex'; }
    }

    bursts.forEach(bx => {
        ctx.fillStyle = 'white'; ctx.shadowBlur = 30; ctx.shadowColor = 'cyan';
        ctx.fillRect(bx - 20, 0, 80, 400);
    });

    p1.update(); bot.update();
    document.getElementById('p1-hp').style.width = p1.hp + "%";
    document.getElementById('p2-hp').style.width = bot.hp + "%";
    document.getElementById('p1-ce').style.width = p1.ce + "%";
    requestAnimationFrame(animate);
}
</script>
</body>
</html>
