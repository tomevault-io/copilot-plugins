## dicom-ts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

Implement the DICOM standard as a TypeScript library. The `source-code/fo-dicom/` directory contains the **Fellow Oak DICOM** C# library (v5.2.5, targeting .NET Standard 2.0) as the reference implementation to port from.

## Repository Structure

```
dicom-ts/
|- source-code/fo-dicom/         # Reference C# implementation (READ-ONLY)
|  |- FO-DICOM.Core/            # Core DICOM classes to port
|  |- Platform/                 # Platform-specific rendering
|  |- Serialization/            # JSON serialization
|  `- Tests/                    # C# tests as reference cases
|- source-code/fo-dicom.Codecs/  # Codec-focused companion repo (READ-ONLY)
|  |- Codec/                    # C# codec manager + transfer syntax bindings
|  |- Native/                   # Native wrappers + third-party codec libs
|  `- Tests/                    # Acceptance fixtures for compressed syntaxes
`- src/                         # TypeScript source
```

## TypeScript Project Setup

No TypeScript project exists yet. When initializing:
- **Package manager: npm only** (not bun/pnpm/yarn)
- `npm init` — package.json with `"type": "module"`
- Target: ES2020, `"moduleResolution": "node16"`
- Output: dual CJS + ESM build using **two separate `tsconfig` files + plain `tsc`** (do NOT use tsup/esbuild/rollup — they carry native bindings that break on ARM/Dameng environments)
- devDependencies: **only 3 packages** — `typescript`, `vitest`, `@types/node`

## Dependency Policy — Minimize Third-Party Packages

**Runtime dependencies: zero npm packages.** Implement everything using Node.js built-ins:
- Deflate/zlib → `node:zlib`
- Charset decoding → Node.js built-in `TextDecoder` / `TextEncoder`
- UID generation → `node:crypto` (`randomUUID`, `randomBytes`)
- Network (DIMSE) → `node:net` / `node:tls`
- File I/O → `node:fs/promises`

**Codec strategy** (compression formats that cannot be trivially ported):
- RLE Lossless → pure TypeScript implementation
- JPEG Lossless Process 14 → pure TypeScript port of fo-dicom's `JpegLossless/` directory
- JPEG Lossy / JPEG2000 / HTJ2K → define `IDicomCodec` plugin interface only; users register their own codec via `TranscoderManager.register()` — no built-in implementation

Additional codec references:
- `source-code/fo-dicom/FO-DICOM.Core/Imaging/Codec/`: pure C# baseline behavior
- `source-code/fo-dicom.Codecs/Codec/`: transfer syntax coverage + codec registration patterns
- `source-code/fo-dicom.Codecs/Native/`: native interoperability behavior reference only (do not introduce native runtime dependency in `dicom-ts`)

Required codec transfer syntax coverage for Phase 10.5:
- JPEG Family
  - `1.2.840.10008.1.2.4.50` JPEG Baseline (Process 1), lossy 8-bit
  - `1.2.840.10008.1.2.4.51` JPEG Extended (Process 2 & 4), lossy 8/12-bit
  - `1.2.840.10008.1.2.4.57` JPEG Lossless (Process 14), all predictors
  - `1.2.840.10008.1.2.4.70` JPEG Lossless SV1 (Process 14, predictor 1)
- JPEG-LS Family
  - `1.2.840.10008.1.2.4.80` JPEG-LS Lossless
  - `1.2.840.10008.1.2.4.81` JPEG-LS Near-Lossless
- JPEG 2000 Family
  - `1.2.840.10008.1.2.4.90` JPEG 2000 Lossless
  - `1.2.840.10008.1.2.4.91` JPEG 2000 (lossy/lossless)
  - `1.2.840.10008.1.2.4.92` JPEG 2000 Multi-component Lossless
  - `1.2.840.10008.1.2.4.93` JPEG 2000 Multi-component

**Logger strategy:** define `IDicomLogger` interface; built-in console/null implementations only — no winston/pino.

This policy exists to ensure the library compiles and runs on ARM, domestic chips, and database-integrated environments (Dameng DB, Kingbase, etc.) without native module compilation issues.

## fo-dicom Reference Architecture

### Core Class Hierarchy (source-code/fo-dicom/FO-DICOM.Core/)

```
DicomFile                          → top-level container; wraps metadata + dataset
├── FileMetaInfo (DicomFileMetaInformation)
└── Dataset (DicomDataset)
    └── _items: SortedList<DicomTag, DicomItem>
        ├── DicomElement              → single data element with tag + VR + buffer
        │   ├── DicomStringElement    → string-valued elements
        │   └── DicomValueElement<T>  → numeric/binary-valued elements
        └── DicomSequence             → nested sequence of DicomDataset[]

