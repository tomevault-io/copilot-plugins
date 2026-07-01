## cdk-dia

> **cdk-dia** is an automated diagram generator for AWS CDK applications. It reads a synthesized CDK cloud assembly (`tree.json`), builds an internal component tree representing AWS resources and their relationships, then renders visual diagrams (PNG via Graphviz or interactive HTML via Cytoscape).

# AGENTS.md — Guide for AI Agents Working on cdk-dia

## Project Overview

**cdk-dia** is an automated diagram generator for AWS CDK applications. It reads a synthesized CDK cloud assembly (`tree.json`), builds an internal component tree representing AWS resources and their relationships, then renders visual diagrams (PNG via Graphviz or interactive HTML via Cytoscape).

## Architecture

### Pipeline (5 stages)

```
tree.json  →  CDK Tree  →  Component Tree  →  Post-Processing  →  Rendered Diagram
```

1. **Parse** — `src/cdk/tree-json-loader.ts` reads `cdk.out/tree.json` into a `Tree`/`Node` model (`src/cdk/cdk-tree.ts`).
2. **Generate** — `src/diagram/aws/aws-diagram-generator.ts` traverses the CDK tree, creates `DiagramComponent` objects for CloudFormation resources, and builds a hierarchical component tree.
3. **Resolve Edges** — `src/diagram/aws/aws-edge-resolver.ts` scrapes CloudFormation properties for `Ref`, `Fn::GetAtt`, and `Fn::ImportValue` to create edges between components.
4. **Post-Process** — The generator applies filtering (include/exclude stacks, ignore), collapsing (CDK constructs, double clusters), asset removal, cross-stack edge simplification, and self-link removal.
5. **Render** — Either `src/render/graphviz/` (PNG via dot) or `src/render/cytoscape/` (interactive HTML).

### Key Files

| File | Purpose |
|------|---------|
| `bin/cli.ts` | CLI entry point (yargs) |
| `src/cdk-dia.ts` | Main orchestrator — ties parsing, generation, rendering |
| `src/cdk/cdk-tree.ts` | CDK tree data model (`Tree`, `Node`, `ConstructInfoFqn`) |
| `src/cdk/tree-json-loader.ts` | Loads and deserializes `tree.json` |
| `src/cdk/construct-decorator.ts` | `CdkDia.decorate()`, `CdkDiaDecorator`, `@DiagramOptions` |
| `src/diagram/aws/aws-diagram-generator.ts` | Core: tree traversal, component creation, all post-processing steps |
| `src/diagram/aws/aws-edge-resolver.ts` | Edge detection from CloudFormation intrinsic functions |
| `src/diagram/aws/aws-icon-supplier.ts` | Maps `AWS::*` resource types to icon paths |
| `src/diagram/aws/awsResouceIconMatches.json` | 643 resource type → icon mappings |
| `src/diagram/component/component.ts` | Abstract `Component` base class, `ComponentTags` enum |
| `src/diagram/component/customizable-attribute.ts` | `CollapssingCustomizer`, `IgnoreCustomizer` |
| `src/diagram/component/diagram-component.ts` | Concrete `DiagramComponent` |
| `src/diagram/component/root-component.ts` | Root of the component tree |
| `src/diagram/component-links.ts` | Bidirectional edge management |
| `src/render/graphviz/GraphvizGenerator.ts` | Converts diagram → ts-graphviz DOT model |
| `src/render/cytoscape/cytoscape-generator.ts` | Converts diagram → Cytoscape elements/styles |

### Exports

- `src/cdk/index.ts` — re-exports all CDK modules
- `src/diagram/index.ts` — re-exports all diagram modules

## Testing

### ALWAYS write and run tests

When making changes to this project, **you must write tests and verify they pass**. The existing test suite has strong snapshot coverage and graph integrity checks that catch regressions.

### How to run tests

```bash
# Compile TypeScript first (required — Jest runs against compiled JS)
npx tsc

# Run all tests
npm run test

# Run a specific test file
npx jest src/diagram/tests/generator.test.ts

# Update snapshots after intentional changes
npx jest --updateSnapshot
```

### Test framework

- **Jest** with **ts-jest** for TypeScript
- **jest-specific-snapshot** for named snapshot files (stored in `snapshots/`)
- Config: `jest.config.js` — test pattern: `**/src/**/tests/**/*.test.ts`
- Setup: `testSetup.js` — creates/cleans `test-generated/` directory before each run

### Test files and what they cover

