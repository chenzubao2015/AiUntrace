<script>
  import ThemeToggle from "./ThemeToggle.svelte";
  import Logo from "./Logo.svelte";
</script>

<header class="topbar">
  <div class="wrap inner">
    <a class="wordmark" href="/" aria-label="AiUntrace 首页">
      <Logo />
      <span>AiUntrace</span>
    </a>

    <nav class="actions" aria-label="站点导航">
      <a class="btn btn-ghost guides" href="/guides/">使用指南</a>
      <ThemeToggle />
    </nav>
  </div>
</header>

<style>
  .actions {
    display: flex;
    align-items: center;
    gap: var(--space-2);
  }

  .guides {
    height: var(--space-8);
    padding-inline: var(--space-3);
  }

  .guides:hover {
    text-decoration: none;
  }
</style>
