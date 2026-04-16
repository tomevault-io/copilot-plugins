## multilingual

> > **Claude Code 请在每个 Phase 开始前阅读此文件，了解全局上下文和当前进度。**

# PLAN.md — PolyGlot 多语言学习 App 开发计划

> **Claude Code 请在每个 Phase 开始前阅读此文件，了解全局上下文和当前进度。**
> **完成一个 Phase 后，请更新下方的进度标记。**

---

## 项目信息

- **项目名**: PolyGlot
- **平台**: macOS 15+ / iPadOS 18+ / iOS 18+
- **语言**: Swift 6, SwiftUI
- **最低部署版本**: iOS 18.0, macOS 15.0, iPadOS 18.0
- **架构**: MVVM + Service Layer
- **持久化**: SwiftData
- **网络**: URLSession (async/await)
- **第三方依赖**: 无（纯 Apple 原生）

---

## 核心规则（所有 Phase 通用）

### 语言规则
- 支持四种语言: 中文 (zh), 英文 (en), 日语 (ja), 韩语 (ko)
- 用户母语为中文 → 中文永远只作为翻译/释义语言，不做语法分析
- 输入某语言时，输出其余三种语言的分析
  - 中文输入 → 英/日/韩
  - 英文输入 → 中/日/韩
  - 日文输入 → 中/英/韩
  - 韩文输入 → 中/日/英

### TTS 规则
- 中文文本永远不朗读（无论输入还是输出）
- 其他语言的单词、例句、翻译结果均可朗读
- SpeakButton 组件在 language == .chinese 时隐藏

### 日语特殊规则
- 所有日语文本中的汉字需要假名注音（furigana）
- 注音格式: `{漢字|かな}`，由 LLM 生成，客户端解析渲染
- 注音显示受日语水平设置控制（低于用户水平的汉字不注音）
- 适用范围: 词典例句、句子翻译、翻译模式输出中的所有日语文本

### LLM 交互规则
- 所有结构化数据请求 LLM 返回 JSON
- JSON 解析失败时 fallback 显示原始文本 + 重试按钮
- 所有释义、语法说明用中文书写（因为用户母语是中文）
- 例句用目标语言原文

---

## 目录结构

```
PolyGlot/
├── App/
│   └── PolyGlotApp.swift              # @main 入口
├── Models/
│   ├── SupportedLanguage.swift         # 语言枚举
│   ├── AppMode.swift                   # 模式枚举
│   ├── LLMProvider.swift               # LLM 供应商枚举
│   ├── TTSProvider.swift               # TTS 供应商枚举
│   ├── JapaneseProficiency.swift       # 日语水平枚举
│   ├── Settings.swift                  # SwiftData 设置模型
│   ├── WordAnalysisResult.swift        # 词典模式返回结构
│   ├── SentenceAnalysisResult.swift    # 句子模式返回结构
│   └── QueryHistory.swift             # 历史记录模型
├── Services/
│   ├── LLM/
│   │   ├── LLMServiceProtocol.swift
│   │   ├── OpenAIService.swift
│   │   ├── ClaudeService.swift
│   │   ├── GeminiService.swift
│   │   └── LLMManager.swift
│   ├── TTS/
│   │   ├── TTSServiceProtocol.swift
│   │   ├── OpenAITTSService.swift
│   │   ├── AppleLocalTTSService.swift
│   │   └── TTSManager.swift
│   ├── LanguageDetector.swift          # NLLanguageRecognizer 封装
│   └── PromptBuilder.swift             # 各模式的 prompt 生成器
├── ViewModels/
│   ├── DictionaryViewModel.swift
│   ├── SentenceViewModel.swift
│   ├── TranslationViewModel.swift
│   ├── QuestionViewModel.swift
│   └── SettingsViewModel.swift
├── Views/
│   ├── ContentView.swift               # 主导航 (TabView / NavigationSplitView)
│   ├── Dictionary/
│   │   └── DictionaryView.swift
│   ├── Sentence/
│   │   └── SentenceView.swift
│   ├── Translation/
│   │   └── TranslationView.swift
│   ├── Question/
│   │   └── QuestionView.swift
│   ├── Settings/
│   │   └── SettingsView.swift
│   └── Components/
│       ├── SpeakButton.swift
│       ├── LanguageBadge.swift
│       ├── FuriganaText.swift
│       ├── LoadingView.swift
│       ├── ErrorBanner.swift
│       └── CollapsibleSection.swift
├── Utilities/
│   ├── FuriganaParser.swift            # {漢字|かな} 解析器
│   ├── Constants.swift
│   └── Extensions/
│       ├── String+Extensions.swift
│       └── Color+Theme.swift
└── Resources/
    └── jlpt_kanji.json                # JLPT N5~N1 汉字表
```

