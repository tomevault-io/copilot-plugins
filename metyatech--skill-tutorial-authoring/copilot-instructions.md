## skill-tutorial-authoring

> >-


# Tutorial authoring

Use this skill when writing, revising, or auditing any document
where a reader follows steps to build or achieve something.

## Target learner (expertise reversal boundary)

This skill is optimised for **beginner-to-intermediate**
learners encountering the subject for the first or second time.
Most multimedia learning principles (Signaling, Pre-training,
Personalization, heavy imagery) are strongest in that range and
weaken — or reverse — for experts (Kalyuga's *expertise
reversal effect*). If the artefact is an expert-facing quick
reference, the author MUST scale back Signaling, Concept
density, and hand-holding narrative, and lean on Reference
tables. When in doubt, state the target learner explicitly in
the document's opening.

## Scientific foundations

All authoring rules below derive from the principles in this
table. Principles are stated so their scopes do NOT overlap;
when two seem to conflict, the "Scope & limits" column
resolves the boundary. The agent MUST apply them actively when
writing new tutorials and when reviewing existing ones.

### Underlying load model (Sweller's CLT)

Every principle in the table below is a tactic for managing one
of the three load types in Cognitive Load Theory (Sweller, 1988).
When two principles compete, resolve by asking *which load type
currently dominates*.

| Load type | What it is | Which principles address it |
|---|---|---|
| **Intrinsic** — inherent difficulty of the material | Cannot be reduced, only sequenced | Segmenting, Pre-training, Activation |
| **Extraneous** — effort wasted on poor presentation | MUST be minimised | Coherence, Redundancy, Spatial/Temporal contiguity, Split-attention, Signaling, Modality |
| **Germane** — effort spent on schema construction | SHOULD be fostered | Multimedia, Personalization, Generative activity, Worked example, Feedback |

Expertise reversal (Kalyuga, 2007) predicts that tactics which
reduce extraneous load for novices can *increase* extraneous
load for experts (because redundant signals compete with
established schemas). This is why the skill scopes itself to
beginner-to-intermediate learners.

