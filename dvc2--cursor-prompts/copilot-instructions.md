## commonsense

> description: Core development best practices and common sense patterns

---
description: Core development best practices and common sense patterns
globs: ["**/*"]
alwaysApply: true
---

# Common Sense Development Rules

## 🎯 Code Quality First

### Simple > Complex
```javascript
// ✅ GOOD: Clear intent
const isActive = user.status === 'active';

// ❌ BAD: Unnecessary complexity
const isActive = (() => user?.status?.toLowerCase() === 'active')();
```

### Early Returns
```javascript
// ✅ GOOD: Fail fast
function processUser(user) {
  if (!user) return null;
  if (!user.email) return null;
  
  // Main logic here
}

// ❌ BAD: Nested hell
function processUser(user) {
  if (user) {
    if (user.email) {
      // Main logic here
    }
  }
}
```

## 🛡️ Error Handling

### Always Handle Errors
```javascript
// ✅ GOOD: Explicit handling
try {
  const data = await fetchData();
  return processData(data);
} catch (error) {
  console.error('Failed to fetch data:', error);
  return defaultData;
}

// ❌ BAD: Silent failures
const data = await fetchData();
return processData(data);
```

### Meaningful Error Messages
```javascript
// ✅ GOOD: Context included
throw new Error(`User ${userId} not found in database`);

// ❌ BAD: Generic
throw new Error('Error');
```

## 📝 Naming Conventions

### Be Descriptive
```javascript
// ✅ GOOD
const userAuthenticationToken = generateToken();
const isEmailValid = validateEmail(email);

// ❌ BAD
const token = genTok();
const valid = check(e);
```

### Boolean Naming
```javascript
// ✅ GOOD: Clear boolean names
const isLoading = true;
const hasPermission = false;
const canEdit = true;

// ❌ BAD: Ambiguous
const loading = true;
const permission = false;
const edit = true;
```

## 🔄 Function Design

### Single Responsibility
```javascript
// ✅ GOOD: One clear purpose
function calculateTax(amount, rate) {
  return amount * rate;
}

function formatCurrency(amount) {
  return `$${amount.toFixed(2)}`;
}

// ❌ BAD: Multiple responsibilities
function calculateAndFormat(amount, rate) {
  const tax = amount * rate;
  return `$${tax.toFixed(2)}`;
}
```

### Consistent Return Types
```javascript
// ✅ GOOD: Always returns same type
function findUser(id) {
  const user = users.find(u => u.id === id);
  return user || null;
}
// ❌ BAD: Mixed returns
function findUser(id) {
  const user = users.find(u => u.id === id);
  if (!user) return 'Not found';
  return user;
}
```

## 🚀 Performance Patterns

### Avoid Premature Optimization
```javascript
// Only optimize when you have metrics showing a problem
// Profile first, optimize second
```

### Use Appropriate Data Structures
```javascript
// ✅ GOOD: O(1) lookup
const userMap = new Map(users.map(u => [u.id, u]));
const user = userMap.get(userId);

// ❌ BAD: O(n) lookup for frequent operations
const user = users.find(u => u.id === userId);
```

## 💾 State Management

### Immutability
```javascript
// ✅ GOOD: Create new objects
const updatedUser = { ...user, name: newName };
const newItems = [...items, newItem];

// ❌ BAD: Mutate directly
user.name = newName;
items.push(newItem);
```

### Clear State Updates
```javascript
// ✅ GOOD: Explicit state changes
setState(prevState => ({
  ...prevState,
  isLoading: false,
  data: newData
}));

// ❌ BAD: Unclear mutations
state.isLoading = false;
state.data = newData;
```

## 🧪 Testing Mindset

### Write Testable Code
```javascript
// ✅ GOOD: Pure, testable
function calculateDiscount(price, percentage) {
  return price * (1 - percentage / 100);
}

// ❌ BAD: Hard to test
function calculateDiscount() {
  const price = document.getElementById('price').value;
  const percentage = globalDiscount;
  return price * (1 - percentage / 100);
}
```

## 📚 Documentation

### Document Why, Not What
```javascript
// ✅ GOOD: Explains reasoning
// Use exponential backoff to avoid overwhelming the server
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000);

// ❌ BAD: States obvious
// Multiply 1000 by 2 to the power of retryCount
const delay = 1000 * Math.pow(2, retryCount);
```

## 🔐 Security Basics

### Never Trust User Input
```javascript
// ✅ GOOD: Validate and sanitize
const userId = parseInt(req.params.id, 10);
if (isNaN(userId) || userId <= 0) {
  return res.status(400).json({ error: 'Invalid user ID' });
}

// ❌ BAD: Direct usage
const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
```

## 🎭 Code Style

### Consistency Over Personal Preference
- Follow project conventions
- Use formatters (Prettier, ESLint)
- Don't mix styles in same file

### Remove Dead Code
- Delete commented code
- Remove unused imports
- Clean up console.logs

## ⚡ Quick Decisions

When unsure:
1. Choose readability
2. Choose maintainability
3. Choose simplicity
4. Choose explicitness

Remember: Code is read 10x more than written. Optimize for the reader!

---
> Source: [DVC2/cursor_prompts](https://github.com/DVC2/cursor_prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