---

## Phase 进度追踪

> **Claude Code: 完成一个 Phase 后，将 `[ ]` 改为 `[x]` 并记录备注。**

### Phase 0 — 项目脚手架 `[ ]`
- [ ] Xcode 多平台项目创建
- [ ] 核心枚举和协议定义
- [ ] SwiftData Settings 模型
- [ ] 主导航结构 (Tab / Sidebar)
- [ ] Settings 页面（API key 输入、供应商选择、日语水平）
- [ ] 编译通过验证
- 备注:

### Phase 1 — LLM Service 层 `[ ]`
- [ ] LLMServiceProtocol 定义
- [ ] OpenAIService 实现
- [ ] ClaudeService 实现
- [ ] GeminiService 实现
- [ ] LLMManager 统一调度
- [ ] LLMError 错误枚举
- [ ] Settings 页 "Test Connection" 按钮
- [ ] 编译通过验证
- 备注:

### Phase 2 — TTS Service 层 `[ ]`
- [ ] TTSServiceProtocol 定义
- [ ] OpenAITTSService 实现 (mp3 播放)
- [ ] AppleLocalTTSService 实现 (AVSpeechSynthesizer)
- [ ] TTSManager 统一调度
- [ ] SpeakButton 可复用组件（中文自动隐藏）
- [ ] 编译通过验证
- 备注:

### Phase 3 — 语言检测 + 提问模式 `[ ]`
- [ ] LanguageDetector (NLLanguageRecognizer)
- [ ] QuestionView UI
- [ ] QuestionViewModel
- [ ] LLM 请求 → 显示 → 朗读 全管线测试
- 备注:

### Phase 4 — 翻译模式 `[ ]`
- [ ] TranslationView UI（源/目标语言选择、交换按钮）
- [ ] TranslationViewModel
- [ ] 云端 LLM 翻译
- [ ] Apple Translation API 本地翻译
- [ ] 输入/输出 SpeakButton（中文隐藏）
- [ ] 编译通过验证
- 备注:

### Phase 5 — 词典模式 `[ ]`
- [ ] WordAnalysisResult 数据模型 (Codable)
- [ ] PromptBuilder: 词典模式 prompt（根据输入语言动态切换）
- [ ] DictionaryView UI（分语言卡片展示）
- [ ] DictionaryViewModel
- [ ] 英语模板: 释义、例句、音标、词源、近义词、反义词
- [ ] 日语模板: 释义、例句、假名、词源、变形、近/反义词
- [ ] 韩语模板: 释义、例句、发音、词源、变形
- [ ] 中文模板: 仅对应词和释义（无语法分析）
- [ ] 各单词/例句 SpeakButton
- [ ] JSON 解析失败 fallback
- 备注:

### Phase 6 — 句子分析模式 `[ ]`
- [ ] SentenceAnalysisResult 数据模型 (Codable)
- [ ] PromptBuilder: 句子模式 prompt（根据输入语言动态切换）
- [ ] SentenceView UI
- [ ] SentenceViewModel
- [ ] 英语语法分析: 句型、时态、语态、从句、关键短语
- [ ] 日语语法分析: 文型、助词、活用、敬语、句型模式
- [ ] 韩语语法分析: 文型、助词、变形、敬语、句型模式
- [ ] 中文: 仅翻译输出（无语法面板）
- [ ] 输入/输出句子 SpeakButton
- 备注:

