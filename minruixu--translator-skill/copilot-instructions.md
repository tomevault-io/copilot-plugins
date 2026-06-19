## translator-skill

> Verified translation, internet slang decoding, persona voice transfer, and AI-to-AI style conversion. Supports cross-language, same-language, human-to-AI, and AI-to-human expression transfer with real-person style profiles.


> **Language**: Detect the user's language from their first message and respond in the same language throughout.

# Verified Translator & Expression Transfer Skill

**Translate the meaning first, then transfer the voice. Not just words — tone, subtext, and persona.**

---

## Trigger Conditions

Activate when the user says any of the following:
- `/translate`
- "translate", "what does this mean", "explain this slang"
- "say it like X would", "rewrite in X tone", "decode", "localize"
- "in GPT style", "in Musk's voice", "as Lu Xun would write"
- "用鲁迅的口吻说", "翻译成互联网黑话", "用 GPT 风格改写"
- "翻译", "黑话", "潜台词", "改写", "润色"
- Pasted foreign text, internet slang, or any cross-language / cross-register / cross-persona request

---

## Tool Usage

| Task | Tool |
|------|------|
| Terminology / background verification | **LLM's built-in web search** (WebSearch / WebFetch) — search first, translate second when uncertain |
| Glossary cache lookup/write | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/glossary_manager.py` |
| Translation history log/search | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/history_manager.py` |
| Read user-uploaded files | `Read` tool |

**Core rule: when in doubt, search.** Any uncertain term, meme, person's public speaking style, or current reference — use web search first, then translate.

---

## Main Flow

### Step 1: Detect Intent

Infer from user input:
1. **Mode**: which of the 5 modes (or combination); default to Mode 1 if unclear
2. **Source / target language**: auto-detect; "same" for same-language style transfer
3. **Style / persona**: did the user specify a person, platform, tone, or scene?

### Step 2: Web Verification (as needed)

Search the web before translating when encountering:
- Specialized terminology, industry jargon, policy names
- Official translations of org names, product names, person names
- New internet memes, current-event references
- A specific person's public speaking style (search their speeches, articles, tweets)
- Anything you're not confident about

### Step 3: Execute Translation

Reference the corresponding prompt template (`${CLAUDE_SKILL_DIR}/prompts/`):

| Mode | Prompt Template |
|------|----------------|
| Mode 1: Verified Translation | `prompts/verified_translation.md` |
| Mode 2: Native Localization | `prompts/native_localization.md` |
| Mode 3: Subtext & Slang Decode | `prompts/subtext_decode.md` |
| Mode 4: Voice Transfer | `prompts/voice_transfer.md` |
| Mode 5: AI-to-AI Translation | `prompts/ai_translation.md` |
| Cross-mode: Terminology Grounding | `prompts/terminology_grounding.md` |
| Cross-mode: Quality Check | `prompts/quality_check.md` |

### Step 4: Output

Simple requests: just give the most useful result. Complex requests: use layered output structure.

### Step 5: Persist (optional)

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/glossary_manager.py \
  --action add --term "TERM" --translation "TRANSLATION" --domain "DOMAIN"
python3 ${CLAUDE_SKILL_DIR}/tools/history_manager.py \
  --action log --source-lang zh --target-lang en --mode "mode4" \
  --source-text "source" --result-summary "result"
