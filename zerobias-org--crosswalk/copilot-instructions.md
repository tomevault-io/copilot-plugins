## crosswalk

> This file provides guidance to Claude Code (claude.ai/code) when working with crosswalk content in this repository.

# CLAUDE.md - Community Crosswalk Repository

This file provides guidance to Claude Code (claude.ai/code) when working with crosswalk content in this repository.

## Common Development Commands

### Setup and Installation
- **Initial setup**: `npm install` (run in root directory)
- **Full reset**: `npm run reset` (clean, install)

### Validation and Testing
- **Validate all crosswalks**: `npm run validate`
- **Validate single crosswalk**: `npm run validate` (in individual package directory)
- **Clean Nx cache**: `npm run clean`
- **Full clean**: `npm run clean:full` (reset Nx + remove node_modules)

### Lerna Operations
- **Dry run version bump**: `npm run lerna:dry-run`
- **Version packages**: `npm run lerna:version`

### Individual Package Commands
When working in a specific crosswalk package (e.g., `package/nist/800-218/v1_1_cncf_sscp_v1/`):
- **Validate crosswalk**: `npm run validate`
- **Correct dependencies**: `npm run correct:deps`

## Repository Architecture

### Monorepo Structure
This is a Lerna-managed monorepo with npm workspaces containing crosswalk packages. Key directories:

- **`package/`**: Contains all crosswalk packages organized by vendor/suite/version
  - Structure: `package/{vendor}/{suite}/{version}/`
  - Example: `package/nist/800-218/v1_1_cncf_sscp_v1/`, `package/opencre/opencre/v1_owasp_samm_v1_0/`
- **`scripts/`**: Validation and utility scripts
- **`templates/`**: Template files for creating new crosswalks
- **`bundle/`**: Bundled package artifacts
- **`examples/`**: Example crosswalk package

### Crosswalk Package Structure
Each crosswalk package follows this structure:
- **`index.yml`**: Main crosswalk definition (metadata, source/target frameworks)
- **`elements.yml`**: Crosswalk mapping elements
- **`versions/`**: Version-specific mapping data
- **`package.json`**: NPM package configuration with `zerobias` metadata key
- **`.npmrc`**: NPM registry configuration

### Technology Stack
- **Lerna**: Monorepo management and versioning
- **Nx**: Build system and caching
- **TypeScript**: Validation scripts (run via `tsx`)
- **YAML**: Crosswalk definition format

### Package Naming Conventions
- **Package name**: `@zerobias-org/crosswalk-{vendor}-{suite}-{version}`
- **Metadata key**: `zerobias` (not `auditmation`)
- **Metadata package value**: `{vendor}.{suite}.{version}.crosswalk` (no prefix)
- **Dataloader version**: `"1.0.0"`
- **Import artifact**: `"crosswalk"`
- **Runner**: `tsx` (not `ts-node`)

### Package.json Template
```json
{
  "name": "@zerobias-org/crosswalk-{vendor}-{suite}-{version}",
  "version": "1.0.0",
  "type": "module",
  "zerobias": {
    "dataloader-version": "1.0.0",
    "import-artifact": "crosswalk",
    "package": "{vendor}.{suite}.{version}.crosswalk"
  },
  "scripts": {
    "correct:deps": "tsx ../../../../scripts/correctDeps.ts",
    "validate": "tsx ../../../../scripts/validate.ts"
  },
  "dependencies": {
    "@zerobias-org/suite-{vendor}-{suite}": "latest"
  }
}
```

### Registry Configuration (.npmrc)
Packages use `@zerobias-org` scoped registry. Packages that depend on `@auditlogic` packages (frameworks/benchmarks not yet migrated) also include:
```
@auditlogic:registry=https://npm.pkg.github.com/
//npm.pkg.github.com/:_authToken=${NPM_TOKEN}
```

### Dependency Notes
Some framework and benchmark dependencies only exist on the `@auditlogic` registry (not yet published to `@zerobias-org`):
- `@auditlogic/framework-nist-800218-v1.1`
- `@auditlogic/framework-owasp-samm-v1.0`
- `@auditlogic/framework-csa-ccm-v4.0.12`
- `@auditlogic/framework-owasp-asvs-v4.0.3`
- `@auditlogic/framework-nist-80053-rev5`
- `@auditlogic/benchmark-owasp-wstg-v5`

Suite packages exist on `@zerobias-org`:
- `@zerobias-org/suite-nist-800-218`
- `@zerobias-org/suite-opencre-opencre`
- `@zerobias-org/suite-nist-ir8397`

## File Format Reference

**Source of Truth:** `../../com/platform/dataloader/src/processors/crosswalk/`

**Key Files:**
- `CrosswalkArtifactLoader.ts` - Main processor
- `CrosswalkFileHandler.ts` - File processing

**Expected Structure:**
- `index.yml` - Crosswalk metadata (name, description, source/target frameworks, status)
- `elements.yml` - Mapping elements
- `versions/` - Version-specific data
- `package.json` - Must include `zerobias.import-artifact: "crosswalk"`

**Valid index.yml statuses:** `active`, `verified`, `inactive`, `deprecated`

## Crosswalk Concept

### What is a Crosswalk?

A crosswalk maps requirements between two compliance frameworks:

