## nabq

> Nabq.ai is a multilingual AI-powered meditation app. Users chat with an AI that understands their needs, then a personalized meditation is generated and spoken in their dialect. Supports Arabic (Saudi, Gulf, Egyptian, Tunisian), English (US, UK), Turkish, and French.

# Nabq.ai — Gemini CLI Project Context

## Project Overview
Nabq.ai is a multilingual AI-powered meditation app. Users chat with an AI that understands their needs, then a personalized meditation is generated and spoken in their dialect. Supports Arabic (Saudi, Gulf, Egyptian, Tunisian), English (US, UK), Turkish, and French.

## Your Role: Audio, TTS & Infrastructure
You handle text-to-speech integration, audio processing, cloud storage, CI/CD pipelines, and DevOps.

## Architecture
- Framework: Next.js 15 (App Router) — all code in `apps/web/`
- TTS Primary: Azure Cognitive Speech (Arabic dialects + Turkish)
- TTS Premium: ElevenLabs (English + French + premium voices)
- TTS Dialect: Lahajati.ai (deep Arabic dialect support)
- Audio: FFmpeg for mixing and encoding
- Storage: DigitalOcean Spaces (S3-compatible) with CDN
- CI/CD: GitHub Actions
- Language: TypeScript strict mode
- Testing: Vitest

## Your File Ownership
**You OWN:**
- `apps/web/src/lib/tts/**/*` — All TTS provider integrations
- `apps/web/src/lib/audio/**/*` — Audio mixing, encoding, ambient tracks
- `apps/web/src/lib/storage/**/*` — DigitalOcean Spaces helpers
- `.github/workflows/**/*` — CI/CD pipelines
- `docs/**/*` — Technical documentation
- `infrastructure/**/*` — Docker, deployment configs

**You DO NOT edit:**
- `apps/web/src/components/**/*` (Codex's domain)
- `apps/web/src/app/[locale]/**/*` page files (Codex's domain)
- `apps/web/src/lib/ai/**/*` (Claude's domain)
- `apps/web/src/lib/supabase/**/*` (Claude's domain)
- `apps/web/src/app/api/**/*` EXCEPT `apps/web/src/app/api/meditation/tts/route.ts` (you own this one)

## Coding Standards
- TypeScript strict mode — NEVER use `any`
- All TTS providers MUST implement the `TTSProvider` interface in `src/types/tts.ts`
- Audio output standard: AAC 128kbps, normalized to -16 LUFS, 44.1kHz
- All audio files uploaded to Spaces MUST use CDN URLs
- SSML must handle Arabic diacritization preprocessing (add tashkeel before TTS)
- Voice selection logic lives in `tts-router.ts` — route by locale
- Environment variables: `process.env.VARIABLE_NAME`, never hardcode secrets
- Tests: colocate test files as `*.test.ts` next to source files

## Voice Catalog
| Locale | Provider | Female Voice | Male Voice |
|--------|----------|-------------|------------|
| ar-SA | Azure | ZariyahNeural | HamedNeural |
| ar-GCC | Azure | FatimaNeural (ar-AE) | HamdanNeural (ar-AE) |
| ar-EG | Azure | SalmaNeural | ShakirNeural |
| ar-TN | Azure | ReemNeural | HediNeural |
| en-US | ElevenLabs | TBD from voice library | TBD |
| en-GB | ElevenLabs | TBD from voice library | TBD |
| tr | Azure | EmelNeural | AhmetNeural |
| fr | ElevenLabs | TBD from voice library | TBD |

## Audio Pipeline
```
MeditationScript JSON
  → SSML Builder (segments → provider-specific SSML with pauses, prosody)
  → TTS API call (returns raw audio)
  → FFmpeg mixing (layer voice over ambient track)
  → Normalize to -16 LUFS
  → Encode to AAC 128kbps
  → Upload to DigitalOcean Spaces
  → Return CDN URL
```

## Environment Variables
All keys are defined in `.env.example` at the repo root (single source of truth).
Your scope:
- `AZURE_SPEECH_KEY`, `AZURE_SPEECH_REGION` — Azure TTS for Arabic + Turkish
- `ELEVENLABS_API_KEY` — ElevenLabs TTS for English + French
- `LAHAJATI_API_KEY` — Lahajati deep Arabic dialect TTS
- `DO_SPACES_KEY`, `DO_SPACES_SECRET`, `DO_SPACES_BUCKET`, `DO_SPACES_REGION`, `DO_SPACES_CDN_URL` — DigitalOcean Spaces for audio storage

## Before Finishing
```bash
npm run typecheck  # Must pass with zero errors
npm run lint       # Must pass
npm run test       # Must pass
```

---
> Source: [alturkestani/nabq](https://github.com/alturkestani/nabq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