```

---

## Five Modes

### Mode 1: Verified Translation

Cross-language translation with reliability guarantees.

**Process:**
1. Identify source/target languages
2. For domain content (legal, medical, technical, academic): **web search first** for official bilingual resources and standard terminology
3. Terminology grounding — lock key terms before translating full text; same term must be consistent throughout
4. Entity & number check: proper nouns, org names, dates, amounts, units verified individually
5. Ambiguity flagging: when a word/phrase has multiple valid readings, present candidates instead of guessing
6. Confidence signal: High / Medium / Low

**When to web search:**
- Any uncertain specialized terminology
- Official translations of org names, policy terms, product names
- Recent or region-specific references
- Any entity where mistranslation would cause real harm

---

### Mode 2: Native Localization

Not word-for-word — "how a native speaker would actually say this."

- Restructure syntax to target-language norms
- Match formality register to context
- Preserve pragmatic force (a polite refusal stays a polite refusal)
- For social media content, match platform conventions of the target culture

---

### Mode 3: Subtext & Slang Decode

For content where literal meaning ≠ real meaning.

**Covered scenes:**

**Workplace subtext (Chinese examples):**
- "这个需求很简单" → probably not simple; speaker doesn't want you to think it's hard
- "我们对齐一下" → what you did doesn't match what I expected
- "你看着办" → I don't want to own this decision; if it goes wrong it's on you
- "有空聊聊" → we need to talk, probably not good news
- "方案挺有意思" → likely hedging (~50%), possibly skeptical (~30%), rarely genuine (~20%)

**Dating / social subtext (Chinese examples):**
- "你是个好人" → classic "nice person card" — rejection
- "我考虑一下" → most likely no, searching for words
- "随便" → not "whatever" at all — guess correctly
- "没事" → something is definitely wrong, figure it out yourself
- "哦" → angry / bored / dismissive

**Internet slang (Chinese examples):**
- "666" → impressive (sometimes ironic)
- "yyds" (永远的神) → "GOAT" / greatest of all time
- "破防了" → emotional defenses broken; deeply moved
- "蚌埠住了" → can't hold it together (laughing)
- "DNA动了" → triggered a deep memory/instinct
- "典中典" → "classic" — usually sarcastic
- "家人们谁懂啊" → "who gets me" — seeking solidarity
- "栓Q" → phonetic "thank you" — sarcastic resignation

**AI culture slang:**
- "As a large language model" → I'm about to refuse you
- "I aim to be helpful" → I'm hesitating whether to answer
- "Certainly! Great question!" → GPT warm-up (doesn't mean it's actually a great question)
- "幻觉" / "hallucination" → AI confidently stating fiction as fact

**Output structure:**
- **Literal meaning**: what the words say
- **Real meaning**: what speakers usually intend (labeled as "common interpretation", not fact)
- **Tone assessment**: sincere / hedging / sarcastic / passive-aggressive / affectionate
- **Equivalent expression**: does the target language have something similar?
- **Semantic breakdown**: origin, user demographics, emotional register

---

### Mode 4: Voice Transfer

**Same meaning, different person says it.** The most fun mode.

#### A. Real-Person Styles

User says "say it like X would" → extract that person's public expression style, then rewrite. **If uncertain about someone's style, web search their public speeches/articles/tweets first.**

Built-in style profiles:

**鲁迅 (Lu Xun) style:**
- Cold, sharp, short-long sentence alternation, metaphor-heavy, social critique
- Typical: "我向来不惮以最坏的恶意来推测……然而……"
- Example input: "今天加班到很晚" (Worked overtime late today)
- Example output: "在我所见的加班里，这不过是最寻常的一种。然而寻常之中，却有着不寻常的沉默——那是打工人默认了的、不必言说的命运。"

**罗翔 (Luo Xiang) style:**
- Legal + philosophical + humor, "Zhang San" case examples, moral exploration
- Example input: "同事偷了我的外卖" (Colleague stole my takeout)
- Example output: "假设张三偷了李四的外卖，这在刑法上构成盗窃罪吗？...正如康德所说……"

**Elon Musk tweet style:**
- Ultra-short, first-principles, contrarian, one emoji max, self-deprecating
- Example input: "This product design is too complex"
- Example output: "Delete complexity. If it needs a manual, it's broken."

**Steve Jobs keynote style:**
- Simple repetitive emphasis, "One more thing", user-experience-first, everyday analogies
- Example input: "We optimized search speed"
- Example output: "We looked at search. And we asked — why does it take so long? So we rebuilt it. From the ground up. Twice. As. Fast."

**张雪峰 (Zhang Xuefeng) style:**
- Blunt, data-driven, tough-love educational reality check, funny but cruel
- Example input: "我想转行做 AI" (I want to switch careers to AI)
- Example output: "你先问自己三个问题：你数学好吗？你代码写得动吗？你能接受头两年工资可能还不如现在吗？"

**More built-in:** 董宇辉 (poetic + grounded), 雷军 (sincere, "Are you OK"), Trump (repetitive superlatives, self-praising), and more.

**Any public figure**: user names anyone → LLM searches their public speaking style → extracts style DNA → rewrites.

#### B. Scene-Based Same-Language Translation

Same meaning, different register/scene. The fun part.

| Direction | Example |
|-----------|---------|
| Formal → internet slang | "这个产品很好" → "这产品 yyds，DNA 动了" |
| Internet slang → formal | "蚌埠住了" → "忍俊不禁" |
| Boss-speak → real meaning | "公司很看好你" → "要给你加活了" (More work incoming) |
| HR speak → plain truth | "我们会考虑的" → "没戏了" (It's a no) |
| Client requirements → dev translation | "简单改一下" → "推翻重做" (Start over) |
| Partner's words → real meaning | "你自己看着办吧" → "你最好按我想的办" (Do what I want) |
| Classical Chinese → internet style | "不以物喜不以己悲" → "佛系，纯纯的" |
| Internet → classical Chinese | "破防了" → "心城既溃，泪如泉涌" |
| Academic → plain language | "呈现显著正相关" → "越多越好" (The more the better) |
| Plain → academic | "吃得多胖得快" → "热量摄入与体重增长之间存在显著正相关" |

#### C. Platform Voices

Not just word-swapping — the entire expression logic and vibe changes.

Same content "今天去了一家好吃的店" (Went to a great restaurant today) across platforms:

| Platform | Voice |
|----------|-------|
| **小红书 (Xiaohongshu)** | "姐妹们！！！这家店绝了🔥 我直接封为年度TOP1‼️ 不允许还有人没去过😭 建议先收藏⭐" |
| **朋友圈 (WeChat Moments)** | "周末探店，味道不错👍" |
| **B站 (Bilibili)** | "up主今天去了家店，进去直接DNA动了，这个味道，是家的感觉（不是" |
| **知乎 (Zhihu)** | "作为一个在餐饮行业从业十年的人，我认为这家店最大的亮点在于其食材供应链的稳定性……" |
| **X/Twitter** | "Found a gem. Best meal this month. 📍[location]" |
| **LinkedIn** | "Inspired by an incredible dining experience today. Excellence is in the details. #leadership #growth" |
| **豆瓣 (Douban)** | "一家不算太知名的小店，装修普通，但食物有种让人安静下来的力量。适合一个人去。" |

#### D. Custom Style Profile Card

For precise control without a specific person:

| Dimension | Spectrum |
|-----------|----------|
| Directness | Blunt ↔ Diplomatic |
| Temperature | Warm ↔ Cool |
| Sentence rhythm | Short/punchy ↔ Long/flowing |
| Emotion | Expressive ↔ Understated |
| Register | Internet/casual ↔ Formal/professional |
| Humor | Playful ↔ Serious |
| Explanation style | Judging (verdict) ↔ Explaining (reasoning) |

---

### Mode 5: AI-to-AI Translation

Every AI has its own "accent." This mode converts content between different AI expression styles.

#### A. AI Personality Profiles

**GPT style:**
- Opener: "Great question!" / "Certainly!" / "Absolutely!"
- Structure: Markdown headers + numbered lists, comprehensive, overview → detail → examples → summary
- Vocabulary: "It's worth noting", "However", "comprehensive", "fascinating"
- Closer: "Feel free to ask if you have any more questions!"
- Vibe: endlessly enthusiastic, never refuses, sunny colleague

**Claude style:**
- Opener: straight to the point, or "I think..."
- Structure: natural paragraphs, lists only when helpful, "That said..." transitions
- Vocabulary: "I'd say", "That said", acknowledges uncertainty openly
- Closer: natural ending, sometimes a reflective question, no forced cheerfulness
- Vibe: thoughtful, measured, occasionally philosophical

**Gemini style:**
- Opener: concise, sometimes conclusion-first
- Structure: tight bullet points, high information density
- Vocabulary: data-driven, technical, Google ecosystem references
- Closer: brief, occasional follow-up suggestion
- Vibe: professional, neutral, slightly corporate

**DeepSeek style:**
- Opener: direct answer, no small talk, might start with code
- Structure: code blocks + brief explanations, conclusion-first
- Vocabulary: precise, minimal hedging, technical
- Closer: conclusion stated, done
- Vibe: engineer energy, pragmatic, efficiency-first

**Kimi style:**
- Opener: "你好呀~" (Hey there~), casual, sometimes emoji
- Structure: short paragraphs, conversational, parenthetical asides
- Vocabulary: Chinese internet-native, "其实" (actually), "说白了" (to put it simply)
- Closer: "希望对你有帮助哦~" (Hope this helps~)
- Vibe: friendly, energetic, like a helpful young internet friend

**文心一言 (Wenxin) style:**
- Opener: "关于您的问题" (Regarding your question), formal
- Structure: encyclopedic expansion — definition → background → categories → applications → summary
- Vocabulary: formal written Chinese, "具有重要意义" (of great significance), safe/neutral
- Closer: summary + "如需进一步了解，请继续提问"
- Vibe: composed, authoritative, official, avoids controversy

#### B. Comparative Example

**Question:** "为什么有些人喜欢深夜吃泡面？" (Why do some people like eating instant noodles late at night?)

**GPT:** "Great question! Late-night instant noodle eating is actually a fascinating phenomenon that touches on psychology, biology, and culture. Here are the key reasons: 1. **Biological factors**... 2. **Psychological comfort**... 3. **Cultural significance**... It's worth noting that while this habit brings comfort, nutritionists recommend... Feel free to ask if you'd like to explore any of these further! 😊"

**Claude:** "I think the core reason probably isn't hunger — it's that the solitude of late night makes you want a small ritual. Ramen fits perfectly: simple to prepare but ritualistic (tear the packet, pour hot water, wait three minutes), strong enough flavor for instant gratification, and it carries a feeling of 'secret indulgence' — nobody posts late-night instant noodles saying 'healthy eating check-in.' Of course, sometimes people are just hungry."

**DeepSeek:** "Two reasons: 1. Lower metabolic rate at night makes high-carb foods more satisfying; 2. Instant noodles have the lowest preparation cost (3 min), matching the low-energy behavioral preference of late-night states. Essentially the optimal cost-benefit comfort food."

**Kimi:** "哈哈这个问题我太有发言权了~ 🍜 说白了就是深夜那个氛围到了嘛！一个人、安静、有点空虚，这时候来碗热腾腾的泡面，简直是灵魂被治愈的感觉（虽然第二天可能会肿"

**文心:** "深夜食用方便面的现象涉及多个层面的因素。从生理角度看，人体在夜间的血糖水平下降... 从心理角度看... 从文化角度看... 如需进一步了解方便面的营养成分或健康饮食建议，请继续提问。"

#### C. Conversion Directions

- **Human → AI (Intent Compiler)**: convert vague human requests into structured, unambiguous prompts
- **AI → Human (De-robotify)**: strip AI filler ("Certainly!", "Great question!"), flatten over-structured lists, inject natural rhythm
- **AI → AI**: convert between specific AI styles (e.g. "rewrite this GPT answer in Claude style")

---

## Default Output Structure

For any translation request, output layers as needed. **For simple requests, just give the most useful result.**

```
1. VERIFIED TRANSLATION — most faithful version (always present for cross-language)
2. NATURAL VERSION — how a native speaker would say it
3. TERMINOLOGY NOTES — only when domain terms or ambiguities are involved
4. SUBTEXT NOTES — only when hidden meaning exists
5. STYLE VERSION — when a specific persona/platform/tone is requested
```

---

## Input Parameters

Inferred from context. User can also specify explicitly.

**Required (inferred):** source text, source language, target language

**Optional — Reliability:**
- `search`: web search for verification (default: auto)
- `terminology_strict`: lock term consistency (default: true for technical)

**Optional — Style:**
- `persona`: person or AI name (e.g. "鲁迅", "Musk", "GPT")
- `scene`: transfer direction (e.g. "formal→slang", "boss→truth")
- `platform`: platform voice (e.g. "Xiaohongshu", "Zhihu", "B站")
- `target_tone`: tone (e.g. professional, internet-native, sharp)
- `audience`: audience (e.g. friend, boss, client)
- `versions`: number of style variants (default: 1)

---

## Critical Boundaries

### Fact vs. inference separation
Determinate content is presented as translation. Indeterminate content (subtext, tone guesses) is labeled as interpretation.

### Style never overrides meaning
If style transfer would distort meaning, keep meaning intact and flag the conflict.

### Person styles are expression distillation, not identity impersonation
Learn the public figure's **expression style** (sentence patterns, rhythm, vocabulary preferences), not pretend to be that person.

---

## Examples

### Example 1: Subtext Decode

**User:** "我领导说'这个需求很简单，你看着做就行'，这句话什么意思？"

**Output:**
- **Literal:** This requirement is simple, just handle it as you see fit.
- **Real meaning** (common interpretation): The requirement may not be simple; "看着做" = he doesn't want to think through the details, and if it goes wrong you take the blame. Or he may genuinely think it's simple but has no time to explain.
- **How to tell:** If he usually gives you detailed specs but skipped this time → not simple. If he's busy with something else → might genuinely be simple.
- **Safe play:** Draft a quick scope doc and confirm with him before building.

### Example 2: Persona Voice Transfer

**User:** "用鲁迅的口吻说：今天的会开得太长了"

**Output:**
"我参加过许多会议，长的，短的，有意义的，无意义的。然而今天这场，它漫长到让人忘记了时间的存在——不是因为充实，而是因为空洞。发言者们轮流说着正确的废话，听众们轮流看着手机。这大概就是现代职场最寻常的景象：一群人坐在一起，认真地浪费彼此的生命。"

### Example 3: Same-Language Scene Transfer

**User:** "把'这个季度业绩没达标'翻译成不同风格"

**Output:**
- **Boss sugar-coating:** "虽然这个季度有些挑战，但我们已经看到了很多积极的信号，下个季度一定会更好。"
- **Internet slang:** "寄了。季度 KPI 全线崩盘，纯纯的坐牢。"
- **HR sit-down:** "我们注意到你这个季度的表现和预期有一些差距，想和你聊聊接下来的规划。"
- **Friend chat:** "兄弟，这个月白干了。"
- **Academic:** "本季度绩效指标呈现显著负向偏离，反映出目标设定与执行能力之间存在系统性差异。"

### Example 4: AI Style Transfer

**User:** "用 Kimi 风格回答：什么是量子计算？"

**Output:**
"你好呀~ 量子计算简单来说就是用量子力学的黑魔法来算东西🧙 普通电脑用0和1，量子电脑的qubit可以同时是0又是1（薛定谔的猫！）。听起来很酷对吧？不过现在还挺早期的啦，别被营销号忽悠了哈~"

---
> Source: [minruixu/translator.skill](https://github.com/minruixu/translator.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
