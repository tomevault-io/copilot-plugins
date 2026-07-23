## kira

> This file provides critical guidance to Claude Code (claude.ai/code) when working with the KIRA project.

# CLAUDE.md - Developer Guide for Claude Code

This file provides critical guidance to Claude Code (claude.ai/code) when working with the KIRA project.

## 🎯 Core Concept: "Install App = Hire AI Coworker"

**CRITICAL UNDERSTANDING**: This project's revolutionary concept is that installing the desktop app instantly transforms any computer into an AI coworker. No servers, no cloud setup, no technical expertise required - just like installing Microsoft Office.

### The Vision
- **End Users**: Non-technical people who need an AI assistant
- **Installation**: As simple as installing any desktop application
- **Configuration**: GUI-based, no terminal or coding required
- **Result**: A fully functional AI coworker working 24/7

### Development Philosophy
When developing ANY feature, always ask:
1. Can a non-developer understand and use this?
2. Does it maintain the "zero configuration" principle?
3. Is the GUI self-explanatory?
4. Will it work immediately after installation?

---

## 📁 Project Structure

```
kira/                           # Root directory
│
├── electron-app/               # 🎯 THE CORE - Desktop app
│   ├── main.js                # Electron main process - manages Python server lifecycle
│   ├── preload.js             # Secure IPC bridge
│   ├── renderer/              # User interface
│   │   ├── index.html         # Main UI structure (Environment Variables Settings UI)
│   │   ├── main.css           # Dark theme styling
│   │   └── main.js            # UI logic, server control, config management
│   ├── package.json           # Node dependencies and build config
│   └── dist/                  # Built installers (.dmg, .exe, .AppImage)
│
├── app/                       # Python AI server (runs invisibly in background)
│   ├── main.py               # Server entry point, worker/scheduler setup
│   ├── cc_agents/            # AI agent modules
│   │   ├── bot_call_detector/        # Bot call detection (Haiku)
│   │   ├── simple_chat/              # Simple conversation (Haiku)
│   │   ├── operator/                 # Complex task execution (Sonnet)
│   │   ├── memory_retriever/         # Memory search (Haiku)
│   │   ├── memory_manager/           # Memory storage (Haiku)
│   │   ├── answer_aggregator/        # Answer collection (Haiku)
│   │   ├── proactive_suggester/      # Proactive suggestions (Sonnet)
│   │   └── proactive_confirm/        # Suggestion approval request (Haiku)
│   ├── cc_checkers/          # Proactive Receiver Channels (Proactive monitors)
│   │   ├── outlook/          # Outlook email checker
│   │   └── atlassian/        # Confluence/Jira checker (Rovo MCP)
│   ├── cc_tools/             # MCP tool implementations
│   │   ├── slack/            # Slack tools (11)
│   │   ├── outlook/          # Outlook tools (7)
│   │   ├── scheduler/        # Message scheduling
│   │   ├── waiting_answer/   # Answer waiting management
│   │   ├── confirm/          # User approval management
│   │   ├── email_tasks/      # Email tasks DB
│   │   ├── jira_tasks/       # Jira tasks DB
│   │   └── x/                # X (Twitter) tools
│   ├── cc_web_interface/     # Web Server / Voice Input Channel
│   │   ├── server.py         # FastAPI server (port 8000, HTTPS)
│   │   ├── auth_handler.py   # Auth router
│   │   ├── auth_azure.py     # MS365 OAuth
│   │   └── auth_slack.py     # Slack OAuth
│   ├── cc_slack_handlers.py  # Slack event handlers
│   ├── queueing_extended.py  # 3-tier queue system
│   ├── scheduler.py          # APScheduler management
│   └── config/               # Configuration management
│       ├── settings.py       # Pydantic settings (environment variable definitions)
│       └── env/
│           ├── dev.env       # Development environment variables
│           └── credential.json  # GCP service account
│
└── docs/                     # User-facing documentation
```

---

## 🚨 Critical Development Rules

