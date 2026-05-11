## esupgrade

> The package includes transformers for upgrading JavaScript syntax.

# esupgrade

The package includes transformers for upgrading JavaScript syntax.

## Categories

- [Widely Available](./src/widelyAvailable/): Transformers for features available in all major browsers for at least 30 months.
- [Newly Available](./src/newlyAvailable/): Transformers for features available in all major browsers for 0-30 months.

For a full list of transformations, see [README.md](./README.md).

## Package Structure

- [bin/](./bin/): Command-line interface script.
- [src/](./src/): Source code and transformers.
- [tests/](./tests/): Test suite.

For contributing guidelines, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## Instructions

Use EOF syntax to run node scripts directly from the command line. For example:

```bash
node --input-type=module <<'EOF'
import { transform } from './src/index.js';

const sample = "const v = $(input).val();";
const res = transform(sample);
console.log('modified:', res.modified);
console.log('code:\n' + res.code);
EOF
```

---
> Source: [codingjoe/esupgrade](https://github.com/codingjoe/esupgrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