| Principle (source) | Core insight | Scope & limits | Authoring implication |
|---|---|---|---|
| **マルチメディアの原理** (Mayer, 2009) | 補完的情報を異なる表現（画像とテキスト）に分担すると学習が促進される | 「組み合わせ」は**補完**であって**重複**ではない。同一情報の二重提示は本原理では正当化されない(→ 冗長性原理) | 操作ステップでは、画像が WHERE（位置・順序・選択肢の外観）を、テキストが WHAT（動作の種類・画像に映らない値）を担う |
| **空間的接近の原理** (Mayer, 2009) | 対応する画像とテキストが空間的に近いほど効果的 | 1:1 対応のペアに限定。無関係な画像とテキストを並置する理由にはならない | 1 Action = 1 画像。画像は対応テキストの直前・隣接に配置する |
| **時間的近接の原理** (Mayer, 2009) | 対応する画像とテキスト（または音声）は同時に提示するほど効果的 | **音声または動画など時間軸を持つ媒体にのみ**適用。静的ページでは空間的接近原理で代替 | ナレーション付き動画では、画像切替とナレーションを同期させる |
| **一貫性の原理** (Mayer, 2009) | 教示目的と無関係な文書・画像・音は学習を阻害する | 「無関係」は学習目的から見た判定。面白さや装飾性は保持の根拠にならない | 装飾画像・余談・BGM・装飾的アニメーションは除去 |
| **モダリティの原理** (Mayer, 2009) | 視覚＋聴覚の分担は視覚独占より有効（視覚チャネル過負荷回避） | **音声モダリティを含む媒体（動画・音声教材）でのみ**適用。静的テキスト＋画像の媒体では無関係 | ナレーションと同一文章を画面に出さない |
| **冗長性の原理** (Mayer, 2009) | 意味的に同一の情報を複数フォーマットで重複提示すると学習を阻害する | 適用対象は**意味的に同一**の情報（同じ UI ラベル・同じ値・同じ説明）に限定。補完的情報の併置は該当しない(→ マルチメディア原理) | 画像に映っている UI ラベル・選択肢名・既定値をテキストで再掲しない |
| **セグメンティングの原理** (Mayer, 2009) | 学習者がペースを制御できる単位に分割するほど効果的 | セグメント単位は**1つの意味的に閉じたサブゴール**。単一画面・単一状態内の連続操作は原則 1 セグメント。画面遷移・状態遷移・モード切替が自然な境界 | 画面内の項目数で機械的にセグメントを割らない。画面遷移で区切る |
| **分割注意の原理** (Ayres & Sweller, 2021) | 空間的に離れた複数の情報源を統合する必要があると外在的処理が増大する | スクリーンショットと注釈テキストの**物理的距離**が問題。空間的接近原理と相補関係にあるが、こちらは**離れた情報源の統合コスト**に焦点を当てる | 番号吹き出し付きスクリーンショットと説明テキストを隣接配置する。ページ下部にまとめた「設定一覧表」から遠いスクリーンショットを参照させる構成を避ける |
| **ミニマリズム P1: 行動志向** (van der Meij & Carroll, 1995; Carroll, 1990) | 学習者はすぐ行動しながら学ぶ（doing で学ぶ）。最初のアクションへの到達を最短にする | 適用対象は**まだ不要な情報の除去**。必要情報の補完的分担(→ マルチメディア原理)を削ることは含意しない | 前置きの概念説明を最小化し、最初の Action を早める。Concept は first-use 直前に置く |
| **ミニマリズム P2: タスク領域への定着** (van der Meij & Carroll, 1995) | 教材は学習者の実際の目標とタスクに基づく。機能ベースではなく目標ベースで構成する | 「実タスク」は学習者が**本当に達成したいこと**を指す。ソフトウェアの機能一覧に沿った構成はこの原則に違反する | 最上位 Section の goal は学習者の実タスク上の成果物で記述する。入れ子 Section の goal で「なぜこの小手順を行うのか」を実タスク文脈で説明する |
| **ミニマリズム P3: エラー認識・回復の支援** (van der Meij & Carroll, 1995) | エラーは学習機会であり、予防・検出・診断・回復の全段階を支援する | Recovery コンポーネントは**回復手順のみ**を扱う。予防（操作前の注意喚起）と検出（エラー症状の記述）も別途必要 | Recovery に加えて、失敗しやすい操作の**直前**に予防的注意を置く。Recovery 内では「症状→原因→回復手順」の3段構成で書く |
| **ミニマリズム P4: 柔軟な利用の支援** (van der Meij & Carroll, 1995) | 学習者は文書を最初から順に読まない。拾い読み・飛ばし読み・逆引きを支援する | 本スキルの主対象は**順序付きチュートリアル**であるため、完全な非線形設計は求めない。ただし各 Step は可能な限り自己完結させる | 各 Step 冒頭の goal で「この Step で何ができるようになるか」を宣言し、途中参入者が必要な Step を特定できるようにする。Concept と Reference を折りたたみにして既知の読者がスキップできるようにする |
| **シグナリングの原理** (Mayer, 2009) | 重要箇所を視覚的手がかりで強調すると注意配分が改善し本質処理に集中できる | 合図は**学習目的に沿った要素**にのみ付ける。装飾目的の強調・感情表現の太字は一貫性原理違反 | Section の goal 宣言、画像の番号吹き出し、太字 UI 要素名、①②③ の順序番号で重要箇所を示す |
| **事前トレーニングの原理** (Mayer, 2009) | キー用語の名前と特徴を事前に提示すると主学習時の外在的処理が下がる | 予習は**これから出る概念のみ**に限定。遠い将来に出る概念や全体概論はミニマリズム違反 | Concept は first-use の直前に置き、「名前＋鍵となる特徴」を最小単位で提示する |
| **個人化の原理** (Mayer, 2009) | 会話的・二人称・能動的文体はフォーマル文体より学習効果が高い | 文体の**親しみやすさ**が本質。馴れ馴れしさ・絵文字濫用・感情過剰は一貫性原理違反になり得る | 学習者に直接語りかける二人称・能動形で書く（日本語：「〜しましょう」「確認してください」）。三人称で読者を描写しない（「受講者が〜する」「初学者向け」等を禁止） |
| **生成活動の原理** (Mayer, 2014) | 学習者に要約・予測・説明などの生成活動を求めると学習が深まる | 活動は**学習目的に関連**していること。単なる作業の追加は一貫性原理違反 | Verify で「何が起きるか」を観察判断させる。Checkpoint で behavior を自己確認させる。Exercise を周期的に織り込む |
| **熟達度反転効果** (Kalyuga, 2007) | 初心者に効く合図・概念予習・詳細説明は熟達者には逆効果になる | 本スキルは**初〜中級者向け**に最適化。熟達者向け資料では Signaling・Concept 密度・narrative を縮退させる | 対象者を冒頭で宣言し、原理の適用量を対象者に合わせる |
| **フィードバックの原理** (Shute, 2008) | 学習者が自分の行動の正誤・結果を確認できると schema 構築が促進される | フィードバックは**即時・具体・観察可能**であるべき。汎用的な「成功しました」表示は Coherence に接するため実体のある状態記述にする | Verify は Section の観察可能な結果、Recovery は失敗の原因と回復手順、Checkpoint は最上位 Section 末の自己確認、という三層のフィードバックを必ず配置する |
| **ワークトエグザンプル効果** (Sweller, 1985; Atkinson et al., 2000) | 完全な解法例を示してから自力演習に移す方が、最初から演習するより初心者には効果的 | 熟達が進むと逆転（Expertise Reversal）して演習先行が有効になる。本スキルは初〜中級向けなので**例→演習**の順を優先 | 新しい手順や概念では、まず完成例（画像＋完結した Action 列）を通しで見せてから、変化点を差し替える演習（Exercise）を置く |
| **既有知識の活性化** (Merrill, 2002) | 新しい知識を学ぶ前に、学習者が既に持っている関連知識を呼び起こすと学習が促進される | 事前トレーニング原理（未知の用語を教える）とは異なり、こちらは**既知の概念との接続**を促す。対象が beginner-to-intermediate であっても、隣接領域の経験は存在する | 新しい概念を導入する際に「〜を使ったことがあれば、それと同じ仕組みです」のような既知概念へのアンカーを Concept 内で提供する |

**Citation note**: The Mayer principles above cite the 2nd
edition (Mayer, 2009) which defined 12 principles. The 3rd
edition (Mayer, 2021) expanded to 15, adding Split-attention,
Transient information, and Immersion. This skill incorporates
Split-attention (relevant to static tutorials) and notes
Immersion as out-of-scope. Transient information applies to
video/animation tutorials and is not covered here; consult
Jiang & Sweller (2021) when authoring such content.

### Principles not applicable to static text-and-image tutorials

The following are canonical CTML principles, but they require
audio or speaker presence and therefore are **not applicable**
to static text-and-image tutorials. If the artefact is a
narrated video or an avatar-driven walkthrough, these
principles MUST be applied from the primary sources — this
skill does not cover their application.

