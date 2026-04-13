## finance-tracker

> This file provides project-specific guidance to Claude Code (claude.ai/code) when working with this finance tracker repository.

# CLAUDE.md

This file provides project-specific guidance to Claude Code (claude.ai/code) when working with this finance tracker repository.

**This file includes:**
- Development commands and architecture overview
- Project preferences and patterns
- Teaching methodology and instruction formats

---

## Development Commands

Start development server:
```bash
npm run dev
```

Build the application:
```bash
npm run build
```

Start production server:
```bash
npm run start
```

Lint the codebase:
```bash
npm run lint
```

Type check the codebase:
```bash
npm run type-check
```

---

## Architecture Overview

This is a Personal Finance Tracker built with Next.js 15 using TypeScript and the App Router architecture.

### Tech Stack
- **Framework**: Next.js 15 with React 19 and TypeScript
- **Styling**: Tailwind CSS
- **Charts**: Recharts for data visualization
- **Icons**: Lucide React
- **Date handling**: date-fns
- **Storage**: LocalStorage (initial implementation)
- **Structure**: Single-page dashboard (will expand to multi-page later)

### Key Files and Structure

- `src/app/layout.tsx` - Root layout with global styles and metadata
- `src/app/page.tsx` - Main dashboard with financial overview
- `src/components/` - React components (AddTransaction, TransactionsSection, StatsCards, etc.)
- `src/features/transactions/` - Transaction types, constants, and utilities
- `src/app/globals.css` - Global styles and Tailwind configuration

### Data Models

Core financial data structures defined in `src/features/transactions/types.ts`:

**Transaction:**
```typescript
{
    id: string;              // Unique identifier
    type: 'expense' | 'income';
    category: string;        // E.g., "Food", "Transport", "Salary"
    amount: string;          // Keep as string for form handling
    description: string;     // Optional details
    date: string;           // ISO date string (YYYY-MM-DD)
}
```

**Categories:**
- **Expense**: Food, Transport, Entertainment, Utilities, Health, Shopping, Education, Other
- **Income**: Salary, Freelance, Investments, Gifts, Other

**Future Models:**
- **Budget**: Monthly spending limits per category
- **Goal**: Savings targets with progress tracking

### Design System

Professional color scheme for financial data:
- **Primary**: Amber-600 for actions and highlights
- **Success**: Emerald-600/700 for income and positive values
- **Danger**: Red-600 for expenses and warnings
- **Neutral**: Stone-50/100/200 for backgrounds and borders
- **Text**: Stone-800 for primary text, Stone-500/600 for secondary

### Current Implementation Status

**✅ Implemented:**
- Transaction CRUD (Create, Read, Update in progress, Delete)
- LocalStorage persistence with data migration safety
- Month/year filtering for transactions and charts
- Financial stats (income, expenses, balance)
- Data visualization (expense breakdown pie chart, monthly trends bar chart)
- Form validation and error handling
- Unique ID generation for transactions

**🚧 In Progress:**
- Edit transaction functionality

**❌ Not Started:**
- Budget tracking feature (placeholder exists)
- Savings goals
- Multi-page routing structure
- Data export (CSV)
- Advanced analytics

### Features Roadmap

1. **Transaction Management** (90% complete)
   - ✅ Add transactions
   - ✅ View/filter by month
   - ✅ Delete with confirmation
   - 🚧 Edit existing transactions
   - ❌ Search/advanced filtering

2. **Budget Tracking** (Not started)
   - Set monthly limits per category
   - Track progress with visual indicators
   - Alerts for overspending
   - Budget vs actual comparisons

3. **Analytics** (Basic charts done)
   - ✅ Expense breakdown by category
   - ✅ Monthly income vs expense trends
   - ❌ Year-over-year comparisons
   - ❌ Spending patterns and insights

4. **Goal Setting** (Not started)
   - Create savings goals
   - Track progress
   - Deadline management
   - Achievement notifications

5. **Data Export** (Not started)
   - CSV export functionality
   - Date range selection
   - Category filtering

6. **Multi-Page Structure** (Not started)
   - Separate routes for Transactions, Budgets, Analytics, Goals
   - Navigation system
   - Shared layouts

---

## Project-Specific Preferences

- **Do what has been asked; nothing more, nothing less**
- Prefer editing existing files over creating new ones
- Don't create documentation files unless explicitly requested
- Keep implementations simple and focused on learning
- Use TypeScript strictly - no implicit any types
- Follow existing code patterns and naming conventions
- Maintain consistent styling with Tailwind CSS utility classes

---

## Data Storage

