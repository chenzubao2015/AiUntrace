<script>
  import { createEventDispatcher, onDestroy } from "svelte";
  import { extractMetadata, findings, isClean } from "./metadata.js";
  import { cleanImage, formatBytes } from "./clean.js";
  import { createZip } from "./zip.js";
  import Icon from "./Icon.svelte";

  export let files;

  const dispatch = createEventDispatcher();

  /** @type {'scanning'|'review'|'cleaning'|'done'} */
  let stage = "scanning";
  let items = files.map((file, id) => ({
    id,
    file,
    state: "pending", // pending | scanned | cleaning | done | failed
    labels: [],
    clean: false,
    result: null,
  }));
  let zipping = false;
  let cancelled = false;

  $: scanned = items.filter((item) => item.state !== "pending").length;
  $: done = items.filter((item) => item.state === "done" || item.state === "failed").length;
  $: succeeded = items.filter((item) => item.state === "done");
  $: dirty = items.filter((item) => !item.clean).length;

  function patch(id, values) {
    items = items.map((item) => (item.id === id ? { ...item, ...values } : item));
  }

  async function scanAll() {
    for (const item of items) {
      if (cancelled) return;
      try {
        const metadata = await extractMetadata(item.file);
        patch(item.id, {
          state: "scanned",
          clean: isClean(metadata),
          labels: findings(metadata).filter((f) => f.severity !== "info"),
        });
      } catch {
        patch(item.id, { state: "scanned", clean: false, labels: [] });
      }
    }
    stage = "review";
  }

  async function cleanAll() {
    stage = "cleaning";
    for (const item of items) {
      if (cancelled) return;
      patch(item.id, { state: "cleaning" });
      const result = await cleanImage(item.file);
      patch(item.id, { state: result.fallback ? "failed" : "done", result });
    }
    stage = "done";
    dispatch("notify", {
      tone: items.some((i) => i.state === "failed") ? "err" : "ok",
      message: items.some((i) => i.state === "failed")
        ? "部分图片无法重新编码，已自动跳过。"
        : `已成功清除 ${items.length} 张图片的元数据。`,
    });
  }

  async function downloadZip() {
    zipping = true;
    try {
      const used = new Set();
      const entries = [];
      for (const item of succeeded) {
        let name = item.result.filename;
        for (let n = 2; used.has(name); n++) {
          name = item.result.filename.replace(/(\.[^.]+)$/, `-${n}$1`);
        }
        used.add(name);
        const data = new Uint8Array(await (await fetch(item.result.url)).arrayBuffer());
        entries.push({ name, data });
      }
      const url = URL.createObjectURL(createZip(entries));
      const link = document.createElement("a");
      link.href = url;
      link.download = "aiuntrace-cleaned.zip";
      link.click();
      setTimeout(() => URL.revokeObjectURL(url), 10000);
    } catch (error) {
      console.error("ZIP failed:", error);
      dispatch("notify", { tone: "err", message: "打包 ZIP 失败，请逐张下载图片。" });
    } finally {
      zipping = false;
    }
  }

  function reset() {
    cancelled = true;
    dispatch("reset");
  }

  scanAll();

  onDestroy(() => {
    cancelled = true;
    for (const item of items) if (item.result?.url) URL.revokeObjectURL(item.result.url);
  });
</script>

<div class="batch" aria-live="polite">
  <div class="head">
    <h3>
      {#if stage === "scanning"}
        正在扫描 {scanned} / {items.length}…
      {:else if stage === "review"}
        已就绪 {items.length} 张图片，其中 {dirty} 张包含隐藏元数据
      {:else if stage === "cleaning"}
        正在清理 {done} / {items.length}…
      {:else}
        已成功清理 {succeeded.length} / {items.length} 张图片
      {/if}
    </h3>
    <progress
      max={items.length}
      value={stage === "scanning" ? scanned : stage === "review" ? items.length : done}
      aria-label="处理进度"
    ></progress>
  </div>

  <ul class="list">
    {#each items as item (item.id)}
      <li class="item">
        <div class="name">
          <span class="file" title={item.file.name}>{item.file.name}</span>
          <span class="size">{formatBytes(item.file.size)}</span>
        </div>

        <div class="state">
          {#if item.state === "pending"}
            <span class="status info">等待中</span>
          {:else if item.state === "cleaning"}
            <span class="status info">清理中</span>
          {:else if item.state === "done"}
            <span class="status ok">已净除</span>
            <a class="btn btn-ghost dl" href={item.result.url} download={item.result.filename}>
              <Icon name="download" size="sm" /> 下载
            </a>
          {:else if item.state === "failed"}
            <span class="status err">无法重编码</span>
          {:else if item.clean}
            <span class="status ok">无隐藏数据</span>
          {:else}
            {#each item.labels.slice(0, 3) as label}
              <span class="status {label.severity}">{label.label}</span>
            {/each}
            {#if item.labels.length > 3}
              <span class="more">另外 +{item.labels.length - 3} 项</span>
            {/if}
          {/if}
        </div>
      </li>
    {/each}
  </ul>

  <div class="action-bar">
    {#if stage === "done"}
      <button
        type="button"
        class="btn btn-primary btn-block"
        aria-disabled={zipping || succeeded.length === 0}
        on:click={() => !zipping && succeeded.length > 0 && downloadZip()}
      >
        <Icon name="download" size="sm" />
        {zipping ? "正在打包 ZIP…" : `打包下载全部 ${succeeded.length} 张图片 (ZIP)`}
      </button>
      <button type="button" class="btn btn-secondary btn-block" on:click={reset}>
        <Icon name="refresh" size="sm" />
        继续清理更多图片
      </button>
    {:else}
      <button
        type="button"
        class="btn btn-primary btn-block"
        aria-disabled={stage !== "review"}
        on:click={() => stage === "review" && cleanAll()}
      >
        {#if stage === "cleaning"}
          <span class="spinner" aria-hidden="true"></span>
          正在清理…
        {:else}
          <Icon name="shield" size="sm" />
          一键清理全部 {items.length} 张图片
        {/if}
      </button>
      <button type="button" class="btn btn-ghost btn-block" disabled={stage === "cleaning"} on:click={reset}>
        重新选择图片
      </button>
    {/if}
  </div>
</div>

<style>
  .batch {
    display: grid;
    gap: var(--space-4);
  }

  .head {
    display: grid;
    gap: var(--space-2);
  }

  .head h3 {
    font-size: var(--fs-4);
    line-height: var(--lh-4);
    font-weight: 600;
  }

  progress {
    width: 100%;
    height: var(--space-2);
    accent-color: var(--accent);
  }

  .list {
    display: grid;
    gap: var(--space-2);
    list-style: none;
    max-height: calc(var(--space-9) * 5);
    overflow-y: auto;
  }

  .item {
    display: grid;
    gap: var(--space-2);
    padding: var(--space-3);
    border: var(--border-w) solid var(--border);
    border-radius: var(--radius-md);
    background: var(--surface);
  }

  @media (min-width: 768px) {
    .item {
      grid-template-columns: minmax(0, 1fr) auto;
      align-items: center;
    }
  }

  .name {
    display: flex;
    align-items: baseline;
    gap: var(--space-3);
    min-width: 0;
  }

  .file {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    font-weight: 500;
  }

  .size,
  .more {
    color: var(--text-3);
    font-size: var(--fs-1);
    line-height: var(--lh-1);
    white-space: nowrap;
  }

  .state {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: var(--space-3);
  }

  .dl {
    height: var(--space-7);
    padding-inline: var(--space-2);
  }
</style>
