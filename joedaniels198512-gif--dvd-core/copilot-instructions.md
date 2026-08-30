## fpga-compile

> FPGA compiles must use GitHub Actions only; never local Rosetta Quartus


# FPGA compile — GitHub Actions only

- **NEVER** run Quartus locally on this Mac.
- **NEVER** use `quartus-mister-rosetta`, local `docker run … quartus_sh`,
  or any amd64 Quartus under Rosetta. It does not work for us.
- **ALWAYS** compile via GitHub Actions `.github/workflows/build-core.yml`.
  Push the FPGA source, then `gh workflow run build-core.yml --ref <branch>`
  (or rely on the `fpga/**` push trigger). Download the `DVD.rbf` artifact.

---
> Source: [joedaniels198512-gif/dvd-core](https://github.com/joedaniels198512-gif/dvd-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