**LocalStorage Key:** `finance-tracker-transactions`

**Data Migration:**
The app includes safety checks to ensure all transactions have unique IDs, even when loading legacy data without IDs. See `src/app/page.tsx` load effect for implementation.

**Storage Format:**
```json
[
  {
    "id": "1730678730000-0",
    "type": "expense",
    "category": "Food",
    "amount": "25.50",
    "description": "Lunch",
    "date": "2025-11-03"
  }
]
```

---

## Common Patterns Used

### State Management
- useState for local component state
- Props drilling for parent-child communication
- No global state management library (yet)

### Form Handling
- Controlled components with useState
- Inline validation with error state
- Reset form after successful submission

### List Rendering
- Always use unique IDs for keys (not array index)
- Filter and sort before rendering
- Show empty states when no data

### Data Flow
```
User Input → Component State → Parent Handler → Update State → LocalStorage → Re-render
```

---

## Notes for Claude

- User is learning React and TypeScript - explain concepts clearly
- Use the teaching methodology defined below
- Follow step-by-step format when implementing features
- Verify work before moving to next step
- Focus on "why" in addition to "what"
- Use concrete examples with real data in explanations
- **CRITICAL: NEVER write code directly into files. ALWAYS guide user to write the code themselves using TODO(human) format:**
  - Show them WHAT to write
  - Show them WHERE to write it
  - Explain WHY it works
  - User types the code themselves
  - Verify their work before moving on
- This applies to ALL code changes - components, functions, logic, styling, everything
- The user learns by DOING, not by watching Claude write code

---

# Teaching Methodology

This section provides teaching methodology guidance for Claude Code when working with this codebase.

## Core Principle
**Start simple. Add complexity only when needed.**

## How to Learn with Claude

### 1. Ask Direct Questions
- "What is X?"
- "How does Y work?"
- "Explain Z like I'm a beginner"

### 2. Request Examples
- "Show me a simple example"
- "Can you demonstrate this?"
- "What would this look like in practice?"

### 3. Build Understanding Step-by-Step
- Start with basics
- Test your understanding
- Ask follow-up questions
- Apply what you learned

## Effective Prompting Patterns

### For Learning Concepts
```
"Explain [topic] in simple terms"
"What are the key points about [subject]?"
"Break down [complex idea] into steps"
```

### For Problem-Solving
```
"Help me understand this problem first"
"What's the simplest solution?"
"Why does this approach work?"
```

### For Code and Technical Topics
```
"Write simple, readable code for [task]"
"Explain what each line does"
"Show me the minimal working example"
```

## Anti-Patterns to Avoid

❌ **Don't ask for "the best" or "most optimal" solution first**
✅ Ask for a working solution, then improve

❌ **Don't request everything at once**
✅ Break complex tasks into smaller parts

❌ **Don't skip understanding the basics**
✅ Ensure you grasp fundamentals before advancing

## Learning Workflow

1. **Explore** - Ask what something is
2. **Understand** - Ask how it works
3. **Practice** - Try simple examples
4. **Apply** - Use in your context
5. **Iterate** - Refine and expand

## Quick Tips

- If confused, ask Claude to simplify
- Request analogies for complex topics
- Ask "why" to deepen understanding
- Test concepts with small experiments
- Focus on one thing at a time

## Example Conversation Starters

**For beginners:**
- "I'm new to [topic]. Where should I start?"
- "Can you explain [concept] without jargon?"

**For debugging:**
- "Help me understand what's happening here"
- "Let's trace through this step by step"

**For learning:**
- "What's the intuition behind [concept]?"
- "Can you give me a real-world analogy?"

## Remember

- Learning is iterative
- Questions are good
- Start small, grow gradually
- Understanding > memorization
- Mistakes are learning opportunities

---

## Step-by-Step Instruction Format

**CRITICAL: Keep steps SMALL - one small change at a time. Don't overwhelm with large code blocks.**

### Template Structure

#### Step X: [Brief Title]

**📍 WHERE:** Brief description of the location (e.g., "In `ChartsPlaceholder.tsx`, after the `availableYears` line")

**✏️ CODE:**
```typescript
// Just the code to add or change - keep it small!
const example = "code here";
```

**📝 EXPLANATION:**

For multi-line code:
- Explain each significant line individually
- Format: **Line: `code snippet`** - explanation
- Use simple language without jargon

Then provide overall context:
- **Why we need this:** Explain the purpose
- **Before vs After:** Show what changes in behavior

**Try making this change yourself, then type "next" when you're ready to continue!**

---

### Verification Pattern (After User Types "next")

