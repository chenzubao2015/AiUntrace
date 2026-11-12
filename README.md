# AiUntrace

**清除照片中的隐藏元数据与内容凭证。**

AiUntrace 可清除图片中的 C2PA 内容凭证、AI 生成标记、GPS 地理坐标以及 EXIF 元数据。所有处理完全在浏览器本地运行——文件仅在当前标签页中读取与重绘，绝不上传至任何服务器。

## 支持清除的内容

| 类别 | 详细说明 |
|---|---|
| C2PA 内容凭证 | 内容真实性倡议（CAI）写入的清单、声明、断言与数字签名 |
| AI 生成标记 | ChatGPT、DALL·E、Midjourney、Stable Diffusion、Adobe Firefly、Google Imagen 等生成标识 |
| JUMBF 容器 | JPEG 内部用于承载溯源数据的 APP11 与 APP2 数据盒 |
| GPS 地理位置 | 经度、纬度、海拔、方位角及关联时间戳 |
| 相机与设备信息 | 品牌、型号、镜头、序列号、曝光参数及拍摄设置 |
| 时间戳 | 照片拍摄、数字化及最后修改时间 |
| XMP 与 IPTC | 创作者姓名、说明、关键词、版权信息及编辑历史记录 |
| 注释与缩略图 | 用户注释、厂商备注（Maker Notes）及内嵌预览缩略图 |
| PNG 文本块 | tEXt、iTXt 与 zTXt 数据块（包含 AI 提示词等文本信息） |

支持输入格式：JPEG、PNG、TIFF、HEIC、AVIF。输出格式为 JPEG（小尺寸 PNG 保持为 PNG 格式）。

## 工作原理

检测与清理分为两个独立阶段执行：

1. **元数据检测**（`src/lib/metadata.js`）：对文件进行双重读取——首先通过 `exifr` 解析结构化元数据段，随后直接扫描原始字节流以捕获 `exifr` 未暴露的 C2PA 字符串与 JUMBF 容器，最后合并归类并展示检测结果。
2. **像素重绘清理**（`src/lib/clean.js`）：将图片解码为 Canvas 像素并重新编码输出，一次性剥离所有元数据容器，随后再次读取输出文件验证是否彻底干净；若仍有残留，则执行第二轮白底 JPEG 强制重编码。

## 本地开发与 Cloudflare Pages 静态部署

```bash
npm install
npm run dev      # 启动本地开发服务器
npm run build    # 构建生产环境静态产物至 dist/
npm run preview  # 本地预览生产构建产物
```

### Cloudflare Pages 部署配置
- **框架预设**：Vite（或无框架静态构建）
- **构建命令**：`npm run build`
- **构建输出目录**：`dist`

## 开源协议

MIT License.
