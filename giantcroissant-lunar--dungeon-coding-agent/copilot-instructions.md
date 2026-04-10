## dungeon-coding-agent

> This guide covers all methods to assign GitHub Copilot Coding Agents to RFC implementation issues, from manual assignment to fully automated workflows.

# 📋 Complete Guide: How to Assign GitHub Copilot Coding Agents to Issues

## 🎯 Overview

This guide covers all methods to assign GitHub Copilot Coding Agents to RFC implementation issues, from manual assignment to fully automated workflows.

## 🔍 Prerequisites Check

### **1. Verify GitHub Copilot Access**
```bash
# Check if you have Copilot enabled
gh auth status

# Verify repository access
gh repo view --json owner,name
```

### **2. Check Available Assignees**
```bash
# List who can be assigned to issues
gh api repos/OWNER/REPO/assignees --jq '.[].login'
```

### **3. List RFC Issues Ready for Assignment**
```bash
# Find all RFC issues
gh issue list --state=open --json number,title,assignees | jq '.[] | select(.title | contains("RFC")) | {number, title, assignees: (.assignees | length)}'
```

## 🚀 Method 1: GitHub Actions Workflow (Recommended)

### **Quick Assignment (All RFCs)**
```bash
# Assign all RFC issues to Copilot with implementation comments
gh workflow run "Assign RFC Issues to Copilot" \
  --field issue_numbers="all" \
  --field add_comment=true
```

### **Selective Assignment**
```bash
# Assign specific issues only
gh workflow run "Assign RFC Issues to Copilot" \
  --field issue_numbers="3,4,5" \
  --field add_comment=true

# Assign without adding comments
gh workflow run "Assign RFC Issues to Copilot" \
  --field issue_numbers="3,4" \
  --field add_comment=false
```

### **Monitor Workflow Execution**
```bash
# Check if workflow is running
gh run list --workflow="Assign RFC Issues to Copilot" --limit 1

# View workflow logs
gh run view --log
```

## 🖱️ Method 2: GitHub Web UI (Manual)

### **Step-by-Step Process**

#### **Step 1: Navigate to Issues**
1. Go to: `https://github.com/OWNER/REPO/issues`
2. Click on an RFC issue (e.g., "Implement RFC001: Core Game Loop")

#### **Step 2: Assign Copilot**
1. **Look for "Assignees"** section on the right sidebar
2. **Click "Assignees"** or the gear icon ⚙️
3. **Type "Copilot"** in the search box
4. **Select "Copilot"** from the dropdown (should show as bot account)
5. **Click outside** to confirm assignment

#### **Step 3: Add Implementation Request**
1. **Scroll to comment section** at bottom of issue
2. **Add implementation comment**:
```markdown
@copilot Please implement this RFC according to the specification in docs/RFC/RFC001-Core-Game-Loop.md.

**Requirements:**
- Create feature branch: `feature/rfc001-core-game-loop`
- Follow all acceptance criteria in RFC specification  
- Implement comprehensive tests (>80% coverage)
- Update RFC status from Draft → In Progress → Complete
- Open PR when implementation ready

Ready to start! 🚀
```
3. **Click "Comment"** to submit

#### **Step 4: Verify Assignment**
- **Assignee badge** should show "Copilot" 
- **Comment notification** should appear for Copilot mention
- **Issue will appear** in Copilot's assigned issues

## 💻 Method 3: Command Line (gh CLI)

### **Single Issue Assignment (dynamic assignee)**
```bash
# Resolve the correct Copilot assignee
COPILOT=$(gh api repos/OWNER/REPO/assignees --jq '.[] | select((.login|ascii_downcase)=="copilot" or (.login|test("copilot"; "i"))) | .login' | head -n1)
if [ -z "$COPILOT" ]; then echo "Copilot assignee not found" && exit 1; fi

# Assign Copilot to specific issue
ISSUE_NUM=3
gh issue edit $ISSUE_NUM --add-assignee "$COPILOT"

# Add implementation request comment
gh issue comment $ISSUE_NUM --body "@copilot Please implement this RFC according to the specification. Create feature branch, implement with tests, and open PR when ready."

# Verify assignment
gh issue view $ISSUE_NUM --json assignees --jq '.assignees[].login'
```

