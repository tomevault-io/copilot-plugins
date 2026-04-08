## chess-coach

> This project implements a chess coaching application with AI agents that can analyze chess positions, suggest moves, and provide strategic advice. The system combines a modern React frontend with a Python-based AI agent backend.

# Chess Coach AI Agents

## Overview

This project implements a chess coaching application with AI agents that can analyze chess positions, suggest moves, and provide strategic advice. The system combines a modern React frontend with a Python-based AI agent backend.

## Architecture

### Frontend (React/Next.js)
- **Framework**: Next.js 15.3.2 with React 19
- **UI Library**: CopilotKit for AI integration
- **Chess Board**: react-chessboard for interactive chess interface
- **Game Logic**: chess.js for move validation and game state management
- **Styling**: Tailwind CSS

### Backend (Python)
- **Framework**: LlamaIndex with OpenAI integration
- **Agent**: Chess assistant with FEN position understanding
- **Server**: FastAPI with Uvicorn

## AI Agents

### Chess Assistant Agent

**Purpose**: Provides chess analysis, move suggestions, and strategic coaching

**Capabilities**:
- Analyzes chess positions using FEN notation
- Suggests optimal moves based on position evaluation
- Explains chess concepts and strategies
- Provides real-time analysis
- Offers coaching advice for improvement

**Technical Implementation**:
- **LLM**: OpenAI GPT-4.1
- **State Management**: FEN position strings for game state
- **Integration**: CopilotKit for real-time UI updates
- **Tools**: Currently configured for chess analysis (frontend/backend tools disabled)

**State Schema**:
```typescript
type ChessState = {
  position: string; // FEN notation of current board position
}
```

## Key Components

### ChessBoard Component
- **Library**: react-chessboard
- **Features**: Drag-and-drop piece movement, move validation
- **Integration**: Syncs with AI agent state via FEN updates
- **Validation**: Uses chess.js for legal move checking

### State Management
- **Shared State**: CopilotKit manages chess position between UI and agent
- **Real-time Updates**: Position changes trigger AI analysis
- **Persistence**: Game state maintained during session

## Development Setup

### Prerequisites
- Node.js 18+
- Python 3.8+
- OpenAI API Key
- uv package manager

### Installation
```bash
# Install frontend dependencies
npm install

# Install Python agent dependencies
npm run install:agent

# Set environment variables
export OPENAI_API_KEY="your-api-key"
```

### Running the Application
```bash
# Start both frontend and agent servers
npm run dev

# Or run separately
npm run dev:ui    # Frontend only
npm run dev:agent # Agent only
```

## Current Status

### Implemented
- ✅ Interactive chess board with drag-and-drop
- ✅ Move validation using chess.js
- ✅ FEN position state management
- ✅ AI agent integration with CopilotKit
- ✅ Real-time position updates
- ✅ Basic chess assistant functionality

### Planned Enhancements
- [ ] Advanced chess analysis tools
- [ ] Opening book integration
- [ ] Tactical puzzle generation
- [ ] Game history and replay
- [ ] Multiple difficulty levels
- [ ] Tournament mode

## Technical Notes

### FEN Notation
The system uses standard FEN (Forsyth-Edwards Notation) for position representation:
- Format: `rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1`
- Enables precise position communication between frontend and AI
- Standard format for chess engines and analysis

### Agent Communication
- **Protocol**: CopilotKit for seamless frontend-backend integration
- **State Sync**: Real-time position updates between chess board and AI
- **Tool Integration**: Ready for advanced chess analysis tools

## File Structure

```
src/
├── app/
│   ├── page.tsx          # Main application component
│   └── api/copilotkit/   # CopilotKit API routes
├── components/
│   ├── ChessBoard.tsx    # Interactive chess board
│   └── WeatherCard.tsx   # Example component (legacy)
agent/
├── agent/
│   ├── agent.py          # Chess assistant agent
│   └── server.py         # FastAPI server
└── pyproject.toml        # Python dependencies
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cedric-scheerlinck)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cedric-scheerlinck)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
