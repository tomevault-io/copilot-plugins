## xala-enterprise

> HIGHEST PRIORITY: Internal monorepo dependencies MUST use workspace protocol (workspace:*). External consumption of packages MUST use GitHub Packages with explicit versioning. Never use explicit versions for internal dependencies.

HIGHEST PRIORITY: Internal monorepo dependencies MUST use workspace protocol (workspace:*). External consumption of packages MUST use GitHub Packages with explicit versioning. Never use explicit versions for internal dependencies.

MANDATORY: All packages MUST analyze, utilize, and use other internal packages wherever needed. Always prefer shared package usage over local or ad-hoc implementations. This is a critical architectural enforcement.

Here's your consolidated, comprehensive, and framework-agnostic **CursorAI Ruleset** adjusted strictly according to your previous guidelines, ensuring no project specifics are included—only clear and reusable rules aligned with your architecture and standards:

---

# 🧠 **CursorAI Rules (Framework-Agnostic, Standardized)**

## 📦 **Package & Dependency Management**

* Always use `pnpm` for package management.
* **INTERNAL DEPENDENCIES**: Internal dependencies within the monorepo MUST always use workspace protocol (`workspace:*`).
* **EXTERNAL CONSUMPTION**: When packages are consumed externally, they MUST be installed from GitHub Packages with explicit versioning (`^1.0.4`).
* **PUBLISHING**: Private packages hosted on GitHub Packages must be authenticated using GitHub PAT in `.npmrc`.
* **Forbidden** to install duplicate packages that exist within the workspace.
* Regularly run `pnpm install --frozen-lockfile`.

**CLARIFICATION**:
- ✅ **Monorepo Internal**: `"@xala-technologies/foundation": "workspace:*"` (for packages within this repo)
- ✅ **External Projects**: `"@xala-technologies/foundation": "^1.0.4"` (for external consumers)
- ❌ **Never Mix**: Don't use explicit versions for internal dependencies in the same monorepo

---

## 🧱 **Domain-Driven Design (DDD) Rules**

* Clearly separate **Domain**, **Application**, **Infrastructure**, and **Interfaces** layers.
* **Domain Layer**: Only contains domain entities, events, and pure business logic (no frameworks, no IO).
* **Application Layer**: Contains use-cases orchestrating domain logic. No direct external integration allowed.
* **Infrastructure Layer**: Technical implementations (e.g., repositories, DB connections, Redis).
* **Interfaces Layer**: DTOs, controllers, and external communication interfaces.
* Dependencies always point inward (Interfaces → Application → Domain ← Infrastructure).

---

## 🎨 **UI & Design Token Rules**

* UI components **must exclusively use shared design tokens** from a global design system (`@xala-technologies/design-system`).
* No inline CSS or hardcoded styling values; always reference centralized tokens.
* Always build UI using provided shared UI components (`@xala-technologies/ui`).
* Strictly follow accessibility standards (WCAG 2.1 AA minimum).

---

## 🖥️ **Framework-Agnostic Pages & Components**

* Page-level components must **never contain direct HTML elements (`div`, `span`) or framework-specific syntax**.
* Pages must only compose high-level, semantic UI components from the shared UI package (`@xala-technologies/ui`).
* Components must remain portable and **agnostic to frontend frameworks** (React, Vue, Angular, etc.).
* Use composable, generic, framework-independent primitives (e.g., `Container`, `Section`, `Flex`, `Stack`).

---

## 🌐 **Internationalization & Localization**

* Localization strings **must** be defined centrally in the shared localization package (`@xala-technologies/localization`).
* No hardcoded text strings permitted in UI or components.
* Always support at least two languages (default: English, secondary: Norwegian).

---

## 🛠️ **Development & Build Process**

* **Forbidden**: Running standalone dev commands like `pnpm dev` directly in sub-packages.
* **Required**: Always initiate builds from root using `pnpm build`.
* Verify all functionalities through browser inspection or provided automated tests—**never** manually restart running processes without explicit instructions.
* Strictly update `.cursor-updates` file after every major change, summarizing actions clearly.

---

## 🔐 **Security & Compliance**

