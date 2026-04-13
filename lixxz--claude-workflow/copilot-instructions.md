## claude-workflow

> > This document contains **universal principles** (applicable to all languages) and **ecosystem-specific rules**.

# Engineering Principles & Coding Standards
---

> This document contains **universal principles** (applicable to all languages) and **ecosystem-specific rules**.
> - Unmarked sections apply universally
> - 🐍 = Python/Django specific
> - ⚛️ = React Native/TypeScript specific

---

## Part I: Universal Principles

These principles apply to ALL programming languages and paradigms.

### 1. Deep Modules

> *"Simple is better than complex. Complex is better than complicated."*

We prefer classes/methods with simple public interfaces that encapsulate complex logic.

| Aspect | Approach |
|--------|----------|
| **Guideline** | A method like `project.build()` or a hook like `useAuth()` can be complex internally if it offers a simple interface |
| **Goal** | Reduce cognitive load for the consumer |
| **Structure** | Public methods = Orchestrators. Private methods = Implementation steps |

### 2. Data Contracts

> *"Explicit is better than implicit."*

We value typed, predictable return shapes over loose collections.

| Concern | Rule |
|---------|------|
| **Structured Returns** | Never return raw tuples, arrays, or untyped dicts/objects for complex data. Use structured types (dataclasses, interfaces, types) so consumers access data by name, not index. |
| **Boundaries** | Data crossing module boundaries should be structured DTOs, decoupling internal models from public contracts. |

### 3. Semantic Failures

> *"Errors should never pass silently. Unless explicitly silenced."*

We prefer clear, domain-specific failures over generic technical crashes.

| Concern | Rule |
|---------|------|
| **Translation** | Catch low-level technical errors at the boundary and re-raise/transform them as Domain Errors that callers can understand. |
| **Granularity** | Do not catch all exceptions blindly. Only catch what you can handle or need to wrap. |

### 4. Diagnostic Discipline

> *"In the face of ambiguity, refuse the temptation to guess."*

Before proposing fixes, validate your assumptions about root cause. This applies to ALL languages.

| Concern | Rule |
|---------|------|
| **Symptom ≠ Cause** | An error message describes WHAT failed, not WHY. Never assume the cause. |
| **Hypothesis First** | State your hypothesis explicitly before investigating: "I suspect X because Y" |
| **Validation Required** | Use evidence (logs, traces, reproduction, code inspection) to confirm or refute hypothesis BEFORE coding a fix |
| **Minimal Reproduction** | Isolate the failure to the smallest possible scope before debugging |

#### Debugging Protocol

1. **Observe**: What exactly is failing? (error message, stack trace, behavior)
2. **Hypothesize**: What could cause this? (state explicitly)
3. **Validate**: How can I confirm this hypothesis? (add logging, reproduce, inspect state)
4. **Confirm or Pivot**: Did evidence support hypothesis? If not, form new hypothesis.
5. **Fix**: Only now write the fix—addressing the validated root cause.

#### Anti-Pattern: Assumption-Driven Fixes

```
# BAD: Jumped to conclusion
Symptom: "Cannot read property 'name' of undefined"
Assumption: "user must be undefined"
Fix without validation: Add null check blindly

# GOOD: Validated first
Symptom: "Cannot read property 'name' of undefined"
Hypothesis: "user might be undefined when session expires"
Validation: Added logging, found user exists but user.profile is undefined
Actual fix: Check user.profile, not just user
```

---

## Part II: React Native / TypeScript ⚛️

### Foundational Philosophies

#### The Tao of React (Mental Model)

Unlike Python's "Zen," React relies on structural constraints.

| Philosophy | Meaning in Practice |
|------------|---------------------|
| **UI = f(State)** | The interface is a pure function of your data. Never manually update a text box; update the state variable, and the box updates itself. |
| **Declarative > Imperative** | Describe what the UI should look like now, not how to change it from the previous state. |
| **Unidirectional Data Flow** | Data flows down (Props). Events flow up (Callbacks). Never try to "reach up" and modify a parent directly. |
| **Composition > Inheritance** | Never subclass Components. Build complex UIs by wrapping simple components inside container components. |
| **Colocation** | Keep logic, styles, and tests close to the component that uses them. Don't scatter files by type. |

#### React Native Design Philosophies

##### Components

