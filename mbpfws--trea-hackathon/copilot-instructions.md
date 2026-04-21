## trea-hackathon

> - **Duration**: 30 hours total development time


# AI-Powered Language Learning Platform - Project Rules & Guidelines

## 1. Project Overview & Constraints

### Hackathon Context
- **Duration**: 30 hours total development time
- **Team Structure**: Frontend AI Developer, Backend AI Developer, Debugger AI Developer
- **Primary Technologies**: Google Gemini 2.5, Zilliz Cloud, Next.js 15, Trae AI IDE
- **Deployment Target**: Vercel serverless platform

### Success Criteria
1. **Application of Technology**: Innovative use of Gemini 2.5 and Zilliz Cloud
2. **Business Value**: Practical learning outcomes demonstration
3. **Originality**: Unique approach to AI-powered language learning
4. **Presentation**: Compelling demo with stable functionality

## Phase 1: MCP Server Integration & Strategic Utilization

### 1.1 MCP Server Capabilities & Usage Patterns

#### Exa AI Research Server (mcp.config.usrlocalmcp.exa)
**Primary Functions**:
- **Technology Trend Analysis**: Research latest AI/ML developments, language learning innovations
- **Competitive Intelligence**: Analyze existing language learning platforms and their approaches
- **Technical Documentation**: Find implementation guides, best practices, and case studies
- **Market Research**: Understand user needs, pain points, and market opportunities

**Usage Patterns**:
```
# Research current AI language learning trends
run_mcp("mcp.config.usrlocalmcp.exa", "web_search_exa", {"query": "AI language learning 2024 trends Gemini integration", "numResults": 5})

# Find competitors and analyze their features
run_mcp("mcp.config.usrlocalmcp.exa", "competitor_finder_exa", {"companyName": "Duolingo", "industry": "language learning", "numResults": 5})

# Research technical implementation approaches
run_mcp("mcp.config.usrlocalmcp.exa", "github_search_exa", {"query": "Next.js language learning vector database", "searchType": "repositories", "numResults": 5})
```

#### Context7 Documentation Server (mcp.config.usrlocalmcp.context7)
**Primary Functions**:
- **Library Compatibility**: Verify latest versions and compatibility of dependencies
- **Implementation Guidance**: Access up-to-date documentation for complex integrations
- **Best Practices**: Retrieve current best practices for frameworks and libraries
- **Troubleshooting**: Access detailed documentation for debugging and optimization

**Usage Patterns**:
```
# Get latest Next.js 15 documentation
resolve-library-id({"libraryName": "Next.js"})
get-library-docs({"context7CompatibleLibraryID": "/vercel/next.js", "topic": "App Router", "tokens": 5000})

# Verify DaisyUI v5+ compatibility
resolve-library-id({"libraryName": "DaisyUI"})
get-library-docs({"context7CompatibleLibraryID": "/saadeghi/daisyui", "topic": "theming components", "tokens": 3000})

# Research Vercel AI SDK integration
resolve-library-id({"libraryName": "Vercel AI SDK"})
get-library-docs({"context7CompatibleLibraryID": "/vercel/ai", "topic": "streaming responses", "tokens": 4000})
```

#### Sequential Thinking Server (mcp.config.usrlocalmcp.Sequential Thinking)
**Primary Functions**:
- **Complex Problem Breakdown**: Analyze architectural decisions and implementation strategies
- **Multi-step Planning**: Break down complex features into manageable tasks
- **Decision Validation**: Verify technical choices through structured reasoning
- **Risk Assessment**: Identify potential issues and mitigation strategies

**Usage Patterns**:
```
# Break down complex AI integration architecture
sequentialthinking({
  "thought": "Analyzing optimal architecture for Gemini 2.5 + Zilliz integration",
  "nextThoughtNeeded": true,
  "thoughtNumber": 1,
  "totalThoughts": 5
})
```

