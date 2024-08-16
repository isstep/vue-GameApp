<template>
  <div class="game-container">
    <canvas ref="gameCanvas" width="800" height="400"></canvas>
    <div v-if="gameOver" class="modal">
      <p>Game Over!</p>
      <p>Score: {{ score.toFixed(2) }}</p>
      <button @click="restartGame">Restart</button>
    </div>
    <div class="score">Score: {{ score.toFixed(2) }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive, onBeforeUnmount } from "vue";

const gameCanvas = ref(null);
const context = ref(null);

const player = reactive({
  x: 50,
  y: 300,
  width: 50,
  height: 50,
  velocityX: 0,
  velocityY: 0,
  speed: 5,
  gravity: 0.5,
  jumpStrength: -10,
  grounded: false,
});

const keys = reactive({
  ArrowRight: false,
  ArrowLeft: false,
  Space: false,
});

const fallingObjects = reactive([]);
const objectFrequency = 1000;
const objectSpeed = 3;
const objectSize = 30;
const objectColor = "red";

const score = ref(0);
const gameOver = ref(false);
let gameLoop;
let lastTime = Date.now();

const createFallingObject = () => {
  const x = Math.random() * (800 - objectSize);
  fallingObjects.push({
    x,
    y: 0,
    width: objectSize,
    height: objectSize,
  });
};

const update = () => {
  const now = Date.now();
  const deltaTime = now - lastTime;
  lastTime = now;

  if (keys.ArrowRight) {
    player.velocityX = player.speed;
  } else if (keys.ArrowLeft) {
    player.velocityX = -player.speed;
  } else {
    player.velocityX = 0;
  }

  if (keys.Space && player.grounded) {
    player.velocityY = player.jumpStrength;
    player.grounded = false;
  }

  player.velocityY += player.gravity;

  player.x += player.velocityX;
  player.y += player.velocityY;

  if (player.x < 0) player.x = 0;
  if (player.x + player.width > 800) player.x = 800 - player.width;

  if (player.y + player.height >= 350) {
    player.y = 350 - player.height;
    player.grounded = true;
    player.velocityY = 0;
  }

  fallingObjects.forEach((obj) => {
    obj.y += objectSpeed;

    if (
      obj.x < player.x + player.width &&
      obj.x + obj.width > player.x &&
      obj.y < player.y + player.height &&
      obj.y + obj.height > player.y
    ) {
      gameOver.value = true;
    }
  });

  draw();
  if (!gameOver.value) {
    score.value += deltaTime / 1000;
  }
};

const draw = () => {
  context.value.clearRect(0, 0, 800, 400);

  context.value.fillStyle = "blue";
  context.value.fillRect(player.x, player.y, player.width, player.height);

  context.value.fillStyle = "green";
  context.value.fillRect(0, 350, 800, 50);

  context.value.fillStyle = objectColor;
  fallingObjects.forEach((obj) => {
    context.value.fillRect(obj.x, obj.y, obj.width, obj.height);
  });
};

onMounted(() => {
  context.value = gameCanvas.value.getContext("2d");

  gameLoop = setInterval(update, 1000 / 60); 

  setInterval(createFallingObject, objectFrequency);

  window.addEventListener("keydown", (e) => {
    if (e.code in keys) {
      keys[e.code] = true;
    }
  });

  window.addEventListener("keyup", (e) => {
    if (e.code in keys) {
      keys[e.code] = false;
    }
  });
});

onBeforeUnmount(() => {
  clearInterval(gameLoop);
  window.removeEventListener("keydown", () => {});
  window.removeEventListener("keyup", () => {});
});

const restartGame = () => {
  gameOver.value = false;
  score.value = 0;
  fallingObjects.length = 0;
  player.x = 50;
  player.y = 300;
  startGame();
};

const startGame = () => {
  gameLoop = setInterval(update, 1000 / 60);
  setInterval(createFallingObject, objectFrequency);
};
</script>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100vh;
  background-color: #87ceeb;
  position: relative;
}

.modal {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  padding: 20px;
  border: 1px solid #000;
  border-radius: 5px;
  text-align: center;
}

.score {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 24px;
  color: white;
}
</style>
