<script>
  import { formatBytes } from "./clean.js";

  export let result;
  export let previewUrl;

  const removed = [
    "EXIF 与 TIFF 标签",
    "GPS 地理坐标",
    "相机品牌、型号与拍摄参数",
    "拍摄时间戳",
    "XMP、IPTC 与 Photoshop 数据块",
    "C2PA 内容凭证",
    "JUMBF 溯源容器",
    "AI 生成标记",
  ];

  $: delta = Math.round(result.sizeDelta);
</script>

<div class="result">
  <div class="side">
    <p class="cap">清理前</p>
    <figure class="preview">
      <img src={previewUrl} alt="清理前的原始图片" />
    </figure>
  </div>

  <div class="side">
    <p class="cap">
      清理后 <span class="status ok">已净除</span>
    </p>
    <figure class="preview">
      <img src={result.url} alt="清理后的结果图片" />
    </figure>
  </div>

  <div class="detail">
    <div class="rows">
      <div class="row">
        <span class="k">输出文件名</span>
        <span class="v" title={result.filename}>{result.filename}</span>
      </div>
      <div class="row">
        <span class="k">文件大小</span>
        <span class="v">
          {formatBytes(result.size)}
          <span class="delta">
            {delta === 0 ? "无变化" : `${delta > 0 ? "+" : ""}${delta}%`}
          </span>
        </span>
      </div>
    </div>

    {#if result.wasSecondPass}
      <p class="note">
        <span class="status warn">已执行二次深度重编码</span>
        首次重绘后仍检测到少量残留元数据，系统已将图片置于纯白底色上强制重编码为 JPEG 格式（透明通道未保留）。
      </p>
    {/if}

    <ul class="removed">
      {#each removed as item}
        <li><span class="status ok">{item}</span></li>
      {/each}
    </ul>
  </div>
</div>

<style>
  .result {
    display: grid;
    gap: var(--space-4);
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .detail {
    grid-column: 1 / -1;
    display: grid;
    gap: var(--space-4);
  }

  @media (min-width: 900px) {
    .result {
      grid-template-columns: repeat(2, minmax(0, 1fr)) minmax(0, 1fr);
      gap: var(--space-5);
      align-items: start;
    }

    .detail {
      grid-column: auto;
    }
  }

  .side {
    display: grid;
    gap: var(--space-2);
    min-width: 0;
  }

  .cap {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    color: var(--text-3);
    font-size: var(--fs-1);
    line-height: var(--lh-1);
    font-weight: 500;
  }

  .delta {
    font-family: var(--font-ui);
    color: var(--text-3);
    margin-left: var(--space-2);
  }

  .row .v {
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 60%;
  }

  .note {
    color: var(--text-2);
    font-size: var(--fs-1);
    line-height: var(--lh-2);
    display: grid;
    gap: var(--space-1);
  }

  .removed {
    display: grid;
    gap: var(--space-2);
    list-style: none;
  }
</style>
