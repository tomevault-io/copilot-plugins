## kojev

> A Kotlin Multiplatform client library for **Jev**, TypeSafe AI's System One model.

# kojev

A Kotlin Multiplatform client library for **Jev**, TypeSafe AI's System One model.

For first-time setup and the development phases, see `docs/bootstrap.md`.
The confirmed Jev API specification lives in `docs/api-notes.md` — read it before writing any implementation.

---

## Why this library exists

kojev takes a position on three things, and every design decision is judged against them:

1. **Kotlin Multiplatform.** `jvm`, Android, and the iOS targets, so that domain types and the
   DSL can be shared with app code.
2. **Answers come back as the caller's own types - for Score as well as Choice.** A Choice is the
   caller's enum. A Score is a distribution over the caller's enum rubric
   (`mostLikely: Urgency`, `probabilities: Map<Urgency, Double>`), not a level number plus a legend.
3. **There is exactly one, typed way to read an answer.** No string-id lookup, no default
   thresholds, no helper that rounds a Score's mean into a level. This is not a feature that is
   missing; it follows from hard rules 2 and 3. The judgement calls belong to the caller, and a
   convenience that makes one on the caller's behalf will be used.

Other Kotlin clients exist and are good at what they choose to be: `pambrose/jev4k` is a JVM
library with typed choices and many conveniences (string ids, bands, level helpers);
`ufec/typesafe-sdk-kotlin` is a faithful port of the official JavaScript SDK. kojev is the
multiplatform, deliberately narrow one. **Reject any design decision that erodes one of the three
points above** - an implementation that gives them up has no reason to exist next to those libraries.

Non-goals:
- Wrapping text generation (Jev does not generate)
- Prompt templates or agent frameworks
- A 1:1 port of the official SDK (`ufec/typesafe-sdk-kotlin` already does that)

---

## Hard rules

### 1. Never guess the API

Jev was released on 15 September 2026 and **does not exist in your training data**.
Writing endpoints, field names, or response shapes from memory is forbidden.

`docs/api-notes.md` is the source of truth. If you need something that is not recorded there,
look it up starting from `https://docs.typesafe.ai/llms.txt`, **update api-notes.md first**,
then implement. Mark any implementation whose source you cannot cite with a comment.

### 2. Do not confuse the three primitives

| Type | Question | Returns |
|---|---|---|
| **Choice** | Pick one option (`instructions` + a label→description map) | the label, a probability distribution, confidence |
| **Score** | Rate against a 2–10 level rubric | the score (a `Double`: the probability-weighted mean of the level numbers), the most likely level, a distribution, confidence |
| **Noul** | "Is this statement true?" | **a probability in 0..1. There is no confidence field** |

- **Noul is not a null check.** It returns the probability that a statement holds.
- **A Noul and a two-option Choice are not interchangeable.** Never carry a threshold between them.
- **A Score's answer is not a level.** The mean usually falls between levels and can land on a
  level the model gave zero probability. The library never rounds it into a level for the caller.
- An empty Choice `criteria`, or a Score with fewer than two levels, is a request-time error.

### 3. Do not weaken type safety

- `result[key]` takes a `QuestionKey<T>` and returns the answer typed as `T`
- **Do not provide string-key lookup, not even as a convenience API** — if it exists, it will be used
- A missing, mistyped, or out-of-range value in a response raises an exception.
  It is never silently turned into a default.
- The library does not pick default thresholds. That judgement belongs to the caller.

### 4. API key assumptions

The official SDK blocks direct use from browsers, because the key would be exposed.

- Treat `baseUrl` override as a first-class feature (proxies and gateways)
- **Never hardcode a key in sample code.** Read from the environment in every example.
- The iOS and Android targets exist so that domain types and the DSL can be shared with app code,
  not so that a device can call the API directly. (BYOK apps are the exception — distinguish the two in the README.)

### 5. Tests must pass without a key

Jev is in waitlisted early access. **Do not build something that stalls without a key.**

- Verify request construction and response parsing with Ktor's `MockEngine`, with no network access
- Base mock JSON on the real schema recorded in `docs/api-notes.md`. Never invent it.
- Put live-API tests in a separate source set, run only when `TYPESAFE_API_KEY` is set
  (skipped when absent — not a failure)
