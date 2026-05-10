## melony-core

> Overview of the Melony minimalist runtime


# Melony Core Runtime

Melony is a minimalist, tiny, and unopinionated agent framework based on an event-driven async generator architecture.

## Core Principles

- **Tiny**: Minimal dependencies, core is < 1000 lines.
- **Unopinionated**: You define your own `State` and `Event` types.
- **Event-Driven**: Everything is an event, processed via async generators.

## Key API

### Creating a Builder

```typescript
import { melony } from 'melony';
const app = melony<MyState, MyEvent>();
```

### Event Handlers

Handlers are async generators that can yield multiple events back to the stream.

```typescript
app.on('my-event', async function* (event, context) {
  // context.state is the current state
  yield { type: 'response', data: 'hello' };
});
```

### Interceptors

Interceptors can modify events or cancel them before handlers run.

```typescript
app.intercept(async (event, context) => {
  console.log('Intercepting:', event.type);
  return event; // Or return undefined to cancel
});
```

### Plugins

Plugins are just functions that take the builder.

```typescript
type MyPlugin = MelonyPlugin<MyState, MyEvent>;
const myPlugin: MyPlugin = (builder) => {
  builder.on('plugin-event', ...);
};
app.use(myPlugin);
```

### Executing

```typescript
const generator = await app.run({ type: 'start' });
for await (const event of generator) {
  console.log(event);
}
```

---
> Source: [ddaras/melony](https://github.com/ddaras/melony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
