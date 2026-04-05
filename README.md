<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PIXEL KAIZEN | ULTRA EDITION</title>
    <style>
        :root { --blue: #00d4ff; --red: #ff2e2e; --purple: #bc00ff; }
        body { margin: 0; background: #050505; color: white; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        
        /* Game Container & Effects */
        #game-container { position: relative; width: 800px; height: 400px; margin: 10px auto; outline: 4px solid #222; overflow: hidden; background: #111; }
        #impact-flash { position: absolute; inset: 0; background: white; opacity: 0; z-index: 101; pointer-events: none; }
        #domain-ui { position: absolute; inset: 0; background: black; opacity: 0; display: flex; align-items: center; justify-content: center; z-index: 100; pointer-events: none; transition: opacity 0.2s; }
        #domain-ui h1 { font-size: 50px; color: var(--purple); text-shadow: 0 0 20px var(--purple); font-style: italic; }

        /* HUD */
        .hud { position: absolute; top: 15px; width: 100%; display: flex; justify-content: space-between; padding: 0 30px; box-sizing: border-box; z-index: 10; }
        .stat-group { width: 40%; }
        .bar-bg { background: #000; height: 20px; border: 2px solid #fff; }
        .hp-fill { background: linear-gradient(90deg, var(--red), #ff7b7b); height: 100%; width: 100%; transition: width 0.1s; }
        .ce-bg { background: #000; height: 6px; margin-top: 4px; border: 1px solid var(--blue); }
        .ce-fill { background: var(--blue); height: 100%; width: 0%; box-shadow: 0 0 8px var(--blue); }

        /* Difficulty Selector */
        #menu { position: absolute; inset: 0; background: rgba(0,0,0,0.9); z-index: 300; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .diff-btn { margin: 10px; padding: 10px 30px; border: 2px solid white; background: none; color: white; font-family: inherit; cursor: pointer; width: 200px; }
        .diff-btn:hover { background: white; color: black; }

        /* K.O. Screen */
        #ko-screen { position: absolute; inset: 0; background: rgba(0,0,0,0.85); display: none; flex-direction: column; align-items: center; justify-content: center; z-index: 200; }
        #ko-screen h1 { font-size: 80px; color: var(--red); text-shadow: 0 0 20px var(--red); }

        /* Controls */
        .mobile-btns { display: flex; justify-content: space-between; width: 800px; margin: 5px auto; max-width: 95vw; }
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
        <div class="stat-group">
            <div class="bar-bg"><div id="p2-hp" class="hp-fill" style="float:right"></div></div>
            <div class="ce-bg"><div id="p2-ce" class="ce-fill" style="float:right"></div></div>
        </div>
    </div>
    <canvas id="gameCanvas" width="800" height="400"></canvas>
</div>

<div class="mobile-btns">
    <div style="display:flex; gap:8px">
        <div class="btn" id="btn-left">◀</div>
        <div class="btn" id="btn-right">▶</div>
        <div class="btn" id="btn-up">UP</div>
    </div>
    <div style="display:flex; gap:8px">
        <div class="btn btn-special" id="btn-charge">CE</div>
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
        this.dir = isBot ? -1 : 1; this.onGrd = false;
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
        if (this.pos.y + this.h >= 380) { this.vel.y = 0; this.pos.y = 280; this.onGrd = true; }
        else { this.vel.y += 0.8; this.onGrd = false; }
        if (this.isCharge) { this.ce = Math.min(100, this.ce + 0.5); this.vel.x = 0; }
        this.draw();
    }
}

const p1 = new Fighter({ x: 100, y: 0, color: '#006eff' });
const bot = new Fighter({ x: 650, y: 0, color: '#ff2e2e', isBot: true });

function triggerHitEffect(power) {
    shake = power;
    const flash = document.getElementById('impact-flash');
    flash.style.opacity = 0.6;
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

// Controls
const keys = { a: false, d: false };
const bind = (id, s, e) => {
    const el = document.getElementById(id);
    el.addEventListener('touchstart', (ev) => { ev.preventDefault(); s(); });
    el.addEventListener('touchend', (ev) => { ev.preventDefault(); if(e) e(); });
};
bind('btn-left', () => {keys.a = true; p1.dir = -1;}, () => keys.a = false);
bind('btn-right', () => {keys.d = true; p1.dir = 1;}, () => keys.d = false);
bind('btn-up', () => { if(p1.onGrd) p1.vel.y = -18; });
bind('btn-charge', () => p1.isCharge = true, () => p1.isCharge = false);
bind('btn-atk', () => { p1.isAtk = true; setTimeout(()=>p1.isAtk=false, 150); });
bind('btn-dom', () => useDomain(p1, bot));

function animate() {
    if (!gameActive) return;
    ctx.setTransform(1,0,0,1,0,0); ctx.clearRect(0,0,800,400);
    if (shake > 0) { ctx.translate(Math.random()*shake-shake/2, Math.random()*shake-shake/2); shake *= 0.9; }
    
    if (!gameOver) {
        // P1 Movement
        p1.vel.x = keys.a ? -6 : (keys.d ? 6 : 0);
        
        // Bot AI
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

        // Hit Detection
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
</html>        }
        .ui {
            position: absolute;
            top: 10px;
            width: 100%;
            display: flex;
            justify-content: space-between;
            padding: 0 20px;
            box-sizing: border-box;
            pointer-events: none;
        }
        .health-bar-container {
            width: 40%;
        }
        .health-bg {
            background: red;
            height: 20px;
            width: 100%;
            border: 2px solid white;
        }
        .health-fill {
            background: #4caf50;
            height: 100%;
            width: 100%;
            transition: width 0.2s;
        }
        .energy-bg {
            background: #555;
            height: 10px;
            width: 100%;
            margin-top: 5px;
            border: 1px solid white;
        }
        .energy-fill {
            background: #00bcd4; /* Cursed energy color */
            height: 100%;
            width: 0%;
        }
        #p1-name, #p2-name {
            font-weight: bold;
            margin-bottom: 5px;
            text-transform: uppercase;
        }
        
        /* Touch Controls */
        #touch-controls {
            display: flex;
            justify-content: space-between;
            width: 800px;
            max-width: 100vw;
            margin-top: 10px;
            user-select: none;
        }
        .d-pad, .action-buttons {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
            width: 45%;
        }
        .action-buttons {
            justify-content: flex-end;
        }
        .btn {
            background: rgba(255, 255, 255, 0.2);
            border: 2px solid white;
            color: white;
            padding: 15px 20px;
            font-size: 16px;
            border-radius: 8px;
            cursor: pointer;
        }
        .btn:active {
            background: rgba(255, 255, 255, 0.5);
        }
        
        /* Domain Expansion Flash */
        #domain-flash {
            position: absolute;
            top: 0; left: 0; right: 0; bottom: 0;
            background: black;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.5s;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <div class="ui">
            <div class="health-bar-container">
                <div id="p1-name">You (Player 1)</div>
                <div class="health-bg"><div id="p1-health" class="health-fill"></div></div>
                <div class="energy-bg"><div id="p1-energy" class="energy-fill"></div></div>
            </div>
            <div class="health-bar-container" style="text-align: right;">
                <div id="p2-name">Curse AI (Player 2)</div>
                <div class="health-bg" style="display: flex; justify-content: flex-end;">
                    <div id="p2-health" class="health-fill"></div>
                </div>
                <div class="energy-bg" style="display: flex; justify-content: flex-end;">
                    <div id="p2-energy" class="energy-fill"></div>
                </div>
            </div>
        </div>
        <canvas id="gameCanvas" width="800" height="400"></canvas>
        <div id="domain-flash"></div>
    </div>

    <div id="touch-controls">
        <div class="d-pad">
            <button class="btn" id="btn-left">◀</button>
            <button class="btn" id="btn-right">▶</button>
            <button class="btn" id="btn-jump">Jump</button>
        </div>
        <div class="action-buttons">
            <button class="btn" id="btn-block">Block</button>
            <button class="btn" id="btn-attack">Attack</button>
            <button class="btn" id="btn-charge" style="background: #00bcd4; color: black;">Charge</button>
            <button class="btn" id="btn-domain" style="background: purple; border-color: white;">Domain</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        const gravity = 0.6;
        let gameOver = false;
        let domainActive = false;

        // Input Tracking
        const keys = {
            a: { pressed: false },
            d: { pressed: false },
            w: { pressed: false },
            s: { pressed: false }, // Block
            c: { pressed: false }  // Charge
        };

        // Fighter Class
        class Fighter {
            constructor({ position, velocity, color, isAI = false }) {
                this.position = position;
                this.velocity = velocity;
                this.width = 50;
                this.height = 100;
                this.color = color;
                this.isAI = isAI;
                
                this.health = 500;
                this.energy = 0; // Cursed Energy (Max 100)
                
                this.isAttacking = false;
                this.isBlocking = false;
                this.isCharging = false;
                this.attackBox = {
                    position: { x: this.position.x, y: this.position.y },
                    width: 80,
                    height: 50
                };
                this.facingRight = true;
            }

            draw() {
                // Draw Character Body
                ctx.fillStyle = this.isBlocking ? 'gray' : this.color;
                
                // Pulsate color if charging cursed energy
                if (this.isCharging && this.energy < 100) {
                    ctx.fillStyle = Math.random() > 0.5 ? '#00bcd4' : this.color;
                }

                ctx.fillRect(this.position.x, this.position.y, this.width, this.height);

                // Draw Attack Hitbox (Debug/Visuals)
                if (this.isAttacking) {
                    ctx.fillStyle = 'rgba(255, 255, 255, 0.5)';
                    ctx.fillRect(this.attackBox.position.x, this.attackBox.position.y, this.attackBox.width, this.attackBox.height);
                }
            }

            update() {
                this.draw();

                // Update Attack Box position based on facing direction
                this.attackBox.position.x = this.facingRight ? this.position.x + this.width : this.position.x - this.attackBox.width;
                this.attackBox.position.y = this.position.y + 10;

                // Move X
                this.position.x += this.velocity.x;

                // Boundaries
                if (this.position.x < 0) this.position.x = 0;
                if (this.position.x + this.width > canvas.width) this.position.x = canvas.width - this.width;

                // Move Y (Gravity)
                this.position.y += this.velocity.y;
                if (this.position.y + this.height + this.velocity.y >= canvas.height - 20) {
                    this.velocity.y = 0;
                    this.position.y = canvas.height - this.height - 20;
                } else {
                    this.velocity.y += gravity;
                }

                // Manual Charging Mechanic
                if (this.isCharging && this.velocity.y === 0 && !this.isAttacking) {
                    this.velocity.x = 0; // Lock movement while charging
                    if (this.energy < 100) {
                        this.energy += 0.5; // Gain half a point per frame
                    }
                }
            }

            attack() {
                if (this.isAttacking || this.isCharging) return;
                this.isAttacking = true;
                setTimeout(() => { this.isAttacking = false; }, 200); // 200ms active frames
            }

            domainExpansion() {
                if (this.energy >= 100) {
                    this.energy = 0;
                    triggerDomainExpansion(this);
                }
            }
        }

        // Initialize Players
        const player = new Fighter({
            position: { x: 150, y: 0 },
            velocity: { x: 0, y: 0 },
            color: '#3498db' // Blue Sorcerer
        });

        const enemy = new Fighter({
            position: { x: 600, y: 0 },
            velocity: { x: 0, y: 0 },
            color: '#e74c3c', // Red Curse
            isAI: true
        });

        // Keydown Events (PC Controls)
        window.addEventListener('keydown', (e) => {
            if (gameOver) return;
            switch (e.key.toLowerCase()) {
                case 'd': keys.d.pressed = true; player.facingRight = true; break;
                case 'a': keys.a.pressed = true; player.facingRight = false; break;
                case 'w': if (player.velocity.y === 0) player.velocity.y = -12; break;
                case 's': player.isBlocking = true; break;
                case 'j': player.attack(); break;
                case 'c': player.isCharging = true; break; // CHARGE BUTTON
                case 'l': player.domainExpansion(); break; // ULTIMATE BUTTON
            }
        });

        window.addEventListener('keyup', (e) => {
            switch (e.key.toLowerCase()) {
                case 'd': keys.d.pressed = false; break;
                case 'a': keys.a.pressed = false; break;
                case 's': player.isBlocking = false; break;
                case 'c': player.isCharging = false; break;
            }
        });

        // --- Touch Controls Setup ---
        const bindTouch = (id, onDown, onUp) => {
            const btn = document.getElementById(id);
            btn.addEventListener('touchstart', (e) => { e.preventDefault(); onDown(); });
            btn.addEventListener('touchend', (e) => { e.preventDefault(); onUp(); });
        };

        bindTouch('btn-left', () => { keys.a.pressed = true; player.facingRight = false; }, () => keys.a.pressed = false);
        bindTouch('btn-right', () => { keys.d.pressed = true; player.facingRight = true; }, () => keys.d.pressed = false);
        bindTouch('btn-jump', () => { if (player.velocity.y === 0) player.velocity.y = -12; }, () => {});
        bindTouch('btn-block', () => player.isBlocking = true, () => player.isBlocking = false);
        bindTouch('btn-attack', () => player.attack(), () => {});
        bindTouch('btn-charge', () => player.isCharging = true, () => player.isCharging = false);
        bindTouch('btn-domain', () => player.domainExpansion(), () => {});

        // Hit Detection Logic
        function rectangularCollision({ rect1, rect2 }) {
            return (
                rect1.attackBox.position.x < rect2.position.x + rect2.width &&
                rect1.attackBox.position.x + rect1.attackBox.width > rect2.position.x &&
                rect1.attackBox.position.y < rect2.position.y + rect2.height &&
                rect1.attackBox.position.y + rect1.attackBox.height > rect2.position.y
            );
        }

        // Domain Expansion Visuals & Damage
        function triggerDomainExpansion(caster) {
            domainActive = true;
            const flash = document.getElementById('domain-flash');
            
            // Background turns black
            flash.style.opacity = '1';
            canvas.style.backgroundColor = "black";
            
            setTimeout(() => {
                // Sure-hit damage
                const target = caster === player ? enemy : player;
                target.health -= 40; // Massive damage
                if (target.health < 0) target.health = 0;
                
                flash.style.opacity = '0';
                canvas.style.backgroundColor = "#333";
                domainActive = false;
            }, 1500);
        }

        // AI Logic Function
        function handleAI() {
            if (enemy.isAttacking || gameOver || domainActive) return;

            const distanceX = player.position.x - enemy.position.x;
            
            // Face the player
            enemy.facingRight = distanceX > 0;

            // Move towards player
            if (Math.abs(distanceX) > 70) {
                enemy.velocity.x = enemy.facingRight ? 3 : -3;
                enemy.isBlocking = false;
            } else {
                enemy.velocity.x = 0;
                
                // Randomly Decide Action when close
                const rand = Math.random();
                
                if (player.isAttacking && rand < 0.6) {
                    enemy.isBlocking = true; // 60% chance to block if player attacks
                } else if (rand < 0.05) { // 5% chance per frame to attack
                    enemy.isBlocking = false;
                    enemy.attack();
                } else {
                    enemy.isBlocking = false;
                }
            }

            // AI occasionally jumps
            if (Math.random() < 0.01 && enemy.velocity.y === 0) {
                enemy.velocity.y = -12;
            }
        }

        // Game Loop
        function animate() {
            window.requestAnimationFrame(animate);
            if (domainActive) {
                // Clear without drawing characters during domain cinematic
                ctx.fillStyle = 'black';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                return;
            }

            // Draw Background (Floor)
            ctx.fillStyle = '#333';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#111';
            ctx.fillRect(0, canvas.height - 20, canvas.width, 20);

            // Update Players
            player.update();
            enemy.update();

            // Handle Player 1 Movement (only if not charging)
            player.velocity.x = 0;
            if (!player.isCharging) {
                if (keys.a.pressed && player.position.x > 0) player.velocity.x = -5;
                else if (keys.d.pressed && player.position.x < canvas.width - player.width) player.velocity.x = 5;
            }

            // Process AI
            handleAI();

            // Collision: Player 1 hits AI
            if (player.isAttacking && rectangularCollision({ rect1: player, rect2: enemy })) {
                player.isAttacking = false; // hit lands once
                if (!enemy.isBlocking) {
                    enemy.health -= 10;
                    player.energy += 15; // Gain energy on hit
                }
            }

            // Collision: AI hits Player 1
            if (enemy.isAttacking && rectangularCollision({ rect1: enemy, rect2: player })) {
                enemy.isAttacking = false;
                if (!player.isBlocking) {
                    player.health -= 10;
                    enemy.energy += 15;
                    player.energy += 5; // Gain minor energy on taking damage
                }
            }

            // Update UI Elements
            if (player.energy > 100) player.energy = 100;
            if (enemy.energy > 100) enemy.energy = 100;

            document.getElementById('p1-health').style.width = player.health + '%';
            document.getElementById('p1-energy').style.width = player.energy + '%';
            document.getElementById('p2-health').style.width = enemy.health + '%';
            document.getElementById('p2-energy').style.width = enemy.energy + '%';

            // Win Condition
            if (player.health <= 0 || enemy.health <= 0) {
                gameOver = true;
                ctx.fillStyle = 'rgba(0,0,0,0.7)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = 'white';
                ctx.font = '40px Courier New';
                ctx.fillText(player.health <= 0 ? 'AI WINS!' : 'PLAYER 1 WINS!', canvas.width/2 - 120, canvas.height/2);
            }
        }

        animate();
    </script>
</body>
</html>
