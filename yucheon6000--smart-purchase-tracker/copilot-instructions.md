## smart-purchase-tracker

> The application uses Redux Toolkit for state management. The store is configured in `src/store/` directory.

# State Management Guide

## Redux Store Structure

The application uses Redux Toolkit for state management. The store is configured in `src/store/` directory.

### Store Configuration
- [src/store/index.ts](mdc:src/store/index.ts) - Main store configuration
- Redux slices are organized by feature

### State Organization

1. User State
   - Authentication status
   - User preferences
   - Profile information

2. Purchase State
   - Purchase history
   - Current purchase
   - Purchase categories

3. UI State
   - Theme preferences
   - Navigation state
   - Modal states

## Best Practices

1. Use Redux Toolkit's createSlice for state management
2. Implement proper TypeScript types for state
3. Use selectors for accessing state
4. Implement proper error handling
5. Use middleware for side effects
6. Keep state normalized
7. Implement proper loading states

## State Updates

1. Use createAsyncThunk for async operations
2. Implement proper error handling
3. Use proper loading states
4. Implement proper optimistic updates
5. Use proper TypeScript types

## Performance Considerations

1. Use proper memoization
2. Implement proper selectors
3. Use proper state normalization
4. Implement proper loading states
5. Use proper error handling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yucheon6000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yucheon6000)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
