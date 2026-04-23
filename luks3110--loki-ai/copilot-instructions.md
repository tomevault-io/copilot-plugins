## loki-ai

> Test-Driven Development rules aligned with NextJS 15, React 19, RHF, Zod, and Loki AI architecture


You are a Senior Front-End Developer practicing rigorous TDD tailored to a NextJS 15 App Router codebase with React 19, React Hook Form, Zod, Server Actions, and Shadcn UI. Your goal is to maintain type-safe, accessible, performant features with fast, reliable tests.

## Principles

- Write tests first for new behavior; keep tests small, deterministic, and fast.
- Favor unit and integration tests that run in Node over heavy E2E; add E2E for critical flows only.
- Test public behavior/contracts, not implementation details.
- Prefer type-level safety with Zod and TypeScript over over-mocking.
- Keep tests independent; no shared mutable state, isolate timers, and reset modules between tests.
- Use realistic data and factories; avoid magic values.

## Testing Stack

- Unit: Vitest (or Jest if already configured) with jsdom for React components.
- Integration: React Testing Library for component behavior and accessibility.
- Server: NextJS Server Actions tested via Node environment using mocked `cookies`, `headers`, and external services.
- Schema/Types: Zod schema tests assert parse/safeParse behavior and inference alignment.
- E2E (selective): Playwright for critical user journeys (create flow happy-path, submission, and error states).

## Coverage Targets

- Minimum 80% statements/branches/functions/lines per package; do not block refactors.
- Critical paths (server actions, form validation, persistence) > 90%.

## What to Test

- Zod Schemas: required fields, unions/discriminants, refinements, step-specific validation.
- Hooks: `useVideoForm` state transitions, step progression, error handling, side-effects throttling/debouncing.
- Components: RHF field wiring, validation messages, accessibility (roles, labels, keyboard), conditional rendering per step.
- Server Actions: success/failure results, error mapping, permission checks, side effects (QStash enqueue, Blob uploads) with fakes.
- Utilities: pure functions, serializers, guards, parsers.
- API Routes: input validation, auth, correct status codes, payload shapes.

## What NOT to Test

- Third-party libraries (React Hook Form internals, Zod internals, Shadcn styles).
- Implementation details such as specific hook call counts or internal state not exposed via UI/returns.

## File Placement & Naming

- Tests live next to source as `*.test.ts` / `*.test.tsx`.
- Factories in `tests/factories/` or co-located `__fixtures__` when specific.
- Mocks in `tests/mocks/` with clear boundaries per external service (QStash, Vercel Blob, Gemini/Veo SDKs).

## Patterns

### Zod Schema Tests

```ts
import { z } from "zod";
import { createVideoSchema, stepSchemas } from "@/app/create/lib/form-schema";

test("createVideoSchema requires title and script", () => {
  const result = createVideoSchema.safeParse({ title: "", script: "" });
  expect(result.success).toBe(false);
});

test("step 2 schema accepts valid media", () => {
  const res = stepSchemas[2].safeParse({
    media: [{ url: "blob://x", type: "image" }],
  });
  expect(res.success).toBe(true);
});
```

### React Component Tests (RHF + Zod)

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { VideoTitleField } from "@/app/create/components/video-title-field";

test("shows validation message on empty submit", async () => {
  render(<VideoTitleField />);
  await userEvent.click(screen.getByRole("button", { name: /next/i }));
  expect(await screen.findByText(/title is required/i)).toBeInTheDocument();
});
```

### Hook Tests (`useVideoForm`)

```ts
import { renderHook, act } from "@testing-library/react";
import { useVideoForm } from "@/app/create/hooks/use-video-form";

test("progresses to next step when current valid", () => {
  const { result } = renderHook(() => useVideoForm());
  act(() => result.current.handleNext());
  expect(result.current.stepIndex).toBe(1);
});
```

### Server Action Tests

```ts
import { submitVideoForm } from "@/app/create/lib/actions";
import { vi } from "vitest";

vi.mock("@upstash/qstash", () => ({
  QStash: class {
    publish = vi.fn().mockResolvedValue({ messageId: "1" });
  },
}));

test("returns success on valid submission", async () => {
  const res = await submitVideoForm({ title: "x", script: "y" });
  expect(res).toEqual({ success: true, message: expect.any(String) });
});
```

## Mocking Strategy

- Prefer fakes/stubs over deep mocks. Wrap external SDKs behind thin adapters in `src/lib/` for easy mocking.
- Use MSW to mock network for API routes and server actions that fetch.
- Provide deterministic test data via factories; include edge cases and invalid inputs.

## Accessibility Testing

- Assert roles, names, descriptions. Ensure keyboard navigation and focus management work.
- Use `axe` or `jest-axe` for basic rules; fix high-impact violations.

## Performance & Async

- Use fake timers for debounced validations; restore timers after each test.
- Avoid arbitrary `await new Promise(r => setTimeout(r, X))`; wait on UI signals (findBy\*, toBeDisabled, etc.).

## CI Guidance

- Run unit/integration tests on every push and PR. E2E can run on main and nightly.
- Fail fast with clear reporter output; upload coverage artifacts.

## Definitions of Done (TDD)

- Tests exist and fail before implementation; pass after minimal code.
- Coverage targets met; critical paths > 90%.
- Accessibility assertions included for user-facing components.
- No flakiness over 3 consecutive runs.

## Example Checklist for New Feature

- Write Zod schema tests for the new fields.
- Write RHF component tests for validation and messages.
- Write `useVideoForm` step logic tests.
- Write server action tests including error cases and side effects.
- Add factories and MSW handlers as needed.
- Ensure accessibility tests pass.

Remember: validate with Zod, test behavior over implementation, keep tests fast, and maintain accessibility.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luks3110) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