- A green `jvmTest` alone is not "tested". Run every target.

---

## Stack

- Kotlin 2.0+ with Coroutines
- Ktor Client (`ktor-client-core`) + `ktor-serialization-kotlinx-json`
- kotlinx.serialization

Targets: `jvm()`, `androidTarget()`, `iosArm64()`, `iosSimulatorArm64()`, `iosX64()`.
Keep platform-specific APIs out of `commonMain` so that `js(IR)` and `wasmJs` can be added later.

**Do not pin a Ktor engine.** Take no dependency on `ktor-client-okhttp` and friends; the caller injects one.

Transport requirements:
- Retry on 408 / 429 / 5xx with exponential backoff and jitter, honouring `Retry-After`. Make it configurable.
- Split errors into subclasses by HTTP status, each carrying the request id
- `JevClient` is thread-safe: create one per application and share it. `close()` releases the HTTP client it created.
- Timeouts apply per attempt; a total budget bounds the whole decision — a wait that would
  exceed it is not taken (the last failure is thrown at once with the server's `retryAfter` on it),
  and the last attempt's timeout is shortened to what remains. Unlike the Python SDK's budget,
  which bounds only the waits (`docs/api-notes.md`)
- Everything a decision throws is a `JevException`: `JevApiException` per status, `JevConnectionException`
  / `JevRequestTimeoutException` for no response, `JevResponseException` for a 2xx that doesn't match the questions

---

## Target DSL

This is a sketch of the intent, not a literal specification. If you can find a better shape
with the same properties (type-safe, declarative, pleasant in the way Compose is), propose it —
but discuss it before implementing.

```kotlin
// What is required is a parameter; what is optional goes in the trailing lambda.
val jev = JevClient(apiKey = System.getenv("TYPESAFE_API_KEY"), engine = CIO.create()) {
    model = "jev-1.13.0"     // default "jev-latest"
    timeout = 10.seconds     // default 10s
}

// The caller's own enum carries the descriptions: a constant without one is a compile error.
enum class Intent(override val description: String) : Criterion {
    REFUND("Refunds and cancellations"),
    TECHNICAL_SUPPORT("Bugs and technical problems"),
    GENERAL_INQUIRY("Anything else"),
}

// Questions are values, usable outside the decide block
val intentQ = choice<Intent>("intent", "Which team should handle this request?")

val angryQ = noul("is_angry", "Is the customer expressing anger?") {
    whenTrue = "Clear irritation or forceful tone"
    whenFalse = "Neutral or calm"
}

// Declaration order is the rubric order, lowest level first.
enum class Urgency(override val description: String) : Criterion {
    LATER("Not time-sensitive"),
    TODAY("Should be handled today"),
    NOW("Needs immediate attention"),
}

val urgencyQ = score<Urgency>("urgency", "How urgently does this need a response?")

// The state is any @Serializable value: a String, or a structure sent as a JSON object/array.
@Serializable
data class Ticket(val subject: String, val body: String, val priorContacts: Int)

val result = jev.decide(ticket) {
    ask(intentQ, angryQ, urgencyQ)
}

val intent: Intent = result[intentQ].value           // typed as Intent, no cast
val dist: Map<Intent, Double> = result[intentQ].probabilities
val conf: Double = result[intentQ].confidence

val angry: Double = result[angryQ]                    // the probability itself; no .confidence

val urgency = result[urgencyQ]                        // no .value - see the Score note below
val mean: Double = urgency.score                      // probability-weighted mean of the level numbers
val likely: Urgency = urgency.mostLikely              // the mode of the distribution
val levels: Map<Urgency, Double> = urgency.probabilities

val tokens: Int = result.usage.inputTokens            // every response carries usage
val id: String? = result.requestId                    // for support requests

// Everything a decision throws is a JevException; catch narrower where you can act narrower.
try {
    jev.decide(ticket) { ask(intentQ) }
} catch (e: JevRateLimitException) {
    e.retryAfter                                      // the server's requested wait, if any
} catch (e: JevApiException) {
    e.status; e.requestId
}
```

Notes:
- **Required things are parameters, optional things are in the lambda.** `instructions`, the
  API key, the engine, and the state are parameters, so forgetting one is a compile error; a
  `var` inside a lambda can only be checked at runtime. This is the same reasoning as `Criterion`.
  Where nothing is optional - a Choice or Score over a `Criterion` enum - there is no lambda.
- `decide(state) { ask(...) }` keeps the state as a parameter and the questions in the block so
  that the boundary between the two stays readable when several questions are asked.
- The primary form for Choice and Score is an `enum` implementing `Criterion`: the descriptions
  live on the enum, so adding a constant without one fails to compile, and the description sits
  next to what it describes. An escape hatch keeps the explicit map form
  (`Intent.REFUND describedAs "..."`, with an explicit label) for enums the caller doesn't own,
  `sealed interface` objects, or asking the same type with a different wording.
- In the `Criterion` form the Choice wire label is the constant's `name.lowercase()`
  (`TECHNICAL_SUPPORT` → `technical_support`). The model reads labels as text next to the
  descriptions, and every official example uses lowercase words. The `Criterion` form has no
  label override on purpose; when a different label is needed, use the explicit form
  (`choice<T>(name, instructions, label = { ... }) { X describedAs "..." }`). Two constants that
  lowercase to the same label are rejected at construction.
- `ScoreAnswer` deliberately has no `value`. The API's answer is `score`, a `Double` mean over
  level numbers `0, 1, 2, ...` (declaration order); `mostLikely` is the highest-probability level
  (ties go to the lowest). Rounding the mean into a level is the caller's decision, never the
  library's. See `docs/api-notes.md` for the confirmed response shape.
- Confidence-gating helpers are welcome (e.g. `result[intentQ].orNull(minConfidence = 0.8)`),
  but ship no default threshold

Every question in one request is evaluated in parallel against the same state.
That is Jev's primary use, so asking several questions at once must be the natural thing to write.

---

## Naming

- The library is `kojev` (Maven: `io.github.itisnomatter:kojev`)
- Identifiers in code need not echo it. `JevClient` and `jev.decide { }` are fine —
  the same way Koin exposes `startKoin` and Ktor exposes `HttpClient`.
- **A public name that could plausibly exist in another library gets a `Jev` prefix; a name that
  is a Jev-specific concept does not.** Transport-level exceptions are generic
  (`JevBadRequestException`, `JevNotFoundException` — both exist in Ktor server;
  `JevRateLimitException`, `JevRequestTimeoutException`), so they are prefixed. Response-shape
  exceptions are Jev concepts (`MissingAnswerException`, `UnknownChoiceLabelException`), so they
  are not. kojev's audience is server-side Kotlin, where an import clash is a real cost.
  The test is the *name*, not the layer the class lives in: `JevUnreadableResponseException` sits
  next to `MissingAnswerException`, but "unreadable response" is something any HTTP client could
  name, so it is prefixed. Ask "could a Ktor, OkHttp, or kotlinx user already have this
  identifier imported?" — if plausibly yes, prefix.

---

## Legal and courtesy

- MIT licensed
- If code is ported or adapted from the official SDK, preserve the upstream copyright notice in `NOTICE`
- "TypeSafe" and "Jev" are trademarks of TypeSafe AI, Inc. State in the README that this project
  is unofficial and not affiliated with or endorsed by them
- Assume an official Kotlin SDK may appear later. Do not take a namespace that would collide with it.

---

## Working agreement

- Do not fill gaps by guessing. Read the docs, or ask.
- Prefer justified code over plausible code.
- If something in this document is wrong, say so rather than following it.
- Everything committed to this repository is written in English:
  documentation, code comments, commit messages, and PR descriptions.
  This holds regardless of the language of the conversation.

## Deferred decisions

Decisions that were consciously postponed live in GitHub Issues with the `deferred` label
(`gh issue list --label deferred`), each naming the phase that decides it. `docs/bootstrap.md`
makes every phase read its own before starting. Read the recorded reasoning before touching
anything one of them covers - do not re-decide from scratch. When you postpone something, record
it the same way; a PR description or a chat is not a record.

---
> Source: [ItisNoMatter/kojev](https://github.com/ItisNoMatter/kojev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
