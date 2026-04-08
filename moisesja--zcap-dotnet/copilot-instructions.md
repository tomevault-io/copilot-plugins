## zcap-dotnet

> This file provides instructions for AI agents and human contributors working in this codebase.

# Agent & Contributor Instructions

This file provides instructions for AI agents and human contributors working in this codebase.

## Project Overview

W3C's ZCAP-LD (Authorization Capabilities for Linked Data) specification defines an object-capability model where authority is granted by possessing a signed "capability" document, rather than by identity or ACLs. A ZCAP-LD capability is a JSON-LD object containing fields like id, invocationTarget, and a cryptographic proof. It can delegate authority by linking to a parent capability (parentCapability) and attaching restrictions called caveats. This model "shifts the burden of identification…to directly work with individuals' actual capabilities" – in other words, "if you have a valid 'capability', you have the authorization" (akin to holding a car key). Our goal is to build a .NET 10 library (for use in-process or via gRPC) that can create, sign, delegate, invoke, and verify ZCAP-LD capabilities for digital wallet agents (using Trinsic for DID/key management). Below are the key requirements and design points.

## Key Requirements and Design

- Data Model (Capabilities): Implement C# classes to represent ZCAP-LD capabilities. A root capability (initial authority) and a delegated capability (child) share common fields. Each capability JSON-LD object should include properties such as id (a URI, e.g. urn:uuid:…), optional parentCapability (URI of parent ZCAP), controller (the DID of the authority issuing it), invocationTarget (the target resource URI), allowedAction (e.g. "read", "write"), optional expires (a timestamp), optional caveat list (restrictions), and a nested proof object. For example, the spec shows a delegated capability JSON with those fields and an @context (e.g. https://w3id.org/zcap/v1). In C#, use properties and JSON serialization attributes ([JsonProperty] etc.) to match these names.

- Linked Data Proofs (Signing): Capabilities must be cryptographically signed using a linked-data proof. Implement code to generate a proof in the style of ZCAP-LD: include fields like type (signature type, e.g. "Ed25519Signature2018" or "Ed25519Signature2020"), created (timestamp), proofPurpose ("capabilityDelegation" when delegating), verificationMethod (the DID key URI), a capabilityChain array, and a signature value (e.g. JWS or base58 string). For a delegation proof, capabilityChain should list the root capability ID and intermediate ancestors (parent fully embedded as object). Use .NET crypto libraries (e.g. System.Security.Cryptography.Ed25519 or RSA) or JSON-LD libraries to canonicalize the capability JSON and produce the signature. The proof format must match the spec examples (e.g. Ed25519Signature2020 with proofValue).

- Capability Delegation (Chains): Support chaining of capabilities. When delegating, build a chain where each delegated capability includes a proof signed by its parent’s controller. The spec requires that the delegation chain is an ordered array: the first element is the root capability’s ID, intermediate ancestors by ID, and the parent capability is embedded and signed. In practice, your code should assemble this chain and include it in the proof. On verification, check each link: ensure each child’s proof is valid using the parent’s public key, and that no chain is too long (limit e.g. 10) to prevent attacks. Store or pass the full chain in invocations so the verifier need not fetch external data.

- Invocation and Verification: Implement invocation processing: when an AI agent invokes an action, it will present the capability and a proof with proofPurpose: "capabilityInvocation". The invocation JSON must include the root capability ID (capability) and requested action (capabilityAction). Verify that the proof’s signature key matches the controller of the root capability and that the requested action is among the capability’s allowedAction. Also check the invocationTarget URI matches (or is a valid prefix of) the capability’s target. According to the spec, the key used to sign must be authorized by the root zcap controller. If valid, allow the action; otherwise deny.

- Caveat Support: Implement handling of caveats (restrictions). The spec notes that each capability may add restrictions via a caveat property, and that child capabilities inherit all caveats of their parents. For example, one could add a time-based caveat (e.g. ValidUntil) or an action-limiting caveat. At minimum, design a Caveat class or interface so common types (timestamp checks, count limits, etc.) can be enforced at invocation time. When verifying a delegated capability, ensure that all caveats from the root through to the leaf are evaluated and honored. (For a minimal implementation, you can start by supporting a simple expiration or true/false caveat and expand later.)