### 1. Electron App is Sacred
- **NEVER** break the Electron app - it's the user's only interface
- **ALWAYS** test GUI changes on multiple screen sizes
- **ENSURE** error messages are user-friendly (no stack traces!)
- **MAINTAIN** backward compatibility with saved configs

### 2. Zero-Setup Principle
- **NO** manual file editing should ever be required
- **NO** terminal commands for end users
- **ALL** configuration through GUI
- **DEFAULT** values for everything optional

### 3. Environment Variable Synchronization Required
**CRITICAL**: When adding/modifying environment variables, you **MUST update all 5 locations**:

1. **app/config/settings.py** - Pydantic model definition
2. **app/config/env/dev.env** - Development default values
3. **electron-app/renderer/index.html** - UI input fields
4. **electron-app/renderer/main.js** - `fields` array (list of fields to save/load)
5. **electron-app/main.js** - config.env save section (`sections` object)

**Be especially careful not to forget the `fields` array in `renderer/main.js`!**
Fields not in this array are ignored by the save/load logic.

The order must also match:
```
1. Slack Integration
2. Bot Information
3. MCP Settings (Perplexity, DeepL, GitLab, Atlassian, Outlook, X, Clova)
4. Computer Use
5. Web Server / Voice Input Channel
6. Proactive Receiver Channel (Outlook, Confluence, Jira)
7. Proactive Suggestion Feature
```

**CRITICAL - Slack Credential Naming:**
- **Bot credentials** (for Slack Bolt framework):
  - `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`, `SLACK_SIGNING_SECRET`, `SLACK_TEAM_ID`
  - Used in "Slack Integration" section
  - Slack Bolt auto-detects these names
- **Web OAuth credentials** (for web interface login):
  - `WEB_SLACK_CLIENT_ID`, `WEB_SLACK_CLIENT_SECRET`
  - Used in "Web Server / Voice Input Channel" section
  - **MUST** have `WEB_` prefix to avoid Slack Bolt conflict
  - If named `SLACK_CLIENT_ID/SECRET`, Bolt switches to OAuth mode and breaks bot token

### 4. Error Handling
```python
# BAD - Developer-focused error
raise ValueError(f"Invalid token format: {token}")

# GOOD - User-friendly error
logger.error("Slack integration failed: token is invalid. Please check settings.")
return "Slack integration failed. Please check Slack token in environment settings."
```

### 5. GUI Text Guidelines
- Use Korean for UI labels (primary users are Korean)
- Provide helpful placeholders
- Include inline help text (info-box, notice-box)
- Show examples where possible

---

## 🏗️ Complete System Architecture

### Message Processing Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│ 1. SLACK MESSAGE RECEIVED                                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. DEBOUNCING (2 seconds)                                   │
│    - Merge consecutive messages from same user              │
│    - debounced_enqueue_message()                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. CHANNEL QUEUE                                            │
│    - Independent queue per channel                          │
│    - 8 workers per channel                                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. MESSAGE PROCESSING (_process_message_logic)             │
│    ├─ Collect Slack Context (last 10 messages)             │
│    ├─ Bot Call Detector (Haiku) - Determine if bot called  │
│    │   ├─ DM: Always process                                │
│    │   ├─ Group: Only when mentioned                        │
│    │   └─ Thread: Additional Thread Context Detector check  │
│    ├─ Answer Aggregator - Check for awaiting answers       │
│    │   └─ Query waiting_answer DB                          │
│    └─ Routing decision                                      │
└─────────────────────────────────────────────────────────────┘
                 ↓                              ↓
    ┌────────────────────┐         ┌────────────────────────┐
    │ 5-A. SIMPLE CHAT   │         │ 5-B. ORCHESTRATOR      │
    │ (Haiku, no MCP)    │         │ (Sonnet, all MCP)      │
    │ - Simple chat      │         │ - Complex tasks        │
    │ - Instant response │         │ - Memory Retriever     │
    └────────────────────┘         └────────────────────────┘
                                               ↓
                              ┌────────────────────────────────┐
                              │ ORCHESTRATOR QUEUE (global)    │
                              │ - 3 workers                    │
                              │ - call_operator_agent()        │
                              └────────────────────────────────┘
                                               ↓
                              ┌────────────────────────────────┐
                              │ MEMORY QUEUE (global)          │
                              │ - 1 worker (sequential)        │
                              │ - call_memory_manager()        │
                              │ - Local filesystem storage     │
                              └────────────────────────────────┘
