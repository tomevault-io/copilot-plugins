## clean-user-facing-text

> Enforce invisible Unicode and steganography hygiene on user-facing text, manuscripts, and documentation


# Cursor Rule: Clean User-Facing Text Hygiene

When writing or editing user-facing copy, documentation, and manuscripts:
1. **Never emit invisible Unicode formatting carriers**: Disallow zero-width spaces (`U+200B`), zero-width non-joiners (`U+200C`), word joiners (`U+2060`), and byte order marks (`U+FEFF`) in ordinary prose.
2. **Standardize Spaces**: Use regular ASCII spaces (`U+0020`) instead of non-breaking spaces (`U+00A0`), thin spaces (`U+2009`), or mathematical spaces unless explicitly required for typography.
3. **Preserve Valid Code and Syntax**: Do not strip syntax tokens from code blocks or formula strings.
4. **Use ClaudeMark For Verification**: Verify clean text with `python claudemark.py unicode inspect <file>` or `python claudemark.py unicode visualize <file>`.

---
> Source: [karthikrshet/ClaudeMark](https://github.com/karthikrshet/ClaudeMark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
