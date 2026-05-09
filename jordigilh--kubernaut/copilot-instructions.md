## 09-interface-method-validation

> Interface and method validation before code generation

# Interface and Method Validation Mandate - ENHANCED

## 🔍 **MANDATORY: Interface and Method Validation Before Code Generation**

### Critical Validation Process - ENHANCED
**MANDATORY**: Before generating ANY test code that calls methods or uses types:

1. **Interface Verification**: Use `codebase_search` to verify actual interface definitions
2. **Method Existence Check**: Confirm all called methods exist with EXACT signatures
3. **Type Validation**: Verify all referenced types and struct fields exist
4. **Import Validation**: Ensure all imported packages and types are available
5. **🆕 COMPILATION VERIFICATION**: MANDATORY compilation check after interface usage
6. **🆕 TYPE COMPATIBILITY CHECK**: Verify parameter and return type compatibility

### 🆕 **MANDATORY CODE GENERATION HALT PROTOCOL - ENHANCED**
**BEFORE generating ANY line of code:**

1. **MANDATORY SEARCH**: Run `codebase_search "existing [ComponentType] real implementations"` first, then `codebase_search "existing [ComponentType] mock implementations"`
2. **MANDATORY VERIFICATION**: If real business logic exists, PREFER it over mocks; if using mocks, use existing ones
3. **🆕 BUILD ERROR PREVENTION**: Check for common build error patterns
4. **🆕 IMPORT CONSISTENCY**: Verify all imports exist and are properly used
5. **VIOLATION RESPONSE**: If attempting to create duplicate mocks, IMMEDIATELY STOP and use existing

**ENFORCEMENT TRIGGER WORDS**:
- Creating any type with "Mock" in name → TRIGGER validation
- Using `NewMock*` → TRIGGER existing pattern search
- Implementing interfaces → TRIGGER interface validation

**VIOLATION AUTO-DETECTION - ENHANCED**:
```bash
# If you find yourself typing any of these, STOP:
type Mock* struct          # ❌ VIOLATION: Check existing mocks first
func NewMock*             # ❌ VIOLATION: Use existing patterns
*Mock struct {            # ❌ VIOLATION: Reuse existing mocks
logrus.New()              # ❌ VIOLATION: Use existing mocks.NewMockLogger()
mockLogger                # ❌ VIOLATION: Check variable declaration
import.*logrus.*\n.*not   # ❌ VIOLATION: Unused import detected
```

**🆕 COMMON BUILD ERROR PATTERNS TO PREVENT**:
```bash
# These patterns MUST trigger immediate validation:
mockLogger.*without.*var  # ❌ Undefined variable usage
import.*unused           # ❌ Unused import statements
mocks\..*without.*import  # ❌ Mock usage without import
NewMock.*duplicate       # ❌ Duplicate mock creation
```

---

## 🚨 **MANDATORY VALIDATION SEQUENCE - ENHANCED**

### **Step 1: Interface Discovery and Verification**
```bash
# MANDATORY: Search for existing interfaces before creating/using
codebase_search "existing [InterfaceName] interface definitions"
codebase_search "existing [InterfaceName] implementations"

# Verify interface exists and get exact signature
grep -r "type.*[InterfaceName].*interface" pkg/ --include="*.go"
```

**Example Validation**:
```bash
# Before using WorkflowEngine interface
codebase_search "existing WorkflowEngine interface definitions"
# Result should show: pkg/workflow/engine/interfaces.go

# Verify method signatures
grep -A 10 "type WorkflowEngine interface" pkg/workflow/engine/interfaces.go
```

### **Step 2: Method Signature Validation**
```bash
# MANDATORY: Verify exact method signatures before calling
grep -A 20 "type.*[InterfaceName].*interface" [interface_file.go]

# Check method parameters and return types
grep "[MethodName].*(" [interface_file.go]
```

**Example Method Validation**:
```go
// ✅ CORRECT: Verify method signature first
// From pkg/workflow/engine/interfaces.go:
// CreateWorkflow(ctx context.Context, alert AlertData) (*Workflow, error)

// Then use in test:
workflow, err := workflowEngine.CreateWorkflow(ctx, alertData)
```