```

### 3-Tier Queue System

```python
# 1. Channel Queues (per-channel)
message_queues: Dict[str, asyncio.Queue] = {}
# - 8 workers per channel
# - Fast response for simple messages

# 2. Orchestrator Queue (global)
orchestrator_queue = asyncio.Queue(maxsize=100)
# - 3 workers
# - Heavy tasks with MCP tools

# 3. Memory Queue (global)
memory_queue = asyncio.Queue(maxsize=100)
# - 1 worker (sequential processing)
# - Sequential local filesystem storage
```

### Proactive Systems

#### Proactive Receiver Channels (Checkers) - Beta

**Outlook Checker**
```
Scheduler (5 minute interval)
  ↓
check_email_updates() - checker.py
  ├─ Query inbox via Outlook MCP
  └─ process_emails_batch()
       ├─ call_email_task_extractor() - agent.py
       │    └─ Extract important tasks → Save to email_tasks DB
       └─ Send pending tasks → Slack Channel Queue
```

**Confluence Checker**
```
Scheduler (60 minute interval)
  ↓
check_confluence_updates() - confluence_checker.py
  ├─ Query recent N-hour updates via Rovo MCP
  ├─ Filter out bot's own posts in Python
  └─ process_pages_batch()
       ├─ call_confluence_summarizer() - confluence_agent.py
       │    └─ Summarize important pages only
       └─ Save to Memory
```

**Jira Checker**
```
Scheduler (30 minute interval)
  ↓
check_jira_updates() - jira_checker.py
  ├─ Query assigned issues via Rovo MCP
  └─ process_issues_batch()
       ├─ Exclude existing issues from DB
       ├─ call_jira_task_extractor() - jira_agent.py
       │    └─ Extract tasks → Save to jira_tasks DB
       └─ Send pending tasks → Slack Channel Queue
```

#### Web Server / Voice Input Channel

```
FastAPI Server (port 8000, HTTPS)
  ├─ Authentication: Microsoft 365 / Slack OAuth (OpenID Connect)
  ├─ X (Twitter) OAuth 2.0 authentication flow
  ├─ Clova Speech voice recognition
  └─ Voice input processing
```

**Required:**
- SSL certificates (`app/config/certs/`)
- Port 8000 available
- Required when using X, Clova Speech

**Authentication System (Critical):**
- **Slack OAuth**: Uses OpenID Connect (OIDC)
  - ❌ Legacy `identity.*` scopes (deprecated, invalid_scope error)
  - ✅ OIDC scopes: `openid`, `email`, `profile`
  - Endpoints: `/openid/connect/authorize`, `/api/openid.connect.token`
  - Enterprise Grid compatible (may require Org-ready)
  - Credentials: `WEB_SLACK_CLIENT_ID`, `WEB_SLACK_CLIENT_SECRET` (separate from Bot Token)
- **Microsoft 365**: Azure AD OpenID Connect
  - Uses Authlib library
  - Credentials: `OUTLOOK_CLIENT_ID`, `OUTLOOK_CLIENT_SECRET`, `OUTLOOK_TENANT_ID`
  - Query user info via Graph API

#### Proactive Suggestion Feature - Beta

```
Dynamic Suggester (15 minute interval)
  ↓
call_dynamic_suggester()
  ├─ Analyze local memory files
  ├─ Generate suggestions (Sonnet)
  ├─ call_proactive_confirm() - Request user approval
  │    └─ Save to confirm DB
  └─ On approval → Send to Orchestrator Queue