- Digital Identity Integration: In practice, capabilities will be issued by entities (e.g. users or services) with DIDs and keypairs managed by Trinsic. Your code should plan for implementors to provide their own DID managing library to fetch a DID document, extract the public key (verificationMethod) for signature verification, or to sign data with a private key. For example, you may call Trinsic APIs to get the public key for a DID used in controller or verificationMethod. In code, this may be abstracted as functions like ResolvePublicKey(did) and SignWithPrivateKey(data, did).

- Architecture (In-Process vs gRPC): Since signing uses private keys, implementing this logic in-process (within the same application or service) is simplest. However, you may optionally expose the functionality over gRPC or HTTP for remote agents. For a library, ensure that signing and verification functions are thread-safe and do not persist private keys beyond needed scope. If exposing via gRPC, design service methods like CreateCapability(), DelegateCapability(), VerifyInvocation().

- WASM/Interop Support: (Optional) .NET 10 supports building WebAssembly via WASI. The spec use-case hints at cross-environment usage (e.g. Python or JS agents). Consider structuring code for AOT compilation: avoid heavy native dependencies, and test with .NET 10's wasi-experimental workload. This would allow consuming the library as a Wasm module in other languages. For now, focus on core functionality; WASM/Trinity integration can be added later.

## Project Structure

```
zcap-dotnet/
├── ZcapLd.sln
├── src/
│   ├── ZcapLd.Core/                    # Core library (NuGet package)
│   │   ├── Cryptography/               # ICryptoSuite, CryptoSuiteProvider, CryptoSuite,
│   │   │                               #   IDocumentCanonicalizer, JcsDocumentCanonicalizer,
│   │   │                               #   RdfcDocumentCanonicalizer, DocumentCanonicalizerProvider,
│   │   │                               #   MultibaseCodec, JsonCanonicalizer, SignatureVerifier,
│   │   │                               #   ProofSigningPayloadBuilder
│   │   ├── Models/                     # Capability, Proof, Invocation, Caveat, ValidWhileTrueCaveat,
│   │   │                               #   InvocationContext, ResolvedKey, SignatureResult,
│   │   │                               #   RevocationRecord, RevocationRequest
│   │   ├── Services/                   # ICapabilityService, ISigningService, IVerificationService,
│   │   │                               #   ICaveatProcessor, IDidResolver, IDidSigner, IRevocationService,
│   │   │                               #   IRevocationStore, INonceStore, IValidWhileTrueHandler + implementations
│   │   └── Exceptions/                 # ZcapLdExceptions
│   └── ZcapLd.AspNetCore/             # ASP.NET adapter (NuGet package)
│       ├── DependencyInjection/        # AddZcapServices(), AddZcapDidSigner<T>(), AddZcapRevocationSupport(),
│       │                               #   AddZcapReplayProtection(), AddZcapCryptoSuite<T>(),
│       │                               #   AddZcapDidResolver<T>(), AddZcapValidWhileTrueSupport()
│       ├── Endpoints/                  # MapZcapRevocationEndpoints()
│       ├── Services/                   # HttpValidWhileTrueHandler
│       └── Contracts/                  # RevokeCapabilityHttpRequest, RevocationStatusHttpResponse
├── tests/ZcapLd.Core.Tests/           # xUnit + FluentAssertions
│   ├── Cryptography/                   # Ed25519, P256, JsonCanonicalizer, MultibaseCodec, etc.
│   ├── Services/                       # CapabilityService, VerificationService, Revocation, Replay, etc.
│   ├── Models/                         # Capability serialization tests
│   ├── Integration/                    # End-to-end workflow tests
│   ├── Compliance/                     # Normative unit + integration spec compliance tests
│   └── Helpers/                        # InMemoryDidProvider (test-only IDidSigner + IDidResolver)
├── examples/
│   ├── ZcapLd.Examples/               # Console examples (8 scenarios)
│   └── ZcapLd.RevocationEndpointsDemo/ # ASP.NET revocation demo (SQLite + ValidWhileTrue)
├── docs/                               # Implementation, security, revocation, release docs
├── tasks/                              # Historical evaluations, task tracking
├── .github/workflows/                  # CI/CD pipelines
├── ARCHITECTURE.md                     # Architecture and service boundaries
├── CONTRIBUTING.md                     # Contributor guide
└── README.md                          # Project overview and quick start
```

