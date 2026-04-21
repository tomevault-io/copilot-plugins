## audioanalyzer

> Never use empty try-catch blocks in C#


# No Empty Try-Catch

Never leave empty `catch` blocks. Either remove the try-catch or add real error handling (e.g. log, rethrow with context, or handle meaningfully).

```csharp
// BAD
try { DoWork(); } catch (Exception) { }
try { DoWork(); } catch { }

// GOOD: remove if not needed
DoWork();

// GOOD: add handling
try { DoWork(); }
catch (Exception ex)
{
    _logger.LogError(ex, "DoWork failed");
    throw;
}

// GOOD: intentional swallow with explicit comment (last resort)
try { Console.Write(x); }
catch (Exception ex) { /* Console unavailable: swallow to avoid crash */ }
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CzarSibelius) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
