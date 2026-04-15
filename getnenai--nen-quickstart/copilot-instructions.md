## nen-quickstart

> Nen Python SDK reference for workflow authoring - execution environment, keyboard/mouse control, and common patterns


# Nen Python SDK - Execution Environment & Best Practices

## Execution Environment

**CRITICAL:** Workflows run in **Linux containers** with the following characteristics:

- **OS:** Linux (Ubuntu-based)
- **Keyboard/Mouse:** xdotool (Linux tool for input control)
- **Browser:** Chromium (open-source browser available in taskbar)
- **Display:** Headless X11 with VNC recording

### Platform Compatibility

**Always use Linux/Windows keyboard modifiers, NOT macOS:**

```python
# ❌ WRONG - macOS only (will crash with xdotool error)
computer.hotkey("command", "a")
computer.hotkey("command", "c")

# ✅ CORRECT - Linux/Windows (works in workflow container)
computer.hotkey("ctrl", "a")
computer.hotkey("ctrl", "c")
```

**Common key mappings:**
- Select All: `computer.hotkey("ctrl", "a")` (not "command")
- Copy: `computer.hotkey("ctrl", "c")` (not "command")
- Paste: `computer.hotkey("ctrl", "v")` (not "command")
- Cut: `computer.hotkey("ctrl", "x")` (not "command")
- Save: `computer.hotkey("ctrl", "s")` (not "command")

### xdotool Key Names

Valid key names for `computer.press()` and `computer.hotkey()`:

**Modifier keys:**
- `ctrl` - Control key (Linux/Windows)
- `shift` - Shift key
- `alt` - Alt key (Option on Mac)
- `super` - Windows/Super key (Command on Mac)

**Special keys:**
- `Return` or `Enter` - Enter/Return key
- `BackSpace` - Backspace key
- `Delete` - Delete key
- `Tab` - Tab key
- `Escape` - Escape key
- `space` - Space bar

**Navigation keys:**
- `Up`, `Down`, `Left`, `Right` - Arrow keys
- `Home`, `End` - Home/End keys
- `Page_Up`, `Page_Down` - Page up/down

**Function keys:**
- `F1`, `F2`, ..., `F12` - Function keys

## Common Patterns

### Clearing Input Fields

Always clear fields before typing new content:

```python
from nen import Agent, Computer

agent = Agent()
computer = Computer()

# Method 1: Select all + delete
agent.execute("Click on the email field")
computer.hotkey("ctrl", "a")  # Select all text
computer.press("BackSpace")   # Delete selected text
computer.type(params.email)   # Type new content

# Method 2: Triple-click to select all + delete
agent.execute("Triple-click the text field to select all")
computer.press("BackSpace")
computer.type(params.value)

# Method 3: For known content length, use multiple backspaces
agent.execute("Click at the end of the text field")
for _ in range(50):  # Exceed expected length
    computer.press("BackSpace")
computer.type(params.text)
```

**Why clearing is important:**
- Fields may have default/pre-filled values
- Previous workflow runs may leave data
- Concatenation errors occur if you don't clear first

### Validation Strategies

**Check failure indicators BEFORE success indicators:**

```python
from nen import Agent

agent = Agent()

# ❌ BAD - checks success first (prone to false positives)
if not agent.verify("Is there a dashboard?"):
    raise RuntimeError("Login failed")

# ✅ GOOD - checks failure first (more reliable)
if agent.verify("Are we still on the login page?"):
    raise RuntimeError("Login failed — still on login page")
if agent.verify("Is there an error message visible?"):
    raise RuntimeError("Login failed — error message displayed")
if not agent.verify("Did we navigate to a dashboard or different URL?"):
    raise RuntimeError("Unable to confirm login — dashboard not visible")
```

**Be specific in verification questions:**

```python
# ❌ VAGUE - prone to misinterpretation
agent.verify("Is the page loaded?")
agent.verify("Did it work?")

# ✅ SPECIFIC - clear and unambiguous
agent.verify("Is the email input field visible with a 'Send login link' button?")
agent.verify("Are we still on login.provetcloud.com URL?")
agent.verify("Is there an 'Invalid email' error message visible?")
```

**Always default to failure when state is unclear:**

```python
# Conservative approach - raise if we can't confirm success
if agent.verify("Success indicator 1") or agent.verify("Success indicator 2"):
    return result  # proceed to return data
raise RuntimeError("Unable to verify success — no expected state found")
```

### Keyboard Typing with Delays

For sensitive fields (passwords, forms with validation), add delays:

```python
from nen import Computer

computer = Computer()

# Standard typing (fast)
computer.type("username@example.com")

# Slower typing for better reliability (10ms between chars)
computer.type("password123", interval=0.01)

# Very slow for temperamental forms (50ms between chars)
computer.type("sensitive-data", interval=0.05)
```

### Mouse Coordinates

Direct mouse control when VLM-based agent.execute() is unreliable:

```python
from nen import Agent, Computer

agent = Agent()
computer = Computer()

# Wait for element to be visible first
if not agent.verify("Is the submit button visible?"):
    raise RuntimeError("Submit button not found")

# Click at specific coordinates
computer.click_at(500, 400)

# Move and click
computer.move(300, 200)
computer.click_at(300, 200)
```

