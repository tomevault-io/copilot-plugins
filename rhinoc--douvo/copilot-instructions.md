## douvo

> This repository is a native macOS app. Treat changes carefully because the app touches microphone input, Accessibility permissions, clipboard state, local Doubao credentials, optional local MLX models, and optional remote LLM providers.

# Agent and Contributor Notes

This repository is a native macOS app. Treat changes carefully because the app touches microphone input, Accessibility permissions, clipboard state, local Doubao credentials, optional local MLX models, and optional remote LLM providers.

## Default workflow

- Inspect relevant files before editing. Do not guess about prompt, model, signing, or permission behavior.
- Keep changes scoped. Do not reformat or refactor unrelated files.
- The worktree may already contain user or other-agent changes. Do not revert them unless explicitly asked.
- Prefer `rg` for search.
- Use `swift test` for code changes when feasible.
- For prompt or correction behavior changes, run a focused Prompt Lab config when feasible:

```bash
swift run Douvo --prompt-lab docs/local-llm-eval/prompt-lab.sample.json
```

## Do not run the user app process casually

- Do not start, stop, kill, or restart a user's running Douvo app unless explicitly asked.
- `swift test` and `swift run Douvo --prompt-lab ...` are acceptable verification commands.
- If UI/manual testing is required, say exactly what must be run and why before doing it.

## Code signing matters

Do not bypass signing with ad-hoc signatures. A locally built `.app` needs a stable code-signing identity because macOS Accessibility permissions and app identity are signature-sensitive. Ad-hoc builds can appear to work once and then fail after rebuilds or permission changes.

`scripts/build-app.sh` intentionally refuses ad-hoc signing. It uses one of:

- `CODESIGN_IDENTITY` set to a valid local signing identity.
- A local keychain identity named `Douvo Local Code Signing`.
- `scripts/ensure-local-code-signing-identity.sh`, run explicitly, which creates `Douvo Local Code Signing` in a Douvo-specific local keychain.

Never set `CODESIGN_IDENTITY="-"` to make a build pass. That produces an ad-hoc signature and invalidates the permission model this project depends on.

Check available identities:

```bash
security find-identity -v -p codesigning
```

Create or verify the local identity:

```bash
scripts/ensure-local-code-signing-identity.sh
LOCAL_CODESIGN_KEYCHAIN="$HOME/Library/Application Support/Douvo/CodeSigning/douvo-local-code-signing.keychain-db"
security find-identity -v -p codesigning "$LOCAL_CODESIGN_KEYCHAIN" | rg "Douvo Local Code Signing"
```

Do not run `scripts/ensure-local-code-signing-identity.sh` implicitly from another script or agent workflow. Creating or trusting a code-signing identity can trigger macOS security authentication.

After the identity exists, build scripts may reuse the Douvo-specific local keychain without prompting for the user's login keychain password. Repeated password prompts usually mean the local signing keychain is stale or being recreated.

Then build:

```bash
./scripts/build-app.sh
open .build/release/Douvo.app
```

`swift run Douvo` does not require code signing, but it should only be used for quick development checks. Permission-sensitive behavior must be checked from a signed `.app` bundle.

If you use a different certificate name:

```bash
CODESIGN_IDENTITY="Your Code Signing Identity" ./scripts/build-app.sh
```

For local UI and permission testing, `scripts/install-dev-app.sh` builds and installs `/Applications/Douvo Dev.app` with bundle identifier `local.douvo.dev`. It may quit an existing dev app, replace the app bundle in `/Applications`, register it with LaunchServices, and open it. Do not run this script from an agent workflow unless the user explicitly asks for that action. Details live in `docs/dev-local-build.md`.

## Local MLX build notes

- `Package.swift` includes MLX, Hugging Face, Tokenizers, and Sparkle dependencies.
- `scripts/build-app.sh` runs `scripts/build-mlx-metallib.sh` and packages the MLX Metal library into the app bundle.
- If local model inference fails, inspect the MLX runtime diagnostic in Settings before changing model code.

## Prompt and correction changes

- Keep default prompt text short.
- Put broad safety constraints once in the global output requirements, not repeated in every optional branch.
- Do not hard-code a single Prompt Lab failure case into the general prompt.
- Prefer engineering candidates, vocabulary normalization, and unit tests before strengthening prompts.
- When changing model lists, update `README.md`, `README.zh.md`, and `docs/local-llm-eval`.

## Privacy boundaries

- Never log cookies, full credential JSON, remote API keys, or secret values.
- Remote LLM API keys must stay in Keychain.
- Prompt snapshots, traces, and Prompt Lab reports may contain transcript text. Keep them local unless the user explicitly chooses to share them.

---
> Source: [rhinoc/douvo](https://github.com/rhinoc/douvo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
