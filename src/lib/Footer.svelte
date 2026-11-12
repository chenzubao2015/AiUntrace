<script>
  import Logo from "./Logo.svelte";

  const year = new Date().getFullYear();
</script>

<footer class="site-footer">
  <div class="wrap">
    <div class="cols">
      <div>
        <p class="wordmark">
          <Logo />
          <span>AiUntrace</span>
        </p>
        <p class="blurb">
          一款完全在浏览器本地运行的隐私保护工具，可彻底清除图片中的 C2PA 内容凭证、AI 生成标记以及 EXIF 元数据。
        </p>
      </div>

      <nav aria-label="页面章节">
        <h2>本页导航</h2>
        <ul>
          <li><a href="#tool">清理图片</a></li>
          <li><a href="#what-is-removed">清除内容</a></li>
          <li><a href="#how-it-works">使用步骤</a></li>
          <li><a href="#trust">隐私验证</a></li>
          <li><a href="#faq">常见问题</a></li>
        </ul>
      </nav>

      <nav aria-label="使用指南">
        <h2>隐私指南</h2>
        <ul>
          <li><a href="/guides/remove-gps-from-photo/">清除照片中的 GPS 定位</a></li>
          <li><a href="/guides/remove-exif-data-iphone/">在 iPhone 上清除 EXIF</a></li>
          <li><a href="/guides/remove-exif-data-android/">在 Android 上清除 EXIF</a></li>
          <li><a href="/guides/do-social-media-sites-remove-exif-data/">社交媒体会自动清除 EXIF 吗？</a></li>
          <li><a href="/guides/remove-exif-data-windows-mac/">在 Windows 与 Mac 上清除 EXIF</a></li>
          <li><a href="/guides/how-to-view-exif-data/">如何查看照片的 EXIF 数据</a></li>
          <li><a href="/guides/photo-metadata-privacy-checklist/">照片元数据隐私自查清单</a></li>
          <li><a href="/guides/what-is-exif-data/">什么是 EXIF 数据？</a></li>
          <li><a href="/guides/what-is-c2pa/">什么是 C2PA 内容凭证？</a></li>
        </ul>
      </nav>
    </div>

    <hr class="divider" />

    <div class="base">
      <p>© {year} AiUntrace. 基于 MIT 协议开源。</p>
      <p class="claims">
        <span class="status ok">无需上传</span>
        <span class="status ok">无需注册</span>
        <span class="status ok">无统计追踪</span>
      </p>
    </div>
  </div>
</footer>

<style>
  .cols {
    display: grid;
    gap: var(--space-6);
    margin-bottom: var(--space-6);
  }

  @media (min-width: 560px) {
    .cols {
      grid-template-columns: repeat(2, 1fr);
    }
  }

  @media (min-width: 900px) {
    .cols {
      grid-template-columns: 2fr 1fr 1fr;
      gap: var(--space-6);
    }
  }

  .blurb {
    margin-top: var(--space-3);
    max-width: calc(var(--space-9) * 4);
    color: var(--text-3);
  }

  h2 {
    font-size: var(--fs-1);
    line-height: var(--lh-1);
    font-weight: 600;
    color: var(--text-2);
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-bottom: var(--space-3);
  }

  ul {
    display: grid;
    gap: var(--space-2);
    list-style: none;
  }

  ul a {
    display: inline-flex;
    align-items: center;
    gap: var(--space-2);
    color: var(--text-2);
  }

  ul a:hover {
    color: var(--accent);
    text-decoration: none;
  }

  .base {
    display: grid;
    gap: var(--space-3);
    margin-top: var(--space-5);
  }

  @media (min-width: 560px) {
    .base {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
  }

  .claims {
    display: flex;
    flex-wrap: wrap;
    gap: var(--space-4);
  }
</style>
