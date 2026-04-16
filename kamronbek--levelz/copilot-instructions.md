## levelz

> providesTags: ["Feature"],


# Redux Toolkit and State Management Patterns

## Store Configuration

- **Store**: Configured in [frontend/src/store.ts](mdc:frontend/src/store.ts) with RTK Query integration
- **Dev Tools**: Always enabled for development debugging
- **Middleware**: Includes RTK Query middleware for API caching

## Slice Patterns

Create feature-specific slices in [frontend/src/slices/](mdc:frontend/src/slices/):

### Regular Slices (authSlice pattern)

```typescript
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

interface SliceState {
  // Define state shape
}

const initialState: SliceState = {
  // Initial state
};

const featureSlice = createSlice({
  name: "feature",
  initialState,
  reducers: {
    actionName: (state, action: PayloadAction<PayloadType>) => {
      // Mutate state directly (Immer under the hood)
    },
  },
});

export const { actionName } = featureSlice.actions;
export default featureSlice.reducer;
```

### API Slices (RTK Query pattern)

```typescript
import { apiSlice } from "./apiSlice";

export const featureApiSlice = apiSlice.injectEndpoints({
  endpoints: (builder) => ({
    getFeatures: builder.query<FeatureType[], void>({
      query: () => "/api/features",
      providesTags: ["Feature"],
    }),
    createFeature: builder.mutation<FeatureType, Partial<FeatureType>>({
      query: (data) => ({
        url: "/api/features",
        method: "POST",
        body: data,
      }),
      invalidatesTags: ["Feature"],
    }),
  }),
});

export const { useGetFeaturesQuery, useCreateFeatureMutation } =
  featureApiSlice;
```

## Authentication State Pattern

- **Auth Slice**: [frontend/src/slices/authSlice.ts](mdc:frontend/src/slices/authSlice.ts) manages user session
- **Local Storage**: Sync auth state with localStorage automatically
- **Actions**: `setCredentials` for login, `logout` for clearing state
- **Error Handling**: Graceful handling of localStorage errors

## API State Management

- **Base API**: [frontend/src/slices/apiSlice.ts](mdc:frontend/src/slices/apiSlice.ts) with JWT token injection
- **Caching**: RTK Query provides automatic caching and invalidation
- **Tags**: Use consistent tag naming for cache invalidation
- **Error Handling**: Global error handling in base API slice

## Hook Patterns

- **Typed Hooks**: [frontend/src/hooks/redux.ts](mdc:frontend/src/hooks/redux.ts) provides typed useSelector and useDispatch
- **Component Usage**: Always use typed hooks instead of raw Redux hooks

```typescript
import { useSelector, useDispatch } from "../hooks/redux";

const MyComponent = () => {
  const { userInfo } = useSelector((state) => state.auth);
  const dispatch = useDispatch();

  // Fully typed throughout
};
```

## State Shape Patterns

- **Normalized**: Keep state flat and normalized when possible
- **Loading States**: RTK Query handles loading/error states automatically
- **Optimistic Updates**: Use RTK Query's optimistic updates for better UX

## Exam System Specific State

- **User Info**: Stored in auth slice with role-based permissions
- **Exam Data**: Managed by examApiSlice with proper caching
- **Results**: Managed by resultsApiSlice with proper cache invalidation
- **Cheating Logs**: Real-time updates via cheatingLogApiSlice

## Performance Considerations

- **Selectors**: Use reselect for computed values (RTK includes createSelector)
- **Memoization**: RTK Query handles most memoization automatically
- **Re-renders**: Use selector patterns to minimize component re-renders

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KAMRONBEK) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