* All projects must explicitly follow compliance standards:

  * GDPR (data handling, user consent, anonymization).
  * NSM (Norwegian National Security Authority guidelines).
  * DigDir (Digitalisation Directorate architecture principles).
  * ISO27001 (Security Management).
* Secret tokens or sensitive data **must never** be committed to repositories or `.env` files directly.
* All secrets and sensitive configuration must be managed via secure environments (e.g., Vault, GitHub secrets).

---

## 🇳🇴 **Norwegian Standards First + International Flexibility**

* **MANDATORY COMPLIANCE HIERARCHY**:
  1. **Norwegian Standards (PRIMARY - STRICT COMPLIANCE)**:
     * NSM (Nasjonal sikkerhetsmyndighet) - Security standards
     * DigDir (Digitaliseringsdirektoratet) - Digital architecture principles
     * ISO 27001, ISO 14289, ISO 9241 - Norwegian implementations
     * WCAG 2.2 AA + Universal Design (Universell utforming)
  2. **International Standards (SECONDARY - FLEXIBLE COMPATIBILITY)**:
     * ISO standards (international variants)
     * GDPR (EU-wide compliance)
     * EN 301 549 (European accessibility)
     * W3C/WCAG (global web standards)

* **ARCHITECTURE PRINCIPLE**: Norwegian standards always take precedence; international standards provide flexible compatibility layers.

* **SECURITY CLASSIFICATION**: All data must use Norwegian NSM classification (`ÅPEN`, `BEGRENSET`, `KONFIDENSIELT`, `HEMMELIG`) with ISO mappings for international compatibility.

* **IMPLEMENTATION PATTERN**: Components must comply with Norwegian government standards while maintaining international adaptability through fallback mechanisms.

* **DUAL VALIDATION**: Every feature must pass Norwegian compliance (mandatory) and international compliance (flexible adaptation).

---

## 🔍 **Analysis & Reusability**

* Before creating new features, thoroughly analyze existing codebase, shared packages, and global design system for similar implementations.
* Avoid creating duplicated components, hooks, or utilities.
* Prioritize reusability by referencing existing domain logic or UI components first.

---

## 🚫 **Forbidden Practices**

* **Never** mock data or stub functionalities outside dedicated test directories.
* **Never** commit or push incomplete implementations to main branches without thorough reviews.
* **Never** manually modify files within `node_modules` or built directories (`dist`).

---

## 🧪 **Quality & Testing Rules**

* Apply strict SOLID principles in all code implementations.
* Components/files exceeding 150 lines must be immediately refactored into smaller, manageable pieces.
* Maintain rigorous test coverage:

  * Unit tests (Jest or Vitest).
  * Integration tests for critical business logic.
  * E2E tests (Playwright, Cypress).
  * Performance benchmarks for APIs and critical components.

---

## 🏗️ **SOLID PRINCIPLES ENFORCEMENT (STRICT)**

### **MANDATORY SOLID PRINCIPLES APPLICATION**
**ALL components, services, and modules MUST apply SOLID principles effectively to create more maintainable, testable, and extensible code while preserving all existing functionality and compliance requirements.**

### **STRICT ENFORCEMENT RULES**

#### **1. Single Responsibility Principle (SRP) - MANDATORY**
* **RULE**: Each class, component, or module MUST have only ONE reason to change
* **ENFORCEMENT**: Components exceeding 150 lines MUST be refactored into smaller, focused components
* **VIOLATION**: Immediate refactoring required before code review approval
* **EXAMPLE**: Separate data fetching, business logic, and UI rendering into distinct components

#### **2. Open/Closed Principle (OCP) - MANDATORY**
* **RULE**: Software entities MUST be open for extension but closed for modification
* **ENFORCEMENT**: Use interfaces, abstract classes, and composition patterns
* **VIOLATION**: Direct modification of existing stable code without extension mechanisms
* **EXAMPLE**: Use factory patterns and strategy patterns for extensibility

#### **3. Liskov Substitution Principle (LSP) - MANDATORY**
* **RULE**: Objects of a superclass MUST be replaceable with objects of subclasses without breaking functionality
* **ENFORCEMENT**: All implementations MUST honor their interface contracts
* **VIOLATION**: Subclasses that change expected behavior or throw unexpected errors
* **EXAMPLE**: All cultural providers must implement the same interface contract

