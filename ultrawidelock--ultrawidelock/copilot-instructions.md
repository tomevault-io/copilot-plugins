## ultrawidelock

> UltraWideLock is an embedded C SDK and a set of complete smart-lock applications.

# UltraWideLock agent guide

UltraWideLock is an embedded C SDK and a set of complete smart-lock applications.
Portable behavior lives in `modules/`. Framework and hardware translation lives
in `ports/`. Applications select both without copying their implementation.

## Start here

| Task | Read first | Primary verification |
|---|---|---|
| Use the C API | `README.md` | `make sdk-check` |
| Port a chipset | `PORTING.md` | `make check`, then the affected target build |
| Change portable behavior | `modules/README.md` | `make check` |
| Change a backend | `ports/README.md` | `make check`, then that port's build |

Run `make help` for supported build and hardware commands. Search with `rg` or
`rg --files`; role manifests under `modules/*/roles/` are the source of truth
for shared source membership. Grep `apps/*/size-baseline*.json` and
`modules/ultrawidelock_dw3000/dwt_uwb_driver/` rather than reading them whole; they are
generated or vendored bulk, not prose.

## Architecture contract

1. `modules/` names no operating system. Public headers are in `include/` and
   private headers and implementation are in `src/`.
2. `ports/zephyr/` names only Zephyr. `ports/esp32/` names only ESP-IDF.
3. `modules/ultrawidelock_port/include/` is headers-only. Backend implementations live in
   a port tree or in `tests/host/port/`.
4. A source must not include another module's `src/`, and build files must not
   propagate a module `src/` include directory.
5. Add shared sources to one role manifest. Do not duplicate source lists in a
   consuming build file.

The application-facing C headers are `<ultrawidelock/reader.h>`,
`<ultrawidelock/device.h>`, `<ultrawidelock/uwb.h>`, and `<ultrawidelock/tlv.h>`. The
all-in-one `<ultrawidelock/ultrawidelock.h>` is for package consumers that intentionally
want every declaration. The chipset contract is `<ultrawidelock/ultrawidelock_hal.h>`. The
HAL names five seams: DW3000 GPIO/IRQ, DW3000 SPI, BLE GATT/L2CAP, BLE central,
and credential storage.

Canonical declarations live in the owning module's `include/ultrawidelock/`
directory. All production and test code uses the namespaced form. The SDK gate
rejects reintroducing the removed flat role-header names.

`VERSION` is the SDK version source. While the SDK is pre-1.0, the generated
CMake package accepts only the same minor series. After changing it, prove both
source-tree and installed consumption with `make sdk-check`.

Do not create a chipset-named top-level port when the chipset still uses a
supported framework. Extend the relevant framework port and keep board policy
in the application. A new operating system is a separate effort because it must
also implement the OSAL, flash, logging, and base port contracts.

## Change discipline

- Keep edits minimal and preserve unrelated working-tree changes.
- Do not edit `modules/ultrawidelock_dw3000/dwt_uwb_driver/` or
  `modules/ultrawidelock_dfu/src/detools/`.
- Do not change `mk/cdk.mk:208`.
- Do not weaken a test, purity rule, or ratchet allowlist to obtain a pass.
- A stale allowlist entry is a failure and must be removed with the change that
  made it unnecessary.
- Do not add private information, credentials, machine paths, or personal
  identity to files, output, or commits.
- Do not push, publish, or modify a remote without explicit permission for that
  action in the current session.

## Verification receipts

Use the narrow check first, then verify in proportion to risk:

```sh
make sdk-check
bash tests/tooling/port_purity_check.sh --self-test
make check
```

Target builds are `make build`, `make nrf-build`, `make anchor-pair`,
`make nrf-init-build`, and `make esp-build`. ESP port integration also has
`bash tests/ports/esp32/verify_port.sh`. The two Zephyr port checks,
`tests/ports/zephyr/matter_srp_lifecycle_check.sh` and
`tests/ports/zephyr/ble_link_liveness_check.sh`, are run by `make check` itself.
`make hitl` runs the attached-reader
and nRF5340 DK end-to-end flow. Hardware tests remain separate from host and
compile checks.

---
> Source: [ultrawidelock/ultrawidelock](https://github.com/ultrawidelock/ultrawidelock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