```

---

## 🤖 Agent Inventory

| Agent | Model | MCP | Purpose | Location |
|-------|-------|-----|---------|----------|
| **Bot Call Detector** | Haiku | ❌ | Detect bot calls | `cc_agents/bot_call_detector/` |
| **Thread Context Detector** | Haiku | ❌ | Detect thread context | `cc_agents/bot_thread_context_detector/` |
| **Answer Aggregator** | Sonnet | ✅ | Check for awaiting answers | `cc_agents/answer_aggregator/` |
| **Simple Chat** | Haiku | ❌ | Simple conversation | `cc_agents/simple_chat/` |
| **Memory Retriever** | Haiku | ✅ | Search memory | `cc_agents/memory_retriever/` |
| **Operator** | Opus | ✅ | Execute complex tasks | `cc_agents/operator/` |
| **Memory Manager** | Sonnet | ✅ | Store memory | `cc_agents/memory_manager/` |
| **Email Task Extractor** | Haiku | ✅ | Extract email tasks | `cc_checkers/outlook/agent.py` |
| **Confluence Summarizer** | Haiku | ✅ | Summarize important pages | `cc_checkers/atlassian/confluence_agent.py` |
| **Jira Task Extractor** | Haiku | ✅ | Extract Jira tasks | `cc_checkers/atlassian/jira_agent.py` |
| **Dynamic Suggester** | Sonnet | ✅ | Generate proactive suggestions | `cc_agents/proactive_dynamic_suggester/` |
| **Proactive Confirm** | Haiku | ✅ | Request suggestion approval | `cc_agents/proactive_confirm/` |

---

## 🛠️ MCP Tools Available

### Core MCP Servers
- **slack**: 11 tools (messages, files, reactions, channel management, etc.)
- **outlook**: 7 tools (email query, compose, reply, attachments, etc.)
- **atlassian**: Rovo MCP (Confluence/Jira unified search and management)
- **gitlab**: Code repository management
- **x**: Twitter/X tools (OAuth 1.0a + OAuth 2.0)
- **perplexity**: Web search
- **deepl**: Translation
- **playwright**: Browser automation (uses Chrome profile)
- **kris**: Internal API

### Custom MCP Servers (Local)
- **scheduler**: Message scheduling (SQLite)
- **waiting_answer**: Answer collection management (SQLite)
- **confirm**: User approval management (SQLite)
- **email_tasks**: Email tasks DB (SQLite)
- **jira_tasks**: Jira tasks DB (SQLite)

---

## 💾 Data Storage

### SQLite Databases (4)
```
~/.kira/
├── waiting_answer.db   # Questions awaiting answers
├── confirm.db          # Tasks awaiting user approval
├── email_tasks.db      # Tasks extracted from emails
└── jira_tasks.db       # Tasks extracted from Jira
```

### Local Memory System
```
$FILESYSTEM_BASE_DIR/memories/  # Default: ~/Documents/KIRA/memories/
├── channels/           # Conversation history per channel
├── projects/           # Project-related information
├── users/              # User-specific information
├── decisions/          # Decisions
└── index.md            # Auto-generated index
```
- **Format**: Markdown files
- **Management**: Uses `slack-memory-store` skill
- **Search**: Memory Retriever performs index-based search
- **Storage**: Memory Manager auto-categorizes and stores

### Configuration
```
~/.kira/config.env      # User environment variables (saved by Electron app)
```

---

## 📝 Development Patterns

### Checker Pattern (2-stage)

**Stage 1: Checker (Data Collection)**
```python
# cc_checkers/*/checker.py
async def check_*_updates():
    """Called periodically by scheduler"""
    # 1. Query data via MCP
    # 2. Filter in Python (exclude bot's own posts, etc.)
    # 3. Pass to Agent
    asyncio.create_task(process_*_batch(data))