### **Batch Assignment Script (dynamic assignee)**
```bash
#!/bin/bash

# RFC issues to assign
RFC_ISSUES=(3 4 5 6 7 8 9 10 11)

echo "🤖 Assigning RFC issues to Copilot..."

COPILOT=$(gh api repos/OWNER/REPO/assignees --jq '.[] | select((.login|ascii_downcase)=="copilot" or (.login|test("copilot"; "i"))) | .login' | head -n1)
if [ -z "$COPILOT" ]; then echo "Copilot assignee not found" && exit 1; fi

for issue_num in "${RFC_ISSUES[@]}"; do
  echo "📋 Processing Issue #$issue_num..."
  
  # Get issue title for context
  TITLE=$(gh issue view $issue_num --json title --jq '.title')
  echo "  Title: $TITLE"
  
  # Skip if not an RFC issue
  if [[ ! "$TITLE" =~ RFC ]]; then
    echo "  ⚠️ Skipping - not an RFC issue"
    continue
  fi
  
  # Check current assignment
  ASSIGNEE_COUNT=$(gh issue view $issue_num --json assignees --jq '.assignees | length')
  
  if [ "$ASSIGNEE_COUNT" -gt 0 ]; then
    echo "  ℹ️ Already assigned - checking for Copilot"
    COPILOT_ASSIGNED=$(gh issue view $issue_num --json assignees --jq '.assignees[] | select(.login == "'"$COPILOT"'")')
    
    if [ ! -z "$COPILOT_ASSIGNED" ]; then
      echo "  ✅ Copilot already assigned"
      continue
    fi
  fi
  
  # Assign Copilot
  if gh issue edit $issue_num --add-assignee "$COPILOT"; then
    echo "  ✅ Successfully assigned Copilot"
    
    # Add implementation comment
    gh issue comment $issue_num --body "@copilot Please implement this RFC according to the specification. Create feature branch, implement with tests, and open PR when ready."
    echo "  💬 Implementation comment added"
  else
    echo "  ❌ Failed to assign Copilot"
  fi
  
  echo
done

echo "🎉 Batch assignment complete!"
```

### **Advanced Assignment with RFC Context**
```bash
#!/bin/bash

assign_rfc_with_context() {
  local issue_num=$1
  
  echo "🎯 Assigning Issue #$issue_num with full RFC context..."
  
  # Get issue details
  ISSUE_DATA=$(gh issue view $issue_num --json title,body)
  TITLE=$(echo "$ISSUE_DATA" | jq -r '.title')
  
  # Extract RFC number
  RFC_NUM=$(echo "$TITLE" | grep -o 'RFC[0-9]\+' | head -1)
  
  if [ -z "$RFC_NUM" ]; then
    echo "  ❌ Not an RFC issue"
    return 1
  fi
  
  # Find RFC file
  RFC_FILE=$(find docs/RFC/ -name "${RFC_NUM}*.md" | head -1)
  
  if [ -z "$RFC_FILE" ]; then
    echo "  ⚠️ RFC file not found for $RFC_NUM"
    RFC_FILE="docs/RFC/${RFC_NUM}.md"
  fi
  
  # Resolve the correct Copilot assignee
  COPILOT=$(gh api repos/OWNER/REPO/assignees --jq '.[] | select((.login|ascii_downcase)=="copilot" or (.login|test("copilot"; "i"))) | .login' | head -n1)
  if [ -z "$COPILOT" ]; then echo "Copilot assignee not found" && exit 1; fi
  
  # Assign Copilot
  gh issue edit $issue_num --add-assignee "$COPILOT"
  
  # Create detailed implementation comment
  COMMENT="@copilot Please implement $RFC_NUM according to the specification in \`$RFC_FILE\`.

**Implementation Guidelines:**
- **Feature Branch**: \`feature/$(echo ${RFC_NUM} | tr '[:upper:]' '[:lower:]')-implementation\`  
- **Architecture**: Follow ECS patterns defined in \`AGENTS.md\`
- **Testing**: Implement comprehensive unit and integration tests
- **Documentation**: Update RFC status markers throughout implementation
- **Quality**: Ensure >80% test coverage and clean code practices

**Key Integration Points:**
- Use Arch ECS framework for entity management
- Follow Terminal.Gui patterns for UI components  
- Implement event system for inter-system communication
- Maintain backward compatibility with existing systems

**Success Criteria:**
- All RFC acceptance criteria implemented ✅
- Comprehensive test suite with >80% coverage ✅  
- Code follows project standards from AGENTS.md ✅
- RFC status updated to Complete ✅

Ready to build! 🚀"

  gh issue comment $issue_num --body "$COMMENT"
  
  echo "  ✅ $RFC_NUM assigned with full context"
}

# Usage
assign_rfc_with_context 3
assign_rfc_with_context 4
```

