## taeshigee

> handleSubmit,

# Frontend Design Guideline

This document summarizes key frontend design principles and rules, showcasing
recommended patterns. Follow these guidelines when writing frontend code.

# Readability

Improving the clarity and ease of understanding code.

## Naming Magic Numbers

**Rule:** Replace magic numbers with named constants for clarity.

**Reasoning:**
- Improves clarity by giving semantic meaning to unexplained values.
- Enhances maintainability.

#### Recommended Pattern:

```typescript
const ANIMATION_DELAY_MS = 300;

async function onLikeClick() {
  await postLike(url);
  await delay(ANIMATION_DELAY_MS); // Clearly indicates waiting for animation
  await refetchPostLike();
}
```

## Abstracting Implementation Details

**Rule:** Abstract complex logic/interactions into dedicated components/HOCs.

**Reasoning:**
- Reduces cognitive load by separating concerns.
- Improves readability, testability, and maintainability of components.

#### Recommended Pattern 1: Auth Guard

```tsx
// App structure
function App() {
  return (
    <AuthGuard>
      <LoginStartPage />
    </AuthGuard>
  );
}

// AuthGuard component encapsulates the check/redirect logic
function AuthGuard({ children }) {
  const status = useCheckLoginStatus();
  useEffect(() => {
    if (status === "LOGGED_IN") {
      location.href = "/home";
    }
  }, [status]);

  return status !== "LOGGED_IN" ? children : null;
}

// LoginStartPage is now simpler, focused only on login UI/logic
function LoginStartPage() {
  return <>{/* ... login related components ... */}</>;
}
```

#### Recommended Pattern 2: Dedicated Interaction Component

```tsx
export function FriendInvitation() {
  const { data } = useQuery(/* ... */);

  return (
    <>
      <InviteButton name={data.name} />
      {/* ... other UI ... */}
    </>
  );
}

// InviteButton handles the confirmation flow internally
function InviteButton({ name }) {
  const handleClick = async () => {
    const canInvite = await overlay.openAsync(({ isOpen, close }) => (
      <ConfirmDialog
        title={`Share with ${name}`}
        // ... dialog setup ...
      />
    ));

    if (canInvite) {
      await sendPush();
    }
  };

  return <Button onClick={handleClick}>Invite</Button>;
}
```

## Separating Code Paths for Conditional Rendering

**Rule:** Separate significantly different conditional UI/logic into distinct components.

**Reasoning:**
- Improves readability by avoiding complex conditionals within one component.
- Ensures each specialized component has a clear, single responsibility.

#### Recommended Pattern:

```tsx
function SubmitButton() {
  const isViewer = useRole() === "viewer";

  return isViewer ? <ViewerSubmitButton /> : <AdminSubmitButton />;
}

function ViewerSubmitButton() {
  return <TextButton disabled>Submit</TextButton>;
}

function AdminSubmitButton() {
  useEffect(() => {
    showAnimation();
  }, []);

  return <Button type="submit">Submit</Button>;
}
```

## Simplifying Complex Ternary Operators

**Rule:** Replace complex/nested ternaries with `if`/`else` or IIFEs for readability.

**Reasoning:**
- Makes conditional logic easier to follow quickly.
- Improves overall code maintainability.

#### Recommended Pattern:

```typescript
const status = (() => {
  if (ACondition && BCondition) return "BOTH";
  if (ACondition) return "A";
  if (BCondition) return "B";
  return "NONE";
})();
```

## Reducing Eye Movement (Colocating Simple Logic)

**Rule:** Colocate simple, localized logic or use inline definitions to reduce context switching.

**Reasoning:**
- Allows top-to-bottom reading and faster comprehension.
- Reduces cognitive load from context switching.

#### Recommended Pattern A: Inline `switch`