#### Task Manager Server (mcp.config.usrlocalmcp.mcp-taskmanager)
**Primary Functions**:
- **Project Planning**: Structure development phases with clear milestones
- **Progress Tracking**: Monitor completion status and manage dependencies
- **Quality Gates**: Implement approval workflows for critical deliverables
- **Timeline Management**: Ensure 30-hour hackathon timeline adherence

**Usage Patterns**:
```
# Initialize project with structured tasks
request_planning({
  "originalRequest": "Develop AI language learning platform",
  "tasks": [
    {"title": "Setup Next.js 15 + DaisyUI foundation", "description": "Initialize project with core dependencies"},
    {"title": "Implement Gemini 2.5 integration", "description": "Setup AI conversation and assessment engines"},
    {"title": "Configure Zilliz Cloud vector database", "description": "Implement embedding storage and retrieval"}
  ]
})
```

#### Desktop Commander Server (mcp.config.usrlocalmcp.desktop-commander)
**Primary Functions**:
- **File Management**: Efficient file operations and project structure management
- **Configuration Management**: Handle environment variables and configuration files
- **Development Workflow**: Streamline development processes and automation
- **System Integration**: Manage local development environment

**Usage Patterns**:
```
# Read multiple configuration files efficiently
read_multiple_files({"paths": ["package.json", "next.config.js", ".env.local"]})

# Write configuration files in chunks
write_file({"path": "/absolute/path/to/config.ts", "content": "config chunk", "mode": "rewrite"})
```

### 1.2 Context Understanding & Preparation

#### Required Knowledge Base
- **Project Architecture**: Next.js 15.1.8 + App Router, DaisyUI + Tailwind CSS, Vercel AI SDK
- **AI Integration**: Google Gemini 2.5 Pro/Flash, Gemini Live API, multimodal capabilities
- **Data Layer**: Zilliz Cloud (Milvus), Neon PostgreSQL, vector embeddings
- **Authentication**: NextAuth.js with secure session management
- **Deployment**: Vercel serverless architecture

#### Critical Success Factors
- **Time Efficiency**: Prioritize MVP features, leverage pre-built components
- **AI Integration Excellence**: Showcase Gemini 2.5 capabilities effectively
- **Vector Database Utilization**: Demonstrate Zilliz Cloud for adaptive learning
- **User Experience**: Intuitive, responsive, engaging interface
- **Demo Readiness**: Stable, impressive demonstration capabilities

## Phase 2: Development Guidelines & Best Practices

### 2.1 Frontend Development Rules

#### Component Architecture
- **Consistency**: Use DaisyUI components as foundation, extend with Tailwind utilities
- **Modularity**: Create reusable components for learning modules, progress indicators, chat interfaces
- **Responsive Design**: Mobile-first approach, ensure cross-device compatibility
- **Performance**: Implement lazy loading, optimize bundle size, use Next.js Image optimization

#### State Management
- **Local State**: React hooks for component-level state
- **Global State**: Context API for user session, learning progress, AI conversation state
- **Server State**: SWR or React Query for API data fetching and caching
- **Form Handling**: React Hook Form + Zod for validation

#### Voice Integration Standards
- **Primary**: Gemini Live API for real-time conversational AI
- **Fallback**: Web Speech API for basic voice input/output
- **Error Handling**: Graceful degradation when voice features unavailable
- **User Feedback**: Clear visual indicators for voice recording, processing states

### 2.2 Backend API Development Rules

#### API Route Structure
```
/api/auth/* - Authentication endpoints (NextAuth.js)
/api/assessment/* - Proficiency evaluation endpoints
/api/conversation/* - AI conversation management
/api/content/* - Dynamic content generation
/api/progress/* - User progress tracking
/api/vector/* - Zilliz Cloud operations
```

#### AI Integration Patterns
- **Gemini 2.5 Pro**: Complex reasoning, content generation, multimodal analysis
- **Gemini 2.5 Flash**: Real-time interactions, quick responses, live conversations
- **Streaming Responses**: Use Vercel AI SDK for streaming AI responses
- **Error Handling**: Implement retry logic, fallback responses, timeout management

