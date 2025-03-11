<script lang="ts">
  import Grid from './Grid.svelte';
  import MobileControls from './MobileControls.svelte';
  import type { GameState } from './types';
  const mobileThreshold = 600;
  const isMobile = window.innerWidth > mobileThreshold;

  let gameState: GameState = $state('active');
  let grid: Grid;
  let score = $state(0);

  const setVelocityUp = () => grid.setVelocity([0, -1]);
  const setVelocityDown = () => grid.setVelocity([0, 1]);
  const setVelocityLeft = () => grid.setVelocity([-1, 0]);
  const setVelocityRight = () => grid.setVelocity([1, 0]);

  function handleControls(
    event: KeyboardEvent & { currentTarget: EventTarget & Document }
  ) {
    let bound = true;

    switch (event.key) {
      case 'ArrowUp':
      case 'w':
        setVelocityUp();
        break;
      case 'ArrowDown':
      case 's':
        setVelocityDown();
        break;
      case 'ArrowLeft':
      case 'a':
        setVelocityLeft();
        break;
      case 'ArrowRight':
      case 'd':
        setVelocityRight();
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

<MobileControls
  onUpPressed={setVelocityUp}
  onDownPressed={setVelocityDown}
  onLeftPressed={setVelocityLeft}
  onRightPressed={setVelocityRight}
  onPausePressed={togglePause}
/>
