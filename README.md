<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PIXEL KAIZEN | BALANCED EDITION</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; --green: #2eff7b; }
        body { margin: 0; background: #050505; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        
        #game-container { position: relative; width: 800px; height: 400px; margin: 10px auto; outline: 4px solid #222; overflow: hidden; background: #111; transition: background 0.5s; }
        .domain-active-bg { background: radial-gradient(circle, #1a0026 0%, #000 100%) !important; outline-color: var(--purple) !important; }
        .boss-domain-bg { background: radial-gradient(circle, #440000 0%, #000 100%) !important; outline-color: var(--red) !important; }

        #impact-flash { position: absolute; inset: 0; background: white; opacity: 0; z-index: 101; pointer-events: none; }
        
        #domain-ui { position: absolute; inset: 0; opacity: 0; display: flex; align-items: center; justify-content: center; z-index: 100; pointer-events: none; }
        #domain-ui h1 { font-size: 60px; color: white; text-shadow: 0 0 30px var(--purple); font-style: italic; transform: scale(0.5); transition: 0.3s; }
        #domain-ui.show { opacity: 1; }
        #domain-ui.show h1 { transform: scale(1.2); }
        .boss-text { color: var(--blue) !important; text-shadow: 0 0 30px var(--blue) !important; }

        .hud { position: absolute; top: 15px; width: 100%; display: flex; justify-content: space-between; padding: 0 30px; box-sizing: border-box; z-index: 10; }
        #stage-counter { position: absolute; top: 50px; width: 100%; text-align: center; color: var(--green); font-size: 14px; display: none; }
        .stat-group { width: 40%; }
        .bar-bg { background: #000; height: 20px; border: 2px solid #fff; }
        .hp-fill { background: linear-gradient(90deg, var(--red), #ff7b7b); height: 100%; width: 100%; transition: width 0.1s; }
        .ce-bg { background: #000; height: 6px; margin-top: 4px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; box-shadow: 0 0 8px var(--blue); }

        #menu, #mode-menu { position: absolute; inset: 0; background: rgba(0,0,0,0.95); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .menu-btn { margin: 10px; padding: 15px 40px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 250px; font-size: 18px; }
        .menu-btn:hover { background: white; color: black; }

        #ko-screen { position: absolute; inset: 0; background: rgba(0,0,0,0.85); display: none; flex-direction: column; align-items: center; justify-content: center; z-index: 200; }
        #ko-screen h1 { font-size: 80px; color: var(--red); text-shadow: 0 0 20px var(--red); }

        .mobile-btns { display: flex; justify-content: space-between; width: 800px; margin: 5px auto; max-width: 95vw; }
        .group { display: flex; gap: 8px; }
        .btn { width: 60px; height: 60px; background: #222; border: 2px solid #444; color: white; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 18px; position: relative; }
        .btn-special { border-color: var(--blue); color: var(--blue); }
        .btn-domain { border-color: var(--purple); color: var(--purple); }
        .btn-disabled { opacity: 0.3; border-color: #555; color: #555; }
        .cooldown-overlay { position: absolute; font-size: 12px; bottom: 2px; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="mode-menu">
        <h1 style="color: var(--purple)">SELECT MODE</h1>
        <button class="menu-btn" onclick="selectMode('1v1')">1v1 BATTLE</button>
        <button class="menu-btn" onclick="selectMode('survival')">SURVIVAL GAUNTLET</button>
    </div>

    <div id="menu" style="display:none">
        <h1 id="diff-title" style="color: var(--blue)">SELECT DIFFICULTY</h1>
        <button class="menu-btn" onclick="startGame('easy')">EASY</button>
        <button class="menu-btn" onclick="startGame('medium')">MEDIUM</button>
        <button class="menu-btn" onclick="startGame('hard')">HARD</button>
    </div>

    <div id="impact-flash"></div>
    <div id="domain-ui"><h1 id="domain-text">DOMAIN EXPANSION</h1></div>
    <div id="stage-counter">STAGE 1</div>
    
    <div id="ko-screen">
        <h1 id="ko-text">K.O.</h1>
        <p id="survival-result" style="display:none"></p>
        <button style="background:white; color:black; border:none; padding:10px 20px;" onclick="location.reload()">BACK TO MENU</button>
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
    <div class="group"><div class="btn" id="btn-left">◀</div><div class="btn" id="btn-right">▶</div><div class="btn btn-special" id="btn-charge">CE</div></div>
    <div class="group">
        <div class="btn" id="btn-up">UP</div>
        <div class="btn" id="btn-atk">HIT</div>
        <div class="btn btn-domain" id="btn-dom">
            DE
            <span id="de-timer" class="cooldown-overlay"></span>
        </div>
    </div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameOver = false;
let shake = 0;
let difficulty = 'medium';
let gameActive = false;
let gameMode = '1v1';
let domainInEffect = false;
let slashes = [];

// Survival/Cooldown Variables
let stage = 1;
let enemiesDefeated = 0;
let maxStages = 50;
let p1LastDomain = 0;
let botLastDomain = 0;
const domainCooldown = 10000; // 10 seconds

const diffSettings = {
    easy: { speed: 2, attackRate: 0.01, ceRate: 0.05 },
    medium: { speed: 4, attackRate: 0.03, ceRate: 0.1 },
    hard: { speed: 6, attackRate: 0.08, ceRate: 0.2 }
};

class SlashEffect {
    constructor(x, y, w, h) {
        this.startX = x + Math.random() * w; this.startY = y + Math.random() * h;
        this.endX = x + Math.random() * w; this.endY = y + Math.random() * h;
        this.color = Math.random() < 0.5 ? '#ff2e2e' : '#00d4ff';
        this.alpha = 1; this.life = 10;
    }
    draw() {
        ctx.save(); ctx.globalAlpha = this.alpha; ctx.strokeStyle = this.color; ctx.lineWidth = 3;
        ctx.beginPath(); ctx.moveTo(this.startX, this.startY); ctx.lineTo(this.endX, this.endY); ctx.stroke(); ctx.restore();
    }
    update() { this.life--; this.alpha = this.life / 10; }
}

class Fighter {
    constructor({ x, y, color, isBot = false, isBoss = false }) {
        this.reset(x, y, color, isBot, isBoss);
    }
    reset(x, y, color, isBot, isBoss) {
        this.isBoss = isBoss;
        this.w = isBoss ? 75 : 50; 
        this.h = isBoss ? 150 : 100;
        this.pos = { x, y: 380 - this.h }; 
        this.vel = { x: 0, y: 0 };
        this.color = color;
        this.maxHp = isBoss ? 150 : 100;
        this.hp = this.maxHp;
        this.ce = 0;
        this.isBot = isBot;
        this.isAtk = false; this.isCharge = false;
        this.dir = isBot ? -1 : 1; this.onGrd = false; this.jumps = 0;
        this.isDomainCaster = false; this.isFloating = false;
    }
    draw() {
        ctx.save();
        if (this.isCharge || (domainInEffect && this.isDomainCaster)) {
            ctx.shadowBlur = 30; ctx.shadowColor = this.isBoss ? '#2eff7b' : (this.isDomainCaster ? '#bc00ff' : '#00d4ff');
            ctx.fillStyle = '#fff';
        } else { ctx.fillStyle = this.color; }
        ctx.fillRect(this.pos.x, this.pos.y, this.w, this.h);
        if (this.isAtk) {
            ctx.fillStyle = "white"; ctx.shadowBlur = 15; ctx.shadowColor = "white";
            ctx.fillRect(this.pos.x + (this.dir === 1 ? this.w : -40), this.pos.y + 20, 40, 15);
        }
        ctx.restore();
    }
    update() {
        if (this.isFloating) { this.pos.y = 80; this.vel.x = 0; this.vel.y = 0; }
        else if (!domainInEffect || (domainInEffect && this.isDomainCaster)) {
            this.pos.x += this.vel.x; this.pos.y += this.vel.y;
        }
        if (this.pos.y + this.h >= 380) { this.vel.y = 0; this.pos.y = 380 - this.h; this.onGrd = true; this.jumps = 0; }
        else if (!this.isFloating) { this.vel.y += 0.8; this.onGrd = false; }
        if (this.isCharge) { this.ce = Math.min(100, this.ce + 0.5); this.vel.x = 0; }
        this.draw();
    }
    jump() { if (this.jumps < 2 && !domainInEffect && !this.isFloating) { this.vel.y = -16; this.jumps++; } }
}

const p1 = new Fighter({ x: 100, y: 280, color: '#006eff' });
const bot = new Fighter({ x: 650, y: 280, color: '#ff2e2e', isBot: true });

function triggerHitEffect(power, flashOpacity = 0.4) {
    shake = power;
    const flash = document.getElementById('impact-flash');
    flash.style.opacity = flashOpacity;
    setTimeout(() => flash.style.opacity = 0, 50);
}

function useDomain(c, t) {
    const now = Date.now();
    const lastTime = c === p1 ? p1LastDomain : botLastDomain;
    
    if (c.ce < 100 || gameOver || domainInEffect || (now - lastTime < domainCooldown)) return;
    
    if(c === p1) p1LastDomain = now;
    else botLastDomain = now;

    c.ce = 0; domainInEffect = true; c.isDomainCaster = true;
    const ui = document.getElementById('domain-ui');
    const txt = document.getElementById('domain-text');
    const container = document.getElementById('game-container');
    
    ui.classList.add('show');
    if(c.isBoss) {
        container.classList.add('boss-domain-bg');
        txt.classList.add('boss-text');
    } else {
        container.classList.add('domain-active-bg');
    }
    
    triggerHitEffect(15, 0.8);
    c.isFloating = true;
    setTimeout(() => {
        let hits = 0;
        const interval = setInterval(() => {
            if (hits >= 30) { clearInterval(interval); endDomain(c); }
            else {
                t.hp = Math.max(0, t.hp - 2);
                triggerHitEffect(10, 0.2);
                slashes.push(new SlashEffect(t.pos.x, t.pos.y, t.w, t.h));
                slashes.push(new SlashEffect(t.pos.x, t.pos.y, t.w, t.h));
                hits++;
            }
        }, 100);
    }, 1500);
}

function endDomain(caster) {
    setTimeout(() => {
        domainInEffect = false; caster.isDomainCaster = false; caster.isFloating = false;
        document.getElementById('domain-ui').classList.remove('show');
        document.getElementById('game-container').className = '';
        document.getElementById('domain-text').className = '';
        triggerHitEffect(20, 1);
    }, 500);
}

function selectMode(mode) {
    gameMode = mode;
    document.getElementById('mode-menu').style.display = 'none';
    document.getElementById('menu').style.display = 'flex';
}

function startGame(level) {
    difficulty = level;
    document.getElementById('menu').style.display = 'none';
    if(gameMode === 'survival') {
        document.getElementById('stage-counter').style.display = 'block';
        updateStageUI();
    }
    gameActive = true;
    animate();
}

function updateStageUI() {
    document.getElementById('stage-counter').innerText = `STAGE ${stage} ${stage % 5 === 0 ? '- BOSS' : ''}`;
}

function spawnNextEnemy() {
    stage++; enemiesDefeated++;
    if(stage > maxStages) { showWinScreen(); return; }
    let isBoss = (stage % 5 === 0);
    bot.reset(650, 280, isBoss ? '#2eff7b' : '#ff2e2e', true, isBoss);
    if(isBoss) triggerHitEffect(20, 0.5);
    updateStageUI();
}

function showWinScreen() {
    gameOver = true;
    const screen = document.getElementById('ko-screen');
    screen.style.display = 'flex';
    document.getElementById('ko-text').innerText = "CONGRATULATIONS";
    document.getElementById('ko-text').style.color = "var(--green)";
}

// Controls
const keys = { a: false, d: false };
const bind = (id, s, e) => {
    const el = document.getElementById(id);
    el.addEventListener('touchstart', (ev) => { ev.preventDefault(); s(); });
    el.addEventListener('touchend', (ev) => { ev.preventDefault(); if(e) e(); });
};
bind('btn-left', () => {keys.a = true; p1.dir = -1;}, () => keys.a = false);
bind('btn-right', () => {keys.d = true; p1.dir = 1;}, () => keys.d = false);
bind('btn-up', () => p1.jump());
bind('btn-charge', () => p1.isCharge = true, () => p1.isCharge = false);
bind('btn-atk', () => { if(!domainInEffect){ p1.isAtk = true; setTimeout(()=>p1.isAtk=false, 150); } });
bind('btn-dom', () => useDomain(p1, bot));

function animate() {
    if (!gameActive) return;
    ctx.setTransform(1,0,0,1,0,0); ctx.clearRect(0,0,800,400);
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }
    
    // Update Cooldown UI
    const now = Date.now();
    const cdLeft = Math.max(0, Math.ceil((domainCooldown - (now - p1LastDomain)) / 1000));
    const deBtn = document.getElementById('btn-dom');
    const timerText = document.getElementById('de-timer');
    
    if (cdLeft > 0 && p1LastDomain !== 0) {
        deBtn.classList.add('btn-disabled');
        timerText.innerText = cdLeft + "s";
    } else {
        deBtn.classList.remove('btn-disabled');
        timerText.innerText = "";
    }

    if (!gameOver) {
        if (!domainInEffect) {
            p1.vel.x = keys.a ? -6 : (keys.d ? 6 : 0);
            const settings = diffSettings[difficulty];
            const dist = p1.pos.x - bot.pos.x;
            bot.dir = dist > 0 ? 1 : -1;
            if (Math.abs(dist) > (bot.isBoss ? 90 : 70)) {
                bot.vel.x = dist > 0 ? settings.speed : -settings.speed;
            } else {
                bot.vel.x = 0;
                if (Math.random() < settings.attackRate) { bot.isAtk = true; setTimeout(()=>bot.isAtk=false, 150); }
            }
            bot.ce = Math.min(100, bot.ce + settings.ceRate);
            if (bot.ce >= 100 && Math.random() < 0.01) useDomain(bot, p1);
        }

        if (!domainInEffect) {
            [p1, bot].forEach(atk => {
                const vic = atk === p1 ? bot : p1;
                const hX = atk.dir === 1 ? atk.pos.x + atk.w : atk.pos.x - 40;
                if (atk.isAtk && hX < vic.pos.x + vic.w && hX + 40 > vic.pos.x && atk.pos.y < vic.pos.y + vic.h && atk.pos.y + 50 > vic.pos.y) {
                    vic.hp -= 0.6; atk.ce = Math.min(100, atk.ce + 0.8);
                    if (Math.random() < 0.1) triggerHitEffect(5); 
                }
            });
        }

        if (p1.hp <= 0) {
            gameOver = true;
            document.getElementById('ko-screen').style.display = 'flex';
            document.getElementById('ko-text').innerText = "K.O. - DEFEAT";
        } else if (bot.hp <= 0) {
            if (gameMode === '1v1') {
                gameOver = true;
                document.getElementById('ko-screen').style.display = 'flex';
                document.getElementById('ko-text').innerText = "K.O. - VICTORY";
            } else {
                if(stage % 5 === 0) p1.hp = 100;
                spawnNextEnemy();
            }
        }
    }

    for (let i = slashes.length - 1; i >= 0; i--) { slashes[i].update(); slashes[i].draw(); if (slashes[i].life <= 0) slashes.splice(i, 1); }
    p1.update(); bot.update();
    document.getElementById('p1-hp').style.width = (p1.hp / p1.maxHp * 100) + "%";
    document.getElementById('p2-hp').style.width = (bot.hp / bot.maxHp * 100) + "%";
    document.getElementById('p1-ce').style.width = p1.ce + "%";
    document.getElementById('p2-ce').style.width = bot.ce + "%";
    requestAnimationFrame(animate);
}
</script>
</body>
</html>