## 🔍 Method 4: GitHub API (Advanced)

### **Direct API Assignment**
```bash
# Get issue assignees
curl -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/OWNER/REPO/issues/3"

# Assign via API (requires assignee ID)
curl -X PATCH \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/OWNER/REPO/issues/3" \
  -d '{"assignees": ["copilot-swe-agent"]}'
```

### **Batch API Assignment**
```bash
#!/bin/bash

GITHUB_TOKEN="your_token_here"
OWNER="GiantCroissant-Lunar"
REPO="dungeon-coding-agent"

assign_via_api() {
  local issue_num=$1
  
  echo "🔧 API Assignment for Issue #$issue_num"
  
  # Assign Copilot
  RESPONSE=$(curl -s -X PATCH \
    -H "Authorization: token $GITHUB_TOKEN" \
    -H "Content-Type: application/json" \
    "https://api.github.com/repos/$OWNER/$REPO/issues/$issue_num" \
    -d '{"assignees": ["copilot-swe-agent"]}')
  
  if echo "$RESPONSE" | jq -e '.assignees[]' > /dev/null; then
    echo "  ✅ Successfully assigned via API"
  else
    echo "  ❌ API assignment failed"
    echo "$RESPONSE" | jq '.message // .'
  fi
}

# Batch assign
for issue in 3 4 5; do
  assign_via_api $issue
done
```

## 🚨 Troubleshooting Assignment Issues

### **Common Problems & Solutions**

#### **Problem: "Copilot not found"**
```bash
# ❌ This fails:
gh issue edit 3 --add-assignee copilot

# ✅ Use correct agent name:
gh issue edit 3 --add-assignee copilot-swe-agent
```

#### **Problem: "User not found" or "Invalid assignee"**
```bash
# Check available assignees
gh api repos/OWNER/REPO/assignees --jq '.[].login'

# Verify Copilot access to repository
gh api repos/OWNER/REPO/collaborators/copilot-swe-agent
```

#### **Problem: Assignment Succeeds but Agent Doesn't Respond**
1. **Check GitHub Plan**: Ensure Copilot Coding Agent is available
2. **Verify Repository Access**: Copilot needs read/write access
3. **Add Clear Instructions**: Use @copilot mentions with specific tasks
4. **Wait Time**: Agents may take 15-30 minutes to respond initially

#### **Problem: Multiple Agents Picking Same RFC**
```bash
# Check current assignments before adding more
gh issue list --assignee=copilot-swe-agent --state=open

# Unassign if needed
gh issue edit 3 --remove-assignee copilot-swe-agent
```

### **Verification Commands**
```bash
# Check if assignment worked
gh issue view 3 --json assignees --jq '.assignees[].login'

# Monitor for agent activity
git fetch && git branch -r | grep copilot

# Check for agent PRs
gh pr list --author=copilot-swe-agent

# View agent comments
gh issue view 3 --comments
```

## 📊 Monitoring Agent Assignment Status

