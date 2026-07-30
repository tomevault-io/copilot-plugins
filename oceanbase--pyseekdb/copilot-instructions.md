## pyseekdb

> Act as a senior technical architect and documentation engineer. Perform a comprehensive review of this Pull Request and generate a detailed design document. Follow this exact structure:

Act as a senior technical architect and documentation engineer. Perform a comprehensive review of this Pull Request and generate a detailed design document. Follow this exact structure:

## 🔍 COMPREHENSIVE PR REVIEW REPORT

### 📊 PR Overview Statistics
- Total files changed: {auto-calculate}
- Lines of code: +{added}/-{deleted}
- Primary change type: [Feature/Bugfix/Refactor/Performance/Docs]

---

## 📝 DESIGN DOCUMENTATION

### 1. Background & Problem Statement
**Business Requirements:**
{Analyze business problems based on PR title and description}

**Technical Pain Points:**
{Identify technical issues before the changes}

**Design Objectives:**
- Objective 1: {inferred from code changes}
- Objective 2: {inferred from code changes}
- Objective 3: {inferred from code changes}

### 2. Architecture Design Changes
**Design Philosophy:**
{Summarize core design patterns and technical choices}

**Architecture Diagram (Text Description):**
Module Hierarchy:

Upper Layer: {affected upstream modules}

Core: {main changed modules}

Lower Layer: {affected downstream modules}

Data Flow:
[Input] → [Processing Logic] → [Output]

**Key Design Decisions:**
1. Decision A: {analyze critical design choices in code}
2. Decision B: {analyze critical design choices in code}
3. Decision C: {analyze critical design choices in code}

### 3. Detailed Change Analysis

#### 🔧 Core Changed Modules
```plaintext
Module Name: {primary module}
- Change Type: [Add/Modify/Refactor/Delete]
- Impact Scope: {affected modules}
- Complexity: [Low/Medium/High]
- Risk Level: [Low/Medium/High]

4. ⚠️ Compatibility Warnings
🚨 Breaking Changes
{List all backward-incompatible changes}

API Changes:

Interface: OldInterface.method() → NewInterface.method()

Impact: {callers that need modification}

Migration Advice: {how to upgrade smoothly}

Data Model Changes:

Schema/Field Changes: {specific description}

Data Migration Required: {yes/no and details}

Compatibility Window: {suggested dual-version runtime}

Configuration Changes:

Old Config: config.old_key

New Config: config.new_key

Default Value Changes: {if any}

⚡ Potential Risk Points
{List changes that might cause issues}

Risk Point 1: {specific risk description}

Impact: {potential affected scope}

Mitigation: {suggested solutions}

Risk Point 2: {specific risk description}

5. 🎯 New Feature Usage Examples
📚 API Usage Demos
# Import new module
import {new_module}

# Basic Usage Example
Example 1: {most common usage}

# Advanced Configuration Example
Example 2: {complex scenario usage}

# Error Handling Example
Example 3: {exception handling}

---
> Source: [oceanbase/pyseekdb](https://github.com/oceanbase/pyseekdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