| Principle | Description |
|-----------|-------------|
| **Functional Components Only** | Class components are legacy. Use Functional Components with Hooks. |
| **Render Purity** | The body of a component function must be pure. No side effects (API calls, subscriptions) in the render body; put them in `useEffect` or event handlers. |
| **Props are Read-Only** | A component must never modify its own props. |

##### State Management

| Principle | Description |
|-----------|-------------|
| **Lift State Up** | If two components need the same data, move the state to their nearest common ancestor. |
| **Server vs. Client State** | Crucial: Do not treat API data the same as UI state (like `isModalOpen`). Use a dedicated async state manager (TanStack Query) for server data. |
| **Single Source of Truth** | Avoid "Derived State" in storage. If you have `firstName` and `lastName`, do not store `fullName`. Calculate it on the fly during render. |

### Project Rules

#### 1. The "Hook" Abstraction (Deep Modules)

> In Python, we use Classes to hide complexity. In React, we use Custom Hooks.

Components should focus on Layout (Visuals). Logic should be extracted to Hooks.

| Aspect | Rule |
|--------|------|
| **Separation** | If a component has more than 3 `useEffect` calls or complex logic, extract it into a custom hook (e.g., `useProfileLogic()`). |
| **The Interface** | The Hook returns an object containing data and handlers. The Component just wires them to the UI. |
| **Benefit** | This makes logic testable in isolation from the UI rendering. |

#### 2. "Props" Hygiene (Data Contracts)

> The equivalent of "No Tuples" in Python.

| Rule | Description |
|------|-------------|
| **Named Parameters (Objects)** | In JavaScript functions (and Component Props), always pass objects, not positional arguments. Bad: `updateUser('John', 25, true)`. Good: `updateUser({ name: 'John', age: 25, isAdmin: true })` |
| **Discriminated Unions** | Never use boolean flags for multi-state logic. Bad: `isLoading, isError, isSuccess`. Good: `status: 'loading' | 'error' | 'success'` |
| **No Prop Drilling** | If you are passing data through more than 2 layers of components that don't use it, use Context or Composition. |

#### 3. Asynchronous Safety

> JavaScript is non-blocking, which creates race conditions Python devs rarely face.

| Concern | Rule |
|---------|------|
| **Race Conditions** | When fetching data in a `useEffect`, ensure the component is still mounted before setting state, or use a library (TanStack Query) that handles cancellation automatically. |
| **Loading States** | Every async action must have a visible Loading state and Error state in the UI. |
| **Optimistic Updates** | For high-quality feel, update the UI before the server responds (then rollback on error). |

#### 4. Styling & Performance

| Concern | Rule |
|---------|------|
| **Flexbox Only** | Never use hardcoded Dimensions (pixels) for layout structure. Use Flexbox (`flex: 1`, `flexDirection`). Screens are different sizes! |
| **Memoization** | If a child component re-renders too often, use `React.memo`. Wrap heavy functions in `useCallback`. |
| **FlatList Hygiene** | Never use `map()` for long lists. Use `FlatList` to recycle memory. |

### Code Examples

#### Custom Hooks (Deep Modules)

```typescript
// BAD: Fat Component (Logic mixed with View)
// Violation: Component knows too much about fetching and transforming data
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    api.fetchUser(userId).then(u => {
      setUser(u);
      setLoading(false);
    });
  }, [userId]);

  if (loading) return <Spinner />;
  return <Text>{user.name}</Text>;
};

// GOOD: Logic Encapsulation
// 1. The Logic (The "Deep Module")
const useUserProfile = (userId: string) => {
  // Uses a library like TanStack Query for caching/loading states
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => api.fetchUser(userId)
  });

  return {
    user: data,
    isLoading,
    isError: !!error
  };
};

// 2. The View (Pure UI)
const UserProfile = ({ userId }: { userId: string }) => {
  const { user, isLoading, isError } = useUserProfile(userId);

  if (isLoading) return <Spinner />;
  if (isError) return <ErrorView />;

  return <Text>{user.name}</Text>;
};
```

#### Named Parameters (Data Contracts)

```typescript
// BAD: Positional Arguments (The Tuple Anti-Pattern)
// Violation: What does 'true' mean? What is 500?
// If I change the order, everything breaks.
function createAnimation(ref, 500, true, 'easeIn') { ... }

// GOOD: Options Object
// Standard JS Pattern: "Roving Object"
interface AnimationConfig {
  duration: number;
  useNativeDriver: boolean;
  easing?: 'easeIn' | 'easeOut';
}

function createAnimation(ref: any, config: AnimationConfig) { ... }

// Usage is self-documenting
createAnimation(myRef, {
  duration: 500,
  useNativeDriver: true
});
```

