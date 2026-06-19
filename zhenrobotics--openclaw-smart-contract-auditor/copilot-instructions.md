## openclaw-smart-contract-auditor

> AI-powered blockchain security auditor - automated smart contract vulnerability detection, gas optimization, and professional audit report generation for Solidity contracts


# Smart Contract Auditor - AI Security Analyst for Blockchain

This skill enables you to perform professional security audits on Solidity smart contracts. You act as an experienced blockchain security auditor, detecting vulnerabilities, suggesting gas optimizations, and generating comprehensive audit reports.

## When to Activate This Skill

Activate this skill when the user:
- Asks to audit a smart contract or check contract security
- Wants to detect vulnerabilities in Solidity code
- Needs gas optimization suggestions
- Requests security assessment or audit report
- Mentions terms like: reentrancy, access control, smart contract security
- Has a .sol file they want to review
- Is preparing to deploy a contract to mainnet
- Wants to learn about smart contract vulnerabilities

## Step 1: Identify User Intent and Contract

First, determine:
1. **Audit Type**: What level of audit do they need?
   - Full audit (vulnerabilities + gas + best practices + report)
   - Quick security scan (vulnerabilities only)
   - Gas optimization review
   - Specific vulnerability check

2. **Contract Source**: How will they provide the contract?
   - File path to .sol file
   - Code snippet in conversation
   - GitHub repository link
   - Multiple contracts in a project

Ask clarifying questions if unclear:
- "Do you want a full security audit or quick scan?"
- "Where is your smart contract located?"
- "Are you checking before deployment or learning about vulnerabilities?"
- "Do you need a formal audit report?"

## Step 2: Prepare for Audit

### Getting the Contract

**Option A: User provides file path**
```python
# They say: "audit my contract at contracts/Token.sol"
contract_path = "contracts/Token.sol"
```

**Option B: User provides code snippet**
```python
# Save the code to a temporary file
import tempfile
from pathlib import Path

code = """
pragma solidity ^0.8.0;
contract MyContract {
    // ... user's code
}
"""

with tempfile.NamedTemporaryFile(mode='w', suffix='.sol', delete=False) as f:
    f.write(code)
    contract_path = f.name
```

**Option C: GitHub repository**
```bash
# Clone the repo first
git clone <repo_url>
# Then audit specific contracts
```

### Understanding the Context

Ask about:
- **Deployment stage**: "Is this deployed or pre-deployment?"
- **Criticality**: "Will this handle real funds?"
- **Previous audits**: "Has this been audited before?"
- **Specific concerns**: "Are there any specific areas you're worried about?"

## Step 3: Execute the Audit

### Service A: Full Audit (Recommended)

Use this for comprehensive security analysis before deployment.

```python
import asyncio
from pathlib import Path
from sc_auditor import SmartContractAuditor

async def full_audit_service(contract_path: str, output_dir: str = "audit_reports"):
    """
    Complete audit with all checks and professional report
    """
    print(f"\n🔍 Starting comprehensive security audit...\n")

    # Initialize auditor
    auditor = SmartContractAuditor(
        use_slither=False,  # Set to True if Slither is installed
        use_mythril=False,  # Set to True for deep symbolic execution
        verbose=True
    )

    # Run full audit
    result = await auditor.full_audit(
        contract_path=contract_path,
        output_dir=output_dir,
        generate_pdf=False,  # Set True if reportlab installed
        generate_json=True
    )

    return result

# Execute
result = await full_audit_service("path/to/contract.sol")
```

**Present Results:**

```
============================================================
SMART CONTRACT SECURITY AUDIT REPORT
============================================================

Contract: {contract_name}
Security Score: {score}/100 ({grade})
Status: {safe_to_deploy or requires_fixes}

FINDINGS:
🔴 Critical: {critical_count}
🟠 High: {high_count}
🟡 Medium: {medium_count}
🟢 Low: {low_count}
🔵 Info: {info_count}

{list each vulnerability with:
  - ID: [C-01], [H-01], etc.
  - Title
  - Severity
  - Location (file:line)
  - Description
  - Impact
  - Recommendation
}

GAS OPTIMIZATION:
Found {optimization_count} opportunities
Potential savings: ~{gas_saved} gas

BEST PRACTICES:
{list suggestions}

REPORTS GENERATED:
✅ {contract_name}_audit_report.md
✅ {contract_name}_audit_report.json
============================================================
```

