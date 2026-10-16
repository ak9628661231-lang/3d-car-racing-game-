<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rasta Rush - Car Racing Game</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
        }
        body {
            background: #222;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
        }
        #game-container {
            position: relative;
            width: 400px;
            height: 650px;
            background: #4e8c33; /* Grass background */
            overflow: hidden;
            border: 4px solid #fff;
            box-shadow: 0 0 20px rgba(0,0,0,0.8);
        }
        /* Road Layout */
        .road {
            position: absolute;
            left: 80px;
            width: 240px;
            height: 100%;
            background: #333;
            border-left: 6px dashed #fff;
            border-right: 6px dashed #fff;
        }
        .road-line {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            width: 4px;
            height: 100%;
            background-image: linear-gradient(to bottom, #fff 50%, transparent 50%);
            background-size: 10px 40px;
            animation: moveRoad 0.5s linear infinite;
        }
        @keyframes moveRoad {
            0% { background-position: 0 0; }
            100% { background-position: 0 40px; }
        }

        /* Side tracks: Train and Bus lanes */
        .train-track {
            position: absolute;
            left: 10px;
            width: 65px;
            height: 100%;
            background: #555;
            border-right: 3px solid #777;
        }
        .side-bus-road {
            position: absolute;
            right: 10px;
            width: 60px;
            height: 100%;
            background: #a89368; /* Dirt road */
        }

        /* Entities */
        .car {
            position: absolute;
            width: 40px;
            height: 70px;
            border-radius: 6px;
        }
        #player {
            background: #e74c3c;
            bottom: 20px;
            left: 180px;
            box-shadow: 0 5px 10px rgba(0,0,0,0.5);
            z-index: 10;
        }
        .enemy {
            background: #3498db;
            top: -80px;
            z-index: 5;
        }
        .train {
            position: absolute;
            left: 12px;
            width: 55px;
            height: 140px;
            background: #8e44ad;
            border-radius: 4px;
            top: -150px;
            z-index: 6;
        }
        .bus {
            position: absolute;
            right: 15px;
            width: 50px;
            height: 100px;
            background: #27ae60;
            border-radius: 4px;
            top: -120px;
            z-index: 6;
        }

        /* UI Elements */
        #ui {
            position: absolute;
            top: 10px;
            left: 10px;
            right: 10px;
            display: flex;
            justify-content: space-between;
            color: #fff;
            font-weight: bold;
            font-size: 14px;
            z-index: 20;
            text-shadow: 1px 1px 2px #000;
        }

        #game-over-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 30;
            display: none;
        }
        #game-over-screen h1 {
            color: #e74c3c;
            font-size: 28px;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px #000;
        }
        .btn {
            background: #2ecc71;
            color: white;
            border: none;
            padding: 10px 20px;
            margin-top: 15px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            border-radius: 5px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }
        .btn:hover {
            background: #27ae60;
        }

        /* Controls UI for Mobile/Click */
        #controls {
            position: absolute;
            bottom: 10px;
            width: 100%;
            display: flex;
            justify-content: space-around;
            z-index: 25;
        }
        .ctrl-btn {
            background: rgba(255, 255, 255, 0.3);
            border: 2px solid white;
            color: white;
            padding: 12px 25px;
            font-size: 18px;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            user-select: none;
        }
        .ctrl-btn:active {
            background: rgba(255, 255, 255, 0.6);
        }
    </style>