**When to use direct mouse control:**
- Small clickable areas
- Elements without clear text labels
- Repeated clicks in same location
- Drawing or dragging operations

## Common Pitfalls

### 1. Platform-Specific Keyboard Commands

**Problem:** Using macOS "command" key in workflows

**Symptom:**
```
ERROR: xdotool key command failed: (symbol) No such key name 'command'
WorkflowError: Keyboard hotkey failed
```

**Solution:** Always use "ctrl" instead of "command"

### 2. Not Clearing Input Fields

**Problem:** Fields contain pre-existing content

**Symptom:** Concatenated text like "oldvaluetest_email"

**Solution:** Always select all + delete before typing

### 3. Validation False Positives

**Problem:** Workflow reports success when it actually failed

**Symptom:** VLM sees error messages but verification still passes

**Solution:** Check failure indicators first, be specific in questions

### 4. Timeout Issues

**Problem:** Operations take longer than expected

**Symptom:** Verification returns false before element appears

**Solution:** Increase timeout for slow operations:

```python
# Default timeout (10 seconds)
if not agent.verify("Is page loaded?"):
    pass

# Increased timeout for slow pages (30 seconds)
if not agent.verify("Is dashboard visible?", timeout=30):
    pass

# Custom max_iterations for agent operations
agent.execute("Fill out the form", max_iterations=20)  # More time/retries
```

### 5. Browser State Assumptions

**Problem:** Assuming clean browser state

**Symptom:** Unexpected cached data, cookies, or logged-in sessions

**Solution:** Always start fresh, check actual state:

```python
# Don't assume logged out
agent.execute("Click the Chromium browser icon in the taskbar")
agent.execute("Navigate to https://example.com")

# Check actual state first
if agent.verify("Are we already logged in (dashboard visible)?"):
    pass  # Already logged in — proceed to extract data

# Then proceed with login if needed
agent.execute("Click login button")
# ...
```

## Debugging Tips

### Reading VLM Logs

When logs show VLM reasoning, look for:

1. **What the VLM saw:**
   ```
   "screenshot shows the login page at login.provetcloud.com with email field 
    containing 'test_email' (marked as invalid)"
   ```

2. **What action was planned:**
   ```
   "I will click on the email input field"
   ```

3. **Verification results:**
   ```
   agent.verify("Are we still on login page?")
   Result: is_valid=true
   Match reason: "The page shows login form with logo..."
   ```

4. **Error messages:**
   ```
   ERROR: xdotool key command failed
   WorkflowError: Keyboard hotkey failed
   ```

### Common Error Patterns

**xdotool errors** → Platform-specific commands (use Linux conventions)

**WorkflowError: Validation timeout** → Increase timeout or check if element exists

**False validation success** → Make questions more specific, check failure first

**Concatenated text in fields** → Clear field before typing

## SDK Function Reference (Quick)

### Agent

```python
from nen import Agent
agent = Agent()
```

#### agent.execute(instruction, max_iterations=10)
Natural language action using vision language model.

```python
agent.execute("Click the blue submit button")
agent.execute("Fill in the entire registration form", max_iterations=20)
agent.execute("Navigate to the settings page")
```

#### agent.verify(condition, timeout=10, model=None)
Check UI state with a natural language condition. Returns bool.

```python
if agent.verify("Is the login form visible?"):
    pass

if not agent.verify("Is dashboard loaded?", timeout=30):
    raise RuntimeError("Dashboard did not load within 30 seconds")
```

#### agent.extract(query, schema)
Extract structured data using JSON schema. Returns dict/list.

```python
data = agent.extract(
    "What is the account balance?",
    schema={
        "type": "object",
        "properties": {
            "balance": {"type": "number"},
            "currency": {"type": "string"}
        },
        "required": ["balance"]
    }
)
# Returns: {"balance": 1250.50, "currency": "USD"}
```

### Computer

```python
from nen import Computer
computer = Computer()
```

#### computer.type(text, interval=0.02)
Type text with optional delay between characters.

```python
computer.type("username@example.com")
computer.type("password", interval=0.01)  # 10ms delay per char
```

#### computer.press(key)
Press a single key.

```python
computer.press("Return")
computer.press("BackSpace")
computer.press("Tab")
```

#### computer.hotkey(*keys)
Press multiple keys simultaneously (modifier combos).

```python
computer.hotkey("ctrl", "a")    # Select all
computer.hotkey("ctrl", "c")    # Copy
computer.hotkey("ctrl", "v")    # Paste
computer.hotkey("shift", "Tab") # Reverse tab
```

#### computer.click_at(x, y, button="left")
Click at specific coordinates.

```python
computer.click_at(500, 300)
computer.click_at(500, 300, button="right")
```

#### computer.move(x, y)
Move mouse to coordinates.

```python
computer.move(400, 200)
```

#### computer.scroll(direction="down", amount=3, *, x=None, y=None)
Scroll the mouse wheel.

```python
computer.scroll("down", 3)
computer.scroll("up", 5)
```

## For Complete Details

- **@workflow-guide-comprehensive** - Architecture, advanced patterns, debugging
- **@workflow-reference-detailed** - Complete SDK reference with all parameters
- **@workflow-core** - Quick start template and basic patterns
- **@mcp-platform-tools** - MCP tools for deployment, execution, debugging

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/getnenai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/getnenai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