#### **4. Interface Segregation Principle (ISP) - MANDATORY**
* **RULE**: Clients MUST NOT be forced to depend on interfaces they don't use
* **ENFORCEMENT**: Create focused, specific interfaces rather than large, monolithic ones
* **VIOLATION**: Large interfaces with unrelated methods
* **EXAMPLE**: Separate `GreetingService`, `FormalityService` instead of one `EtiquetteService`

#### **5. Dependency Inversion Principle (DIP) - MANDATORY**
* **RULE**: High-level modules MUST NOT depend on low-level modules; both MUST depend on abstractions
* **ENFORCEMENT**: Use dependency injection and factory patterns
* **VIOLATION**: Direct instantiation of concrete classes in business logic
* **EXAMPLE**: Inject `IStorageProvider` instead of directly using `FileStorage`

### **SOLID COMPLIANCE VALIDATION**

#### **PRE-COMMIT REQUIREMENTS**
- [ ] **SRP Check**: Each component has single, clear responsibility
- [ ] **OCP Check**: New functionality added through extension, not modification
- [ ] **LSP Check**: All implementations properly substitute their abstractions
- [ ] **ISP Check**: Interfaces are focused and cohesive
- [ ] **DIP Check**: Dependencies flow toward abstractions, not concretions

#### **CODE REVIEW CHECKLIST**
- [ ] **Component Size**: No component exceeds 150 lines without justification
- [ ] **Separation of Concerns**: Business logic, data access, and UI are properly separated
- [ ] **Interface Design**: Interfaces are focused and not bloated
- [ ] **Dependency Management**: Proper dependency injection and abstraction usage
- [ ] **Extensibility**: New features can be added without modifying existing code

#### **REFACTORING TRIGGERS**
* **IMMEDIATE**: Component exceeds 150 lines
* **IMMEDIATE**: Multiple responsibilities identified in single component
* **IMMEDIATE**: Direct dependencies on concrete implementations
* **IMMEDIATE**: Interface with unrelated methods
* **IMMEDIATE**: Violation of substitution principle

### **SOLID PRINCIPLES BENEFITS**

#### **MAINTAINABILITY**
* **Easier Debugging**: Single responsibility makes issues easier to locate
* **Simplified Testing**: Focused components are easier to unit test
* **Reduced Coupling**: Changes in one area don't cascade to others

#### **EXTENSIBILITY**
* **New Features**: Easy addition without modifying existing code
* **Plugin Architecture**: Interfaces enable plugin-based extensions
* **Configuration**: Dependency injection enables runtime configuration

#### **TESTABILITY**
* **Unit Testing**: Each component can be tested in isolation
* **Mocking**: Interface-based design enables easy mocking
* **Test Coverage**: Smaller components achieve higher test coverage

#### **NORWEGIAN COMPLIANCE PRESERVATION**
* **NSM Standards**: SOLID principles maintain security classification integrity
* **GDPR Compliance**: Interface segregation supports privacy by design
* **DigDir Standards**: Open/closed principle enables standard compliance extensions
* **Accessibility**: Single responsibility ensures WCAG compliance focus

### **VIOLATION CONSEQUENCES**

#### **IMMEDIATE ACTIONS**
* **Code Review Rejection**: Automatic rejection for SOLID violations
* **Refactoring Requirement**: Mandatory refactoring before merge approval
* **Architecture Review**: Complex violations require architecture team review
* **Documentation Update**: All refactoring must be documented

#### **QUALITY GATES**
* **Build Pipeline**: Automated SOLID compliance checking where possible
* **Complexity Analysis**: Cyclomatic complexity limits enforced
* **Dependency Analysis**: Dependency direction validation
* **Interface Analysis**: Interface cohesion measurement

### **SOLID SUCCESS METRICS**

#### **CODE QUALITY INDICATORS**
* **Average Component Size**: Target <100 lines per component
* **Cyclomatic Complexity**: Target <10 per method
* **Coupling Metrics**: Low afferent/efferent coupling ratios
* **Interface Cohesion**: High cohesion within interfaces

