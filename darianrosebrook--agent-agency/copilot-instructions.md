## agent-agency

> Scope management, change budgets, and emergency waiver procedures for critical fixes


# Scope Management & Waiver System

## Core Principle

**All changes must stay within defined scope and budget limits.** Emergency situations require structured waiver procedures with proper documentation and mitigation plans.

## Change Budget Enforcement

### Budget Definition

```yaml
# Working spec requirements
change_budget:
  max_files: 25
  max_loc: 1000
  max_days: 3
  max_complexity: 10  # Cyclomatic complexity
```

### Scope Boundaries

```yaml
scope:
  in: ["src/auth/", "tests/auth/", "package.json"]
  out: ["src/billing/", "node_modules/", "dist/"]
  blast_radius:
    modules: ["auth", "api"]
    data_migration: false
    external_apis: false
```

### Automatic Detection

**CI/CD Integration:**
- Block commits that touch files outside `scope.in`
- Fail if change exceeds `max_files` or `max_loc`
- Require justification for budget increases
- Track scope violations in metrics

## Emergency Waiver System

### Waiver Triggers

**Critical situations requiring waivers:**
- Security vulnerabilities
- Production outages
- Data corruption
- Critical business impact
- Regulatory compliance issues

### Waiver Process

```yaml
# Emergency waiver workflow
waiver:
  id: "CRITICAL-FIX-001"
  title: "Security Hotfix for Auth Bypass"
  reason: "security_vulnerability"
  impact_level: "high"  # low, medium, high, critical
  approved_by: "security_team"
  expires_at: "2024-01-15"
  gates_waived: ["coverage_threshold", "mutation_score"]
  mitigation_plan: "Full test suite added in follow-up PR"
  rollback_plan: "Revert to previous auth implementation"
```

### Waiver Documentation Requirements

**Must include:**
- Root cause analysis
- Minimal fix description
- Risk assessment
- Mitigation timeline
- Prevention measures
- Rollback procedures

## Scope Blowout Protocols

### Detection & Response

**Automatic Detection:**
```bash
# Check for scope violations
git diff --name-only HEAD~1 | grep -v -E "^($(echo "${SCOPE_IN}" | tr ' ' '|'))"
```

**Response Strategies:**
1. **Split Strategy**: Break large changes into focused PRs
2. **Budget Increase**: Requires human approval with justification
3. **Emergency Override**: Security fixes can bypass scope limits
4. **Scope Update**: Modify scope.in with proper documentation

### Budget Increase Process

**Requirements for budget increase:**
- Detailed justification
- Risk assessment
- Mitigation plan
- Timeline for completion
- Human approval (not automated)

## Critical Fix Procedures

### Emergency Fix Workflow

```yaml
emergency_fix:
  triggers:
    - security_vulnerability
    - production_outage
    - data_corruption
    - critical_business_impact
    
  process:
    1. "Create emergency waiver"
    2. "Implement minimal fix"
    3. "Deploy with monitoring"
    4. "Follow up with comprehensive solution"
    
  documentation:
    - "Root cause analysis"
    - "Minimal fix description"
    - "Risk assessment"
    - "Mitigation timeline"
    - "Prevention measures"
```

### Emergency Override Conditions

**Can bypass scope limits when:**
- Security vulnerability with active exploitation
- Production system completely down
- Data corruption in progress
- Regulatory compliance deadline

**Cannot bypass:**
- Code quality gates (linting, type checking)
- Security scanning
- Basic testing requirements
- Documentation standards

## Waiver Management Commands

### CAWS Integration

```bash
# Create waiver
caws waiver create \
  --title "Security Hotfix" \
  --reason "security_vulnerability" \
  --gates "coverage_threshold,mutation_score" \
  --expires-at "2024-01-15" \
  --approved-by "security_team" \
  --impact-level "high" \
  --mitigation-plan "Full test suite in follow-up PR"

# List active waivers
caws waivers list --status=active

# Check waiver status
caws waivers status --waiver-id="CRITICAL-FIX-001"
```

## Quality Gates by Risk Tier

### Tier 1 (Critical Systems)
- **Coverage**: 90%+ line, 95%+ branch
- **Mutation Score**: 70%+
- **Contracts**: Required
- **Review**: Manual required
- **Security Scan**: Required
- **Waiver Approval**: Security team

### Tier 2 (Standard Features)
- **Coverage**: 80%+ line, 90%+ branch
- **Mutation Score**: 50%+
- **Contracts**: Required
- **Review**: Optional
- **Security Scan**: Required
- **Waiver Approval**: Tech lead

### Tier 3 (Low Risk)
- **Coverage**: 70%+ line, 80%+ branch
- **Mutation Score**: 30%+
- **Contracts**: Optional
- **Review**: Optional
- **Security Scan**: Recommended
- **Waiver Approval**: Developer

## Enforcement Mechanisms

### Pre-Commit Hooks

```bash
# Check scope compliance
check-scope-compliance() {
  local changed_files=$(git diff --cached --name-only)
  local scope_violations=$(echo "$changed_files" | grep -v -E "^($(echo "${SCOPE_IN}" | tr ' ' '|'))")
  
  if [ -n "$scope_violations" ]; then
    echo "❌ Scope violation detected:"
    echo "$scope_violations"
    echo "Files outside scope.in boundaries"
    exit 1
  fi
}
```

### CI/CD Integration

```yaml
# GitHub Actions example
- name: Check Scope Compliance
  run: |
    if ! caws validate --scope-check; then
      echo "❌ Scope violation detected"
      exit 1
    fi

- name: Check Active Waivers
  run: |
    active_waivers=$(caws waivers list --status=active --format=count)
    if [ "$active_waivers" -gt 5 ]; then
      echo "⚠️ High number of active waivers: $active_waivers"
    fi
```

## Metrics & Monitoring

### Scope Compliance Metrics

- **Scope Violation Rate**: % of commits violating scope
- **Budget Overrun Rate**: % of changes exceeding budget
- **Waiver Usage**: Number of active waivers by type
- **Emergency Fix Frequency**: Rate of emergency fixes
- **Mitigation Completion**: % of waivers with completed mitigation

### Quality Trends

- **Coverage by Tier**: Track coverage trends per risk tier
- **Mutation Score Trends**: Monitor mutation testing scores
- **Review Coverage**: % of changes with proper review
- **Security Scan Results**: Track security scan violations

## Continuous Improvement

### Monthly Reviews

- Review scope violation patterns
- Analyze waiver usage and effectiveness
- Update scope boundaries based on project evolution
- Refine budget limits based on team capacity

### Quarterly Audits

- Full scope compliance audit
- Waiver effectiveness review
- Emergency procedure testing
- Quality gate threshold review

This rule ensures proper scope management while providing structured escape hatches for critical situations that require immediate attention.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darianrosebrook)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/darianrosebrook)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