```

**Stage 2: Agent (Analysis & Processing)**
```python
# cc_checkers/*/agent.py
async def call_*_extractor(data):
    """Analyze and process data via Claude SDK"""
    # 1. Generate system prompt
    # 2. Access MCP via ClaudeSDKClient
    # 3. Save results to DB or Memory
    # 4. Send to Slack Queue if needed
```

### Agent Pattern (Standard)

```python
# cc_agents/*/agent.py
async def call_agent_name(
    user_query: str,
    slack_data: dict,
    message_data: dict,
    retrieved_memory: Optional[str] = None
) -> str:
    """
    Standard agent pattern using Claude SDK

    Returns:
        str: Response to send to user (Korean)
        bool: For Simple Chat, whether it was handled
    """
    from claude_agent_sdk import ClaudeAgentOptions, ClaudeSDKClient

    # 1. Generate system prompt
    system_prompt = create_system_prompt(...)

    # 2. Configure MCP servers
    mcp_servers = {...}

    # 3. Create Options
    options = ClaudeAgentOptions(
        system_prompt=system_prompt,
        model="haiku",  # or sonnet-4-5
        permission_mode="bypassPermissions",
        allowed_tools=["*"],
        disallowed_tools=[...],
        mcp_servers=mcp_servers,
    )

    # 4. Execute SDK
    async with ClaudeSDKClient(options=options) as client:
        await client.query(user_query)
        async for message in client.receive_response():
            if isinstance(message, ResultMessage):
                return message.result
```

### MCP Tool Pattern

```python
# cc_tools/*/tools.py
@tool("tool_name", "User-friendly description", schema)
async def tool_function(args: Dict[str, Any]) -> Dict[str, Any]:
    """
    CRITICAL: Use httpx.AsyncClient, NEVER requests
    ALWAYS return user-friendly Korean error messages
    """
    try:
        async with httpx.AsyncClient() as client:
            # Implementation
            pass
    except Exception as e:
        logger.error(f"[TOOL_NAME] Error: {e}")
        return {
            "content": [{
                "type": "text",
                "text": f"Task failed: {str(e)}"
            }]
        }
```

### Web Authentication Pattern

**Multi-Provider OAuth Architecture:**

```python
# auth_handler.py - Provider routing
class AuthHandler:
    def __init__(self):
        self.provider = self._get_provider()  # From WEB_INTERFACE_AUTH_PROVIDER

        if self.provider == AuthProvider.MICROSOFT:
            self.azure_oauth = AzureOAuth()
        elif self.provider == AuthProvider.SLACK:
            self.slack_oauth = SlackOAuth()

    async def handle_login(self, request):
        if self.provider == AuthProvider.SLACK:
            # Slack OIDC flow
            return RedirectResponse(url=slack_oauth.get_authorize_url())
        else:
            # Microsoft Azure AD flow
            return await azure_oauth.authorize_redirect()

    async def handle_callback(self, request):
        # Provider-specific token exchange and user info retrieval
        # Both providers return: {name, email, id, provider}
```

**Critical OAuth Implementation Details:**

1. **Slack OIDC** (auth_slack.py):
   - Endpoints: `/openid/connect/authorize`, `/api/openid.connect.token`, `/api/openid.connect.userInfo`
   - Scopes: `openid email profile`
   - Response: `{ok: true, sub: "U...", name: "...", email: "...", picture: "..."}`
   - User ID field: `sub` (not `id`)

2. **Microsoft Azure** (auth_azure.py):
   - Uses Authlib OAuth library
   - Server metadata: `https://login.microsoftonline.com/{tenant_id}/v2.0/.well-known/openid-configuration`
   - Scopes: `openid email profile User.Read`
   - Graph API: `https://graph.microsoft.com/v1.0/me`

3. **Session Management**:
   - FastAPI SessionMiddleware
   - Session data: `{email, name, id, provider, avatar}`
   - Authorization check: `is_authorized_user(name)` from `cc_slack_handlers.py`

---

## 🖥️ Electron App Development

### Configuration File Structure

**Storage Location**: `~/.kira/config.env`

