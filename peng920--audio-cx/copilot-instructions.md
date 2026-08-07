## audio-cx

> 本目录(`/data/github/audio-cx`,WSL Ubuntu)是一个**视频角色语音提取流水线**,目标是从影视视频中提取特定角色的语音,产出 **TTS 训练数据**(分段 wav + JSONL 清单)。

# Agent 环境说明 — 音频角色提取流水线

本目录(`/data/github/audio-cx`,WSL Ubuntu)是一个**视频角色语音提取流水线**,目标是从影视视频中提取特定角色的语音,产出 **TTS 训练数据**(分段 wav + JSONL 清单)。

## 项目目标

采集并整理某个特定角色的语音素材,用于 **Qwen3-TTS** 模型微调训练。最终输出是符合 Qwen3-TTS 训练格式的音文对数据。

## 执行环境(重要)

### 运行在 WSL2 而非 Windows

所有 Python 脚本在 **WSL2 Ubuntu 的 conda 环境 `audio_extract`** 中运行,**不在 Windows 的 .venv 里**。原因:Windows 下 torch CUDA 下载受阻、torchcodec DLL 装不上;WSL2 有现成的 torch/torchaudio 2.11+cu130 且 GPU 直通正常。

**调用方式(WSL 内):**
```bash
cd /data/github/audio-cx
/root/miniconda3/envs/audio_extract/bin/python scripts/cli.py process -v 01
```
- Python 解释器:`/root/miniconda3/envs/audio_extract/bin/python`(Python 3.13)
- conda 环境 `audio_extract`(torch/torchaudio 2.11 cu130,RTX 4070 12GB)
- 从 Windows 侧可通过 `wsl -d Ubuntu -- bash -lc 'cd /data/github/audio-cx && ...'` 调用

### 中文路径的坑(必读)

**命令行传中文路径会编码错乱。** 所有中文路径**必须在 Python 脚本内部用变量/`__file__` 拼接**,绝不能通过 `wsl bash -c "中文"` 或命令行参数传递。脚本已用 `__file__` 自动定位 config,无需传路径参数。

## 流水线架构(11 阶段)

```
s1 抽音轨(ffmpeg) → s1b 去伴奏(Demucs,GPU) → s1c 去噪(DeepFilterNet3,GPU)
→ s2 转写+情感检测(SenseVoiceSmall,GPU,120s块级仅参考) → s3 说话人分离(pyannote community-1,GPU)
→ s4 声纹比对锁定目标(Resemblyzer) + diarization边界选段 + merge + 逐段短段转写 → s4b 导出审核文件
→ [人工审核] → s5 切片输出 → s6 响度匹配 → s7 导出训练集 → s8 质量校验
```

