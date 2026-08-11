## pom1-aci-cassette-program-output

> ACI TAPE OUT ($C030) requires empty cassette deck — WOZ_talk blocks program output


# ACI program output vs cassette deck mode

Programs that drive the **ACI TAPE OUT flip-flop** (any read/write of `$C0xx`, e.g. `$C030` speaker in GEN2 *A-1-CrazyCycle*) need the deck in **program / pulse** mode, not **audio-stream** mode.

- Bundled `cassettes/WOZ_talk.mp3` preloads only on the **POM1 Multiplexing Fantasy (2026)** preset (last index). While inserted, `toggleOutput()` pulses are not mixed — chiptunes are silent.
- **DevBench Run** on presets with ACI and no Integer-BASIC tape (GEN2 dev bench = preset 2, …): `ejectTapeForAciProgramOutput()` in `Pom1BenchHost.cpp`.

Manual test: if a GEN2 demo has graphics but no ACI sound, eject the cassette in the Cassette Deck window.

---
> Source: [habib256/pom1](https://github.com/habib256/pom1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