#### Vector Database Operations
- **Embedding Generation**: Use Gemini for text embeddings, store in Zilliz
- **Similarity Search**: Efficient proficiency matching, content recommendation
- **Data Indexing**: Optimize vector indices for learning content, user progress
- **Batch Operations**: Minimize API calls through intelligent batching

### 2.3 Database Schema Consistency

#### User Management
```sql
-- Core user profile with language preferences
users: id, email, name, native_language, target_language, proficiency_level

-- Learning session tracking
learning_sessions: id, user_id, session_type, duration, performance_score

-- Granular progress tracking
user_progress: id, user_id, skill_type, current_level, experience_points

-- Vocabulary and content progress
vocabulary_progress: id, user_id, word, familiarity_score, review_count
```

#### Vector Data Management
- **Content Embeddings**: Store learning material vectors in Zilliz
- **User Response Vectors**: Track learning patterns through embeddings
- **Proficiency Benchmarks**: Reference vectors for skill level assessment

### 2.4 AI Prompt Engineering Standards

#### Assessment Prompts
- **Structured Evaluation**: Clear rubrics for proficiency assessment
- **Adaptive Questioning**: Dynamic difficulty adjustment based on responses
- **Multimodal Integration**: Combine text, voice, and visual inputs

#### Content Generation Prompts
- **Personalization**: Include user interests, proficiency level, learning goals
- **Educational Principles**: Apply scaffolding, zone of proximal development
- **Cultural Sensitivity**: Ensure appropriate, inclusive content generation

#### Conversation Prompts
- **Natural Dialogue**: Encourage realistic, engaging conversations
- **Immediate Feedback**: Provide constructive, encouraging corrections
- **Adaptive Responses**: Adjust complexity based on user performance

## Phase 3: MCP-Enhanced Task Breakdown & Implementation Workflow

### 3.1 Strategic Task Planning with MCP Integration

#### Pre-Development Research Phase (Hours 0-2)
**MCP Server Utilization**:
```
# Research latest trends and competitive landscape
run_mcp("mcp.config.usrlocalmcp.exa", "web_search_exa", {
  "query": "AI language learning platforms 2024 Gemini integration best practices",
  "numResults": 5
})

# Analyze competitors for feature inspiration
run_mcp("mcp.config.usrlocalmcp.exa", "competitor_finder_exa", {
  "companyName": "Duolingo",
  "industry": "language learning",
  "numResults": 3
})

# Verify latest documentation for tech stack
run_mcp("mcp.config.usrlocalmcp.context7", "resolve-library-id", {"libraryName": "Next.js"})
run_mcp("mcp.config.usrlocalmcp.context7", "get-library-docs", {
  "context7CompatibleLibraryID": "/vercel/next.js",
  "topic": "App Router streaming",
  "tokens": 3000
})
```

#### Task Structure Initialization
```
# Initialize comprehensive task management
run_mcp("mcp.config.usrlocalmcp.mcp-taskmanager", "request_planning", {
  "originalRequest": "Develop AI-powered language learning platform for hackathon",
  "splitDetails": "30-hour timeline with Gemini 2.5 + Zilliz Cloud integration",
  "tasks": [
    {
      "title": "Foundation & Architecture Setup",
      "description": "Next.js 15 + DaisyUI + authentication + database connections"
    },
    {
      "title": "AI Integration Core",
      "description": "Gemini 2.5 Pro/Flash integration with streaming responses"
    },
    {
      "title": "Vector Database Implementation",
      "description": "Zilliz Cloud setup with embedding storage and retrieval"
    },
    {
      "title": "User Experience & Features",
      "description": "Assessment engine, conversation practice, progress tracking"
    },
    {
      "title": "Demo Optimization & Deployment",
      "description": "Performance tuning, demo content, production deployment"
    }
  ]
})
```
## 2. Technical Architecture Standards

### Technology Stack Requirements

#### Frontend Stack
- **Framework**: Next.js 15.1.8 with App Router
- **UI Library**: DaisyUI + Tailwind CSS V4
- **State Management**: Zustand for global state
- **Voice Inte

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mbpfws) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
