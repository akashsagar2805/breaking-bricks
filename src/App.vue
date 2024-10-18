<template>
  <div>
    <h1>Breaking Bricks!</h1>
    <button @click="toggleRules" class="btn rules-btn">Show Rules</button>

    <div :class="['rules', { show: showRules }]">
      <h2>How To Play:</h2>
      <p>
        Use your right and left keys to move the paddle to bounce the ball up
        and break the blocks.
      </p>
      <p>If you miss the ball, your score and the blocks will reset.</p>
      <button @click="toggleRules" class="btn">Close</button>
    </div>

    <canvas ref="canvasRef" width="800" height="600"></canvas>
  </div>
</template>


<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue';
import './assets/style.css';

// Reactive data for game state
const showRules = ref(false);
const canvasRef = ref(null);
let ctx;

// Set initial score from localStorage or 0
const highestScore = ref(localStorage.getItem('highestScore') || 0);

// Game properties
const score = ref(0);
const delay = 500;
const brickRowCount = 9;
const brickColumnCount = 5;

// Ball properties
const ball = reactive({
  x: 0,
  y: 0,
  size: 10,
  speed: 4,
  dx: 4,
  dy: -4,
  visible: true
});

// Paddle properties
const paddle = reactive({
  x: 0,
  y: 0,
  w: 90,
  h: 10,
  speed: 8,
  dx: 0,
  visible: true
});

// Brick properties
const brickInfo = {
  w: 70,
  h: 20,
  padding: 10,
  offsetX: 45,
  offsetY: 60,
  visible: true
};

// Bricks setup
const bricks = reactive([]);
for (let i = 0; i < brickRowCount; i++) {
  bricks[i] = [];
  for (let j = 0; j < brickColumnCount; j++) {
    const x = i * (brickInfo.w + brickInfo.padding) + brickInfo.offsetX;
    const y = j * (brickInfo.h + brickInfo.padding) + brickInfo.offsetY;
    bricks[i][j] = { x, y, ...brickInfo };
  }
}

// Lifecycle: onMounted to set up the canvas and start game logic
onMounted(() => {
  ctx = canvasRef.value.getContext('2d');
  ball.x = canvasRef.value.width / 2;
  ball.y = canvasRef.value.height / 2;
  paddle.x = canvasRef.value.width / 2 - 40;
  paddle.y = canvasRef.value.height - 20;

  // Start the game loop
  update();

  // Set up event listeners
  window.addEventListener('keydown', keyDown);
  window.addEventListener('keyup', keyUp);
});

// Cleanup event listeners on component unmount
onUnmounted(() => {
  window.removeEventListener('keydown', keyDown);
  window.removeEventListener('keyup', keyUp);
});

// Game consts

const drawBall = () => {
  ctx.beginPath();
  ctx.arc(ball.x, ball.y, ball.size, 0, Math.PI * 2);
  ctx.fillStyle = ball.visible ? '#42b883' : 'transparent';
  ctx.fill();
  ctx.closePath();
}

// Method to toggle the rules' visibility
const toggleRules = () => {
  showRules.value = !showRules.value;
}

const drawPaddle = () => {
  ctx.beginPath();
  ctx.rect(paddle.x, paddle.y, paddle.w, paddle.h);
  ctx.fillStyle = paddle.visible ? '#42b883' : 'transparent';
  ctx.fill();
  ctx.closePath();
}

const drawScore = () => {
  ctx.font = '20px Arial';
  ctx.fillText(`Score: ${score.value}`, canvasRef.value.width - 100, 30);
}

const drawBricks = () => {
  bricks.forEach(column => {
    column.forEach(brick => {
      ctx.beginPath();
      ctx.rect(brick.x, brick.y, brick.w, brick.h);
      ctx.fillStyle = brick.visible ? '#42b883' : 'transparent';
      ctx.fill();
      ctx.closePath();
    });
  });
}

const movePaddle = () => {
  paddle.x += paddle.dx;

  if (paddle.x + paddle.w > canvasRef.value.width) {
    paddle.x = canvasRef.value.width - paddle.w;
  }

  if (paddle.x < 0) {
    paddle.x = 0;
  }
}

const moveBall = () => {
  ball.x += ball.dx;
  ball.y += ball.dy;

  if (ball.x + ball.size > canvasRef.value.width || ball.x - ball.size < 0) {
    ball.dx *= -1;
  }

  if (ball.y + ball.size > canvasRef.value.height || ball.y - ball.size < 0) {
    ball.dy *= -1;
  }

  if (ball.x - ball.size > paddle.x && ball.x + ball.size < paddle.x + paddle.w && ball.y + ball.size > paddle.y) {
    ball.dy = -ball.speed;
  }

  bricks.forEach(column => {
    column.forEach(brick => {
      if (brick.visible) {
        if (ball.x - ball.size > brick.x && ball.x + ball.size < brick.x + brick.w && ball.y + ball.size > brick.y && ball.y - ball.size < brick.y + brick.h) {
          ball.dy *= -1;
          brick.visible = false;
          increaseScore();
        }
      }
    });
  });

  if (ball.y + ball.size > canvasRef.value.height) {
    showAllBricks();
    score.value = 0;
  }
}

const increaseScore = () => {
  score.value++;

  if (score.value % (brickRowCount * brickColumnCount) === 0) {
    ball.visible = false;
    paddle.visible = false;

    setTimeout(() => {
      showAllBricks();
      resetGame();
    }, delay);
  }
}

// const to store the highest score in localStorage
const updateHighestScore = () => {
  if (game.score > highestScore.value) {
    highestScore.value = game.score;
    localStorage.setItem('highestScore', highestScore.value);
  }
}

const resetGame = () => {
  score.value = 0;
  paddle.x = canvasRef.value.width / 2 - 40;
  paddle.y = canvasRef.value.height - 20;
  ball.x = canvasRef.value.width / 2;
  ball.y = canvasRef.value.height / 2;
  ball.visible = true;
  paddle.visible = true;
  updateHighestScore();
}

const showAllBricks = () => {
  bricks.forEach(column => {
    column.forEach(brick => {
      brick.visible = true;
    });
  });
}

const draw = () => {
  ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
  drawBall();
  drawPaddle();
  drawScore();
  drawBricks();
}

const update = () => {
  movePaddle();
  moveBall();
  draw();
  requestAnimationFrame(update);
}

const keyDown = (e) => {
  if (e.key === 'Right' || e.key === 'ArrowRight') {
    paddle.dx = paddle.speed;
  } else if (e.key === 'Left' || e.key === 'ArrowLeft') {
    paddle.dx = -paddle.speed;
  }
}

const keyUp = (e) => {
  if (e.key === 'Right' || e.key === 'ArrowRight' || e.key === 'Left' || e.key === 'ArrowLeft') {
    paddle.dx = 0;
  }
}
</script>
