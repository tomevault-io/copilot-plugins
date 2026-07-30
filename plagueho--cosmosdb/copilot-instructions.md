## cosmosdb

> PowerShell Pester testing best practices based on Pester v5 conventions


# PowerShell Pester v5 Testing Guidelines

This guide provides PowerShell-specific instructions for creating automated tests using PowerShell Pester v5 module. Follow PowerShell cmdlet development guidelines in [powershell.instructions.md](./powershell.instructions.md) for general PowerShell scripting best practices.

## File Naming and Structure

- **File Convention:** Use `*.Tests.ps1` naming pattern
- **Placement:** Place test files next to tested code or in dedicated test directories
- **Import Pattern:** Use `BeforeAll { . $PSScriptRoot/FunctionName.ps1 }` to import tested functions. Use `$PSScriptRoot` or `$PSCommandPath` to locate scripts — do NOT use `$MyInvocation.MyCommand.Path` inside `BeforeAll` (it does not work in Pester v5).
- **No Direct Code:** Put ALL code inside `It`, `BeforeAll`, `BeforeEach`, `AfterAll`, or `AfterEach`. Code placed directly in a `Describe` or `Context` body outside these blocks runs during **Discovery**, not during test execution — its state is NOT available when tests run. Use `BeforeDiscovery` for code that must explicitly run during Discovery.

## Test Structure Hierarchy

```powershell
BeforeAll { # Import tested functions }
Describe 'FunctionName' {
    Context 'When condition' {
        BeforeAll { # Setup for context }
        It 'Should behavior' { # Individual test }
        AfterAll { # Cleanup for context }
    }
}
```

## Discovery and Run

Pester v5 runs test files in two phases: **Discovery** (collecting tests) and **Run** (executing tests).

- `Describe` and `Context` scriptblocks are invoked during **Discovery** to collect tests — all other scriptblocks (`It`, `BeforeAll`, etc.) are saved and invoked during **Run**
- Code placed directly in a `Describe` or `Context` body (outside sub-blocks) runs during Discovery — its results are NOT available during Run
- Variables set in `BeforeAll` are NOT available in `-TestCases`/`-ForEach` or `-Skip` conditions — these are evaluated during Discovery before `BeforeAll` runs
- Use `BeforeDiscovery { }` for code that must run during Discovery (e.g., building test case data from external sources)
- Use `-ForEach` on `Describe`, `Context`, or `It` to pass Discovery-time data into Run-phase blocks
- `$TestDrive` is only available during Run — it cannot be used in `-ForEach` data

```powershell
# WRONG — $items is from BeforeAll (Run phase), not available in -ForEach (Discovery phase)
Describe 'Example' {
    BeforeAll {
        $script:items = @('a', 'b')
    }
    It 'test <_>' -ForEach $script:items {  # $script:items is $null here
        $_ | Should -Not -BeNullOrEmpty
    }
}

# CORRECT — use BeforeDiscovery to populate data used during Discovery
BeforeDiscovery {
    $items = @('a', 'b')
}
Describe 'Example' {
    It 'test <_>' -ForEach $items {
        $_ | Should -Not -BeNullOrEmpty
    }
}
```

## Core Keywords

- **`Describe`**: Top-level grouping, typically named after function being tested
- **`Context`**: Sub-grouping within Describe for specific scenarios
- **`It`**: Individual test cases, use descriptive names
- **`Should`**: Assertion keyword for test validation
- **`BeforeAll/AfterAll`**: Setup/teardown once per block
- **`BeforeEach/AfterEach`**: Setup/teardown before/after each test

## Setup and Teardown

- **`BeforeAll`**: Runs once during the **Run** phase at the start of the containing block; shared with all child blocks and tests. Use for expensive operations (importing the tested function, one-time API calls).
- **`BeforeEach`**: Runs before every `It` in the current or any child block. Use for per-test prerequisites (e.g., creating a fresh file).
- **`AfterEach`**: Runs after every `It` in the current or any child block, inside a `finally` block — guaranteed to run even if the test or its setup fails. Placement within the block does not affect ordering; it always runs last. Write teardown defensively (e.g., check `Test-Path` before removing a file that may not have been created).
- **`AfterAll`**: Runs once after the containing `Describe`/`Context` block, guaranteed even on failure. Use for shared cleanup.
- **Multiple setups/teardowns**: Multiple `BeforeAll`/`BeforeEach` run in definition order; `AfterAll`/`AfterEach` run in the opposite order. There can be only ONE setup and ONE teardown of each kind per block.
- **Skipped when no tests run**: Setups and teardowns are skipped when filtering excludes all tests in the block tree.