**Example: SOC 2 -> ISO 27001**
```yaml
# mappings.yml
mappings:
  - source:
      framework: aicpa.soc2.2022
      requirement: CC6.1  # Logical and Physical Access Controls
    target:
      framework: iso.27001.2022
      requirement: A.9.1.1  # Access Control Policy
    relationship: equals
```

**Valid Relationship Types (from platform schema `crosswalkElementRelationshipType`):**
- `equals` - Requirements are essentially the same
- `subset_of` - Source is a subset of target
- `superset_of` - Source encompasses target
- `intersects` - Requirements overlap but are not equivalent

**IMPORTANT:** Only these 4 values are accepted by the dataloader. Do NOT use `related`, `equivalent`, `complementary`, or any other values.

## Integration with Platform

### Dataloader Integration
**Handler Location:** `../../com/platform/dataloader/src/processors/crosswalk/`
**Database Table:** `catalog.crosswalk`

### Usage in Platform
- **Audit Planning:** Show which SOC 2 controls also satisfy ISO 27001
- **Gap Analysis:** Identify requirements not covered by current controls
- **Multi-Framework Compliance:** Leverage existing controls across frameworks
- **Evidence Reuse:** Use same evidence for multiple framework requirements

## ZeroBias Task Integration

For creating crosswalks from ZeroBias tasks, use the skill:

```
/create-crosswalk [task-id]
```

See **[.claude/skills/create-crosswalk.md](.claude/skills/create-crosswalk.md)** for the complete workflow.

### Quick Reference

**Orchestration Documentation:**
- [Meta-repo: DEPENDENCY_CHAIN.md](../../docs/orchestration/DEPENDENCY_CHAIN.md) - **STRICT dependency rules**
- [Meta-repo: TASK_MANAGEMENT.md](../../docs/orchestration/TASK_MANAGEMENT.md) - Task API patterns
- [Meta-repo: API_REFERENCE.md](../../docs/orchestration/API_REFERENCE.md) - Quick API reference

**Dependency Chain:**
```
vendor → suite → framework/standard/benchmark → crosswalk
```

**CRITICAL:** Crosswalks require BOTH source and target frameworks. Check/create the full chain first.

### Key APIs

```javascript
// Check if source framework exists (REQUIRED before crosswalk)
zerobias_execute("portal.Framework.search", { searchFrameworkBody: { search: "source framework" }})

// Check if target framework exists (REQUIRED before crosswalk)
zerobias_execute("portal.Framework.search", { searchFrameworkBody: { search: "target framework" }})

// Check if crosswalk already exists
zerobias_execute("portal.Crosswalk.search", { searchCrosswalkBody: { search: "crosswalk" }})

// Get your party ID for assignment
zerobias_execute("platform.Party.getMyParty", {})

// Transition task to in_progress (use transitionId, NOT status)
zerobias_execute("platform.Task.update", {
  id: taskId,
  updateTask: {
    assigned: partyId,
    transitionId: "7f140bbe-4c10-54ac-922c-460c66392fad"
  }
})

// Link tasks together
zerobias_execute("platform.Resource.linkResources", {
  fromResource: sourceTaskId,
  toResource: targetTaskId,  // Note: toResource, NOT toResourceId
  linkType: "b8bd95d0-b33c-11f0-8af3-dfaccf31600e"  // relates_to
})
```

### Workflow Transitions

| Transition | Target Status | ID |
|------------|---------------|-----|
| Start | in_progress | `7f140bbe-4c10-54ac-922c-460c66392fad` |
| Peer Review | awaiting_approval | `f017a447-0994-594d-9417-39cbc9a4de88` |
| Accept | released | `1d2e9381-f609-5e26-8bc6-7bbb65a9048d` |

**Note:** Always get actual IDs from `task.nextTransitions`.

---

## Related Documentation

- **[Root CLAUDE.md](../../CLAUDE.md)** - Meta-repo guidance
- **[ContentArtifacts.md](../../ContentArtifacts.md)** - Content catalog system
- **[auditlogic/crosswalk/CLAUDE.md](../../auditlogic/crosswalk/CLAUDE.md)** - Proprietary crosswalks (same pattern)
- **[com/platform/dataloader/CLAUDE.md](../../com/platform/dataloader/CLAUDE.md)** - Dataloader processor
- **[zerobias-org/framework/CLAUDE.md](../framework/CLAUDE.md)** - Community frameworks

## Important Notes

### Community vs Proprietary

**This Repository (zerobias-org/crosswalk):**
- Open-source, community-contributed crosswalks
- Public GitHub repository
- MIT/Apache license
- Community validation and updates

**Proprietary Repository (auditlogic/crosswalk):**
- Closed-source, professionally validated crosswalks
- Private GitHub repository
- Commercial license
- Expert review and certification

Both follow identical structure and use same dataloader processor.

### Commit and Versioning
- Follow Conventional Commits specification
- Commit messages format: `<type>(<scope>): <subject>`
- Types: feat, fix, docs, style, refactor, perf, test, chore
- Lerna automatically handles versioning and changelog generation
- PRs must target the `dev` branch (not `main`)

### Mapping Accuracy

Crosswalk mappings should be:
- Validated by compliance experts
- Reviewed against official framework documentation
- Updated when frameworks change
- Tested with real-world use cases
- Peer-reviewed by community

---

**Last Updated:** 2026-02-11
**Maintainers:** ZeroBias Community

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zerobias-org)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zerobias-org)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
