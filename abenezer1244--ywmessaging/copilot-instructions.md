## ywmessaging

> ALERT: THIS IS NOT A MOCK OR TEST OR DUMMY PROJECT. IT IS A REAL WORLD ENTERPRISE LEVEL SAAS SO NEVER ADD A MOCK OR TEST OR DUMMY CODE.

ALERT: THIS IS NOT A MOCK OR TEST OR DUMMY PROJECT. IT IS A REAL WORLD ENTERPRISE LEVEL SAAS SO NEVER ADD A MOCK OR TEST OR DUMMY CODE.

When read a files and make sure to read the whole lines chunk by chunk so you wouldn't not miss anything because of any reason.

1. First think through the problem, read the codebase for relevant files, and write a plan to tasks/todo.md.
2. The plan should have a list of todo items that you can check off as you complete them
3. Before you begin working, check in with me and I will verify the plan.
4. Then, begin working on the todo items, marking them as complete as you go.
5. Please every step of the way just give me a high level explanation of what changes you made
6. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.
7. Finally, add a review section to the todo.md file with a summary of the changes you made and any other relevant information.
8. DO NOT BE LAZY. NEVER BE LAZY. IF THERE IS A BUG FIND THE ROOT CAUSE AND FIX IT. NO TEMPORARY FIXES. YOU ARE A SENIOR DEVELOPER. NEVER BE LAZY
9. MAKE ALL FIXES AND CODE CHANGES AS SIMPLE AS HUMANLY POSSIBLE. THEY SHOULD ONLY IMPACT NECESSARY CODE RELEVANT TO THE TASK AND NOTHING ELSE. ASK IS THE WORK IS ENTERPRISE OR A HOBBY SO EVERY FIX WILL HAVE LEVEL OF FIX. YOUR GOAL IS TO NOT INTRODUCE ANY BUGS. IT'S ALL ABOUT SIMPLICITY

CRITICAL: When debugging, you MUST trace through the ENTIRE code flow step by step. No assumptions. No shortcuts.

## ⚠️ DATABASE SAFETY RULES (CRITICAL)

**NEVER** run these commands on production:
- `npx prisma db push --accept-data-loss` ❌ - CAN DROP TABLES AND DELETE ALL DATA
- `npx prisma migrate reset` ❌ - DELETES ALL DATA

**ALWAYS** do this before schema changes:
1. Ask user to confirm they have backups enabled on Render
2. Use `npx prisma db push` WITHOUT `--accept-data-loss`
3. If Prisma warns about data loss, STOP and ask user

**For tenant schema:** NEVER run `prisma db push` with tenant-schema.prisma on the main DATABASE_URL - it will drop registry tables (Church, Tenant, AdminEmailIndex).



```

**ErrorBoundary Features**:

- Automatic fallback UI with retry/refresh options
- Development mode error details
- Custom error handling callbacks
- Integration with NextUI design system

**ErrorFallback Variants**:

- `default` - Full error display with actions
- `minimal` - Compact inline error message  
- `minimal` - Compact inline error message
- `detailed` - Includes error stack trace in development

## Visual Development & Testing

### Design System

The project follows S-Tier SaaS design standards inspired by Stripe, Airbnb, and Linear. All UI development must adhere to:

- **Design Principles**: `/context/design-principles.md` - Comprehensive checklist for world-class UI
- **Component Library**: NextUI with custom Tailwind configuration

### Quick Visual Check

**IMMEDIATELY after implementing any front-end change:**

1. **Identify what changed** - Review the modified components/pages
2. **Navigate to affected pages** - Use `mcp__playwright__browser_navigate` to visit each changed view
3. **Verify design compliance** - Compare against `/context/design-principles.md`
4. **Validate feature implementation** - Ensure the change fulfills the user's specific request
5. **Check acceptance criteria** - Review any provided context files or requirements
6. **Capture evidence** - Take full page screenshot at desktop viewport (1440px) of each changed view
7. **Check for errors** - Run `mcp__playwright__browser_console_messages` ⚠️

This verification ensures changes meet design standards and user requirements.

### Comprehensive Design Review

For significant UI changes or before merging PRs, use the design review agent:

```bash
# Option 1: Use the slash command
/design-review

# Option 2: Invoke the agent directly
@agent-design-review
```

The design review agent will:

- Test all interactive states and user flows
- Verify responsiveness (desktop/tablet/mobile)
- Check accessibility (WCAG 2.1 AA compliance)
- Validate visual polish and consistency
- Test edge cases and error states
- Provide categorized feedback (Blockers/High/Medium/Nitpicks)

### Playwright MCP Integration

#### Essential Commands for UI Testing

```javascript
// Navigation & Screenshots
mcp__playwright__browser_navigate(url); // Navigate to page
mcp__playwright__browser_take_screenshot(); // Capture visual evidence
mcp__playwright__browser_resize(
  width,
  height
); // Test responsiveness

// Interaction Testing
mcp__playwright__browser_click(element); // Test clicks
mcp__playwright__browser_type(
  element,
  text
); // Test input
mcp__playwright__browser_hover(element); // Test hover states

