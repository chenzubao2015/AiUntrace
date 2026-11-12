<script>
  import {
    countEntries,
    findings,
    formatMetadata,
    isClean,
  } from "./metadata.js";
  import { formatBytes } from "./clean.js";
  import Icon from "./Icon.svelte";

  export let file;
  export let metadata;
  export let previewUrl;

  $: clean = isClean(metadata);
  $: list = findings(metadata);
  $: entries = countEntries(metadata);
  $: raw = formatMetadata(metadata);
</script>

<div class="review">
  <figure class="preview">
    <img src={previewUrl} alt="所选文件的预览图" />
  </figure>

  <div class="detail">
    <div class="rows">
      <div class="row">
        <span class="k">文件名</span>
        <span class="v" title={file.name}>{file.name}</span>
      </div>
      <div class="row">
        <span class="k">文件大小</span>
        <span class="v">{formatBytes(file.size)}</span>
      </div>
      <div class="row">
        <span class="k">元数据条目数</span>
        <span class="v">{entries}</span>
      </div>
    </div>

    {#if clean}
      <div class="verdict">
        <span class="status ok">已是纯净状态</span>
        <p>
          {metadata?.message ||
            "未在此图片中发现任何元数据。"}您仍然可以对其进行重绘清理，以确保没有任何隐藏数据残留在未知容器中。
        </p>
      </div>
    {:else}
      <div class="verdict">
        <h3>此图片中隐藏的信息</h3>
        <ul class="found">
          {#each list as item}
            <li>
              <span class="status {item.severity}">{item.label}</span>
              <span class="detail-text">{item.detail}</span>
            </li>
          {/each}
        </ul>
      </div>

      <details class="disclosure">
        <summary>
          查看原始元数据详情
          <span class="caret"><Icon name="chevron" size="sm" /></span>
        </summary>
        <div class="body">
          <pre class="raw">{raw}</pre>
        </div>
      </details>
    {/if}
  </div>
</div>

<style>
  .review {
    display: grid;
    gap: var(--space-4);
  }

  @media (min-width: 900px) {
    .review {
      grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
      gap: var(--space-5);
      align-items: start;
    }
  }

  .detail {
    display: grid;
    gap: var(--space-4);
    min-width: 0;
  }

  .row .v {
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 60%;
  }

  .verdict {
    display: grid;
    gap: var(--space-3);
  }

  .verdict h3 {
    font-size: var(--fs-3);
    line-height: var(--lh-3);
    font-weight: 600;
  }

  .verdict p {
    color: var(--text-2);
    font-size: var(--fs-2);
    line-height: var(--lh-3);
  }

  .found {
    display: grid;
    gap: var(--space-3);
    list-style: none;
  }

  .found li {
    display: grid;
    gap: var(--space-1);
  }

  .detail-text {
    color: var(--text-3);
    font-size: var(--fs-1);
    line-height: var(--lh-1);
    padding-left: var(--space-4);
    overflow-wrap: anywhere;
  }
</style>
