## post-push-mirror

> Post-push: also push to mirror remote when configured


# Post-Push Mirror Remote

- After any successful `git push`, also run `git push mirror`.
- Only do this if a remote named `mirror` exists (check `git remote -v`).
- If `mirror` is not configured, ignore this rule.

---
> Source: [pRizz/opencode-cloud](https://github.com/pRizz/opencode-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
