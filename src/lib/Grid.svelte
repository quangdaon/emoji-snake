<script lang="ts">
  import { onMount } from 'svelte';
  import { cells } from './config';

  type Vector = [number, number];

  const rows = 30;
  const columns = rows;
  const snake: Vector[] = $state([[10, 10]]);
  const gameFrameRate = 10;

  let apple = $state(spawnApple());

  let velocity: Vector = [0, 0];

  function advanceGameState() {
    if (!snake.length) return;
    const head = snake[0];
    const newHead: Vector = [head[0] + velocity[0], head[1] + velocity[1]];

    if (isApple(...newHead)) {
      apple = spawnApple();
    } else {
      snake.pop();
    }

    snake.unshift(newHead);
  }

  let frames = 0;
  function gameLoop() {
    if (frames % gameFrameRate === 0) advanceGameState();
    frames++;
    requestAnimationFrame(gameLoop);
  }

  onMount(() => requestAnimationFrame(gameLoop));

  function handleControls(
    event: KeyboardEvent & { currentTarget: EventTarget & Document }
  ) {
    let bound = true;

    switch (event.key) {
      case 'ArrowUp':
        velocity = [0, -1];
        break;
      case 'ArrowDown':
        velocity = [0, 1];
        break;
      case 'ArrowLeft':
        velocity = [-1, 0];
        break;
      case 'ArrowRight':
        velocity = [1, 0];
        break;
      default:
        bound = false;
    }

    if (bound) event.preventDefault();
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
</script>

<svelte:document onkeydown={handleControls} />

<div class="grid">
  {#each { length: rows } as _, row}
    <div>
      {#each { length: columns } as _, col}
        {#if inSnake(col, row)}
          {cells.snake}
        {:else if isApple(col, row)}
          {cells.apple}
        {:else}
          {cells.empty}
        {/if}
      {/each}
    </div>
  {/each}
</div>
