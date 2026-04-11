## lemini

> **Project:** Personalized Learning Hub (Laravel + Filament + Gemini)

# GEMINI.md — AI Integration Guide
**Project:** Personalized Learning Hub (Laravel + Filament + Gemini)  
**Purpose:** This document specifies how we use Gemini to power adaptive learning features shown in the wireframe (Student + Instructor flows).
It defines API contracts, prompt templates, guardrails, and Laravel integration patterns so engineers can implement consistently.

---

## Framework
- Laravel
- Filament
- Vue + inertia
- use google-gemini-php/laravel for integration

## General Instructions
- Don't implement without confirmation
- Ensure use best practices
- Implement the feature incrementally, focus on the functionality

## 1) What Gemini does in our app
| Wireframe Area | Capability | Gemini Role | Notes |
|---|---|---|---|
| Student → Lesson Player | Just‑in‑time hints & formative feedback | Generate rubric‑aware hints and encouragement | Human‑in‑the‑loop for summatives |
| Student → Today | Next best activity | Explain **why** a recommendation is made | Backed by rule/IRT; Gemini writes the rationale |
| Student → Spaced Review | Review cards text | Summarize/simplify key concepts for flash prompts | Keep ≤ 30 words each |
| Student → Accessibility | UDL transforms | Simplify/translate lesson text | Preserve technical terms; add analogies |
| Instructor → Item Bank | Item variants/rationales | Draft 3 MCQs (or SAQ) tied to objective & Bloom level | Include keys + concise rationales |
| Instructor → Feedback Queue | Draft feedback | Produce short, actionable comments linked to rubric | Teacher approves/edits before release |
| Instructor → Analytics | Theming open‑text | Optional: cluster survey comments | Keep under 5 themes with quotes |

---

## 2) Environment & config
Set the following in `.env` and config files:
```
GEMINI_API_KEY=xxxxx
GEMINI_MODEL=text‑embedding‑004  # for embeddings (optional)
GEMINI_MODEL_TEXT=gemini‑1.5‑pro  # for generation
GEMINI_TIMEOUT=12                 # seconds
GEMINI_MAX_OUTPUT_TOKENS=512
GEMINI_SAFETY_CATEGORY_BLOCK=harassment,hate,sexual,medical,legal  # adjust
```

Laravel service provider (pseudo):
```php
// app/Providers/GeminiServiceProvider.php
public function register() {
    $this->app->singleton(\App\Services\GeminiClient::class, function () {
        return new \App\Services\GeminiClient(
            apiKey: config('services.gemini.key'),
            model: config('services.gemini.text_model', 'gemini-1.5-pro'),
            timeout: (int) env('GEMINI_TIMEOUT', 12),
        );
    });
}
```

`config/services.php`:
```php
'gemini' => [
  'key' => env('GEMINI_API_KEY'),
  'text_model' => env('GEMINI_MODEL_TEXT', 'gemini-1.5-pro'),
  'embedding_model' => env('GEMINI_MODEL', 'text-embedding-004'),
],
```

---

## 3) API surface (internal)
All AI routes are **internal** and queued. Use controllers to enqueue jobs and return 202 + polling token.

### 3.1 Feedback (rubric-aware)
- **POST** `/api/ai/feedback`
- **Body** `FeedbackRequest`
```jsonc
{
  "attempt_id": 123,
  "rubric": {
    "criteria": [
      {"id":"C1","name":"Correctness"},
      {"id":"C2","name":"Reasoning clarity"},
      {"id":"C3","name":"Errors/minor"}
    ],
    "levels":["Exemplary","Proficient","Developing","Beginning"]
  },
  "student_answer": "…",
  "max_words": 120
}
```
- **Response (draft)** `FeedbackResponse`
```jsonc
{
  "attempt_id": 123,
  "feedback": {
    "hints": ["…", "…"],
    "encouragement": "…",
    "next_step": "…"
  },
  "tokens": 248,
  "model": "gemini-1.5-pro"
}
```

### 3.2 Item generator
- **POST** `/api/ai/item-variants`
```jsonc
{
  "objective_code": "ALG.2",
  "bloom": "Apply",
  "type": "MCQ",
  "difficulty": "medium",
  "count": 3,
  "constraints": {"mcq_options": 4, "rationale": true}
}
```
- **Response**
```jsonc
{
  "variants": [
    {
      "stem": "…",
      "options": ["A","B","C","D"],
      "answer": "B",
      "rationale": "…",
      "meta": {"objective":"ALG.2","bloom":"Apply","difficulty":"medium"}
    }
  ]
}
```