**Section Order** (main.js sections):
```javascript
const sections = {
  'Slack Integration': [...],
  'Bot Information': [...],
  'MCP Settings - Perplexity': [...],
  'MCP Settings - DeepL': [...],
  'MCP Settings - GitLab': [...],
  'MCP Settings - Atlassian Rovo': [...],
  'MCP Settings - Outlook': [...],
  'MCP Settings - X': [...],
  'MCP Settings - Clova Speech': [...],
  'Computer Use': [...],
  'Web Server / Voice Input Channel': [...],
  'Proactive Receiver Channel - Outlook': [...],
  'Proactive Receiver Channel - Confluence': [...],
  'Proactive Receiver Channel - Jira': [...],
  'Proactive Suggestion Feature': [...]
};
```

### UI Section Structure (index.html)

```html
<!-- Required Settings -->
<section class="section">
  <h3>Required Settings - Slack Integration</h3>
  <!-- SLACK_BOT_TOKEN, SLACK_APP_TOKEN, etc. -->
</section>

<section class="section">
  <h3>Required Settings - Bot Information</h3>
  <!-- BOT_NAME, BOT_EMAIL, etc. -->
</section>

<!-- MCP Settings -->
<section class="section">
  <h3>MCP Settings</h3>
  <!-- Toggle + Fields pattern -->
  <div class="mcp-item">
    <div class="mcp-header">
      <span class="mcp-title">Service Name</span>
      <label class="toggle-switch">
        <input type="checkbox" id="SERVICE_ENABLED">
        <span class="toggle-slider"></span>
      </label>
    </div>
    <div class="mcp-fields" data-mcp="service" style="display: none;">
      <!-- Fields shown when enabled -->
    </div>
  </div>
</section>

<!-- Computer Use -->
<section class="section">
  <h3>Computer Use</h3>
</section>

<!-- Web Server -->
<section class="section">
  <h3>Web Server / Voice Input Channel</h3>
</section>

<!-- Proactive Receiver Channels (Beta) -->
<section class="section">
  <h3>Proactive Receiver Channels <span class="beta-chip">beta</span></h3>
</section>

<!-- Proactive Suggestions (Beta) -->
<section class="section">
  <h3>Proactive Suggestion Feature <span class="beta-chip">beta</span></h3>
</section>
```

### Main Process (main.js) Key Responsibilities

1. **Window Management**: Create window, save/restore size/position
2. **Python Server Lifecycle**: Find uv, spawn process, pass environment variables
3. **Configuration**: Read/write config.env, parseConfigFile()
4. **IPC Handlers**: get-config, save-config, start-server, stop-server
5. **Log Streaming**: Stream Python stdout/stderr to renderer

### Renderer Process (renderer/main.js) Key Responsibilities

1. **Config Load/Save**: window.api.getConfig(), window.api.saveConfig()
2. **Toggle Visibility**: Show/hide MCP fields, channel fields, voice fields
3. **Auth Provider Handling**: Conditionally show Slack OAuth fields
4. **Server Control**: startServer(), stopServer()
5. **Log Display**: Real-time log streaming

---

## 🔥 Development Commands

### Python Server Development

```bash
# Hot reload development (recommended)
uv run python dev.py

# Direct server start (no reload)
uv run python -m app.main

# Install/sync dependencies
uv sync

# Install specific package
uv add package-name
```

### Electron App Development

```bash
cd electron-app

# Development mode (opens dev tools)
npm run start

# Build installers
npm run build           # macOS .dmg and .zip (arm64 (Apple Silicon))
npm run build -- --mac  # macOS only
npm run build -- --win  # Windows (requires Windows or Wine)
npm run build -- --linux  # Linux AppImage

# Publish to S3
npm run publish  # Requires AWS credentials

# Install dependencies
npm install
```

### Testing and Quality

```bash
# Python formatting (Black)
uv run black app/

# Check Python syntax
uv run python -m py_compile app/**/*.py

# Manual testing workflow:
# 1. Start Python server: uv run python dev.py
# 2. Start Electron app: cd electron-app && npm start
# 3. Test in Slack workspace
# 4. Check logs: tail -f ~/.kira/logs/kira.log
```

