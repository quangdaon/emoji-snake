<script lang="ts">
  import Grid from './Grid.svelte';
  import MobileControls from './MobileControls.svelte';
  import type { GameState } from './types';
  const mobileThreshold = 600;
  const isMobile = window.innerWidth < mobileThreshold;

  let gameState: GameState = $state('active');
  let grid: Grid;
  let score = $state(0);

  function handleControls(
    event: KeyboardEvent & { currentTarget: EventTarget & Document }
  ) {
    let bound = true;

    switch (event.key) {
      case 'ArrowUp':
      case 'w':
        grid.setDirection('up');
        break;
      case 'ArrowDown':
      case 's':
        grid.setDirection('down');
        break;
      case 'ArrowLeft':
      case 'a':
        grid.setDirection('left');
        break;
      case 'ArrowRight':
      case 'd':
        grid.setDirection('right');
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

<h3>Your Score: {score}</h3>

<Grid
  bind:this={grid}
  {gameState}
  {isMobile}
  onDied={() => (gameState = 'dead')}
  onScored={() => score++}
/>

{#if isMobile}
  <MobileControls
    onUpPressed={() => grid.setDirection('up')}
    onDownPressed={() => grid.setDirection('down')}
    onLeftPressed={() => grid.setDirection('left')}
    onRightPressed={() => grid.setDirection('right')}
    onPausePressed={togglePause}
  />
{/if}
