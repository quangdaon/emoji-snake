<script lang="ts">
  import { onMount } from 'svelte';
  import { cells } from './config';
  import type { GameState } from './types';

  type Vector = [number, number];

  const rows = 30;
  const columns = rows;
  const wrap = false;
  const snake: Vector[] = $state([[10, 10]]);
  const gameFrameRate = $derived(
    Math.max(10 - Math.floor(snake.length / 10), 2)
  );
  const deathFrameRate = 10;

  type Props = {
    gameState: GameState;
    ondeath: () => void;
  };

  let { gameState, ondeath }: Props = $props();
  let frames = $state(0);
  let apple = $state(spawnApple());
  let velocity: Vector = $state([0, 0]);
  const velocityQueue: Vector[] = [];

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
      return;
    }

    if (inSnake(...head) || !isInBounds(...head)) {
      ondeath();
      return;
    }

    snake.pop();
  }

  function advanceDeathState() {
    if (snake.length <= 1) return;

    snake.pop();
  }

  export function setVelocity(target: Vector) {
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
