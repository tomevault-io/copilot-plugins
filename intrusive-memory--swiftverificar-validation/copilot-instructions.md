## swiftverificar-validation

> Module name: **SwiftVerificarValidation**

# SwiftVerificar-validation — Agent Instructions

Module name: **SwiftVerificarValidation**

## What This Library Does

Core PDF validation engine for the SwiftVerificar ecosystem. Provides rule execution against parsed PDF documents, PD-layer validation types (annotations, fonts, color spaces, structure trees), SA-layer semantic accessibility types, feature extraction, metadata fixing, and serialization. This is the largest package in the suite with 238+ public types across 9 functional layers, 3050+ tests, completed over 19 sprints.

## Dependencies

- **SwiftVerificarValidationProfiles** (required) — provides profile/rule types
- **SwiftVerificarParser** (optional) — uses `#if canImport` with fallback stubs when not available

## Key Public Types

### Core

- `ValidationEngine` — protocol defining the validation contract
- `ValidationResult` — aggregate result of a validation run
- `RuleResult` — result of a single rule evaluation
- `ValidationError` — error types during validation
- `ValidationContext` — context passed through validation pipeline

### Engine

- `RuleExecutor` — executes rules against PDF objects
- `ObjectValidator` — validates individual PDF objects
- `ProfileRuleEvaluator` — evaluates rules from a validation profile
- `EvaluationContext` — context for rule evaluation

### Validators

- `PDFAValidator` — base PDF/A validator
  - `PDFA1Validator`, `PDFA2Validator`, `PDFA3Validator`, `PDFA4Validator`
- `PDFUAValidator` — base PDF/UA validator
  - `PDFUA1Validator`, `PDFUA2Validator`

### Object Model

- `PDFObject` — protocol for all PDF objects in the validation model
- `PropertyValue` — typed property access on PDF objects
- `ValidatedOperator` — enum with 70 operators across 14 categories (graphics state, path construction, path painting, clipping, text state, text positioning, text showing, type3 font, color, inline image, XObject, marked content, compatibility, page boundary)

### PD Layer (Physical Document)

- `PDValidationObject` — base protocol for PD-layer objects
- `ValidatedDocument` — top-level document validation object
- `ValidatedPage` — page-level validation
- `ValidatedResource` — resource dictionary validation
- `ValidatedContentStream` — content stream validation
- `StructureElementType` — enum with 55 cases for PDF structure element types
- `ValidatedStructTreeRoot` — structure tree root validation
- `ValidatedStructElem` — structure element validation
- `ValidatedAnnotation` — annotation validation
- `AnnotationType` — enum with 27 annotation type cases
- `ValidatedFont` — font validation types
- `FontSubtype`, `FontValidation`, `Type0Font`, `Type1Font`, `TrueTypeFont`, `CIDFont`
- `ValidatedAcroForm` — form field validation
- Color space validation: `ColorSpaceFamily`, `DeviceGray`, `DeviceRGB`, `DeviceCMYK`, `ICCBased`, `CalGray`, `CalRGB`
- External objects: `ICCProfile`, `JPEG2000`, `EmbeddedFile`, `PKCS` validation
- Additional: `OutputIntent`, `Action`, `Destination`, `OptionalContentGroup`, `Pattern`, `Shading`, `XObject`, `Outline`

### SA Layer (Semantic Accessibility)

- `SAObject` — protocol for SA-layer objects
- `SADocument` — semantic document representation
- `SAPage` — semantic page representation
- `SAStructureRoot` — structure tree root in SA layer
- `SANode` — generic node in the semantic tree
- `SAStructureElement` — structure element in SA layer
- `ContentChunkContainer` — container for content chunks
- `ContentChunkFactory` — factory for creating content chunks
- `WCAGValidationContext` — context for WCAG validation
- `SADocumentEncoder` — serialization of SA documents
- `SALayerHelpers` — utility functions for SA layer
- `SAValidationReport` — SA-specific validation report

### Features

- `FeatureExtractor` — extracts features from PDF documents
- `FeatureType` — enum of extractable feature categories
- `FeatureNode` — node in the feature tree
- `FeatureReport` — aggregated feature extraction report

### Metadata

- `MetadataFixer` — repairs and normalizes PDF metadata
- XMP schema types for metadata validation
- `InfoDictionary` — PDF info dictionary access

## Common Usage

```swift
import SwiftVerificarValidation

// Validate operator types
let op = ValidatedOperator.textShowTj
op.isTextOperator // true
op.category // .textShowing

// Check structure element types
let elemType = StructureElementType.h1
elemType.isHeading // true
elemType.headingLevel // 1
```

## Build Commands

**NEVER use `swift build` or `swift test`.** Always use `xcodebuild`.

```bash
# Build
xcodebuild build -scheme SwiftVerificarValidation -destination 'platform=macOS'

# Test
xcodebuild test -scheme SwiftVerificarValidation -destination 'platform=macOS'
```

Run these from the `/SwiftVerificar-validation/` package directory.

## Technical Details

- **Swift version**: 6.0 (strict concurrency)
- **All types are Sendable**
- **Testing framework**: Swift Testing
- **Platforms**: macOS 14.0+, iOS 17.0+
- **Stats**: 238+ public types, 3050+ tests, 19 sprints

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/intrusive-memory)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/intrusive-memory)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