```tsx
function Page() {
  const user = useUser();

  switch (user.role) {
    case "admin":
      return (
        <div>
          <Button disabled={false}>Invite</Button>
          <Button disabled={false}>View</Button>
        </div>
      );
    case "viewer":
      return (
        <div>
          <Button disabled={true}>Invite</Button>
          <Button disabled={false}>View</Button>
        </div>
      );
    default:
      return null;
  }
}
```

#### Recommended Pattern B: Colocated simple policy object

```tsx
function Page() {
  const user = useUser();
  const policy = {
    admin: { canInvite: true, canView: true },
    viewer: { canInvite: false, canView: true },
  }[user.role];

  if (!policy) return null;

  return (
    <div>
      <Button disabled={!policy.canInvite}>Invite</Button>
      <Button disabled={!policy.canView}>View</Button>
    </div>
  );
}
```

## Naming Complex Conditions

**Rule:** Assign complex boolean conditions to named variables.

**Reasoning:**
- Makes the _meaning_ of the condition explicit.
- Improves readability and self-documentation by reducing cognitive load.

#### Recommended Pattern:

```typescript
const matchedProducts = products.filter((product) => {
  const isSameCategory = product.categories.some(
    (category) => category.id === targetCategory.id
  );

  const isPriceInRange = product.prices.some(
    (price) => price >= minPrice && price <= maxPrice
  );

  return isSameCategory && isPriceInRange;
});
```

# Predictability

Ensuring code behaves as expected based on its name, parameters, and context.

## Standardizing Return Types

**Rule:** Use consistent return types for similar functions/hooks.

**Reasoning:**
- Improves code predictability; developers can anticipate return value shapes.
- Reduces confusion and potential errors from inconsistent types.

#### Recommended Pattern 1: API Hooks (React Query)

```typescript
import { useQuery, UseQueryResult } from "@tanstack/react-query";

function useUser(): UseQueryResult<UserType, Error> {
  const query = useQuery({ queryKey: ["user"], queryFn: fetchUser });
  return query;
}

function useServerTime(): UseQueryResult<Date, Error> {
  const query = useQuery({
    queryKey: ["serverTime"],
    queryFn: fetchServerTime,
  });
  return query;
}
```

#### Recommended Pattern 2: Validation Functions

```typescript
type ValidationResult = { ok: true } | { ok: false; reason: string };

function checkIsNameValid(name: string): ValidationResult {
  if (name.length === 0) return { ok: false, reason: "Name cannot be empty." };
  if (name.length >= 20)
    return { ok: false, reason: "Name cannot be longer than 20 characters." };
  return { ok: true };
}

function checkIsAgeValid(age: number): ValidationResult {
  if (!Number.isInteger(age))
    return { ok: false, reason: "Age must be an integer." };
  if (age < 18) return { ok: false, reason: "Age must be 18 or older." };
  if (age > 99) return { ok: false, reason: "Age must be 99 or younger." };
  return { ok: true };
}
```

## Revealing Hidden Logic (Single Responsibility)

**Rule:** Avoid hidden side effects; functions should only perform actions implied by their signature (SRP).

**Reasoning:**
- Leads to predictable behavior without unintended side effects.
- Creates more robust, testable code through separation of concerns.

#### Recommended Pattern:

```typescript
// Function *only* fetches balance
async function fetchBalance(): Promise<number> {
  const balance = await http.get<number>("...");
  return balance;
}

// Caller explicitly performs logging where needed
async function handleUpdateClick() {
  const balance = await fetchBalance();
  logging.log("balance_fetched");
  await syncBalance(balance);
}
```

## Using Unique and Descriptive Names

**Rule:** Use unique, descriptive names for custom wrappers/functions to avoid ambiguity.

**Reasoning:**
- Avoids ambiguity and enhances predictability.
- Allows developers to understand specific actions directly from the name.

#### Recommended Pattern:

