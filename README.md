# annabellelangley9.github.io
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ralph's Adventure</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: #4169E1;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            overflow: hidden;
            user-select: none;
        }

        .game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            overflow: hidden;
        }

        .game-map {
            position: absolute;
            width: 95vw;
            height: 80vh;
            max-width: 1000px;
            max-height: 600px;
            background: #87CEEB;
            border: 3px solid #4169E1;
            border-radius: 20px;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }

        .player {
            position: absolute;
            font-size: 32px;
            z-index: 100;
            transition: all 0.2s ease;
        }

        .heart {
            position: absolute;
            font-size: 24px;
            z-index: 50;
            animation: pulse 2s infinite;
        }

        .villain {
            position: absolute;
            font-size: 28px;
            z-index: 60;
        }

        .safe-zone {
            position: absolute;
            font-size: 40px;
            z-index: 40;
        }

        .london-eye {
            position: absolute;
            font-size: 48px;
            z-index: 40;
        }

        .girlfriend {
            position: absolute;
            font-size: 32px;
            z-index: 45;
            animation: wave 3s infinite;
        }

        .hud {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(255, 255, 255, 0.9);
            padding: 15px 20px;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            z-index: 200;
        }

        .game-title {
            position: absolute;
            top: 20px;
            right: 20px;
            background: rgba(255, 255, 255, 0.9);
            padding: 10px 20px;
            border-radius: 15px;
            font-size: 24px;
            font-weight: bold;
            color: #4169E1;
            z-index: 200;
        }

        .modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }

        .modal-content {
            background: white;
            padding: 40px;
            border-radius: 20px;
            text-align: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
        }

        .modal h2 {
            font-size: 48px;
            margin: 0 0 20px 0;
            color: #4169E1;
        }

        .modal button {
            background: #4169E1;
            color: white;
            border: none;
            padding: 15px 30px;
            font-size: 18px;
            border-radius: 10px;
            cursor: pointer;
            margin-top: 20px;
        }

        .modal button:hover {
            background: #1E90FF;
        }

        .instructions {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: rgba(255, 255, 255, 0.9);
            padding: 10px 15px;
            border-radius: 10px;
            font-size: 14px;
            z-index: 200;
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.2); }
        }

        @keyframes wave {
            0%, 100% { transform: translateY(0px); }
            50% { transform: translateY(-10px); }
        }

        .collected {
            opacity: 0.3;
            transform: scale(0.5);
        }
    </style>
