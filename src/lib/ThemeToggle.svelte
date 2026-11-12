<script>
  import { theme } from "./theme.js";
  import Icon from "./Icon.svelte";
</script>

<button
  class="theme-toggle"
  type="button"
  on:click={() => theme.toggle()}
  aria-label="切换为{$theme === 'light' ? '深色' : '浅色'}主题"
>
  <Icon name={$theme === "light" ? "moon" : "sun"} size="sm" />
  <span class="text">{$theme === "light" ? "深色" : "浅色"}</span>
</button>

<style>
  .text {
    display: none;
  }

  @media (min-width: 560px) {
    .text {
      display: inline;
    }
  }
</style>
