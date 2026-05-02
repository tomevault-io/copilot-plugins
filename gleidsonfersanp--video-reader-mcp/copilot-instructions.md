## video-reader-mcp

> > MCP server enabling AI models to read and analyze videos using **Progressive Context Enrichment** principles.

# AGENTS.md - mcp-video-reader

> MCP server enabling AI models to read and analyze videos using **Progressive Context Enrichment** principles.

## Quick Start

```bash
# Build and run
npm install && npm run build
node build/index.js
```

## MCP Session Workflow

**ALWAYS** follow this workflow at the start of every conversation:

```typescript
// 1. Identify where you are (RELATIVE PATH!)
identify_context({ file_path: "./src/index.ts" })

// 2. Check for active session
get_current_focus()

// 3. Start session OR load guidelines
start_session({ context: "backend", current_focus: "implementing new tool" })
// OR
get_merged_guidelines({ context: "backend" })

// 4. Work on task...

// 5. Create checkpoint for progress
create_checkpoint({ summary: "Added get_scene_frames tool", next_focus: "Write tests" })

// 6. Complete when done
complete_session()
```

**⚠️ PATH RULE**: Always use relative paths ( `./src/...` ) in MCP calls, never absolute paths.

---

## Project Overview

| Aspect | Details |
|--------|---------|
| **Type** | MCP Server (Model Context Protocol) |
| **Language** | TypeScript |
| **Runtime** | Node.js (ES Modules) |
| **Entry** | `./build/index.js` |
| **Package Manager** | npm |
| **Key Deps** | `@modelcontextprotocol/sdk` , `fluent-ffmpeg` , `sharp` |

### Core Principle

This server implements **Progressive Context Enrichment** - a pattern where AI agents:
1. Start with lightweight metadata (~200 tokens)
2. Fetch specific data on demand (~10K tokens per frame)
3. Never dump entire video content into context

```
❌ Bad:  analyze_video_full → ALL frames → 100K+ tokens
✅ Good: get_video_overview → get_frame(t=30) → get_frame(t=90) → ~20K tokens
```

---

## Dev Environment

### Prerequisites

* Node.js 18+
* npm

### Commands

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Watch mode for development
npm run watch

# Start server
npm start

# Verify setup
node test-setup.js
```

### No FFmpeg Required!

All binaries included via npm packages:
* `@ffmpeg-installer/ffmpeg`
* `@ffprobe-installer/ffprobe`

---

## Code Conventions

### TypeScript Strict Mode

* All code must pass `tsc` with strict settings
* Explicit types for function parameters and return values
* No `any` types - use `unknown` with type guards

### File Organization

```
src/
├── index.ts           # Server setup, tool handlers
├── video-processor.ts # Video processing logic
└── types.ts           # All TypeScript interfaces
```

### Naming Conventions

* **Files**: kebab-case (`video-processor.ts`)
* **Interfaces**: PascalCase (`VideoMetadata`,  `FrameReference`)
* **Functions**: camelCase (`getVideoOverview`,  `extractFrame`)
* **Constants**: UPPER_SNAKE (`TOKENS_PER_KB_BASE64`)

### Tool Design Principles

Following Context Engineering best practices:

1. **Tiered Tools**: Discovery → Progressive Fetch → Comprehensive
2. **Token Budget**: Each tool documents its approximate token cost
3. **Context Hints**: Return actionable suggestions for next steps
4. **Fail Fast**: Check file existence before processing

---

## Architecture

### Tool Tiers

```
TIER 1: Discovery (use first)
├── get_video_overview    ~200 tokens   Metadata + frame timestamps
├── get_video_metadata    ~100 tokens   Technical specs only
└── estimate_analysis_cost ~50 tokens   Plan before fetching

TIER 2: Progressive Fetch (on demand)
├── get_frame            ~10K tokens   Single frame at timestamp
├── get_frames_batch     ~50K tokens   Max 5 frames
└── extract_audio        ~50 tokens    Returns file path

TIER 2.5: Smart Analysis (scene-aware)
├── detect_scenes        ~200 tokens   Find scene change timestamps
├── get_scene_frames     ~10-50K tokens Frames at scene boundaries
├── get_video_chunks     ~150 tokens   Plan chunk analysis
└── analyze_chunk        ~15-25K tokens Single chunk + optional audio

TIER 2.6: Stream Analysis (progressive watching)
├── stream_start         ~15K tokens   Initialize streaming analysis
├── stream_next          ~15K tokens   Continue to next segment
└── stream_status        ~100 tokens   Check progress without advancing

TIER 2.7: Audio Transcription (requires OPENAI_API_KEY)
└── transcribe_audio     ~100+ tokens  Transcribe with Whisper API

TIER 3: Comprehensive (avoid)
└── analyze_video_full   ~100K+ tokens Only for <1 min videos
```

### Data Flow

```
Video File → VideoProcessor → Metadata/Frames → JSON Response
                   │
                   ├── getMetadata()        Lightweight FFprobe
                   ├── getVideoOverview()   References only
                   ├── extractFrame()       Single frame extraction
                   └── extractAudio()       Audio track extraction
```

### Key Interfaces

```typescript
// Progressive Context Pattern
interface VideoOverview {
  filename: string;
  metadata: VideoMetadataSummary;    // Human-readable summary
  availableFrames: FrameReference[]; // Timestamps only, no data
  contextHints: ContextHint[];       // Guides next actions
}

// Context Engineering Primitive
interface ContextHint {
  type: 'action' | 'warning' | 'info' | 'suggestion';
  message: string;
  suggestedTool?: string;
  priority: number;
}
```

---

## Adding New Tools

1. **Define tool** in `TOOLS` array in `./src/index.ts`
2. **Add handler** in `switch` statement
3. **Update types** in `./src/types.ts` if needed
4. **Document token cost** in tool description
5. **Include context hints** in response

### Tool Template

```typescript
{
  name: 'get_scene_frames',
  description: `Extract keyframes from scene changes. ~20K tokens for 5 scenes.
  
Use after get_video_overview to fetch representative frames.`,
  inputSchema: {
    type: 'object',
    properties: {
      videoPath: { type: 'string', description: 'Absolute path to video' },
      maxScenes: { type: 'number', description: 'Max scenes (default: 5)', default: 5 },
    },
    required: ['videoPath'],
  },
}
```

---

## Testing

No automated tests yet. Manual testing:

```bash
# Build
npm run build

# Test with Claude Desktop config
# Add to ~/Library/Application Support/Claude/claude_desktop_config.json:
{
  "mcpServers": {
    "video-reader": {
      "command": "node",
      "args": ["/path/to/mcp-video-reader/build/index.js"]
    }
  }
}
```

---

## Progressive Context Resources

For detailed workflows and patterns, see:
* [.ai-agents/QUICK-REFERENCE.md](.ai-agents/QUICK-REFERENCE.md) - Condensed checklist
* [.ai-agents/skills/SKILL.md](.ai-agents/skills/SKILL.md) - Progressive disclosure hub
* [.ai-agents/skills/PATTERNS-REFERENCE.md](.ai-agents/skills/PATTERNS-REFERENCE.md) - Code patterns

### External Resources

* [Anthropic Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
* [Progressive Context Enrichment](https://www.inferable.ai/blog/posts/llm-progressive-context-encrichment)
* [AGENTS.md Standard](https://agents.md/)

---
> Source: [GleidsonFerSanP/video-reader-mcp](https://github.com/GleidsonFerSanP/video-reader-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