| Principle | Applies to | Why out of scope here |
|---|---|---|
| **Voice principle** (Mayer, 2009) | 音声ナレーション媒体 | 人間の声 vs 機械音声の比較。静的ページに音声はない |
| **Image principle** (Mayer, 2009) | 動画教材で話者の画像を画面に出すか | 話者の顔映像の有無は静的ページで判断不能 |
| **Embodiment principle** (Mayer, 2014) | 動画で話者がジェスチャーを伴うか | ジェスチャーは動画特有 |
| **Immersion principle** (Mayer, 2021) | VR / 没入型媒体 | 本スキルは 2D 静的ページに限定 |

Authors writing narrated video or VR content MUST NOT assume
this skill covers these principles; apply them separately from
Mayer & Fiorella (2021), *Cambridge Handbook of Multimedia
Learning* (3rd ed.).

## Information hierarchy

Every tutorial MUST be composed of exactly these layers:

```
Tutorial (page)
 ├── Prerequisites — what the learner needs before starting (optional)
 └── Section (depth 0) — milestone ("when done, you can X"); `goal` required
      ├── goal — 1 future-tense sentence declaring what the learner will achieve
      ├── Concept × N — term/background, always collapsible, before first use
      ├── Reference × N — lookup tables, always collapsible, near relevant sub-section
      ├── Section × N (depth 1+, nested) — a group of actions toward one sub-goal
      │    ├── goal — optional at depth > 0 (use to express sub-goal in task language)
      │    ├── Action × N — the atomic unit (image + instruction + result)
      │    ├── Recovery — error recovery, inline, after the action that can fail
      │    └── Verify — "→ expected result" (1 text line; optional result screenshot)
      ├── Checkpoint — end-of-section checklist (exactly one per top-level Section)
      └── NextSteps — what to do after the tutorial (final top-level Section only, optional)
```

A single `<Section>` component is used recursively — it replaces what earlier
versions of this skill called `<Step>` (top-level milestone) and `<Procedure>`
(sub-goal grouping). Nesting depth is computed at compile time and mapped to
`h2` (depth 0) → `h3` → `h4` → … capped at `h6`.

## Seven information types and display rules

| Type | Content | Display | Placement |
|---|---|---|---|
| **Prerequisites** | Required environment, software versions, prior knowledge, completed prior tutorials | Always visible, bullet list | Page top, before the first top-level Section |
| **Action** | Image + instruction + result | Always visible. Image above text, 1:1 mapping | Inside a Section (nested or top-level) |
| **Verify** | Success confirmation + optional result-state screenshot | Always visible, `→` prefix, 1 text line; optional `img` displayed above the text row (Spatial contiguity) | End of a Section that contains Actions |
| **Concept** | Term definition, background | Collapsible (`<details>`) | Before the Section that first uses the term |
| **Reference** | Key tables, panel lists | Collapsible (`<details>`) | Near the Section that needs it |
| **Recovery** | Error recovery steps | Always visible, short | After the Action that can fail |
| **Next steps** | What to do after completing this tutorial | Always visible, bullet list | End of the final top-level Section or after the last Checkpoint |

## Image / text / video hierarchy

| Subject | Primary | Secondary | Rationale |
|---|---|---|---|
| UI operation (where to click) | Annotated screenshot | Label text only | Unknown UI requires visual anchor |
| Result (what happened) | Text (`→` 1 line in `<Verify>`) | Screenshot via `<Verify img="...">` (optional) | State changes are faster to judge via text; screenshot confirms visual result state when text alone is ambiguous |
| Concept (why) | Text only | — | Abstraction does not benefit from images |
| Multi-step continuous flow | Video/GIF | Text (supplement) | Motion cannot be conveyed in stills |

## Atomic unit: Action

The smallest learning unit is:

```
[Image: WHERE to interact] → [Text: WHAT to do] → [Text: WHAT happens (optional)]
```

Rules:

- The author MUST use one image per action. The author MUST NOT
  batch images. *(mechanised: `tutorial/action-single-image`)*
- The image MUST be placed above or before the text (spatial
  contiguity).
- The author MUST NOT describe in text what the image already
  shows (redundancy elimination).
- The image and the text MUST cover non-overlapping channels:
  the image carries **WHERE** (position, visual anchor, step
  ordering via numbered callouts); the text carries the
  imperative **WHAT** plus any values the image cannot convey.
- The author MUST keep in the text: values the learner types by
  hand (e.g. `UE90min`, `CLEAR!`), user-specific paths the
  screenshot cannot generalise, and UI element names that are
  not labelled in the shot.
- The author MUST remove from the text: positional prefixes when
  the image already shows position *(mechanised:
  `tutorial/action-positional-prefix`)*; label/value pairs
  already paired in the image's numbered callouts; micro-
  interaction details already conveyed by the image's arrows or
  step markers.
- Reducing Action text to a bare verb such as "クリックします"
  MUST NOT be used as a redundancy fix. The imperative WHAT
  plus the values the image cannot convey is the minimum;
  strip only what the image already carries.
- If the action produces a visible result, the author MUST state
  it inline. The author MUST NOT create a separate Verify for
  this — Verify is reserved for Section-level confirmation only.

## Signaling (visual cueing)

学習者の注意を本質的な情報に誘導するために、情報階層を**視覚的な
合図**で表現する。合図は学習目的に沿った要素にのみ付ける。

Signaling surfaces in this skill:

| Surface | Cue | Purpose |
|---|---|---|
| Section heading (depth 0) | `goal` banner (future-declarative, required) | Section の到達点を宣言 |
| Section heading (nested, depth > 0) | `goal` banner (optional) | サブゴールをタスク言語で宣言 |
| Action image | Numbered callout (①②③) + arrow | Position / Sequence を強調 |
| Action text | Bold for unlabelled UI element names, typed values, key gestures | Identity / Typed value の強調 |
| Verify / Recovery | Component framing (→, title) | 状態判定と回復手段の境界を明示 |

