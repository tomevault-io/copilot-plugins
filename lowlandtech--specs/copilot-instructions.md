## specs

> A GitHub issue is only considered **complete** when ALL of the following conditions are met:

# GitHub Issue Completion Requirements

## Definition of Done
A GitHub issue is only considered **complete** when ALL of the following conditions are met:

1. **Feature Files Exist**: Each User Story must have a `.feature` file in the `USXX.{StoryName}` folder that:
   - Uses proper Gherkin syntax with `@USXX` tags
   - Contains `@SCXX` tags for each scenario
   - Contains `#UACXX` tags before each `Then` and `And` assertion
   - References the SPEC ID in the test file traits

2. **Tests Reference SPEC ID**: Scenario test files (`SCXX.{ScenarioName}.cs`) must exist with:
   - `[Trait(Spec.SPEC, "XXXX")]` - Specification ID (without "SPEC" prefix)
   - `[Trait(Spec.US, "XX")]` - User Story ID
   - `[Trait(Spec.SC, "XX")]` - Scenario ID
   - `[Trait(Spec.UAC, "XX")]` - User Acceptance Criterion ID on each Fact/Theory

3. **Complete Coverage**: Every UAC in the `.feature` file must have a corresponding test method:
   - Each `#UACXX` in the feature file = one `[Fact]` or `[Theory]` method
   - UAC numbers in tests must match feature file exactly
   - All scenarios in the feature file must be implemented (no partial implementation)
   - DisplayName should match the Gherkin step: `[Fact(DisplayName = "Then the sidebar should be collapsed")]`

4. **All Tests Pass**: All tests for the SPEC ID must pass:

## GitHub Issue Spec ID Convention

When creating GitHub issues, use the following spec ID format:

### 1. Epic Level Issues
**Format**: `SPECXXXX`
- Used for epics covering multiple user stories
- Example: Issue #2 "SPEC0850 - Package Publishing Epic"

### 2. User Story Level Issues
**Format**: `SPECXXXX.USXX`
- Used for individual user stories with multiple scenarios
- Example: Issue #3 "SPEC0850.US01 - Upload New Packages"

### 3. Scenario Level Issues (Optional)
**Format**: `SPECXXXX.USXX.SCXX`
- Used for individual scenarios with many UACs
- Example: Issue #4 "SPEC0850.US01.SC01 - Push Valid Package"

### 4. Acceptance Criteria Level Issues (Optional)
**Format**: `SPECXXXX.USXX.SCXX.UACXX`
- Used for detailed acceptance criteria when granular tracking needed
- Example: Issue #5 "SPEC0850.US01.SC01.UAC01 - Upload Should Return 201 Created"

### Key Points
- Each issue references one level of specification
- Clear hierarchy: Epic > User Story > Scenario > UAC
- Easy traceability between GitHub issues, feature files, and tests
- Format matches folder structure exactly (dots, not underscores)
   - Run: `dotnet test --filter "SPEC=XXXX"`
   - No failing tests
   - No skipped tests (unless explicitly approved)

## Example Structure

```
src/test/
SPEC0850.PackagePublish/
  ├── US01.UploadNewPackages/             # User Story subfolder
  │   ├── US01.UploadNewPackages.feature  # Feature file (all scenarios)
  │   ├── SC01.PushValidPackage.cs        # Test file for scenario 1
  │   ├── SC02.InvalidPackageFormat.cs      # Test file for scenario 2
  │   ├── SC03.PackageAlreadyExists.cs       # Test file for scenario 3
  │   └── ...
  ├── US02.DeletePackages/
  │   ├── US02.DeletePackages.feature
  │   ├── SC01.DeleteWithUnlist.cs
  │   └── ...

SPEC0900.PackageDiscovery/
  ├── US01.VulnerabilityScanning/
  │   ├── US01.VulnerabilityScanning.feature
  │   ├── SC01.DisplayVulnerabilityScore.cs
  │   └── ...
  ├── US02.PackageGovernance/
  │   ├── US02.PackageGovernance.feature
  │   ├── SC01.MarkPackageAsDeprecated.cs
  │   └── ...
```

**Key Points:**
- Each User Story = ONE subfolder at SPEC level (`USXX.Name/`)
- Each subfolder contains:
  - ONE `.feature` file with all scenarios for that user story
  - ONE `.cs` file per scenario (`SC01.SenarioName.cs`)
- Test files use DOT notation, not underscore
- All scenarios for a user story are in ONE `.feature` file
src/test/
??? SPEC2050.MainLayout/
    ??? US01.SidebarCollapse/
        ??? US01.SidebarCollapse.feature     # Gherkin specification
        ??? SC01.ToggleFromExpanded.cs       # Scenario test
        ??? SC02.ToggleFromCollapsed.cs
```

### Feature File Example

```gherkin
@US01
Feature: US01 - Sidebar Collapse
    Given sidebar is expanded
    When I dispatch a ToggleSidebar action
    #UAC01
    Then the sidebar should be collapsed
    #UAC02
    And the width should remain unchanged
```

Note: Each User Story has ONE `.feature` file containing ALL scenarios for that story, located in `USXX.Name/` folder.

### Test File Example

```csharp
[Trait(Spec.SPEC, "0850")]
[Trait(Spec.US, "01")]
[Trait(Spec.SC, "01")]
[Trait(Spec.UAC, "01")]
public class SC01.PushValidPackage : Scenario
{
    protected override void Given()
    {
        Store.State.IsSidebarCollapsed.ShouldBeFalse();
    }

    protected override async Task WhenAsync()
    {
        await Store.DispatchAsync(new PushPackageAction());
    }

    [Fact(DisplayName = "Then the upload should return status 201 Created")]
    [Trait(Spec.UAC, "01")]
    public async Task UAC01_UploadShouldReturn201Created()
    {
        await ArrangeAndActAsync();
        Store.State.UploadStatus.ShouldBe(HttpStatusCode.Created);
    }

    [Fact(DisplayName = "And the package should be stored in the database")]
    [Trait(Spec.UAC, "02")]
    public async Task UAC02_PackageShouldBeStoredInDatabase()
    {
        await ArrangeAndActAsync();
        Store.State.Package.ShouldNotBeNull();
    }
}
```

Note: Test files use DOT notation (`SC01.PushValidPackage.cs`) and are in the same folder as the `.feature` file (`US01.UploadNewPackages/`).

## Enforcement

- Pull requests must not be approved until all conditions are met
- Verify test coverage: `dotnet test --filter "SPEC=0850"` (for SPEC0850-SPEC0950 range)
- Code reviews must confirm:
   - All scenarios from feature file are implemented
   - All UAC tags have corresponding test methods
   - UAC numbering matches between .feature and .cs files
   - All tests pass
- Folder structure verification:
   - Each User Story = ONE subfolder at SPEC level (`USXX.Name/`)
   - Each subfolder contains ONE `.feature` file + scenario test `.cs` files
   - Test files use DOT notation (e.g., `SC01.PushValidPackage.cs`) not underscore

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lowlandtech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