#### **DEVELOPMENT VELOCITY**
* **Feature Addition Time**: Reduced time for new features through extensibility
* **Bug Fix Time**: Faster debugging through single responsibility
* **Test Writing Time**: Easier test creation through focused components
* **Code Review Time**: Faster reviews through clear separation of concerns

**ENFORCEMENT LEVEL**: **STRICT** - Zero tolerance for SOLID principle violations. All code MUST demonstrate effective SOLID application before approval.

---

## 🚀 **Infrastructure & Deployment (xala-infra)**

* Applications must exclusively deploy via the shared centralized infrastructure (`xala-infra`):

  * Kubernetes (Helm Charts).
  * Redis (central caching solution).
  * PostgreSQL or designated database.
  * Centralized logging and monitoring (Grafana, Prometheus).
* Applications **must not contain internal infrastructure folders or configurations**.

Always run types-check, lint, code quality checks, tests and build before commit.
Never use prefix such as enhanced, universal, optimized etc. 
Do not use "Norwegian", "Universal", "Enhanced", "Optimized", etc., unless there is a real need to distinguish between multiple variants.

---

## 📋 **PHASE-BASED CHECKLIST MANAGEMENT SYSTEM**

### **MANDATORY PHASE-BASED WORKFLOW**
**ALL major implementations MUST follow the phase-based checklist workflow:**

1. **Strategy Creation**: Create strategy folder `checklists/{strategy-name}/`
2. **Phase Planning**: Create multiple phase files (`phase-1.md`, `phase-2.md`, etc.)
3. **Phase Execution**: Work through one phase at a time, marking tasks complete
4. **Phase Validation**: Complete all phase criteria before advancing to next phase

### **CHECKLIST STRUCTURE REQUIREMENTS**
- **Strategy Folders**: `checklists/{strategy-name}/` (e.g., `comprehensive-testing/`)
- **Phase Files**: `phase-1.md`, `phase-2.md`, `phase-3.md` within strategy folders
- **Story Points**: Each story must have story points (1-8 scale)
- **Task Granularity**: Each task must be completable in 1-2 hours
- **Specificity**: Tasks must include exact file paths, method names, and functionality
- **Testing**: Each task must specify testing requirements
- **Compliance**: Norwegian compliance requirements must be explicit

### **PHASE-BASED FOLDER STRUCTURE**
```
checklists/
├── {strategy-name}/           # Strategy-specific folders
│   ├── phase-1.md            # Phase 1 implementation tasks
│   ├── phase-2.md            # Phase 2 implementation tasks
│   └── phase-3.md            # Phase 3 implementation tasks
├── phase-checklist-template.md  # Template for creating new phases
└── README.md                 # Documentation and usage guidelines
```

### **TASK COMPLETION RULES**
- **MANDATORY**: Use `- [ ]` unchecked checkboxes for all tasks
- **REQUIRED**: Check `- [x]` when task is completed and tested
- **FORBIDDEN**: Marking tasks complete without proper testing
- **MANDATORY**: Update status indicators (📋 PLANNED, 🔄 IN PROGRESS, ✅ COMPLETED)
- **MANDATORY**: Update cursor-updates after each story completion

### **WHEN USER SAYS "PLAN"**
**MANDATORY**: Use the checklist template workflow:
1. Copy `checklists/phase-checklist-template.md`
2. Create new strategy folder `checklists/{strategy-name}/`
3. Generate phase files with specific implementation plans
4. Replace all template placeholders with specific information
5. Define actionable tasks with exact file paths and requirements

### **QUALITY GATES FOR PHASE COMPLETION**
- [ ] All tasks in phase must be checked off
- [ ] All specified tests must pass
- [ ] Code coverage targets must be met
- [ ] Norwegian compliance validation must pass
- [ ] Performance benchmarks must be achieved
- [ ] Security validation must complete successfully
- [ ] All phase completion criteria validated before advancing

---

These comprehensive rulesets ensure a clean, compliant, reusable, maintainable, and consistent codebase for all future SaaS application developments.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Xala-Technologies) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