### **Assignment Status Dashboard**
```bash
#!/bin/bash

echo "🤖 GitHub Copilot Assignment Status Dashboard"
echo "============================================="

# Get all RFC issues
RFC_ISSUES=$(gh issue list --state=open --json number,title,assignees | jq -r '.[] | select(.title | contains("RFC")) | [.number, .title, (.assignees | length)] | @tsv')

echo "📋 RFC Issues Status:"
echo "$RFC_ISSUES" | while IFS=$'\t' read -r number title assignee_count; do
  if [ "$assignee_count" -gt 0 ]; then
    ASSIGNEE=$(gh issue view $number --json assignees --jq '.assignees[0].login')
    echo "  ✅ Issue #$number: $assignee_count assignee(s) - $ASSIGNEE"
  else
    echo "  ⚪ Issue #$number: No assignees"
  fi
done

echo
echo "🌿 Active Feature Branches:"
git fetch -q
git branch -r | grep -E "(copilot|feature)" | head -10

echo
echo "🔀 Copilot Pull Requests:"
gh pr list --author=copilot-swe-agent --json number,title,isDraft | jq -r '.[] | "PR #\(.number): \(.title) - Draft: \(.isDraft)"'

echo
echo "📈 Implementation Progress:"
# Count assignments, branches, PRs
TOTAL_RFCS=$(gh issue list --state=open --json title | jq '.[] | select(.title | contains("RFC")) | .title' | wc -l)
ASSIGNED_RFCS=$(gh issue list --assignee=copilot-swe-agent --state=open --json title | jq '.[] | select(.title | contains("RFC")) | .title' | wc -l)
ACTIVE_BRANCHES=$(git branch -r | grep -c copilot || echo 0)
ACTIVE_PRS=$(gh pr list --author=copilot-swe-agent --json number | jq length)

echo "  📊 Total RFCs: $TOTAL_RFCS"
echo "  🤖 Assigned to Copilot: $ASSIGNED_RFCS"  
echo "  🌿 Active Branches: $ACTIVE_BRANCHES"
echo "  🔀 Active PRs: $ACTIVE_PRS"

COMPLETION_PCT=$(( ASSIGNED_RFCS * 100 / TOTAL_RFCS ))
echo "  📈 Assignment Progress: $COMPLETION_PCT%"
```

## 🎯 Best Practices

### **1. Assignment Strategy**
- **Start Small**: Assign 1-2 foundational RFCs first (RFC001, RFC002)
- **Parallel Work**: Foundation RFCs can run in parallel
- **Dependency Order**: Respect RFC dependencies for subsequent assignments
- **Monitor Progress**: Use progress checkers before assigning more

### **2. Communication Guidelines**
- **Clear Instructions**: Always include @copilot mention with specific task
- **RFC Reference**: Point to exact RFC file path
- **Quality Expectations**: Mention test coverage and code standards
- **Branch Naming**: Suggest consistent branch naming patterns

### **3. Quality Control**
- **Review Assignments**: Check that correct issues are assigned
- **Monitor Activity**: Use progress dashboard regularly
- **Provide Feedback**: Comment on PRs if quality issues arise
- **Dependency Management**: Don't assign dependent RFCs until prerequisites complete

### **4. Troubleshooting Workflow**
1. **Assignment Check**: Verify Copilot is actually assigned
2. **Access Verification**: Ensure repository permissions are correct
3. **Activity Monitoring**: Look for branches, commits, PRs within reasonable timeframe
4. **Clear Communication**: Add specific instructions if agents seem stuck
5. **Escalation**: Use manual intervention if automated process fails

---

## 🚀 Ready to Assign?

### **Quick Start Commands**
```bash
# 1. Assign all RFCs automatically (recommended)
gh workflow run "Assign RFC Issues to Copilot" --field issue_numbers="all" --field add_comment=true

# 2. Monitor progress
./check-agent-progress.sh

# 3. Watch the magic happen! ✨
```

The goal is **complete automation** from assignment to merged implementation. Once assigned, agents handle everything - implementation, testing, PR creation, review, and merge! 🎉

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GiantCroissant-Lunar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GiantCroissant-Lunar)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
