    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PIXEL KAIZEN | DOMAIN OVERHAUL</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; }
        body { margin: 0; background: #050505; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        
        #game-container { position: relative; width: 800px; height: 400px; margin: 10px auto; outline: 4px solid #222; overflow: hidden; background: #111; transition: background 0.5s; }
        
        /* Domain Warp Class */
        .domain-active-bg { background: radial-gradient(circle, #1a0026 0%, #000 100%) !important; outline-color: var(--purple) !important; }

        #impact-flash { position: absolute; inset: 0; background: white; opacity: 0; z-index: 101; pointer-events: none; }
        
        /* Domain UI Animation */
        #domain-ui { position: absolute; inset: 0; opacity: 0; display: flex; align-items: center; justify-content: center; z-index: 100; pointer-events: none; }
        #domain-ui h1 { font-size: 60px; color: white; text-shadow: 0 0 30px var(--purple), 0 0 60px var(--purple); font-style: italic; transform: scale(0.5); transition: 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        #domain-ui.show { opacity: 1; }
        #domain-ui.show h1 { transform: scale(1.2); }

        .hud { position: absolute; top: 15px; width: 100%; display: flex; justify-content: space-between; padding: 0 30px; box-sizing: border-box; z-index: 10; }
        .stat-group { width: 40%; }
        .bar-bg { background: #000; height: 20px; border: 2px solid #fff; }
        .hp-fill { background: linear-gradient(90deg, var(--red), #ff7b7b); height: 100%; width: 100%; transition: width 0.1s; }
        .ce-bg { background: #000; height: 6px; margin-top: 4px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; box-shadow: 0 0 8px var(--blue); }

        #menu { position: absolute; inset: 0; background: rgba(0,0,0,0.9); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .diff-btn { margin: 10px; padding: 10px 30px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 200px; }
        
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
let domainInEffect = false;

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
        this.jumps = 0;
        this.isDomainCaster = false;
    }

    draw() {
        ctx.save();
        if (this.isCharge || (domainInEffect && this.isDomainCaster)) { 
            ctx.shadowBlur = 30; 
            ctx.shadowColor = this.isDomainCaster ? '#bc00ff' : '#00d4ff'; 
            ctx.fillStyle = '#fff'; 
        } else { 
            ctx.fillStyle = this.color; 
        }
        
        ctx.fillRect(this.pos.x, this.pos.y, this.w, this.h);
        
        if (this.isAtk) {
            ctx.fillStyle = "white";
            ctx.shadowBlur = 15; ctx.shadowColor = "white";
            ctx.fillRect(this.pos.x + (this.dir === 1 ? 50 : -40), this.pos.y + 20, 40, 15);
        }
        ctx.restore();
    }

    update() {
        if (!domainInEffect || (domainInEffect && this.isDomainCaster)) {
            this.pos.x += this.vel.x; 
            this.pos.y += this.vel.y;
        }

        if (this.pos.y + this.h >= 380) { 
            this.vel.y = 0; this.pos.y = 280; this.onGrd = true; this.jumps = 0;
        } else { 
            this.vel.y += 0.8; this.onGrd = false; 
        }
        
        if (this.isCharge) { this.ce = Math.min(100, this.ce + 0.5); this.vel.x = 0; }
        this.draw();
    }

    jump() {
        if (this.jumps < 2 && !domainInEffect) {
            this.vel.y = -16;
            this.jumps++;
        }
    }
}

const p1 = new Fighter({ x: 100, y: 0, color: '#006eff' });
const bot = new Fighter({ x: 650, y: 0, color: '#ff2e2e', isBot: true });

function triggerHitEffect(power, flashOpacity = 0.4) {
    shake = power;
    const flash = document.getElementById('impact-flash');
    flash.style.opacity = flashOpacity;
    setTimeout(() => flash.style.opacity = 0, 50);
}

function useDomain(c, t) {
    if (c.ce < 100 || gameOver || domainInEffect) return;
    
    c.ce = 0; 
    domainInEffect = true;
    c.isDomainCaster = true;
    
    const ui = document.getElementById('domain-ui');
    const container = document.getElementById('game-container');
    
    // Phase 1: Casting
    ui.classList.add('show');
    container.classList.add('domain-active-bg');
    triggerHitEffect(15, 0.8);

    // Phase 2: The Sure-Hit Sequence
    setTimeout(() => {
        let hits = 0;
        const interval = setInterval(() => {
            t.hp = Math.max(0, t.hp - 5);
            triggerHitEffect(10, 0.2);
            hits++;
            if (hits >= 10) { // Total 50 damage over 10 quick hits
                clearInterval(interval);
                endDomain(c);
            }
        }, 100);
    }, 1500);
}

function endDomain(caster) {
    setTimeout(() => {
        domainInEffect = false;
        caster.isDomainCaster = false;
        document.getElementById('domain-ui').classList.remove('show');
        document.getElementById('game-container').classList.remove('domain-active-bg');
        triggerHitEffect(20, 1); // Final impact flash
    }, 500);
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
bind('btn-up', () => p1.jump());
bind('btn-charge', () => p1.isCharge = true, () => p1.isCharge = false);
bind('btn-atk', () => { if(!domainInEffect){ p1.isAtk = true; setTimeout(()=>p1.isAtk=false, 150); } });
bind('btn-dom', () => useDomain(p1, bot));

function animate() {
    if (!gameActive) return;
    ctx.setTransform(1,0,0,1,0,0); ctx.clearRect(0,0,800,400);
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }
    
    if (!gameOver) {
        if (!domainInEffect) {
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
        }

        // Standard Hit Detection (only when Domain is NOT active)
        if (!domainInEffect) {
            [p1, bot].forEach(atk => {
                const vic = atk === p1 ? bot : p1;
                const hX = atk.dir === 1 ? atk.pos.x + 50 : atk.pos.x - 40;
                if (atk.isAtk && hX < vic.pos.x + 50 && hX + 40 > vic.pos.x && atk.pos.y < vic.pos.y + 100 && atk.pos.y + 50 > vic.pos.y) {
                    vic.hp -= 0.6; atk.ce = Math.min(100, atk.ce + 0.8);
                    if (Math.random() < 0.1) triggerHitEffect(5); 
                }
            });
        }

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
