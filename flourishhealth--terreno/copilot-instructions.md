## terreno

> Every field in a Mongoose schema **must** have a `description` property. Descriptions flow through to the OpenAPI spec via `mongoose-to-swagger`, making the generated API documentation and SDK more useful.

# Model Field Descriptions

Every field in a Mongoose schema **must** have a `description` property. Descriptions flow through to the OpenAPI spec via `mongoose-to-swagger`, making the generated API documentation and SDK more useful.

## Rule

When defining or modifying a Mongoose model schema, include a `description` on every field. The description should briefly explain what the field represents.

## Example

```typescript
const todoSchema = new mongoose.Schema<TodoDocument, TodoModel>(
  {
    completed: {
      default: false,
      description: "Whether the todo item has been completed",
      type: Boolean,
    },
    ownerId: {
      description: "The user who owns this todo",
      ref: "User",
      required: true,
      type: mongoose.Schema.Types.ObjectId,
    },
    title: {
      description: "The title of the todo item",
      required: true,
      trim: true,
      type: String,
    },
  },
  {strict: "throw", toJSON: {virtuals: true}, toObject: {virtuals: true}}
);
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FlourishHealth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