### Build Artifacts Location

```
electron-app/dist/
├── KIRA-X.X.X-arm64.dmg        # macOS installer
├── KIRA-X.X.X-arm64-mac.zip    # macOS portable
├── KIRA-1.0.5.exe                  # Windows installer
└── KIRA-1.0.5.AppImage             # Linux portable
```

---

## 🧪 Testing Checklist

### Before ANY Commit

#### Electron App
- [ ] App launches without errors
- [ ] Configuration saves/loads correctly
- [ ] Server starts/stops properly
- [ ] Logs display in real-time
- [ ] Error messages are user-friendly
- [ ] Toggle switches work (MCP, channels, voice)
- [ ] Auth provider dropdown shows/hides Slack fields

#### Python Server
- [ ] Starts with minimal config (just Slack tokens)
- [ ] Handles missing optional configs gracefully
- [ ] Responds to Slack messages
- [ ] Logs are informative but not overwhelming
- [ ] Checkers run on schedule (if enabled)
- [ ] Web server starts (if enabled)

#### Environment Variables
- [ ] Order matches settings.py
- [ ] Order matches dev.env
- [ ] Order matches index.html UI
- [ ] Order matches main.js sections

#### Integration
- [ ] Fresh install works (no existing config)
- [ ] Upgrade preserves existing config
- [ ] Server restarts handle gracefully
- [ ] Memory/CPU usage is reasonable

---

## 🎨 UI/UX Standards