| Test File | What It Tests |
|-----------|---------------|
| `src/cdk/tests/cdk-tree.test.ts` | CDK tree parsing from JSON fixtures; snapshot of parsed tree structure |
| `src/diagram/tests/generator.test.ts` | Diagram generation (snapshot), stack include/exclude filtering, ignore feature, graph integrity (all links exist in tree, all sub-components in tree, all links bidirectional) |
| `src/diagram/tests/aws-edge-resolver.test.ts` | Edge target scraping from CloudFormation props (`Fn::GetAtt`, `Ref`, `Fn::ImportValue`); uses custom Jest matcher `arrayToContainEdgeTarget` |
| `src/diagram/tests/stack-exports-container.test.ts` | Stack export parsing |
| `src/render/graphviz/tests/generator.test.ts` | Graphviz DOT output (snapshot); writes `.dot` files to `test-generated/` |
| `src/render/cytoscape/tests/generator.test.ts` | Cytoscape HTML output structure (index.html, icons/, js/, cy-elements.json, cy-styles.json) |

### Test patterns used in this project

**1. Parametrized snapshot tests over all fixtures:**
Most test suites iterate over `testCases` from `src/test-fixtures/testCases.ts` and compare output against named snapshots. When you add a new fixture, you get coverage across CDK parsing, diagram generation, Graphviz DOT, and Cytoscape rendering automatically.

**2. Graph integrity assertions:**
The generator tests include structural validations that run for every fixture:
- All linked components exist in the tree
- All sub-components exist in the tree
- All links are bidirectional

**3. Dedicated behavioral tests:**
For specific features (stack filtering, ignore), there are focused `describe` blocks with explicit assertions rather than just snapshots.

### Test fixtures

Located in `src/test-fixtures/`. Each is a `tree.json` representing a synthesized CDK app:

| Fixture | Scenario |
|---------|----------|
| `function-and-queue.json` | Simple Lambda + SQS |
| `function-and-queue-with-ignored-resource.json` | Lambda + SQS with ignored queue |
| `multiple-similar-stacks.json` | Two similar microservice stacks |
| `multiple-similar-stacks-with-usage-of-collapsing-customizer.json` | Stacks with `CDK-DIA_CollapssingCustomizer` attributes |
| `2_queues-are-subscribed-to-a-topic.json` | SNS → SQS |
| `the-eventbridge-etl.json` | EventBridge ETL workflow |
| `fargate-service-with-vpc.json` | ECS Fargate + VPC |
| `ec-loadbalancer-rds.json` | ALB + RDS |
| `cloudstructs-static-website.json` | Static website hosting |
| `cdk_pipeline_with_stages_stacks_and_construct_infos.json` | CDK Pipelines with nested stages |
| `custom-resource.json` | CloudFormation custom resources |
| `two-stack-with-a-relationship.json` | Cross-stack references |

Test cases in `testCases.ts` pair each fixture with collapsed/non-collapsed variants.

### How to add a test fixture

1. Create a JSON file in `src/test-fixtures/` following the `tree.json` format (copy an existing fixture as a template).
2. Add entries to `src/test-fixtures/testCases.ts` (typically a collapsed and non-collapsed variant).
3. Run `npx tsc && npx jest` — new snapshots will be written automatically on first run.
4. Verify the snapshots make sense, then commit them.

### Snapshots

Stored in `snapshots/` at the project root. Naming convention:
- `cdk-tree-parsing-as-expected-{fixture}-{id}.snapshot`
- `diagram-JSON-as-expected-{fixture}-{id}.snapshot`
- `diagram-converted-to-DOT-file-as-expected-{fixture}-{id}.snapshot`

If your change intentionally alters diagram output, update snapshots with `npx jest --updateSnapshot` and review the diffs carefully.

### Customization System

User-facing customization works via CDK tree attributes prefixed with `CDK-DIA_`:

- **Collapsing**: `CDK-DIA_CollapssingCustomizer` with values `FORCE_COLLAPSE`, `FORCE_NON_COLLAPSE`, `FORCE_NON_COLLAPSE_RECURSIVE`
- **Ignoring**: `CDK-DIA_IgnoreCustomizer` with value `"true"`

These are applied in `AwsDiagramGenerator.applyAttributeSetCustomizers()` which reads attributes from CDK nodes and dispatches to the corresponding `CustomizableAttribute` subclass. To add a new customizer, follow the pattern: create a class extending `CustomizableAttribute`, add a `ComponentTags` entry, wire it in the switch statement in `applyAttributeSetCustomizers()`, and expose it via `CdkDiaDecorator`.

---
> Source: [pistazie/cdk-dia](https://github.com/pistazie/cdk-dia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
