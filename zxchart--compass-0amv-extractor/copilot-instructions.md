## compass-0amv-extractor

> - 指南针安装目录及其中的 `day.vdat`、`.cde` 文件始终只读；不得修改、移动或重命名。

# Compass 0AMV Extractor rules

- 指南针安装目录及其中的 `day.vdat`、`.cde` 文件始终只读；不得修改、移动或重命名。
- `day.vdat` 是已落盘日期的权威来源；`.cde` 只可补收盘后 OHLC，不得把其 volume/amount 宣称为最终值。
- 不恢复群共享脚本的全文件启发式扫描或 `OHLC > 1000` 过滤。
- 正式数据写到仓库外的可配置目录；不得提交生成的 CSV、PKL、manifest 或日志。
- 公开源码不得包含个人绝对路径、账号信息、聊天记录或真实指南针二进制样本；测试只使用合成字节。
- 修改解析契约时必须运行单元测试和当前真实文件的定向提取，并报告实际覆盖范围。

---
> Source: [ZXcharT/compass-0amv-extractor](https://github.com/ZXcharT/compass-0amv-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
