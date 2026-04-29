## sajtmaskin

> Windows/PowerShell och några repo-specifika runtimefallgropar


# Platform quirks

Använd den här regeln bara när uppgiften faktiskt är plattforms-, shell-, Playwright-, stream- eller preview-skrivningskänslig.

## PowerShell / Windows

- **PowerShell på Windows**: bash-heredocs som `<<'EOF'` fungerar inte nativt.
- **Git commit i PowerShell**: använd `@' ... '@ | git commit -F -` eller flera `-m`; använd inte bash-syntax.
- **PowerShell pipe -> Vercel env**: `echo 'value' | npx vercel env add ...` lägger ofta till `\n` i värdet. Föredra temp-fil + `cmd /c "type ..."`.
- **Quoted one-liners i PowerShell**: snabba `node -e` / `python -c`-kommandon med många citattecken, `\n`, regexar eller JSON-strängar går lätt sönder i PowerShell även när kommandot hade fungerat i bash. Föredra att:
  - använda repoets vanliga scripts/testkommandon när möjligt
  - läsa/ändra riktiga filer i stället för att bygga långa inline-snippets
  - hålla inline-kommandon enkla och undvika tätt nestade `"..."` + `'...'` + backslashes

## Repo- och runtimefallgropar

- **Vercel Sandbox** (sekundär/verifiering, inte primär preview): föredra `sandbox.writeFiles([{ path, content: Buffer.from(str, "utf-8") }])` för större filträd.
- **Playwright lokalt**: Chromium kan behöva installeras separat med `npx playwright install chromium`.
- **Stream proxy**: kalla inte `reader.releaseLock()` medan `read()` väntar; avbryt via `reader.cancel()` i `ReadableStream.cancel()`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jakeminator123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
