## mpc2000-2000xl-mod-pack

> - Keep exactly the eight feature source files under mods/. Integrate corrections into the relevant feature; do not add historical candidates or separate fix packages.

# Working on the final eight mods

- Keep exactly the eight feature source files under mods/. Integrate corrections into the relevant feature; do not add historical candidates or separate fix packages.
- Do not use skills, build, test, regenerate CMake projects, assemble firmware, run installers, launch apps or emulators, or execute firmware without the user's explicit instruction. Static reading and syntax inspection are allowed.
- Preserve unrelated edits and every original input. Never overwrite input firmware or existing build outputs.
- Do not publish firmware, ROMs, HFE/IMG media, user samples, credentials or signing files.
- Any future firmware integration must validate exact input identities and expected bytes. Never assume that matching filenames imply matching firmware.
- Classic uses 29-byte note records; XL uses 24-byte records. Keep file offsets, segment-relative offsets and RAM addresses distinct. Do not transfer offsets between models.
- Preserve Classic MZ relocations, boot chaining, footer and RAM ownership. Preserve XL size, checksum, bootloader, native program arena and MIDI Sample Dump code.
- Keep realtime paths bounded; review PCM cache lifetime, voice ownership and Note Off pairing together.
- This is a source collection, without the complete firmware integration recipe. Do not claim that the eight assembly files alone reproduce a working OS.
- Keep shared routines, section selectors and model-specific dependencies explicit when editing the source.
- Distinguish static byte evidence, actual assembly, emulator results, audio and hardware verification. No assembly or runtime validation was performed for this source-only package.
- Retain the PolyForm Noncommercial license and required copyright notice. Do not claim rights to third-party firmware or describe this release as unrestricted open source.

---
> Source: [dariuszchynek/mpc2000_2000xl_mod_pack](https://github.com/dariuszchynek/mpc2000_2000xl_mod_pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