### 3.3 UDL Simplify/Translate
- **POST** `/api/ai/simplify`
```jsonc
{
  "text": "Original lesson body…",
  "reading_level": "A2",
  "max_words": 160,
  "include_analogies": 2,
  "preserve_terms": ["factor","quadratic"]
}
```
- **Response**
```jsonc
{"simplified": "…", "notes":"2 analogies added"}
```

### 3.4 Recommendation rationale
- **POST** `/api/ai/recommendation-note`
```jsonc
{
  "learner": {"id": 9, "first_name": "Aftika"},
  "context": {
    "weak_objective": "ALG.2",
    "recent_attempts": [{"score":0.4},{"score":0.55}],
    "time_on_task":"high"
  },
  "target_activity": {"type":"item","id": 556, "eta_min": 5}
}
```
- **Response**
```jsonc
{"note":"Because ALG.2 is still forming and your last two tries were tough, let’s do a quick 5‑min scaffold on factoring by grouping."}
```

---

## 4) Prompt libraries (templates)
We use a **system + developer + user** template. Keep outputs **JSON‑first** when appropriate.

### 4.1 FEEDBACK_PROMPT
**System**: “You are a concise, supportive formative feedback generator for a learning platform. Follow rubric criteria and keep within word limits.”  
**Developer**:  
- “Return JSON: {hints: string[2], encouragement: string, next_step: string}. No prose outside JSON.”  
- “Hints must reference rubric criteria by name. Be specific and actionable.”  
- “Use encouraging, non‑judgmental tone.”  
**User (variables)**:
```
Rubric:
{{rubric_json}}

Student answer:
{{student_answer}}

Constraints: max_words={{max_words}}, require_2_hints=true
```

### 4.2 ITEM_VARIANT_PROMPT
**System**: “You draft assessment items aligned to stated objective and Bloom level.”  
**Developer**:  
- “Output JSON: {variants:[{stem, options, answer, rationale, meta:{objective,bloom,difficulty}}]}.”  
- “Options must be plausible distractors; rationale concise.”  
**User**:
```
Objective: {{objective_code}}
Bloom: {{bloom}}
Type: {{type}}  (MCQ|SAQ)
Difficulty: {{difficulty}}
Count: {{count}}
Constraints: {{constraints_json}}
```

### 4.3 UDL_SIMPLIFY_PROMPT
**System**: “You transform text for accessibility while preserving key terms.”  
**Developer**: “Reply with JSON: {simplified, notes}.”  
**User**:
```
Reading level: {{reading_level}} (e.g., A2)
Max words: {{max_words}}
Preserve terms: {{preserve_terms}}
Add analogies: {{include_analogies}}

Text:
{{text}}
```

### 4.4 RECOMMENDATION_NOTE_PROMPT
**System**: “You explain recommendations to learners in one sentence.”  
**Developer**: “Output JSON: {note} (≤ 30 words). Friendly tone.”  
**User**:
```
Learner: {{first_name}}
Target activity: {{activity_desc}} (e.g., 5‑min scaffold on factoring by grouping)
Signals: {{signals_json}} (weak objective, attempts, time-on-task)
```

### 4.5 THEMING_PROMPT (optional for surveys)
Return up to 5 themes with short quotes.
```text
Output JSON: {themes:[{label, count, quotes:string[] (≤2)}]}
Text:
{{multiline_feedback}}
```

---

## 5) Laravel integration (code skeletons)

### 5.1 Client
```php
// app/Services/GeminiClient.php
namespace App\Services;

use Illuminate\Support\Facades\Http;

class GeminiClient {
  public function __construct(
    private string $apiKey,
    private string $model,
    private int $timeout = 12,
  ) {}

  public function generate(array $messages, array $opts = []): array {
    $payload = [
      'model' => $opts['model'] ?? $this->model,
      'temperature' => $opts['temperature'] ?? 0.2,
      'max_output_tokens' => (int) env('GEMINI_MAX_OUTPUT_TOKENS', 512),
      'messages' => $messages,
      'safety_settings' => $opts['safety'] ?? [],
    ];
    $res = Http::timeout($this->timeout)
      ->withToken($this->apiKey)
      ->post('https://generativelanguage.googleapis.com/v1beta/models:generateContent', $payload);
    $res->throw();
    return $res->json();
  }
}
```

### 5.2 Job: Feedback
```php
// app/Jobs/GeminiFeedbackJob.php
public function handle(GeminiClient $ai) {
  $attempt = Attempt::with('item.rubric')->findOrFail($this->attemptId);
  $rubricJson = $attempt->item->rubric->toJson();
  $messages = [
    ['role'=>'system','content'=>'You are a concise, supportive formative feedback generator...'],
    ['role'=>'user','content'=> view('ai.feedback_prompt', [
        'rubric_json'=>$rubricJson,
        'student_answer'=>$attempt->response,
        'max_words'=>120
    ])->render()],
  ];
  $raw = $ai->generate($messages);
  $json = json_decode($raw['candidates'][0]['content']['parts'][0]['text'] ?? '{}', true);
  Feedback::updateOrCreate(
    ['attempt_id'=>$attempt->id],
    ['ai_text'=>$json, 'status'=>'draft']
  );
}
```

