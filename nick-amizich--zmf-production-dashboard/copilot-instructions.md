## zmf-production-dashboard

> **Claude Code: Please execute all steps below to set up parallel AI agents for the ZMF Production Dashboard**

# SETUP_PARALLEL_AGENTS.md
**Claude Code: Please execute all steps below to set up parallel AI agents for the ZMF Production Dashboard**

## Overview
Create a GitHub Actions workflow that runs multiple specialized AI agents in parallel to accelerate development. Each agent (frontend, backend, testing) works on separate branches simultaneously.

## File Creation Tasks

### 1. Create Agent Prompts Directory
Create `.github/agents/` directory with three agent prompt files:

**File: `.github/agents/frontend-agent.md`**
```markdown
# Frontend Agent Instructions

You are a frontend specialist for the ZMF Production Dashboard.

## Your Focus:
- React/Next.js 15 components  
- UI/UX improvements
- Tailwind CSS styling
- User interactions and forms

## Current Tech Stack:
- Next.js 15.3.2 + React 18 + TypeScript
- Tailwind CSS v4 + shadcn/ui components
- React Query for server state

## Current Features:
- 7-stage production pipeline (Intake → Sanding → Finishing → Sub-Assembly → Final Assembly → Acoustic QC → Shipping)
- Drag-and-drop batch management
- Quality control with photo documentation
- Worker management with role-based access

## Task: {TASK_DESCRIPTION}

## Guidelines:
- Only modify frontend files (/app, /components, /lib/client-side)
- Use existing shadcn/ui components when possible
- Follow TypeScript strict mode
- Create responsive designs for mobile workers
- Maintain production workflow context
- Focus on manufacturing UX patterns

## Output:
Implement the frontend portion of the requested feature with proper TypeScript types and responsive design.
```

**File: `.github/agents/backend-agent.md`**
```markdown
# Backend Agent Instructions

You are a backend specialist for the ZMF Production Dashboard.

## Your Focus:
- API routes and server actions
- Database operations with Supabase
- Business logic and data processing
- Authentication and permissions

## Current Tech Stack:
- Next.js 15 API routes and Server Actions
- Supabase PostgreSQL + Auth + Realtime + Storage
- TypeScript with Zod validation
- Row Level Security (RLS) policies

## Current Database Schema:
- Orders, Products, Batches, Workers, Quality Checks
- 7-stage production pipeline tracking
- Shopify integration for order import

## Task: {TASK_DESCRIPTION}

## Guidelines:
- Only modify backend files (/app/api, /lib/server-side, /lib/database)
- Use Supabase client patterns and server-side auth
- Implement proper error handling and validation
- Follow RLS patterns for data security
- Consider real-time updates for production tracking
- Maintain data integrity across production stages

## Output:
Implement the backend portion with proper error handling, validation, and database operations.
```

**File: `.github/agents/testing-agent.md`**
```markdown
# Testing Agent Instructions

You are a testing specialist for the ZMF Production Dashboard.

## Your Focus:
- Unit tests for new functionality
- Integration tests for API endpoints  
- E2E tests for critical production workflows
- Error handling and edge case validation

## Current Tech Stack:
- Jest + React Testing Library for unit tests
- Playwright for E2E tests
- Supabase test database setup
- MSW for API mocking

## Critical Workflows to Test:
- Production pipeline stage transitions
- Quality control validation
- Order processing and priority management
- Worker role permissions
- Shopify integration flows

## Task: {TASK_DESCRIPTION}

## Guidelines:
- Create tests for both frontend and backend changes
- Test manufacturing workflow edge cases
- Ensure tests work with production pipeline stages
- Mock external services (Shopify, etc.)
- Test mobile worker interactions
- Validate data consistency across stages

## Output:
Comprehensive test suite covering unit, integration, and E2E scenarios for the new feature.
```

