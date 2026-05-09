## 01-ai-assistant-behavior

> AI assistant behavioral checkpoints and validation requirements


# AI Assistant Behavioral Guidelines

## 🎯 **Core Principle**

**PREVENTION over CORRECTION**: Execute validation BEFORE generating code, not after.

**Authority**: These guidelines enforce compliance with [00-kubernaut-core-rules.mdc](mdc:.cursor/rules/00-kubernaut-core-rules.mdc)

---

## 🚨 **MANDATORY VALIDATION CHECKPOINTS**

### **CHECKPOINT A: Type Reference Validation**

**TRIGGER**: About to reference any struct field (e.g., `object.FieldName`)

**ACTION**:
```bash
# HALT: Read type definition file FIRST
read_file pkg/path/to/type_definition.go
# Verify field exists in struct definition before referencing
```

**RULE**: All referenced fields MUST exist in type definitions
**VIOLATION**: "🚨 CHECKPOINT A VIOLATION: Type reference attempted without validation - DEVELOPMENT STOPPED"

---

### **CHECKPOINT B: Test Creation Validation**

**TRIGGER**: About to create test file with business logic references

**ACTION**:
```bash
# HALT: Search for existing implementations FIRST
codebase_search "existing [ComponentType] implementations"
grep -r "[ComponentType]" pkg/ --include="*.go"
```

**RULE**: Enhance existing patterns instead of creating new
**VIOLATION**: "🚨 CHECKPOINT B VIOLATION: Test creation attempted without existing implementation analysis - DEVELOPMENT STOPPED"

---

### **CHECKPOINT C: Business Integration Validation**

**TRIGGER**: Creating new business types or interfaces

**ACTION**:
```bash
# HALT: Verify main application integration
grep -r "[NewComponentType]" cmd/ --include="*.go"
```

**RULE**: Business code MUST be integrated in main applications (cmd/)
**VIOLATION**: "🚨 CHECKPOINT C VIOLATION: Business component creation attempted without main app integration validation - DEVELOPMENT STOPPED"

---

### **CHECKPOINT D: Build Error Investigation**

**TRIGGER**: User reports build errors or undefined symbols

**ACTION**:
```bash
# HALT: Execute comprehensive symbol analysis
codebase_search "[undefined_symbol] usage patterns and dependencies"
grep -r "[undefined_symbol]" . --include="*.go" -n
go build [affected_file] 2>&1
```

**REQUIRED REPORT**:
```
🚨 UNDEFINED SYMBOL ANALYSIS:
Symbol: [undefined_symbol]
References found: [N files with paths]
Dependent infrastructure: [list missing types/functions]
Scope: [minimal/medium/extensive with evidence]

OPTIONS (Evidence-Based):
A) Implement complete infrastructure ([X] files affected)
B) Create minimal stub ([Z] files affected, may break [W] files)
C) Alternative approach: [evidence-based alternative]

🚫 MANDATORY USER DECISION REQUIRED: Which approach? (A/B/C)
```

**RULE**: NO implementation without user approval after complete analysis
**VIOLATION**: "🚨 CHECKPOINT D VIOLATION: Build error resolution attempted without comprehensive analysis + user approval - DEVELOPMENT STOPPED"

---

## 🚫 **FORBIDDEN AI ACTIONS**

**NEVER DO THESE**:
1. ❌ Reference struct fields without reading type definition file
2. ❌ Assume testutil types exist - validate with `read_file` or `grep`
3. ❌ Create test code without `codebase_search` for existing implementations
4. ❌ Generate business types without confirming main application usage
5. ❌ Proceed if any validation step fails
6. ❌ Implement missing types without full dependency analysis (CHECKPOINT D)

---

## 🔧 **MANDATORY TOOL USAGE PATTERN**

AI MUST use tools in this sequence for any code generation:

### **Step 1: Discovery (REQUIRED)**
```bash
codebase_search "existing [ComponentType] implementations"
# OR
grep -r "[TypeName]" pkg/ --include="*.go"
```

### **Step 2: Type Validation (REQUIRED)**
```bash
read_file pkg/path/to/type_definition.go
# Verify all referenced fields exist in struct definitions
```

### **Step 3: Integration Check (REQUIRED)**
```bash
grep -r "[NewType]" cmd/ --include="*.go"
# Verify business code is used in main applications
```

---

## ✅ **MANDATORY DECISION GATES**

