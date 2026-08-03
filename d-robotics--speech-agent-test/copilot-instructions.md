## speech-agent-test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

RDK Speech Agent 自动化测试框架。SSH 到 RDK 设备启动 TTS / Audio / ASR / LLM 服务，采集日志、解析性能数据、生成 Allure 报告。

## 常用命令

```bash
# 运行测试
python3 main.py                          # 全部模块，默认 X5 + 4mic
python3 main.py --device X5_Module --mic 6 --filter "Audio"

# 生成报告
python3 scripts/generate_report.py                        # 生成到 reports/<ts>/allure-results/
python3 scripts/generate_report.py --serve                # 生成 + 后台挂起 http://localhost:8080
python3 scripts/generate_report.py --filter asr           # 只生成指定模块
python3 scripts/generate_report.py --list                 # 列出所有报告

# 单脚本
python3 scripts/speech_tts_launch.py --device X5 --loop 3
python3 scripts/speech_audio_launch.py --device X5 --mic 6
python3 scripts/speech_asr_launch.py --device X5 --mic 6
python3 scripts/speech_llm_launch.py --device X5 --mic 6

# 录音/播放
python3 scripts/speech_record_play.py --device X5 --mic 6 loopback /root/wav/1.wav --volume 20

# 关闭后台报告
kill -9 $(lsof -ti :8080)
```

## 架构

### 数据流

```
main.py → subprocess → speech_*_launch.py → SSH → RDK 设备 (ROS2)
                                    ↓
                              log/<device>/<mic>mic/<module>/
                              table/<device>/<mic>mic/<module>/
                                    ↓
                        generate_report.py → reports/<ts>/allure-results/
                                    ↓
                              allure generate → 静态 HTML
```

### 目录结构约定

- `log/<device>/<mic>mic/<module>/` — 服务日志 + 脚本日志 + PCM
- `table/<device>/<mic>mic/<module>/` — CSV 数据
- `wav/<device>/<mic>/` — 录音文件
- TTS 无 mic 层级: `log/<device>/tts/`, `table/<device>/tts/`
- Record: `log/<device>/<mic>mic/record/`

### 关键模块

- **analyze_log.py**: 增量日志解析器，`feed(line)` 实时解析。TTSLogParser 解析合成结果，ASRLogParser 解析识别会话，LLMLogParser 继承 ASR 叠加 LLM 问答。WakeupLogParser/CmdWordLogParser 解析唤醒/命令词。
- **launch_util.py**: `SSHSession` 封装 sshpass+ssh，`run_command()` 返回 Popen 进程可流式读 stdout。
- **monitor.py**: `DeviceMonitor` 远端 top 性能采样，输出 CSV 表格。
- **generate_report.py**: 扫描 log/table 数据生成 Allure JSON。按 `device/mic/module` 层级组织标签，数据表格嵌入 `descriptionHtml`，性能数据按进程生成独立折线图。

### CSV 格式

CSV 分 `=== Section Name ===` 段，表头以 `#` 开头。各模块段名不同（Wakeup Events / ASR Sessions / LLM Q&A / Performance 等）。`generate_report.py` 的 `_csv_to_html` 动态解析这些段生成 HTML 表格。

### 设备配置

`config/device.yaml` 中每个设备一个 key，`--device` 参数直接对应 key 名。

## CSV 手动编辑

用户常需手动在 Wakeup Events 或 ASR Sessions 段中插入 `speech` 列，并填充播放的语音内容。示例模式：

- "土豆土豆，5次; 向左转/向右转...各2次" = 5+10=15行的循环块
- "土豆土豆，今天天气怎么样？/明天会下雨吗？..." = 每5行一组的循环

操作要点：保留原有数据，在 keyword 或 text 列前插入 speech，重新编号 `#` 列，删除重复序号行。

---
> Source: [D-Robotics/speech_agent_test](https://github.com/D-Robotics/speech_agent_test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
