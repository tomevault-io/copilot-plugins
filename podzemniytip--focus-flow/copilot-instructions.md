## focus-flow

> Pomodoro timer with habit tracking, daily goals, and productivity statistics. Helps maintain focus and track progress over time.


# Focus Flow - Pomodoro Timer & Productivity Tracker

## Description
Focus Flow is a comprehensive productivity tool that combines:
- **Pomodoro Timer**: Classic 25/5 minute focus/break cycles with customizable durations
- **Habit Tracker**: Track daily habits and build streaks
- **Daily Goals**: Set and complete daily objectives
- **Statistics**: Visualize your productivity trends over time
- **Offline Storage**: All data saved locally using localStorage

## Instructions

When the user wants to:
- Start a focus session, Pomodoro timer, or work session
- Take a break or start a break timer
- Track habits, daily goals, or productivity
- View focus statistics or productivity stats
- Manage their time or improve concentration

Call the `run_js` tool with the following parameters:
- data: A JSON string with the following fields:
  - action: String (optional). One of: "focus", "break", "stats", "habits", "goals". Default: "focus"
  - duration: Number (optional). Timer duration in minutes. Default: 25 for focus, 5 for break
  - customMessage: String (optional). A motivational message or goal for the session

## Examples

User: "Start a 25 minute focus session"
→ Call run_js with: `{"action": "focus", "duration": 25}`

User: "I need a break"
→ Call run_js with: `{"action": "break", "duration": 5}`

User: "Show my productivity stats"
→ Call run_js with: `{"action": "stats"}`

User: "Help me track my habits"
→ Call run_js with: `{"action": "habits"}`

User: "Let me focus on writing code for 50 minutes"
→ Call run_js with: `{"action": "focus", "duration": 50, "customMessage": "Writing code"}`

## Features
- ⏱️ Customizable timer durations
- 🔔 Audio notifications when sessions complete
- 📊 Visual progress tracking with animations
- 🎯 Daily goal management
- 📈 Long-term statistics and trends
- 🌙 Dark mode support (auto-detects system preference)
- 💾 Persistent storage across sessions
- 📱 Mobile-optimized touch interface
- 🎨 Beautiful gradient design with smooth animations

---
> Source: [podzemniytip/focus-flow](https://github.com/podzemniytip/focus-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
