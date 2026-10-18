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
        if (!running) return;

    if (!lastTime) lastTime = timestamp;
    const dt = Math.min((timestamp - lastTime) / 1000, 0.04);
    lastTime = timestamp;

    const direction =
      (keys.ArrowRight || keys.d ? 1 : 0) -
      (keys.ArrowLeft || keys.a ? 1 : 0) ||
      touchDirection;

    movePlayer(direction, dt);

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

