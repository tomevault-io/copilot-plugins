## anti-ai-trace

> Bun + TypeScript + sharp

# anti-ai-trace - 反 AI 识别图片清洗工具
Bun + TypeScript + sharp

<directory>
src/ - 核心源码 (1文件: scrub.ts)
inputs/ - 批处理输入目录，放入待清洗图片
outputs/ - 批处理输出目录，清洗后的图片
</directory>

<config>
package.json - 项目配置，scripts: scrub / scrub:batch
tsconfig.json - TypeScript 编译配置，target ESNext + bundler resolution
.gitignore - 忽略 node_modules、dist、inputs/outputs 内容
</config>

## 流水线

RGBA raw → 暗水印像素归零 → JPEG 92 有损中转（毁 LSB/频域签名）→ PNG 输出 → 剥离 PNG ancillary chunks + EXIF/XMP/C2PA 元数据

## 用法

```bash
bun install
bun run scrub                        # 批量处理 inputs/ → outputs/
bun run scrub -- <input> [output]    # 处理单张图片
```

---
> Source: [BellyBook/anti-ai-trace](https://github.com/BellyBook/anti-ai-trace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