```typescript
// In httpService.ts
import { http as httpLibrary } from "@some-library/http";

export const httpService = {
  async getWithAuth(url: string) {
    const token = await fetchToken();
    return httpLibrary.get(url, {
      headers: { Authorization: `Bearer ${token}` },
    });
  },
};

// In fetchUser.ts
import { httpService } from "./httpService";
export async function fetchUser() {
  return await httpService.getWithAuth("...");
}
```

# Cohesion

Keeping related code together and ensuring modules have a well-defined, single purpose.

## Considering Form Cohesion

**Rule:** Choose field-level or form-level cohesion based on form requirements.

**Reasoning:**
- Balances field independence (field-level) vs. form unity (form-level).
- Ensures related form logic is appropriately grouped based on requirements.

#### Recommended Pattern (Field-Level Example):

```tsx
import { useForm } from "react-hook-form";

export function Form() {
  const {
    register,
    formState: { errors },
    handleSubmit,
  } = useForm();

  const onSubmit = handleSubmit((formData) => {
    console.log("Form submitted:", formData);
  });

  return (
    <form onSubmit={onSubmit}>
      <div>
        <input
          {...register("name", {
            validate: (value) =>
              value.trim() === "" ? "Please enter your name." : true,
          })}
          placeholder="Name"
        />
        {errors.name && <p>{errors.name.message}</p>}
      </div>
      <div>
        <input
          {...register("email", {
            validate: (value) =>
              /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(value)
                ? true
                : "Invalid email address.",
          })}
          placeholder="Email"
        />
        {errors.email && <p>{errors.email.message}</p>}
      </div>
      <button type="submit">Submit</button>
    </form>
  );
}
```

#### Recommended Pattern (Form-Level Example):

```tsx
import * as z from "zod";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

const schema = z.object({
  name: z.string().min(1, "Please enter your name."),
  email: z.string().min(1, "Please enter your email.").email("Invalid email."),
});

export function Form() {
  const {
    register,
    formState: { errors },
    handleSubmit,
  } = useForm({
    resolver: zodResolver(schema),
    defaultValues: { name: "", email: "" },
  });

  const onSubmit = handleSubmit((formData) => {
    console.log("Form submitted:", formData);
  });

  return (
    <form onSubmit={onSubmit}>
      <div>
        <input {...register("name")} placeholder="Name" />
        {errors.name && <p>{errors.name.message}</p>}
      </div>
      <div>
        <input {...register("email")} placeholder="Email" />
        {errors.email && <p>{errors.email.message}</p>}
      </div>
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Organizing Code by Feature/Domain

**Rule:** Organize directories by feature/domain, not just by code type.

**Reasoning:**
- Increases cohesion by keeping related files together.
- Simplifies feature understanding, development, maintenance, and deletion.

#### Recommended Pattern:

```
src/
├── components/ # Shared/common components
├── hooks/      # Shared/common hooks
├── utils/      # Shared/common utils
├── domains/
│   ├── user/
│   │   ├── components/
│   │   │   └── UserProfileCard.tsx
│   │   ├── hooks/
│   │   │   └── useUser.ts
│   │   └── index.ts
│   ├── product/
│   │   ├── components/
│   │   │   └── ProductList.tsx
│   │   ├── hooks/
│   │   │   └── useProducts.ts
│   │   └── ...
│   └── order/
│       ├── components/
│       │   └── OrderSummary.tsx
│       ├── hooks/
│       │   └── useOrder.ts
│       └── ...
└── App.tsx
```

## Relating Magic Numbers to Logic

**Rule:** Define constants near related logic or ensure names link them clearly.

**Reasoning:**
- Improves cohesion by linking constants to the logic they represent.
- Prevents silent failures caused by updating logic without updating related constants.

#### Recommended Pattern:

```typescript
const ANIMATION_DELAY_MS = 300;