</head>
<body>
    <div class="game-container">
        <div class="game-title">🎮 Ralph's Adventure</div>
        
        <div class="hud">
            <div>❤️ Lives: <span id="lives">5</span></div>
            <div>💖 Hearts: <span id="hearts">0</span>/15</div>
        </div>

        <div class="instructions">
            Use WASD or Arrow Keys to move Ralph around London!
        </div>

        <div class="game-map" id="gameMap">
            <!-- Player -->
            <div class="player" id="player" style="top: 50px; left: 100px;">👨</div>
            
            <!-- Tower of London (Safe Zone) -->
            <div class="safe-zone" id="towerOfLondon" style="top: 30px; left: 80px;">🏰</div>
            
            <!-- London Eye -->
            <div class="london-eye" id="londonEye" style="bottom: 50px; right: 80px;">🎡</div>
            
            <!-- Annabelle at London Eye -->
            <div class="girlfriend" id="annabelle" style="bottom: 80px; right: 140px;">👱‍♀️</div>
        </div>
    </div>

    <!-- Win Modal -->
    <div class="modal" id="winModal" style="display: none;">
        <div class="modal-content">
            <h2>🎉 YOU WIN! 🎉</h2>
            <p>Ralph successfully delivered all the hearts to Annabelle!</p>
            <button onclick="restartGame()">Play Again</button>
        </div>
    </div>

    <!-- Lose Modal -->
    <div class="modal" id="loseModal" style="display: none;">
        <div class="modal-content">
            <h2>💔 YOU LOSE 💔</h2>
            <p>Ralph ran out of lives!</p>
            <p>Try again to help him reach Annabelle!</p>
            <button onclick="restartGame()">Try Again</button>
        </div>
    </div>

    <script>
        class Game {
            constructor() {
                this.player = document.getElementById('player');
                this.gameMap = document.getElementById('gameMap');
                this.lives = 5;
                this.heartsCollected = 0;
                this.playerX = 100;
                this.playerY = 50;
                this.hearts = [];
                this.villains = [];
                this.gameRunning = true;
                this.recentlyHit = false;
                
                this.init();
                this.setupControls();
                this.gameLoop();
            }

            init() {
                // Create hearts at tube stations (trains)
                const heartPositions = [
                    {x: 150, y: 80}, {x: 300, y: 60}, {x: 450, y: 120},
                    {x: 600, y: 90}, {x: 750, y: 70}, {x: 200, y: 200},
                    {x: 500, y: 220}, {x: 100, y: 280}, {x: 350, y: 320},
                    {x: 650, y: 280}, {x: 180, y: 400}, {x: 450, y: 450},
                    {x: 650, y: 420}, {x: 300, y: 500}, {x: 800, y: 480}
                ];

                heartPositions.forEach((pos, index) => {
                    const heartContainer = document.createElement('div');
                    heartContainer.style.position = 'absolute';
                    heartContainer.style.left = pos.x + 'px';
                    heartContainer.style.top = pos.y + 'px';
                    heartContainer.innerHTML = '🚂💖';
                    heartContainer.className = 'heart';
                    heartContainer.id = 'heart' + index;
                    this.gameMap.appendChild(heartContainer);
                    this.hearts.push({element: heartContainer, x: pos.x, y: pos.y, collected: false});
                });

                // Create villains
                const villainTypes = ['💣', '🥷', '🔪', '🦹‍♂️'];
                const villainPositions = [
                    {x: 250, y: 150}, {x: 400, y: 180}, {x: 550, y: 140},
                    {x: 150, y: 250}, {x: 450, y: 240}, {x: 600, y: 260},
                    {x: 300, y: 380}, {x: 550, y: 360}, {x: 200, y: 450},
                    {x: 700, y: 200}, {x: 120, y: 380}, {x: 750, y: 280},
                    {x: 350, y: 280}, {x: 500, y: 380}, {x: 700, y: 450}
                ];

                villainPositions.forEach((pos, index) => {
                    const villain = document.createElement('div');
                    villain.style.position = 'absolute';
                    villain.style.left = pos.x + 'px';
                    villain.style.top = pos.y + 'px';
                    villain.innerHTML = villainTypes[index % villainTypes.length];
                    villain.className = 'villain';
                    this.gameMap.appendChild(villain);
                    
                    this.villains.push({
                        element: villain,
                        x: pos.x,
                        y: pos.y,
                        dx: (Math.random() - 0.5) * 2,
                        dy: (Math.random() - 0.5) * 2
                    });
                });
            }

            setupControls() {
                document.addEventListener('keydown', (e) => {
                    if (!this.gameRunning) return;
                    
                    const speed = 15;
                    let newX = this.playerX;
                    let newY = this.playerY;

                    switch(e.key.toLowerCase()) {
                        case 'w':
                        case 'arrowup':
                            newY = Math.max(10, this.playerY - speed);
                            break;
                        case 's':
                        case 'arrowdown':
                            newY = Math.min(550, this.playerY + speed);
                            break;
                        case 'a':
                        case 'arrowleft':
                            newX = Math.max(10, this.playerX - speed);
                            break;
                        case 'd':
                        case 'arrowright':
                            newX = Math.min(950, this.playerX + speed);
                            break;
                    }

                    this.playerX = newX;
                    this.playerY = newY;
                    this.player.style.left = this.playerX + 'px';
                    this.player.style.top = this.playerY + 'px';
                });
            }

            checkCollisions() {
                // Check heart collection
                this.hearts.forEach(heart => {
                    if (!heart.collected && this.isColliding(this.playerX, this.playerY, heart.x, heart.y, 40)) {
                        heart.collected = true;
                        heart.element.classList.add('collected');
                        this.heartsCollected++;
                        document.getElementById('hearts').textContent = this.heartsCollected;
                    }
                });

                // Check if Ralph is in Tower of London safe zone
                const inSafeZone = this.isColliding(this.playerX, this.playerY, 80, 30, 60);

                // Check villain collision (only if not in safe zone and not recently hit)
                if (!inSafeZone && !this.recentlyHit) {
                    this.villains.forEach(villain => {
                        if (this.isColliding(this.playerX, this.playerY, villain.x, villain.y, 35)) {
                            this.loseLife();
                        }
                    });
                }

                // Check if at Tower of London with all hearts (win condition)
                if (this.heartsCollected >= 15 && inSafeZone) {
                    this.winGame();
                }
                
                // Check if at London Eye with all hearts (alternative win condition)
                if (this.heartsCollected >= 15 && 
                    this.isColliding(this.playerX, this.playerY, 850, 500, 60)) {
                    this.winGame();
                }
            }

            isColliding(x1, y1, x2, y2, threshold) {
                const distance = Math.sqrt((x1 - x2) ** 2 + (y1 - y2) ** 2);
                return distance < threshold;
            }

            loseLife() {
                this.lives--;
                document.getElementById('lives').textContent = this.lives;
                
                // Set hit protection for 2 seconds
                this.recentlyHit = true;
                setTimeout(() => {
                    this.recentlyHit = false;
                }, 2000);
                
                // Reset to Tower of London
                this.playerX = 100;
                this.playerY = 50;
                this.player.style.left = this.playerX + 'px';
                this.player.style.top = this.playerY + 'px';

                if (this.lives <= 0) {
                    this.gameOver();
                }
            }

            moveVillains() {
                this.villains.forEach(villain => {
                    villain.x += villain.dx;
                    villain.y += villain.dy;

                    // Bounce off walls
                    if (villain.x <= 20 || villain.x >= 960) villain.dx *= -1;
                    if (villain.y <= 20 || villain.y >= 560) villain.dy *= -1;

                    // Keep in bounds
                    villain.x = Math.max(20, Math.min(960, villain.x));
                    villain.y = Math.max(20, Math.min(560, villain.y));

                    villain.element.style.left = villain.x + 'px';
                    villain.element.style.top = villain.y + 'px';
                });
            }

            gameLoop() {
                if (!this.gameRunning) return;
                
                this.moveVillains();
                this.checkCollisions();
                
                requestAnimationFrame(() => this.gameLoop());
            }

            winGame() {
                this.gameRunning = false;
                document.getElementById('winModal').style.display = 'flex';
            }

            gameOver() {
                this.gameRunning = false;
                document.getElementById('loseModal').style.display = 'flex';
            }
        }

        function restartGame() {
            // Hide modals
            document.getElementById('winModal').style.display = 'none';
            document.getElementById('loseModal').style.display = 'none';
            
            // Clear existing game elements
            const hearts = document.querySelectorAll('.heart');
            const villains = document.querySelectorAll('.villain');
            hearts.forEach(heart => heart.remove());
            villains.forEach(villain => villain.remove());
            
            // Reset HUD
            document.getElementById('lives').textContent = '5';
            document.getElementById('hearts').textContent = '0';
            
            // Start new game
            new Game();
        }

        // Start the game
        window.onload = () => {
            new Game();
        };
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'97f01fc7058c0753',t:'MTc1Nzg1NTQyMi4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
