<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PIXEL KAIZEN | DOUBLE JUMP EDITION</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; }
        body { margin: 0; background: #050505; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        
        #game-container { position: relative; width: 800px; height: 400px; margin: 10px auto; outline: 4px solid #222; overflow: hidden; background: #111; }
        #impact-flash { position: absolute; inset: 0; background: white; opacity: 0; z-index: 101; pointer-events: none; }
        #domain-ui { position: absolute; inset: 0; background: black; opacity: 0; display: flex; align-items: center; justify-content: center; z-index: 100; pointer-events: none; transition: opacity 0.2s; }
        #domain-ui h1 { font-size: 50px; color: var(--purple); text-shadow: 0 0 20px var(--purple); font-style: italic; }

        .hud { position: absolute; top: 15px; width: 100%; display: flex; justify-content: space-between; padding: 0 30px; box-sizing: border-box; z-index: 10; }
        .stat-group { width: 40%; }
        .bar-bg { background: #000; height: 20px; border: 2px solid #fff; }
        .hp-fill { background: linear-gradient(90deg, var(--red), #ff7b7b); height: 100%; width: 100%; transition: width 0.1s; }
        .ce-bg { background: #000; height: 6px; margin-top: 4px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; box-shadow: 0 0 8px var(--blue); }

        #menu { position: absolute; inset: 0; background: rgba(0,0,0,0.9); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .diff-btn { margin: 10px; padding: 10px 30px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 200px; }
        .diff-btn:hover { background: white; color: black; }

        #ko-screen { position: absolute; inset: 0; background: rgba(0,0,0,0.85); display: none; flex-direction: column; align-items: center; justify-content: center; z-index: 200; }
        #ko-screen h1 { font-size: 80px; color: var(--red); text-shadow: 0 0 20px var(--red); }

        .mobile-btns { display: flex; justify-content: space-between; width: 800px; margin: 5px auto; max-width: 95vw; }
        .group { display: flex; gap: 8px; }
        .btn { width: 60px; height: 60px; background: #222; border: 2px solid #444; color: white; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 18px; }
        .btn-special { border-color: var(--blue); color: var(--blue); }
        .btn-domain { border-color: var(--purple); color: var(--purple); }
    </style>
</head>
<body>

<div id="game-container">
    <div id="menu">
        <h1 style="color: var(--blue)">PIXEL KAIZEN</h1>
        <button class="diff-btn" onclick="startGame('easy')">EASY</button>
        <button class="diff-btn" onclick="startGame('medium')">MEDIUM</button>
        <button class="diff-btn" onclick="startGame('hard')">HARD</button>
    </div>

    <div id="impact-flash"></div>
    <div id="domain-ui"><h1>DOMAIN EXPANSION</h1></div>
    <div id="ko-screen">
        <h1 id="ko-text">K.O.</h1>
        <button style="background:white; color:black; border:none; padding:10px 20px;" onclick="location.reload()">REMATCH</button>
    </div>

    <div class="hud">
        <div class="stat-group">
            <div class="bar-bg"><div id="p1-hp" class="hp-fill"></div></div>
            <div class="ce-bg"><div id="p1-ce" class="ce-fill"></div></div>
        </div>
        <div class="stat-group" style="text-align: right;">
            <div class="bar-bg"><div id="p2-hp" class="hp-fill" style="float:right"></div></div>
            <div class="ce-bg"><div id="p2-ce" class="ce-fill" style="float:right"></div></div>
        </div>
    </div>
    <canvas id="gameCanvas" width="800" height="400"></canvas>
</div>

<div class="mobile-btns">
    <div class="group">
        <div class="btn" id="btn-left">◀</div>
        <div class="btn" id="btn-right">▶</div>
        <div class="btn btn-special" id="btn-charge">CE</div>
    </div>
    <div class="group">
        <div class="btn" id="btn-up">UP</div>
        <div class="btn" id="btn-atk">HIT</div>
        <div class="btn btn-domain" id="btn-dom">DE</div>
    </div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameOver = false;
let shake = 0;
let difficulty = 'medium';
let gameActive = false;

const diffSettings = {
    easy: { speed: 2, attackRate: 0.01, ceRate: 0.05 },
    medium: { speed: 4, attackRate: 0.03, ceRate: 0.1 },
    hard: { speed: 6, attackRate: 0.08, ceRate: 0.2 }
};

class Fighter {
    constructor({ x, y, color, isBot = false }) {
        this.pos = { x, y }; this.vel = { x: 0, y: 0 };
        this.w = 50; this.h = 100; this.color = color;
        this.hp = 100; this.ce = 0; this.isBot = isBot;
        this.isAtk = false; this.isCharge = false;
        this.dir = isBot ? -1 : 1; 
        this.onGrd = false;
        this.jumps = 0; // Track number of jumps
    }

    draw() {
        ctx.save();
        if (this.isCharge) { ctx.shadowBlur = 20; ctx.shadowColor = '#00d4ff'; ctx.fillStyle = '#fff'; }
        else { ctx.fillStyle = this.color; }
        ctx.fillRect(this.pos.x, this.pos.y, this.w, this.h);
        if (this.isAtk) {
            ctx.fillStyle = "white";
            ctx.shadowBlur = 10; ctx.shadowColor = "white";
            ctx.fillRect(this.pos.x + (this.dir === 1 ? 50 : -40), this.pos.y + 20, 40, 15);
        }
        ctx.restore();
    }

    update() {
        this.pos.x += this.vel.x; this.pos.y += this.vel.y;
        
        // Floor Collision Logic
        if (this.pos.y + this.h >= 380) { 
            this.vel.y = 0; 
            this.pos.y = 280; 
            this.onGrd = true; 
            this.jumps = 0; // Reset jumps when touching ground
        } else { 
            this.vel.y += 0.8; 
            this.onGrd = false; 
        }
        
        if (this.isCharge) { this.ce = Math.min(100, this.ce + 0.5); this.vel.x = 0; }
        this.draw();
    }

    // New Jump Function
    jump() {
        if (this.jumps < 2) { // Allow up to 2 jumps
            this.vel.y = -16;
            this.jumps++;
            if (this.jumps === 2) triggerHitEffect(5); // Little shake on double jump
        }
    }
}

const p1 = new Fighter({ x: 100, y: 0, color: '#006eff' });
const bot = new Fighter({ x: 650, y: 0, color: '#ff2e2e', isBot: true });

function triggerHitEffect(power) {
    shake = power;
    const flash = document.getElementById('impact-flash');
    flash.style.opacity = 0.4;
    setTimeout(() => flash.style.opacity = 0, 50);
}

function useDomain(c, t) {
    if (c.ce < 100 || gameOver) return;
    c.ce = 0; 
    document.getElementById('domain-ui').style.opacity = 1;
    setTimeout(() => { 
        t.hp = Math.max(0, t.hp - 35); 
        document.getElementById('domain-ui').style.opacity = 0;
        triggerHitEffect(30);
    }, 1200);
}

function startGame(level) {
    difficulty = level;
    document.getElementById('menu').style.display = 'none';
    gameActive = true;
    animate();
}

const keys = { a: false, d: false };
const bind = (id, s, e) => {
    const el = document.getElementById(id);
    el.addEventListener('touchstart', (ev) => { ev.preventDefault(); s(); });
    el.addEventListener('touchend', (ev) => { ev.preventDefault(); if(e) e(); });
};

bind('btn-left', () => {keys.a = true; p1.dir = -1;}, () => keys.a = false);
bind('btn-right', () => {keys.d = true; p1.dir = 1;}, () => keys.d = false);
bind('btn-up', () => { p1.jump(); }); // Uses the new jump logic
bind('btn-charge', () => p1.isCharge = true, () => p1.isCharge = false);
bind('btn-atk', () => { p1.isAtk = true; setTimeout(()=>p1.isAtk=false, 150); });
bind('btn-dom', () => useDomain(p1, bot));

// Keyboard support for Double Jump
window.addEventListener('keydown', e => {
    if (e.key === 'a') { keys.a = true; p1.dir = -1; }
    if (e.key === 'd') { keys.d = true; p1.dir = 1; }
    if (e.key === 'w') p1.jump();
    if (e.key === 'j') { p1.isAtk = true; setTimeout(()=>p1.isAtk=false, 150); }
    if (e.key === 'c') p1.isCharge = true;
    if (e.key === 'l') useDomain(p1, bot);
});
window.addEventListener('keyup', e => {
    if (e.key === 'a') keys.a = false;
    if (e.key === 'd') keys.d = false;
    if (e.key === 'c') p1.isCharge = false;
});

function animate() {
    if (!gameActive) return;
    ctx.setTransform(1,0,0,1,0,0); ctx.clearRect(0,0,800,400);
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }
    
    if (!gameOver) {
        p1.vel.x = keys.a ? -6 : (keys.d ? 6 : 0);
        
        const settings = diffSettings[difficulty];
        const dist = p1.pos.x - bot.pos.x;
        bot.dir = dist > 0 ? 1 : -1;

        if (Math.abs(dist) > 70) {
            bot.vel.x = dist > 0 ? settings.speed : -settings.speed;
        } else {
            bot.vel.x = 0;
            if (Math.random() < settings.attackRate) { bot.isAtk = true; setTimeout(()=>bot.isAtk=false, 150); }
        }
        
        bot.ce = Math.min(100, bot.ce + settings.ceRate);
        if (bot.ce >= 100 && Math.random() < 0.01) useDomain(bot, p1);

        [p1, bot].forEach(atk => {
            const vic = atk === p1 ? bot : p1;
            const hX = atk.dir === 1 ? atk.pos.x + 50 : atk.pos.x - 40;
            if (atk.isAtk && hX < vic.pos.x + 50 && hX + 40 > vic.pos.x && atk.pos.y < vic.pos.y + 100 && atk.pos.y + 50 > vic.pos.y) {
                vic.hp -= 0.6; atk.ce = Math.min(100, atk.ce + 0.8);
                if (Math.random() < 0.1) triggerHitEffect(5); 
            }
        });

        if (p1.hp <= 0 || bot.hp <= 0) {
            gameOver = true;
            document.getElementById('ko-screen').style.display = 'flex';
            document.getElementById('ko-text').innerText = p1.hp <= 0 ? "K.O. - DEFEAT" : "K.O. - VICTORY";
            triggerHitEffect(40);
        }
    }

    p1.update(); bot.update();
    document.getElementById('p1-hp').style.width = p1.hp + "%";
    document.getElementById('p2-hp').style.width = bot.hp + "%";
    document.getElementById('p1-ce').style.width = p1.ce + "%";
    document.getElementById('p2-ce').style.width = bot.ce + "%";
    requestAnimationFrame(animate);
}
</script>
</body>
</html>
