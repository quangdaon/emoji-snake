<script lang="ts">
  import '@quangdao/brandkit/scss/core.scss';
  import Game from './lib/Game.svelte';
</script>

<header>
  <h1>Ssssnek</h1>
</header>

<main>
  <Game />
</main>

<footer>
  <p>
    A <a href="https://www.quangdao.com">Quangdao Nguyen</a> catastrophe. All rights
    reserved.
  </p>
  <p>
    <a href="https://github.com/quangdaon/emoji-snake">GitHub Repo</a>
  </p>
</footer>

<style>
  footer p {
    margin: 0.5em 0;
  }
</style>
