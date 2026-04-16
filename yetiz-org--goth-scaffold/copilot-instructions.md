## goth-scaffold

> E2E tests guidelines without parallel execution


# E2E Testing Guidelines

## 🚫 PARALLEL EXECUTION (STRICTLY FORBIDDEN)
**E2E tests MUST NOT use `t.Parallel()`** - Shared database causes race conditions and flaky tests.

## 📁 FILE STRUCTURE

### Package Doc Comment (MANDATORY)
```go
/*
Package e2e - {Suite Name} Test Suite

API Path: /api/v1/{resource}
Permission Requirements: {required acceptances}

Test Case List:
  - Test{Domain}_{Operation}_{Scenario}: {description}
*/
package e2e
```

## 🏷️ NAMING CONVENTIONS

### Test Functions
**Pattern**: `Test{Domain}_{Operation}_{Scenario}`
- `Domain`: Feature area (e.g., `AdminToken`, `Users`)
- `Operation`: Action (e.g., `Post`, `Delete`, `Parse`)
- `Scenario`: Case (e.g., `Success`, `Unauthorized`, `InvalidJSON`)

### Setup Functions
**Pattern**: `setup{Domain}Test(t *testing.T) (..., cleanup func())`

## 📝 FUNCTION DOC COMMENTS (MANDATORY)
```go
// TestDomain_Operation_Scenario tests {what is being tested}
//
// Test scenarios:
//   - {Step 1}
//   - {Expected outcome}
func TestDomain_Operation_Scenario(t *testing.T) {
```

## 🏗️ TEST STRUCTURE (Arrange-Act-Assert)
```go
func TestDomain_Operation_Scenario(t *testing.T) {
    // Arrange
    result := testutils.GenerateAppToken(t)
    defer testutils.CleanupToken(t, result)  // IMMEDIATELY after creation

    // Act
    parsedToken, err := service.ParseToken(result.TokenString)

    // Assert
    require.NoError(t, err, "Should parse token successfully")
    assert.Equal(t, result.Token.Id, parsedToken.Id, "Token ID should match")
}
```

## ✅ ASSERTION RULES

| Function | Use Case | On Failure |
|----------|----------|------------|
| `require` | Prerequisites, fatal conditions | Stops test |
| `assert` | Verifications, non-fatal checks | Continues |

**Rules:**
- `require` for setup/prerequisites that must succeed
- `assert` for actual test verifications
- **ALWAYS include descriptive message as last parameter**

## 📊 TABLE-DRIVEN TESTS
Use for multiple similar scenarios:
```go
tests := []struct {
    name     string
    input    InputType
    wantErr  bool
}{...}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        // test logic
    })
}
```

## 🌐 HTTP TEST PATTERN
```go
func TestAPI_Operation_Scenario(t *testing.T) {
    token := testutils.GenerateAppToken(t)
    defer testutils.CleanupToken(t, token)

    client := testutils.NewTestClient()
    req, err := http.NewRequest("POST", baseURL+"/api/v1/resource", body)
    require.NoError(t, err)
    req.Header.Set("Authorization", "Bearer "+token.TokenString)

    resp, err := client.Do(req)
    require.NoError(t, err)
    defer resp.Body.Close()

    assert.Equal(t, http.StatusOK, resp.StatusCode)
}
```

## 🔧 TESTUTILS (tests/e2e/testutils/)
| Helper | Purpose |
|--------|---------|
| `GenerateAppToken`, `GenerateAdminToken`, `CleanupToken` | Token management |
| `NewTestClient` | HTTP client |
| `CompleteSignup`, `Login`, `GetUserAccessToken` | User/session |

## ⚠️ COMMON MISTAKES

| ❌ DON'T | ✅ DO |
|----------|-------|
| Missing `defer cleanup()` | Always cleanup after creation |
| `assert.NoError` for setup | Use `require.NoError` for prerequisites |
| No error message | Always add descriptive message |
| `t.Parallel()` | Never use in E2E tests |

## 📋 CHECKLIST
- [ ] **NO `t.Parallel()`**
- [ ] Package doc with test list
- [ ] Naming: `Test{Domain}_{Operation}_{Scenario}`
- [ ] Doc comments with scenarios
- [ ] `defer cleanup()` for all resources
- [ ] `require` for setup, `assert` for verify
- [ ] Descriptive assertion messages
- [ ] `go test -v ./tests/e2e/...` passes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yetiz-org) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