Rules:

- 合図は**必ず学習目的に沿う**こと。装飾目的の太字、感情表現の
  強調、文末の飾り記号は Signaling ではなく一貫性原理違反。
- 画像の番号吹き出しとテキストの ① 番号は**併記する**（Sequence
  channel の二重化は Redundancy に該当しない補完関係）。
- 太字の濫用（1 文に 3 箇所以上など）は合図の効力を破壊するため
  禁止。強調は本当に注視すべき要素のみに絞る。

## Personalization (reader-addressing voice)

学習者に**直接語りかける**二人称・能動・会話的な文体で書く。

Rules:

- Use second-person direct address to the reader. Do not
  describe the reader in third person ("受講者が〜する",
  "初学者が〜", "学習者は〜").
- Use active, conversational Japanese: 「〜しましょう」
  「〜してください」「ここで〜を確認します」.
- Do not open a page by describing what the document *is* or
  *who it is for* ("この教材は〜のための資料です" は NG).
  Open with the first learner-facing step or an inviting
  goal statement.
- Personalization is about **friendliness, not familiarity**.
  Emoji spam, 余談, 感情過剰な装飾はむしろ Coherence 原理違反
  になるため避ける。親しみやすさの上限は「先輩が隣で教えて
  くれる」程度が目安。
- Goal strings already use future-declarative form; they also
  implicitly address the reader — do not revert them to
  third-person ("受講者が〜する状態になります" は Goal と
  Personalization の両方に違反する).

## Accessibility (authoring obligations)

画像主体のチュートリアルでは、アクセシビリティは**プラットフォーム
実装の責務**と**オーサリングの責務**の両方にまたがる。以下はオーサ
リング段階で著者が守るべき最低限のルール（WCAG 2.2 Level AA 準拠、
Section 508 E205 の教育・訓練資料要件に基づく）。

Rules:

- Every `<Action>` image MUST have an `alt` prop that describes
  what the image shows in the context of the step. The alt text
  MUST convey the **WHERE** information (which panel, which
  button, which area) so that a screen-reader user can follow
  the procedure without seeing the image. If the image is purely
  decorative (rare in tutorials), use `alt=""`.
- Numbered callouts, arrows, and highlights in images MUST NOT
  rely on colour alone to convey meaning. Pair colour with
  shape (numbered circles, arrows with labels) so that readers
  with colour vision deficiencies can follow the sequence
  (WCAG SC 1.4.1).
- When an image conveys information not present anywhere in the
  surrounding text (e.g. a UI layout, a spatial relationship
  between panels), the author MUST provide a text equivalent
  nearby — either in the Action text, a Concept, or a
  Reference — so that the meaning is recoverable without the
  image.
- Text annotations overlaid on screenshots MUST meet a minimum
  contrast ratio of 3:1 against the background they sit on
  (WCAG SC 1.4.11 for non-text UI components).
- The tutorial's heading hierarchy is produced by `<Section>`
  nesting depth (depth 0 → `h2`, depth 1 → `h3`, … capped at
  `h6`). The `remark-section-headings` plugin injects these
  semantic headings automatically so that screen-reader
  navigation by heading works correctly.
- Interactive examples or embedded widgets (if any) MUST be
  operable by keyboard alone in a logical tab order.

## Generative activity (prediction / retrieval)

学習者が**受動的に読むだけ**にならないよう、生成活動（予測・
説明・自己確認）を組み込む。活動は必ず学習目的に関連させる。

Surfaces:

| Surface | Generative role |
|---|---|
| Verify | 直前の Section の結果を**観察判断**させる（受動受領でなく能動観察） |
| Checkpoint | 最上位 Section 末で behavior を**自己確認**させる retrieval 活動 |
| Exercise (course-docs-platform の `<Exercise>`) | 学習単位の周期的な応用課題 |
| Recovery | 失敗時の**原因推論**を補助する（一行で原因→回復） |

Rules:

- Verify の文面は「**観察可能な状態**」で書く（例: 「キューブが
  消えれば成功」）。内部処理・実行履歴の記述（例: 「Destroy
  Actor が実行されました」）は生成活動を奪う。
- Checkpoint の項目は学習者自身が視覚・操作で確かめられる内容に
  限定。内部状態や jargon は不可。
- Exercise は**学習目的に関連**していること。作業量稼ぎの演習、
  本筋と関係ない応用は一貫性原理違反。
- 予測を促すプロンプト（「実行前に、何が起こるか予想してみて
  ください」）は有効だが、過剰使用は認知負荷を上げる。Step ごとに
  1 回が目安。

### Progressive independence (scaffolding / fading)

ワークトエグザンプル効果と生成活動原理を組み合わせて、チュートリアル
全体を通して**段階的に足場を外す**構成にする（Van de Pol et al.,
2010; backward fading: Renkl et al., 2002）。

静的チュートリアルにおける段階的撤退の実装:

| Phase | Structure | Learner role |
|---|---|---|
| **Phase 1: 完全例** (序盤の Step) | 全 Action に画像＋詳細テキスト。Concept で用語を丁寧に導入。Verify で結果を明示 | 観察・模倣（worked example） |
| **Phase 2: ガイド付き変形** (中盤の Step) | 基本手順は示すが、一部の値・選択肢を「〜に変更してみましょう」で学習者に委ねる | 部分的な意思決定（completion problem） |
| **Phase 3: 独立課題** (終盤の Exercise) | Goal と期待結果のみ提示。手順は示さない | 自力での手順構成（independent practice） |

Rules:

- Tutorial の最初の Step は Phase 1（完全例）で始める。
  いきなり Phase 3 の独立課題を出すのはミニマリズム P1 と
  ワークトエグザンプル効果に反する。
