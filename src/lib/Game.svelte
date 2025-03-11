<script lang="ts">
  import Grid from './Grid.svelte';
  import type { GameState } from './types';

  let gameState: GameState = $state('active');
  let grid: Grid;

  function handleControls(
    event: KeyboardEvent & { currentTarget: EventTarget & Document }
  ) {
    let bound = true;

    switch (event.key) {
      case 'ArrowUp':
      case 'w':
        grid.setVelocity([0, -1]);
        break;
      case 'ArrowDown':
      case 's':
        grid.setVelocity([0, 1]);
        break;
      case 'ArrowLeft':
      case 'a':
        grid.setVelocity([-1, 0]);
        break;
      case 'ArrowRight':
      case 'd':
        grid.setVelocity([1, 0]);
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
</script>

<svelte:document onkeydown={handleControls} />

<Grid bind:this={grid} {gameState} ondeath={() => (gameState = 'dead')} />
