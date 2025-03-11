<script lang="ts">
  import { onMount } from 'svelte';

  type Vector = [number, number];

  const rows = 30;
  const columns = rows;
  const snake: Vector[] = $state([[10, 10]]);
  const gameFrameRate = 60;

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
</script>

{#each { length: rows } as _, row}
  <div>
    {#each { length: columns } as _, col}
      {#if snake.find(([x, y]) => y === col && x === row)}
        🟡
      {:else}
        🔵
      {/if}
    {/each}
  </div>
{/each}