AI must answer YES to ALL questions before proceeding:

### **For Type References:**
- ✅ Have I read the actual type definition file?
- ✅ Do all referenced fields exist in the struct?
- ✅ Are there no empty struct{} definitions?

### **For Test Creation:**
- ✅ Have I searched for existing test patterns?
- ✅ Am I following TDD RED-GREEN-REFACTOR sequence?
- ✅ Do all testutil dependencies actually exist?

### **For Business Code:**
- ✅ Is this integrated into main applications?
- ✅ Are all interfaces implemented by real code?
- ✅ Have I verified the complete dependency chain?

### **For Build Error Fixes (CHECKPOINT D):**
- ✅ Have I analyzed ALL references to the undefined symbol?
- ✅ Have I mapped the complete dependency chain?
- ✅ Have I presented options A/B/C to the user?
- ✅ Have I received explicit approval before implementing?

---

## 📊 **CONFIDENCE ASSESSMENT REQUIREMENTS**

After any code generation, AI must provide:

```
Validation Confidence: [60-100]%
Type Safety: ✅/❌ All referenced fields exist in type definitions
TDD Compliance: ✅/❌ Follows RED-GREEN-REFACTOR sequence
Integration Status: ✅/❌ Business code integrated in main applications
Build Error Analysis: ✅/❌ Complete dependency analysis performed (if applicable)
Risk Assessment: [Description of potential issues]
```

---

## 🛑 **EMERGENCY STOP CONDITIONS**

AI must immediately halt and request manual intervention if:

1. **Type definitions incomplete** (empty structs) - SESSION RISK: Runtime panics
2. **Referenced fields don't exist** - SESSION RISK: Compilation failures
3. **Business code has no main app integration** - SESSION RISK: Orphaned code
4. **TDD sequence being bypassed** - SESSION RISK: Incomplete implementations
5. **Validation scripts missing/failing** - SESSION RISK: Undetected problems
6. **Undefined symbols without complete analysis** - SESSION RISK: Cascade failures

---

## 🎯 **QUALITY GATES**

No code generation permitted without:

- ✅ Type safety validation passing
- ✅ TDD methodology compliance confirmed
- ✅ Business integration verified
- ✅ Dependency existence validated
- ✅ Main application usage confirmed
- ✅ Build error analysis completed (when applicable)

---

## 🔄 **ERROR PREVENTION PROTOCOL**

### **Recovery Actions**

When validation fails:
1. **STOP** code generation immediately
2. **REPORT** specific validation failure
3. **SUGGEST** corrective action following TDD methodology
4. **REQUIRE** validation success before proceeding

### **Example Tool Call Enforcement**

**For ANY struct field reference (object.FieldName)**:
```xml
<function_calls>
<invoke name="read_file">
<parameter name="target_file">pkg/workflow/engine/types.go</parameter>
</invoke>
</function_calls>
```

**For ANY undefined symbol error**:
```xml
<function_calls>
<invoke name="codebase_search">
<parameter name="query">[symbol_name] usage patterns and dependencies</parameter>
<parameter name="target_directories">[]</parameter>
</invoke>
</function_calls>
```

---

## 📚 **INTEGRATION WITH OTHER RULES**

This rule **enforces** compliance with:
- [00-kubernaut-core-rules.mdc](mdc:.cursor/rules/00-kubernaut-core-rules.mdc) - Core TDD methodology
- [03-testing-strategy.mdc](mdc:.cursor/rules/03-testing-strategy.mdc) - Testing framework
- [02-go-coding-standards.mdc](mdc:.cursor/rules/02-go-coding-standards.mdc) - Technical patterns

**PRIORITY LEVEL**: 3 - BEHAVIORAL (per [13-conflict-resolution-matrix.mdc](mdc:.cursor/rules/13-conflict-resolution-matrix.mdc))

**Scope**: AI assistant behavior - does not override APDC methodology or testing strategy

---

## ⚡ **QUICK REFERENCE FOR BUILD ERROR INVESTIGATION**

| User Command | AI Response |
|--------------|-------------|
| "Fix build errors" | Execute CHECKPOINT D + ask for approval |
| "undefined [symbol]" | Complete symbol analysis + show options A/B/C |
| "implement recommended approach" | Follow approved option exactly as specified |

---

**Remember**: Problems are PREVENTED during the session through mandatory AI validation, not caught after commit.

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
