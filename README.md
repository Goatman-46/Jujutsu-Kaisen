<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PIXEL KAIZEN 4.0 | DEFINITIVE</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; --green: #2eff7b; }
        body { margin: 0; background: #050505; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        #game-container { position: relative; width: 800px; height: 400px; margin: 10px auto; border: 4px solid #333; overflow: hidden; background: #111; }
        
        /* Domain Borders */
        .border-active { border-color: var(--purple) !important; box-shadow: 0 0 20px var(--purple); }
        
        .hud { position: absolute; top: 15px; width: 100%; display: flex; justify-content: space-between; padding: 0 30px; box-sizing: border-box; z-index: 10; pointer-events: none;}
        .bar-bg { background: rgba(0,0,0,0.8); height: 18px; border: 2px solid #fff; width: 250px; }
        .hp-fill { background: #ff2e2e; height: 100%; width: 100%; transition: width 0.3s; }
        .ce-bg { background: #000; height: 6px; margin-top: 4px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; }

        #menu, #mode-menu, #custom-menu { position: absolute; inset: 0; background: rgba(0,0,0,0.95); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .menu-btn { margin: 8px; padding: 12px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 260px; text-transform: uppercase; }
        .menu-btn:disabled { opacity: 0.4; cursor: not-allowed; }

        #domain-ui { position: absolute; inset: 0; display: flex; align-items: center; justify-content: center; z-index: 100; pointer-events: none; opacity: 0; transition: 0.5s; }
        #domain-ui.active { opacity: 1; transform: scale(1.2); }

        .mobile-btns { display: flex; justify-content: space-between; width: 800px; margin: 5px auto; max-width: 95vw; }
        .btn { width: 60px; height: 60px; background: #222; border: 2px solid #444; color: white; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 12px; }
        #cd-timer { color: var(--red); font-weight: bold; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="mode-menu">
        <h1 style="color: var(--blue); font-style: italic;">PIXEL KAIZEN 4.0</h1>
        <button class="menu-btn" onclick="setMode('1v1')">1v1 Duel</button>
        <button class="menu-btn" onclick="setMode('survival')">Survival Gauntlet</button>
        <button class="menu-btn" style="border-color: var(--purple)" onclick="openCustom()">Customization</button>
    </div>

    <div id="custom-menu" style="display:none">
        <h2>DOMAIN TYPE</h2>
        <button class="menu-btn" id="dom-label" onclick="cycleDomain()">TYPE: SLASHES</button>
        <button class="menu-btn" onclick="closeCustom()">BACK</button>
    </div>

    <div id="domain-ui"><h1 id="domain-text" style="text-shadow: 0 0 20px #fff;">DOMAIN EXPANSION</h1></div>

    <div id="ko-screen" style="display:none; position:absolute; inset:0; background:rgba(0,0,0,0.9); z-index:400; flex-direction:column; align-items:center; justify-content:center;">
        <h1 id="ko-msg" style="font-size:50px;">K.O.</h1>
        <button class="menu-btn" onclick="location.reload()">RESTART</button>
    </div>

    <div class="hud">
        <div>
            <div class="bar-bg"><div id="p1-hp" class="hp-fill"></div></div>
            <div class="ce-bg"><div id="p1-ce" class="ce-fill"></div></div>
        </div>
        <div style="text-align: right;">
            <div class="bar-bg" style="margin-left: auto;"><div id="p2-hp" class="hp-fill"></div></div>
            <div class="ce-bg" style="margin-left: auto;"><div id="p2-ce" class="ce-fill"></div></div>
        </div>
    </div>
    <canvas id="gameCanvas" width="800" height="400"></canvas>
</div>

<div class="mobile-btns">
    <div style="display:flex; gap:10px;"><div class="btn" id="btn-left">◀</div><div class="btn" id="btn-right">▶</div><div class="btn" id="btn-charge">AURA</div></div>
    <div style="display:flex; gap:10px;"><div class="btn" id="btn-up">JUMP</div><div class="btn" id="btn-atk">PUNCH</div><div class="btn" id="btn-dom">DE <span id="cd-timer"></span></div></div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const domainTypes = ['slashes', 'burst', 'whiteout', 'rain', 'clone'];
let currentDomainIdx = 0;
let gameActive = false, gameMode = '1v1', domainActive = false, gameOver = false;
let stage = 1, shake = 0, lastDomainTime = 0, domainCooldown = 15000;
let clone = null, hitSparks = [], particles = [];

class Fighter {
    constructor(x, color, isBot = false) {
        this.isBot = isBot;
        this.color = color;
        this.reset(x);
    }
    reset(x) {
        this.pos = { x, y: 280 };
        this.vel = { x: 0, y: 0 };
        this.maxHp = (this.isBot && stage % 5 === 0) ? 200 : 100;
        this.hp = this.maxHp;
        this.ce = 0;
        this.dir = this.pos.x > 400 ? -1 : 1;
        this.isAtk = false; this.atkFrame = 0;
        this.isCharge = false; this.jumps = 0;
        this.anim = 0;
    }
    update() {
        this.pos.x += this.vel.x;
        this.pos.y += this.vel.y;
        if (this.pos.y > 280) { this.pos.y = 280; this.vel.y = 0; this.jumps = 0; }
        else { this.vel.y += 0.8; }
        if (this.isCharge) this.ce = Math.min(100, this.ce + 0.3); // Slower charging
        if (this.isAtk) {
            this.atkFrame += 0.15;
            if (this.atkFrame >= 1) { this.isAtk = false; this.atkFrame = 0; }
        }
        this.anim += 0.1;
    }
    draw(context) {
        context.save();
        context.translate(this.pos.x + 20, this.pos.y + 45);
        
        if (this.isCharge) {
            context.shadowBlur = 15; context.shadowColor = this.color;
            context.fillStyle = this.color;
            context.globalAlpha = 0.3;
            context.beginPath();
            context.moveTo(-25, 40); context.lineTo(0, -70); context.lineTo(25, 40);
            context.fill();
            context.globalAlpha = 1.0;
        }

        context.scale(this.dir, 1);
        context.strokeStyle = this.color;
        context.lineWidth = 4;
        context.lineCap = 'round';

        let ext = this.isAtk ? Math.sin(this.atkFrame * Math.PI) * 35 : 0;
        let walk = (Math.abs(this.vel.x) > 0.1) ? Math.sin(this.anim * 5) * 15 : 0;
        let jumpY = (this.pos.y < 280) ? 10 : 0;

        // Head
        context.beginPath(); context.arc(0, -35, 10, 0, Math.PI*2); context.stroke();
        // Body
        context.beginPath(); context.moveTo(0, -25); context.lineTo(0, 10); context.stroke();
        // Arms
        context.beginPath(); context.moveTo(0, -15); context.lineTo(10 + ext, 5); context.stroke(); // Front
        context.beginPath(); context.moveTo(0, -15); context.lineTo(-15, 10); context.stroke(); // Back
        // Legs
        context.beginPath(); context.moveTo(0, 10); context.lineTo(10 + walk, 35 - jumpY); context.stroke();
        context.beginPath(); context.moveTo(0, 10); context.lineTo(-10 - walk, 35 - jumpY); context.stroke();
        
        context.restore();
    }
}

const p1 = new Fighter(100, '#00d4ff');
let bot = new Fighter(650, '#ff2e2e', true);

function setMode(m) { gameMode = m; document.getElementById('mode-menu').style.display='none'; startGame(); }
function openCustom() { document.getElementById('mode-menu').style.display='none'; document.getElementById('custom-menu').style.display='flex'; }
function closeCustom() { document.getElementById('custom-menu').style.display='none'; document.getElementById('mode-menu').style.display='flex'; }
function cycleDomain() { currentDomainIdx = (currentDomainIdx + 1) % domainTypes.length; document.getElementById('dom-label').innerText = "TYPE: " + domainTypes[currentDomainIdx].toUpperCase(); }

function startGame() { gameActive = true; loop(); }

function useDomain(caster, target) {
    let now = Date.now();
    if (caster.ce < 100 || domainActive || (caster === p1 && now - lastDomainTime < domainCooldown)) return;
    
    if(caster === p1) lastDomainTime = now;
    caster.ce = 0;
    domainActive = true;
    const type = caster.isBot ? domainTypes[Math.floor(Math.random()*domainTypes.length)] : domainTypes[currentDomainIdx];
    
    const ui = document.getElementById('domain-ui');
    ui.classList.add('active');
    document.getElementById('domain-text').innerText = type.toUpperCase();
    document.getElementById('game-container').classList.add('border-active');

    setTimeout(() => {
        if (type === 'slashes') {
            let hits = 0;
            let itv = setInterval(() => {
                target.hp -= 2; createHit(target.pos.x+20, target.pos.y+40); shake = 5;
                if (++hits > 25) { clearInterval(itv); endDomain(); }
            }, 100);
        } else if (type === 'clone') {
            clone = new Fighter(caster.pos.x, 'rgba(0, 212, 255, 0.5)');
            clone.hp = 50;
            endDomain();
        } else if (type === 'whiteout') {
            shake = 20; target.hp -= 40;
            setTimeout(endDomain, 1000);
        } else if (type === 'rain') {
            let drops = 0;
            let itv = setInterval(() => {
                particles.push({x: Math.random()*800, y: 0});
                target.hp -= 0.8;
                if (++drops > 50) { clearInterval(itv); endDomain(); particles=[]; }
            }, 60);
        } else if (type === 'burst') {
            target.hp -= 45; shake = 30;
            setTimeout(endDomain, 1000);
        }
    }, 1000);
}

function endDomain() {
    domainActive = false;
    document.getElementById('domain-ui').classList.remove('active');
    document.getElementById('game-container').classList.remove('border-active');
}

function createHit(x, y) { hitSparks.push({x, y, t: 10}); }

function loop() {
    if (!gameActive) return;
    ctx.setTransform(1,0,0,1,0,0);
    ctx.clearRect(0, 0, 800, 400);

    // --- Dynamic Camera Logic ---
    let centerX = (p1.pos.x + bot.pos.x) / 2;
    let dist = Math.abs(p1.pos.x - bot.pos.x);
    let scale = Math.max(0.7, Math.min(1.2, 800 / (dist + 300)));
    
    ctx.translate(400, 200);
    ctx.scale(scale, scale);
    ctx.translate(-centerX - 20, -230);
    
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }

    // --- Logic ---
    if (!gameOver) {
        // P1 Movement
        p1.vel.x = keys.a ? -5 : (keys.d ? 5 : 0);
        if (keys.a) p1.dir = -1; if (keys.d) p1.dir = 1;

        // Bot AI
        if (!domainActive) {
            let d = p1.pos.x - bot.pos.x;
            bot.vel.x = Math.abs(d) > 70 ? (d > 0 ? 3 : -3) : 0;
            bot.dir = d > 0 ? 1 : -1;
            if (Math.random() < 0.02 && !bot.isAtk) bot.isAtk = true;
            if (bot.ce >= 100 && Math.random() < 0.01) useDomain(bot, p1);
            bot.ce += 0.1;
        }

        // Clone AI
        if (clone) {
            let d = bot.pos.x - clone.pos.x;
            clone.vel.x = d > 0 ? 2 : -2;
            if (Math.abs(d) < 60 && Math.random() < 0.05) clone.isAtk = true;
            if (clone.isAtk && Math.abs(bot.pos.x - clone.pos.x) < 60) { bot.hp -= 0.4; createHit(bot.pos.x+20, bot.pos.y+40); }
            clone.update(); clone.draw(ctx);
        }

        // Damage Detection (Punches)
        [p1, bot].forEach(attacker => {
            let victim = (attacker === p1) ? bot : p1;
            if (attacker.isAtk && attacker.atkFrame > 0.3 && attacker.atkFrame < 0.7) {
                let hitX = attacker.pos.x + (attacker.dir === 1 ? 40 : -20);
                if (Math.abs(hitX - victim.pos.x) < 40 && Math.abs(attacker.pos.y - victim.pos.y) < 50) {
                    victim.hp -= 1.5;
                    createHit(victim.pos.x+20, victim.pos.y+40);
                    shake = 3;
                }
            }
        });

        // Survival Logic
        if (bot.hp <= 0) {
            if (gameMode === 'survival') {
                stage++;
                bot.reset(650);
                p1.hp = Math.min(100, p1.hp + 30);
            } else {
                gameOver = true; document.getElementById('ko-screen').style.display='flex';
            }
        }
        if (p1.hp <= 0) { gameOver = true; document.getElementById('ko-screen').style.display='flex'; }
    }

    // --- Render ---
    p1.update(); p1.draw(ctx);
    bot.update(); bot.draw(ctx);
    
    particles.forEach(p => { 
        ctx.fillStyle = 'cyan'; p.y += 15; ctx.fillRect(p.x, p.y, 2, 20); 
    });

    hitSparks.forEach((s, i) => {
        ctx.strokeStyle = 'white'; ctx.beginPath();
        ctx.moveTo(s.x-5, s.y-5); ctx.lineTo(s.x+5, s.y+5); ctx.stroke();
        s.t--; if(s.t <= 0) hitSparks.splice(i, 1);
    });

    // --- HUD ---
    document.getElementById('p1-hp').style.width = p1.hp + "%";
    document.getElementById('p2-hp').style.width = (bot.hp / bot.maxHp * 100) + "%";
    document.getElementById('p1-ce').style.width = p1.ce + "%";
    document.getElementById('p2-ce').style.width = bot.ce + "%";
    
    let cd = Math.max(0, Math.ceil((domainCooldown - (Date.now() - lastDomainTime))/1000));
    document.getElementById('cd-timer').innerText = cd > 0 && lastDomainTime !== 0 ? cd + "s" : "";
    document.getElementById('btn-dom').style.opacity = (p1.ce >= 100 && cd === 0) ? "1" : "0.5";

    requestAnimationFrame(loop);
}

const keys = { a: false, d: false };
const bind = (id, s, e) => {
    const el = document.getElementById(id);
    el.onmousedown = el.ontouchstart = (ev) => { ev.preventDefault(); s(); };
    el.onmouseup = el.onmouseleave = el.ontouchend = (ev) => { ev.preventDefault(); if(e) e(); };
};
bind('btn-left', () => keys.a = true, () => keys.a = false);
bind('btn-right', () => keys.d = true, () => keys.d = false);
bind('btn-up', () => { if(p1.jumps < 2) { p1.vel.y = -15; p1.jumps++; } });
bind('btn-charge', () => p1.isCharge = true, () => p1.isCharge = false);
bind('btn-atk', () => { if(!p1.isAtk) p1.isAtk = true; });
bind('btn-dom', () => useDomain(p1, bot));
</script>
</body>
</html>
