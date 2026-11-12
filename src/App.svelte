<script>
  import { onMount } from "svelte";
  import { theme } from "./lib/theme.js";
  import TopBar from "./lib/TopBar.svelte";
  import Tool from "./lib/Tool.svelte";
  import Removals from "./lib/Removals.svelte";
  import HowItWorks from "./lib/HowItWorks.svelte";
  import Faq from "./lib/Faq.svelte";
  import Footer from "./lib/Footer.svelte";
  import Toast from "./lib/Toast.svelte";
  import HeroDemo from "./lib/HeroDemo.svelte";
  import Trust from "./lib/Trust.svelte";

  let toasts = [];
  let nextId = 0;

  const assurances = [
    {
      title: "纯浏览器本地运行",
      detail: "无需上传、无需排队，没有任何服务器会保存您的图片副本。",
    },
    {
      title: "深度扫描完整文件",
      detail: "全面检测 EXIF、XMP、IPTC 以及承载 C2PA 数据的 JUMBF 容器。",
    },
    {
      title: "检测结果透明可见",
      detail: "在执行任何修改前，清晰列出图片中发现的每一项隐藏信息。",
    },
  ];

  function notify(event) {
    const id = nextId++;
    toasts = [...toasts, { id, ...event.detail }];
    setTimeout(() => dismiss(id), 5000);
  }

  function dismiss(id) {
    toasts = toasts.filter((toast) => toast.id !== id);
  }

  onMount(() => theme.init());
</script>

<TopBar />

<main>
  <div class="wrap">
    <div class="intro">
      <section class="lede">
        <p class="eyebrow">EXIF · GPS · C2PA</p>
        <h1>一键清除照片中的隐藏元数据与内容凭证</h1>
        <p>
          每一张照片都可能包含您的拍摄位置、设备信息以及关于它是如何生成的数字签名记录。AiUntrace 能帮您直观查看这些隐藏信息并将其彻底清除——完全免费，且文件绝不会离开您的设备。
        </p>
        <div class="chip-row">
          <span class="chip">完全免费</span>
          <span class="chip">无需上传</span>
          <span class="chip">免登录</span>
          <span class="chip">开源透明</span>
        </div>
        <a class="btn btn-secondary" href="#tool">立即清理照片</a>
      </section>
      <HeroDemo />
    </div>

    <Tool on:notify={notify} />

    <ul class="assurances">
      {#each assurances as item}
        <li>
          <h2>{item.title}</h2>
          <p>{item.detail}</p>
        </li>
      {/each}
    </ul>

    <div class="stack sections">
      <Removals />
      <HowItWorks />
      <Trust />
      <Faq />
    </div>
  </div>
</main>

<Footer />

<Toast {toasts} {dismiss} />

<style>
  main {
    padding-bottom: var(--space-8);
  }

  .intro {
    display: grid;
    gap: var(--space-4);
    align-items: center;
  }

  @media (min-width: 900px) {
    .intro {
      grid-template-columns: minmax(0, 1.1fr) minmax(0, 1fr);
      gap: var(--space-7);
    }
  }

  .assurances {
    display: grid;
    gap: var(--space-3);
    margin-top: var(--space-5);
    list-style: none;
  }

  @media (min-width: 768px) {
    .assurances {
      grid-template-columns: repeat(3, 1fr);
      gap: var(--space-4);
    }
  }

  .assurances li {
    display: grid;
    gap: var(--space-1);
    padding-left: var(--space-4);
    border-left: var(--border-w) solid var(--border-strong);
  }

  .assurances h2 {
    font-size: var(--fs-2);
    line-height: var(--lh-2);
    font-weight: 600;
  }

  .assurances p {
    color: var(--text-3);
    font-size: var(--fs-1);
    line-height: var(--lh-2);
  }

  .sections {
    margin-top: var(--space-9);
  }
</style>