## Development Commands

```bash
dotnet restore                                                         # Restore NuGet packages
dotnet build ZcapLd.sln                                                # Build entire solution
dotnet test ZcapLd.sln                                                 # Run all tests
dotnet pack src/ZcapLd.Core/ZcapLd.Core.csproj -c Release             # Pack core library
dotnet pack src/ZcapLd.AspNetCore/ZcapLd.AspNetCore.csproj -c Release  # Pack ASP.NET adapter
dotnet run --project examples/ZcapLd.Examples                          # Run console examples
```

## Architecture Notes

- **Target framework**: .NET 10 (`net10.0`)
- **Specification**: W3C ZCAP-LD v0.3 (CG-DRAFT), 95%+ compliance (283 tests)
- **Crypto suites**: Ed25519 (NSec.Cryptography) and P-256 (System.Security.Cryptography), extensible via `ICryptoSuite` / `ICryptoSuiteProvider`
- **Key management**: No default `IDidSigner` — consumers must provide their own (HSM/KMS/Key Vault)
- **DID resolution**: `DidKeyResolver` (wraps NetDid's `DidKeyMethod`), `CompositeDidResolver` (multi-method routing); returns `ResolvedKey(byte[] PublicKeyBytes, string KeyType)`
- **DID packages**: [NetDid.Core](https://www.nuget.org/packages/NetDid.Core) + [NetDid.Method.Key](https://www.nuget.org/packages/NetDid.Method.Key) for W3C-compliant did:key creation/resolution; multibase via [NetCid](https://www.nuget.org/packages/NetCid)
- **Signing**: `SigningService` delegates to `IDidSigner`; resolves verification methods via `IDidResolver` and context URLs via `ICryptoSuiteProvider`
- **Verification**: `VerificationService` dispatches to correct `ICryptoSuite` by proof type; enforces chain validity, attenuation, caveats, revocation, and replay protection
- **Revocation**: `IRevocationService` / `IRevocationStore` abstractions; `InMemoryRevocationStore` for dev; ASP.NET endpoints via `ZcapLd.AspNetCore`
- **Replay protection**: `INonceStore` interface; `InMemoryNonceStore` (default), `NullNonceStore` (opt-out)
- **Canonicalization**: `IDocumentCanonicalizer` interface with `JcsDocumentCanonicalizer` (RFC 8785) and `RdfcDocumentCanonicalizer` (W3C RDFC-1.0 via dotNetRdf); suite-specific via `ICryptoSuite.CanonicalizationMethod`
- **ValidWhileTrue**: `ValidWhileTrueCaveat` model + `IValidWhileTrueHandler` interface in Core; `HttpValidWhileTrueHandler` in AspNetCore; `AddZcapValidWhileTrueSupport()` for DI; fail-closed when no handler configured
- **ASP.NET integration**: `AddZcapServices()` registers all core services; `AddZcapDidSigner<T>()` for signer; `AddZcapRevocationSupport()` and `MapZcapRevocationEndpoints()` for revocation; `AddZcapValidWhileTrueSupport()` for remote revocation checking; `AddZcapRdfcCanonicalization()` to enable RDFC-1.0

## Workflow Orchestration

### 1. Plan Mode Fault

- Enter plan mode for ANY non-trivial task defined as a task that takes 3 steps or more or that requires architectural decisions.
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy

- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop

- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this,"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing

- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

# Task Management

1. **Plan First**: Write plan to `tasks/todo{timestamp}.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to 'tasks/todo.m,
6. **Capture Lessons**: Update 'tasks/lessons.md' after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Staff Engineer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/moisesja)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/moisesja)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
