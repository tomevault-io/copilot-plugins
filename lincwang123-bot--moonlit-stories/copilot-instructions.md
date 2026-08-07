## moonlit-stories

> When the user asks to run or start this program, run this single command from the repository root:

# Moonlit Stories project instructions

## Run requests

When the user asks to run or start this program, run this single command from the repository root:

```bash
npm start
```

- Keep the returned local service process running while the user uses the app.
- Wait for the launcher to print the ready URL. It normally opens the browser automatically; if it cannot, open that exact `http://127.0.0.1:<port>/` URL for the user.
- Do not make the user run `setup`, `doctor`, `build`, or `local:start` separately. The launcher handles those stages and skips completed work.
- If the launcher reuses an existing Moonlit service, do not stop or replace that service.
- Never open the production form through `file://`; the local workflow and parent-voice features require the HTTP service URL.

## Continue a submitted book

When the user says the form is finished or asks to continue the submitted picture book:

1. Read `skills/moonlit-book-workflow/SKILL.md` completely if `$moonlit-book-workflow` is not already available in the current Codex task.
2. Run `npm run book:workflow -- list`, then inspect the exact newly submitted job with `status` and its private manifest. Never create a replacement job for an existing form submission.
3. Follow the Skill checkpoints for the 12-page story, character sheet, fixed-style illustrations, authorized-adult local TTS, validation, and self-contained release.
4. Preserve all guardian approval gates. Automation must not bypass the page-1 voice audition or permission to share the child's display name and generated likeness.

The browser cannot invoke Codex built-in image generation by itself. The local form creates a resumable job and presents an exact continuation instruction; image generation continues inside the user's Codex task.

## Privacy

Never commit or expose `.moonlit/`, `.local-models/`, `.venv-tts/`, `dist/`, voice references, consent files, real child profiles, or generated private books. Do not terminate unknown processes or delete another running Moonlit instance's data.

---
> Source: [lincwang123-bot/moonlit-stories](https://github.com/lincwang123-bot/moonlit-stories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