### Variable Scoping

- Variables defined in `BeforeAll` are available (read-only) to all child blocks and tests, but CANNOT be written back — each test runs in its own scope to stay isolated. Assigning to such a variable inside `It` creates a test-local copy only.
- `BeforeEach`, `It`, and `AfterEach` share the SAME scope — variables defined in `BeforeEach` or `It` are available in `AfterEach` (e.g., to clean up a file path created in the test).

## Assertions (Should)

- **Basic Comparisons**: `-Be`, `-BeExactly`, `-Not -Be`
- **Collections**: `-Contain`, `-BeIn`, `-HaveCount`
- **Numeric**: `-BeGreaterThan`, `-BeLessThan`, `-BeGreaterOrEqual`
- **Strings**: `-Match`, `-Like`, `-BeNullOrEmpty`
- **Types**: `-BeOfType`, `-BeTrue`, `-BeFalse`
- **Files**: `-Exist`, `-FileContentMatch`
- **Exceptions**: `-Throw`, `-Not -Throw` — message matching uses `-like` wildcard semantics, NOT `.Contains()`. Always use `*` wildcards for partial message matching (e.g., `Should -Throw "*not found*"`)
- **Legacy syntax removed**: `Should Be` (without `-`) from Pester v3 is **removed**. Always use `Should -Be` with the hyphen.

## Mocking

- **`Mock CommandName { ScriptBlock }`**: Replace command behavior
- **`-ParameterFilter`**: Mock only when parameters match condition. No `param()` block needed — parameters are available directly (e.g., `-ParameterFilter { $Name -eq 'x' }`).
- **`-Verifiable`**: Mark mock as requiring verification
- **`Should -Invoke`**: Verify mock was called specific number of times
- **`Should -InvokeVerifiable`**: Verify all verifiable mocks were called
- **Placement scope**: Mocks are scoped to the **block where they are defined** — a mock defined in one `Context` is NOT available in a sibling `Context` or parent `Describe`. A mock in `BeforeAll` applies to all tests in that block and its children.
- **Counting scope**: `Should -Invoke` counts depend on placement. In `It`, `BeforeEach`, and `AfterEach` it defaults to `It` scope; in `Describe`, `Context`, `BeforeAll`, and `AfterAll` it defaults to the containing `Describe`/`Context`. Override with the `-Scope` parameter (e.g., `-Scope Describe`).
- **`$PesterBoundParameters`**: Inside a mock scriptblock, `$PSBoundParameters` is overwritten by Pester's proxy. Use `$PesterBoundParameters` (Pester 5.2.0+) to access or forward bound parameters.
- **Native commands**: Native executables expose no named parameters — match on `$args` inside the mock scriptblock and `-ParameterFilter`.
- **NEVER use** `Assert-MockCalled` (deprecated), `Assert-VerifiableMock` (deprecated), or `Assert-VerifiableMocks` (**removed**) — use `Should -Invoke` and `Should -InvokeVerifiable` instead.

```powershell
Mock Get-Service { @{ Status = 'Running' } } -ParameterFilter { $Name -eq 'TestService' }
Should -Invoke Get-Service -Exactly 1 -ParameterFilter { $Name -eq 'TestService' }
```

## Test Cases (Data-Driven Tests)

Use `-TestCases` or `-ForEach` for parameterized tests:

```powershell
It 'Should return <Expected> for <Input>' -TestCases @(
    @{ Input = 'value1'; Expected = 'result1' }
    @{ Input = 'value2'; Expected = 'result2' }
) {
    Get-Function $Input | Should -Be $Expected
}
```

## Data-Driven Tests

