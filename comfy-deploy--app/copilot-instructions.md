## frontend-rules

> - The frontend uses a custom override for `useQuery` where the queryKey follows a resource-based pattern

# Codebase Query Handling Documentation

## Frontend Query Pattern
- The frontend uses a custom override for `useQuery` where the queryKey follows a resource-based pattern
- No need to specify `queryFn` as it's handled by the override
- For mutations, use `useMutation` with proper error handling and query invalidation

### Example Usage:
```typescript
// ❌ Don't do this:
const { data } = useQuery({
  queryKey: [`${process.env.NEXT_PUBLIC_CD_API_URL}/api/machine/${machineId}/check-custom-nodes`],
  queryFn: async () => {
    const response = await fetch(...);
    return response.json();
  }
});

// ✅ Do this instead:
const { data } = useQuery({
  queryKey: ["machine", machineId, "check-custom-nodes"]
});

// ✅ For mutations:
const mutation = useMutation({
  mutationFn: async () => {
    return api({
      url: `machine/${machine.id}/update-custom-nodes`,
      init: { method: "POST" }
    });
  },
  onSuccess: () => {
    // Invalidate and refetch relevant queries
    queryClient.invalidateQueries({ queryKey: ["machine", machine.id] });
  }
});
```

The queryKey pattern follows a resource-based structure:
- First element: resource type (e.g., "machine")
- Second element: resource identifier (e.g., machineId)
- Additional elements: specific actions or sub-resources (e.g., "check-custom-nodes")

This pattern makes the queries more consistent and easier to manage across the application.

## Mutation Pattern
- Use the `api` utility for all API calls instead of direct fetch
- Handle loading states with proper UI feedback
- Always invalidate relevant queries on success
- Provide clear error messages through toast notifications

### Example Usage:
```typescript
const updateMutation = useMutation({
  mutationFn: async () => {
    return api({
      url: `machine/${machine.id}/update-custom-nodes`,
      init: { method: "POST" }
    });
  },
  onSuccess: () => {
    toast.success("Custom nodes update initiated");
    onOpenChange(false);
    // Invalidate and refetch relevant queries
    queryClient.invalidateQueries({ queryKey: ["machine", machine.id] });
    queryClient.invalidateQueries({
      queryKey: ["machine", machine.id, "versions"]
    });
  },
  onError: (error) => {
    toast.error("Failed to update custom nodes");
  }
});
```

## Dialog Management
- Use URL parameters to control dialog state when appropriate
- Handle dialog state cleanup on navigation
- Provide clear loading states during async operations
- Include proper TypeScript types for all props

### Example Structure:
```typescript
function UpdateDialog({
  machine,
  open,
  onOpenChange,
}: {
  machine: Machine;  // Use proper types
  open: boolean;
  onOpenChange: (open: boolean) => void;
}) {
  const navigate = useNavigate();
  const queryClient = useQueryClient();

  // Handle URL-based state
  useEffect(() => {
    if (someURLParam) {
      setDialogOpen(true);
    }
  }, [someURLParam]);

  // Cleanup on dialog close
  const handleClose = () => {
    onOpenChange(false);
    navigate({
      to: "/some/path",
      params: { /* cleaned params */ }
    });
  };

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      {/* Dialog content */}
    </Dialog>
  );
}
```

## Frontend Components
- Workspace components are in `frontend/src/components/workspace/`
- `WorkspaceClientWrapper.tsx` handles the main workspace UI logic
- Update alerts should be placed before the main content but after loading states
- Dialog components should follow this structure:
  ```typescript
  function CustomDialog({
    machine,
    open,
    onOpenChange,
  }: {
    machine: Machine; // Use proper types, avoid 'any'
    open: boolean;
    onOpenChange: (open: boolean) => void;
  })
  ```

## Error Handling
- Always handle loading states with proper UI feedback
- Use toast notifications for success/error feedback
- Implement proper TypeScript types to avoid 'any' usage
- Handle edge cases in useEffect hooks with proper cleanup

## CSS Classes
- Sort CSS classes alphabetically
- Use consistent naming conventions
- Prefer Tailwind utility classes
- Use cn() utility for conditional classes

## State Management
- Use proper state initialization in useEffect
- Clean up side effects in useEffect returns
- Handle URL parameters through proper router hooks
- Use URL state for dialog visibility when appropriate 

---
> Source: [comfy-deploy/app](https://github.com/comfy-deploy/app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