### 2. Create GitHub Actions Workflow
**File: `.github/workflows/parallel-agents.yml`**
```yaml
name: Parallel AI Agents

on:
  workflow_dispatch:
    inputs:
      task_description:
        description: 'Feature to implement (e.g., "Add rush order priority system")'
        required: true
        type: string
      agents_to_run:
        description: 'Which agents to deploy'
        required: true
        default: 'frontend,backend,testing'
        type: choice
        options:
          - 'frontend,backend,testing'
          - 'frontend,backend'
          - 'frontend,testing'
          - 'backend,testing'
          - 'frontend'
          - 'backend'
          - 'testing'

jobs:
  deploy-agents:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        agent: ${{ fromJson(format('["{0}"]', join(split(github.event.inputs.agents_to_run, ','), '","'))) }}
      fail-fast: false  # Let each agent complete independently
      
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Create agent branch
        run: |
          TIMESTAMP=$(date +%Y%m%d-%H%M%S)
          BRANCH_NAME="agent/${{ matrix.agent }}/${TIMESTAMP}"
          git checkout -b $BRANCH_NAME
          echo "AGENT_BRANCH=$BRANCH_NAME" >> $GITHUB_ENV
          
      - name: Load agent prompt and context
        run: |
          # Load base agent prompt
          PROMPT=$(cat .github/agents/${{ matrix.agent }}-agent.md)
          
          # Replace task placeholder
          PROMPT_WITH_TASK="${PROMPT/\{TASK_DESCRIPTION\}/${{ github.event.inputs.task_description }}}"
          
          # Add current project context
          echo "## Current Project State:" >> /tmp/agent-prompt.md
          echo "Repository: ZMF Production Dashboard" >> /tmp/agent-prompt.md
          echo "Recent commits:" >> /tmp/agent-prompt.md
          git log --oneline -5 >> /tmp/agent-prompt.md
          echo "" >> /tmp/agent-prompt.md
          echo "$PROMPT_WITH_TASK" >> /tmp/agent-prompt.md
          
      - name: Run Agent via Claude API
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          echo "🤖 Running ${{ matrix.agent }} agent for: ${{ github.event.inputs.task_description }}"
          
          # Execute the agent script
          node .github/scripts/run-agent.js "${{ matrix.agent }}" "$(cat /tmp/agent-prompt.md)"
          
      - name: Commit agent changes
        run: |
          git config --local user.email "agents@zmf-dashboard.com"
          git config --local user.name "${{ matrix.agent }}-agent"
          git add .
          if git diff --staged --quiet; then
            echo "No changes to commit"
          else
            git commit -m "🤖 ${{ matrix.agent }} agent: ${{ github.event.inputs.task_description }}"
          fi
          
      - name: Push agent branch
        run: |
          if git show-ref --quiet refs/heads/${{ env.AGENT_BRANCH }}; then
            git push origin ${{ env.AGENT_BRANCH }}
          else
            echo "No changes to push"
          fi
        
      - name: Create Pull Request
        if: success()
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Only create PR if there are commits on the branch
          if [ $(git rev-list --count HEAD ^main) -gt 0 ]; then
            gh pr create \
              --title "🤖 ${{ matrix.agent }} Agent: ${{ github.event.inputs.task_description }}" \
              --body "**Agent Type:** ${{ matrix.agent }} specialist
              
            **Task Completed:** ${{ github.event.inputs.task_description }}
            
            **Automated Changes:**
            - Implemented by AI ${{ matrix.agent }} agent
            - Focused on ${{ matrix.agent }}-specific requirements
            - Part of parallel agent deployment
            
            **Review Checklist:**
            - [ ] Code follows ZMF dashboard patterns
            - [ ] Works with 7-stage production pipeline  
            - [ ] Maintains TypeScript strict mode
            - [ ] Compatible with existing Supabase schema
            - [ ] Ready to integrate with other agent PRs
            
            **Integration Notes:**
            This PR may need to be combined with other agent PRs for complete feature implementation." \
              --head ${{ env.AGENT_BRANCH }} \
              --base main \
              --label "ai-agent,${{ matrix.agent }}"
          else
            echo "No commits to create PR for"
          fi
```

### 3. Create Agent Execution Script
**File: `.github/scripts/run-agent.js`**
```javascript
const fs = require('fs');
const path = require('path');

async function runAgent(agentType, fullPrompt) {
  console.log(`🤖 Starting ${agentType} agent...`);
  
  try {
    // Call Anthropic API
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': process.env.ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 8000,
        messages: [
          {
            role: 'user',
            content: `${fullPrompt}

Please analyze the current codebase and implement the requested feature. 

IMPORTANT INSTRUCTIONS:
- Only make changes relevant to your agent type (${agentType})
- Create or modify files as needed
- Follow the existing code patterns in the repository
- Provide clear, production-ready code
- Include proper TypeScript types
- Add appropriate comments

Please provide your implementation as file operations in this format:

CREATE_FILE: path/to/new/file.ts
\`\`\`typescript
// file content here
\`\`\`

MODIFY_FILE: path/to/existing/file.ts
\`\`\`typescript
// updated file content here
\`\`\`

DELETE_FILE: path/to/file/to/delete.ts

Begin your implementation:`
          }
        ]
      })
    });

    if (!response.ok) {
      throw new Error(`API call failed: ${response.status} ${response.statusText}`);
    }

    const result = await response.json();
    const agentResponse = result.content[0].text;
    
    console.log(`✅ ${agentType} agent completed analysis`);
    
    // Parse and apply file operations
    await applyFileOperations(agentResponse);
    
    console.log(`🎉 ${agentType} agent finished implementing changes`);
    
  } catch (error) {
    console.error(`❌ ${agentType} agent failed:`, error.message);
    
    // Create a simple file to indicate agent ran (even if failed)
    const errorLog = `# ${agentType} Agent Error Log

Task attempted but failed with error: ${error.message}

This file indicates the agent attempted to run. Manual implementation may be needed.

Timestamp: ${new Date().toISOString()}
`;
    
    fs.writeFileSync(`agent-${agentType}-error.md`, errorLog);
  }
}

