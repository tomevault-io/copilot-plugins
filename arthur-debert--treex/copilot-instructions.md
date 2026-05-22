## treex

> `treex` is a command line utility for displaying file trees, a modernized view of the unix tree tool.

# treex

`treex` is a command line utility for displaying file trees, a modernized view of the unix tree tool.
This phase of the project will not touch or deal with the cli app, but instead focus on the core engine and data structures.

We'll be working on the tree builder part, the code that recives the root path we want to view and set of filters and conditions and returns a tree of `treex/types.go Node` structs..

## FileMap (generated with previous version of treex)

Treex will store annotations in .info files , so always check and use `.info` when relevant.

```text
treex
├─ dist                                 All build / distribution should be done inside this dir
│  ├─ completions                       Shell completions
│  ├─ man                               Man page
│  └─ bin
│     └─ treex                          Binary treex file
├─ scripts                              Codebase utilities / dayliy functions
│  ├─ build                             Builds the binary into dist/bin
│  ├─ gen-completion                    IGNORE for now: will generate the cli completion into dis/completions
│  ├─ gen-manpage                       IGNORE for now: will generate the cli man into dis/man
│  ├─ lint                              Runs linting, called by pre-commit
│  ├─ loc-tree                          Shows loc separating app and test code
│  ├─ pre-commit                        Hook for git: lint + tests
│  ├─ release                           Uses goreleaser to generate the full dist
│  ├─ test                              Runs tests, called by pre-commit
│  ├─ coverage
│  └─ fmt
├─ docs
│  └─ dev
│     └─ architecture.md
└─ treex                                 Codebase entry point
   ├─ cmd                        .       Command line entry point
   ├─ treebuilder                       Package for generating the node tree
   │  └─ filesystem.go
   └─ types                             Package for types, only the core 3 types should go here, watch for type creep
      ├─ annotations.go
      └─ node.go
```

## Tooling

- scripts/build -> builds should
- scripts/test -> runs tests
- scripts/lint

## Logging

- Use `logging.Get()` or inject custom loggers for consistent logging across modules
- View logs: check `~/.cache/treex/treex.log` (or `$XDG_CACHE_HOME/treex/treex.log`) and use `-v` `-vv` `-vvv` flags for console verbosity (works for both program and tests)

## NO CLI

  For now there will be no shell work, we will implement the core enine at the data structure level.
  The only reson we want the build, release and other infrascture setup is to have the layout ready .

## No Backwards Compatibility

  This is unreleased code. There are no integrations, no third parties, no users. Hence it makes no sense for us to build backard compatibility, adapters, and on son. Each of these has a cost on complexity that compounds and in this codebase it has no gain to be had.

## Testing

- Unit Testing not Integration Testiong:
  - Its critical that we do unit testing. The entire code base is designed functionally.
  - For this we need to dependency inject the file systme operations so we can mock this smartly on tests.
- The Sourcd Code Testability:
  - Make sure tested code has injectable file system (aftero)
  - Whenever possible file system can be in one isolated function that's injectable, and all other logic cna be tested easily.
- Tests tend to be way more verbose than app code, hence for most modules you want various tests files thematically groupped, not 1o1 to modules.
- E2E Testing:
  - Use e2e-sandbox/run to test treex end-to-end with various filesystem structures from JSON files
File Sytem and Fixtures
  - All tests must use the shared setup / filesytem helpers, no exceptions., and no writing to disk an never ver creating files inside the codebase source files.
  - ALWAYS use afero's in-memory filesystem (afero.NewMemMapFs()) for tests, NEVER the real filesystem. This ensures tests are fast, deterministic, and can run in any environment.
  - The testutil package provides TestFS and helper methods for creating test directory structures. Use these helpers instead of creating your own.
  - use helper function for repetitve setup (like generation command line stirngs from a argument/ flags map, not strings one by one)

## Project Scope

  This is detailed in the [architecture docs](./docs/dev/architecture.txt)

## Standards

  All documentation for the project is to be written in .txt , no markdown. See the [architecture] (./docs/dev/architecture.txt) for a formatting reference

## ENV VARS

  the .envrc file has all the core env vars the project relies on (and it does.). The easiest way to ensure it's working is to have direnv (and do direnv allow for starers). If not, source .envrc will work.

  For example, do not do long relative path traversals like ../../../ , use ${PROJECT_ROOT}/scripts for example

## WORKFLOW

We leverage github heavily. You will either receive a gh issue or will help craft one for yout task.
Then you wil create a branch for it, do the work , the testing , push and when done write the pr for review.
Always reference the gh issue on commit messasges (i.e. for #<issue>)
Commit as granularly as possible, do not bundle several standalone deliverables into gian commits.

---
> Source: [arthur-debert/treex](https://github.com/arthur-debert/treex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
