<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PIXEL KAIZEN: DEFINITIVE</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; --green: #2eff7b; }
        body { margin: 0; background: #000; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        #game-container { position: relative; width: 800px; height: 400px; margin: 5px auto; border: 4px solid #333; overflow: hidden; background: #0a0a0a; }
        
        .hud { position: absolute; top: 10px; width: 100%; display: flex; justify-content: space-between; padding: 0 20px; box-sizing: border-box; z-index: 50; pointer-events: none;}
        .bar-bg { background: rgba(0,0,0,0.8); height: 16px; border: 2px solid #fff; width: 220px; }
        .hp-fill { background: var(--red); height: 100%; width: 100%; transition: width 0.2s; }
        .ce-bg { background: #000; height: 6px; margin-top: 2px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; }

        .menu-screen { position: absolute; inset: 0; background: rgba(0,0,0,0.95); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .menu-btn { margin: 6px; padding: 12px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 240px; text-transform: uppercase; font-size: 14px; }
        .menu-btn:hover { background: #fff; color: #000; }
        
        .swatch-box { display: flex; gap: 10px; margin-bottom: 15px; }
        .swatch { width: 35px; height: 35px; border: 2px solid #444; border-radius: 50%; cursor: pointer; }
        .swatch.active { border-color: #fff; transform: scale(1.2); }

        .controls-layer { position: absolute; bottom: 5px; width: 800px; left: 50%; transform: translateX(-50%); display: flex; justify-content: space-between; pointer-events: none; }
        .ctrl-group { display: flex; gap: 8px; pointer-events: auto; padding: 15px; }
        .btn { width: 65px; height: 65px; background: rgba(40,40,40,0.9); border: 2px solid #777; color: white; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 12px; user-select: none; }
        .btn:active { background: #fff; color: #000; border-color: #fff; }
        
        #domain-flash { position: absolute; inset: 0; pointer-events: none; z-index: 100; mix-blend-mode: overlay; opacity: 0; transition: 0.4s; }
        #stage-ui { position: absolute; top: 50px; left: 50%; transform: translateX(-50%); color: var(--green); font-size: 18px; z-index: 40; display: none; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="domain-flash"></div>
    <div id="stage-ui">STAGE 1</div>
    
    <div id="mode-menu" class="menu-screen">
        <h1 style="color:var(--blue)">KAIZEN 5.0</h1>
        <button class="menu-btn" onclick="showDiff('1v1')">VS COMPUTER</button>
        <button class="menu-btn" onclick="setupGame('survival', 'medium')">SURVIVAL GAUNTLET</button>
        <button class="menu-btn" onclick="setupGame('split')">SPLIT-SCREEN 1V1</button>
        <button class="menu-btn" style="border-color:var(--purple)" onclick="toggleMenu('custom-menu', true)">CUSTOMIZE</button>
    </div>

    <div id="diff-menu" class="menu-screen" style="display:none">
        <h2>SELECT DIFFICULTY</h2>
        <button class="menu-btn" onclick="setupGame(tempMode, 'easy')">EASY</button>
        <button class="menu-btn" onclick="setupGame(tempMode, 'medium')">NORMAL</button>
        <button class="menu-btn" onclick="setupGame(tempMode, 'hard')">KAIZEN</button>
    </div>

    <div id="custom-menu" class="menu-screen" style="display:none">
        <h3>CHARACTER COLOR</h3>
        <div class="swatch-box" id="swatches"></div>
        <h3>CHARACTER SHAPE</h3>
        <button class="menu-btn" id="shape-btn" onclick="toggleShape()">SHAPE: STICKMAN</button>
        <button class="menu-btn" onclick="toggleMenu('custom-menu', false)">DONE</button>
    </div>

    <div id="ko-screen" class="menu-screen" style="display:none; background:rgba(255,0,0,0.2)">
        <h1 style="font-size:60px">K.O.</h1>
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
let gameActive = false, currentGameMode = '', diff = 'medium', tempMode = '', gameOver = false;
let p1Color = '#00d4ff', p2Color = '#ff2e2e', pShape = 'stickman';
let p1, p2, clone = null, stage = 1, shake = 0, domainActive = false;
let hitSparks = [], particles = [];

// Init UI
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
function toggleShape() { pShape = pShape === 'stickman' ? 'box' : 'stickman'; document.getElementById('shape-btn').innerText = "SHAPE: " + pShape.toUpperCase(); }

class Fighter {
    constructor(x, color, isBot = false, id = 1) {
        this.id = id; this.isBot = isBot; this.color = color;
        this.reset(x);
    }
    reset(x) {
        this.pos = { x, y: 280 }; this.vel = { x: 0, y: 0 };
        this.hp = 100; this.ce = 0; this.dir = x > 400 ? -1 : 1;
        this.isAtk = false; this.atkF = 0; this.isCharge = false; this.jumps = 0;
        this.shape = this.id === 1 ? pShape : 'stickman';
    }
    update() {
        this.pos.x += this.vel.x; this.pos.y += this.vel.y;
        if (this.pos.y > 280) { this.pos.y = 280; this.vel.y = 0; this.jumps = 0; } else { this.vel.y += 0.8; }
        if (this.isCharge) this.ce = Math.min(100, this.ce + 0.3);
        if (this.isAtk) { this.atkF += 0.15; if(this.atkF >= 1) { this.isAtk = false; this.atkF = 0; } }
    }
    draw(c) {
        c.save(); c.translate(this.pos.x + 20, this.pos.y + 45);
        if (this.isCharge) { c.shadowBlur = 20; c.shadowColor = this.color; }
        c.scale(this.dir, 1); c.strokeStyle = this.color; c.fillStyle = this.color; c.lineWidth = 5; c.lineCap = 'round';
        
        let ext = this.isAtk ? Math.sin(this.atkF * Math.PI) * 35 : 0;
        let walk = (Math.abs(this.vel.x) > 0.1) ? Math.sin(Date.now()*0.012) * 15 : 0;

        if (this.shape === 'box') {
            c.fillRect(-15, -40, 30, 70);
            c.strokeStyle = '#fff'; c.strokeRect(-15, -40, 30, 70);
            if(this.isAtk) c.fillRect(15, -15, 20 + ext, 10);
        } else {
            // Full Stickman Body Parts
            c.beginPath(); c.arc(0, -45, 12, 0, Math.PI*2); c.stroke(); // Head
            c.beginPath(); c.moveTo(0, -33); c.lineTo(0, 10); c.stroke(); // Torso
            // Two Arms
            c.beginPath(); c.moveTo(0, -25); c.lineTo(15 + ext, -10); c.stroke(); 
            c.beginPath(); c.moveTo(0, -25); c.lineTo(-15, 0); c.stroke();
            // Two Legs
            c.beginPath(); c.moveTo(0, 10); c.lineTo(15 + walk, 40); c.stroke();
            c.beginPath(); c.moveTo(0, 10); c.lineTo(-15 - walk, 40); c.stroke();
        }
        c.restore();
    }
}

function setupGame(mode, d) {
    currentGameMode = mode; diff = d || 'medium'; gameOver = false;
    p1 = new Fighter(150, p1Color, false, 1);
    p2 = new Fighter(650, p2Color, mode !== 'split', 2);
    if(mode === 'split') document.getElementById('p2-ctrls').style.display = 'flex';
    if(mode === 'survival') { stage = 1; document.getElementById('stage-ui').style.display = 'block'; }
    toggleMenu('mode-menu', false); toggleMenu('diff-menu', false);
    gameActive = true; loop();
}

function checkHit(a, v) {
    if(a.isAtk && a.atkF > 0.4 && a.atkF < 0.7) {
        let hitX = a.pos.x + (a.dir * 45);
        if(Math.abs(hitX - v.pos.x) < 45 && Math.abs(a.pos.y - v.pos.y) < 60) {
            v.hp -= 2.5; shake = 5;
            hitSparks.push({x: v.pos.x+20, y: v.pos.y+40, life: 10});
        }
    }
}

function updateAI(bot, target) {
    let d = target.pos.x - bot.pos.x;
    let speed = diff === 'easy' ? 2 : (diff === 'medium' ? 4 : 6);
    bot.vel.x = Math.abs(d) > 70 ? (d > 0 ? speed : -speed) : 0;
    bot.dir = d > 0 ? 1 : -1;
    if(Math.random() < 0.03) bot.isAtk = true;
}

function loop() {
    if (!gameActive || gameOver) return;
    ctx.setTransform(1,0,0,1,0,0); ctx.clearRect(0, 0, 800, 400);

    // Camera Zoom Logic
    let dist = Math.abs(p1.pos.x - p2.pos.x);
    let scale = Math.max(0.65, Math.min(1.1, 800/(dist+250)));
    ctx.translate(400, 200); ctx.scale(scale, scale);
    ctx.translate(-(p1.pos.x+p2.pos.x)/2 - 20, -260);
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }

    // Update & Collision
    p1.vel.x = keys.p1l ? -6 : (keys.p1r ? 6 : 0);
    if(currentGameMode === 'split') p2.vel.x = keys.p2l ? -6 : (keys.p2r ? 6 : 0);
    
    p1.update(); p2.update();
    if(p2.isBot) updateAI(p2, p1);
    checkHit(p1, p2); checkHit(p2, p1);

    // Death Checks
    if (p1.hp <= 0) { gameOver = true; toggleMenu('ko-screen', true); }
    if (p2.hp <= 0) {
        if(currentGameMode === 'survival') {
            stage++; p1.hp = Math.min(100, p1.hp + 40);
            p2.reset(650); p2.hp = 100 + (stage * 10);
            document.getElementById('stage-ui').innerText = "STAGE " + stage;
        } else {
            gameOver = true; toggleMenu('ko-screen', true);
        }
    }

    // Drawing
    p1.draw(ctx); p2.draw(ctx);
    hitSparks.forEach((s, i) => {
        ctx.strokeStyle = '#fff'; ctx.beginPath(); ctx.moveTo(s.x-10, s.y-10); ctx.lineTo(s.x+10, s.y+10); ctx.stroke();
        s.life--; if(s.life <= 0) hitSparks.splice(i, 1);
    });

    // UI Updates
    document.getElementById('p1-hp').style.width = p1.hp + "%";
    document.getElementById('p2-hp').style.width = (p2.hp / (p2.isBot ? (100 + stage*10)/100 : 1)) + "%";
    document.getElementById('p1-ce').style.width = p1.ce + "%";
    document.getElementById('p2-ce').style.width = p2.ce + "%";
    
    requestAnimationFrame(loop);
}

const keys = {};
const bind = (id, k) => {
    const el = document.getElementById(id);
    el.ontouchstart = (e) => { e.preventDefault(); 
        if(id.includes('j')) { 
            let target = k === 'p1' ? p1 : p2;
            if(target.jumps < 2) { target.vel.y = -16; target.jumps++; }
        }
        else if(id.includes('a')) { if(k === 'p1') p1.isAtk = true; else p2.isAtk = true; }
        else if(id.includes('c')) { if(k === 'p1') p1.isCharge = true; else p2.isCharge = true; }
        else keys[k] = true;
    };
    el.ontouchend = () => keys[k] = false;
};

bind('p1-l', 'p1l'); bind('p1-r', 'p1r'); bind('p1-j', 'p1'); bind('p1-a', 'p1'); bind('p1-c', 'p1');
bind('p2-l', 'p2l'); bind('p2-r', 'p2r'); bind('p2-j', 'p2'); bind('p2-a', 'p2'); bind('p2-c', 'p2');
</script>
</body>
</html>