### **Step 3: Mock Existence and Reuse Check**
```bash
# MANDATORY: Check for existing mocks before creating new ones
find pkg/testutil/mocks/ -name "*[ComponentName]*" -type f
grep -r "Mock[ComponentName]" pkg/testutil/ --include="*.go"

# If mocks exist, REUSE them
# If no mocks exist, check if real component should be used instead
```

**Mock Reuse Decision Matrix**:
| Component Type | Action |
|---------------|--------|
| **External Services** (AI, K8s, DB) | Use existing mocks from `pkg/testutil/mocks/` |
| **Business Logic** (Engine, Analytics) | Use REAL components |
| **Configuration** | Use real config with test values |
| **Utilities** | Use real utilities |

### **🆕 Step 4: Compilation Verification**
```bash
# MANDATORY: Test compilation after interface usage
go build ./test/[test_package]/ 2>&1 | tee build_check.log

# Check for common errors:
grep "undefined:" build_check.log    # Undefined symbols
grep "cannot use" build_check.log    # Type mismatches
grep "not enough arguments" build_check.log  # Parameter mismatches
```

### **🆕 Step 5: Import Consistency Check**
```bash
# MANDATORY: Verify all imports are used and correct
go mod tidy
goimports -w [test_file.go]

# Check for unused imports
go build [test_file.go] 2>&1 | grep "imported and not used"
```

---

## 🔧 **AUTOMATED VALIDATION TOOLS**

### **Interface Validation Script**
```bash
#!/bin/bash
# scripts/validate-interface-usage.sh

INTERFACE_NAME="$1"
TEST_FILE="$2"

echo "🔍 VALIDATING INTERFACE USAGE: $INTERFACE_NAME in $TEST_FILE"

# Step 1: Find interface definition
INTERFACE_FILE=$(find pkg/ -name "*.go" -exec grep -l "type.*$INTERFACE_NAME.*interface" {} \;)
if [ -z "$INTERFACE_FILE" ]; then
    echo "❌ ERROR: Interface $INTERFACE_NAME not found"
    exit 1
fi

echo "✅ Interface found in: $INTERFACE_FILE"

# Step 2: Extract interface methods
METHODS=$(grep -A 50 "type.*$INTERFACE_NAME.*interface" "$INTERFACE_FILE" | grep -E "^\s*[A-Z].*\(" | sed 's/^\s*//')
echo "📋 Available methods:"
echo "$METHODS"

# Step 3: Check test file usage
echo "🔍 Checking method usage in $TEST_FILE..."
while IFS= read -r method; do
    METHOD_NAME=$(echo "$method" | cut -d'(' -f1 | xargs)
    if grep -q "$METHOD_NAME" "$TEST_FILE"; then
        echo "✅ Method used: $METHOD_NAME"
    fi
done <<< "$METHODS"

# Step 4: Compilation check
echo "🔨 Testing compilation..."
if go build "$TEST_FILE" 2>/dev/null; then
    echo "✅ Compilation successful"
else
    echo "❌ Compilation failed:"
    go build "$TEST_FILE" 2>&1
    exit 1
fi

echo "✅ Interface validation complete"
```

### **Mock Reuse Validation Script**
```bash
#!/bin/bash
# scripts/validate-mock-reuse.sh

COMPONENT_NAME="$1"

echo "🔍 VALIDATING MOCK REUSE: $COMPONENT_NAME"

# Check for existing mocks
EXISTING_MOCKS=$(find pkg/testutil/mocks/ -name "*$COMPONENT_NAME*" -type f)
if [ -n "$EXISTING_MOCKS" ]; then
    echo "✅ Existing mocks found:"
    echo "$EXISTING_MOCKS"
    echo "🔧 RECOMMENDATION: Reuse existing mocks"

    # Show mock usage patterns
    echo "📋 Mock usage patterns:"
    grep -r "NewMock$COMPONENT_NAME\|Mock$COMPONENT_NAME" pkg/testutil/mocks/ --include="*.go"
else
    echo "⚠️  No existing mocks found for $COMPONENT_NAME"

    # Check if real component should be used
    REAL_COMPONENT=$(find pkg/ -name "*.go" -not -path "*/mocks/*" -not -name "*_test.go" -exec grep -l "$COMPONENT_NAME" {} \;)
    if [ -n "$REAL_COMPONENT" ]; then
        echo "💡 RECOMMENDATION: Consider using real component instead of creating mock"
        echo "📁 Real component locations:"
        echo "$REAL_COMPONENT"
    fi
fi
```