#### Discriminated Unions (State Hygiene)

```typescript
// BAD: Boolean Soup
// Violation: Impossible states (e.g., loading=true AND error=true)
interface State {
  isLoading: boolean;
  isError: boolean;
  data: User | null;
  error: string | null;
}

// GOOD: Mathematical State
// The compiler ensures you handle every case
type State =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User }
  | { status: 'error'; error: string };

// Usage
if (state.status === 'success') {
  console.log(state.data.name); // TS knows 'data' exists here
}
```

#### Derived State (Single Source of Truth)

```typescript
// BAD: Redundant State
const [items, setItems] = useState([1, 2, 3]);
// Violation: 'count' is redundant. If 'items' updates, 'count' might drift.
const [count, setCount] = useState(3);

// GOOD: Calculated Values
const [items, setItems] = useState([1, 2, 3]);
// 'count' is calculated on every render. It cannot be out of sync.
const count = items.length;
```

---

## Part III: Python/Django Philosophies & Rules 🐍

### Foundational Philosophies

#### The Zen of Python (PEP 20)

```
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

#### Django Design Philosophies

##### Overall

| Principle | Description |
|-----------|-------------|
| **Loose Coupling** | Layers shouldn't "know" about each other unless necessary. Template system knows nothing about web requests; database layer knows nothing about display |
| **Less Code** | Apps should lack boilerplate. Take advantage of Python's dynamic capabilities |
| **Quick Development** | Make tedious aspects of web development fast |
| **DRY** | Every concept/data should live in one place. Redundancy is bad. Normalization is good |
| **Explicit > Implicit** | No "magic" unless it creates huge convenience unattainable otherwise |
| **Consistency** | Consistent at all levels—coding style to user experience |

##### Models

| Principle | Description |
|-----------|-------------|
| **Explicit > Implicit** | Fields shouldn't assume behaviors based on name. Use keyword arguments |
| **Include All Domain Logic** | Models encapsulate every aspect of an "object" (Active Record pattern). All info needed to understand a model should be stored *in* the model |

##### Database API

| Principle | Description |
|-----------|-------------|
| **SQL Efficiency** | Execute SQL as few times as possible. Optimize internally. Explicit `save()` calls |
| **Terse, Powerful Syntax** | Rich statements in minimal syntax. Automatic joins when needed |
| **Raw SQL Escape Hatch** | Easy to write custom SQL when the ORM isn't enough |

##### Views

| Principle | Description |
|-----------|-------------|
| **Simplicity** | Writing a view = writing a Python function. No class instantiation when function suffices |
| **Request Objects** | Pass request directly to view (not global). Makes testing easy |
| **GET vs POST** | Explicitly distinguish between them |

### Project Rules

#### 1. State Hygiene

> *"Explicit is better than implicit. Errors should never pass silently."*

While Django is stateful, we minimize side effects in business logic.

| Rule | Description |
|------|-------------|
| **Look, Don't Touch** | Do not mutate Model instances unless the function's explicit purpose is mutation |
| **Isolate IO** | Separate pure logic (transformations) from IO (saving, emailing) |
| **Immutability** | Prefer Pydantic models or dicts over raw Django models when mutation isn't required |

#### 2. Database Sovereignty

> *"There should be one-- and preferably only one --obvious way to do it."*

The Database is the source of truth, not Python.

| Concern | Rule |
|---------|------|
| **Enums** | ALWAYS use `TextChoices` for status/types. NEVER use string literals |
| **Constraints** | ALWAYS use `UniqueConstraint` and `CheckConstraint` in Meta |
| **Transactions** | ALWAYS wrap multi-step DB writes in `transaction.atomic()` |
| **Deletes** | Default to `on_delete=models.PROTECT`. Use `CASCADE` only for strict ownership |

#### 3. Coding Style & Safety

> *"Readability counts. Explicit is better than implicit."*

| Concern | Rule |
|---------|------|
| **Typing** | Strictly typed signatures: `def func(x: int) -> bool:`. |
| **Naming** | Descriptive names (`extract_paths` > `data`). Named kwargs for complex calls. |
| **Model-Centric Logic** | Models are the **entry point** for logic. If a method exceeds ~20 lines or requires complex imports, extract it to a standalone Service or Selector class. |
| **Bulk Ops** | Use `bulk_create`/`bulk_update`. Never save in a loop. |

---

## Part IV: Python/Django Examples & Practices 🐍

### Code Examples

#### State Hygiene

```python
# BAD: Implicit mutation and mixed concerns
# Violates: "Explicit is better than implicit"
def check_status(job):
    if job.is_stuck:
        job.status = 'failed'  # Side effect
        job.save()             # IO inside logic
        return False
    return True