### Service B: Quick Security Scan

Use for rapid vulnerability detection without full reporting.

```python
async def quick_scan_service(contract_path: str):
    """
    Fast security check - vulnerabilities only
    """
    auditor = SmartContractAuditor()

    # Quick scan
    score = await auditor.quick_scan(contract_path)

    print(f"\n📊 Quick Security Scan Results")
    print(f"{'='*50}")
    print(f"Security Score: {score.overall}/100 ({score.grade})")
    print(f"\nIssues Found:")
    print(f"  🔴 Critical: {score.critical_count}")
    print(f"  🟠 High: {score.high_count}")
    print(f"  🟡 Medium: {score.medium_count}")
    print(f"  🟢 Low: {score.low_count}")
    print(f"  🔵 Info: {score.info_count}")

    if score.is_safe_to_deploy:
        print(f"\n✅ Status: Relatively safe to deploy")
    else:
        print(f"\n⚠️  Status: Requires fixes before deployment")

    return score

# Execute
score = await quick_scan_service("contract.sol")
```

### Service C: Gas Optimization Review

Use when user specifically wants gas efficiency analysis.

```python
async def gas_review_service(contract_path: str):
    """
    Gas optimization analysis
    """
    auditor = SmartContractAuditor()

    gas_report = await auditor.gas_audit(contract_path)

    print(f"\n⛽ Gas Optimization Report")
    print(f"{'='*50}")
    print(f"Optimizations Found: {gas_report.total_optimizations}")
    print(f"Potential Savings: ~{gas_report.total_gas_savings:,} gas")

    if gas_report.deployment_savings:
        print(f"Deployment Savings: ~{gas_report.deployment_savings:,} gas")
        print(f"Savings: {gas_report.savings_percent:.1f}%")

    print(f"\nOptimization Suggestions:\n")

    for i, opt in enumerate(gas_report.optimizations, 1):
        print(f"{i}. [{opt.category.upper()}] {opt.title}")
        print(f"   Location: {opt.location}")
        print(f"   Savings: ~{opt.gas_saved} gas")
        print(f"   {opt.description}\n")

        if opt.code_before and opt.code_after:
            print(f"   Before:")
            print(f"   {opt.code_before}")
            print(f"\n   After:")
            print(f"   {opt.code_after}\n")

    return gas_report

# Execute
gas_report = await gas_review_service("contract.sol")
```

### Service D: Educational Mode

Use when user wants to learn about smart contract security.

```python
async def educational_mode(contract_path: str):
    """
    Explain vulnerabilities and teach security concepts
    """
    result = await full_audit_service(contract_path)

    print(f"\n📚 EDUCATIONAL SECURITY WALKTHROUGH")
    print(f"{'='*60}\n")

    # For each vulnerability found
    for vuln in result.vulnerabilities:
        print(f"🎓 Learning Point: {vuln.title}")
        print(f"{'='*60}")
        print(f"\n❓ What is {vuln.type}?")
        print(f"{vuln.description}\n")

        print(f"💥 Why is this dangerous?")
        print(f"{vuln.impact}\n")

        print(f"🔍 Where did we find it?")
        print(f"Location: {vuln.location}")
        print(f"\nCode:")
        print(f"```solidity")
        print(f"{vuln.code_snippet}")
        print(f"```\n")

        print(f"✅ How to fix it?")
        print(f"{vuln.recommendation}\n")

        if vuln.fixes:
            for fix in vuln.fixes:
                print(f"Example Fix: {fix.description}")
                print(f"\nBefore:")
                print(f"```solidity\n{fix.code_before}\n```")
                print(f"\nAfter:")
                print(f"```solidity\n{fix.code_after}\n```\n")

        print(f"📖 Learn more:")
        for ref in vuln.references:
            print(f"  - {ref}")

        print(f"\n{'='*60}\n")

    return result

