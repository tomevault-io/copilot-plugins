## laravel-ai-transcribe

> Amazon Transcribe (batch) driver for the official `laravel/ai` SDK.

# clinically/laravel-ai-transcribe

Amazon Transcribe (batch) driver for the official `laravel/ai` SDK.

## Architecture

- `AiTranscribeServiceProvider` registers the `aws-transcribe` driver via
  `Ai::extend()`. No config file, no migrations, no views — connection config
  lives in the consuming app's `config/ai.php`.
- `TranscribeProvider` implements `Laravel\Ai\Contracts\Providers\TranscriptionProvider`
  using the SDK's `GeneratesTranscriptions` + `HasTranscriptionGateway` traits.
  It mirrors the SDK's `BedrockProvider` constructor shape
  `(array $config, Dispatcher $events)`.
- `TranscribeGateway` implements `Laravel\Ai\Contracts\Gateway\TranscriptionGateway`:
  upload audio to S3 → `StartTranscriptionJob` → poll with capped backoff →
  parse transcript JSON → **always** delete both S3 objects in `finally`.
  Timeout also issues `DeleteTranscriptionJob`. The gateway requires the
  provider to be a `TranscribeProvider` (guarded instanceof).
- `TranscriptParser` is pure: Transcribe result JSON → `ParsedTranscript`
  (text + `TranscriptionSegment` collection from `results.audio_segments`).

## Invariants

- S3 objects are ephemeral. Never persist audio or transcripts in the bucket.
- `model === 'standard'` means no `ModelSettings`; any other model name is sent
  as a custom language model (`ModelSettings.LanguageModelName`).
- `$providerOptions` are merged into the job request **last** via
  `array_replace_recursive` — they win over everything. That means they can
  redirect `OutputBucketName`/`OutputKey` away from the ephemeral-cleanup
  path: they are a trusted developer escape hatch, never expose them to
  untrusted input.
- Cleanup failures log warnings; they never mask the transcription result or error.

## Commands

- `composer test` — Pest
- `composer analyse` — PHPStan level 6 (memory limit 1G)
- `composer format` — Pint (phpstan.cache excluded)

## Testing conventions

AWS calls are mocked with `Aws\MockHandler` via helpers in `tests/Pest.php`
(`mockedHandler`, `mockedS3`, `mockedTranscribe`, `makeProvider`). `Sleep::fake()`
keeps polling tests instant. No network access in tests, ever. Note:
`Ai::transcriptionProvider()` (no args) resolves `ai.default`, while the
`Transcription::...` flow resolves `ai.default_for_transcription` — the
TestCase sets both to `aws`.

---
> Source: [clinically-au/laravel-ai-transcribe](https://github.com/clinically-au/laravel-ai-transcribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
