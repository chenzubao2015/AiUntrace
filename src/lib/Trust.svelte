<script>
  import Icon from "./Icon.svelte";

  const checks = [
    {
      icon: "eye",
      title: "检查浏览器网络面板",
      detail:
        "打开浏览器的开发者工具（F12），切换到“网络（Network）”选项卡，然后清理一张图片。您会看到没有任何请求携带您的图片数据，因为根本没有后端服务器。",
    },
    {
      icon: "code",
      title: "代码完全开源透明",
      detail:
        "AiUntrace 基于 MIT 协议开源。核心清理逻辑十分精简清晰，任何人都可以直接审阅源码实现。",
    },
    {
      icon: "scan",
      title: "二次拖入复检验证",
      detail:
        "将清理后的图片重新拖入工具中扫描，结果将显示无任何隐藏数据。事实上，工具在提供下载前都会自动重读复检每一次输出。",
    },
  ];

  const limits = [
    "像素本身不会被篡改，因此基于视觉特征的 AI 检测器或嵌入像素内部的隐形数字水印（如 SynthID）仍可能被识别。",
    "如果您此前已将原图上传至某平台，该平台已保存的元数据记录不会因此消失——请在上传分享前先清理文件。",
    "非 JPEG 格式（以及超大尺寸 PNG）图片会被重新编码为 JPEG 格式，因此文件体积和格式可能会发生轻微变化。",
    "请将本工具用于保护个人隐私，切勿在法律法规、新闻出版或赛事规则要求声明 AI 生成内容的场景下用于隐瞒欺骗。",
  ];
</script>

<section class="section" id="trust" aria-labelledby="trust-heading">
  <div class="section-head">
    <p class="eyebrow">安全与信任</p>
    <h2 id="trust-heading">支持自行验证，坦诚说明能力边界</h2>
    <p>
      隐私工具不应仅靠口头承诺。以下是您可以亲自验证每一项隐私保障的方法，以及 AiUntrace 的能力边界。
    </p>
  </div>

  <div class="tiles">
    {#each checks as item}
      <article class="tile">
        <span class="ico"><Icon name={item.icon} /></span>
        <h3>{item.title}</h3>
        <p>{item.detail}</p>
      </article>
    {/each}
  </div>

  <div class="card limits">
    <h3>AiUntrace 不会做的事</h3>
    <ul>
      {#each limits as line}
        <li>{line}</li>
      {/each}
    </ul>
  </div>
</section>

<style>
  .section {
    scroll-margin-top: calc(var(--topbar-h) + var(--space-4));
  }

  .ico {
    display: inline-flex;
    color: var(--accent);
    margin-bottom: var(--space-2);
  }

  .limits {
    margin-top: var(--space-4);
    padding: var(--space-5);
    display: grid;
    gap: var(--space-3);
  }

  .limits h3 {
    font-size: var(--fs-4);
    line-height: var(--lh-4);
    font-weight: 600;
  }

  .limits ul {
    display: grid;
    gap: var(--space-2);
    padding-left: var(--space-4);
    color: var(--text-2);
  }
</style>
