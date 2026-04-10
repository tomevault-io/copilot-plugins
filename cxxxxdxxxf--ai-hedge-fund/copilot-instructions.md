## ai-hedge-fund

> Your agents are currently **IDLE** because they haven't been added to a flow yet. Here's how to get them working:

# Quick Start: Activate Your Agents 🚀

Your agents are currently **IDLE** because they haven't been added to a flow yet. Here's how to get them working:

## Step-by-Step Activation

### Step 1: Open Your Flow Canvas
1. Go to `http://localhost:5173`
2. You should see a canvas with a **"Portfolio Analyzer"** node (or create a new flow)

### Step 2: Add Agents to Your Flow

**Method 1: Right Sidebar (Easiest)**
1. **Open the right sidebar**: Click the right sidebar icon (top right) or press `⌘ + I` (Mac) / `Ctrl + I` (Windows)
2. **Find "Analysts" section**: Scroll down to see all your agents
3. **Click the "+" button** next to any agent you want to activate:
   - Value Composite
   - Growth Composite
   - Momentum
   - Mean Reversion
   - Or any others you want
4. **The agent will appear on your canvas** as a new node

**Method 2: Click on Agent Name**
- Simply **click on the agent name** in the right sidebar
- It will be added to your canvas automatically

### Step 3: Connect the Agents

Your agents need to be connected in a flow:

```
Portfolio Analyzer → Agents → Portfolio Manager
```

1. **Connect Portfolio Analyzer to Agents**:
   - Hover over the Portfolio Analyzer node
   - Drag from the **output handle** (right side) to each agent's **input handle** (left side)
   - Or click and drag to create connections

2. **Connect Agents to Portfolio Manager**:
   - Drag from each agent's **output handle** (right side)
   - To the Portfolio Manager's **input handle** (left side)

3. **Add Portfolio Manager** (if not present):
   - From right sidebar → "End Nodes" → Click "+" on "Portfolio Manager"

### Step 4: Configure Your Portfolio

1. **Click on the "Portfolio Analyzer" node**
2. **Set Available Cash**: Enter your starting capital (e.g., 100000)
3. **Add Stock Positions**:
   - Click the **"+" button** to add a position
   - Enter:
     - **Ticker**: AAPL, MSFT, NVDA, TSLA, or GOOGL (free)
     - **Quantity**: Number of shares
     - **Trade Price**: Price per share
   - Add multiple positions as needed

### Step 5: Start Your Agents! 🎯

**Option A: Single Run (Live Analysis)**
1. Make sure **"Single Run"** is selected in Portfolio Analyzer
2. Click the **Play button (▶️)** on the Portfolio Analyzer node
3. Or press **`⌘ + Enter`** / **`Ctrl + Enter`**

**Option B: Backtest (Historical Training)**
1. Switch to **"Backtest"** mode in Portfolio Analyzer
2. Set **Start Date** and **End Date**
3. Click **Play (▶️)**

### Step 6: Watch Them Work!

**In the Flow View:**
- Agents will change from **IDLE** (gray) → **IN_PROGRESS** (yellow) → **COMPLETE** (green)
- Click on any agent to see their analysis

**In the Dashboard:**
- Click the **Dashboard icon (📊)** in the top bar
- Watch the Agent Roster update in real-time:
  - Status changes from IDLE to IN_PROGRESS
  - Shows which ticker each agent is analyzing
  - Displays progress messages

## Example: Activate 3 Agents

1. **Add 3 agents**:
   - Value Composite (from right sidebar)
   - Growth Composite (from right sidebar)
   - Momentum (from right sidebar)

2. **Connect them**:
   - Portfolio Analyzer → Value Composite → Portfolio Manager
   - Portfolio Analyzer → Growth Composite → Portfolio Manager
   - Portfolio Analyzer → Momentum → Portfolio Manager

3. **Configure portfolio**:
   - Cash: $100,000
   - Add AAPL: 100 shares @ $150

4. **Click Play** → Watch all 3 agents start working!

## Troubleshooting

### "Agents still showing IDLE"
- ✅ Make sure agents are **added to the canvas** (not just in the sidebar)
- ✅ Verify agents are **connected** to Portfolio Analyzer
- ✅ Check that Portfolio Analyzer has **tickers configured**
- ✅ Ensure Portfolio Manager is connected

### "Nothing happens when I click Play"
- ✅ Check you have at least one **API key** configured (Settings → API Keys)
- ✅ Verify **tickers are entered** in Portfolio Analyzer
- ✅ Make sure agents are **properly connected** (see connections above)

### "Agents show ERROR"
- ✅ Check API keys are valid and have credits
- ✅ Verify ticker symbols are correct
- ✅ Check the bottom panel for error messages

## Quick Checklist

Before clicking Play, make sure:
- [ ] At least one agent is on the canvas
- [ ] Agents are connected: Portfolio Analyzer → Agents → Portfolio Manager
- [ ] Portfolio Analyzer has at least one ticker configured
- [ ] At least one API key is configured (Settings → API Keys)
- [ ] Portfolio Manager node exists and is connected

## Visual Guide

```
┌─────────────────────┐
│ Portfolio Analyzer  │ ← Configure with stocks here
│   (Pie Chart Icon)  │
└──────────┬──────────┘
           │
           ├──→ ┌─────────────────┐
           │    │ Value Composite  │ ← Add from right sidebar
           │    └────────┬─────────┘
           │             │
           ├──→ ┌────────┴─────────┐
           │    │ Growth Composite  │ ← Add from right sidebar
           │    └────────┬──────────┘
           │             │
           └──→ ┌────────┴──────────┐
                │ Portfolio Manager │ ← Add from right sidebar
                └───────────────────┘
```

**Once connected, click Play on Portfolio Analyzer and watch your agents come to life!** 🎉

---

**Your agents are ready - they just need to be added to a flow and connected!** Once you do that and click Play, you'll see them change from IDLE to IN_PROGRESS to COMPLETE in real-time! 🚀

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CxxxxDxxxF)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CxxxxDxxxF)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
