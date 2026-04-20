## tcc-puc

> Multiplayer tower defense game (Clash Royale-style) — university project (TCC - PUC).

# Project Context

Multiplayer tower defense game (Clash Royale-style) — university project (TCC - PUC).
Stack: Unity, Netcode for GameObjects, Odin Inspector, DOTween.

## User Preferences

- Values SOLID principles and scalable architecture
- Prefers discussion about design trade-offs before implementing
- Works across multiple PCs

## Architecture Decisions

### Card Validation System

Card activation uses a composable validation chain via `CardValidation` struct (not raw bool or Predicate<T>).

**Why:** Cards have shared base predicates (mana cost) and type-specific ones (buildable needs position check, spell does not). A bool loses the failure reason needed for UI feedback. A Predicate delegate loses Unity Inspector visibility and stack clarity.

**How it works:**
- `AbstractCard` implements `ICardActivatable` and owns the base mana check in `CanPlayCard()`
- `CanPlayCardAt()` defaults to `CanPlayCard()` — cards that don't need position checks (spells) inherit this for free
- Subclasses override and chain `base.CanPlayCard()` / `base.CanPlayCardAt()` to add their own checks (e.g. BuildableCard adds position + waitingResult)
- `CardValidation` struct carries `IsValid` + `CardInvalidReason` enum, with `implicit operator bool` for ergonomic if-checks
- `OnEndDrag` in AbstractCard handles the validate-then-activate flow — subclasses only override the three ICardActivatable methods
- Key files: `CardValidation.cs`, `ICardActivatable.cs`, `AbstractCard.cs`, `BuildableCard.cs` (all under `Assets/Scripts/Gameplay/Cards/`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kecek05) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