| 阶段 | 脚本 | 输入 | 输出 | 备注 |
|------|------|------|------|------|
| s1 | pipeline/stages/s1_extract.py | 视频 | work/\<video\>/audio.wav | ffmpeg,16kHz单声道 |
| s1b | pipeline/stages/s1b_separate.py | audio.wav | work/\<video\>/vocals.wav | Demucs htdemucs,config 开关 |
| s1c | pipeline/stages/s1c_denoise.py | vocals.wav | vocals.wav(原地覆盖)+denoise_done.flag | DeepFilterNet 去噪去混响,config 开关 |
| s2 | pipeline/stages/s2_sensevoice.py | vocals.wav(优先) | transcript.json | SenseVoiceSmall,带情感+音频事件检测 |
| s3 | pipeline/stages/s3_diarize.py | vocals.wav(优先) | diarization.json | pyannote community-1,用 .speaker_diarization 取结果 |
| s4 | pipeline/stages/s4_match.py | diarization.json + reference_voice.wav + vocals.wav | target_segments.json | Resemblyzer 锁定说话人 + **diarization 边界选段 + merge + 逐段短段转写**(见决策 5.6) |
| s4b | pipeline/stages/s4b_review.py | target_segments.json + **vocals.wav** | review.wav+review.csv **或** review_full.wav+review_audacity.txt | 审核导出,`--format wav_csv/audacity`,**之后是人工卡点** |
| — | (人工) | review.csv | review.done 标记 | 听review.wav,编辑keep/text |
| s5 | pipeline/stages/s5_slice.py | review.csv(优先)或 target_segments.json + vocals.wav | output/\<video\>/clips/ + manifest.jsonl | 读review.csv时按keep过滤,manifest 含 emotion/events 字段 |
| s6 | pipeline/stages/s6_loudness.py | clips/*.wav | 原地覆盖 + loudness_done.flag | pyloudnorm,ITU-R BS.1770-4,先loudness再限peak |
| s7 | pipeline/stages/s7_export.py | clips + manifest | datasets/{qwen,fish}/ | 插件式导出器(exporters/),自动重采样 |
| s8 | pipeline/stages/s8_validate.py | clips + manifest | quality_report.json + quality.csv | 静音/削波/文本异常/重复检测 |
| 调度 | cli.py / pipeline.py | config.yaml | 串联全流程 + 断点续传 | Typer CLI(推荐) / 旧 argparse(兼容) |

### ASR 引擎切换

默认使用 SenseVoiceSmall。如需切回 whisper:
```yaml
# config.yaml
asr:
  engine: "whisper"   # 改回 whisper
```
两种引擎的输出格式兼容,下游阶段无需改动。

## 已验证的关键技术决策(不要偏离)

这些是实测踩坑后确定的写法,改动前务必理解原因:

### 1. pyannote 用 community-1 模型(不是 3.1)
- `speaker-diarization-3.1` 要拼凑 3 个独立模型且缺 PLDA 文件,**不要用**。
- `speaker-diarization-community-1` 自包含(embedding/segmentation/plda 全在一个 repo),从 ModelScope 下载。
- 路径:`models/pyannote/speaker-diarization-community-1/config.yaml`

### 2. 音频必须用 soundfile 预加载成 waveform dict
pyannote 4.x 默认用 torchcodec 读音频,但统一用 soundfile 预加载更稳(见 `utils.load_audio_as_dict`):
```python
audio_dict = load_audio_as_dict(audio_path)  # {'waveform': (1,time) tensor, 'sample_rate': int}
out = pipeline(audio_dict)
```

### 3. 分离结果用 `.speaker_diarization` 属性
pyannote 4.x 返回 `DiarizeOutput`,**不是** `Annotation`:
```python
out = pipeline(audio_dict)
annotation = out.speaker_diarization  # ← 这个才有 itertracks
```
**不要用** `pipeline.to_annotation(out)`(会报错)。

### 4. 强制离线(全程无 HuggingFace)
HuggingFace 在国内被墙。所有 pyannote/whisper 模型从 **ModelScope** 下载,`utils.py` 顶部已设:
```python
os.environ.setdefault("HF_HUB_OFFLINE", "1")
os.environ.setdefault("TRANSFORMERS_OFFLINE", "1")
```
whisper large-v3 的 model.bin(2.87GB)需手动下载(SSL 大文件不稳),放进 `models/faster-whisper-large-v3/`。

### 5. 情绪声保留策略
TTS 训练需要非语言情绪声(嗯/啊/笑/叹气等)。s5 不过滤无文本片段,而是标 `[non-verbal]` 占位,加 `type: "nonverbal"` 字段,供后续人工标注成 Qwen3-TTS 音频标签(如 `(laughter)` `(sigh)`)。

### 5.5 人工审核卡点(s4b)
声纹比对不 100% 准(相似度接近时可能误判)。s4b 在 s4 后导出 `review.wav`(拼接试听,段间0.5s静音)+ `review.csv`(清单,可编辑 keep/text)。**pipeline 在 s4b 后阻塞**,检测 `review.done` 标记文件存在才继续 s5。
- s5 优先读 review.csv(按 keep 过滤、用修正后的 text),不存在才回退 target_segments.json
- **跳过审核**:两种方式 —— ① `config review.required: false`(pipeline 不阻塞,直通 s5-s8,批量处理用);② 或直接创建 `review.done` 标记(按原始 target_segments 全切)
- review.wav 拼接注意:所有切片必须统一 `-ar 16000 -ac 1` 规格才能用 concat demuxer,否则只拼进第一段
- **review.wav 用 vocals.wav**(去伴奏人声),与最终训练数据同源,无 BGM 干扰
- **两种审核格式**(config `review.format` 或 `--format`):
  - `wav_csv`: 拼接试听(review.wav,段间0.5s静音)+ review.csv(keep/text 可编辑)
  - `audacity`: 完整音频(review_full.wav,原时间轴)+ Audacity 标签(review_audacity.txt,每行 start\tend\ttext)
- s5 读审核文件的优先级: review.csv > review_audacity.txt > target_segments.json
- Audacity 标签解析:每行 `start\tend\ttext`,删除段=删行,所有保留行都切(无 keep 字段)

### 5.6 s4 选段 + 短段逐段转写(s4 核心逻辑,重要)
s4 用 **pyannote 的 diarization segment 边界**选目标说话人片段(不用 whisper/SenseVoice 句子边界),并在 merge 后**对每段单独 SenseVoice 转写**回填文本。这是 v0.1.0 经过多轮调试后的最终方案,改动前务必理解演进过程(详见 `docs/TUNING_NOTES.md` 第 1、3 节):

**演进(踩过的坑,别走回头路):**
1. ❌ **初版**:用 s2 的 transcript(SenseVoice 120s 块级)句子边界选段 → 块级时间戳导致整块入选,混入同块里其他说话人(实测某段 SPEAKER_02 仅占 62%,其余是 SPEAKER_10/18)。
2. ❌ **改进1**:改用 diarization segment 边界(音频纯净 99.9%),但文本仍从 transcript 拿 → ①文本和音频对不上(块级)②同块多段拿同句 → 54% 段重复。
3. ✅ **当前**:diarization segment 边界 + merge 合并碎片 + **merge 后对每段切短音频单独 SenseVoice 转写**(不用 transcript)。重复 54%→8%,文本准确对应每段音频。

**当前 s4 流程**(`s4_match.py`):
1. Resemblyzer 锁定目标说话人(`min_similarity=0.7`)
2. `select_target_sentences`:取目标说话人的 diarization segment 作边界(`min_segment_duration=0.3s` 过滤极短碎片),文本留空
3. `merge_adjacent_segments`:合并相邻同人短句(`merge_max_gap=1.5s`,**调过,见下**;`merge_max_duration=15s`),用 dominant_speaker 检查同人
4. `_transcribe_segments`:对每个 merge 后的段切短音频(vocals.wav),用 SenseVoice(复用 s2 单例,带 `hotword_zh` 热词)单独转写回填 text

**关键参数(调过):**
- `matching.merge_max_gap`:当前 **1.5**(默认曾是 0.5)。pyannote 把连续语音切碎(实测 SPEAKER_02 的 4.6min 切成 189 段),0.5 太严不合并 → review.wav 听起来句子截断。实验:0.5→136段/280s, 1.5→72段/336s(完整自然句), 2.0→跨句。详见 TUNING_NOTES 1.1。
- `matching.min_segment_duration`:`0.3s`,过滤 pyannote 的极短碎片(0.04s 重复幻觉段等)。

**为什么不用 whisper 句子边界**:whisper/SenseVoice 在长音频(120s 块)上时间戳不准,与 pyannote 细切粒度不同,用"重叠最大"挂文本会错配。diarization 边界天然不含他人(同说话人),更干净。

**静音裁剪**(s5 `_trim_silence`):切片后用 Python(soundfile+numpy)按 0.02s 窗口算 RMS,裁掉首尾静音,保留 0.1s 缓冲。阈值 0.02 关键——demucs 去伴奏后残余噪声约 0.005-0.015,必须高于它才能区分静音和人声。config `output.trim_silence` 可关。

### 5.7 审核格式选择(s4b/s5)
config `review.format` 或 s4b `--format`:
- `wav_csv`: review.wav(拼接,段间0.5s静音)+ review.csv(keep/text)。**注意拼接静音会造成"断开"听觉错觉,非真实停顿。**
- `audacity`: review_full.wav(完整原时间轴)+ review_audacity.txt(标签)。听完整音频不会断。
s5 按 format 优先读对应文件。Audacity 工作流:编辑的是**标签**(时间戳+文本),不是音频;导出标签(非wav/aup3)覆盖 review_audacity.txt;s5 从原始 vocals.wav 切片。

### 6. 响度匹配(s6)
TTS 训练要求所有样本响度一致,否则模型学到混乱的音量模式。用 pyloudnorm(FishAudio 同款,ITU-R BS.1770-4)归一化到 -18 LUFS(语音标准)。
**关键:归一化顺序**——必须**先 loudness 归一化,再限 peak 防削波**。反过来(先 peak 后 loudness)会导致短 clip 的 peak 过冲到 1.0 削波。实测:归一化前响度范围 -30~-12 LUFS(差18dB),归一化后统一 -18 LUFS。
**注意:** pyloudnorm 的 `integrated_loudness` 期望输入 shape 是 `(samples, channels)`(soundfile 原始输出),不要 reshape 成 `(channels, samples)`(会触发 "more than five channels" 错误)。短于 0.4 秒(块大小)的片段无法测响度,跳过。

### 7. 多格式训练集导出(s7)
不同 TTS 模型数据格式不同,s7 从同一批已切好+响度匹配的 clips(16kHz)转换:
- **Qwen3-TTS**(24kHz): `datasets/qwen/manifest.jsonl`(集中式清单,每行 `{audio, text, type}`)
- **Fish-Speech**(44.1kHz): `datasets/fish/SPK1/`(分布式,每句一个 `.wav` + 同名 `.lab`)
两套格式都带 s4b 人工审核的修正(keep/text)。
**采样率:** 上游 s1-s6 保持 16kHz(whisper/pyannote 规格)。s7 用 ffmpeg `_resample_copy` 按各模型原生规格重采样(Qwen 24kHz / Fish 44.1kHz)。Qwen3-TTS codec 明确"only support 24kHz",16kHz/48kHz 会崩,故必须 24kHz。Fish 原生 44.1kHz。

## 模型文件

均在 `models/` 下,**不纳入 git**(见 .gitignore):

```
models/
├── funasr/hub/models/iic/SenseVoiceSmall/  # SenseVoiceSmall (ASR+情感, ~897MB)
│   ├── model.pt, config.yaml, configuration.json
│   ├── am.mvn, tokens.json, chn_jpn_yue_eng_ko_spectok.bpe.model
├── faster-whisper-large-v3/                # whisper large-v3 (可选,对比用, 2.87GB)
│   └── model.bin
├── pyannote/speaker-diarization-community-1/
│   ├── config.yaml
│   ├── segmentation/pytorch_model.bin
│   ├── embedding/pytorch_model.bin
│   └── plda/{plda.npz, xvec_transform.npz}
└── DeepFilterNet3_extracted/DeepFilterNet3/  # DeepFilterNet3 去噪模型(用户手动放 zip 后解压)
    ├── config.ini
    └── checkpoints/model_120.ckpt.best
```

## 输出结构(TTS 训练数据)

```
output/<video>/
├── clips/                  s5 切好的单句音频(16kHz)
├── manifest.jsonl          s5 通用清单(含 emotion/audio_events 字段)
├── loudness_done.flag      s6 响度完成标记
├── quality_report.json     s8 质量校验报告
├── quality.csv             s8 问题样本 CSV
├── export_done.flag        s7 导出完成标记
└── datasets/               s7 按模型导出
    ├── qwen/clips/ + manifest.jsonl    Qwen3-TTS (24kHz)
    └── fish/SPK1/*.wav + *.lab         Fish-Speech (44.1kHz)
```

## CLI 使用

### 新版 Typer CLI(推荐)

```
python scripts/cli.py process -v 01           # 全流程
python scripts/cli.py separate 01             # s1→s1b→s1c
python scripts/cli.py transcribe 01           # s2 转写+情感
python scripts/cli.py diarize 01              # s3 说话人分离
python scripts/cli.py match 01                # s4 声纹比对
python scripts/cli.py review 01               # s4b 审核导出
python scripts/cli.py slice 01                # s5→s6
python scripts/cli.py export 01 -m qwen       # s7 导出
python scripts/cli.py validate 01             # s8 质量校验
python scripts/cli.py status                  # 查看进度
python scripts/cli.py clean 01 -s 5           # 清理
```

### 旧版 pipeline.py(向后兼容)

```
python scripts/pipeline.py --video 01 --force-stage 4
```

## 流水线架构(代码层)

```
scripts/
├── cli.py                    Typer CLI, 子命令路由
├── pipeline.py               旧接口, 委托给新架构(向后兼容)
├── pipeline/
│   ├── engine.py             PipelineEngine + build_default_engine 工厂(装配 11 阶段)
│   ├── stage.py              Stage 基类: is_done/run/skip_if 钩子 + order 排序属性
│   ├── config_schema.py      config dataclass 校验(属性+字典双访问)
│   ├── logging_setup.py      统一 logging(console INFO + pipeline.log DEBUG)
│   └── stages/               每个阶段一个文件, 继承 Stage(s1~s8)
├── exporters/
│   ├── base.py               BaseExporter(resample_copy)+ discover_exporters() 自动发现
│   ├── qwen.py / fish.py     各模型导出器(新增模型只需新建文件)
├── utils.py                  公共工具(load_config/get_processing_audio/run_ffmpeg 等)
└── config.yaml               配置
```

- **Stage 钩子**:`skip_if(ctx)->bool`(按 config 跳过,如 demucs.enabled=false)、`order: float`(显式排序,替代魔法表)
- **build_default_engine**:cli.py 和 pipeline.py 共用的引擎工厂,消除重复装配

## 配置(config.yaml + config_schema.py)

config 经 `pipeline/config_schema.py` 的 dataclass 做启动校验:缺键补默认+warning,类型错启动即报清晰错误。返回的 config 对象**同时支持属性访问**(`config.matching.merge_max_gap`)**和字典访问**(`config["matching"]["merge_max_gap"]`),旧代码无需改。调参经验详见 `docs/TUNING_NOTES.md`。

关键开关:
- `asr.engine`:`sensevoice`(默认)/`whisper`(旧,fallback)
- `asr.hotword_zh`:中文热词(标准文本作上下文提示,提准),s4 短段转写用
- `demucs.enabled`:是否启用去伴奏(默认 true)
- `denoise.enabled`:是否启用去噪去混响(默认 true,DeepFilterNet3 已修兼容)
- `output.mode`:`tts`(切片+清单,训练用)或 `concat`(拼大文件,试听用)
- `output.source`:`vocals`(从去伴奏人声切)或 `audio`(从原始音轨切)
- `output.min_duration`/`max_duration`:片段时长过滤(0.3s ~ 30s)
- `matching.min_similarity`:声纹比对阈值(默认 0.7,按实际分布校准)
- `matching.merge_max_gap`:合并同人短句的最大间隔(**当前 1.5**,默认 0.5 太严致截断,见 5.6/TUNING_NOTES 1.1)
- `loudness.enabled`/`target_lufs`:响度匹配开关与目标(默认 -18 LUFS 语音标准)
- `validate.enabled`:是否启用 s8 质量校验(默认 true)
- `export.models`:要导出的模型格式列表(插件式,自动发现 exporters/)

## 重要约束

- **不删除用户文件**:脚本只删自己创建的临时文件(tempdir);删除任何用户文件需用户手动操作。
- **不写 pytest**:流水线正确性靠真实音频验证,不靠 mock 单测。
- **断点续传**:每阶段产物落盘即视为完成,重跑自动跳过(`--force-stage N` 可强制从某阶段重跑)。
- **PipelineEngine 统一调度**:所有阶段通过 Stage 基类接入,按 stage_number 排序执行。

### SenseVoice 集成要点

- **模型路径**: `models/funasr/hub/models/iic/SenseVoiceSmall/`。s2 `get_model` 优先传本地目录路径给 `AutoModel(model=本地路径)`(不传 ModelScope id,否则 funasr 会回 ModelScope 重下 model.pt 超时卡死)。
- **长音频退化(重要坑)**:SenseVoice 是短语音模型,s2 把整段 vocals.wav 按 120s 硬切喂它会**退化**(注意力衰减、漏识、后段乱码)。实测:120s 块产 38 字(乱),30s 块产 97 字(通顺)。**所以 s2 的 transcript 只供 s4 锁定参考,真正的转写在 s4 短段逐段做**(见 5.6)。详见 TUNING_NOTES 2.2。
- **标点切句(s2 diarize 模式)**:SenseVoice 输出整段文本,按中文标点切句,按字符位置比例估算时间戳(块级,不准——这是 s4 不用它赋文本的原因)。
- **情感/事件标签**: 解析 `<|EMO_*|>` 和 `<|event|>` token, 存入 transcript.json。
- **热词**:`asr.hotword_zh` 一段标准中文作上下文提示,s4 短段转写时传入提准。
- **引擎切换**: `asr.engine: "whisper"` 回退到旧 faster-whisper(需保留模型文件)。但短段下 SenseVoice ≈ whisper 且无幻觉,默认 SenseVoice(对比结论见 TUNING_NOTES 2.5)。
- **DeepFilterNet(s1c)兼容**:df 0.5.6 与 torchaudio 2.11 不兼容(后者删了 `torchaudio.backend`),s1c 已注入 shim + 本地模型 + 分块 enhance 解决。详见 TUNING_NOTES 2.1。

## 待办/已知限制

- SenseVoice 情感检测精度有限(大部分标为 unknown),可考虑后续用更大模型或人工修正。
- 情绪声/非语言声标为 `[non-verbal]`,SenseVoice 的 audio_events 字段可自动标注 laughter/breath 等,但精度待验证。
- s2 的 120s 长音频转写退化问题:当前靠 s4 短段转写绕过(s2 transcript 仅参考)。若要 s2 本身产出准确文本,需改短段(参考已回退的 slice 架构,代码在 `feat/silence-slice-refactor` 分支)。
- 单条视频全流程约 12-15 分钟(GPU),后台任务有 10 分钟限制,大批量需分批或用 nohup。
- 详细调参/踩坑记录见 `docs/TUNING_NOTES.md`。

---
> Source: [peng920/audio-cx](https://github.com/peng920/audio-cx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
