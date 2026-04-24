## loghn

> │  └─ loghn                 # CLI launcher (shebang)

# Project Structure (Planned)

```
.
├─ bin/
│  └─ loghn                 # CLI launcher (shebang)
├─ src/
│  ├─ cli.ts               # CLI entry, command routing
│  ├─ server/
│  │  ├─ index.ts          # HTTP server (UI + WS)
│  │  ├─ routes.ts         # REST/WS endpoints
│  │  └─ redaction.ts      # Secret redaction rules
│  ├─ core/
│  │  ├─ bus.ts            # event bus
│  │  ├─ normalizer.ts     # normalize different log shapes
│  │  ├─ parser.ts         # JSON/text detectors
│  │  └─ query.ts          # filter/search logic
│  ├─ sources/
│  │  ├─ stdin.ts          # pipe/stdin source
│  │  ├─ file.ts           # tail file source
│  │  └─ docker.ts         # docker stream source (optional)
│  ├─ adapters/
│  │  ├─ vite.ts
│  │  ├─ next.ts
│  │  ├─ express.ts
│  │  ├─ fastapi.ts
│  │  └─ spring.ts         # Spring Boot/Servlet
│  ├─ config/
│  │  ├─ schema.ts         # zod/yup schema
│  │  └─ load.ts           # load `loghn.config.(js|ts)`
│  └─ types.ts
├─ web/                     # static assets/app for UI
├─ README.md
└─ package.json             # `name: loghn`, `bin: { loghn: "bin/loghn" }`
```

Primary files to look for in navigation:
- [src/cli.ts](mdc:src/cli.ts)
- [src/server/index.ts](mdc:src/server/index.ts)
- [src/core/normalizer.ts](mdc:src/core/normalizer.ts)
- [src/adapters/](mdc:src/adapters/)
- [src/config/schema.ts](mdc:src/config/schema.ts)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndyAnh174) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
