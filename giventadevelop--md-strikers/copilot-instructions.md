## html-documentation-styling-guide

> HTML documentation styling guide — design system, code highlighting, command blocks, section containers, and accessibility for project HTML docs.


# HTML Documentation Styling Guide and Design System

## Overview
This rule defines the standard styling patterns, design system, and best practices for creating HTML documentation files in the project. It ensures consistency, accessibility, and user-friendly presentation across all documentation.

## Problem Solved
- **Consistent Documentation Styling**: Ensures all HTML documentation follows the same visual patterns
- **Code Highlighting**: Standardizes code syntax highlighting with light blue background and dark blue text
- **Copy Button Functionality**: Provides consistent copy-to-clipboard functionality for commands **and for copy-paste prompts** (see [Copyable prompt blocks](#copyable-prompt-blocks))
- **Windows Compatibility**: Ensures commands are provided in single-line format for Windows users
- **Visual Hierarchy**: Creates clear visual distinction between different types of content
- **Accessibility**: Ensures proper contrast and readable text

## Core Design Principles

### Color Scheme
- **Code/Command Blocks**: Light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`) and border (`#b8d4e3`)
- **Info Boxes**: Light blue background (`#d1ecf1`) with dark blue border (`#0c5460`)
- **Warning Boxes**: Light yellow background (`#fff3cd`) with yellow border (`#ffc107`)
- **Success Boxes**: Light green background (`#d4edda`) with green border (`#28a745`)
- **Error Boxes**: Light red background (`#f8d7da`) with red border (`#dc3545`)
- **Command Blocks**: Color-coded gradients based on script type (setup=blue, expedite=purple, etc.)

### Code Highlighting in Boxes
- **All code references** within info/warning/success/error boxes must use light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`) and border (`#b8d4e3`)
- Use `<code class="code-highlight">` for inline code that needs highlighting
- Standard `<code>` tags inherit box background (use `code-highlight` class for light blue/dark blue styling)

### Script Name Highlighting
- **All script file names** (e.g., `setup-test-clock.js`, `expedite-stripe-renewal-test-clock.js`) must use the `script-name` class
- Script names should be visually distinct with purple gradient background
- Use `<code class="script-name">script-name.js</code>` for all script file references

### Section Introduction Containers
- **Major section introductions** (like "Using the Setup Test Clock Script") should use `section-intro` class
- Provides light blue gradient background with blue left border
- Wraps introductory paragraphs that explain what a script or section does

### Parameters List Styling
- **Parameters sections** must use `parameters-list` class
- Orange gradient background with orange left border
- All parameter names in code tags should use light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`)

## Command Block Styling

### Color-Coded Command Blocks
Each script type has a distinct color scheme:

```css
/* Setup Scripts - Blue Gradient */
.command-block.setup {
    background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
    border-color: #2196f3;
}

/* Expedite Scripts - Purple Gradient */
.command-block.expedite {
    background: linear-gradient(135deg, #f3e5f5 0%, #e1bee7 100%);
    border-color: #9c27b0;
}

/* Advance Scripts - Orange Gradient */
.command-block.advance {
    background: linear-gradient(135deg, #fff3e0 0%, #ffe0b2 100%);
    border-color: #ff9800;
}

/* Verify Scripts - Green Gradient */
.command-block.verify {
    background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
    border-color: #4caf50;
}
```

### Command Block Structure with Parameter Customization
```html
<div class="command-block setup"
     data-command-type="setup"
     data-command-base="node scripts/path/to/script.js"
     data-template-command="node scripts/path/to/script.js --param1=value1 --param2=&quot;value2&quot;">
    <div class="command-header">
        <span class="command-type">Setup Script</span>
        <div class="command-header-buttons">
            <button class="copy-button-template" onclick="copyTemplateCommand(this, 'command-id')" title="Copy template command">📄 Template</button>
            <button class="copy-button" onclick="copyCommand(this, 'command-id')" title="Copy customized command">📋 Copy</button>
        </div>
    </div>
    <div class="command-parameters">
        <h5>🔧 Customize Parameters:</h5>
        <div class="parameter-group">
            <div class="parameter-input">
                <label for="command-id-param1">Parameter 1:</label>
                <input type="text" id="command-id-param1" placeholder="value1" value="default1" oninput="updateCommand('command-id', 'param1', this.value)">
            </div>
            <div class="parameter-input">
                <label for="command-id-param2">Parameter 2:</label>
                <input type="text" id="command-id-param2" placeholder="value2" value="default2" oninput="updateCommand('command-id', 'param2', this.value)">
            </div>
        </div>
        <div class="parameter-group single">
            <div class="parameter-input">
                <label for="command-id-param3">Parameter 3:</label>
                <input type="text" id="command-id-param3" placeholder="value3" oninput="updateCommand('command-id', 'param3', this.value)">
            </div>
        </div>
    </div>
    <div class="command-content" id="command-id">node scripts/path/to/script.js --param1=default1 --param2=default2</div>
    <div class="command-single-line" id="command-id-single">
        <div class="command-single-line-label">Windows Single-Line:</div>
        <code id="command-id-single-code">node scripts/path/to/script.js --param1=default1 --param2=default2</code>
    </div>
</div>
```

### Required Attributes
- **`data-command-base`**: Base command without parameters (e.g., `node scripts/path/to/script.js`)
- **`data-template-command`**: **REQUIRED** - Original template/sample command with example values (e.g., `node scripts/path/to/script.js --param1=value1 --param2=&quot;value2&quot;`)
  - Use `&quot;` for quotes in HTML attributes
  - This is what the Template button copies
- **`data-command-type`**: Script type for color coding (setup, expedite, advance, verify, default)
- **`id` on command-content**: Unique ID for the command (used by `updateCommand` and `copyCommand`)
- **`id="{command-id}-single-code"`**: ID for single-line code element (must include `-single-code` suffix)

### Required Copy Buttons
- **Template Button**: `copy-button-template` class, calls `copyTemplateCommand(this, 'command-id')`
- **Customized Button**: `copy-button` class, calls `copyCommand(this, 'command-id')`
- Both buttons must be wrapped in `command-header-buttons` div

### Windows Single-Line Sections
- **Background**: Light blue (`#e8f4f8`)
- **Text**: Dark blue (`#0d3b66`)
- **Border**: `#b8d4e3`
- **Label**: Light gray (`#cccccc`)
- **Padding**: `12px 15px`
- **Border Radius**: `4px`
- **Always provide single-line version** (no backslashes) for Windows compatibility

## Parameter Customization Feature

### Overview
All command blocks should include interactive parameter input fields that allow users to customize command parameters before copying. The command updates dynamically as users type, and the copy button copies the customized command.

### Parameter Input Styling
```css
.command-parameters {
    padding: 15px;
    background: rgba(255, 255, 255, 0.5);
    border-bottom: 1px solid rgba(0,0,0,0.1);
}

.command-parameters h5 {
    margin: 0 0 12px 0;
    font-size: 0.9em;
    color: #333;
    font-weight: bold;
}

.parameter-group {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    margin-bottom: 10px;
}

.parameter-group.single {
    grid-template-columns: 1fr;
}

.parameter-input {
    display: flex;
    flex-direction: column;
    gap: 4px;
}

.parameter-input label {
    font-size: 0.8em;
    color: #555;
    font-weight: 600;
}

.parameter-input input {
    padding: 8px 12px;
    border: 2px solid #ddd;
    border-radius: 4px;
    font-family: 'Courier New', monospace;
    font-size: 0.9em;
    transition: border-color 0.2s ease;
}

.parameter-input input:focus {
    outline: none;
    border-color: #0066cc;
    box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.1);
}

.parameter-input input::placeholder {
    color: #999;
    font-style: italic;
}

@media (max-width: 768px) {
    .parameter-group {
        grid-template-columns: 1fr;
    }
}
```

### JavaScript Functions Required

#### updateCommand Function
```javascript
// Store parameter values for each command block
const commandParams = {};

// Update command dynamically as user types
function updateCommand(commandId, paramName, paramValue) {
    // Store the parameter value
    if (!commandParams[commandId]) {
        commandParams[commandId] = {};
    }
    commandParams[commandId][paramName] = paramValue;

    // Get the command element
    const commandElement = document.getElementById(commandId);
    if (!commandElement) return;

    // Get the command block
    const commandBlock = commandElement.closest('.command-block');
    if (!commandBlock) return;

    // Get base command from data attribute
    let baseCommand = commandBlock.getAttribute('data-command-base');
    if (!baseCommand) {
        // Fallback: extract from current command
        const currentCommand = commandElement.textContent.trim();
        const firstParamIndex = currentCommand.indexOf('--');
        baseCommand = firstParamIndex > 0 ? currentCommand.substring(0, firstParamIndex).trim() : currentCommand.split(' ').slice(0, 2).join(' ');
    }

    // Build the command with all parameters
    let command = baseCommand;
    const params = commandParams[commandId] || {};

    // Add parameters based on command type
    // Example for setup script:
    if (commandId.startsWith('setup-1') || commandId.startsWith('setup-3')) {
        if (params['customer-id'] && params['customer-id'].trim()) {
            command += ` --customer-id=${params['customer-id'].trim()}`;
        }
        if (params['test-clock-name'] && params['test-clock-name'].trim()) {
            command += ` --test-clock-name="${params['test-clock-name'].trim()}"`;
        }
    } else if (commandId.startsWith('setup-2')) {
        if (params['customer-email'] && params['customer-email'].trim()) {
            command += ` --customer-email=${params['customer-email'].trim()}`;
        }
        if (params['customer-name'] && params['customer-name'].trim()) {
            command += ` --customer-name="${params['customer-name'].trim()}"`;
        }
        if (params['test-clock-name'] && params['test-clock-name'].trim()) {
            command += ` --test-clock-name="${params['test-clock-name'].trim()}"`;
        }
    }
    // Add more command types as needed...

    // Update the command display
    if (commandElement) {
        commandElement.textContent = command;
    }

    // Update the single-line version
    const singleLineCode = document.getElementById(commandId + '-single-code');
    if (singleLineCode) {
        singleLineCode.textContent = command;
    }
}
```

#### Two Copy Buttons: Template and Customized

Each command block must have **TWO copy buttons**:
1. **Template Button** (`copy-button-template`): Copies the original template/sample command
2. **Customized Button** (`copy-button`): Copies the command with user-entered parameters

```html
<div class="command-header-buttons">
    <button class="copy-button-template" onclick="copyTemplateCommand(this, 'command-id')" title="Copy template command">📄 Template</button>
    <button class="copy-button" onclick="copyCommand(this, 'command-id')" title="Copy customized command">📋 Copy</button>
</div>
```

#### Template Copy Button Styling
```css
.copy-button-template {
    background-color: #6c757d;
    color: white;
    border: none;
    padding: 6px 12px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.85em;
    font-weight: bold;
    transition: all 0.3s ease;
}

.copy-button-template:hover {
    background-color: #5a6268;
    transform: translateY(-1px);
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.copy-button-template.copied {
    background-color: #28a745;
}
```

#### Template Command Storage
Store the template command in the `data-template-command` attribute:

```html
<div class="command-block setup"
     data-command-base="node scripts/path/to/script.js"
     data-template-command="node scripts/path/to/script.js --param1=value1 --param2=&quot;value2&quot;">
```

**Note**: Use `&quot;` for quotes in HTML attributes, which will be converted to `"` when copying.

#### copyTemplateCommand Function
```javascript
function copyTemplateCommand(button, commandId) {
    // Find the command block from the button
    const commandBlock = button.closest('.command-block');
    if (!commandBlock) return;

    // Get template command from data attribute
    let templateCommand = commandBlock.getAttribute('data-template-command');
    if (!templateCommand) {
        // Fallback: use the initial command content
        const commandElement = document.getElementById(commandId);
        if (!commandElement) return;
        templateCommand = commandElement.textContent.trim();
    }

    // Replace HTML entities
    templateCommand = templateCommand.replace(/&quot;/g, '"');

    // Copy to clipboard
    if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(templateCommand).then(() => {
            button.classList.add('copied');
            button.textContent = '✓ Copied!';
            setTimeout(() => {
                button.classList.remove('copied');
                button.textContent = '📄 Template';
            }, 2000);
        }).catch(err => {
            console.error('Failed to copy template:', err);
            fallbackCopy(templateCommand, button);
        });
    } else {
        fallbackCopy(templateCommand, button);
    }
}
```

#### Updated copyCommand Function
The `copyCommand` function automatically uses the customized command from the `command-content` element, which is updated by `updateCommand`:

```javascript
function copyCommand(button, commandId) {
    const commandElement = document.getElementById(commandId);
    if (!commandElement) return;

    // Get the command text (already customized by updateCommand)
    const commandText = commandElement.textContent.trim();

    // Copy to clipboard (existing implementation)
    // ...
}
```

#### Command Initialization
Commands must be initialized on page load to read initial input values:

```javascript
// Initialize commands from input values on page load
function initializeCommands() {
    document.querySelectorAll('.command-block[data-command-base]').forEach(block => {
        const commandContent = block.querySelector('.command-content');
        if (!commandContent) return;

        const commandId = commandContent.id;
        if (!commandId) return;

        // Initialize parameters from input values
        block.querySelectorAll('.parameter-input input').forEach(input => {
            const inputId = input.id;
            if (!inputId || !inputId.startsWith(commandId + '-')) return;

            const paramName = inputId.substring(commandId.length + 1);
            const paramValue = input.value || '';

            if (!commandParams[commandId]) {
                commandParams[commandId] = {};
            }
            commandParams[commandId][paramName] = paramValue;
        });

        // Update command with initial values
        updateCommandFromParams(commandId);
    });
}

// Initialize on page load
document.addEventListener('DOMContentLoaded', function() {
    initializeCommands();
});
```

### Parameter Input Best Practices
- ✅ **Use appropriate input types**: `text`, `email`, `number` based on parameter type
- ✅ **Provide placeholders**: Show example values in placeholder text
- ✅ **Set default values**: Pre-fill common/default values
- ✅ **Use descriptive labels**: Clear labels indicating parameter name and format
- ✅ **Group related parameters**: Use `parameter-group` for 2-column layout, `single` for full-width
- ✅ **Handle quotes**: Automatically add quotes around values that may contain spaces
- ✅ **Validate input**: Use HTML5 validation (e.g., `type="email"` for email fields)

### Parameter Naming Convention
- Use kebab-case for parameter names: `customer-id`, `test-clock-name`, `days-to-advance`
- Match the actual command-line parameter names
- Use consistent naming across all command blocks

## Copy Button Implementation

### Copy Button Styling
```css
.copy-button {
    background-color: #0066cc;
    color: white;
    border: none;
    padding: 6px 12px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.85em;
    font-weight: bold;
    transition: all 0.3s ease;
}

.copy-button:hover {
    background-color: #0052a3;
    transform: translateY(-1px);
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.copy-button.copied {
    background-color: #28a745;
}

.copy-button.copied::after {
    content: " ✓ Copied!";
}
```

### Copy Button JavaScript
```javascript
function copyCommand(button, commandId) {
    const commandElement = document.getElementById(commandId);
    if (!commandElement) return;

    // Get the command text (already single-line for Windows)
    const commandText = commandElement.textContent.trim();

    // Copy to clipboard
    if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(commandText).then(() => {
            // Show success feedback
            button.classList.add('copied');
            button.textContent = '✓ Copied!';

            // Reset after 2 seconds
            setTimeout(() => {
                button.classList.remove('copied');
                button.textContent = '📋 Copy';
            }, 2000);
        }).catch(err => {
            console.error('Failed to copy:', err);
            fallbackCopy(commandText, button);
        });
    } else {
        // Fallback for older browsers
        fallbackCopy(commandText, button);
    }

    // Show single-line version for Windows
    const singleLineElement = document.getElementById(commandId + '-single');
    if (singleLineElement) {
        singleLineElement.classList.add('show');
    }
}
```

### Copy Button Requirements
- ✅ **Always provide single-line commands** (no backslashes) for Windows compatibility
- ✅ **Show visual feedback** when copied (green background, "✓ Copied!" text)
- ✅ **Include fallback** for older browsers that don't support Clipboard API
- ✅ **Auto-detect Windows** and show single-line hints automatically

## Copyable prompt blocks

### Scope
This applies whenever documentation includes **multi-line text meant to be copied as a single unit**—for example **prompts for AI image editors** (e.g. Nano Banana), **AI video tools** (e.g. Kling Omni), or **LLM instructions**—**not** shell/CLI commands (those use [Command Block Styling](#command-block-styling) above).

### Default rule: copy control is required
- **Yes — add a copy button by default** for every standalone prompt block. Readers should **not** have to manually select text in a `<pre>` or long paragraph.
- **One prompt block → one copy control** (e.g. **Copy** or **Copy prompt**) that copies the **entire** prompt string, preserving line breaks.

### Implementation pattern
- Wrap the block in a container (e.g. `prompt-block` or reuse `command-block` styling without fake CLI semantics) with:
  - A **header row**: short label (e.g. “Prompt”, “Full prompt”, “Kling”) + **`<button type="button" class="copy-button" ...>`** using the same [Copy Button Styling](#copy-button-styling) / feedback behavior as command copy buttons.
  - The prompt body in **one element** with a **unique `id`** (e.g. `id="prompt-encoding-kling-01"`) so `copyCommand(button, 'prompt-encoding-kling-01')` or a small `copyPromptText(button, id)` wrapper can reuse the existing `navigator.clipboard.writeText` pattern.
- Use **`white-space: pre-wrap`** (on a `<div>` or `<pre>`) so multi-line prompts display and copy correctly.
- **Accessibility:** `title` / `aria-label` on the button, e.g. `aria-label="Copy prompt to clipboard"`.

### What not to do
- ❌ **Do not** add prompts only inside a plain `<pre>` or `<div>` **without** a copy control when the intent is copy-paste.
- ❌ **Do not** rely on “users can select all” as a substitute for a button on long prompts.

### Relationship to command blocks
- **Command blocks** = executable `node ...` / script lines → **Template + Copy** + parameters as already specified.
- **Prompt blocks** = natural-language prompts → **at minimum one Copy** (no Template/customization unless you add optional fields).

## Code Highlighting in Content Boxes

### Info/Warning/Success/Error Boxes
All code references within these boxes must use light blue background with dark blue text:

```css
.info code,
.info .code-highlight,
.warning code,
.warning .code-highlight,
.success code,
.success .code-highlight,
.error code,
.error .code-highlight {
    background-color: #e8f4f8;
    color: #0d3b66;
    padding: 2px 6px;
    border-radius: 3px;
    border: 1px solid #b8d4e3;
    font-family: 'Courier New', monospace;
}
```

### Usage Pattern
```html
<div class="info">
    <strong>💡 Next Steps:</strong>
    <ol>
        <li>Verify the subscription has <code class="code-highlight">test_clock</code> set to your test clock ID</li>
        <li>Use <code class="code-highlight">expedite-stripe-renewal-test-clock.js</code> to advance time</li>
    </ol>
</div>
```

## Content Box Styling

### Info Boxes
- **Background**: `#d1ecf1` (light blue)
- **Border**: `4px solid #0c5460` (dark blue, left side)
- **Use for**: Informational content, tips, next steps

### Warning Boxes
- **Background**: `#fff3cd` (light yellow)
- **Border**: `4px solid #ffc107` (yellow, left side)
- **Use for**: Warnings, important notes, limitations

### Success Boxes
- **Background**: `#d4edda` (light green)
- **Border**: `4px solid #28a745` (green, left side)
- **Use for**: Success messages, confirmations, completed steps

### Error Boxes
- **Background**: `#f8d7da` (light red)
- **Border**: `4px solid #dc3545` (red, left side)
- **Use for**: Error messages, failures, critical issues

## Standard Code Block Styling

### Regular Code Blocks
```css
.code-block,
pre, pre code {
    background-color: #e8f4f8;
    color: #0d3b66;
    padding: 15px;
    border-radius: 5px;
    border: 1px solid #b8d4e3;
    overflow-x: auto;
    margin: 15px 0;
}

.code-block code,
pre code {
    background-color: transparent;
    color: #0d3b66;
    padding: 0;
}
```

### Inline Code
```css
code {
    background-color: #e8f4f8;
    color: #0d3b66;
    padding: 2px 6px;
    border-radius: 3px;
    border: 1px solid #b8d4e3;
    font-family: 'Courier New', monospace;
    font-size: 0.9em;
}
```

## Command Format Standards

### Windows Compatibility
- ✅ **Always provide single-line commands** (no backslashes `\`)
- ✅ **Remove line continuations** for Windows PowerShell/CMD
- ✅ **Use quotes properly** for Windows (double quotes)
- ✅ **Show Windows single-line version** in dedicated section

### Command Examples

**❌ DON'T (Unix-style with backslashes):**
```bash
node scripts/test-subscription-renewal/setup-test-clock.js \
  --customer-id=cus_xxx \
  --test-clock-name="My Test Clock"
```

**✅ DO (Windows-compatible single-line):**
```bash
node scripts/test-subscription-renewal/setup-test-clock.js --customer-id=cus_xxx --test-clock-name="My Test Clock"
```

## Section Container Styling

### Colored Section Containers
All major sections (Script Usage, Script Parameters, What the Script Does, Script Output, etc.) must be wrapped in colored section containers for better visibility and visual hierarchy.

```css
/* Default Section Container - Blue */
.section-container {
    background: linear-gradient(135deg, #f0f7ff 0%, #e6f2ff 100%);
    border-left: 4px solid #0066cc;
    border-radius: 8px;
    padding: 20px;
    margin: 20px 0;
    box-shadow: 0 2px 8px rgba(0, 102, 204, 0.1);
}

/* Parameters Section - Orange */
.section-container.parameters {
    background: linear-gradient(135deg, #fff5e6 0%, #ffe6cc 100%);
    border-left-color: #ff9800;
}

/* Output Section - Green */
.section-container.output {
    background: linear-gradient(135deg, #e6ffe6 0%, #ccffcc 100%);
    border-left-color: #28a745;
}

/* Usage Section - Purple */
.section-container.usage {
    background: linear-gradient(135deg, #f0e6ff 0%, #e6ccff 100%);
    border-left-color: #9c27b0;
}
```

### Section Headers
Each section container must have a colored header with icon and title:

```css
.section-header {
    background: linear-gradient(135deg, #0066cc 0%, #0052a3 100%);
    color: white;
    padding: 12px 20px;
    margin: -20px -20px 20px -20px;
    border-radius: 8px 8px 0 0;
    font-size: 1.2em;
    font-weight: bold;
    display: flex;
    align-items: center;
    gap: 10px;
}

.section-header.parameters {
    background: linear-gradient(135deg, #ff9800 0%, #f57c00 100%);
}

.section-header.output {
    background: linear-gradient(135deg, #28a745 0%, #1e7e34 100%);
}

.section-header.usage {
    background: linear-gradient(135deg, #9c27b0 0%, #7b1fa2 100%);
}
```

### Section Container Structure
```html
<div class="section-container usage">
    <div class="section-header usage">📋 Script Usage</div>
    <!-- Section content -->
</div>

<div class="section-container parameters">
    <div class="section-header parameters">⚙️ Script Parameters</div>
    <!-- Table or content -->
</div>

<div class="section-container output">
    <div class="section-header output">📊 Script Output</div>
    <!-- Output description -->
</div>
```

### Section Type Guidelines
- **Script Usage**: Use `usage` class (purple gradient)
- **Script Parameters**: Use `parameters` class (orange gradient)
- **What the Script Does**: Use default (blue gradient)
- **Why This Script is Important**: Use default (blue gradient)
- **Script Output**: Use `output` class (green gradient)
- **Example Script Output**: Use `output` class (green gradient)

## Script Name Highlighting

### Script Name Styling
All script file names must be highlighted with a distinctive purple gradient:

```css
.script-name {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 4px 10px;
    border-radius: 5px;
    font-family: 'Courier New', monospace;
    font-weight: bold;
    display: inline-block;
    box-shadow: 0 2px 4px rgba(102, 126, 234, 0.3);
}
```

### Usage
```html
<!-- ✅ DO: Use script-name class for all script file references -->
<p>The <code class="script-name">setup-test-clock.js</code> script is a dedicated tool...</p>
<p>Use <code class="script-name">expedite-stripe-renewal-test-clock.js</code> to advance time</p>

<!-- ❌ DON'T: Use plain code tags for script names -->
<p>The <code>setup-test-clock.js</code> script...</p>
```

## Section Introduction Containers

### Section Intro Styling
Major section introductions should use a light blue gradient container:

```css
.section-intro {
    background: linear-gradient(135deg, #e8f4f8 0%, #d1e7f0 100%);
    border-left: 4px solid #0066cc;
    border-radius: 8px;
    padding: 20px;
    margin: 20px 0;
    box-shadow: 0 2px 8px rgba(0, 102, 204, 0.1);
}

.section-intro code {
    background-color: #e8f4f8;
    color: #0d3b66;
    padding: 3px 8px;
    border-radius: 3px;
    border: 1px solid #b8d4e3;
}
```

### Usage
```html
<!-- ✅ DO: Wrap section introductions in section-intro container -->
<div class="section-intro">
    <p>The <code class="script-name">setup-test-clock.js</code> script is a dedicated tool...</p>
</div>
```

## Parameters List Styling

### Parameters List Container
Parameters sections should use an orange gradient container:

```css
.parameters-list {
    background: linear-gradient(135deg, #fff5e6 0%, #ffe6cc 100%);
    border-left: 4px solid #ff9800;
    border-radius: 8px;
    padding: 15px 20px;
    margin: 15px 0;
}

.parameters-list strong {
    color: #ff9800;
    font-weight: bold;
}

.parameters-list code {
    background-color: #e8f4f8;
    color: #0d3b66;
    padding: 3px 8px;
    border-radius: 3px;
    border: 1px solid #b8d4e3;
}
```

### Usage
```html
<!-- ✅ DO: Wrap parameters lists in parameters-list container -->
<div class="parameters-list">
    <p><strong>Parameters:</strong></p>
    <ul>
        <li><code>name</code> - A descriptive name for the test clock</li>
        <li><code>frozen_time</code> - Unix timestamp for the initial time</li>
    </ul>
</div>
```

## Table Styling

### Enhanced Table Styling
Tables must have colored headers and hover effects for better visibility:

```css
table {
    border-collapse: collapse;
    width: 100%;
    margin: 15px 0;
    background: white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

th, td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: left;
}

th {
    background: linear-gradient(135deg, #0066cc 0%, #0052a3 100%);
    color: white;
    font-weight: bold;
    text-transform: uppercase;
    font-size: 0.9em;
    letter-spacing: 0.5px;
}

tr:nth-child(even) {
    background-color: #f8f9fa;
}

tr:hover {
    background-color: #e3f2fd;
    transition: background-color 0.2s ease;
}

td code {
    background-color: #e8f4f8;
    color: #0d3b66;
    padding: 3px 8px;
    border-radius: 3px;
    border: 1px solid #b8d4e3;
    font-family: 'Courier New', monospace;
}
```

### Table Requirements
- ✅ **Colored headers**: Blue gradient background with white text
- ✅ **Hover effects**: Light blue background on row hover
- ✅ **Code in cells**: Light blue background with dark blue text
- ✅ **Alternating rows**: Light gray background for even rows
- ✅ **Shadow**: Subtle shadow for depth

## Typography Standards

### Headings
- **H1**: `2.5em`, border-bottom `3px solid #0066cc`
- **H2**: `1.8em`, border-left `4px solid #0066cc`, padding-left `15px`
- **H3**: `1.4em`, color `#333`
- **H4**: `1.2em`, color `#555`

### Body Text
- **Font Family**: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif`
- **Line Height**: `1.6`
- **Color**: `#333`
- **Font Size**: `1em` (base)

### Code Font
- **Font Family**: `'Courier New', monospace`
- **Font Size**: `0.9em` (inline), `0.95em` (blocks)

## Responsive Design

### Container
```css
body {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

@media (max-width: 768px) {
    body {
        padding: 10px;
    }

    .command-block {
        font-size: 0.85em;
    }
}
```

## Accessibility Requirements

### Color Contrast
- ✅ **Text on white**: Minimum `#333` (WCAG AA compliant)
- ✅ **Text on light blue code blocks**: Dark blue (`#0d3b66`) for good contrast
- ✅ **Interactive elements**: Clear hover states and focus indicators

### Keyboard Navigation
- ✅ All buttons must be keyboard accessible
- ✅ Focus states visible (outline or ring)
- ✅ Tab order logical

### Screen Readers
- ✅ Semantic HTML (`<button>`, `<nav>`, `<main>`, etc.)
- ✅ ARIA labels where needed
- ✅ Alt text for images
- ✅ Descriptive link text

## Color Contrast and Visibility Standards

### CRITICAL: Avoid Gray Backgrounds
- ❌ **NEVER use gray backgrounds** (`#f2f2f2`, `#f9f9f9`, `#f4f4f4`, etc.) for important sections
- ✅ **ALWAYS use colored section containers** for Script Usage, Script Parameters, Script Output, etc.
- ✅ **Use gradient backgrounds** with clear color distinction
- ✅ **Ensure high contrast** between text and background (WCAG AA minimum)

### Section Visibility Requirements
All major sections must use colored containers:
- **Script Usage**: Purple gradient container (`usage` class)
- **Script Parameters**: Orange gradient container (`parameters` class)
- **What the Script Does**: Blue gradient container (default)
- **Why This Script is Important**: Blue gradient container (default)
- **Script Output**: Green gradient container (`output` class)
- **Example Script Output**: Green gradient container (`output` class)

### Table Visibility Requirements
- ❌ **NEVER use gray table headers** (`#f2f2f2`)
- ✅ **ALWAYS use colored gradient headers** (blue gradient with white text)
- ✅ **Include hover effects** for better interactivity
- ✅ **Code in table cells** must use light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`)

### Code Visibility Requirements
- ❌ **NEVER use gray backgrounds** for code (`#f4f4f4`, `#f2f2f2`)
- ❌ **NEVER use charcoal/black backgrounds** (`#1a1a1a`, `#2d2d2d`) for code
- ✅ **ALWAYS use light blue background** (`#e8f4f8`) with dark blue text (`#0d3b66`) and border (`#b8d4e3`) for code
- ✅ **Apply to all code** in content boxes, tables, and command blocks
- ✅ **Use `code-highlight` class** for inline code that needs highlighting

## Best Practices

### DO
- ✅ Use color-coded command blocks for different script types
- ✅ Always provide Windows single-line commands
- ✅ Use light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`) for code in content boxes
- ✅ Include copy buttons for all commands
- ✅ **Include a copy button for every copy-paste AI/video/editor prompt block** (see [Copyable prompt blocks](#copyable-prompt-blocks))
- ✅ Show visual feedback when copying
- ✅ Use semantic HTML elements
- ✅ Maintain consistent spacing and padding
- ✅ Use descriptive class names
- ✅ Include fallback for older browsers
- ✅ **Wrap all major sections in colored containers** (no plain gray sections)
- ✅ **Use colored table headers** (blue gradient, not gray)
- ✅ **Ensure all code has light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`)**

### DON'T
- ❌ Use gray backgrounds for code highlighting (use light blue/dark blue)
- ❌ Use gray backgrounds for section containers (use colored gradients)
- ❌ Use gray table headers (use colored gradient headers)
- ❌ Use backslashes in commands (provide single-line versions)
- ❌ Skip copy buttons for commands
- ❌ **Ship long prompts without a copy button** (prompt blocks must include copy control by default)
- ❌ Use inline styles (use CSS classes)
- ❌ Mix different styling patterns
- ❌ Skip Windows compatibility
- ❌ Use low contrast colors
- ❌ Forget accessibility features
- ❌ **Leave sections in plain gray** (always use colored containers)
- ❌ **Use plain code tags for script names** (always use `script-name` class)
- ❌ **Leave section introductions without containers** (use `section-intro` class)
- ❌ **Leave parameters sections without highlighting** (use `parameters-list` class)
- ❌ **Skip parameter customization inputs** (always include `command-parameters` section)
- ❌ **Forget `data-command-base` attribute** (required for dynamic command updates)
- ❌ **Forget `data-template-command` attribute** (required for Template copy button)
- ❌ **Use only one copy button** (must have both Template and Customized buttons)
- ❌ **Use static command text only** (commands should update as users type)
- ❌ **Skip command initialization** (must call `initializeCommands()` on page load)
- ❌ **Skip single-line code element ID** (must have `-single-code` suffix for updates)

## File Structure Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="...">
    <title>Documentation Title</title>
    <style>
        /* Include all standard styles */
    </style>
</head>
<body>
    <h1>Documentation Title</h1>

    <div class="toc">
        <h2>Table of Contents</h2>
        <!-- TOC links -->
    </div>

    <!-- Content sections -->

    <script>
        // Copy button functionality
        // Windows detection
    </script>
</body>
</html>
```

## Reference Implementations

- **Test Clocks Guide**: [`documentation/domain_agnostic_payment/membership_susbscription/STRIPE_TEST_CLOCKS_GUIDE.html`](mdc:documentation/domain_agnostic_payment/membership_susbscription/STRIPE_TEST_CLOCKS_GUIDE.html)
  - Complete example with parameter customization for `setup-test-clock.js` and `expedite-stripe-renewal-test-clock.js`
  - Shows all styling patterns: section containers, script name highlighting, parameters lists, interactive command blocks
- **Testing Scripts Guide**: [`documentation/domain_agnostic_payment/membership_susbscription/SUBSCRIPTION_RENEWAL_TESTING_SCRIPTS_GUIDE.html`](mdc:documentation/domain_agnostic_payment/membership_susbscription/SUBSCRIPTION_RENEWAL_TESTING_SCRIPTS_GUIDE.html)

## Summary Checklist

Before submitting HTML documentation:

- [ ] All code references in info/warning/success boxes use light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`)
- [ ] All commands have copy buttons
- [ ] **All copy-paste prompt blocks** (AI/video/editor prompts) **have a copy button** — not prompt-only `<pre>` without a control
- [ ] All commands are provided in Windows-compatible single-line format
- [ ] Command blocks are color-coded by script type
- [ ] Windows single-line sections have light blue background with dark blue text
- [ ] Copy buttons show visual feedback when clicked
- [ ] **All major sections (Script Usage, Parameters, Output, etc.) are wrapped in colored section containers**
- [ ] **Section headers have colored gradient backgrounds with white text**
- [ ] **Section introductions use `section-intro` class with light blue gradient**
- [ ] **All script file names use `script-name` class with purple gradient**
- [ ] **Parameters sections use `parameters-list` class with orange gradient**
- [ ] **Table headers use blue gradient background with white text (not gray)**
- [ ] **Code in table cells uses light blue background with dark blue text**
- [ ] **No gray backgrounds for important sections** (use colored gradients)
- [ ] **Command blocks include parameter customization inputs** (use `command-parameters` section)
- [ ] **All command blocks have `data-command-base` attribute** (base command without parameters)
- [ ] **All command blocks have `data-template-command` attribute** (template/sample command for Template button)
- [ ] **Two copy buttons are present** (Template button and Customized button)
- [ ] **Template button copies original template command** (uses `data-template-command` attribute)
- [ ] **Customized button copies command with user parameters** (uses updated `command-content`)
- [ ] **Parameter inputs update command dynamically** (use `updateCommand` function)
- [ ] **Commands are initialized on page load** (read initial input values)
- [ ] **Single-line code element has `-single-code` suffix ID** (for dynamic updates)
- [ ] Table of contents is included with anchor links
- [ ] Responsive design works on mobile devices
- [ ] Accessibility features are included (ARIA labels, keyboard navigation)
- [ ] Color contrast meets WCAG AA standards
- [ ] All styling uses CSS classes (no inline styles)
- [ ] Consistent spacing and padding throughout
- [ ] **Hover effects on table rows for better interactivity**

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
