## kaiyantool

> kaiyanTool is an AI video content creation platform

# kaiyanTool Cursor Rules - Long-Running Agent Configuration

## Project Overview
kaiyanTool is an AI video content creation platform
- Backend: Express + Prisma + TypeScript (port 3001)
- Frontend: React 19 + Vite + Tailwind (port 3000)
- Core: Auth, Projects, Scripts, Episodes, Scenes, Storylines, Storyboards, Image/Video Generation, TTS, AI Providers

## Anthropic Best Practices
This configuration follows the article "Effective harnesses for long-running agents" from Anthropic Engineering.

### Two-Part Solution
1. **Initializer Agent**: First run - sets up environment, feature list, progress file
2. **Coding Agent**: Subsequent runs - makes incremental progress one feature at a time

## Existing Implementation
The project already has a complete long-running agent system:
- `apps/api/src/agents/long-running-orchestrator.ts` - Main orchestrator
- `apps/api/src/agents/initializer.agent.ts` - Initializer agent
- `apps/api/src/agents/coding.agent.ts` - Coding agent
- `apps/api/src/services/progress-tracking.service.ts` - Progress tracking
- `apps/api/src/services/feature-list.service.ts` - Feature list management

## Agent Work Mode

### Session Start (ALWAYS DO THIS)
1. Run: `pwd` to confirm working directory
2. Read: `claude-progress.txt` to understand current state
3. Read: `feature_list.json` to see what needs to be done
4. Run: `git log --oneline -5` to see recent commits
5. Start dev servers if needed: `./init.sh`

### Feature Implementation (ONE AT A TIME)
1. Choose ONE feature from feature_list.json with `passes: false`
2. Implement the feature
3. Test the feature end-to-end
4. Update feature_list.json: set `passes: true` for completed feature
5. Run: `git add -A && git commit -m "Feature: <id> - <description>"`
6. Update claude-progress.txt with progress

### Using the Built-in Agent API
You can also use the LongRunningOrchestrator API:
```typescript
import { createLongRunningOrchestrator } from './src/agents/long-running-orchestrator';

// Initialize
const orchestrator = await createLongRunningOrchestrator({
  project_id: 'your-project-id',
  user_id: 'user-id',
  create_git_commits: true,
});

// Start new project
await orchestrator.startNewProject({
  task_description: 'Build an AI image generator',
}, 'task-id');

// Run coding session
await orchestrator.runCodingSession({
  max_features: 1,
}, 'task-id');

// Get status
const status = await orchestrator.getProjectStatus();
```

### Testing Requirements
- API tests: `cd apps/api && npm test`
- TypeScript check: `cd apps/api && npx tsc --noEmit`
- Lint: `cd apps/api && npm run lint`
- Verify frontend loads: http://localhost:3000
- Verify API works: http://localhost:3001/api/health

## Prohibited Behaviors

### DO NOT
- ❌ Never try to implement multiple features at once
- ❌ Never mark a feature as passes: true without proper testing
- ❌ Never leave the environment in a broken state
- ❌ Never skip the session start steps
- ❌ Never make large commits without incremental progress
- ❌ Never declare job done prematurely

### Anti-Patterns to Avoid
- One-shot implementation (trying to do everything at once)
- Premature completion (declaring job done before all features pass)
- Leaving undocumented/broken code
- Skipping git commits

## Environment Files

### Required Files (MUST EXIST)
- `feature_list.json` - Feature tracking with passes status
- `claude-progress.txt` - Progress log
- `init.sh` - Dev server startup script
- `agent-start.sh` - Agent launch script

### After Each Session
- [ ] Update feature_list.json with completed features
- [ ] Update claude-progress.txt with progress
- [ ] Make git commit with descriptive message
- [ ] Ensure no bugs in current state

## Git Workflow
```
Feature: auth-001 - Add user registration functionality
Feature: project-002 - Implement project list pagination
Feature: image-001 - Add AI image generation feature
```

## Common Agent Failure Modes & Solutions

| Problem | Solution |
|---------|----------|
| Agent declares victory too early | Use feature_list.json with passes status |
| Agent leaves broken state | Always run tests before marking passes: true |
| Agent skips git commits | Commit after each feature |
| Agent can't resume work | Read progress files at session start |

## Success Criteria
- Feature passes status accurately reflects implementation
- Git history shows incremental progress
- No broken states between sessions
- Tests pass for implemented features

---
> Source: [xiashao/KaiyanTool](https://github.com/xiashao/KaiyanTool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
