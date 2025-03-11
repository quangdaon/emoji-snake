<script lang="ts">
  import { onMount } from 'svelte';
  import { cells } from './config';

  type Vector = [number, number];

  const rows = 30;
  const columns = rows;
  const snake: Vector[] = $state([[10, 10]]);
  const gameFrameRate = 30;

  let velocity: Vector = [0, 0];

  function advanceGameState() {
    if (!snake.length) return;
    const tail = snake.shift()!;
    const newHead: Vector = [tail[0] + velocity[0], tail[1] + velocity[1]];

    snake.push(newHead);
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
        velocity = [-1, 0];
        break;
      case 'ArrowDown':
        velocity = [1, 0];
        break;
      case 'ArrowLeft':
        velocity = [0, -1];
        break;
      case 'ArrowRight':
        velocity = [0, 1];
        break;
      default:
        bound = false;
    }

    if (bound) event.preventDefault();
  }
</script>

<svelte:document onkeydown={handleControls} />

<div class="grid">
  {#each { length: rows } as _, row}
    <div>
      {#each { length: columns } as _, col}
        {#if snake.find(([x, y]) => y === col && x === row)}
          {cells.snake}
        {:else}
          {cells.empty}
        {/if}
      {/each}
    </div>
  {/each}
</div>