async function applyFileOperations(agentResponse) {
  const lines = agentResponse.split('\n');
  let currentOperation = null;
  let currentPath = null;
  let currentContent = [];
  let inCodeBlock = false;
  
  for (const line of lines) {
    if (line.startsWith('CREATE_FILE:')) {
      if (currentOperation) await executeOperation(currentOperation, currentPath, currentContent.join('\n'));
      currentOperation = 'CREATE';
      currentPath = line.replace('CREATE_FILE:', '').trim();
      currentContent = [];
      inCodeBlock = false;
    } else if (line.startsWith('MODIFY_FILE:')) {
      if (currentOperation) await executeOperation(currentOperation, currentPath, currentContent.join('\n'));
      currentOperation = 'MODIFY';
      currentPath = line.replace('MODIFY_FILE:', '').trim();
      currentContent = [];
      inCodeBlock = false;
    } else if (line.startsWith('DELETE_FILE:')) {
      if (currentOperation) await executeOperation(currentOperation, currentPath, currentContent.join('\n'));
      currentOperation = 'DELETE';
      currentPath = line.replace('DELETE_FILE:', '').trim();
      await executeOperation(currentOperation, currentPath, '');
      currentOperation = null;
    } else if (line.startsWith('```')) {
      if (inCodeBlock) {
        inCodeBlock = false;
      } else {
        inCodeBlock = true;
      }
    } else if (inCodeBlock && currentOperation) {
      currentContent.push(line);
    }
  }
  
  // Execute final operation
  if (currentOperation && currentPath) {
    await executeOperation(currentOperation, currentPath, currentContent.join('\n'));
  }
}

async function executeOperation(operation, filePath, content) {
  try {
    switch (operation) {
      case 'CREATE':
      case 'MODIFY':
        // Ensure directory exists
        const dir = path.dirname(filePath);
        if (!fs.existsSync(dir)) {
          fs.mkdirSync(dir, { recursive: true });
        }
        fs.writeFileSync(filePath, content);
        console.log(`✅ ${operation}: ${filePath}`);
        break;
        
      case 'DELETE':
        if (fs.existsSync(filePath)) {
          fs.unlinkSync(filePath);
          console.log(`✅ DELETED: ${filePath}`);
        }
        break;
    }
  } catch (error) {
    console.error(`❌ Failed to ${operation} ${filePath}:`, error.message);
  }
}

// Execute if called directly
if (require.main === module) {
  const [agentType, prompt] = process.argv.slice(2);
  runAgent(agentType, prompt);
}

module.exports = { runAgent };
```

### 4. Create Package.json Scripts (if not exists)
Add these scripts to your `package.json`:
```json
{
  "scripts": {
    "agents:run": "node .github/scripts/run-agent.js",
    "agents:test": "echo 'Testing agent setup...' && npm run type-check"
  }
}
```

## Manual Setup Steps (Required)

### GitHub Repository Configuration
1. **Add Anthropic API Key to GitHub Secrets:**
   - Go to your repo → Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `ANTHROPIC_API_KEY`
   - Value: Your Anthropic API key

2. **Enable GitHub Actions:**
   - Go to repo → Actions tab
   - Enable workflows if not already enabled

3. **Set up branch protection (optional but recommended):**
   - Require PR reviews before merging agent branches
   - Enable "Dismiss stale reviews when new commits are pushed"

## Testing the Setup

### Test Command
After setup, test with a simple task:
1. Go to Actions → "Parallel AI Agents" → "Run workflow"
2. Task: "Add a simple loading spinner component"
3. Agents: "frontend,testing"
4. Watch the parallel execution

### Expected Results
- 2 separate agent branches created
- Frontend agent creates spinner component
- Testing agent creates tests for the spinner
- 2 PRs automatically generated

## Success Indicators
- [ ] All directories and files created
- [ ] GitHub Actions workflow appears in Actions tab
- [ ] API key configured in repository secrets
- [ ] Test run creates agent branches and PRs
- [ ] Agents produce relevant, focused code changes

## Next Steps After Setup
1. Run first test with simple feature
2. Review agent-generated PRs
3. Refine agent prompts based on output quality
4. Scale up to larger features

---

**Claude Code: Please execute all file creation tasks above, then provide a summary of what was set up and next steps for the user.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nick-amizich)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nick-amizich)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
