<script lang="ts">
  import { onMount } from 'svelte';
  import { cells } from './config';
  import type { GameState, Vector } from './types';
  import { desktopGameOver, mobileGameOver } from './screens';

  type Props = {
    gameState: GameState;
    isMobile: boolean;
    onDied: () => void;
    onScored: () => void;
  };

  const { gameState, isMobile, onDied, onScored }: Props = $props();

  const rows = isMobile ? 12 : 30;
  const columns = rows;
  const wrap = false;
  const snake: Vector[] = $state([getStart()]);
  const gameFrameRate = $derived(
    isMobile
      ? Math.max(20 - 2 * Math.floor(snake.length / 10), 4)
      : Math.max(10 - Math.floor(snake.length / 10), 2)
  );
  const deathFrameRate = 60;

  let frames = $state(0);
  let apple = $state(spawnApple());
  let velocity: Vector = $state([0, 0]);
  let showGameOver = $state(false);
  const velocityQueue: Vector[] = [];

  const deathScreenModel = isMobile ? mobileGameOver : desktopGameOver;
  const deathScreenCells = deathScreenModel
    .trim()
    .split(/[\n\r]+/g)
    .map((e) => e.split(''))
    .flat()
    .map((e) => (e === '⭕' ? null : e));

  const directions = {
    up: [0, -1] as Vector,
    down: [0, 1] as Vector,
    left: [-1, 0] as Vector,
    right: [1, 0] as Vector,
  };

  onMount(() => requestAnimationFrame(gameLoop));

  function gameLoop() {
    if (gameState === 'active' && frames % gameFrameRate === 0)
      advanceGameState();

    if (gameState === 'dead' && frames % deathFrameRate === 0)
      advanceDeathState();

    frames++;

    requestAnimationFrame(gameLoop);
  }

  function advanceGameState() {
    if (!snake.length) return;

    if (velocityQueue.length) {
      velocity = velocityQueue.shift()!;
    }

    const head = snake[0];
    const newHead: Vector = calculateNextHead(head);

    handleCollision(newHead);

    snake.unshift(newHead);
  }

  function calculateNextHead([headX, headY]: Vector): Vector {
    const [velocityX, velocityY] = velocity;
    let x = headX + velocityX;
    let y = headY + velocityY;

    if (!wrap) return [x, y];

    x = (x + columns) % columns;
    y = (y + rows) % rows;

    return [x, y];
  }

  function handleCollision(head: Vector) {
    if (velocity[0] === 0 && velocity[1] === 0) {
      snake.pop();
      return;
    }

    if (isApple(...head)) {
      apple = spawnApple();
      onScored();
      return;
    }

    if (inSnake(...head) || !isInBounds(...head)) {
      onDied();
      return;
    }

    snake.pop();
  }

  function advanceDeathState() {
    showGameOver = !showGameOver;
  }

  function setVelocity(target: Vector) {
    if (gameState !== 'active') return;

    const previousTargetVelocity = velocityQueue.length
      ? velocityQueue[velocityQueue.length - 1]
      : velocity;

    if (
      snake.length > 1 &&
      previousTargetVelocity[0] + target[0] === 0 &&
      previousTargetVelocity[1] + target[1] === 0
    )
      return;

    velocityQueue.push(target);
  }

  function spawnApple(): Vector {
    const x = Math.floor(Math.random() * columns);
    const y = Math.floor(Math.random() * rows);

    return inSnake(x, y) ? spawnApple() : [x, y];
  }

  function getStart(): Vector {
    const buffer = 10;
    const x = Math.floor(Math.random() * (columns - buffer * 2)) + buffer;
    const y = Math.floor(Math.random() * (rows - buffer * 2)) + buffer;

    return [x, y];
  }

  function inSnake(x: number, y: number) {
    return snake.find(([snakeX, snakeY]) => snakeX === x && snakeY === y);
  }

  function isApple(x: number, y: number) {
    return apple[0] === x && apple[1] === y;
  }

  function isInBounds(x: number, y: number) {
    return x >= 0 && y >= 0 && x < columns && y < rows;
  }

  export const setDirection = (direction: keyof typeof directions) =>
    setVelocity(directions[direction]);
</script>

<div class="grid">
  {#each { length: rows } as _, row}
    <div>
      {#each { length: columns } as _, col}
        {@const index = row * columns + col}
        {#if showGameOver && deathScreenCells[index] !== null}
          {deathScreenCells[index]}
        {:else if inSnake(col, row)}
          {#if gameState === 'dead' && showGameOver}
            {cells.corpse}
          {:else}
            {cells.snake}
          {/if}
        {:else if isApple(col, row)}
          {cells.apple}
        {:else}
          {cells.empty}
        {/if}
      {/each}
    </div>
  {/each}
</div>

<style>
  .grid {
    margin: 1em 0;
  }
</style>
