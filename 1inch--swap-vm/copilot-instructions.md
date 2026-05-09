## 00-base

> VM-based swap protocol where orders are bytecode programs.


# SwapVM

VM-based swap protocol where orders are bytecode programs.

## Program Encoding

```
[opcode: 1 byte][argsLength: 1 byte][args: argsLength bytes]
```

## Key Structures

- `VM` - program counter, opcodes array, static context flag
- `Context` - VM + query + swap registers
- `SwapRegisters` - balanceIn/Out, amountIn/Out

## Swap Flow

1. `router.swap(order, tokenIn, tokenOut, amount, takerData)`
2. Extract program from `order.traits.program(order.data)`
3. VM executes bytecode
4. Handle transfers and hooks

## Key Locations

- Core VM: `src/SwapVM.sol` (abstract base)
- Instructions: `src/instructions/`
- Opcodes: `src/opcodes/` (Opcodes, LimitOpcodes, AquaOpcodes)
- Libraries: `src/libs/` (VM.sol, MakerTraits.sol, TakerTraits.sol)
- Routers: `src/routers/`
- Interfaces: `src/interfaces/`

---
> Source: [1inch/swap-vm](https://github.com/1inch/swap-vm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
