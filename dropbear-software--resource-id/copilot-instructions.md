## resource-id

> The `resource_id` package implements a secure, human-friendly resource identifier scheme based on Crockford's Base32 encoding and Modulo-37 checksums. It is designed to be URL-safe, easy to communicate, and robust against typos.

# Resource ID Package

## Purpose
The `resource_id` package implements a secure, human-friendly resource identifier scheme based on Crockford's Base32 encoding and Modulo-37 checksums. It is designed to be URL-safe, easy to communicate, and robust against typos.

## Implementation Details

### Core Logic
*   **Encoding**: Uses `base32_codec` with the `Crockford` variant.
*   **Checksum**: Custom implementation in `lib/src/checksum.dart`. Calculates `bytes % 37`.
*   **Class**: `ResourceId` in `lib/src/resource_id.dart` encapsulates the logic.
    *   **Generation**: Uses `dart:math` `Random.secure()`.
    *   **Parsing**: Splits input string by `/`. Treats the last segment as the ID payload + checksum. Everything prior is the `type`.
    *   **Validation**: Decodes the Base32 payload, recalculates checksum, and compares with the check digit.

### File Structure
*   `lib/resource_id.dart`: Main export file.
*   `lib/src/resource_id.dart`: Core `ResourceId` class.
*   `lib/src/checksum.dart`: Checksum algorithms and character constants.
*   `test/`: Unit tests covering generation, parsing, checksums, and edge cases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dropbear-software)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dropbear-software)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