- Phase 間の移行は**同種の手順の繰り返し**で自然に起こる。
  同じ操作パターンが 2 回目に出るときに Phase 2 へ、
  3 回目以降に Phase 3 へ移行するのが目安。
- Phase 2 の Exercise では、変更点（差分）を明示し、学習者が
  ゼロから構成する必要がない形にする。
- 足場の撤退量は対象読者の熟達度に比例させる。beginner 向け
  では Phase 1 を長めに、intermediate 向けでは Phase 2 から
  開始してもよい。

## Writing rules

### Goal text

- The `goal` string is rendered verbatim as a banner directly below
  the section heading; no prefix such as "ゴール:" is added. It MUST
  read as a complete sentence on its own. Bare noun-phrase endings
  such as 「〜した状態」 are forbidden because they render as
  incomplete prose.
- The author MUST write goals in future-declarative form describing
  what the learner will achieve by the time the section is complete:
  - Action completion → 「〜します」（例: 「キューブを 1 つ置きます」）
  - Acquired capability → 「〜できるようになります」（例:
    「キャラクターを操作できるようになります」）
  - Acquired behavior / state → 「〜ようになります」 /
    「〜の状態になります」（例: 「触れたら消えるようになります」）
- The author MUST NOT write goals in past or completed form
  (「〜した」「〜された」「〜した状態」「〜している」「〜できます」) because
  those frame the section as a retrospective of what already happened
  instead of a preview of what the learner is about to build.
  *(mechanised: `tutorial/section-goal-required`, `tutorial/section-goal-tense`)*

### Action text

- The author MUST use the imperative mood: 「〜をクリックします」
  「〜と入力します」.
- The author MUST name a UI element in bold **only when** the
  image does not clearly label it with a visible caption or a
  numbered callout, or cannot disambiguate it from similar
  elements. Repeating an image-labelled element in text violates
  the Redundancy principle; Identity is image-primary when
  labelled in the shot (see the WHERE/WHAT channel rules in
  Atomic unit: Action).
- Panel and location names appear inline on first use **only
  when** the shot does not already make the panel unambiguous.
  A full-screen screenshot that shows the panel in context does
  not require redundant naming in text.
- Regardless of the above, values the learner must type
  (e.g. `UE90min`), user-specific paths, and gestures/motions
  (drag direction, hover vs click) MUST remain in text because
  a still image cannot convey them.

### Concept text

Concept serves the **Pre-training** principle: it teaches the
*name* and the *key features* of a term the learner is about
to encounter, so that main-task cognitive load is reduced.

- A Concept MUST be at most 5 sentences or 1 short table.
- A Concept MUST answer "what is it?" (name + key features)
  and "why does the learner need to know right now?".
- A Concept MUST be placed immediately before the first
  Section that uses the term. Placing Concepts far upfront
  violates Minimalism; omitting them until after first use
  violates Pre-training.
- Concepts for terms that appear much later MUST NOT be written
  now. Pre-training applies to the *next* sub-task, not to the
  entire page.
- If a Concept needs more than 5 sentences, the author MUST split
  it into multiple Concepts and place each before its own
  first-use Section.

### Verify text

- In component-based tutorials the Verify component renders its
  own leading `→`; the author MUST NOT include `→` in the source
  or the rendered output will have a doubled arrow.
  *(mechanised: `tutorial/verify-no-duplicate-arrow`)*
- In plain-Markdown tutorials (no component), the Verify line
  MUST start with `→`.
- A Verify line MUST describe observable state, not internal
  mechanics:
  - ✅ 「キューブが消えれば成功です」
  - ❌ 「Destroy Actor が実行されました」

### Prerequisites text

- Prerequisites MUST appear at the page top, before the first
  top-level Section.
- Each prerequisite MUST be actionable or verifiable: state
  the required software version, completed prior tutorial,
  or assumed knowledge concretely.
  - ✅ 「Unreal Engine 5.4 以上がインストール済みであること」
  - ✅ 「Step 1〜3（前回のチュートリアル）を完了していること」
  - ❌ 「基本的な知識があること」(what knowledge?)
- If no prerequisites exist, omit the section entirely (do not
  write "特になし").

### Recovery text

Recovery serves **ミニマリズム P3** (error recognition and
recovery support). It covers the full error lifecycle:
prevention, detection, and correction.

- A Recovery block MUST be placed **after** the Action that can
  plausibly fail.
- A Recovery block MUST follow the structure: **symptom →
  cause → fix** (in that order). The symptom comes first
  because the learner sees the symptom, not the cause.
  - ✅ 「ブループリントが動かない場合 → コンパイルエラーが
    出ていないか確認してください → ノード名のタイプミスが
    原因です。正しい名前は〜」
  - ❌ 「うまくいかない場合はやり直してください」
- For actions with a high failure probability, the author
  SHOULD place a **preventive note** (1 sentence) immediately
  before the Action, warning about the common mistake. This
  note is distinct from Recovery (which is reactive).
- Recovery MUST NOT be placed at the end of a Section as a
  catch-all. Each Recovery block addresses a specific failure
  point.

### Next steps text

- Next steps MUST appear only at the end of the final top-level
  Section or after the last Checkpoint.
- Each item MUST link to a concrete next action: another
  tutorial, a documentation page, or an exercise.
