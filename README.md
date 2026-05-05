<!DOCTYPE html>
<html lang="ca">
<head>
<meta charset="UTF-8">
<title>Laberint PRO</title>
<style>
  body {
    margin: 0;
    background: #0f172a;
    color: white;
    font-family: Arial;
    text-align: center;
  }
  canvas {
    background: #020617;
    border: 3px solid #38bdf8;
    margin-top: 10px;
  }
  #info {
    margin-top: 10px;
  }
</style>
</head>
<body>

<h1>🧠 Laberint PRO</h1>
<div id="info">
  Temps: <span id="time">0</span> | Nivell: <span id="level">1</span>
</div>

<canvas id="game" width="500" height="500"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let player, ball, goal, walls;
let keys = {};
let level = 1;
let time = 0;

document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

function initLevel() {
  player = { x: 20, y: 20, size: 12, speed: 2.5 };
  ball = { x: 460, y: 460, size: 12, speed: 1 + level * 0.3 };
  goal = { x: 470, y: 470, size: 20 };

  walls = generateMaze();
}

function generateMaze() {
  let w = [];

  // laberint bàsic però diferent cada nivell
  for (let i = 0; i < 8 + level; i++) {
    w.push({
      x: Math.random() * 450,
      y: Math.random() * 450,
      w: 10 + Math.random() * 100,
      h: 10
    });
  }

  for (let i = 0; i < 6 + level; i++) {
    w.push({
      x: Math.random() * 450,
      y: Math.random() * 450,
      w: 10,
      h: 10 + Math.random() * 100
    });
  }

  return w;
}

function movePlayer() {
  let newX = player.x;
  let newY = player.y;

  if (keys["ArrowUp"]) newY -= player.speed;
  if (keys["ArrowDown"]) newY += player.speed;
  if (keys["ArrowLeft"]) newX -= player.speed;
  if (keys["ArrowRight"]) newX += player.speed;

  if (!collides(newX, newY)) {
    player.x = newX;
    player.y = newY;
  } else {
    lose("💥 Has tocat una paret!");
  }
}

function collides(x, y) {
  for (let w of walls) {
    if (x < w.x + w.w &&
        x + player.size > w.x &&
        y < w.y + w.h &&
        y + player.size > w.y) {
      return true;
    }
  }
  return false;
}

function moveBall() {
  let dx = player.x - ball.x;
  let dy = player.y - ball.y;
  let dist = Math.sqrt(dx*dx + dy*dy);

  // moviment més "intel·ligent"
  ball.x += (dx / dist) * ball.speed;
  ball.y += (dy / dist) * ball.speed;
}

function checkGame() {
  let dx = player.x - ball.x;
  let dy = player.y - ball.y;
  let dist = Math.sqrt(dx*dx + dy*dy);

  if (dist < player.size) {
    lose("😈 La bola t'ha atrapat!");
  }

  if (player.x > goal.x && player.y > goal.y) {
    level++;
    initLevel();
  }
}

function lose(msg) {
  alert(msg + " Tornes al nivell 1.");
  level = 1;
  time = 0;
  initLevel();
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // glow efecte jugador
  ctx.shadowColor = "lime";
  ctx.shadowBlur = 10;
  ctx.fillStyle = "lime";
  ctx.fillRect(player.x, player.y, player.size, player.size);

  ctx.shadowBlur = 0;

  // bola
  ctx.fillStyle = "red";
  ctx.beginPath();
  ctx.arc(ball.x, ball.y, ball.size, 0, Math.PI*2);
  ctx.fill();

  // meta
  ctx.fillStyle = "gold";
  ctx.fillRect(goal.x, goal.y, goal.size, goal.size);

  // parets
  ctx.fillStyle = "#38bdf8";
  for (let w of walls) {
    ctx.fillRect(w.x, w.y, w.w, w.h);
  }
}

function updateUI() {
  document.getElementById("time").textContent = Math.floor(time);
  document.getElementById("level").textContent = level;
}

function loop() {
  movePlayer();
  moveBall();
  checkGame();
  draw();
  updateUI();

  time += 0.016;
  requestAnimationFrame(loop);
}

initLevel();
loop();
</script>

</body>
</html>
