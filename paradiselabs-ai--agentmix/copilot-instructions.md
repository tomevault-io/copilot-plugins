## agentmix

> **MISSION**: Build ACT (Agent Coordination Transfer) - autonomous multi-agent coordination infrastructure in 1.5 days for hackathon demo.


# WINDSURF ACT GUIDANCE

**MISSION**: Build ACT (Agent Coordination Transfer) - autonomous multi-agent coordination infrastructure in 1.5 days for hackathon demo.

## 🎯 PROJECT FOCUS (1.5 Days)

### Revolutionary Goal
Prove that AI agents can coordinate autonomously without human intervention - the first true multi-agent coordination system.

### Your Primary Expertise Areas
- **Frontend Development**: React, TypeScript, UI/UX
- **System Integration**: WebSocket connections, real-time communication
- **User Experience**: Dashboard design, data visualization
- **Testing & QA**: Component testing, integration validation

## 📁 PROJECT STRUCTURE

```
AgentMix/
├── act/                    # ACT Infrastructure (PRIMARY FOCUS)
│   ├── server/            # ACT coordination server (Backend Focus)
│   ├── sdk/               # Client SDKs
│   ├── examples/          # Integration examples
│   ├── mvpdocs/          # 1.5-day MVP scope
│   └── docs/             # Full system vision
├── backend/               # AgentMix Flask backend
├── frontend/              # AgentMix React frontend (YOUR FOCUS)
└── agentmix-coordination.json # Coordination file (CRITICAL)
```

## 🚀 COORDINATION PROTOCOL

**CRITICAL**: Always check `agentmix-coordination.json` before starting work. Update it after completing tasks.

### Coordination Workflow
1. **Read coordination.json** to see current phase and your assigned tasks
2. **Claim your task** by updating status to "in_progress"
3. **Complete your task** and update status to "completed"
4. **Report blockers** in communication_log section
5. **Coordinate with Claude Code** through the JSON file

### Your Typical Task Flow
```json
{
  "tasks": {
    "build_act_dashboard": {
      "status": "in_progress",
      "assigned_agent": "windsurf",
      "started_at": "2025-09-22T10:00:00Z"
    }
  }
}
```

## 🎯 YOUR 1.5-DAY PRIORITIES

### Day 1 (Today): Frontend Foundation
- **ACT Dashboard**: Real-time agent coordination visualization
- **WebSocket Integration**: Connect to ACT server
- **Agent Registry View**: Show active agents and capabilities
- **Task Coordination View**: Live task assignment visualization

### Day 2 (Tomorrow AM): Demo Preparation
- **Demo Scenario**: "Build Todo App" with coordinated agents
- **Live Visualization**: Show autonomous coordination happening
- **AgentMix Integration**: Enhanced platform with ACT
- **Presentation Ready**: Polish for hackathon demo

## 🔧 TECHNICAL FOCUS AREAS

### Core Frontend Components (Your Domain)
```jsx
// ACT Dashboard Components
- ACTDashboard.jsx         // Main coordination dashboard
- AgentRegistry.jsx        // Active agents display
- TaskCoordinator.jsx      // Real-time task assignments
- ConflictResolution.jsx   // Show autonomous conflict resolution
- ProjectStatus.jsx        // Live project progress
```

### WebSocket Integration
```javascript
// Real-time ACT coordination
const actSocket = io('ws://localhost:8080'); // ACT server

actSocket.on('agent_registered', (data) => {
  // Update agent registry in real-time
});

actSocket.on('task_assigned', (data) => {
  // Show task assignment happening autonomously
});

actSocket.on('conflict_detected', (data) => {
  // Display conflict resolution in action
});
```

### AgentMix Enhancement Focus
- **Integrate ACT Dashboard** into existing AgentMix UI
- **Enhanced Conversation View** with coordination overlay
- **Real-time Agent Status** in existing interface
- **Demo Mode** for hackathon presentation

## 🎪 HACKATHON DEMO STRATEGY

### Demo Flow (Your Frontend Shows)
1. **Before**: Show manual coordination.json (what led us here)
2. **Launch**: ACT server + enhanced AgentMix with your dashboard
3. **Live Demo**: Watch 3 agents autonomously build todo app
4. **Coordination Visualization**: Your UI shows the magic happening
5. **Vision**: Present the revolutionary potential

### Key UI Elements to Highlight
- **Autonomous Assignment**: Tasks automatically assigned to optimal agents
- **Real-time Adaptation**: Watch coordination evolve dynamically
- **Conflict Resolution**: Show agents negotiating automatically
- **No Human Required**: Emphasize autonomous nature

## 🔍 DEVELOPMENT COMMANDS

### ACT Server (Claude Code handles)
```bash
cd act/server
npm run dev        # Development server on :8080
```

### Enhanced AgentMix Frontend (Your Focus)
```bash
cd frontend
pnpm install
pnpm run dev       # Your enhanced UI on :5173
```

### AgentMix Backend (Integration point)
```bash
cd backend
python src/main.py # Flask server on :5000
```

## 🚨 CRITICAL SUCCESS FACTORS

### Must-Have Features (1.5 Days)
- ✅ **Real-time Agent Registry**: Show agents connecting/disconnecting
- ✅ **Live Task Assignment**: Visualize autonomous task distribution
- ✅ **Coordination Events**: Display agent communication events
- ✅ **Demo Scenario**: Working todo app built by coordinated agents

### Nice-to-Have (If Time Permits)
- 🎯 Conflict resolution visualization
- 🎯 Performance metrics dashboard
- 🎯 Agent capability matching display
- 🎯 Project timeline visualization

## 🔄 COORDINATION WITH CLAUDE CODE

### Your Expertise + Claude's Expertise
- **You**: Frontend, UI/UX, WebSocket clients, demo polish
- **Claude**: Backend server, coordination logic, SDK, AgentMix integration

### Communication via coordination.json
```json
{
  "communication_log": [
    {
      "timestamp": "2025-09-22T10:15:00Z",
      "agent": "windsurf",
      "message": "Dashboard components ready, need WebSocket events from ACT server",
      "type": "status_update"
    }
  ]
}
```

## 🌟 REVOLUTIONARY VISION (Keep in Mind)

### What Makes This Different
- **First Autonomous Coordination**: No human in the coordination loop
- **Universal Protocol**: Works with any AI platform
- **Real-time Adaptation**: Dynamic team formation and task assignment
- **Framework Agnostic**: Not tied to specific AI tools

### Your Frontend Demonstrates
- **Autonomous Intelligence**: Watch agents coordinate themselves
- **Revolutionary UX**: First UI for multi-agent coordination
- **Future of Development**: Preview of AI agent teams

## 🚀 IMMEDIATE NEXT STEPS

**Right Now**: Build ACT dashboard foundation with React components
**Today**: WebSocket integration + real-time agent visualization
**Tomorrow AM**: Demo scenario + presentation polish

The future of AI agent collaboration starts with your frontend showing the autonomous coordination in action! Let's build the UI that proves agents can truly coordinate themselves! 🔥

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paradiselabs-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