- The author MUST NOT use vague pointers ("詳しくは公式
  ドキュメントを参照してください" without a link).

### Checkpoint

- A Checkpoint MUST be a bullet list of observable behaviors.
- A Checkpoint MUST NOT include internal state or jargon.
- Exactly one Checkpoint per top-level Section, placed as the
  last element of that Section.
  *(mechanised: `tutorial/checkpoint-placement`)*

## Anti-patterns (do NOT do)

Judgement-based anti-patterns; a tool cannot reliably detect
these, so the author is responsible for catching them.

| Anti-pattern | Violated principle | Fix |
|---|---|---|
| Text restating what image shows | Redundancy | Remove the text or remove the image |
| Button/selection label repeated in bold text while the image already labels it with a numbered callout | Redundancy | Keep text to "① を選びます" etc.; Identity is carried by the image |
| Mechanical splitting of a single-screen unified task into many Actions (one per item in the same dialog) | Segmenting (misapplied) | Keep 1 screen = 1 Action when the sub-goal is unified; split only on screen/state transitions |
| Decorative images, fun sidebars, background music | Coherence | Remove entirely; they impair learning |
| Same content in narration AND on-screen text | Redundancy / Modality | Use narration OR on-screen text, not both |
| `:::note` for concepts | Segmenting | Not collapsible; use Concept component |
| Verify after every action | Segmenting | Verify at Section end only |
| Front-loading reference tables | Minimalism | Use Reference, near first use |
| Term introduced before it's needed | Minimalism | Concept before first-use Section |
| Reducing Action text to a bare "クリックします" to avoid redundancy | Redundancy (over-correction) | Keep the imperative WHAT plus the values the image cannot convey |
| Settings table duplicating the image's numbered callouts row-for-row | Redundancy | Keep in text only the values the image cannot convey (typed input, user-specific paths, dropdown values absent from the shot) |
| Micro-interaction detail ("空白で離す", "カーソルを乗せ") redundantly described when image's arrows already convey it | Redundancy | Remove — but only after confirming the image truly conveys the gesture; motion attributes ("drop in **empty** space", "hover vs click") often need text because a still image cannot encode them |
| Opening a page by describing what the document *is* or *who it is for* ("この教材は〜のための資料です", "受講者が〜する授業") | Personalization | Rewrite in second-person direct address; open with the first learner-facing action or an inviting goal |
| Describing the reader in third person ("学習者は〜", "初学者向け", "受講者が〜") anywhere in the tutorial body | Personalization | Use second-person active voice ("〜しましょう", "確認してください") |
| Front-loading a long concept chapter before the first Action (Pre-training misapplied) | Pre-training × Minimalism | Move each term's Concept to immediately before its first-use Section; keep each Concept to name + key features only |
| Bold/highlight used for emotional emphasis or decoration, not tied to a learning-objective cue | Signaling × Coherence | Reserve bold/highlight for the element the learner must find or type; remove decorative emphasis |
| Multiple bold spans crammed in one sentence | Signaling (dilution) | Bold only the single element that most matters; demote the rest to plain text |
| Verify line that describes internal mechanics instead of observable state ("Destroy Actor が実行されました") | Generative activity | Rewrite as an observable outcome the learner can check ("キューブが消えれば成功") |
| Exercises tacked on for practice volume rather than learning objective | Coherence / Generative activity (misapplied) | Tie every Exercise to the Step's stated goal; drop unrelated drills |
| Applying beginner-weight Signaling/Concept density to an expert-facing reference | Expertise reversal | Scale back: use compact Reference tables, drop hand-holding narrative |
| Using `<Action img>` to show a result-state screenshot while the text contains result-check language ("〜になれば成功", "〜ていることを確認") | Feedback (Verify workaround) | Replace with `<Verify img="...">` — the image carries the observable result state, the text carries the 1-line confirmation *(mechanised: `tutorial/verify-visual-workaround-as-action`)* |
| Organising tutorial sections by software feature/menu rather than by learner's task goal | ミニマリズム P2 (task anchoring) | Reorganise by what the learner wants to achieve, not by where the feature lives in the UI |
| No Recovery block after an action that commonly fails | ミニマリズム P3 (error support) | Add Recovery with symptom → cause → fix structure |
| Recovery that says "やり直してください" without diagnosing the cause | ミニマリズム P3 (error support) | Rewrite with concrete symptom, cause, and fix |
| All Steps require reading every prior Step to make sense; no standalone entry point | ミニマリズム P4 (flexible use) | Make each Step's goal self-explanatory; use collapsible Concepts/References so known readers can skip |
| Tutorial starts without stating required environment, software version, or prior knowledge | Prerequisites (ISO 26514) | Add a Prerequisites section at the page top listing concrete, verifiable requirements |
| Images with no `alt` text, or `alt` text that says "screenshot" / "image" | Accessibility (WCAG SC 1.1.1) | Write `alt` that describes WHERE information: which panel, button, or area is shown |
| Numbered callouts or highlights that use colour alone (no shape or label) to convey sequence | Accessibility (WCAG SC 1.4.1) | Pair colour with numbered circles, arrows with text labels, or other shape cues |
| Jumping straight to independent exercises without first showing a complete worked example | Scaffolding / Worked example | Start with Phase 1 (full example), then Phase 2 (guided variation), then Phase 3 (independent) |
| Introducing a new concept without connecting it to anything the learner already knows | Activation (Merrill) | Add an analogy or reference to a familiar concept in the Concept block |
| Screenshot + explanation table placed far apart, requiring the reader to scroll between them | Split-attention | Place the explanation immediately adjacent to (or overlaid on) the screenshot |

## Mechanised checks (enforced at MDX build/dev time)

The following conventions are enforced by the
`remarkTutorialLint` plugin in
`@metyatech/course-docs-platform`. Violations surface in
`npm run dev` and `npm run build` output; author reliance on
memory is not required.

### Severity policy (evidence-tiered)

Severity is tied to how strongly the rule is anchored in the
underlying research, so the tool does not over-reach.

| Severity | Semantics | Build effect |
|---|---|---|
| **error** | Structural break that makes the MDX incoherent or loses required authoring metadata | Fails the MDX compile |
| **warn**  | Principle violation with solid empirical support, or a render/technical bug | Emitted via `console.warn` + `file.message()`. Fails under `TUTORIAL_LINT_STRICT=1` |
| **note**  | Advisory derived from a principle whose specific numeric threshold or lexical pattern is a professional guess rather than a direct research finding | Emitted via `console.info` only. **Never** promoted to an error, even under strict. In collect-all mode, notes appear in the summary but do not by themselves fail the build |

`TUTORIAL_LINT_COLLECT=1` aggregates every finding in a file
into a single failure message, so a PR author can fix all
violations in one pass instead of one at a time. If the
aggregated collection contains only notes, the summary is
printed via `console.info` and the build still passes.

### Rule → severity mapping

| Rule ID | Severity | Intent |
|---|---|---|
| `tutorial/page-authoring-mode-invalid` | **error** | Frontmatter `authoringMode` must be `tutorial` or `non-tutorial` |
| `tutorial/page-mode-tutorial-requires-section` | **error** | A page declared `authoringMode: tutorial` must contain at least one `<Section>` |
| `tutorial/page-mode-non-tutorial-has-section` | **error** | A page without `authoringMode: tutorial` must not use `<Section>` |
| `tutorial/section-goal-required` | **error** | Every `<Section>` declares its `goal` |
| `tutorial/action-single-image`   | **error** | One image per Action |
| `tutorial/checkpoint-placement`  | warn  | Exactly one `<Checkpoint>` per top-level Section, placed last |
| `tutorial/section-no-hrule`      | warn  | No `---` inside a Section |
| `tutorial/verify-no-duplicate-arrow` | warn  | `<Verify>` body starts with `→` — component already renders it |
| `tutorial/action-positional-prefix`  | warn  | `img`-bearing `<Action>` body starts with a positional prefix |
| `tutorial/section-lacks-feedback`    | warn  | A Section with Actions has no `<Verify>` / `<Recovery>` / `<Checkpoint>` |
| `tutorial/section-goal-tense`        | *note* | Goal endings matching heuristic future-declarative patterns |
| `tutorial/reference-image-only`      | *note* | `<Reference>` whose only content is an image |
| `tutorial/action-bold-overuse`       | *note* | 6+ bold spans in one Action (dilution threshold is advisory) |
| `tutorial/third-person-reader`       | *note* | Third-person reader references match a heuristic pattern list |
| `tutorial/page-opens-with-doc-description` | *note* | First paragraph starts with "この教材は〜" etc. (opener-only scope is heuristic) |
| `tutorial/verify-internal-mechanics` | *note* | Verify text matches the engine-state pattern list |
| `tutorial/concept-length`            | *note* | Concept body exceeds 10 sentences or 1 table (numeric threshold is advisory) |
| `tutorial/concept-placement`        | *note* | Concept has no following Action / Section / Exercise |
| `tutorial/decorative-emoji`         | *note* | Non-allowlisted emoji outside signalling surfaces (allowlist is a cultural convention) |
| `tutorial/verify-visual-workaround-as-action` | *note* | `<Action img>` whose text matches result-check patterns ("〜になれば成功", "〜ていることを確認" etc.) — likely a Verify disguised as an Action |
| `tutorial/prerequisites-placement` | warn | `<Prerequisites>` appears after the first `<Section>` |
| `tutorial/nextsteps-placement` | *note* | `<NextSteps>` appears before the last `<Section>` (advisory) |

The *note* tier exists because these rules are correct in
principle but their specific numeric boundary or lexical
trigger has no direct empirical backing — they are the
authoring equivalent of professional code review hints, not
hard gates.

## Page authoring mode (frontmatter)

Tutorial pages MUST declare their authoring mode in
frontmatter so the lint plugin can apply tutorial-specific
rules only to tutorial pages:

```yaml
---
title: ページタイトル
authoringMode: 'tutorial'
---
```

Mode values:

| Value | Meaning | Effect |
|---|---|---|
| `tutorial` | This page is a tutorial authored per this skill | Page MUST contain at least one `<Section>`; all tutorial lint rules apply |
| `non-tutorial` | Reference, overview, or narrative page | Page MUST NOT use `<Section>`; tutorial lint rules are skipped |
| *(omitted)* | Defaults to `non-tutorial` | Same as explicit `non-tutorial` |

Mixing modes on one page is not supported. A page that mixes
tutorial structure with free-form narrative MUST either be
split, or be authored as `non-tutorial` without `<Section>`.

## Component system (course-docs-platform)

When writing for `@metyatech/course-docs-platform`-based sites,
the author MUST use the provided MDX components. They are
globally available (no import needed):

```mdx
---
title: コリジョンを設定する
authoringMode: 'tutorial'
---

<Section title="Step N：コリジョンを設定する" goal="アイテムに触れると消えてスコアが増えるようになります">

  <Concept title="コリジョンとは">
    当たり判定のこと。ブロック＝壁、オーバーラップ＝すり抜け＋検知。
  </Concept>

  <Section title="N-1. 触れたことを検知できるようにする" goal="アイテムをすり抜けて通れるようになります">
    <Action img="./img/overlap-events-on.png">
      **Generate Overlap Events** をオンにする
    </Action>
    <Action img="./img/overlap-all-dynamic.png">
      **コリジョンプリセット**を **OverlapAllDynamic** に変更する
    </Action>
    <Verify>アイテムをすり抜けて通れる</Verify>
  </Section>

  <Checkpoint>
    - アイテムに触れるとスコアが増える
    - アイテムに触れると消える
  </Checkpoint>

</Section>
```

A single `<Section>` is used recursively at both the milestone
level (depth 0) and the sub-goal grouping level (depth > 0).
Nesting depth maps to heading level: `h2` → `h3` → … capped at
`h6`.

### Components

| Component | Props | Purpose |
|---|---|---|
| `<Prerequisites>` | children | Page-level requirements; placed at page top before the first Section |
| `<Section>` | `title: string` (required), `goal?: string` (required at depth 0, optional deeper) | Recursive milestone / sub-goal container; replaces the legacy `<Step>` and `<Procedure>` |
| `<Action>` | `img?: string`, `alt?: string`, children | Atomic operation with optional screenshot |
| `<Verify>` | `img?: string`, `alt?: string`, children | Section-level success confirmation; `img` renders a result-state screenshot above the text row |
| `<Concept>` | `title: string`, children | Collapsible background/term; supports expertise-scaled default-collapse (see `?level=` query) |
| `<Reference>` | `title: string`, children | Collapsible lookup table |
| `<Recovery>` | `title: string`, children | Inline error-recovery block attached to the preceding Action |
| `<Checkpoint>` | children | End-of-Section checklist; exactly one per top-level Section |
| `<NextSteps>` | children | End-of-tutorial next actions with links; placed at the end of the final top-level Section |

## Non-component tutorials

When components are not available (plain Markdown, Docusaurus,
etc.), the author MUST apply the same information hierarchy
using native syntax:

| Component equivalent | Plain Markdown |
|---|---|
| Prerequisites | `## 前提条件` + bullet list at page top |
| `<Section>` (depth 0) | `## タイトル` + first line = goal sentence |
| `<Section>` (nested, depth 1+) | `### タイトル` / `#### タイトル` (heading level = Markdown depth + 2, capped at `######`) |
| `<Concept>` | `<details><summary>💡 Title</summary>...</details>` |
| `<Reference>` | `<details><summary>📖 Title</summary>...</details>` |
| `<Action>` | `![alt](img)` on its own line, then numbered list item |
| `<Verify>` | `**→ expected result**` |
| `<Recovery>` | `:::caution[〜のとき]` or equivalent admonition right after the Action |
| `<Checkpoint>` | `:::tip[確認ポイント]` or equivalent admonition |
| Next steps | `## 次のステップ` + bullet list with links, after the last Checkpoint |

The hierarchy and writing rules MUST remain identical regardless
of tooling.

## Limits of principled authoring

This skill encodes the best available tactics, but it does not
guarantee pedagogically correct output. Authors and reviewers
MUST treat the following as known limitations.

### Semantic checks that tooling cannot enforce

The `remarkTutorialLint` plugin catches structural violations
only. The following judgements are **author-only**; treat them
as explicit review gates, not as automated safety nets.

| Judgement | Why machine-unreachable |
|---|---|
| "Is this bold/highlight serving a learning objective, or decorative?" (Signaling × Coherence) | Requires knowing what is objective-relevant at this step |
| "Is this image decorative or essential?" (Coherence) | Same |
| "Is this prose second-person direct address, or third-person description?" (Personalization) | Japanese grammar allows zero-subject sentences; pattern matching over-triggers |
| "Does this Exercise serve the stated Step goal?" (Generative activity × Coherence) | Requires semantic alignment with the Step's goal string |
| "Is this Concept's 'why does the learner need to know now?' actually satisfied?" (Pre-training) | Intent-level check |
| "Is Signaling density appropriate for the target learner?" (Expertise reversal) | Requires modelling the reader's prior knowledge |
| "Does this `alt` text convey WHERE information adequately for a screen-reader user?" (Accessibility) | Requires understanding what the image contributes to the step |
| "Does this Recovery follow symptom → cause → fix, and is the symptom what the learner actually sees?" (ミニマリズム P3) | Requires knowing the real failure mode, not just the technical cause |
| "Is this Exercise at the right scaffolding phase (1/2/3) for this point in the tutorial?" (Scaffolding) | Requires tracking cumulative learner exposure to the pattern |
| "Does this Concept's analogy accurately bridge to prior knowledge the target learner has?" (Activation) | Requires modelling the reader's adjacent-domain experience |

### Generalisation limits of the underlying research

- Mayer's effect sizes were measured primarily on **educational
  videos and science lessons** with short durations. Applying
  them to long-form software tutorials requires extrapolation;
  the *direction* of each effect is robust, but magnitude is not
  guaranteed.
- Most studies use novice learners in controlled settings.
  Real-world learners mix expertise levels, read non-linearly,
  and bring prior frustration. The skill's rules are the
  *baseline*, not a replacement for observing real learners.
- Language and cultural effects on Personalization / Voice have
  been studied mainly in English. Japanese-specific tactics
  (「〜しましょう」 vs 「〜します」 の丁寧度階調, etc.) are
  transferred by analogy, not by direct evidence.

### When principles appear to conflict

1. Identify which load type (Intrinsic / Extraneous / Germane)
   is currently dominant for the target learner at this step.
2. Apply the principle that directly addresses that load type.
3. If still ambiguous, ask *which tactic would a novice
   benefit more from right now?* — the skill is novice-biased.
4. If the conflict involves an out-of-scope principle (Voice,
   Image, Embodiment, Immersion), treat it as not resolved by
   this skill and consult the primary sources.

### Self-review

Use `REVIEW-CHECKLIST.md` (same repository) as a reviewer-facing
checklist. It mirrors the principle table and is the single
place to audit a draft tutorial against every principle in
sequence.

---
> Source: [metyatech/skill-tutorial-authoring](https://github.com/metyatech/skill-tutorial-authoring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