async function onLikeClick() {
  await postLike(url);
  await delay(ANIMATION_DELAY_MS);
  await refetchPostLike();
}
```

# Coupling

Minimizing dependencies between different parts of the codebase.

## Balancing Abstraction and Coupling

**Rule:** Avoid premature abstraction of duplicates if use cases might diverge; prefer lower coupling.

**Reasoning:**
- Avoids tight coupling from forcing potentially diverging logic into one abstraction.
- Allowing some duplication can improve decoupling and maintainability when future needs are uncertain.

## Scoping State Management

**Rule:** Break down broad state management into smaller, focused hooks/contexts.

**Reasoning:**
- Reduces coupling by ensuring components only depend on necessary state slices.
- Improves performance by preventing unnecessary re-renders from unrelated state changes.

#### Recommended Pattern:

```typescript
import { useQueryParam, NumberParam } from "use-query-params";
import { useCallback } from "react";

export function useCardIdQueryParam() {
  const [cardIdParam, setCardIdParam] = useQueryParam("cardId", NumberParam);

  const setCardId = useCallback(
    (newCardId: number | undefined) => {
      setCardIdParam(newCardId, "replaceIn");
    },
    [setCardIdParam]
  );

  return [cardIdParam ?? undefined, setCardId] as const;
}
```

## Eliminating Props Drilling with Composition

**Rule:** Use Component Composition instead of Props Drilling.

**Reasoning:**
- Significantly reduces coupling by eliminating unnecessary intermediate dependencies.
- Makes refactoring easier and clarifies data flow in flatter component trees.

#### Recommended Pattern:

```tsx
function ItemEditModal({ open, items, recommendedItems, onConfirm, onClose }) {
  const [keyword, setKeyword] = useState("");

  return (
    <Modal open={open} onClose={onClose}>
      <div style={{ display: "flex", justifyContent: "space-between", marginBottom: "1rem" }}>
        <Input
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Search items..."
        />
        <Button onClick={onClose}>Close</Button>
      </div>
      <ItemEditList
        keyword={keyword}
        items={items}
        recommendedItems={recommendedItems}
        onConfirm={onConfirm}
      />
    </Modal>
  );
}
```

# Additional Guidelines

## Code Style & Standards
- Use consistent indentation (2 spaces for TypeScript/JavaScript)
- Follow TypeScript best practices and strict mode
- Write clean, readable, and maintainable code
- Add appropriate comments for complex logic
- Use meaningful variable and function names

## Testing
- Write tests for new functionality
- Ensure existing tests pass before making changes
- Use appropriate testing frameworks (Jest, React Testing Library)
- Include both unit and integration tests where applicable

## Git Workflow
- Use descriptive commit messages
- Make small, focused commits
- Follow conventional commit format when possible
- Keep the main branch stable

## Security
- Never hardcode sensitive information (API keys, passwords, etc.)
- Use environment variables for configuration
- Validate user inputs
- Follow security best practices for the technology stack

## Performance
- Optimize code for performance when necessary
- Avoid unnecessary computations
- Use appropriate data structures
- Consider scalability in design decisions

## Error Handling
- Implement proper error handling
- Provide meaningful error messages
- Log errors appropriately
- Gracefully handle edge cases

## Accessibility
- Follow accessibility guidelines (WCAG) for web applications
- Use semantic HTML
- Ensure keyboard navigation works
- Provide alternative text for images

## Internationalization
- Use UTF-8 encoding
- Support multiple languages if required
- Use appropriate date/time formatting
- Consider cultural differences in UI design

## Dependencies
- Keep dependencies up to date
- Use specific version numbers when possible
- Document why specific dependencies are needed
- Minimize the number of dependencies

## Code Review
- Review code for potential issues
- Suggest improvements when appropriate
- Ensure code follows project standards
- Check for security vulnerabilities

## Communication
- Ask clarifying questions when requirements are unclear
- Provide explanations for suggested changes
- Be helpful and constructive in feedback
- Respect the user's coding preferences and constraints

## Development Workflow
- Commit major changes with clear, concise summaries during development
- Use absolute paths when working with specific directories in Cursor IDE
- Make small, focused commits for better version control
- Document significant architectural decisions in commit messages 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dantwingx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