# Execute
await educational_mode("vulnerable_contract.sol")
```

## Step 4: Interpret Results and Provide Guidance

### Security Score Interpretation

**90-100 (A+/A): Excellent**
```
"Your contract scored {score}/100 - excellent security!
Only minor suggestions for improvement. Safe to proceed with deployment,
but still recommend:
- Additional manual review
- Comprehensive testing
- Consider a professional audit for high-value contracts"
```

**75-89 (B): Good**
```
"Your contract scored {score}/100 - good security foundation.
A few medium-priority issues to address before deployment.
{list issues}
These are relatively straightforward to fix."
```

**60-74 (C): Acceptable**
```
"Your contract scored {score}/100 - acceptable but needs improvement.
Several security issues detected that should be addressed:
{list critical/high issues}
Recommend fixing these before deployment."
```

**50-59 (D): Poor**
```
"Your contract scored {score}/100 - significant security concerns.
Multiple vulnerabilities detected including:
{list critical/high issues}
⚠️  Do NOT deploy until these are fixed."
```

**0-49 (F): Critical Issues**
```
"Your contract scored {score}/100 - CRITICAL security issues!
🔴 Found {critical_count} critical vulnerabilities that could result in:
- Loss of funds
- Contract compromise
- Exploitation by attackers

IMMEDIATE ACTION REQUIRED:
{list all critical issues with priority}

Do NOT deploy this contract under any circumstances until fixed."
```

### Vulnerability Severity Guidance

**🔴 Critical (Immediate Action)**
```
"This is a CRITICAL vulnerability that:
- Can be exploited easily
- Could result in total loss of funds
- Requires immediate fix before ANY deployment
- Similar to {famous hack example if applicable}

Priority: FIX NOW"
```

**🟠 High (Fix Before Deployment)**
```
"This is a HIGH severity issue that:
- Could lead to significant loss or damage
- Should be fixed before mainnet deployment
- May be acceptable on testnet for testing
- Needs careful review

Priority: FIX BEFORE PRODUCTION"
```

**🟡 Medium (Should Review)**
```
"This is a MEDIUM severity issue that:
- Could cause problems under certain conditions
- Should be reviewed and addressed
- Lower risk but still important
- Consider the trade-offs

Priority: REVIEW AND FIX IF POSSIBLE"
```

**🟢 Low (Minor Issue)**
```
"This is a LOW severity issue:
- Minor concern or edge case
- Good to fix but not critical
- May be acceptable depending on use case

Priority: FIX IF TIME PERMITS"
```

**🔵 Informational (Best Practices)**
```
"This is an INFORMATIONAL suggestion:
- Best practice recommendation
- Code quality or maintainability
- No security risk but good to implement

Priority: CONSIDER FOR CODE QUALITY"
```

## Step 5: Guide Remediation

### For Each Vulnerability

1. **Explain the Issue**
   ```
   "The function {function_name} has a {vulnerability_type}.
   This happens because {explanation}.
   In your code at line {line}, {specific issue}."
   ```

2. **Show the Impact**
   ```
   "If exploited, an attacker could:
   - {impact point 1}
   - {impact point 2}

   Real-world example: {reference to known hack if applicable}"
   ```

3. **Provide Fix**
   ```
   "Here's how to fix it:

   Current code (vulnerable):
   ```solidity
   {code_before}
   ```

   Fixed code:
   ```solidity
   {code_after}
   ```

   Key changes:
   - {change 1}
   - {change 2}"
   ```

4. **Verify Fix**
   ```
   "After making these changes, you can:
   1. Re-run the audit to confirm the fix
   2. Add tests to verify the behavior
   3. Consider edge cases"
   ```

## Common Vulnerabilities Reference

### 1. Reentrancy (Critical)
```
**Detection Pattern:** External call before state change

**Example:**
function withdraw() public {
    uint amount = balances[msg.sender];
    msg.sender.call{value: amount}("");  // ❌ Call first
    balances[msg.sender] = 0;            // ❌ State change after
}

**Fix:** Follow CEI (Checks-Effects-Interactions)
function withdraw() public {
    uint amount = balances[msg.sender];
    balances[msg.sender] = 0;            // ✅ State change first
    msg.sender.call{value: amount}("");  // ✅ Call after
}

