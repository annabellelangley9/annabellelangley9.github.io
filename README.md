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

    .station-heart {
      position: absolute;
      font-size: 28px;
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
      
      <!-- Girlfriend at London Eye -->
      <div class="girlfriend" id="girlfriend" style="bottom: 80px; right: 140px;">👱‍♀️</div>
    </div>
  </div>

  <!-- Win Modal -->
  <div class="modal" id="winModal" style="display: none;">
    <div class="modal-content">
      <h2>🎉 YOU WIN! 🎉</h2>
      <p>Ralph successfully delivered all the hearts to the London Eye!</p>
      <button onclick="restartGame()">Play Again</button>
    </div>
  </div>

  <!-- Lose Modal -->
  <div class="modal" id="loseModal" style="display: none;">
    <div class="modal-content">
      <h2>💔 YOU LOSE 💔</h2>
      <p>Ralph ran out of lives!</p>
      <p>Try again to help him reach the London Eye!</p>
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
        this.stations = [];
        this.villains = [];
        this.gameRunning = true;
        this.recentlyHit = false;
        
        this.init();
        this.setupControls();
        this.gameLoop();
      }

      init() {
        // Perfectly even placement: 5x3 grid
        const cols = 5, rows = 3;
        const cellWidth = 1000 / cols;
        const cellHeight = 600 / rows;
        let stationId = 0;

        for (let row = 0; row < rows; row++) {
          for (let col = 0; col < cols; col++) {
            const x = Math.floor(col * cellWidth + cellWidth / 2);
            const y = Math.floor(row * cellHeight + cellHeight / 2);

            const stationHeart = document.createElement('div');
            stationHeart.style.position = 'absolute';
            stationHeart.style.left = x + 'px';
            stationHeart.style.top = y + 'px';
            stationHeart.innerHTML = '🚂💖';
            stationHeart.className = 'station-heart';
            stationHeart.id = 'station' + stationId;
            this.gameMap.appendChild(stationHeart);

            this.stations.push({element: stationHeart, x: x, y: y, collected: false});
            stationId++;
          }
        }

        // Random 25 villains
        const villainTypes = ['💣', '🥷', '🔪', '🦹‍♂️'];
        for (let i = 0; i < 25; i++) {
          const x = Math.floor(Math.random() * 900) + 50;
          const y = Math.floor(Math.random() * 500) + 50;

          const villain = document.createElement('div');
          villain.style.position = 'absolute';
          villain.style.left = x + 'px';
          villain.style.top = y + 'px';
          villain.innerHTML = villainTypes[i % villainTypes.length];
          villain.className = 'villain';
          this.gameMap.appendChild(villain);

          this.villains.push({
            element: villain,
            x: x,
            y: y,
            dx: (Math.random() - 0.5) * 2,
            dy: (Math.random() - 0.5) * 2
          });
        }
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
        // Collect hearts
        this.stations.forEach(station => {
          if (!station.collected && this.isColliding(this.playerX, this.playerY, station.x, station.y, 40)) {
            station.collected = true;
            station.element.classList.add('collected');
            this.heartsCollected++;
            document.getElementById('hearts').textContent = this.heartsCollected;
          }
        });

        // Safe zones: Tower of London + London Eye
        const inTower = this.isColliding(this.playerX, this.playerY, 80, 30, 60);
        const inLondonEye = this.isColliding(this.playerX, this.playerY, 920, 550, 80);
        const inSafeZone = inTower || inLondonEye;

        // Villain collisions
        if (!inSafeZone && !this.recentlyHit) {
          this.villains.forEach(villain => {
            if (this.isColliding(this.playerX, this.playerY, villain.x, villain.y, 35)) {
              this.loseLife();
            }
          });
        }

        // Win condition
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
        
        this.recentlyHit = true;
        setTimeout(() => { this.recentlyHit = false; }, 2000);
        
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

          if (villain.x <= 20 || villain.x >= 960) villain.dx *= -1;
          if (villain.y <= 20 || villain.y >= 560) villain.dy *= -1;

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
      document.getElementById('winModal').style.display = 'none';
      document.getElementById('loseModal').style.display = 'none';
      
      const stations = document.querySelectorAll('.station-heart');
      const villains = document.querySelectorAll('.villain');
      stations.forEach(station => station.remove());
      villains.forEach(villain => villain.remove());
      
      document.getElementById('lives').textContent = '5';
      document.getElementById('hearts').textContent = '0';
      
      new Game();
    }

    window.onload = () => { new Game(); };
  </script>
</body>
</html>
