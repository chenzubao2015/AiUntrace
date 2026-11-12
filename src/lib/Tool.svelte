<script>
  import { createEventDispatcher, onDestroy } from "svelte";
  import { fade } from "svelte/transition";
  import { extractMetadata } from "./metadata.js";
  import { cleanImage } from "./clean.js";
  import { motionSafe } from "./theme.js";
  import Dropzone from "./Dropzone.svelte";
  import ReviewPanel from "./ReviewPanel.svelte";
  import ResultPanel from "./ResultPanel.svelte";
  import Icon from "./Icon.svelte";
  import Batch from "./Batch.svelte";

  const dispatch = createEventDispatcher();

  const steps = ["扫描检测", "查看详情", "下载图片"];

  /** @type {'idle'|'scanning'|'review'|'cleaning'|'done'} */
  let stage = "idle";
  let file = null;
  let metadata = null;
  let previewUrl = null;
  let result = null;
  /** Set when more than one image is added; Batch owns that flow. */
  let batchFiles = null;

  $: current = stage === "idle" || stage === "scanning" ? 0 : stage === "done" ? 2 : 1;
  $: busy = stage === "scanning" || stage === "cleaning";

  function stepState(index, active) {
    if (index < active) return "done";
    if (index === active) return "current";
    return "upcoming";
  }

  function release() {
    if (previewUrl) URL.revokeObjectURL(previewUrl);
    if (result?.url) URL.revokeObjectURL(result.url);
    previewUrl = null;
    result = null;
  }

  async function onSelect(event) {
    release();
    if (event.detail.files.length > 1) {
      batchFiles = event.detail.files;
      return;
    }
    file = event.detail.files[0];
    previewUrl = URL.createObjectURL(file);
    metadata = null;
    stage = "scanning";

    try {
      metadata = await extractMetadata(file);
      stage = "review";
    } catch (error) {
      console.error("Scan failed:", error);
      stage = "idle";
      dispatch("notify", {
        tone: "err",
        message: "无法读取该图片，请尝试选择其他文件。",
      });
    }
  }

  async function clean() {
    if (!file) return;
    stage = "cleaning";
    result = await cleanImage(file);
    stage = "done";

    if (result.fallback) {
      dispatch("notify", {
        tone: "err",
        message:
          "该图片无法重新编码，未移除任何数据。请尝试先将其转换为 JPEG 或 PNG 格式。",
      });
    } else {
      dispatch("notify", { tone: "ok", message: "元数据已成功清除。" });
    }
  }

  function reset() {
    release();
    batchFiles = null;
    file = null;
    metadata = null;
    stage = "idle";
  }

  onDestroy(release);
</script>

<section class="card tool" id="tool" aria-labelledby="tool-heading">
  <h2 class="visually-hidden" id="tool-heading">清理图片</h2>

  {#if batchFiles}
    <Batch
      files={batchFiles}
      on:notify={(e) => dispatch("notify", e.detail)}
      on:reset={reset}
    />
  {:else}
  <ol class="stepper">
    {#each steps as label, index}
      <li class="step" data-state={stepState(index, current)}>
        <span class="num" aria-hidden="true">
          {#if stepState(index, current) === "done"}
            <Icon name="check" size="sm" />
          {:else}
            {index + 1}
          {/if}
        </span>
        <span class="label">{label}</span>
      </li>
    {/each}
  </ol>

  <div class="panel" aria-live="polite">
    {#if stage === "idle"}
      <div in:fade={{ duration: motionSafe(200) }}>
        <Dropzone
          {busy}
          on:select={onSelect}
          on:reject={(e) => dispatch("notify", { tone: "err", message: e.detail.message })}
        />
      </div>
    {:else if stage === "scanning"}
      <div class="loading" in:fade={{ duration: motionSafe(200) }}>
        <div class="skeleton block"></div>
        <div class="lines">
          <div class="skeleton line"></div>
          <div class="skeleton line short"></div>
          <div class="skeleton line"></div>
        </div>
        <p class="loading-text">正在读取 {file.name} 的元数据…</p>
      </div>
    {:else}
      <div in:fade={{ duration: motionSafe(200) }}>
        {#if stage === "done" && result}
          <ResultPanel {result} {previewUrl} />
        {:else}
          <ReviewPanel {file} {metadata} {previewUrl} />
        {/if}
      </div>
    {/if}
  </div>

  {#if stage !== "idle" && stage !== "scanning"}
    <div class="action-bar">
      {#if stage === "done" && result}
        {#if result.fallback}
          <button type="button" class="btn btn-primary btn-block" on:click={reset}>
            <Icon name="refresh" size="sm" />
            尝试其他图片
          </button>
        {:else}
          <a
            class="btn btn-primary btn-block"
            href={result.url}
            download={result.filename}
          >
            <Icon name="download" size="sm" />
            下载已净除的图片
          </a>
          <button type="button" class="btn btn-secondary btn-block" on:click={reset}>
            <Icon name="refresh" size="sm" />
            继续清理其他图片
          </button>
        {/if}
      {:else}
        <button
          type="button"
          class="btn btn-primary btn-block"
          aria-disabled={stage === "cleaning"}
          on:click={clean}
        >
          {#if stage === "cleaning"}
            <span class="spinner" aria-hidden="true"></span>
            正在清理…
          {:else}
            <Icon name="shield" size="sm" />
            清除元数据
          {/if}
        </button>
        <button
          type="button"
          class="btn btn-ghost btn-block"
          disabled={stage === "cleaning"}
          on:click={reset}
        >
          重新选择图片
        </button>
      {/if}
    </div>
  {/if}
  {/if}
</section>

<style>
  .tool {
    padding: var(--space-4);
    scroll-margin-top: calc(var(--topbar-h) + var(--space-4));
  }

  @media (min-width: 768px) {
    .tool {
      padding: var(--space-5);
    }
  }

  .panel {
    min-height: calc(var(--space-9) * 2);
  }

  .loading {
    display: grid;
    gap: var(--space-4);
  }

  @media (min-width: 900px) {
    .loading {
      grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
      align-items: start;
    }

    .loading-text {
      grid-column: 1 / -1;
    }
  }

  .block {
    height: calc(var(--space-9) * 2);
  }

  .lines {
    display: grid;
    gap: var(--space-3);
    align-content: start;
  }

  .line {
    height: var(--space-5);
  }

  .line.short {
    width: 60%;
  }

  .loading-text {
    color: var(--text-3);
    font-size: var(--fs-2);
    line-height: var(--lh-2);
    overflow-wrap: anywhere;
  }

  .action-bar {
    margin-top: var(--space-5);
    margin-bottom: calc(var(--space-4) * -1);
    border-bottom-left-radius: var(--radius-md);
    border-bottom-right-radius: var(--radius-md);
  }

  @media (min-width: 768px) {
    .action-bar {
      margin-bottom: 0;
    }
  }
</style>