// Validation
mcp__playwright__browser_console_messages(); // Check for errors
mcp__playwright__browser_snapshot(); // Accessibility check
mcp__playwright__browser_wait_for(
  text / element
); // Ensure loading
```

### Design Compliance Checklist

When implementing UI features, verify:

- [ ] **Visual Hierarchy**: Clear focus flow, appropriate spacing
- [ ] **Consistency**: Uses design tokens, follows patterns
- [ ] **Responsiveness**: Works on mobile (375px), tablet (768px), desktop (1440px)
- [ ] **Accessibility**: Keyboard navigable, proper contrast, semantic HTML
- [ ] **Performance**: Fast load times, smooth animations (150-300ms)
- [ ] **Error Handling**: Clear error states, helpful messages
- [ ] **Polish**: Micro-interactions, loading states, empty states

## When to Use Automated Visual Testing

### Use Quick Visual Check for:

- Every front-end change, no matter how small
- After implementing new components or features
- When modifying existing UI elements
- After fixing visual bugs
- Before committing UI changes

### Use Comprehensive Design Review for:

- Major feature implementations
- Before creating pull requests with UI changes
- When refactoring component architecture
- After significant design system updates
- When accessibility compliance is critical

### Skip Visual Testing for:

- Backend-only changes (API, database)
- Configuration file updates
- Documentation changes
- Test file modifications
- Non-visual utility functions

## Environment Setup

Requires these environment variables:
@@ -164,9 +277,8 @@ Requires these environment variables:
- Cloudinary configuration
- Email service credentials (Resend)

[byterover-mcp]

# Specialized Agents for Team Roles

Your project has 8 specialized agents configured to handle specific jobs. Invoke them using slash commands or direct agent references.

## Available Agents

### 1. 📊 Product Manager Agent
**Purpose**: Product strategy, requirements analysis, feature prioritization, and roadmap planning
**Invocation**: `/product-manager` or `@agent-product-manager`
**When to use**:
- Defining new features and user stories
- Analyzing market opportunities
- Prioritizing product work using RICE scoring
- Creating product specifications and roadmaps
**Config**: `/.claude/agents/product-manager-agent.md`

### 2. 🎨 UI/UX Agent
**Purpose**: UI/UX design review, component design, accessibility audit, and design system consistency
**Invocation**: `/ui-ux` or `@agent-ui-ux`
**When to use**:
- Reviewing new UI implementations
- Auditing accessibility compliance (WCAG 2.1)
- Testing responsive design
- Ensuring design system consistency
**Config**: `/.claude/agents/ui-ux-agent.md`

### 3. 🏗️ System Architecture Agent
**Purpose**: System design, scalability analysis, architecture patterns, and technology selection
**Invocation**: `/system-architecture` or `@agent-system-architecture`
**When to use**:
- Designing new systems for scale
- Evaluating technology choices
- Analyzing scalability concerns
- Creating architecture decision records
**Config**: `/.claude/agents/system-architecture-agent.md`

### 4. 💻 Senior Frontend Engineer Agent
**Purpose**: Frontend code review, component architecture, performance optimization, and testing
**Invocation**: `/senior-frontend` or `@agent-senior-frontend`
**When to use**:
- Code reviews for React components
- Performance optimization and profiling
- Component architecture assessment
- Testing strategy and coverage analysis
**Config**: `/.claude/agents/senior-frontend-engineer-agent.md`

### 5. 🔧 Backend Engineer Agent
**Purpose**: Backend code review, API design, database optimization, and business logic validation
**Invocation**: `/backend-engineer` or `@agent-backend-engineer`
**When to use**:
- API design and contract review
- Database query optimization
- Performance bottleneck analysis
- Business logic validation
- Security audit for backend
**Config**: `/.claude/agents/backend-engineer-agent.md`

### 6. ✅ QA Testing Agent
**Purpose**: Test planning, test case creation, bug reporting, and quality assurance
**Invocation**: `/qa-testing` or `@agent-qa-testing`
**When to use**:
- Creating comprehensive test plans
- Designing test cases with edge cases
- Planning test automation
- Creating clear bug reports
- Assessing feature quality
**Config**: `/.claude/agents/qa-testing-agent.md`

### 7. 🚀 DevOps Configuration Agent
**Purpose**: DevOps and infrastructure configuration, CI/CD pipeline design, deployment strategies, and monitoring setup
**Invocation**: `/devops` or `@agent-devops`
**When to use**:
- Designing deployment pipelines
- Planning scaling strategies
- Configuring monitoring and alerting
- Creating disaster recovery plans
- Infrastructure optimization
**Config**: `/.claude/agents/devops-agent.md`

### 8. 🔒 Security Analyst Agent
**Purpose**: Security analysis, vulnerability assessment, threat modeling, and compliance audit
**Invocation**: `/security` or `@agent-security-analyst`
**When to use**:
- Security code review and vulnerability assessment
- Threat modeling and analysis
- OWASP Top 10 compliance check
- Compliance audit (GDPR, HIPAA, SOC 2)
- Security hardening recommendations
**Config**: `/.claude/agents/security-analyst-agent.md`

---

## Quick Examples

**Product Planning:**
```
/product-manager

We need to plan the 10DLC delivery tier feature. What are the key user stories and MVP components?
```

**UI/UX Review:**
```
/ui-ux

Review the AdminSettingsPage for accessibility issues and design consistency.
```

**Architecture Decision:**
```
/system-architecture

How should we design our messaging queue system to handle 10x growth?
```

**Code Review:**
```
/backend-engineer

Review the SMS sending service for performance bottlenecks and database optimization.
```

**Testing Plan:**
```
/qa-testing

Create a test plan for the authentication system including edge cases.
```

**Deployment:**
```
/devops

Design a CI/CD pipeline for our Render deployment with monitoring.
```

**Security Audit:**
```
/security

Conduct a security audit of the API authentication and authorization system.
```

---

## Important

- Always use byterover-retrive-knowledge tool to get related context before tasks
- Always use byterover-store-knowledge to store critical information after successful tasks
- Design review agent configuration: `/.claude/agents/design-review-agent.md`
- Design principles checklist: `/context/design-principles.md`
- Custom slash commands: `/context/design-review-slash-command.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abenezer1244) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