# GOOD: Pure logic + Explicit IO
# Follows: "Simple is better than complex"
def get_job_status(job) -> Status:
    if job.is_stuck:
        return Status.FAILED
    return Status.RUNNING

# Caller handles the mutation explicitly
new_status = get_job_status(job)
if new_status != job.status:
    job.status = new_status
    job.save(update_fields=['status'])
```

#### Deep Modules & Model Centric Logic

```python
# Follows: "Include all relevant domain logic" (Django Models philosophy)
# Follows: "Simple is better than complex"
class DbtProject(models.Model):
    def build(self):
        """
        Deep Method: Simple interface, complex execution.
        Acts as a 'Table of Contents' for the operation.
        """
        # Note: If these methods are huge, they delegate to a Service class
        self._load_configs()
        self._run_dbt_cli()
        self._update_nodes()
        self._generate_flows()
```

#### Data Contracts (No Tuples)

```python
# BAD: "Mystery Meat" Return
# Violates: "Explicit is better than implicit"
# Problem: Consumer must memorize that index 1 is conversion_rate
def get_dashboard_metrics():
    return 120, 45.5, True

# GOOD: Structured Contract
# Follows: "Data Contracts"
from dataclasses import dataclass

@dataclass(frozen=True)
class DashboardMetrics:
    total_users: int
    conversion_rate: float
    is_feature_flag_enabled: bool

def get_dashboard_metrics() -> DashboardMetrics:
    return DashboardMetrics(
        total_users=120,
        conversion_rate=45.5,
        is_feature_flag_enabled=True
    )
```

#### Semantic Failures

```python
# BAD: Leaking Implementation Details
# Violates: "Simplicity" (The view has to know about Database integrity)
def register_user(email):
    # This might raise IntegrityError if email exists, crashing the view
    User.objects.create(email=email)

# GOOD: Domain Language
# Follows: "Semantic Failures"
class UserAlreadyExists(Exception):
    pass

def register_user(email):
    try:
        User.objects.create(email=email)
    except IntegrityError:
        # Translate technical error to business error
        raise UserAlreadyExists(f"User with email {email} already exists")
```

#### Database Constraints

```python
# GOOD: Database enforces integrity
# Follows: "Errors should never pass silently"
class Plot(models.Model):
    class Status(models.TextChoices):
        ACTIVE = 'active', 'Active'
        ARCHIVED = 'archived', 'Archived'

    status = models.CharField(max_length=20, choices=Status.choices)
    map_area = models.FloatField()

    class Meta:
        constraints = [
            models.CheckConstraint(
                check=models.Q(map_area__gt=0),
                name='positive_map_area'
            ),
        ]
```

#### Bulk Operations

```python
# BAD: N+1 saves
# Violates: "SQL Efficiency" (Django Database API philosophy)
for item in items:
    obj = MyModel(name=item['name'])
    obj.save()

# GOOD: Single bulk insert
# Follows: "Less code", "SQL Efficiency"
MyModel.objects.bulk_create([
    MyModel(name=item['name']) for item in items
])
```

#### Transactions

```python
# GOOD: Atomic multi-step operation
# Follows: "Errors should never pass silently"
from django.db import transaction

def transfer_ownership(plot, new_owner):
    with transaction.atomic():
        old_owner = plot.owner
        plot.owner = new_owner
        plot.save(update_fields=['owner'])

        OwnershipLog.objects.create(
            plot=plot,
            from_owner=old_owner,
            to_owner=new_owner
        )
```

### Concrete Dos and Don'ts

#### Imports

- Put imports in these groups: future, standard library, third-party libraries, other Django components, local Django component, try/excepts. Sort lines in each group alphabetically by the full module name. Place all import module statements before from module import objects in each section. Use absolute imports for other Django components and relative imports for local components.
- Be especially careful to not leave unused imports or variables lying around in the code. Double check the written code.

#### Python Tooling

- Use uv as package manager and program runner. For temp, you can use uvx.
- User ruff as formatter. We are fine with line width of up to 120.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lixxz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lixxz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
