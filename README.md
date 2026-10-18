
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Highway Racing</title>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  -webkit-tap-highlight-color: transparent;
}

body {
  background: #17251b;
  font-family: Arial, sans-serif;
  color: white;
  text-align: center;
  overflow: hidden;
  touch-action: manipulation;
}

h2 {
  padding: 8px;
  font-size: 22px;
  background: #111;
}

#hud {
  display: flex;
  justify-content: space-around;
  align-items: center;
  background: #202020;
  padding: 9px 4px;
  font-size: 15px;
  font-weight: bold;
}

#game {
  position: relative;
  width: min(100vw, 420px);
  height: min(67vh, 590px);
  min-height: 350px;
  margin: 0 auto;
  overflow: hidden;
  background: #378b35;
  border-left: 5px solid #d9d9d9;
  border-right: 5px solid #d9d9d9;
}

#road {
  position: absolute;
  inset: 0;
  background: #424242;
  overflow: hidden;
}

#road::before,
#road::after {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  width: 5px;
  background: #f8f8f8;
}

#road::before { left: 5px; }
#road::after { right: 5px; }

.lane {
  position: absolute;
  top: -100%;
  height: 200%;
  width: 5px;
  background: repeating-linear-gradient(
    to bottom,
    #fff 0px,
    #fff 42px,
    transparent 42px,
    transparent 82px
  );
  animation: roadMove .65s linear infinite;
}

.lane.one { left: 33.33%; }
.lane.two { left: 66.66%; }

@keyframes roadMove {
  from { transform: translateY(0); }
  to { transform: translateY(82px); }
}

.scenery {
  position: absolute;
  top: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.tree {
  position: absolute;
  font-size: 28px;
  animation: treeMove linear infinite;
}

@keyframes treeMove {
  from { transform: translateY(-80px); }
  to { transform: translateY(650px); }
}

.car {
  position: absolute;
  width: 48px;
  height: 82px;
  border-radius: 12px 12px 8px 8px;
  border: 2px solid rgba(255,255,255,.8);
  box-shadow: 0 4px 5px #171717;
  z-index: 5;
}

.car::before {
  content: "";
  position: absolute;
  top: 13px;
  left: 7px;
  width: 30px;
  height: 21px;
  border-radius: 5px;
  background: #a9e6f4;
  border: 2px solid #222;
}

.car::after {
  content: "";
  position: absolute;
  bottom: 9px;
  left: 8px;
  width: 28px;
  height: 12px;
  border-radius: 3px;
  background: #252525;
  border-top: 3px solid #ddd;
}

.player {
  background: #e52424;
  bottom: 22px;
  left: calc(50% - 24px);
}

.enemy {
  background: #1685ee;
}

.enemy.yellow { background: #f1bd17; }
.enemy.purple { background: #8e44ad; }
.enemy.white { background: #eeeeee; }

.wheel {
  position: absolute;
  top: 20px;
  width: 5px;
  height: 19px;
  background: #111;
  border-radius: 2px;
}

.wheel.left { left: -5px; }
.wheel.right { right: -5px; }

#overlay {
  position: absolute;
  inset: 0;
  z-index: 20;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 15px;
  background: rgba(0,0,0,.78);
  padding: 20px;
}

#overlay h1 {
  font-size: 34px;
  color: #ffe34c;
}

#overlay p {
  font-size: 16px;
  line-height: 1.5;
}

button {
  border: 0;
  border-radius: 10px;
  padding: 13px 23px;
  font-size: 17px;
  font-weight: bold;
  cursor: pointer;
}

#startBtn {
  background: #19c963;
  color: #071b0e;
}

#controls {
  display: flex;
  justify-content: center;
  gap: 40px;
  padding: 12px;
  background: #17251b;
}

.control {
  width: 100px;
  height: 54px;
  font-size: 28px;
  color: white;
  background: #303b45;
  border: 2px solid #a5b5c1;
  touch-action: none;
}

#hint {
  font-size: 12px;
  color: #d5e7d4;
  padding-bottom: 5px;
}
</style>
</head>

<body>

<h2>🏎️ HIGHWAY RACING</h2>

