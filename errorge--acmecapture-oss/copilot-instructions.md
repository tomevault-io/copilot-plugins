## acmecapture-oss

> 给自动化助手的**本地开发约定**（不是产品文档）。

# acmeCapture · agent notes

给自动化助手的**本地开发约定**（不是产品文档）。

## 开发数据根

- 默认：`~/acme-dev`（`./tools/dev.sh` → `ACME_DATA_ROOT`）
- 当前项目：`~/acme-dev/workspace/dev`
- 改 `clothes/**` 种子后，同步到开发目录的 `modules/` / `scripts/` 再测
- 正式包数据根（`~/Library/Application Support/acmeCapture`）勿默认改动

## 文档

- [`docs/README.md`](docs/README.md) — 索引
- 扩展衣服包：[`docs/ops/EXTENDING.md`](docs/ops/EXTENDING.md)
- 子项目：`engine/` · `gui/` · `mcp/` · `runtime/` · `clothes/` · `tools/` README

---
> Source: [errorge/acmeCapture-oss](https://github.com/errorge/acmeCapture-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
