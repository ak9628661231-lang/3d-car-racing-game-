<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>3D Arrow Escape Puzzle</title>

<style>
* {
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
}

body {
  margin: 0;
  background: #10182b;
  color: white;
  font-family: Arial, sans-serif;
  text-align: center;
}

header {
  padding: 15px 8px;
  background: linear-gradient(135deg,#172554,#312e81);
  box-shadow: 0 4px 15px #0008;
}

h1 {
  margin: 0 0 8px;
  font-size: 25px;
  color: #67e8f9;
}

.stats {
  display: flex;
  justify-content: center;
  gap: 18px;
  flex-wrap: wrap;
  font-size: 14px;
}

#game {
  width: min(94vw, 480px);
  margin: 22px auto 12px;
  padding: 12px;
  background: linear-gradient(145deg,#26344e,#131d32);
  border: 2px solid #536581;
  border-radius: 18px;
  box-shadow: 0 12px 30px #0008;
}

#board {
  width: 100%;
  aspect-ratio: 1;
  display: grid;
  gap: 3px;
  padding: 8px;
  background: #0b1220;
  border-radius: 12px;
  perspective: 700px;
}

.cell {
  position: relative;
  min-width: 0;
  min-height: 0;
  background: linear-gradient(145deg,#273951,#17253b);
  border-radius: 5px;
  box-shadow: inset 0 2px 3px #52657a55,
              0 2px 0 #080e1b;
  display: flex;
  align-items: center;
  justify-content: center;
}

.arrow {
  width: 90%;
  height: 90%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: clamp(19px, 6vw, 35px);
  font-weight: 900;
  cursor: pointer;
  user-select: none;
  border-radius: 5px;
  color: var(--c);
  text-shadow: 0 3px 0 #0008, 0 0 8px var(--c);
  filter: drop-shadow(0 3px 1px #0009);
  transition: transform .18s, opacity .18s;
}

.arrow:active {
  transform: scale(.82);
}

.arrow.removing {
  opacity: 0;
  transform: translate(var(--tx),var(--ty)) scale(.2);
}

.controls {
  display: flex;
  justify-content: center;
  gap: 8px;
  flex-wrap: wrap;
  padding: 10px;
}

button {
  border: none;
  border-radius: 10px;
  padding: 12px 16px;
  color: white;
  font-size: 14px;
  font-weight: bold;
  background: linear-gradient(135deg,#2563eb,#7c3aed);
  box-shadow: 0 4px 0 #172554;
  cursor: pointer;
}

button:active {
  transform: translateY(2px);
  box-shadow: none;
}

#message {
  min-height: 30px;
  padding: 8px;
  color: #fde68a;
  font-weight: bold;
}

#help {
  color: #cbd5e1;
  font-size: 13px;
  padding: 0 15px 20px;
  line-height: 1.6;
}

#win {
  display: none;
  position: fixed;
  inset: 0;
  z-index: 5;
  background: #050b18ed;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.winbox {
  width: 100%;
  max-width: 350px;
  background: linear-gradient(145deg,#1e3a5f,#312e81);
  padding: 28px;
  border-radius: 22px;
  border: 2px solid #67e8f9;
  box-shadow: 0 0 35px #22d3ee55;
}

.winbox h2 {
  color: #67e8f9;
  font-size: 30px;
}

footer {
  padding: 15px;
  color: #94a3b8;
  font-size: 12px;
}
</style>
</head>

<body>

<header>
  <h1>🏹 ARROW ESCAPE 3D</h1>
  <div class="stats">
    <span>LEVEL: <b id="level">1</b></span>
    <span>SCORE: <b id="score">0</b></span>
    <span>MOVES: <b id="moves">0</b></span>
  </div>
</header>

<div id="game">
  <div id="board"></div>

  <div id="message">Tap an arrow to help it escape!</div>

  <div class="controls">
    <button onclick="restartLevel()">↻ Restart</button>
    <button onclick="showHint()">💡 Hint</button>
    <button onclick="nextLevel()">Next Level ➜</button>
  </div>
</div>

<div id="help">
  HOW TO PLAY<br>
  Tap an arrow to move it in its pointing direction.<br>
  If another arrow blocks its path, it cannot escape.<br>
  Clear the board to win the level!
</div>

<footer>ARROW ESCAPE PUZZLE • THINK • PLAN • ESCAPE</footer>

<div id="win">
  <div class="winbox">
    <div style="font-size:55px">🏆</div>
    <h2>LEVEL COMPLETE!</h2>
    <p id="winText">Great job!</p>
    <button onclick="nextLevel()">PLAY NEXT LEVEL ➜</button>
  </div>
</div>

<script>
const board = document.getElementById("board");

let level = 1;
let score = 0;
let moves = 0;
let arrows = [];
let history = [];
let locked = false;
let hintIndex = -1;

const colors = [
  "#22d3ee", "#fb7185", "#fbbf24",
  "#a78bfa", "#4ade80", "#f97316",
  "#f472b6", "#60a5fa"
];

function randomColor(i) {
  return colors[i % colors.length];
}

/*
  Each arrow has a starting cell and direction.
  Directions:
  R = right, L = left, U = up, D = down.
*/

function generatePuzzle(n) {
  const list = [];
  const occupied = new Set();

  function add(x, y, d) {
    const key = x + "," + y;
    if (x < 0 || y < 0 || x >= n || y >= n) return false;
    if (occupied.has(key)) return false;

    occupied.add(key);
    list.push({
      x, y, d,
      id: list.length,
      color: randomColor(list.length),
      removed: false
    });
    return true;
  }

  /*
    Make arrows whose paths lead toward the board edge.
    Some arrows start blocked by other arrows.
  */

  const directions = ["R", "D", "L", "U"];
  const count = Math.min(n * n - 3, 8 + level * 3);

  let attempts = 0;

  while (list.length < count && attempts < 2000) {
    attempts++;

    const x = Math.floor(Math.random() * n);
    const y = Math.floor(Math.random() * n);
    const d = directions[Math.floor(Math.random() * 4)];

    add(x, y, d);
  }

  /*
    Arrange arrows in a deterministic, solvable order:
    arrows pointing outward are always permitted to escape.
    Remaining arrows may be blocked temporarily.
  */

  return list;
}

function directionSymbol(d) {
  return {
    R: "➜",
    L: "⬅",
    U: "⬆",
    D: "⬇"
  }[d];
}

function getGridSize() {
  return Math.min(7, 4 + Math.floor((level - 1) / 2));
}

function getRay(a) {
  const cells = [];
  let x = a.x;
  let y = a.y;

  const delta = {
    R: [1, 0],
    L: [-1, 0],
    U: [0, -1],
    D: [0, 1]
  }[a.d];

  while (true) {
    x += delta[0];
    y += delta[1];

    if (x < 0 || y < 0 || x >= getGridSize() || y >= getGridSize()) {
      break;
    }

    cells.push([x, y]);
  }

  return cells;
}

function canEscape(a) {
  const ray = getRay(a);

  for (const [x, y] of ray) {
    const blocker = arrows.find(
      b => !b.removed && b.id !== a.id &&
           b.x === x && b.y === y
    );

    if (blocker) return false;
  }

  return true;
}

function render() {
  const n = getGridSize();

  board.style.gridTemplateColumns = `repeat(${n}, 1fr)`;
  board.innerHTML = "";

  for (let y = 0; y < n; y++) {
    for (let x = 0; x < n; x++) {
      const cell = document.createElement("div");
      cell.className = "cell";

      const a = arrows.find(
        z => !z.removed && z.x === x && z.y === y
      );

      if (a) {
        const el = document.createElement("div");
        el.className = "arrow";
        el.textContent = directionSymbol(a.d);
        el.style.setProperty("--c", a.color);

        el.setAttribute("role", "button");
        el.setAttribute("aria-label", "Arrow " + a.d);

        if (a.id === hintIndex) {
          el.style.background = "#ffffff22";
          el.style.outline = "2px solid white";
        }

        el.onclick = () => tapArrow(a.id);

        cell.appendChild(el);
      }

      board.appendChild(cell);
    }
  }

  document.getElementById("level").textContent = level;
  document.getElementById("score").textContent = score;
  document.getElementById("moves").textContent = moves;
}

function tapArrow(id) {
  if (locked) return;

  const a = arrows.find(z => z.id === id);

  if (!a || a.removed) return;

  if (!canEscape(a)) {
    document.getElementById("message").textContent =
      "🚧 Path blocked! Remove the blocking arrow first.";
    return;
  }

  history.push({
    id: a.id,
    x: a.x,
    y: a.y,
    removed: a.removed,
    score,
    moves
  });

  locked = true;
  moves++;
  score += 10;

  const element = [...document.querySelectorAll(".arrow")]
    .find(el => el.getAttribute("aria-label") === "Arrow " + a.d &&
      el.parentElement.parentElement.querySelector(".arrow") === el);

  const delta = {
    R: [150, 0],
    L: [-150, 0],
    U: [0, -150],
    D: [0, 150]
  }[a.d];

  const el = findArrowElement(a);

  if (el) {
    el.style.setProperty("--tx", delta[0] + "px");
    el.style.setProperty("--ty", delta[1] + "px");
    el.classList.add("removing");
  }

  document.getElementById("message").textContent = "✨ Arrow escaped!";

  setTimeout(() => {
    a.removed = true;
    locked = false;
    hintIndex = -1;
    render();

    if (arrows.every(z => z.removed)) {
      finishLevel();
    }
  }, 190);
}

function findArrowElement(a) {
  const cells = [...board.children];
  const n = getGridSize();
  const index = a.y * n + a.x;
  return cells[index]?.querySelector(".arrow") || null;
}

function finishLevel() {
  score += level * 100;

  document.getElementById("score").textContent = score;
  document.getElementById("winText").textContent =
    "You escaped all arrows! Bonus: " + (level * 100) + " points.";

  document.getElementById("win").style.display = "flex";
}

function createLevel() {
  const n = getGridSize();
  arrows = generatePuzzle(n);
  history = [];
  moves = 0;
  locked = false;
  hintIndex = -1;

  document.getElementById("win").style.display = "none";
  document.getElementById("message").textContent =
    "Tap an arrow to help it escape!";

  render();
}

function restartLevel() {
  createLevel();
}

function nextLevel() {
  level++;
  createLevel();
}

/*
  The hint highlights an arrow that can escape now.
*/

function showHint() {
  const a = arrows.find(z => !z.removed && canEscape(z));

  if (!a) {
    document.getElementById("message").textContent =
      "No available move! Restart this level.";
    return;
  }

  hintIndex = a.id;
  document.getElementById("message").textContent =
    "💡 Try the highlighted arrow!";

  render();
}

createLevel();
</script>

</body>
</html>
