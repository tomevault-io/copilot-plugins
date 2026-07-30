## dasblog-core

> - **Framework:** xUnit. Do not add MSTest or NUnit.


# Test conventions

- **Framework:** xUnit. Do not add MSTest or NUnit.
- **Naming:** `MethodUnderTest_State_ExpectedBehavior` (e.g.
  `GetEntry_MissingId_ReturnsNull`).
- **Layout:** Mirror the production namespace under
  `DasBlog.Tests/UnitTests/<Area>`. Integration tests go in
  `DasBlog.Tests/DasBlog.Test.Integration`.
- **Fixtures:** Reuse `DasBlogSettingsMock`, `TestEntry`, and helpers in
  `Common/`, `TestContent/`, and `UnitTestsConstants.cs` rather than rolling
  new ones.
- **Isolation:** Unit tests must not touch the real file system, network, or a
  developer's `Config/` folder. Use the `TestContent/` fixtures.
- **Integration tests:** May spin up the web host but must clean up any files
  they create under temp directories.
- **Run before pushing:**
  `dotnet test source/DasBlog.Tests/UnitTests/DasBlog.Tests.UnitTests.csproj`

---
> Source: [poppastring/dasblog-core](https://github.com/poppastring/dasblog-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
