## aroosi

> When installing new npm packages in this project, **always use the `--legacy-peer-deps` flag**:

# NPM Install Rule: Use --legacy-peer-deps

When installing new npm packages in this project, **always use the `--legacy-peer-deps` flag**:

```
npm install <package-name> --legacy-peer-deps
```

This avoids dependency resolution errors and ensures compatibility with the current lockfile and package versions.

If you forget this flag, you may encounter errors or failed installs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karthiknish)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/karthiknish)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
