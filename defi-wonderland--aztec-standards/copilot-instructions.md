## aztec-standards

> Aztec contract design patterns


# Aztec Contract Design Patterns

## Contract Structure
- Define storage struct with appropriate visibility modifiers
- Group storage by type: immutable config, mutable state, private collections
- Implement standard interfaces consistently (token, NFT patterns)
- Use #[aztec] macro for contract definition
- Apply appropriate function decorators: #[public], #[private], #[view], #[utility]

## Common Patterns
- Commitment pattern: initialize_transfer_commitment returns Field
- Validation pattern: separate _validate_* functions for reuse
- Balance management: _increase_balance, _decrease_balance helpers
- Recursive operations: recurse_subtract_balance_internal for note limits
- Library methods: #[contract_library_method] for shared logic

## State Management
- Private state uses note-based system (UintNote, NFTNote)
- Public state uses traditional mappings
- Support state transitions in both directions
- Emit encrypted notes for private recipients
- Complete partial notes in public context

---
> Source: [defi-wonderland/aztec-standards](https://github.com/defi-wonderland/aztec-standards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
