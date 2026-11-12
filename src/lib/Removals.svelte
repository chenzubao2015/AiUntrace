<script>
  const groups = [
    {
      title: "C2PA 内容凭证",
      detail:
        "由内容真实性倡议（CAI）写入的清单（Manifest）、声明、断言及数字签名。",
    },
    {
      title: "AI 生成标记",
      detail:
        "ChatGPT、DALL·E、Midjourney、Stable Diffusion、Adobe Firefly 及 Google Imagen 等工具留下的标识。",
    },
    {
      title: "JUMBF 容器",
      detail:
        "JPEG 文件内部用于存储溯源与凭证数据的 APP11 与 APP2 数据盒。",
    },
    {
      title: "GPS 地理位置",
      detail:
        "纬度、经度、海拔高度、拍摄方位角以及关联的定位时间戳。",
    },
    {
      title: "相机与设备信息",
      detail: "相机或手机品牌、型号、镜头参数、机身序列号、曝光参数及拍摄设置。",
    },
    {
      title: "时间戳",
      detail: "照片拍摄时间、数字化转换时间以及最后修改时间。",
    },
    {
      title: "XMP 与 IPTC",
      detail: "创作者姓名、图注说明、关键词、版权归属以及软件编辑历史记录。",
    },
    {
      title: "注释与内嵌缩略图",
      detail:
        "用户自定义注释、厂商备注（Maker Notes）以及可能与裁剪后画面不一致的内嵌预览缩略图。",
    },
    {
      title: "PNG 文本数据块",
      detail: "tEXt、iTXt 与 zTXt 文本块（包含 AI 绘图提示词 Prompt 等隐藏文本）。",
    },
  ];
</script>

<section class="section" id="what-is-removed" aria-labelledby="removed-heading">
  <div class="section-head">
    <p class="eyebrow">全面覆盖</p>
    <h2 id="removed-heading">AiUntrace 能清除哪些内容</h2>
    <p>
      通过底层像素重新编码，一次性剥离所有元数据容器，而不是逐个删除已知标签而遗漏未知隐患。
    </p>
  </div>

  <div class="tiles">
    {#each groups as group}
      <article class="tile">
        <h3>{group.title}</h3>
        <p>{group.detail}</p>
      </article>
    {/each}
  </div>
</section>

<style>
  .section {
    scroll-margin-top: calc(var(--topbar-h) + var(--space-4));
  }
</style>
