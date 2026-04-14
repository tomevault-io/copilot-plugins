## github-scout

> Starts Vite dev server with hot reload at http://localhost:5173

# Github Scout Agent Guidelines

## 🔧 Development Commands

### Installation
```bash
npm install
```

### Development Server
```bash
npm run dev
```
Starts Vite dev server with hot reload at http://localhost:5173

### Production Build
```bash
npm run build
```
Creates optimized production build in `dist/` folder

### Preview Production Build
```bash
npm run preview
```
Locally preview the production build

### Type Checking
```bash
npm run type-check
```
Runs vue-tsc for TypeScript type checking

### Linting
```bash
npm run lint
```
Runs both ESLint and oxlint with auto-fix

#### Individual Linters
```bash
npm run lint:eslint   # ESLint only
npm run lint:oxlint   # oxlint only
```

### Formatting
```bash
npm run format
```
Formats source files with oxfmt

### Unit Testing
```bash
npm run test:unit     # Run all unit tests
npm run test:unit -- --run tests/unit/example.test.ts  # Run single test file
npm run test:unit -- -t "test name"  # Run tests matching pattern
```

### End-to-End Testing
```bash
npm run test:e2e              # Run all E2E tests
npm run test:e2e -- tests/example.spec.ts  # Run specific test file
npm run test:e2e -- --project=chromium  # Run on specific browser
npm run test:e2e -- --debug   # Run in debug mode
```

## 📝 Code Style Guidelines

### File Organization
- Components: `src/components/` (PascalCase filenames)
- Views/Pages: `src/views/` 
- Stores: `src/stores/` (Pinia)
- Router: `src/router/`
- Assets: `src/assets/`
- Composables: `src/composables/`
- Types: `src/types/`

### Import Order
1. Vue/Vue Router/Pinia imports
2. Third-party libraries (alphabetical)
3. Local components/views (relative paths)
4. Local composables/utils (relative paths)
5. Styles/assets (relative paths)
6. Type definitions

### Template Guidelines
- Use kebab-case for custom components in templates
- Always close self-closing tags: `<MyComponent />`
- Use v-bind shorthand `:` for attributes
- Use v-on shorthand `@` for events
- Multi-line attributes indented with 2 spaces
- No trailing whitespace

### Script Setup (for beginners - using life metaphors)
Since you're starting with just HTML knowledge, think of Vue components as:

**盒子概念 (Box Concept)** - Instead of `ref()` or `reactive()`:
- A box that holds a value (like a number or text)
- When you put something new in the box, the screen updates automatically
- Example: `<div>{{ count }}</div>` where count is a box holding a number

**開關概念 (Switch Concept)** - Instead of lifecycle hooks:
- Think of components having power switches
- When mounted: switch turns ON (component appears)
- When unmounted: switch turns OFF (component disappears)
- You can perform actions when switches flip

### Small Incremental Development
1. Write ONLY one small piece at a time (max 5-10 lines)
2. Add plain language comments beside code explaining what it does
3. Test frequently after each small change
4. Example:
```vue
<script setup>
// 這是一個計數器的盒子，開始時裡面有0
let count = 0
</script>

<template>
  <!-- 顯示盒子裡面的數字 -->
  <div>你點了 {{ count }} 次</div>
  
  <!-- 按鈕：點一下讓盒子裡的數字加1 -->
  <button @click="count = count + 1">
    點我！
  </button>
</template>
```

### Naming Conventions
- Components: PascalCase (ButtonCard.vue)
- Functions/variables: camelCase (handleClick, userName)
- Constants: UPPER_SNAKE_case (MAX_ITEMS, API_URL)
- Events: kebab-case (update-title, form-submit)
- Props: camelCase in JS, kebab-case in template
- Private methods/variables: prefix with underscore (_privateMethod)

### TypeScript Guidelines
- Prefer interfaces over types for object shapes
- Use type aliases for unions/intersections
- Avoid `any` type; use `unknown` when uncertain
- Generic constraints: `<T extends SomeType>`
- Export types from dedicated `types/` folder

### Error Handling
- Try/catch for async operations
- Display user-friendly error messages in UI
- Log errors to console for development
- Use Pinia stores for global error state when needed
- Validate props with sensible defaults

### Commenting Style
- Use plain language comments (生活化註解)
- Explain WHY, not WHAT
- Place comments beside code, not above blocks
- Use traditional Chinese for comments as requested
- Example: `let count = 0  // 這個盒子開始時裡面是空的`

## 🎯 Vue 3 Specific Guidelines (Beginner Friendly)

### 監聽器概念 (Watcher Concept) 
Instead of `watch()`:
- Think of having a看守員 (guard) that watches a box
- When the box content changes, the guard runs a task
- Good for: responding to changes in values

### 計算屬性概念 (Computed Concept)
Instead of `computed()`:
- Think of having an自動算盤 (auto abacus)
- It automatically calculates when its source boxes change
- Example: 總價格 = 單價 × 數量 (updates automatically when either changes)

### 樣式規範
- Scoped CSS preferred: `<style scoped>`
- Use CSS variables for theme colors
- BEM naming for complex components
- Limit use of !important
- Mobile-first responsive design

## 🔍 Debugging Guidelines
- Use `console.log()` sparingly for development
- Remove debug statements before commits
- Use Vue Devtools for component inspection
- Test edge cases: empty values, null, extreme numbers
- Test in multiple browsers (Chrome, Firefox, Safari)

## 📦 Git Guidelines
- Commit frequently with descriptive messages
- Format: `feat: add user login` or `fix: resolve button alignment`
- Reference issues when applicable: `fix: resolve #123`
- Keep commits small and focused
- Use branches for features/bug fixes

## 🧪 Testing Guidelines
### Unit Tests
- Test one thing per test file
- Arrange-Act-Assert pattern
- Mock external dependencies
- Test both success and failure cases
- Use descriptive test names: `should display error when invalid input`

### E2E Tests
- Test user workflows, not implementation details
- Use data-testid attributes for selectors
- Keep tests independent
- Clean up test state between tests
- Use page objects for complex interactions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simonwanggggggg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