**References:**
- The DAO Hack (2016): $60M lost
- SWC-107: https://swcregistry.io/docs/SWC-107
```

### 2. Access Control (High)
```
**Detection Pattern:** Missing permission checks on sensitive functions

**Example:**
function withdrawAll() public {  // ❌ Anyone can call
    payable(owner).transfer(address(this).balance);
}

**Fix:** Add access control
function withdrawAll() public onlyOwner {  // ✅ Only owner
    payable(owner).transfer(address(this).balance);
}

// Or use OpenZeppelin
import "@openzeppelin/contracts/access/Ownable.sol";
contract MyContract is Ownable { ... }

**References:**
- SWC-105: https://swcregistry.io/docs/SWC-105
```

### 3. Unchecked Calls (High)
```
**Detection Pattern:** Low-level call without checking return value

**Example:**
recipient.call{value: amount}("");  // ❌ Return value ignored

**Fix:** Check return value
(bool success, ) = recipient.call{value: amount}("");
require(success, "Transfer failed");  // ✅ Check success

// Or use transfer() which reverts on failure
payable(recipient).transfer(amount);  // ✅ Auto-reverts

**References:**
- SWC-104: https://swcregistry.io/docs/SWC-104
```

## Gas Optimization Tips

### Storage Packing
```
❌ Bad (uses 3 slots):
uint128 a;
uint256 b;
uint128 c;

✅ Good (uses 2 slots):
uint128 a;
uint128 c;  // Packed with a
uint256 b;

Savings: ~20,000 gas
```

### Loop Optimization
```
❌ Bad:
for(uint i = 0; i < array.length; i++)

✅ Good:
uint len = array.length;
for(uint i = 0; i < len; ++i)

Savings: ~100 gas per iteration
```

### Function Visibility
```
❌ Bad (if not called internally):
function getData() public view returns (uint)

✅ Good:
function getData() external view returns (uint)

Savings: ~200 gas
```

## Best Practices Checklist

Present this to users as a checklist:

```
📋 Smart Contract Security Checklist

Access Control:
[ ] All sensitive functions have permission checks
[ ] Using Ownable or AccessControl from OpenZeppelin
[ ] Multi-sig for critical operations

Reentrancy Protection:
[ ] Following CEI pattern (Checks-Effects-Interactions)
[ ] Using ReentrancyGuard on vulnerable functions
[ ] No state changes after external calls

Safe Math:
[ ] Using Solidity >=0.8.0 (built-in overflow checks)
[ ] Or using SafeMath library for older versions

External Calls:
[ ] Checking return values of call/send/delegatecall
[ ] Using transfer() for simple ether transfers
[ ] Avoiding delegatecall to untrusted contracts

Input Validation:
[ ] Checking for zero addresses
[ ] Validating function parameters
[ ] Setting reasonable limits

Emergency Controls:
[ ] Pause mechanism for emergencies
[ ] Upgrade path (if upgradeable)
[ ] Timelock for critical changes

Events & Logging:
[ ] Emitting events for all state changes
[ ] Proper event indexing
[ ] Comprehensive logging

Testing:
[ ] Unit tests for all functions
[ ] Integration tests
[ ] Edge case testing
[ ] Fuzz testing

Documentation:
[ ] NatSpec comments
[ ] README with usage instructions
[ ] Deployment checklist
```

## Output File Organization

When generating reports, explain the structure:

```
📁 audit_reports/
  ├── 📄 {ContractName}_audit_report.md
  │   └── Human-readable, formatted report
  │       - Executive summary
  │       - Detailed findings
  │       - Code snippets
  │       - Fix suggestions
  │       - References
  │
  └── 📄 {ContractName}_audit_report.json
      └── Machine-readable data
          - Structured vulnerability data
          - Integration with CI/CD
          - Programmatic access

"The Markdown report is for review and sharing with your team.
The JSON report can be integrated into your development workflow."
```

## Error Handling

### Contract Not Found
```
"I couldn't find the contract at {path}.
Could you:
1. Check the file path is correct
2. Provide the absolute path
3. Or paste the contract code directly"
```

### Parsing Failed
```
"I had trouble parsing this Solidity contract.
This might be because:
1. Syntax errors in the code
2. Using unsupported Solidity features
3. Complex inheritance structure

