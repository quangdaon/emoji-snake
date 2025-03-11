<script lang="ts">
  import { onMount } from 'svelte';
  import { cells } from './config';

  type GameState = 'active' | 'paused' | 'dead';
  type Vector = [number, number];

  const rows = 30;
  const columns = rows;
  const snake: Vector[] = $state([[10, 10]]);
  const gameFrameRate = 10;

  let gameState: GameState = $state('active');
  let apple = $state(spawnApple());
  let velocity: Vector = [0, 0];
  const velocityQueue: Vector[] = [];

  function die() {
    gameState = 'dead';
  }

  function advanceGameState() {
    if (!snake.length) return;

    if (velocityQueue.length) {
      velocity = velocityQueue.shift()!;
    }

    const head = snake[0];
    const newHead: Vector = [head[0] + velocity[0], head[1] + velocity[1]];

    handleCollision(newHead);

    snake.unshift(newHead);
  }

  function handleCollision(head: Vector) {
    if (velocity[0] === 0 && velocity[1] === 0) {
      snake.pop();
      return;
    }

    if (isApple(...head)) {
      apple = spawnApple();
      return;
    }

    if (inSnake(...head) || !isInBounds(...head)) {
      die();
      return;
    }

    snake.pop();
  }

  function advanceDeathState() {
    if (snake.length <= 1) return;

    snake.pop();
  }

  let frames = 0;
  function gameLoop() {
    if (gameState === 'active' && frames % gameFrameRate === 0)
      advanceGameState();

    if (gameState === 'dead' && frames % gameFrameRate === 0)
      advanceDeathState();

    frames++;

    requestAnimationFrame(gameLoop);
  }

  onMount(() => requestAnimationFrame(gameLoop));

  function setVelocity(target: Vector) {
    if (gameState !== 'active') return;
    
    const previousTarget = velocityQueue.length
      ? velocityQueue[velocityQueue.length - 1]
      : velocity;
    if (
      snake.length > 1 &&
      previousTarget[0] + target[0] === 0 &&
      previousTarget[1] + target[1] === 0
    )
      return;

    velocityQueue.push(target);
  }

  function handleControls(
    event: KeyboardEvent & { currentTarget: EventTarget & Document }
  ) {
    let bound = true;

    switch (event.key) {
      case 'ArrowUp':
      case 'w':
        setVelocity([0, -1]);
        break;
      case 'ArrowDown':
      case 's':
        setVelocity([0, 1]);
        break;
      case 'ArrowLeft':
      case 'a':
        setVelocity([-1, 0]);
        break;
      case 'ArrowRight':
      case 'd':
        setVelocity([1, 0]);
        break;
      case 'Escape':
      case 'p':
        togglePause();
        break;
      default:
        bound = false;
    }

    if (bound) event.preventDefault();
  }

  function togglePause() {
    if (gameState === 'active') {
      gameState = 'paused';
      return;
    }

    if (gameState === 'paused') gameState = 'active';
  }

  function spawnApple(): Vector {
    const x = Math.floor(Math.random() * columns);
    const y = Math.floor(Math.random() * rows);

    return inSnake(x, y) ? spawnApple() : [x, y];
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
</script>

<svelte:document onkeydown={handleControls} />

<div class="grid">
  {#each { length: rows } as _, row}
    <div>
      {#each { length: columns } as _, col}
        {#if inSnake(col, row)}
          {#if gameState === 'dead'}
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
