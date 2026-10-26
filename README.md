const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const levelEl = document.getElementById('level');
const messageEl = document.getElementById('message');

const gridSize = 20;
const tileCount = canvas.width / gridSize;

let snake = [];
let direction = { x: 1, y: 0 };
let nextDirection = { x: 1, y: 0 };
let food = { x: 0, y: 0 };
let score = 0;
let level = 1;
let gameInterval = null;
let isGameOver = false;

function setMessage(text) {
  messageEl.textContent = text;
}

function resetSnake() {
  snake = [
    { x: 10, y: 10 },
    { x: 9, y: 10 },
    { x: 8, y: 10 },
  ];
  direction = { x: 1, y: 0 };
  nextDirection = { x: 1, y: 0 };
}

function placeFood() {
  let newFood;
  do {
    newFood = {
      x: Math.floor(Math.random() * tileCount),
      y: Math.floor(Math.random() * tileCount),
    };
  } while (snake.some(segment => segment.x === newFood.x && segment.y === newFood.y));

  food = newFood;
}

function updateHud() {
  scoreEl.textContent = score;
  levelEl.textContent = level;
}

function startGame() {
  resetSnake();
  score = 0;
  level = 1;
  isGameOver = false;
  placeFood();
  updateHud();
  clearInterval(gameInterval);
  gameInterval = setInterval(gameTick, 120);
  setMessage('Use arrow keys to play. Press E for a +20 length boost.');
}

function endGame() {
  isGameOver = true;
  clearInterval(gameInterval);
  setMessage('Game over! Press any arrow key to restart.');
}

function growSnake(amount = 1) {
  const tail = snake[snake.length - 1];
  for (let i = 0; i < amount; i += 1) {
    snake.push({ x: tail.x, y: tail.y });
  }
}

function advanceSnake() {
  direction = nextDirection;

  const head = {
    x: snake[0].x + direction.x,
    y: snake[0].y + direction.y,
  };

  const hitWall =
    head.x < 0 ||
    head.x >= tileCount ||
    head.y < 0 ||
    head.y >= tileCount;

  const hitSelf = snake.some(
    (segment, index) => index < snake.length - 1 && segment.x === head.x && segment.y === head.y
  );

  if (hitWall || hitSelf) {
    endGame();
    return;
  }

  snake.unshift(head);

  if (head.x === food.x && head.y === food.y) {
    score += 10;
    level = Math.floor(score / 50) + 1;
    setMessage('Nice! Food collected.');
    placeFood();
  } else {
    snake.pop();
  }

  updateHud();
}

function gameTick() {
  if (isGameOver) return;
  advanceSnake();
}

function drawBoard() {
  ctx.fillStyle = '#020617';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  for (let x = 0; x < tileCount; x += 1) {
    for (let y = 0; y < tileCount; y += 1) {
      ctx.strokeStyle = '#0f172a';
      ctx.strokeRect(x * gridSize, y * gridSize, gridSize, gridSize);
    }
  }

  ctx.fillStyle = '#dc2626';
  ctx.fillRect(food.x * gridSize, food.y * gridSize, gridSize, gridSize);

  snake.forEach((segment, index) => {
    const color = index === 0 ? '#22c55e' : '#16a34a';
    ctx.fillStyle = color;
    ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
  });
}

function loop() {
  drawBoard();
  requestAnimationFrame(loop);
}

window.addEventListener('keydown', event => {
  const key = event.key.toLowerCase();

  if (['arrowup', 'arrowdown', 'arrowleft', 'arrowright'].includes(key)) {
    event.preventDefault();

    if (isGameOver) {
      startGame();
    }

    const mapping = {
      arrowup: { x: 0, y: -1 },
      arrowdown: { x: 0, y: 1 },
      arrowleft: { x: -1, y: 0 },
      arrowright: { x: 1, y: 0 },
    };

    const newDirection = mapping[key];
    const isOpposite =
      newDirection.x === -direction.x && newDirection.y === -direction.y;

    if (!isOpposite) {
      nextDirection = newDirection;
    }

    return;
  }

  if (key === 'e') {
    event.preventDefault();

    if (isGameOver) {
      startGame();
      return;
    }

    const tail = snake[snake.length - 1];
    for (let i = 0; i < 20; i += 1) {
      snake.push({ x: tail.x, y: tail.y });
    }

    score += 20;
    level += 1;
    setMessage('E key activated: +20 snake length!');
    updateHud();
  }
});

startGame();
loop();