</head>
<body>

    <div id="game-container">
        <!-- UI Header -->
        <div id="ui">
            <span id="score-txt">SCORE: 0</span>
            <span id="lives-txt">LIVES: 3</span>
            <span id="level-txt">LEVEL: 1</span>
        </div>

        <!-- Side Tracks and Road -->
        <div class="train-track"></div>
        <div class="road">
            <div class="road-line"></div>
        </div>
        <div class="side-bus-road"></div>

        <!-- Game Elements -->
        <div id="player" class="car"></div>
        <div id="enemy" class="enemy car"></div>
        <div id="train" class="train"></div>
        <div id="bus" class="bus"></div>

        <!-- Game Over Screen -->
        <div id="game-over-screen">
            <h1>GAME OVER</h1>
            <p>Aapke Points: <span id="final-score">0</span></p>
            <button class="btn" onclick="restartGame()">RETRY</button>
        </div>

        <!-- Control Buttons -->
        <div id="controls">
            <div class="ctrl-btn" ontouchstart="movePlayer(-40)" onclick="movePlayer(-40)">◀</div>
            <div class="ctrl-btn" ontouchstart="movePlayer(40)" onclick="movePlayer(40)">▶</div>
        </div>
    </div>

    <script>
        const player = document.getElementById('player');
        const enemy = document.getElementById('enemy');
        const train = document.getElementById('train');
        const bus = document.getElementById('bus');
        const scoreTxt = document.getElementById('score-txt');
        const livesTxt = document.getElementById('lives-txt');
        const levelTxt = document.getElementById('level-txt');
        const gameOverScreen = document.getElementById('game-over-screen');
        const finalScore = document.getElementById('final-score');

        let score = 0;
        let lives = 3;
        let level = 1;
        let isGameOver = false;

        // Player Position within road (Road starts at left: 80px, width: 240px)
        let playerX = 180; 

        // Speeds
        let enemyY = -100;
        let enemyX = 130; // Random lane on road
        let enemySpeed = 5;

        let trainY = -200;
        let trainSpeed = 4;

        let busY = -150;
        let busSpeed = 3;

        // Keyboard Controls
        document.addEventListener('keydown', (e) => {
            if (isGameOver) return;
            if (e.key === 'ArrowLeft') movePlayer(-20);
            if (e.key === 'ArrowRight') movePlayer(20);
        });

        function movePlayer(val) {
            if (isGameOver) return;
            playerX += val;
            // Road boundary limits (85 to 280)
            if (playerX < 90) playerX = 90;
            if (playerX > 270) playerX = 270;
            player.style.left = playerX + 'px';
        }

        function updateGame() {
            if (isGameOver) return;

            // Move Enemy Car
            enemyY += enemySpeed;
            if (enemyY > 650) {
                enemyY = -100;
                // Randomly change enemy lane inside road limits
                enemyX = Math.random() > 0.5 ? 130 : 220;
                score += 10;
                scoreTxt.innerText = "SCORE: " + score;

                // Increase Level based on score
                if (score % 50 === 0) {
                    level++;
                    enemySpeed += 1;
                    levelTxt.innerText = "LEVEL: " + level;
                }
            }
            enemy.style.top = enemyY + 'px';
            enemy.style.left = enemyX + 'px';

            // Move Train (Parallel Track)
            trainY += trainSpeed;
            if (trainY > 650) {
                trainY = -250;
            }
            train.style.top = trainY + 'px';

            // Move Bus (Outside Road)
            busY += busSpeed;
            if (busY > 650) {
                busY = -200;
            }
            bus.style.top = busY + 'px';

            // Collision Detection with Enemy Car
            let playerRect = player.getBoundingClientRect();
            let enemyRect = enemy.getBoundingClientRect();

            if (
                playerRect.left < enemyRect.right &&
                playerRect.right > enemyRect.left &&
                playerRect.top < enemyRect.bottom &&
                playerRect.bottom > enemyRect.top
            ) {
                lives--;
                livesTxt.innerText = "LIVES: " + lives;
                enemyY = -100; // Reset enemy position after crash
                
                if (lives <= 0) {
                    endGame();
                }
            }

            requestAnimationFrame(updateGame);
        }

        function endGame() {
            isGameOver = true;
            finalScore.innerText = score;
            gameOverScreen.style.display = 'flex';
        }

        function restartGame() {
            score = 0;
            lives = 3;
            level = 1;
            enemySpeed = 5;
            isGameOver = false;
            enemyY = -100;
            trainY = -200;
            busY = -150;
            playerX = 180;
            player.style.left = playerX + 'px';
            scoreTxt.innerText = "SCORE: 0";
            livesTxt.innerText = "LIVES: 3";
            levelTxt.innerText = "LEVEL: 1";
            gameOverScreen.style.display = 'none';
            updateGame();
        }

        // Start Game Loop
        updateGame();
    </script>
</body>
</html>
