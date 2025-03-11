<script lang="ts">
  type Props = {
    onUpPressed: () => void;
    onDownPressed: () => void;
    onLeftPressed: () => void;
    onRightPressed: () => void;
    onPausePressed: () => void;
  };

  const {
    onUpPressed,
    onDownPressed,
    onLeftPressed,
    onRightPressed,
    onPausePressed,
  }: Props = $props();
</script>

<div class="controls">
  <div class="row">
    <button class="button" onclick={onUpPressed}>⬆️</button>
  </div>
  <div class="row">
    <button class="button" onclick={onLeftPressed}>⬅️</button>
    <button class="button" onclick={onPausePressed}>⏯️</button>
    <button class="button" onclick={onRightPressed}>➡️</button>
  </div>
  <div class="row">
    <button class="button" onclick={onDownPressed}>⬇️</button>
  </div>
</div>

<style>
  .row {
    display: flex;
    justify-content: center;
  }

  .button {
    background: none;
    border: none;
    padding: 0;
    margin: 0;
    font-size: 3em;
    display: block;
  }
</style>
