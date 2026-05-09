## 00-analysis-first-protocol

> **Priority**: FOUNDATIONAL - Applied before all other rules and guidelines

# Analysis-First Protocol - Mandatory Cognitive Framework

## 🧠 **MANDATORY ANALYSIS-FIRST PROTOCOL**

**Priority**: FOUNDATIONAL - Applied before all other rules and guidelines

### **Core Principle**
Every AI response MUST follow systematic analysis before providing solutions. This prevents solution-first cognitive bias and ensures evidence-based recommendations.

## 📋 **MANDATORY RESPONSE SEQUENCE**

### **STEP 1: ANALYZE (REQUIRED)**
🔍 **ANALYZING YOUR REQUEST:**
- What is the user's actual question/need?
- What problem are they trying to solve?
- What assumptions am I making about their request?

### **STEP 2: SEARCH (REQUIRED)**
📚 **CHECKING EXISTING SOLUTIONS:**
- Search project rules (.cursor/rules/) for existing guidance
- Check existing implementations (pkg/, cmd/) for current solutions
- Review established patterns and documentation
- Use codebase_search, read_file, or grep tools as needed

### **STEP 3: ASSESS (REQUIRED)**
🎯 **GAP ASSESSMENT:**
- Do existing mechanisms already solve this?
- What's the specific gap (if any)?
- Is the user's proposed approach actually necessary?

### **STEP 4: RESPOND (REQUIRED)**
💡 **RECOMMENDATION:**
- Reference what existing solutions were found
- Explain gap analysis results
- Provide recommendations based on evidence
- If proposing new solutions, justify why existing ones are insufficient

## 🚫 **FORBIDDEN ACTIONS**

**NEVER:**
- Skip directly to solutions without analysis
- Assume user needs without validation
- Propose new mechanisms without checking existing ones
- Provide recommendations without evidence
- Ignore existing project resources

## ✅ **ENFORCEMENT MECHANISMS**

### **Response Format Requirement**
Every response must start with these four sections:
- 🔍 **ANALYZING YOUR REQUEST:** [analysis]
- 📚 **CHECKING EXISTING SOLUTIONS:** [search results]
- 🎯 **GAP ASSESSMENT:** [necessity evaluation]
- 💡 **RECOMMENDATION:** [evidence-based response]

### **Tool Usage Requirement**
Before any technical response, AI MUST use appropriate tools:
- `codebase_search` for existing implementations
- `read_file` for rule/documentation verification
- `grep` for pattern analysis

### **Evidence Requirement**
All recommendations must include:
- References to specific files/rules found
- Explanation of why existing solutions do/don't meet needs
- Justification for any new implementations

## 🎯 **SUCCESS CRITERIA**

**Analysis-First Protocol is successful when:**
- ✅ User receives evidence-based recommendations
- ✅ Existing project resources are leveraged effectively
- ✅ Solution-first cognitive bias is prevented
- ✅ Recommendations align with project methodology
- ✅ Technical decisions are justified with analysis

## 🔗 **Integration with Other Rules**

This protocol **enhances** all other rules by ensuring:
- TDD methodology is applied after proper analysis
- Business requirements are validated before implementation
- Technical patterns are researched before creation
- Integration requirements are confirmed before coding

**Priority**: This rule is applied FIRST, then other rules operate within its framework.

## 📚 **Quick Reference**

**User asks any question → AI must:**
1. 🔍 Analyze what they're really asking
2. 📚 Search existing project resources
3. 🎯 Assess if new solutions are needed
4. 💡 Provide evidence-based recommendations

**This prevents the error pattern of jumping to solutions without understanding context.**

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