### 5.3 Controller (enqueue + token)
```php
// app/Http/Controllers/AiFeedbackController.php
public function store(FeedbackRequest $r) {
  $attempt = Attempt::findOrFail($r->attempt_id);
  dispatch(new GeminiFeedbackJob($attempt->id));
  return response()->json(['status'=>'queued','attempt_id'=>$attempt->id], 202);
}
```

### 5.4 Filament: Feedback Queue action
```php
// In table actions: Approve -> copies ai_text -> human_revision -> status=published
TableAction::make('approve')
  ->action(fn(Feedback $f) => $f->update(['status'=>'published','released_at'=>now()]));
```

---

## 6) Contracts & validation
Use Laravel Form Requests to validate payloads. Reject overlong `text` (> 6k chars) and non‑ASCII if unsupported. Enforce JSON outputs using robust parsing; if parsing fails, retry with a “JSON‑only” repair prompt.

---

## 7) Safety, privacy, and guardrails
- **Human‑in‑the‑loop** for summative grading. Do not auto‑publish grades.
- **Content filters:** set safety categories to block harmful content; fall back with generic messages.
- **Data minimization:** send only fields needed for the task; never send PII beyond first name for notes.
- **Audit logs:** store `prompt_hash`, `response_hash`, model, tokens, duration, status.
- **Explainability:** include “why this?” note with recommendations.
- **Boundaries:** AI coaching is not clinical/medical/legal advice; show disclaimers where relevant.

---

## 8) Latency, queues, and retries
- Queue all generation with Redis + Horizon. UI shows “pending” banners.
- Timeouts: 12s per call; 2 retries with exponential backoff (200ms, 1s).
- Circuit breaker: if 5 consecutive failures, disable AI actions for 10 minutes and notify admin.

---

## 9) Evaluation (quality checks)
- **Feedback quality rubric:** specificity, actionability, tone (1–5). Target ≥ 4.0.
- **Item validity:** distractor plausibility, key uniqueness, objective alignment.
- **A/B test:** Gemini hints vs. no hints → compare mastery delta per objective.
- Log human edits to AI drafts to quantify usefulness.

---

## 10) Failure modes & fallbacks
- **JSON parse error:** Re‑prompt with “Return valid JSON only.”
- **Empty/unsafe output:** return generic template feedback and flag for review.
- **Content too long:** chunk text; summarize first, then run task.
- **Model outage:** switch to cached suggestions or local templates.

---

## 11) Versioning
Prefix requests with `ai_version` in payload; store per‑attempt which prompt version was used. Keep a `/prompts` table with active templates and changelogs.

---

## 12) Example prompts (rendered)
**Feedback** (user content excerpt):
```
Rubric:
{"criteria":[{"id":"C1","name":"Correctness"},{"id":"C2","name":"Reasoning clarity"},{"id":"C3","name":"Errors/minor"}],"levels":["Exemplary","Proficient","Developing","Beginning"]}

Student answer:
"I factored x²+5x+6 as (x+1)(x+5)."

Constraints: max_words=120, require_2_hints=true
```
**Expected JSON**:
```json
{"hints":["Recheck the factor pairs of 6 that sum to 5 (C1).","Show your check by expanding the factors to verify 5x appears (C2)."],"encouragement":"Good start spotting positive factors—your process is on the right track.","next_step":"Try (x+2)(x+3) and expand to confirm."}
```

---

## 13) Data schema references
- `attempts(id,user_id,item_id,response,score,metadata)`
- `feedback(id,attempt_id,ai_text,json,human_revision,status,released_at)`
- `items(id,objective_id,stem,type,options,answer,rationale,meta)`
- `masteries(user_id,objective_id,level,last_seen_at)`
- `recommendations(user_id,payload json,chosen bool)`
- `rubrics(id,name,criteria json,levels json)`

---

## 14) Security
- Store API keys in secrets; rotate quarterly.
- Don’t log raw student answers in external vendors; logs stay internal.
- Pseudonymize learner identity for analytics exports.

---

## 15) Roadmap hooks
- Add embeddings for semantic search in Item Bank (optional).
- Add knowledge tracing (BKT/LSTM) feeding into recommendation; Gemini only writes explanations.
- Add multilingual support: pass `lang` to prompts; store locale per user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dhanifudin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dhanifudin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
