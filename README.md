# Game
Seruuu
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game 2D Platformer Browser</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Inter:wght@400;600;700&display=swap');
        
        .font-pixel {
            font-family: 'Press Start 2P', cursive;
        }

        /* Prevent scrolling on touch controls */
        .no-select {
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            -khtml-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
            touch-action: manipulation;
        }

        canvas {
            image-rendering: pixelated;
            image-rendering: crisp-edges;
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 font-sans min-h-screen flex flex-col items-center justify-between p-2 sm:p-6 no-select">

    <!-- Header Section -->
    <header class="text-center my-3">
        <h1 class="text-2xl sm:text-4xl font-pixel text-amber-400 drop-shadow-[0_4px_0_rgba(0,0,0,0.8)] tracking-wider mb-2">
            SUPER DASH 2D
        </h1>
        <p class="text-slate-400 text-xs sm:text-sm">Kumpulkan koin emas dan hindari jebakan duri merah!</p>
    </header>

    <!-- Main Game Container -->
    <main class="w-full max-w-4xl flex flex-col items-center">
        <!-- Canvas Wrapper -->
        <div class="relative w-full aspect-[16/9] bg-slate-950 rounded-xl overflow-hidden border-4 border-slate-700 shadow-2xl">
            <canvas id="gameCanvas" class="w-full h-full block bg-slate-950"></canvas>

            <!-- Touch Controls Overlay (Desktop will hide automatically or allow both) -->
            <div id="touchControls" class="absolute bottom-2 left-2 right-2 flex justify-between items-center pointer-events-none md:hidden opacity-80">
                <div class="flex gap-2 pointer-events-auto">
                    <button id="btnLeft" class="w-14 h-14 bg-slate-800/80 active:bg-amber-500 border-2 border-slate-600 rounded-lg text-white font-pixel text-xl flex items-center justify-center shadow-lg active:scale-95 transition-transform">◀</button>
                    <button id="btnRight" class="w-14 h-14 bg-slate-800/80 active:bg-amber-500 border-2 border-slate-600 rounded-lg text-white font-pixel text-xl flex items-center justify-center shadow-lg active:scale-95 transition-transform">▶</button>
                </div>
                <div class="pointer-events-auto">
                    <button id="btnJump" class="w-16 h-16 bg-emerald-600/80 active:bg-emerald-400 border-2 border-emerald-400 rounded-full text-white font-pixel text-xs flex items-center justify-center shadow-lg active:scale-95 transition-transform">LOMPAT</button>
                </div>
            </div>
        </div>

        <!-- Controls Guide (Desktop) -->
        <div class="hidden md:flex justify-center gap-6 mt-3 text-xs text-slate-400 bg-slate-800/60 px-4 py-2 rounded-lg border border-slate-700">
            <span><strong>Gerak:</strong> A / D atau ← / →</span>
            <span><strong>Lompat:</strong> W / Space / ↑</span>
            <span><strong>Mulai/Restart:</strong> R</span>
        </div>
    </main>

    <!-- Explanation and Tutorial Section -->
    <section class="w-full max-w-4xl mt-8 mb-4 bg-slate-800/80 border border-slate-700 rounded-xl p-5 text-sm sm:text-base leading-relaxed">
        <h2 class="text-xl font-bold text-amber-400 mb-3 flex items-center gap-2">
            💡 Cara Kerja Kodingan Game Ini
        </h2>
        <ul class="list-disc list-inside space-y-2 text-slate-300 mb-6">
            <li><strong class="text-white">HTML5 Canvas (<code class="bg-slate-900 px-1.5 py-0.5 rounded text-amber-300">&lt;canvas&gt;</code>):</strong> Bertindak sebagai papan gambar digital tempat elemen game (karakter, platform, koin, duri) digambar ulang setiap bingkai (frame).</li>
            <li><strong class="text-white">Game Loop (<code class="bg-slate-900 px-1.5 py-0.5 rounded text-amber-300">requestAnimationFrame</code>):</strong> Fungsi siklus terus-menerus yang memperbarui posisi pemain (fisika/gravitasi) lalu membersihkan & menggambar ulang seluruh tampilan canvas (~60 kali per detik).</li>
            <li><strong class="text-white">Deteksi Tabrakan (AABB Collision):</strong> Memeriksa apakah kotak koordinat pemain bertabrakan dengan platform (agar bisa dipijak), koin (menambah skor), atau duri (mengurangi nyawa).</li>
            <li><strong class="text-white">Web Audio API:</strong> Menghasilkan efek suara sintetis (Lompat, Koin, Hit) langsung secara algoritma tanpa memerlukan file mp3/wav eksternal.</li>
        </ul>

        <h2 class="text-xl font-bold text-emerald-400 mb-3 flex items-center gap-2">
            💻 Cara Simpan & Mainkan di Laptop Sendiri
        </h2>
        <ol class="list-decimal list-inside space-y-2 text-slate-300">
            <li>Salin seluruh kode dari file HTML ini.</li>
            <li>Buka aplikasi <strong>Notepad</strong> (Windows) atau <strong>TextEdit</strong> (Mac).</li>
            <li>Tempelkan (*paste*) kodenya ke dalam aplikasi teks tersebut.</li>
            <li>Simpan file dengan nama <code class="bg-slate-900 px-2 py-0.5 rounded text-emerald-300 font-mono">game.html</code> (Pastikan formatnya <em>.html</em>, bukan <em>.txt</em>).</li>
            <li>Klik ganda file <code class="bg-slate-900 px-2 py-0.5 rounded text-emerald-300 font-mono">game.html</code> tersebut untuk langsung memainkannya di Google Chrome, Edge, atau Mozilla Firefox tanpa perlu koneksi internet!</li>
        </ol>
    </section>

    <!-- JavaScript Logic -->
    <script>
        // --- SOUND EFFECTS SYNTHESIZER (Web Audio API) ---
        class SoundEffects {
            constructor() {
                this.ctx = null;
            }

            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
            }

            playJump() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'square';
                osc.frequency.setValueAtTime(150, this.ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(400, this.ctx.currentTime + 0.15);
                gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
                gain.gain.linearRampToValueAtTime(0.01, this.ctx.currentTime + 0.15);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.15);
            }

            playCoin() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(800, this.ctx.currentTime);
                osc.frequency.setValueAtTime(1200, this.ctx.currentTime + 0.08);
                gain.gain.setValueAtTime(0.2, this.ctx.currentTime);
                gain.gain.linearRampToValueAtTime(0.01, this.ctx.currentTime + 0.25);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.25);
            }

            playHit() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(120, this.ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(40, this.ctx.currentTime + 0.3);
                gain.gain.setValueAtTime(0.3, this.ctx.currentTime);
                gain.gain.linearRampToValueAtTime(0.01, this.ctx.currentTime + 0.3);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.3);
            }
        }

        const sound = new SoundEffects();

        // --- GAME ENGINE SETUP ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Fixed Canvas Internal Resolution for Retro Crispness
        canvas.width = 800;
        canvas.height = 450;

        // Game Constants
        const GRAVITY = 0.5;
        const PLAYER_SPEED = 4.5;
        const JUMP_FORCE = -10.5;

        // Input Tracking
        const keys = {
            left: false,
            right: false,
            up: false
        };

        // Game State Variables
        let score = 0;
        let lives = 3;
        let gameState = 'START'; // 'START', 'PLAYING', 'GAMEOVER', 'WIN'

        // --- GAME OBJECTS DEFINITION ---
        class Player {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.width = 24;
                this.height = 32;
                this.vx = 0;
                this.vy = 0;
                this.isGrounded = false;
                this.facingRight = true;
            }

            reset(x, y) {
                this.x = x;
                this.y = y;
                this.vx = 0;
                this.vy = 0;
                this.isGrounded = false;
            }

            update(platforms) {
                // Apply Horizontal Movement
                if (keys.left) {
                    this.vx = -PLAYER_SPEED;
                    this.facingRight = false;
                } else if (keys.right) {
                    this.vx = PLAYER_SPEED;
                    this.facingRight = true;
                } else {
                    this.vx *= 0.7; // Friction
                }

                // Apply Jump
                if (keys.up && this.isGrounded) {
                    this.vy = JUMP_FORCE;
                    this.isGrounded = false;
                    sound.playJump();
                }

                // Apply Gravity
                this.vy += GRAVITY;

                // Move Horizontally & Collision
                this.x += this.vx;
                platforms.forEach(p => {
                    if (this.checkCollision(p)) {
                        if (this.vx > 0) this.x = p.x - this.width;
                        else if (this.vx < 0) this.x = p.x + p.width;
                    }
                });

                // Move Vertically & Collision
                this.y += this.vy;
                this.isGrounded = false;
                platforms.forEach(p => {
                    if (this.checkCollision(p)) {
                        if (this.vy > 0) { // Falling down
                            this.y = p.y - this.height;
                            this.vy = 0;
                            this.isGrounded = true;
                        } else if (this.vy < 0) { // Hitting ceiling
                            this.y = p.y + p.height;
                            this.vy = 0;
                        }
                    }
                });

                // Screen Boundaries
                if (this.x < 0) this.x = 0;
                if (this.x + this.width > canvas.width) this.x = canvas.width - this.width;

                // Fall Out Boundary (Dead)
                if (this.y > canvas.height + 50) {
                    handleDamage();
                }
            }

            checkCollision(rect) {
                return this.x < rect.x + rect.width &&
                       this.x + this.width > rect.x &&
                       this.y < rect.y + rect.height &&
                       this.y + this.height > rect.y;
            }

            draw() {
                ctx.save();
                // Player Body (Retro Cyan Boy)
                ctx.fillStyle = '#38bdf8'; // Sky blue
                ctx.fillRect(this.x, this.y, this.width, this.height);

                // Eyes
                ctx.fillStyle = '#0f172a';
                const eyeX = this.facingRight ? this.x + 14 : this.x + 4;
                ctx.fillRect(eyeX, this.y + 6, 6, 6);

                // Headband/Hat
                ctx.fillStyle = '#f43f5e'; // Red cap line
                ctx.fillRect(this.x, this.y, this.width, 5);

                ctx.restore();
            }
        }

        // --- MAP DATA (Platforms, Coins, Spikes) ---
        let player = new Player(50, 300);

        const platforms = [
            // Floor Platforms
            { x: 0, y: 400, width: 250, height: 50 },
            { x: 300, y: 380, width: 200, height: 70 },
            { x: 550, y: 400, width: 250, height: 50 },

            // Floating Platforms
            { x: 120, y: 300, width: 100, height: 16 },
            { x: 260, y: 230, width: 100, height: 16 },
            { x: 420, y: 170, width: 120, height: 16 },
            { x: 600, y: 260, width: 120, height: 16 },
            { x: 50, y: 150, width: 100, height: 16 }
        ];

        let coins = [
            { x: 160, y: 260, radius: 8, collected: false },
            { x: 300, y: 190, radius: 8, collected: false },
            { x: 470, y: 130, radius: 8, collected: false },
            { x: 650, y: 220, radius: 8, collected: false },
            { x: 90, y: 110, radius: 8, collected: false },
            { x: 380, y: 340, radius: 8, collected: false },
            { x: 720, y: 360, radius: 8, collected: false }
        ];

        const spikes = [
            { x: 380, y: 360, width: 40, height: 20 },
            { x: 180, y: 280, width: 20, height: 20 }
        ];

        function resetGame() {
            score = 0;
            lives = 3;
            player.reset(50, 300);
            coins.forEach(c => c.collected = false);
            gameState = 'PLAYING';
        }

        function handleDamage() {
            lives -= 1;
            sound.playHit();
            if (lives <= 0) {
                gameState = 'GAMEOVER';
            } else {
                player.reset(50, 300);
            }
        }

        // --- RENDER HELPERS ---
        function drawPlatforms() {
            platforms.forEach(p => {
                // Top Grass Layer
                ctx.fillStyle = '#22c55e';
                ctx.fillRect(p.x, p.y, p.width, 6);
                // Dirt Layer
                ctx.fillStyle = '#78350f';
                ctx.fillRect(p.x, p.y + 6, p.width, p.height - 6);
            });
        }

        function drawCoins() {
            coins.forEach(c => {
                if (c.collected) return;
                ctx.save();
                ctx.beginPath();
                ctx.arc(c.x, c.y, c.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#eab308'; // Gold
                ctx.fill();
                ctx.strokeStyle = '#fef08a';
                ctx.lineWidth = 2;
                ctx.stroke();
                ctx.restore();

                // Check Coin Pickup Collision
                const dx = (player.x + player.width / 2) - c.x;
                const dy = (player.y + player.height / 2) - c.y;
                const dist = Math.sqrt(dx * dx + dy * dy);
                if (dist < player.width / 2 + c.radius) {
                    c.collected = true;
                    score += 10;
                    sound.playCoin();

                    // Check Win Condition
                    if (coins.every(coin => coin.collected)) {
                        gameState = 'WIN';
                    }
                }
            });
        }

        function drawSpikes() {
            spikes.forEach(s => {
                ctx.save();
                ctx.fillStyle = '#ef4444'; // Red dangerous spikes
                const numTriangles = Math.floor(s.width / 10);
                for (let i = 0; i < numTriangles; i++) {
                    ctx.beginPath();
                    ctx.moveTo(s.x + (i * 10), s.y + s.height);
                    ctx.lineTo(s.x + (i * 10) + 5, s.y);
                    ctx.lineTo(s.x + (i * 10) + 10, s.y + s.height);
                    ctx.closePath();
                    ctx.fill();
                }
                ctx.restore();

                // Check Spike Collision
                if (player.checkCollision(s)) {
                    handleDamage();
                }
            });
        }

        function drawUI() {
            ctx.save();
            ctx.font = '14px "Press Start 2P", cursive';
            
            // Score
            ctx.fillStyle = '#facc15';
            ctx.fillText(`SKOR: ${score}`, 20, 35);

            // Lives
            ctx.fillStyle = '#f43f5e';
            let hearts = '';
            for (let i = 0; i < lives; i++) hearts += '♥ ';
            ctx.fillText(`NYAWA: ${hearts}`, canvas.width - 220, 35);

            ctx.restore();
        }

        function drawOverlay(title, subtitle, color) {
            ctx.save();
            ctx.fillStyle = 'rgba(15, 23, 42, 0.85)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.textAlign = 'center';
            ctx.font = '24px "Press Start 2P", cursive';
            ctx.fillStyle = color;
            ctx.fillText(title, canvas.width / 2, canvas.height / 2 - 20);

            ctx.font = '12px "Press Start 2P", cursive';
            ctx.fillStyle = '#f8fafc';
            ctx.fillText(subtitle, canvas.width / 2, canvas.height / 2 + 30);
            ctx.restore();
        }

        // --- MAIN GAME LOOP ---
        function gameLoop() {
            // Clear Screen
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            if (gameState === 'START') {
                drawPlatforms();
                drawCoins();
                drawSpikes();
                drawOverlay('SUPER DASH 2D', 'Tekan LOMPAT / SPACE / R untuk Mulai', '#facc15');
            } else if (gameState === 'PLAYING') {
                player.update(platforms);
                drawPlatforms();
                drawCoins();
                drawSpikes();
                player.draw();
                drawUI();
            } else if (gameState === 'GAMEOVER') {
                drawPlatforms();
                drawOverlay('GAME OVER!', 'Tekan R atau Tombol Lompat untuk Ulang', '#ef4444');
            } else if (gameState === 'WIN') {
                drawPlatforms();
                drawOverlay('KAMU MENANG!', `Skor Akhir: ${score} | Tekan R untuk Main Lagi`, '#4ade80');
            }

            requestAnimationFrame(gameLoop);
        }

        // --- EVENT LISTENERS ---
        function handleKeyDown(e) {
            sound.init(); // Initialize audio context on first user interaction
            if (e.code === 'KeyA' || e.code === 'ArrowLeft') keys.left = true;
            if (e.code === 'KeyD' || e.code === 'ArrowRight') keys.right = true;
            if (e.code === 'KeyW' || e.code === 'Space' || e.code === 'ArrowUp') {
                keys.up = true;
                if (gameState !== 'PLAYING') resetGame();
            }
            if (e.code === 'KeyR') resetGame();
        }

        function handleKeyUp(e) {
            if (e.code === 'KeyA' || e.code === 'ArrowLeft') keys.left = false;
            if (e.code === 'KeyD' || e.code === 'ArrowRight') keys.right = false;
            if (e.code === 'KeyW' || e.code === 'Space' || e.code === 'ArrowUp') keys.up = false;
        }

        window.addEventListener('keydown', handleKeyDown);
        window.addEventListener('keyup', handleKeyUp);

        // --- TOUCH / ON-SCREEN CONTROLS ---
        const btnLeft = document.getElementById('btnLeft');
        const btnRight = document.getElementById('btnRight');
        const btnJump = document.getElementById('btnJump');

        function bindTouch(element, action) {
            element.addEventListener('touchstart', (e) => {
                e.preventDefault();
                sound.init();
                if (action === 'jump') {
                    keys.up = true;
                    if (gameState !== 'PLAYING') resetGame();
                } else if (action === 'left') {
                    keys.left = true;
                } else if (action === 'right') {
                    keys.right = true;
                }
            });

            element.addEventListener('touchend', (e) => {
                e.preventDefault();
                if (action === 'jump') keys.up = false;
                if (action === 'left') keys.left = false;
                if (action === 'right') keys.right = false;
            });
        }

        bindTouch(btnLeft, 'left');
        bindTouch(btnRight, 'right');
        bindTouch(btnJump, 'jump');

        // Start Animation Loop on Window Load
        window.onload = () => {
            gameLoop();
        };
    </script>
</body>
</html>
```
