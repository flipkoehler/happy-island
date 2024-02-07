<template>
  <div>
    <canvas ref="canvas" tabindex="0"></canvas>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

interface Position {
  x: number;
  y: number;
}

const canvasWidth = 1024;
const canvasHeight = 576;
const movementSpeed = 3;
const initialValueX = -900;
const initialValueY = -830;

const canvas = ref<HTMLCanvasElement | null>(null);
const ctx = ref<CanvasRenderingContext2D | null>(null);

const keys = {
  w: { pressed: false },
  a: { pressed: false },
  s: { pressed: false },
  d: { pressed: false },
};

class Sprite {
  position: Position;
  image: HTMLImageElement;

  constructor({
    position,
    image,
  }: {
    position: Position;
    image: HTMLImageElement;
  }) {
    this.position = position;
    this.image = image;
  }

  draw() {
    ctx.value?.drawImage(this.image, this.position.x, this.position.y);
  }

  updatePosition(x: number, y: number) {
    this.position.x = x;
    this.position.y = y;
  }
}

let background: Sprite;
let playerImage: HTMLImageElement;

const handleKeyDown = (e: KeyboardEvent) => {
  switch (e.key) {
    case "w":
      keys.w.pressed = true;
      break;

    case "a":
      keys.a.pressed = true;
      break;

    case "s":
      keys.s.pressed = true;
      break;

    case "d":
      keys.d.pressed = true;
      break;

    default:
      break;
  }
};

const handleKeyUp = (e: KeyboardEvent) => {
  switch (e.key) {
    case "w":
      keys.w.pressed = false;
      break;

    case "a":
      keys.a.pressed = false;
      break;

    case "s":
      keys.s.pressed = false;
      break;

    case "d":
      keys.d.pressed = false;
      break;

    default:
      break;
  }
};

const animate = () => {
  window.requestAnimationFrame(animate);

  if (canvas.value) {
    ctx.value?.clearRect(0, 0, canvasWidth, canvasHeight);

    background.draw();

    ctx.value?.drawImage(
      playerImage,
      0,
      0,
      playerImage.width / 4,
      playerImage.height,
      canvasWidth / 2 - playerImage.width / 4 / 2,
      canvasHeight / 2 - playerImage.height / 2,
      playerImage.width / 4,
      playerImage.height
    );
  }
  if (keys.w.pressed) {
    background.updatePosition(
      background.position.x,
      background.position.y + movementSpeed
    );
    // background.draw();
  }

  if (keys.s.pressed) {
    background.updatePosition(
      background.position.x,
      background.position.y - movementSpeed
    );
    // background.draw();
  }

  if (keys.a.pressed) {
    background.updatePosition(
      background.position.x + movementSpeed,
      background.position.y
    );
    // background.draw();
  }

  if (keys.d.pressed) {
    background.updatePosition(
      background.position.x - movementSpeed,
      background.position.y
    );
    // background.draw();
  }
};

onMounted(() => {
  const canvasElement = canvas.value;
  const context = canvasElement?.getContext("2d");
  if (canvasElement && context) {
    canvasElement.width = canvasWidth;
    canvasElement.height = canvasHeight;

    canvas.value = canvasElement;
    ctx.value = context;

    document.addEventListener("keydown", handleKeyDown);
    document.addEventListener("keyup", handleKeyUp);

    const backgroundImagePromise = loadImage("./images/happyislandmap.png");
    const playerImagePromise = loadImage("./images/playerDown.png");

    Promise.all([backgroundImagePromise, playerImagePromise]).then(([backgroundImg, playerImg]) => {
      background = new Sprite({
        position: {
          x: initialValueX,
          y: initialValueY,
        },
        image: backgroundImg,
      });
      playerImage = playerImg;
      animate();
    }).catch(error => {
      console.error("Error loading images:", error);
    });
  }
});

onUnmounted(() => {
  document.removeEventListener("keydown", handleKeyDown);
  document.removeEventListener("keyup", handleKeyUp);
});

function loadImage(src: string): Promise<HTMLImageElement> {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.onload = () => resolve(img);
    img.onerror = reject;
    img.src = src;
  });
}
</script>