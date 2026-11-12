<script>
  import { createEventDispatcher } from "svelte";
  import { ACCEPTED_EXTENSIONS, isAccepted } from "./metadata.js";
  import Icon from "./Icon.svelte";

  const dispatch = createEventDispatcher();

  export let busy = false;

  let dragging = false;
  let fileInput;
  let depth = 0;

  const MAX_FILES = 50;

  function offer(list) {
    const incoming = [...(list || [])].filter(Boolean);
    if (incoming.length === 0) return;

    const accepted = incoming.filter(isAccepted);
    if (accepted.length === 0) {
      const type = incoming[0].type;
      dispatch("reject", {
        message: `AiUntrace 无法读取 ${type || "该文件类型"}。请选择 JPEG、PNG、TIFF、HEIC 或 AVIF 格式的图片。`,
      });
      return;
    }

    const skipped = incoming.length - accepted.length;
    const capped = accepted.slice(0, MAX_FILES);
    const over = accepted.length - capped.length;
    if (skipped > 0 || over > 0) {
      dispatch("reject", {
        message:
          over > 0
            ? `单次最多处理 ${MAX_FILES} 张图片，已自动添加前 ${MAX_FILES} 张。`
            : `已跳过 ${skipped} 个 AiUntrace 不支持的文件。`,
      });
    }
    dispatch("select", { files: capped });
  }

  function onDrop(event) {
    event.preventDefault();
    depth = 0;
    dragging = false;
    if (busy) return;
    offer(event.dataTransfer?.files);
  }

  function onDragEnter(event) {
    event.preventDefault();
    if (busy) return;
    depth += 1;
    dragging = true;
  }

  function onDragLeave(event) {
    event.preventDefault();
    depth = Math.max(0, depth - 1);
    if (depth === 0) dragging = false;
  }

  function onChange(event) {
    offer(event.target.files);
    // Reset so choosing the same file twice still fires a change event.
    event.target.value = "";
  }

  function onPaste(event) {
    if (busy) return;
    const item = [...(event.clipboardData?.items || [])].find((entry) =>
      entry.type.startsWith("image/")
    );
    if (item) offer([item.getAsFile()]);
  }
</script>

<svelte:window
  on:paste={onPaste}
  on:dragover|preventDefault
  on:drop|preventDefault
/>

<button
  type="button"
  class="dropzone"
  data-drag={dragging}
  disabled={busy}
  on:click={() => fileInput.click()}
  on:dragenter={onDragEnter}
  on:dragover|preventDefault
  on:dragleave={onDragLeave}
  on:drop={onDrop}
>
  <span class="glyph"><Icon name="upload" size="lg" /></span>
  <span class="title">
    {dragging ? "松开鼠标开始扫描" : "拖拽图片到此处，或点击选择文件"}
  </span>
  <span class="hint">
    文件绝不上传。支持单张处理，或一次性批量添加最多 50 张图片。
  </span>
  <span class="chip-row formats">
    <span class="chip">JPEG</span>
    <span class="chip">PNG</span>
    <span class="chip">TIFF</span>
    <span class="chip">HEIC</span>
    <span class="chip">AVIF</span>
  </span>
</button>

<p class="paste-hint">
  您也可以直接按 <kbd>Ctrl</kbd>+<kbd>V</kbd> 粘贴剪贴板中的图片。
</p>

<input
  class="visually-hidden"
  type="file"
  multiple
  accept={ACCEPTED_EXTENSIONS}
  bind:this={fileInput}
  on:change={onChange}
  tabindex="-1"
  aria-hidden="true"
/>

<style>
  .formats {
    justify-content: center;
    margin-top: var(--space-2);
  }

  .paste-hint {
    margin-top: var(--space-3);
    text-align: center;
    color: var(--text-3);
    font-size: var(--fs-1);
    line-height: var(--lh-1);
  }

  kbd {
    font-family: var(--font-mono);
    font-size: var(--fs-1);
    color: var(--text-2);
    padding: 0 var(--space-1);
    border: var(--border-w) solid var(--border-strong);
    border-radius: var(--radius-sm);
  }
</style>
