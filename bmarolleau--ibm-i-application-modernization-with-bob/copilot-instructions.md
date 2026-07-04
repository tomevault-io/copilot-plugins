## ibm-i-application-modernization-with-bob

> This file provides guidance to agents when working with code in this repository.

# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Build System (Non-Standard)

- Build uses Tobi (`/QOpenSys/pkgs/bin/makei`) NOT gmake or standard make
- Build MUST set `export lib1=SAMCO` before running (`iproj.json` uses `&lib1` variable)
- Compile single file: `/QOpenSys/pkgs/bin/makei c -f QRPGLESRC/filename.PGM.SQLRPGLE`
- Full build: `/QOpenSys/pkgs/bin/makei build` (from `SAMCO/` directory)
- `Rules.mk` files define build dependencies — always check them before modifying source
- BNDDIR must be deleted before recreation (`SAMPLE.BNDDIR` line 4 explains why — `CRTBNDDIR` fails if it already exists)

## File Naming Conventions (Critical)

- Display files: `XXX###D-Description.DSPF` (e.g., `ART200D-Work_with_Article.DSPF`)
- Programs: `XXX###-Description.PGM.SQLRPGLE` (e.g., `ART200-Work_with_article.PGM.SQLRPGLE`)
- Modules (no `PGM`): `ART300-Function_Article.RPGLE` — absence of `.PGM.` = module, not program
- Prototypes: `NAME-Description.RPGLEINC` — `.RPGLEINC` avoids `*NONE` object type requirement
- OPM RPG lives in `QRPGSRC/` (legacy); ILE RPGLE lives in `QRPGLESRC/`

## CCSID Configuration (Per-Directory — Not Global)

- Root `SAMCO/.ibmi.json` sets `tgtCcsid: 37` but **subdirectories override this**:
  - `QRPGLESRC/`, `QSRVSRC/`, `QPROTOSRC/`: CCSID **297** (French)
  - `QCLSRC/`, `functionsVAT/`: CCSID **500** (EBCDIC International)
- Always check the subdirectory `.ibmi.json` before compiling — wrong CCSID causes silent data issues

## Code Patterns (Project-Specific)

- Panel-step pattern: Programs use `panel` (screen number) and `step##` (`prp`/`lod`/`dsp`/`key`/`chk`/`act`) — these are named constants defined at the top of each program
- Indicator data structures: All programs declare `indds` DS mapping indicator numbers to named fields (e.g., `exit 3 3n`, `sflclr 30 30n`) — never use raw `*IN##` in logic
- `/copy` uses bare names (`/copy familly`, `/copy article`) resolved via `iproj.json` `includePath: ["includes", "QPROTOSRC"]` — no path prefix, no extension
- Exception: When copy is outside `includePath`, use explicit path: `/copy qprotosrc/Xss`
- `SAMREF.PF` in `common/` is the central field-definition reference file — all physical files `REF(SAMREF)` for shared field definitions (types, lengths, edit codes)
- `REFFLD` in DDS references `SAMREF` records: e.g., `REFFLD(FCOUN/COID *LIBL/COUNTRY)`

## Service Program Architecture (Two Parallel Systems)

- `QSRVSRC/*.BND` + `QRPGLESRC/Rules.mk` → built by Tobi/makei (standard path)
- `QILESRVSRC/*.ILESRVPGM` → alternative CL-based build scripts (legacy/ARCAD-generated)
- Both systems coexist — `FARTICLE.SRVPGM` is in both; `Rules.mk` in `QILESRVSRC/` takes precedence for those service programs
- `SAMPLE.BNDDIR` aggregates all service programs — programs bind to `bnddir('SAMPLE')` not individual srvpgms

## SQL Conventions

- `TEXT` attribute in DDS becomes `LABEL ON` in SQL (documented in `QSQLSRC/readme.md`)
- SQLRPGLE programs check `SQLCODE` only (0=success, 100=not found) — never use SQLCA DS
- REST API service programs use `PGMINFO(*PCML : *MODULE : *DCLCASE)` for IWS integration
- SQL source types use distinct extensions: `.SQLPRC` (stored procedure), `.VIEW` (view), `.SQLUDF` (UDF), `.SQLSEQ` (sequence), `.SQLTRG` (trigger), `.TABLE` (table DDL)

## Build Object Library

- Code resides in `SAMCO/` directory, synchronized with IFS — always prefer local source
- Build objects default to `SAMCOx` libraries (user-specified suffix) — never use `SAMCO` library as target unless confirmed by user
- Only fall back to IBM i connection for source not present locally

## Dependencies (Hidden)

- `SAMPLE.BNDDIR` in `QBNDSRC/Rules.mk` depends on: XML, TXT, FARTICLE, FCUSTOMER, FFAMILLY, FPARAMETER, FPROVIDER, LOG, FCOUNTRY, ORDER, XSS — build these srvpgms before programs
- `QPROTOSRC/` prototypes are shared across many programs — changes here cascade broadly
- `functionsVAT/` is a self-contained subdirectory (own `.ibmi.json`, `Rules.mk`) that builds `FVAT.SRVPGM` — not listed in `QSRVSRC/Rules.mk`
- `CUS200.PGM` depends on `CUSSEQ.DTAARA` (a SQL sequence object built from `QSQLSRC/`)

---
> Source: [bmarolleau/IBM-i-Application-Modernization-with-Bob](https://github.com/bmarolleau/IBM-i-Application-Modernization-with-Bob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
