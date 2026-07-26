## android-embeddedserver

> Best practices for WorkManager usage in background tasks

- Use `CoroutineWorker` for suspendable background work.
- Keep work inputs and outputs small and serializable.
- Chain dependent work using `WorkContinuation`.
- Use appropriate `Constraints` for battery and network considerations.
- Monitor and handle work statuses for retries and failures gracefully.
- Avoid long-running tasks; break them into smaller units if needed.
- Clean up or cancel obsolete work to prevent resource leaks.

---
> Source: [nphausg/android.embeddedserver](https://github.com/nphausg/android.embeddedserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