- **`-ForEach`**: Available on `Describe`, `Context`, and `It` for generating multiple tests from data
- **`-TestCases`**: Alias for `-ForEach` on `It` blocks (backwards compatibility)
- **Hashtable Data**: Each item defines variables available in test (e.g., `@{ Name = 'value'; Expected = 'result' }`)
- **Array Data**: Uses `$_` variable for current item
- **Templates**: Use `<variablename>` in test names for dynamic expansion
- **Discovery scoping**: `-TestCases`/`-ForEach` data and `-Skip` conditions are evaluated during **Discovery**. Variables set in `BeforeAll` are NOT available — use `BeforeDiscovery` to prepare data used in `-ForEach` or `-Skip`.
- **TestDrive**: `$TestDrive` is only available during Run and cannot be used in `-ForEach` data.

```powershell
# Hashtable approach
It 'Returns <Expected> for <Name>' -ForEach @(
    @{ Name = 'test1'; Expected = 'result1' }
    @{ Name = 'test2'; Expected = 'result2' }
) { Get-Function $Name | Should -Be $Expected }

# Array approach
It 'Contains <_>' -ForEach 'item1', 'item2' { Get-Collection | Should -Contain $_ }
```

## Tags

- **Available on**: `Describe`, `Context`, and `It` blocks
- **Filtering**: Use `-TagFilter` and `-ExcludeTagFilter` with `Invoke-Pester`
- **Wildcards**: Tags support `-like` wildcards for flexible filtering

```powershell
Describe 'Function' -Tag 'Unit' {
    It 'Should work' -Tag 'Fast', 'Stable' { }
    It 'Should be slow' -Tag 'Slow', 'Integration' { }
}

# Run only fast unit tests
Invoke-Pester -TagFilter 'Unit' -ExcludeTagFilter 'Slow'
```

## Skip

- **`-Skip`**: Available on `Describe`, `Context`, and `It` to skip tests
- **Conditional**: Use `-Skip:$condition` for dynamic skipping
- **Runtime Skip**: Use `Set-ItResult -Skipped` during test execution (setup/teardown still run). `Set-ItResult -Pending` is deprecated — do NOT use it.
- **Skip conditions**: `-Skip:$condition` is evaluated during **Discovery** — do NOT use variables initialized in `BeforeAll` as skip conditions (they will be `$null` during Discovery).

```powershell
It 'Should work on Windows' -Skip:(-not $IsWindows) { }
Context 'Integration tests' -Skip { }
```

## Error Handling

- **Continue on Failure**: Set `$PesterPreference.Should.ErrorAction = 'Continue'` inside a test file, or configure via `$config.Should.ErrorAction = 'Continue'` when calling `Invoke-Pester`, to collect multiple assertion failures per test
- **Stop on Critical**: Use `-ErrorAction Stop` for pre-conditions
- **Test Exceptions**: Use `{ Code } | Should -Throw` for exception testing

## Unit Testing within Modules

To test commands defined inside a script module, choose one of two approaches:

- **Testing public (exported) functions** — inject mocks into the module scope with `-ModuleName`:
  - Add `-ModuleName MyModule` to every `Mock` and `Should -Invoke` so calls made *inside* the module invoke the mock.
  - The test still only accesses exported members. Preferred approach — verifies functions are actually published.

```powershell
BeforeAll {
    Import-Module MyModule
}

Describe 'BuildIfChanged' {
    Context 'When there are changes' {
        BeforeAll {
            Mock -ModuleName MyModule Get-Version { 1.1 }
            Mock -ModuleName MyModule Get-NextVersion { 1.2 }
        }

        It 'Builds the next version' {
            $result = BuildIfChanged
            $result | Should -Be 1.2
        }
    }
}
```

- **Testing private (non-exported) functions** — use `InModuleScope` to run code inside the module's session state:
  - Inside `InModuleScope`, do NOT add `-ModuleName` to `Mock`/`Should -Invoke`, and you can call non-exported functions directly.
  - **Keep `InModuleScope` inside `It`** — do NOT wrap `Describe`/`Context` in it (it loads the module during Discovery, slows things down, and hides publishing/scoping issues).
  - Variables/functions created inside `InModuleScope` do NOT persist by default — use the `script:` scope modifier to reuse them across scriptblocks inside the module.

```powershell
BeforeAll {
    Import-Module MyModule
}

Describe "Module's internal Build function" {
    It 'Outputs the correct message' {
        InModuleScope MyModule {
            Mock Write-Host { }
            Build 5.0
            Should -Invoke Write-Host -ParameterFilter {
                $Object -eq 'a build was run for version: 5.0'
            }
        }
    }
}
```