DicomTag          → (group: ushort, element: ushort) pair; odd group = private
DicomVR           → value representation descriptor (LO, OB, SQ, UI, etc.)
DicomTransferSyntax → encoding scheme (ExplicitVRLittleEndian, JPEG, etc.)
DicomDictionary   → maps DicomTag → DicomDictionaryEntry (VR, name, VM)
```

### Reading Pipeline

```
DicomFile.Open(path)
  → DicomFileReader
    → DicomReader (IsExplicitVR, IsDeflated)
      → DicomReaderWorker  [observer pattern]
        → IDicomReaderObserver  [callbacks that build DicomDataset]
```

Key files:
- [FO-DICOM.Core/IO/Reader/DicomReader.cs](source-code/fo-dicom/FO-DICOM.Core/IO/Reader/DicomReader.cs)
- [FO-DICOM.Core/IO/Buffer/](source-code/fo-dicom/FO-DICOM.Core/IO/Buffer/) — IByteBuffer, LazyByteBuffer, BulkDataUriByteBuffer

### Network (DIMSE) Services

Base class: [FO-DICOM.Core/Network/DicomService.cs](source-code/fo-dicom/FO-DICOM.Core/Network/DicomService.cs)
- Manages PDU (Protocol Data Unit) queue and DICOM message queue
- Services: C-ECHO, C-STORE, C-FIND, C-MOVE, C-GET, N-ACTION

### Serialization

JSON DICOM serialization: [Serialization/FO-DICOM.Json/](source-code/fo-dicom/Serialization/FO-DICOM.Json/)

### Codec References (for Phase 10.5)

- `source-code/fo-dicom/FO-DICOM.Core/Imaging/Codec/`
  - Pure C# codec and transcoder abstractions; primary porting source for JPEG Lossless Process 14.
- `source-code/fo-dicom.Codecs/Codec/`
  - Production plugin implementations for JPEG/JPEG-LS/JPEG2000/HTJ2K and codec registration.
- `source-code/fo-dicom.Codecs/Native/`
  - Native bridges and bundled codec dependencies (`libijg*`, `CharLS`, `OpenJPEG`, `OpenJPH`).
  - Use as compatibility reference, not as a direct dependency target for this TypeScript project.

## Key DICOM Concepts to Implement (in order of priority)

1. **DicomTag** — group/element pair, private tag detection, UID comparison
2. **DicomVR** — all 27 standard VRs with length rules, padding, byte-swap behavior
3. **DicomDictionary** — parsed from `FO-DICOM.Core/Dictionaries/DICOMDictionary.xml.gz`
4. **DicomDataset** — sorted tag→item map, transfer syntax tracking, SQ nesting
5. **DicomElement subtypes** — one class per VR (or generic `DicomValueElement<T>`)
6. **IO/Reader** — parse binary DICOM (preamble + DICM magic, explicit/implicit VR, little/big endian)
7. **IO/Writer** — serialize dataset back to binary
8. **DicomFile** — file meta information (group 0002), open/save API
9. **DicomUID** — UID registry, SOP classes, transfer syntaxes
10. **DicomTransferSyntax** — encoding descriptor tied to dataset
11. **JSON serialization** — DICOM JSON model (PS3.18 Annex F)
12. **Network/DIMSE** — PDU protocol, association negotiation, service classes

## Important Implementation Notes from fo-dicom

- **Tag ordering**: `DicomDataset` uses a sorted dictionary; tags must be in ascending (group, element) order
- **Private tags**: group number is odd; require a `PrivateCreator` block at `(gggg,00xx)` before use
- **Transfer syntax cascades**: changing `InternalTransferSyntax` on a dataset must recursively update nested sequences
- **FileReadOption**: support lazy/on-demand loading for pixel data (`7FE0,0010`) — large tags can be skipped or deferred
- **Value Multiplicity (VM)**: multiple values in non-SQ elements are backslash-delimited in strings, or packed in binary
- **Endianness**: almost all modern DICOM is Little Endian; Big Endian (retired) exists in older files
- **Deflated transfer syntax**: `DeflatedExplicitVRLittleEndian` — dataset bytes are zlib-deflated after group 0002
- **DicomDictionary** is a singleton loaded once; private dictionaries can be registered additionally
- **DI pattern**: fo-dicom uses `Microsoft.Extensions.DependencyInjection`; TypeScript port can use simple service locator or exported singletons

## Navigating the C# Reference

- All standard DICOM tags as constants: [FO-DICOM.Core/DicomTag.Generated.cs](source-code/fo-dicom/FO-DICOM.Core/DicomTag.Generated.cs)
- All standard UIDs: [FO-DICOM.Core/DicomUID.Generated.cs](source-code/fo-dicom/FO-DICOM.Core/DicomUID.Generated.cs)
- All VR element types: [FO-DICOM.Core/DicomElement.cs](source-code/fo-dicom/FO-DICOM.Core/DicomElement.cs)
- Network service examples: [FO-DICOM.Core/Network/Client/](source-code/fo-dicom/FO-DICOM.Core/Network/Client/)
- Test cases for parsing edge cases: [Tests/FO-DICOM.Tests/](source-code/fo-dicom/Tests/FO-DICOM.Tests/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cocosip) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
