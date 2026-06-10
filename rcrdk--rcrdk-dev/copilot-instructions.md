## http-layer

> server-only import in http/; try-catch and Sentry reporting


# HTTP Layer

### Server-Only Import

- All files in the `http/` directory must import `'server-only'` at the top of the file.
- This ensures the code is not included in the client bundle.

- ✅ Good:

  ```typescript
  import 'server-only'

  export async function callRocketAPI() {}
  ```

### Error Handling

- All HTTP functions should use try-catch blocks and report errors to Sentry with proper context.

- ✅ Good:
  ```typescript
  export async function getDevelopmentDocument({ id, src }: { id: string; src: string }) {
    try {
      // ... implementation
      return blob
    } catch (error) {
      reportToSentry(error, {
        tags: { module: 'http', method: 'getDevelopmentDocument' },
        fingerprint: ['http-error', 'getDevelopmentDocument', id, src],
        extra: { documentId: id, documentSrc: src },
      })
      throw error
    }
  }
  ```

---
> Source: [rcrdk/rcrdk.dev](https://github.com/rcrdk/rcrdk.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