- **Module type limitations**: Script, Dynamic, and Manifest modules (Manifest fully supported in Pester 5.4+) support both `Mock -ModuleName` and `InModuleScope`. **Binary modules** cannot have mocks injected and do not support `InModuleScope` — only their exported commands can be mocked for calls made from outside.

## Removed and Deprecated APIs

Never use these in Pester v5 test files:

| API | Status | Replacement |
|-----|--------|-------------|
| `Assert-VerifiableMocks` | **Removed** | `Should -InvokeVerifiable` |
| `Assert-MockCalled` | **Deprecated** | `Should -Invoke` |
| `Assert-VerifiableMock` | **Deprecated** | `Should -InvokeVerifiable` |
| `Set-ItResult -Pending` | **Deprecated** | `Set-ItResult -Skipped` |
| `Should Be` (no `-`) | **Removed** | `Should -Be` |
| `$MyInvocation.MyCommand.Path` in `BeforeAll` | **Broken** | `$PSScriptRoot` or `$PSCommandPath` |

## Best Practices

- **Descriptive Names**: Use clear test descriptions that explain behavior
- **AAA Pattern**: Arrange (setup), Act (execute), Assert (verify)
- **Isolated Tests**: Each test should be independent
- **Avoid Aliases**: Use full cmdlet names (`Where-Object` not `?`)
- **Single Responsibility**: One assertion per test when possible
- **Test File Organization**: Group related tests in Context blocks. Context blocks can be nested.
- **Module Testing**: Prefer `Mock -ModuleName` over `InModuleScope`; when `InModuleScope` is required, keep it inside `It` (see [Unit Testing within Modules](#unit-testing-within-modules)).

## Example Test Pattern

```powershell
BeforeAll {
    . $PSScriptRoot/Get-UserInfo.ps1
}

Describe 'Get-UserInfo' {
    Context 'When user exists' {
        BeforeAll {
            Mock Get-ADUser { @{ Name = 'TestUser'; Enabled = $true } }
        }

        It 'Should return user object' {
            $result = Get-UserInfo -Username 'TestUser'
            $result | Should -Not -BeNullOrEmpty
            $result.Name | Should -Be 'TestUser'
        }

        It 'Should call Get-ADUser once' {
            Get-UserInfo -Username 'TestUser'
            Should -Invoke Get-ADUser -Exactly 1
        }
    }

    Context 'When user does not exist' {
        BeforeAll {
            Mock Get-ADUser { throw "User not found" }
        }

        It 'Should throw exception' {
            { Get-UserInfo -Username 'NonExistent' } | Should -Throw "*not found*"
        }
    }
}
```

## Configuration

Configuration is defined **outside** test files when calling `Invoke-Pester` to control execution behavior.

```powershell
# Create configuration (Pester 5.2+)
$config = New-PesterConfiguration
$config.Run.Path = './Tests'
$config.Output.Verbosity = 'Detailed'
$config.TestResult.Enabled = $true
$config.TestResult.OutputFormat = 'NUnitXml'
$config.Should.ErrorAction = 'Continue'
Invoke-Pester -Configuration $config
```

**Key Sections**: Run (Path, Exit), Filter (Tag, ExcludeTag), Output (Verbosity), TestResult (Enabled, OutputFormat), CodeCoverage (Enabled, Path), Should (ErrorAction), Debug

**Removed/renamed `Invoke-Pester` parameters** — do NOT use these v4 parameters:

| Removed/renamed parameter | Replacement |
|---------------------------|-------------|
| `-Show` | `-Output` or `$config.Output.Verbosity` |
| `-Script` | `-Path` (paths only — hashtables no longer accepted) |
| `-TestName` | `-FullNameFilter` |
| `-Strict` | Removed — no replacement |
| `-PesterOption` | Removed — use `New-PesterConfiguration` |

**Valid `-Output`/`Verbosity` values**: `None`, `Normal`, `Detailed`, `Diagnostic` only. The `-Show` alias is removed.

---
> Source: [PlagueHO/CosmosDB](https://github.com/PlagueHO/CosmosDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
