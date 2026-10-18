
<!DOCTYPE html>
<html lang="en">
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Car Racing Game</title>
<style>
body {
  margin: 0;
  background: #101820;
  color: white;
  font-family: Arial, sans-serif;
  text-align: center;
}
h2 { color: #ffd166; }
canvas {
  width: 95%;
  max-width: 400px;
  background: #555;
  border: 3px solid white;
  border-radius: 8px;
  touch-action: none;
}
button {
  padding: 14px 24px;
  margin: 8px;
  border: 0;
  border-radius: 10px;
  background: #ffd166;
  font-size: 18px;
  font-weight: bold;
}
.controls button { background: #263746; color: white; }
</style>
</head>
<body>
<h2>🏁 CAR RACING GAME</h2>
<p>Score: <span id="score">0</span></p>

<canvas id="game" width="400" height="600"></canvas>
<br>
<button onclick="start()">START / RESTART</button>
<div class="controls">
<button id="left">⬅ LEFT</button>
<button id="right">RIGHT ➡</button>
</div>
<p>Traffic cars se bacho!</p>

<script>
const c = document.getElementById("game");
const ctx = c.getContext("2d");
const scoreText = document.getElementById("score");

let player, enemies, score, speed, playing;
let left = false, right = false, roadLine = 0;
let last = 0, spawn = 0, animation = 0;

function car(x, y, color, w = 38, h = 65) {
  ctx.fillStyle = "#111";
  ctx.fillRect(x-4,y+10,5,15);
  ctx.fillRect(x+w-1,y+10,5,15);
  ctx.fillRect(x-4,y+h-22,5,15);
  ctx.fillRect(x+w-1,y+h-22,5,15);
  ctx.fillStyle = color;
  ctx.fillRect(x,y,w,h);
  ctx.fillStyle = "#bde0fe";
  ctx.fillRect(x+6,y+10,w-12,15);
  ctx.fillStyle = "#222";
  ctx.fillRect(x+6,y+h-20,w-12,12);
}

function drawRoad(dt) {
  ctx.fillStyle = "#24753b";
  ctx.fillRect(0,0,400,600);
  ctx.fillStyle = "#484848";
  ctx.fillRect(50,0,300,600);

  ctx.fillStyle = "white";
  ctx.fillRect(57,0,4,600);
  ctx.fillRect(339,0,4,600);

  roadLine = (roadLine + speed * dt / 16) % 80;
  for (let y = -80 + roadLine; y < 600; y += 80) {
    ctx.fillRect(148,y,4,42);
    ctx.fillRect(248,y,4,42);
  }
}

function draw(dt) {
  drawRoad(dt);
  enemies.forEach(e => car(e.x,e.y,e.color));
  car(player.x,player.y,"#06d6a0");
}

function start() {
  cancelAnimationFrame(animation);
  player = {x:181,y:510,w:38,h:65};
  enemies = [];
  score = 0;
  speed = 4;
  spawn = 0;
  roadLine = 0;
  playing = true;
  left = right = false;
  last = performance.now();
  scoreText.textContent = score;
  animation = requestAnimationFrame(loop);
}

function loop(time) {
  if (!playing) return;
  const dt = Math.min(time-last, 32);
  last = time;

  if (left) player.x -= 5 * dt/16;
  if (right) player.x += 5 * dt/16;
  player.x = Math.max(65,Math.min(297,player.x));

  speed = Math.min(10,4+score/250);
  spawn += dt;

  if (spawn > Math.max(380,900-score*2)) {
    const lane = Math.floor(Math.random()*3);
    enemies.push({
      x: 81 + lane*100,
      y: -70,
      color: ["#ef476f","#118ab2","#ffd166","#eeeeee"][
        Math.floor(Math.random()*4)
      ]
    });
    spawn = 0;
  }

  for (let i=enemies.length-1;i>=0;i--) {
    const e = enemies[i];
    e.y += speed*dt/16;

    if (player.x+30 > e.x+5 &&
        player.x+8 < e.x+33 &&
        player.y+55 > e.y+5 &&
        player.y+5 < e.y+60) {
      playing = false;
      ctx.fillStyle = "#000b";
      ctx.fillRect(0,0,400,600);
      ctx.fillStyle = "#ffd166";
      ctx.textAlign = "center";
      ctx.font = "bold 36px Arial";
      ctx.fillText("GAME OVER",200,270);
      ctx.font = "24px Arial";
      ctx.fillText("Score: "+score,200,315);
      ctx.font = "18px Arial";
      ctx.fillText("Press START to play again",200,355);
      return;
    }

    if (e.y > 610) {
      enemies.splice(i,1);
      score += 10;
      scoreText.textContent = score;
    }
  }

  draw(dt);
  animation = requestAnimationFrame(loop);
}

function bindButton(id, direction) {
  const b = document.getElementById(id);
  b.addEventListener("pointerdown", e => {
    e.preventDefault();
    b.setPointerCapture(e.pointerId);
    if (direction === "left") left = true;
    else right = true;
  });
  function release() {
    if (direction === "left") left = false;
    else right = false;
  }
  b.addEventListener("pointerup",release);
  b.addEventListener("pointercancel",release);
  b.addEventListener("lostpointercapture",release);
}

bindButton("left","left");
bindButton("right","right");

document.addEventListener("keydown", e => {
  if (e.key === "ArrowLeft" || e.key.toLowerCase() === "a")
    left = true;
  if (e.key === "ArrowRight" || e.key.toLowerCase() === "d")
    right = true;
});
document.addEventListener("keyup", e => {
  if (e.key === "ArrowLeft" || e.key.toLowerCase() === "a")
    left = false;
  if (e.key === "ArrowRight" || e.key.toLowerCase() === "d")
    right = false;
});

draw(16);
</script>
</body>
</html>
