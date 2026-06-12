## architecture

> ├── chrome-extension/     # Chrome extension core


# Web Memo Technical Architecture

## 🏗 **Project Structure**

### **Monorepo Organization**
```
web-memo/
├── chrome-extension/     # Chrome extension core
├── pages/               # Extension pages (popup, side-panel, options)
├── packages/            # Shared utilities and components
├── shared/             # Common types and utilities
├── supabase/           # Database and authentication
└── e2e/               # End-to-end testing
```

### **Key Technologies**
- **Frontend**: React 18.3.1, TypeScript 5.5.3, TailwindCSS 3.4.x
- **Build Tools**: Vite 5.3.3, Turbo 2.1.1
- **State Management**: TanStack Query v5.59.0, React Hook Form 7.53.2
- **Testing**: Playwright 1.47.0, Vitest

## 🔧 **Development Patterns**

### **Component Architecture**
```typescript
// ✅ DO: Use functional components with clear interfaces
interface MemoCardProps {
  memo: Memo;
  onEdit: (id: string) => void;
  onDelete: (id: string) => void;
}

function MemoCard({ memo, onEdit, onDelete }: MemoCardProps) {
  // Component logic
}
```

### **State Management**
```typescript
// ✅ DO: Use TanStack Query for server state
function useMemos() {
  return useQuery({
    queryKey: ['memos'],
    queryFn: fetchMemos,
  });
}

// ✅ DO: Use React Hook Form for form state
function MemoForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<MemoFormData>();
  // Form logic
}
```

## 📱 **Chrome Extension Specifics**

### **Manifest V3 Compliance**
```typescript
// ✅ DO: Follow Manifest V3 patterns
const manifest = {
  manifest_version: 3,
  permissions: ['storage', 'activeTab'],
  host_permissions: ['<all_urls>'],
  background: { service_worker: 'background.js' }
};
```

### **Content Script Integration**
```typescript
// ✅ DO: Handle cross-origin restrictions gracefully
function injectContentScript(tabId: number) {
  try {
    await chrome.scripting.executeScript({
      target: { tabId },
      files: ['content.js']
    });
  } catch (error) {
    // Handle restricted pages (chrome://, etc.)
    console.warn('Cannot inject script on this page:', error);
  }
}
```

## 🎨 **UI/UX Guidelines**

### **Responsive Design**
```typescript
// ✅ DO: Use TailwindCSS responsive classes
<div className="w-full md:w-96 lg:w-[500px]">
  <div className="p-4 sm:p-6 lg:p-8">
    {/* Content */}
  </div>
</div>
```

### **Accessibility**
```typescript
// ✅ DO: Include proper ARIA labels and keyboard navigation
<button
  aria-label="Save memo"
  onKeyDown={(e) => e.key === 'Enter' && handleSave()}
  className="focus:ring-2 focus:ring-blue-500 focus:outline-none"
>
  Save
</button>
```

## 🧪 **Testing Strategy**

### **E2E Testing with Playwright**
```typescript
// ✅ DO: Test critical user flows
test('create memo from side panel', async ({ page }) => {
  await page.goto('https://example.com');
  await page.click('[data-testid="side-panel-toggle"]');
  await page.fill('[data-testid="memo-input"]', 'Test memo');
  await page.click('[data-testid="save-memo"]');

  await expect(page.locator('[data-testid="memo-list"]')).toContainText('Test memo');
});
```

### **Unit Testing with Vitest**
```typescript
// ✅ DO: Test utility functions and hooks
describe('useMemos', () => {
  it('should fetch memos successfully', async () => {
    const { result } = renderHook(() => useMemos());

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });
  });
});
```

## 🔒 **Security & Performance**

### **Data Validation**
```typescript
// ✅ DO: Validate all user inputs
import { z } from 'zod';

const MemoSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(1).max(10000),
  url: z.string().url(),
});

function validateMemo(data: unknown) {
  return MemoSchema.parse(data);
}
```

### **Performance Optimization**
```typescript
// ✅ DO: Implement proper memoization
const MemoList = memo(function MemoList({ memos }: { memos: Memo[] }) {
  // Component logic
});

// ✅ DO: Use React Query for caching
const { data: memos } = useQuery({
  queryKey: ['memos'],
  queryFn: fetchMemos,
  cacheTime: 10 * 60 * 1000,
});
```

## 📦 **Package Management**

### **Dependencies**
```json
// ✅ DO: Use pnpm for consistent package management
{
  "packageManager": "pnpm@9.5.0",
  "engines": {
    "node": ">=18.12.0"
  }
}
```

### **Workspace Configuration**
```json
// ✅ DO: Configure Turborepo for monorepo management
{
  "turbo": {
    "pipeline": {
      "build": {
        "dependsOn": ["^build"],
        "outputs": ["dist/**"]
      }
    }
  }
}
```

---
> Source: [guesung/Web-Memo](https://github.com/guesung/Web-Memo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