---

## 🚨 **VIOLATION DETECTION AND PREVENTION**

### **Real-Time Violation Detection**
```bash
# Monitor for interface violations during development
fswatch test/ | while read file; do
    if [[ "$file" == *_test.go ]]; then
        echo "🔍 Checking $file for interface violations..."
        # Prevention through interface verification during APDC Analysis phase "$file"
    fi
done
```

### **Pre-Commit Interface Validation**
```bash
#!/bin/bash
# .git/hooks/pre-commit addition

echo "🔍 Validating interface usage in modified test files..."

MODIFIED_TESTS=$(git diff --cached --name-only --diff-filter=ACM | grep "_test.go$")
for test_file in $MODIFIED_TESTS; do
    echo "Validating: $test_file"

    # Check for interface usage
    INTERFACES_USED=$(grep -o "[A-Z][a-zA-Z]*Engine\|[A-Z][a-zA-Z]*Client\|[A-Z][a-zA-Z]*Service" "$test_file" | sort -u)

    for interface_name in $INTERFACES_USED; do
        # Prevention through interface verification during APDC Analysis phase "$interface_name" "$test_file"
        if [ $? -ne 0 ]; then
            echo "❌ Interface validation failed for $interface_name in $test_file"
            exit 1
        fi
    done
done

echo "✅ All interface validations passed"
```

---

## 📋 **INTERFACE VALIDATION CHECKLIST**

### **Before Writing ANY Test Code**
- [ ] **Interface Discovery**: `codebase_search "existing [Interface] definitions"`
- [ ] **Method Verification**: Confirmed exact method signatures
- [ ] **Mock Check**: Verified existing mocks or decided on real components
- [ ] **Import Validation**: All imports are correct and used
- [ ] **Compilation Test**: Code compiles without errors

### **During Test Development**
- [ ] **Method Calls**: All method calls match exact interface signatures
- [ ] **Parameter Types**: All parameters match expected types
- [ ] **Return Handling**: All return values handled correctly
- [ ] **Error Handling**: All error returns properly handled

### **After Test Completion**
- [ ] **Final Compilation**: Full test suite compiles
- [ ] **Import Cleanup**: No unused imports
- [ ] **Mock Consistency**: Consistent mock usage patterns
- [ ] **Interface Compliance**: All interface contracts satisfied

---

## 🎯 **QUALITY GATES**

### **Interface Usage Gates**
- **Gate 1**: Interface must exist before usage
- **Gate 2**: Method signatures must match exactly
- **Gate 3**: Existing mocks must be reused
- **Gate 4**: Code must compile after interface usage
- **Gate 5**: All imports must be used and correct

### **Mock Usage Gates**
- **Gate 1**: Check existing mocks before creating new
- **Gate 2**: Prefer real business logic over mocks
- **Gate 3**: External dependencies only for mocks
- **Gate 4**: Consistent mock patterns across tests

---

## 🔗 **INTEGRATION WITH OTHER RULES**

**Enforces**: [08-testing-anti-patterns.mdc](mdc:.cursor/rules/08-testing-anti-patterns.mdc) mock usage guidelines
**Supports**: [03-testing-strategy.mdc](mdc:.cursor/rules/03-testing-strategy.mdc) pyramid testing approach
**Validates**: [12-ai-ml-development-methodology.mdc](mdc:.cursor/rules/12-ai-ml-development-methodology.mdc) AI interface usage
**Prevents**: Build errors and undefined symbol issues
**Priority**: VALIDATION - Prevents interface-related development errors before they occur

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