1. **✅ Read the file** - Check what they actually wrote
2. **✅ Compare** - Match against expected result
3. **✅ Give feedback** - Correct or guide to fix
4. **✅ Only then proceed** - Move to next step when verified

#### Example Verification:

```markdown
Let me check if you made the change correctly:

<Read file at specific lines>

**Expected to see:**
```language
[correct code]
```

**Actual result:**
- ✅ Line X: Correct! You added [...]
- ✅ Line Y: Perfect! You changed [...]
- ⚠️  Line Z: Small issue - should be [...] not [...]

**Feedback:**
"Almost perfect! Just need to adjust line Z from [...] to [...]"

[If all correct]
"Perfect! ✅ Excellent work! You've successfully [accomplishment]"

Now let's move to the next step...
```

---

### Full Step Template Example

#### Step 1: Add Year State

**📍 WHERE:** In `ChartsPlaceholder.tsx`, after the `const now = new Date();` line

**✏️ CODE:**
```typescript
const [selectedYear, setSelectedYear] = useState(now.getFullYear());
```

**📝 EXPLANATION:**

**Line: `const [selectedYear, setSelectedYear] = useState(now.getFullYear());`**
- Creates state to track which year is selected
- `selectedYear` stores the current value
- `setSelectedYear` is the function to update it
- `now.getFullYear()` sets initial value to current year (like 2025)

**Why we need this:**
- Tracks user's year selection for filtering chart data
- **Before:** No way to remember which year user selected
- **After:** Component remembers and can filter by selected year

**Try making this change yourself, then type "next" when you're ready!**

---

## "Explain This" - Concrete Example Format

When the user says **"explain this"** or asks you to explain code/concepts, ALWAYS use the **Concrete Example Format** with real data.

### Required Format Elements

#### 1. Replace Abstract Values with Real Data
- ❌ Don't say: "selectedDate is a Date object"
- ✅ Do say: "selectedDate = new Date(2025, 0, 15) // January 15, 2025"

#### 2. Show Before/After States
```
BEFORE:
selectedDate = January 15, 2025
selectedMonth = 0

USER ACTION: Clicks "March"

AFTER:
selectedDate = March 15, 2025
selectedMonth = 2
```

#### 3. Step-by-Step with Actual Values
```typescript
// Step 1: User clicks "Previous Year" button
const d = new Date(selectedDate);
// d = new Date(2024, 2, 15)  // March 15, 2024

// Step 2: Subtract 1 from year
d.setFullYear(selectedDate.getFullYear() - 1);
// 2024 - 1 = 2023
// d is now: March 15, 2023
```

#### 4. Use Visual Diagrams
```
┌─────────────────────────┐
│ Component A             │
│ selectedDate: Jan 2025  │
└───────┬─────────────────┘
        │
        ▼
┌─────────────────────────┐
│ Component B             │
│ Receives: Jan 2025      │
└─────────────────────────┘
```

#### 5. Multiple Scenarios
- Show what happens in different cases
- Example: "When viewing 2024..." vs "When viewing 2025..."
- Example: "If array is empty..." vs "If array has data..."

#### 6. Evaluation of Conditions
```typescript
disabled = {availableYears.length > 0 && selectedMonthYear <= availableYears[0]}
// With real data:
disabled = {3 > 0 && 2023 <= 2023}
disabled = {true && true}
disabled = true  // Button is DISABLED
```

### Example Response to "Explain This"

User asks: "explain this: `const selectedMonth = selectedDate.getMonth()`"

Response should include:
```typescript
const selectedMonth = selectedDate.getMonth()

// With real data:
selectedDate = new Date(2025, 2, 15)  // March 15, 2025
selectedMonth = selectedDate.getMonth()  // Returns 2

// Why 2? Months are 0-indexed:
// 0 = January
// 1 = February
// 2 = March  ← We're here
// 3 = April
// ...
// 11 = December

// Different example:
selectedDate = new Date(2024, 11, 25)  // December 25, 2024
selectedMonth = 11  // December
```

### Key Principles

- Always use dates that make sense (e.g., January 15, 2025, not just "a date")
- Show the actual numbers/strings that flow through the code
- Walk through conditionals with true/false evaluations
- Use emojis sparingly (✓, ✗, →, ▼) to show state
- Include "★ Insight" boxes to highlight important concepts

**CRITICAL:** The user learns best by SEEING data flow through code, not by reading abstract descriptions.

---

## General Instruction Reminders

- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (*.md) or README files unless explicitly requested

---

*Focus on understanding, not perfection. Keep it simple, keep learning.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jercydy29)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jercydy29)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
