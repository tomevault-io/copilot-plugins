## ex-openai

> - Repo: dvcrn/ex_openai

- Repo: dvcrn/ex_openai

# Agent Notes

## Generated SDK workflow

- `lib/ex_openai/generated` is committed and is the runtime SDK surface.
- Do not hand-edit generated files unless the task is explicitly about generated output inspection. Normal changes should go into the generators under `lib/ex_openai/codegen`, then regenerate sources.
- After changing codegen or `lib/ex_openai/docs/docs.yaml`, run:
  - `mise exec -- mix generate_openai_sources`
  - `mise exec -- mix compile --warnings-as-errors`
  - `mise exec -- mix test`

## Formatting

- Run `mise exec -- mix format` on the whole repo, not on a single file subset. Formatting is expected to cover all files.

## Where to look

- `docs/codegen.md` for the overall codegen architecture and regeneration flow.
- `docs/parsingv2.md` for how `docs.yaml` is parsed into `Schema` and `Path` structs and then converted into typespecs.
- `docs/streaming.md` for streaming semantics, callback shapes, and `StreamingClient` behavior.
- `docs/configuration.md` for runtime configuration and HTTP client options.
- `docs/examples.md` for user-facing usage examples.
- Before relying on a doc example, verify it against the current generated API and tests.
- If behavior changes, update the matching doc in `docs/` and any mirrored example in `README.md`.

## Clean validation

- For authoritative validation, prefer a clean build:
  - `mise exec -- mix clean`
  - `mise exec -- mix compile --warnings-as-errors`
  - `mise exec -- mix dialyzer --no-compile --format dialyzer`
- Warm recompiles can emit module redefinition warnings for generated modules. Treat the clean build as the source of truth.

## Dialyzer

- `mix dialyzer` is expected to pass from a clean build.
- If you create scratch Dialyzer probes, keep them outside normal compilation flow. Manually compiled probes can pollute `_build` and create misleading warnings.
- When Dialyzer noise appears across many generated endpoint modules, check shared generator/runtime contracts first, especially `ExOpenAI.Codegen.DocsParser.Schema.t/0`.

## Streaming behavior

- `stream: true` API calls return `{:ok, reference()}`, not a PID.
- Chat streaming callbacks receive `%ExOpenAI.Components.CreateChatCompletionStreamResponse{}` chunks.
- Legacy `/completions` streaming uses `%ExOpenAI.Components.CreateCompletionResponse{}` chunks.
- Stream callback examples must handle partial deltas safely. Do not assume `choice.delta.content` always exists; use `Map.get(choice.delta, :content, "")`.

## Response conversion

- Stream and union response conversion relies on discriminator metadata normalized in `ExOpenAI.Codegen.DocsParser.Schema`.
- If you change discriminator parsing, keep the runtime `%Schema{}` shape compatible with `ExOpenAI.Codegen.ResponseConverter`.
- Prefer testing unions and streaming conversion using real examples from `lib/ex_openai/docs/docs.yaml`, not synthetic schemas.

## Tests

- When adding generator tests, prefer real excerpts and shapes from `lib/ex_openai/docs/docs.yaml`.
- If a test uses a synthetic schema, document why the real spec does not currently contain that exact shape.

## Public API typing

- Large keyword option specs are valid, but in practice Dialyzer only gets strong value from the positional arguments and broad return shape.
- If stronger static typing is needed, prefer API shape changes such as separate streaming functions over making keyword-list specs more complex.

---
> Source: [dvcrn/ex_openai](https://github.com/dvcrn/ex_openai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