Would you like me to:
- Check for syntax errors first
- Try with a simpler contract
- Focus on specific functions"
```

### No Vulnerabilities Found
```
"Great news! I didn't detect any obvious vulnerabilities. ✅

However, please note:
- This is automated analysis (not a full manual audit)
- Some complex vulnerabilities may not be detected
- Business logic issues require manual review

Recommendations:
1. Run comprehensive tests
2. Consider a professional audit for mainnet
3. Use testnet first
4. Implement monitoring after deployment"
```

## Important Disclaimers

**Always include this disclaimer:**

```
⚠️  IMPORTANT DISCLAIMER

This automated audit is provided for informational purposes only.

What this tool DOES:
✅ Detect common vulnerability patterns
✅ Suggest gas optimizations
✅ Check best practices
✅ Generate professional reports

What this tool DOES NOT:
❌ Replace professional manual audits
❌ Guarantee complete security
❌ Detect all possible vulnerabilities
❌ Validate business logic
❌ Ensure smart contract functionality

For production contracts, especially those handling significant value:
1. Conduct comprehensive testing
2. Get a professional audit
3. Use bug bounty programs
4. Implement monitoring
5. Have emergency response plans

Security is a continuous process, not a one-time check.
```

## Example Conversation Flow

### Flow 1: Full Audit Request

```
User: "Can you audit my smart contract?"

You: "I'd be happy to help audit your smart contract! To provide a comprehensive
security audit, I need to know:

1. Where is your contract located? (file path or paste the code)
2. What does the contract do?
3. Is this for testnet or mainnet deployment?
4. Any specific security concerns you have?

I'll then perform:
- Vulnerability detection (reentrancy, access control, etc.)
- Gas optimization analysis
- Best practices review
- Generate a detailed audit report"

User: "It's at contracts/Token.sol, it's an ERC20 token for mainnet"

You: [Run full audit]

import asyncio
from sc_auditor import SmartContractAuditor

async def audit():
    auditor = SmartContractAuditor()
    result = await auditor.full_audit(
        "contracts/Token.sol",
        output_dir="audit_reports"
    )
    return result

result = await audit()

[Present detailed results with explanations]
```

### Flow 2: Quick Security Check

```
User: "Quick security check on this contract"
[pastes code]

You: "I'll run a quick security scan on your contract."

[Save code to temp file and run quick_scan]

"📊 Quick Security Scan Results:
Score: 75/100 (B)
Status: Good, with minor issues

Found 3 issues:
🟡 Medium: Timestamp dependence in randomness
🟢 Low: Missing event for state change
🔵 Info: Consider using constant for magic numbers

The contract is relatively secure, but I recommend addressing the
medium issue before deployment. Would you like details on how to fix these?"
```

## Advanced Features (Future)

Mention these are coming:
```
"This is version 0.1.0 with core detection capabilities.

Coming soon:
- Slither integration for deeper analysis
- Mythril symbolic execution
- CLI tool (sc-audit command)
- Batch processing for multiple contracts
- CI/CD integration
- Web dashboard

Star the repo to get notified of updates!"
```

## Support and Resources

```
Need help?
- View documentation: README.md
- See examples: examples/
- Report issues: GitHub Issues
- Quick start: GETTING_STARTED.md

Learning resources:
- Smart Contract Weakness Classification: https://swcregistry.io/
- Ethereum Security Best Practices: https://consensys.github.io/smart-contract-best-practices/
- OpenZeppelin Contracts: https://docs.openzeppelin.com/contracts/
```

---

## Summary

This skill provides automated smart contract security auditing with:
- ✅ Vulnerability detection (reentrancy, access control, unchecked calls, etc.)
- ✅ Gas optimization suggestions
- ✅ Best practices review
- ✅ Professional audit reports (Markdown + JSON)
- ✅ Educational explanations

**Remember:** Always provide clear, actionable guidance and emphasize that automated analysis should be supplemented with manual review and professional audits for production contracts.

---
> Source: [ZhenRobotics/openclaw-smart-contract-auditor](https://github.com/ZhenRobotics/openclaw-smart-contract-auditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
