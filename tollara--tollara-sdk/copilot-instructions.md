## project-guidlines

> Please keep these guidlines in mind when operating on the SDKs.


Please keep these guidlines in mind when operating on the SDKs.
- This is an SDK project for Tollara; maintainer context may exist locally under <project-root>/docs/ (gitignored) and <project-root>/docs-sdk/
- SDKs cover these languages: java, python, dotnet, js, php, go, ruby and rust
- SDK locations are as follows:
<project-root>/sdk-java
<project-root>/sdk-python
<project-root>/sdk-dotnet
<project-root>/sdk-js
<project-root>/sdk-php
<project-root>/sdk-go
<project-root>/sdk-ruby
<project-root>/sdk-rust
- The Tollara API spec is defined here in <project-root>/docs-sdk/MAIN-SDK-API-SPEC.md
- SDKs should adhere to the spec and should be consistent with each other (i.e. same behaviour and consistent names in each language sdk).
- SDKs are published to their respective public package locations (e.g. Java to MavenCentral, DotNet to NuGet, js to NPM etc )
- Each sdk folder has it's own sdk readme.md which is public facing, do not put any implementation details in this document. 
- Do not mention any internal paths in the sdk readme.md ('/api/v1' etc) - the SDK client is hardcoded to Tollara's external URL and does not need configuration and our users just need to know about the main functions not how it works under the hood. 
- Make sure the sdk readme.md is as easy to understand as possible.
- Do not refer to the API spec or the HMAC spec or any other private documents in the sdk readme.md documents.
- Do not refer to low level clients in the sdk readme.md (e.g. UsageClient, GatewayClient etc), every user action should be done via the unified TollaraClient.

---
> Source: [tollara/tollara-sdk](https://github.com/tollara/tollara-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