<div id="hud">
  <span>🏆 Score: <b id="score">0</b></span>
  <span>🚗 Speed: <b id="speed">1</b></span>
  <span>⭐ Best: <b id="best">0</b></span>
</div>

<div id="game">
  <div id="road">
    <div class="lane one"></div>
    <div class="lane two"></div>
  </div>

  <div class="scenery" id="scenery"></div>

  <div class="car player" id="player">
    <span class="wheel left"></span>
    <span class="wheel right"></span>
  </div>

  <div id="overlay">
    <h1 id="message">HIGHWAY RACING</h1>
    <p id="instructions">
      Dodge enemy cars and drive as far as you can!<br>
      Use the left and right buttons to steer.
    </p>
    <button id="startBtn">▶ START GAME</button>
  </div>
</div>

<div id="controls">
  <button class="control" id="leftBtn">⬅️</button>
  <button class="control" id="rightBtn">➡️</button>
</div>

<div id="hint">Move left or right to avoid other cars</div>

<script>
(() => {
  const game = document.getElementById("game");
  const road = document.getElementById("road");
  const player = document.getElementById("player");
  const overlay = document.getElementById("overlay");
  const startBtn = document.getElementById("startBtn");
  const message = document.getElementById("message");
  const instructions = document.getElementById("instructions");
  const scoreEl = document.getElementById("score");
  const speedEl = document.getElementById("speed");
  const bestEl = document.getElementById("best");
  const scenery = document.getElementById("scenery");

  let running = false;
  let playerX = 0;
  let score = 0;
  let speed = 220;
  let enemies = [];
  let lastTime = 0;
  let spawnTimer = 0;
  let animationId = 0;
  let roadOffset = 0;
  let keys = {};
  let touchDirection = 0;
  let touchTimer = 0;
  let best = 0;

  try {
    best = Number(localStorage.getItem("highwayBest")) || 0;
  } catch (e) {}
  bestEl.textContent = best;

  const lanePositions = [1/6, 1/2, 5/6];

  function laneX(index) {
    return game.clientWidth * lanePositions[index] - 24;
  }

  function makeTree() {
    const tree = document.createElement("div");
    tree.className = "tree";
    tree.textContent = Math.random() > .5 ? "🌳" : "🌲";
    tree.style.left = Math.random() > .5
      ? (Math.random() * 13) + "%"
      : (84 + Math.random() * 10) + "%";
    tree.style.top = (-30 - Math.random() * 500) + "px";
    tree.style.animationDuration = (4 + Math.random() * 4) + "s";
    scenery.appendChild(tree);
  }

  for (let i = 0; i < 12; i++) makeTree();

  function clearEnemies() {
    enemies.forEach(e => e.el.remove());
    enemies = [];
  }

  function resetGame() {
    cancelAnimationFrame(animationId);
    clearEnemies();

    score = 0;
    speed = 220;
    spawnTimer = 0;
    roadOffset = 0;
    lastTime = 0;

    playerX = laneX(1);
    player.style.left = playerX + "px";

    scoreEl.textContent = "0";
    speedEl.textContent = "1";

    document.querySelectorAll(".tree").forEach(t => t.remove());
    for (let i = 0; i < 12; i++) makeTree();

    overlay.style.display = "none";
    running = true;
    animationId = requestAnimationFrame(loop);
  }

  function spawnEnemy() {
    const available = [0, 1, 2];
    const lastLane = enemies.length
      ? enemies[enemies.length - 1].lane
      : -1;

    let options = available.filter(l => l !== lastLane);
    const lane = options[Math.floor(Math.random() * options.length)];

    const el = document.createElement("div");
    const types = ["", "yellow", "purple", "white"];
    el.className = "car enemy " +
      types[Math.floor(Math.random() * types.length)];

    el.innerHTML =
      '<span class="wheel left"></span>' +
      '<span class="wheel right"></span>';

    el.style.left = laneX(lane) + "px";
    el.style.top = "-100px";
    road.appendChild(el);

    enemies.push({
      el,
      lane,
      y: -100,
      passed: false,
      extraSpeed: Math.random() * 45
    });
  }

  function movePlayer(direction, dt) {
    if (!direction) return;

    playerX += direction * 330 * dt;

    const minX = 9;
    const maxX = game.clientWidth - player.offsetWidth - 9;

    playerX = Math.max(minX, Math.min(maxX, playerX));
    player.style.left = playerX + "px";
  }

  function collision(a, b) {
    const pad = 10;
    return (
      a.left + pad < b.right - pad &&
      a.right - pad > b.left + pad &&
      a.top + pad < b.bottom - pad &&
      a.bottom - pad > b.top + pad
    );
  }

  function gameOver() {
    if (!running) return;
    running = false;

    cancelAnimationFrame(animationId);

    if (score > best) {
      best = score;
      bestEl.textContent = best;
      try {
        localStorage.setItem("highwayBest", best);
      } catch (e) {}
    }

    message.textContent = "GAME OVER!";
    instructions.innerHTML =
      "Your score: <b>" + score +
      "</b><br>Try again and beat your best score!";
    startBtn.textContent = "↻ RESTART GAME";
    overlay.style.display = "flex";
  }

  function loop(timestamp) {
    if (!running) return;

    if (!lastTime) lastTime = timestamp;
    const dt = Math.min((timestamp - lastTime) / 1000, 0.04);
    lastTime = timestamp;

    const direction =
      (keys.ArrowRight || keys.d ? 1 : 0) -
      (keys.ArrowLeft || keys.a ? 1 : 0) ||
      touchDirection;

    movePlayer(direction, dt);

    speed = Math.min(390, 220 + score * 1.2);
    roadOffset += speed * dt;

    document.querySelectorAll(".lane").forEach(lane => {
      lane.style.backgroundPositionY = roadOffset + "px";
    });

    document.querySelectorAll(".tree").forEach(tree => {
      let y = parseFloat(tree.dataset.y || "-30");
      y += speed * dt;

      if (y > game.clientHeight + 30) {
        y = -50;
        tree.style.left = Math.random() > .5
          ? (Math.random() * 13) + "%"
          : (84 + Math.random() * 10) + "%";
      }

      tree.dataset.y = y;
      tree.style.animation = "none";
      tree.style.top = y + "px";
    });

    spawnTimer += dt * 1000;

    if (spawnTimer > Math.max(650, 1150 - score * 2)) {
      spawnEnemy();
      spawnTimer = 0;
    }

    const playerRect = player.getBoundingClientRect();

    for (let i = enemies.length - 1; i >= 0; i--) {
      const e = enemies[i];
      e.y += (speed * .72 + e.extraSpeed) * dt;
      e.el.style.top = e.y + "px";

      const enemyRect = e.el.getBoundingClientRect();

      if (collision(playerRect, enemyRect)) {
        gameOver();
        return;
      }

      if (!e.passed && e.y > game.clientHeight) {
        e.passed = true;
        score += 10;
        scoreEl.textContent = score;
        speedEl.textContent = Math.round(speed / 100);
      }

      if (e.y > game.clientHeight + 100) {
        e.el.remove();
        enemies.splice(i, 1);
      }
    }

    animationId = requestAnimationFrame(loop);
  }

  startBtn.addEventListener("click", resetGame);

  document.addEventListener("keydown", e => {
    if (["ArrowLeft", "ArrowRight", " "].includes(e.key)) {
      e.preventDefault();
    }
    keys[e.key] = true;
    if (!running && e.key === " ") resetGame();
  });

  document.addEventListener("keyup", e => {
    keys[e.key] = false;
  });

  function holdButton(button, direction) {
    button.addEventListener("pointerdown", e => {
      e.preventDefault();
      touchDirection = direction;
    });

    button.addEventListener("pointerup", () => {
      if (touchDirection === direction) touchDirection = 0;
    });

    button.addEventListener("pointercancel", () => {
      touchDirection = 0;
    });

    button.addEventListener("pointerleave", () => {
      touchDirection = 0;
    });
  }

  holdButton(document.getElementById("leftBtn"), -1);
  holdButton(document.getElementById("rightBtn"), 1);

  window.addEventListener("blur", () => {
    keys = {};
    touchDirection = 0;
  });

  window.addEventListener("resize", () => {
    if (!running) {
      playerX = laneX(1);
      player.style.left = playerX + "px";
    }
  });
})();
</script>

</body>
</html>