### Phase 7 — 日语 Furigana 系统 `[ ]`
- [ ] FuriganaSegment 数据模型
- [ ] FuriganaParser: 解析 `{漢字|かな}` 格式
- [ ] FuriganaText SwiftUI 组件（汉字上方显示假名）
- [ ] jlpt_kanji.json 数据文件（N5 + N4，约 300 字）
- [ ] 日语水平过滤逻辑（根据设置隐藏低级汉字注音）
- [ ] 在词典/句子/翻译模式中集成 FuriganaText
- [ ] 更新 LLM prompt: 要求日语文本附带 furigana 标记
- 备注:

### Phase 8 — UI 打磨 `[ ]`
- [ ] 颜色主题（Light/Dark）+ 语言标识色
- [ ] 统一卡片/圆角/阴影样式
- [ ] LanguageBadge 组件
- [ ] LoadingView 加载动画
- [ ] ErrorBanner 错误提示
- [ ] CollapsibleSection 折叠面板
- [ ] 响应式布局: iPhone 单栏 / iPad Sidebar / Mac Sidebar
- [ ] 输入体验: 回车提交 (Mac)、自动语言检测标签
- [ ] QueryHistory 历史记录（SwiftData）
- 备注:

### Phase 9 — 错误处理与优化 `[ ]`
- [ ] API key 未设置引导
- [ ] 网络状态检测 (NWPathMonitor)
- [ ] LLM 请求 loading + 取消 (Task cancellation)
- [ ] 音频播放队列管理
- [ ] 离线能力: 本地 TTS、本地翻译、缓存查询
- [ ] VoiceOver 支持
- [ ] Dynamic Type 支持
- [ ] 键盘导航 (macOS)
- 备注:

---

## LLM API 快速参考

### OpenAI
```
POST [https://api.openai.com/v1/chat/completions](https://api.openai.com/v1/responses)
Header: Authorization: Bearer {key}
Body: { model: "gpt-5.4", messages: [...], temperature: 0.3 }
Response: choices[0].message.content
```

### Anthropic Claude
```
POST https://api.anthropic.com/v1/messages
Headers: x-api-key: {key}, anthropic-version: 2023-06-01
Body: { model: "claude-sonnet-4-20250514", max_tokens: 4096, system: "...", messages: [...] }
Response: content[0].text
```

### Google Gemini
```
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key={key}
Body: { contents: [{parts: [{text: "..."}]}], systemInstruction: {parts: [{text: "..."}]} }
Response: candidates[0].content.parts[0].text
```

### OpenAI TTS
```
POST https://api.openai.com/v1/audio/speech
Header: Authorization: Bearer {key}
Body: { model: "tts-1", input: "text", voice: "alloy" }
Response: audio/mpeg binary
```

---

## 已知风险与决策记录

| 编号 | 风险/决策 | 状态 | 说明 |
|------|----------|------|------|
| D-01 | Furigana 渲染方案 | 待定 | 方案A: 纯SwiftUI自定义Layout / 方案B: WKWebView+ruby标签。建议MVP先用A，复杂场景fallback到B |
| D-02 | LLM JSON 可靠性 | 已知风险 | LLM 不保证返回合法 JSON，需要健壮的 fallback |
| D-03 | JLPT 汉字数据完整性 | 分阶段 | Phase 7 先做 N5+N4，后续补全 N3~N1 |
| D-04 | Apple Translation 可用性 | 需验证 | iOS 18+ 新 API，需要真机测试离线包下载 |
| D-05 | Streaming 响应 | 延后 | Phase 1 先用非流式，后续迭代加 SSE streaming |
| D-06 | 多模型 prompt 差异 | 需关注 | 同一 prompt 在 OpenAI/Claude/Gemini 上 JSON 输出质量可能不同，可能需要微调 |

---

## Claude Code 使用说明

1. **开始新 Phase 前**：先读 PLAN.md 确认前置 Phase 已完成
2. **开发中**：遵循「核心规则」章节的约束
3. **完成后**：更新进度追踪的 checkbox 和备注
4. **遇到问题**：在备注中记录问题和临时方案，不要阻塞后续开发
5. **每个 Phase 结束**：确保项目能编译通过（`xcodebuild` 或 Xcode）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darmau) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
