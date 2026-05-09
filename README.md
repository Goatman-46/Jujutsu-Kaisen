<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>STICKMAN KAIZEN | ULTIMATE</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; --green: #2eff7b; }
        body { margin: 0; background: #000; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        #game-container { position: relative; width: 800px; height: 400px; margin: 10px auto; outline: 4px solid #222; overflow: hidden; background: #111; }
        
        /* Domain Background Effects */
        .domain-slashes { background: radial-gradient(circle, #1a0026 0%, #000 100%) !important; }
        .domain-whiteout { background: #fff !important; transition: background 1s ease-in; }
        .domain-rain { background: #00051a !important; }

        #impact-flash { position: absolute; inset: 0; background: white; opacity: 0; z-index: 101; pointer-events: none; }
        #domain-ui { position: absolute; inset: 0; opacity: 0; display: flex; align-items: center; justify-content: center; z-index: 100; pointer-events: none; transition: 0.5s; }
        #domain-ui.show { opacity: 1; transform: scale(1.2); }
        
        .hud { position: absolute; top: 15px; width: 100%; display: flex; justify-content: space-between; padding: 0 30px; box-sizing: border-box; z-index: 10; }
        .bar-bg { background: #000; height: 18px; border: 2px solid #fff; width: 280px; }
        .hp-fill { background: linear-gradient(90deg, #ff2e2e, #ff7b7b); height: 100%; width: 100%; transition: 0.2s; }
        .ce-bg { background: #000; height: 6px; margin-top: 4px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; box-shadow: 0 0 10px var(--blue); }
        
        #menu, #mode-menu, #custom-menu { position: absolute; inset: 0; background: rgba(0,0,0,0.9); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .menu-btn { margin: 5px; padding: 12px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 260px; text-transform: uppercase; }
        .menu-btn:hover { background: white; color: black; }
        
        .swatch-container { display: flex; gap: 8px; margin: 10px 0; }
        .swatch { width: 30px; height: 30px; border: 2px solid #444; cursor: pointer; border-radius: 50%; }
        .swatch.active { border-color: white; transform: scale(1.2); }

        .mobile-btns { display: flex; justify-content: space-between; width: 800px; margin: 5px auto; max-width: 95vw; }
        .btn { width: 60px; height: 60px; background: #222; border: 2px solid #444; color: white; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 14px; user-select: none; }
        #stage-tag { position: absolute; top: 50px; left: 50%; transform: translateX(-50%); color: var(--green); font-size: 14px; display: none; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="mode-menu">
        <h1 style="color: var(--blue)">PIXEL KAIZEN 2.0</h1>
        <button class="menu-btn" onclick="selectMode('1v1')">Local Duel</button>
        <button class="menu-btn" onclick="selectMode('survival')">Survival Gauntlet</button>
        <button class="menu-btn" style="border-color: var(--purple)" onclick="openCustom()">Customization</button>
    </div>

    <div id="custom-menu" style="display:none">
        <h2>SKIN COLOR</h2>
        <div class="swatch-container" id="swatch-box"></div>
        <h2>DOMAIN EXPANSION</h2>
        <button class="menu-btn" id="dom-toggle" onclick="cycleDomain()">TYPE: SLASHES</button>
        <button class="menu-btn" onclick="closeCustom()">SAVE</button>
    </div>

    <div id="menu" style="display:none">
        <h2>DIFFICULTY</h2>
        <button class="menu-btn" onclick="startGame('easy')">Easy</button>
        <button class="menu-btn" onclick="startGame('medium')">Normal</button>
        <button class="menu-btn" onclick="startGame('hard')">Hard</button>
    </div>

    <div id="impact-flash"></div>
    <div id="domain-ui"><h1 id="domain-name">DOMAIN EXPANSION</h1></div>
    <div id="stage-tag">STAGE 1</div>
    
    <div id="ko-screen" style="display:none; position:absolute; inset:0; background:rgba(0,0,0,0.8); z-index:200; flex-direction:column; align-items:center; justify-content:center;">
        <h1 id="ko-title" style="font-size:60px; color:var(--red)">K.O.</h1>
        <button class="menu-btn" onclick="location.reload()">MAIN MENU</button>
    </div>

    <div class="hud">
        <div><div class="bar-bg"><div id="p1-hp" class="hp-fill"></div></div><div class="ce-bg"><div id="p1-ce" class="ce-fill"></div></div></div>
        <div style="text-align: right;"><div class="bar-bg"><div id="p2-hp" class="hp-fill"></div></div><div class="ce-bg"><div id="p2-ce" class="ce-fill"></div></div></div>
    </div>
    <canvas id="gameCanvas" width="800" height="400"></canvas>
</div>

<div class="mobile-btns">
    <div style="display:flex; gap:8px;"><div class="btn" id="btn-left">◀</div><div class="btn" id="btn-right">▶</div><div class="btn" id="btn-charge">AURA</div></div>
    <div style="display:flex; gap:8px;"><div class="btn" id="btn-up">JUMP</div><div class="btn" id="btn-atk">HIT</div><div class="btn" id="btn-dom" style="border-color:var(--purple)">DE</div></div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameOver = false, shake = 0, gameActive = false, gameMode = '1v1', domainInEffect = false;
let p1Color = '#00d4ff', currentDomainIdx = 0, stage = 1;
const domainTypes = ['slashes', 'burst', 'whiteout', 'rain'];
let particles = [], hitSparks = [];

const colors = ['#00d4ff', '#ffffff', '#ffeb3b', '#e91e63', '#2eff7b', '#ff9800'];
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
        this.reset(x, color, isBot);
    }
    reset(x, color, isBot) {
        this.pos = { x, y: 280 }; this.vel = { x: 0, y: 0 };
        this.color = color; this.hp = 100; this.ce = 0;
        this.isBot = isBot; this.dir = isBot ? -1 : 1;
        this.anim = 0; this.isAtk = false; this.isCharge = false; this.jumps = 0;
    }
    draw() {
        ctx.save();
        ctx.translate(this.pos.x + 20, this.pos.y + 45);
        
        // Aura Animation
        if (this.isCharge) {
            ctx.save();
            ctx.globalAlpha = 0.6;
            for(let i=0; i<3; i++) {
                ctx.beginPath();
                let pulse = Math.sin(Date.now()*0.01 + i) * 10;
                ctx.fillStyle = this.color;
                ctx.moveTo(-25-pulse, 40);
                ctx.quadraticCurveTo(0, -80-pulse, 25+pulse, 40);
                ctx.fill();
            }
            ctx.restore();
        }

        ctx.scale(this.dir, 1);
        ctx.strokeStyle = this.color; ctx.lineWidth = 5; ctx.lineCap = 'round';
        this.anim += 0.15;
        let walk = Math.sin(this.anim) * 20;
        
        if (this.isAtk) this.drawMan(0, 0, 25, -5, 25, 5, 10, 35, -10, 35);
        else if (Math.abs(this.vel.x) > 0.1) this.drawMan(0, 0, walk, -walk, -walk, walk, walk, 35, -walk, 35);
        else this.drawMan(0, Math.sin(this.anim*0.5)*2, 10, 10, -10, -10, 10, 35, -10, 35);
        
        ctx.restore();
    }
    drawMan(hx, hy, rax, ray, lax, lay, rlx, rly, llx, lly) {
        ctx.beginPath(); ctx.arc(0, -35 + hy, 10, 0, Math.PI*2); ctx.stroke(); // Head
        ctx.beginPath(); ctx.moveTo(0, -25 + hy); ctx.lineTo(0, 10); ctx.stroke(); // Body
        ctx.beginPath(); ctx.moveTo(0, -15); ctx.lineTo(rax, ray); ctx.stroke(); // Arms
        ctx.beginPath(); ctx.moveTo(0, -15); ctx.lineTo(lax, lay); ctx.stroke();
        ctx.beginPath(); ctx.moveTo(0, 10); ctx.lineTo(rlx, rly); ctx.stroke(); // Legs
        ctx.beginPath(); ctx.moveTo(0, 10); ctx.lineTo(llx, lly); ctx.stroke();
    }
    update() {
        this.pos.x += this.vel.x; this.pos.y += this.vel.y;
        if (this.pos.y > 280) { this.pos.y = 280; this.vel.y = 0; this.jumps = 0; }
        else this.vel.y += 0.8;
        if (this.isCharge) this.ce = Math.min(100, this.ce + 0.8);
        this.draw();
    }
}

let p1 = new Fighter(100, p1Color);
let bot = new Fighter(650, '#ff2e2e', true);

function spawnEnemy() {
    bot.reset(650, stage % 5 === 0 ? '#2eff7b' : '#ff2e2e', true);
    if (stage % 5 === 0) bot.hp = 150; // Boss HP
    document.getElementById('stage-tag').innerText = "STAGE " + stage;
}

function selectMode(m) { gameMode = m; document.getElementById('mode-menu').style.display='none'; document.getElementById('menu').style.display='flex'; }
function openCustom() { document.getElementById('mode-menu').style.display='none'; document.getElementById('custom-menu').style.display='flex'; }
function closeCustom() { document.getElementById('custom-menu').style.display='none'; document.getElementById('mode-menu').style.display='flex'; p1.color = p1Color; }
function cycleDomain() { currentDomainIdx = (currentDomainIdx + 1) % domainTypes.length; document.getElementById('dom-toggle').innerText = "TYPE: " + domainTypes[currentDomainIdx].toUpperCase(); }

function startGame(diff) {
    document.getElementById('menu').style.display='none';
    if(gameMode === 'survival') { stage = 1; spawnEnemy(); document.getElementById('stage-tag').style.display = 'block'; }
    gameActive = true; animate();
}

function useDomain(c, t) {
    if (c.ce < 100 || domainInEffect) return;
    c.ce = 0; domainInEffect = true;
    const type = c.isBot ? domainTypes[Math.floor(Math.random()*domainTypes.length)] : domainTypes[currentDomainIdx];
    
    const ui = document.getElementById('domain-ui');
    const container = document.getElementById('game-container');
    ui.classList.add('show');
    document.getElementById('domain-name').innerText = type.toUpperCase() + " EXPANSION";

    setTimeout(() => {
        if (type === 'slashes') {
            container.classList.add('domain-slashes');
            let h = 0; const itv = setInterval(() => {
                if (h++ > 30) { endDomain(); clearInterval(itv); }
                else { t.hp -= 2; createHitSpark(t.pos.x+20, t.pos.y+40); shake = 5; }
            }, 100);
        } 
        else if (type === 'burst') {
            let h = 0; const itv = setInterval(() => {
                if (h++ >= 3) { endDomain(); clearInterval(itv); }
                else { t.hp -= 20; shake = 40; particles.push({x: t.pos.x, type: 'burst'}); setTimeout(()=>particles=[], 300); }
            }, 600);
        }
        else if (type === 'whiteout') {
            container.classList.add('domain-whiteout');
            let radius = 0;
            const itv = setInterval(() => {
                radius += 20;
                if(radius > 1000) { t.hp -= 50; endDomain(); clearInterval(itv); container.classList.remove('domain-whiteout'); }
                shake = 5;
            }, 30);
        }
        else if (type === 'rain') {
            container.classList.add('domain-rain');
            let drops = 0;
            const itv = setInterval(() => {
                if(drops++ > 60) { endDomain(); clearInterval(itv); container.classList.remove('domain-rain'); }
                else {
                    let rx = Math.random() * 800;
                    particles.push({x: rx, y: 0, type: 'rain'});
                    if(Math.abs(rx - t.pos.x) < 40) { t.hp -= 1; createHitSpark(t.pos.x+20, t.pos.y+40); }
                }
            }, 50);
        }
    }, 1000);
}

function endDomain() {
    domainInEffect = false;
    document.getElementById('domain-ui').classList.remove('show');
    document.getElementById('game-container').className = '';
}

function createHitSpark(x, y) {
    hitSparks.push({x, y, life: 10});
}

function animate() {
    if (!gameActive) return;
    ctx.clearRect(0,0,800,400);
    if (shake > 0) { ctx.save(); ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }

    if (!gameOver) {
        if (!domainInEffect) {
            p1.vel.x = keys.a ? -5 : (keys.d ? 5 : 0);
            let dist = p1.pos.x - bot.pos.x;
            bot.vel.x = Math.abs(dist) > 60 ? (dist > 0 ? 3 : -3) : 0;
            if (Math.random() < 0.03) { bot.isAtk = true; setTimeout(()=>bot.isAtk=false, 200); }
            if (p1.isAtk && Math.abs(p1.pos.x - bot.pos.x) < 50) { bot.hp -= 0.8; createHitSpark(bot.pos.x+20, bot.pos.y+40); }
            if (bot.isAtk && Math.abs(p1.pos.x - bot.pos.x) < 50) { p1.hp -= 0.8; createHitSpark(p1.pos.x+20, p1.pos.y+40); }
            if (bot.ce >= 100 && Math.random() < 0.01) useDomain(bot, p1);
        }
        
        // Survival Logic
        if (bot.hp <= 0) {
            if (gameMode === 'survival') { stage++; spawnEnemy(); p1.hp = Math.min(100, p1.hp + 20); }
            else { gameOver = true; document.getElementById('ko-screen').style.display='flex'; }
        }
        if (p1.hp <= 0) { gameOver = true; document.getElementById('ko-screen').style.display='flex'; }
    }

    // Drawing hit sparks
    hitSparks.forEach((s, i) => {
        ctx.strokeStyle = 'white'; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(s.x-10, s.y-10); ctx.lineTo(s.x+10, s.y+10); ctx.stroke();
        ctx.beginPath(); ctx.moveTo(s.x+10, s.y-10); ctx.lineTo(s.x-10, s.y+10); ctx.stroke();
        s.life--; if(s.life <= 0) hitSparks.splice(i, 1);
    });

    // Drawing Particles (Rain/Burst)
    particles.forEach((p, i) => {
        if(p.type === 'burst') { ctx.fillStyle = 'white'; ctx.fillRect(p.x-20, 0, 80, 400); }
        if(p.type === 'rain') { 
            ctx.fillStyle = 'cyan'; p.y += 10; ctx.fillRect(p.x, p.y, 4, 15);
            if(p.y > 400) particles.splice(i,1);
        }
    });

    p1.update(); bot.update();
    if(shake > 0) ctx.restore();
    
    document.getElementById('p1-hp').style.width = p1.hp + "%";
    document.getElementById('p2-hp').style.width = (bot.hp / (bot.isBoss ? 1.5 : 1)) + "%";
    document.getElementById('p1-ce').style.width = p1.ce + "%";
    document.getElementById('p2-ce').style.width = bot.ce + "%";
    requestAnimationFrame(animate);
}

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
</script>
</body>
</html>
