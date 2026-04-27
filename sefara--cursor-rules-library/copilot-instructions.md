## cursor-rules-library

> These patterns apply to both n8n, ActivePieces, and other workflow automation tools.

# Workflow Automation Patterns

## Universal Workflow Patterns

These patterns apply to both n8n, ActivePieces, and other workflow automation tools.

## 1. Linear Workflow

Simple sequential processing:
```
Trigger → Step 1 → Step 2 → Step 3 → Result
```

**Use when**: Simple, linear processes with no branching.

## 2. Conditional Branching

Different paths based on conditions:
```
Trigger → Condition Check → [Path A] → [Path B] → Merge → Result
```

**Use when**: Different actions needed based on data or state.

## 3. Parallel Processing

Execute multiple branches simultaneously:
```
Trigger → [Branch A] ─┐
       → [Branch B] ─┼→ Merge → Result
       → [Branch C] ─┘
```

**Use when**: Independent operations can run concurrently.

## 4. Loop Pattern

Process each item in a collection:
```
Trigger → Split Items → Process Item → Collect → Result
```

**Use when**: Processing batches of similar items.

## 5. Error Handling Pattern

Graceful error recovery:
```
Main Flow → Try → [Success Path] ─┐
         ↓ Error                    │
         → Catch → Log → Notify ──→ Merge → Result
```

**Use when**: External APIs or unreliable services are involved.

## 6. Retry Pattern

Automatic retry on failure:
```
Action → Success? → [Yes] → Continue
       ↓ No
       → Wait → Retry (max 3x) → [Success] → Continue
                              ↓ Fail
                              → Error Handler
```

## 7. Fan-Out/Fan-In

Distribute work, then aggregate:
```
Trigger → Split → [Worker 1] ─┐
                → [Worker 2] ─┼→ Aggregate → Result
                → [Worker 3] ─┘
```

## 8. Queue Pattern

Asynchronous processing:
```
Trigger → Enqueue → Background Processor → Result
```

## 9. Data Transformation Pipeline

Transform data through stages:
```
Raw Data → Validate → Transform → Enrich → Format → Output
```

## 10. Event-Driven

React to events:
```
Event Source → Filter → Route → Process → Action
```

## Best Practices Across All Tools

### Data Management
- **Validate Early**: Check data format and required fields at entry
- **Transform Once**: Transform data shape early, reuse downstream
- **Store Temporarily**: Use variables/workflow state for complex data structures

### Error Handling
- **Always Handle Errors**: Every external call should have error handling
- **Log Everything**: Log errors with context for debugging
- **User-Friendly Messages**: Convert technical errors to user-friendly messages

### Performance
- **Batch Operations**: Process items in batches when possible
- **Parallel Execution**: Use parallel branches for independent operations
- **Rate Limiting**: Add delays/throttling for external APIs
- **Caching**: Cache data that doesn't change frequently

### Security
- **Secrets Management**: Never hardcode credentials
- **Input Validation**: Validate all external inputs
- **Sanitization**: Sanitize data before processing
- **Authentication**: Verify webhooks and API requests

### Testing
1. **Unit Testing**: Test individual nodes/pieces
2. **Integration Testing**: Test complete workflows
3. **Edge Cases**: Test with empty, null, and invalid data
4. **Load Testing**: Test with realistic data volumes

### Documentation
- **Workflow Description**: Document purpose and trigger conditions
- **Node/Piece Documentation**: Add descriptions for complex logic
- **Data Flow**: Document expected data structures
- **Dependencies**: List external services and their requirements

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sefara) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