### Visual Design
- **Theme**: Dark mode (easier on eyes for 24/7 operation)
- **Font**: SF Pro Display (macOS), Segoe UI (Windows)
- **Colors**: Blue accent (#007AFF), success green (#34C759), error red (#FF3B30)

### Korean Language Standards
```javascript
// UI Labels (Korean)
"시작하기"      // Start
"중지하기"      // Stop
"설정하기"      // Configure
"로그 보기"     // View Logs

// Status Messages (Korean + English technical terms)
"Slack 연결 성공"
"Outlook 모니터링 시작 (5분 간격)"
"Confluence 페이지 업데이트 확인 중"
```

### Info Box Types
```html
<!-- General information -->
<div class="info-box">
  General information or help text
</div>

<!-- Important notice -->
<div class="notice-box">
  <strong>Important:</strong> Web interface is required.
</div>
```

---

## 🚀 Performance Considerations

### Electron App
- **Window State**: Save and restore window position/size
- **Log Streaming**: Limit to last 1000 lines to prevent memory issues
- **Config Updates**: Debounce saves to prevent excessive disk writes

### Python Server
- **Queue Workers**:
  - 8 workers per channel (fast response)
  - 3 orchestrator workers (heavy tasks)
  - 1 memory worker (sequential)
- **Debouncing**: 2-second delay for message grouping
- **Memory Management**: Rotate logs, clean tmp files
- **Async Everything**: Use asyncio for all I/O operations

---

## 🔐 Security Principles

1. **Credentials**: Never log tokens or passwords
2. **Storage**: `~/.kira/config.env` with proper permissions
3. **Communication**: IPC only through preload script
4. **Validation**: Sanitize all user inputs
5. **Permissions**: Request minimum required scopes
6. **Web Auth**: Microsoft 365 or Slack OAuth only (no "none" option)

---

## 📊 Monitoring & Debugging

### Development Tools
```bash
# View Python logs
tail -f ~/.kira/logs/kira.log

# Phoenix tracing (if configured)
open https://phoenix.arize.com

# Electron DevTools
# Cmd+Option+I (macOS) or F12 (Windows/Linux)
```

### Production Diagnostics
- Check `~/.kira/logs/` for server logs
- View Electron logs in app's log viewer
- Use Phoenix for tracing agent behavior
- Monitor Slack App dashboard for API limits

---

## 💡 Common Issues & Solutions

### Issue: "uv not found"
**Solution**: Install uv with `curl -LsSf https://astral.sh/uv/install.sh | sh`

### Issue: "Slack token invalid"
**Solution**: Check Slack App settings, regenerate tokens if needed

### Issue: "Server won't start"
**Solution**: Check port 8000 availability, verify Python version 3.10+

### Issue: "Checker not working"
**Solution**:
1. Check MCP is enabled (OUTLOOK_ENABLED, ATLASSIAN_ENABLED)
2. Check checker is enabled (*_CHECK_ENABLED)
3. View logs for authentication errors

### Issue: "Web Interface dependencies"
**Solution**: Show user that X, Clova require WEB_INTERFACE_ENABLED=True

### Issue: "Slack OAuth invalid_scope error"
**Solution**:
1. Legacy `identity.*` scopes are deprecated - use OpenID Connect instead
2. In Slack App settings, User Token Scopes:
   - Remove: `identity.basic`, `identity.email`, `identity.avatar`, `identity.team`
   - Add: `openid`, `email`, `profile`
3. For Enterprise Grid, enable "Org-ready" in Slack App settings
4. Reinstall app to workspace after scope changes

---

## 🎯 Remember: The User Experience is Everything

Every line of code should consider:
- **Sarah from Marketing** who's never used a terminal
- **John from Sales** who just wants it to work
- **The IT Admin** who needs to deploy to 50 computers
- **The Solo Founder** working at 2 AM

This isn't just a bot - it's an AI coworker that lives in a desktop app. Make it feel like hiring a helpful colleague, not configuring a server.

---

## 📦 Deployment

### Production URL

**Documentation Site:**
```
https://kira.krafton-ai.com/
```

**App Download:**
```
https://kira.krafton-ai.com/download/KIRA-{version}-arm64.dmg
```

**Current Version**: 0.9.0

### Deployment Infrastructure

- **S3 Bucket**: `kira-releases` (ap-northeast-2)
  - Documentation HTML files (VitePress)
  - App download files (`.dmg`, `.zip`)
- **CloudFront**: Custom domain `kira.krafton-ai.com`
  - Origin: S3 Website Endpoint
  - SSL: AWS ACM certificate
- **Route 53**: `kira.krafton-ai.com` A record (Alias)

### Deployment Methods

**Deploy Documentation:**
```bash
cd vitepress-app
npm run deploy
```

**Deploy App:**
```bash
cd electron-app
npm version patch  # Update version
npm run deploy     # Build + S3 upload
```

**Detailed Guide**: See `DEPLOY.md`

---

## 📌 Quick Reference

### Key Files to Check When Adding Features

1. **Environment Variables**:
   - `app/config/settings.py`
   - `app/config/env/dev.env`
   - `electron-app/renderer/index.html`
   - `electron-app/main.js`

2. **Message Processing**:
   - `app/cc_slack_handlers.py`
   - `app/queueing_extended.py`

3. **Agents**:
   - `app/cc_agents/*/agent.py`

4. **Checkers**:
   - `app/cc_checkers/*/checker.py`
   - `app/cc_checkers/*/agent.py`

5. **Scheduler**:
   - `app/main.py` (scheduler registration)

6. **Web Server**:
   - `app/cc_web_interface/server.py`
   - `app/cc_web_interface/auth_handler.py` (provider routing)
   - `app/cc_web_interface/auth_slack.py` (Slack OIDC)
   - `app/cc_web_interface/auth_azure.py` (MS365 OAuth)

---

**Final Note for Claude Code**: When in doubt, prioritize simplicity and user experience over technical elegance. The best code is the code that lets non-developers successfully deploy their own AI coworker.

**Always check when making additional changes**:
- Synchronize environment variables in 4 locations (settings.py, dev.env, index.html, main.js)
- Match section order
- Korean UI labels
- User-friendly error messages

---
> Source: [krafton-ai/KIRA](https://github.com/krafton-ai/KIRA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
