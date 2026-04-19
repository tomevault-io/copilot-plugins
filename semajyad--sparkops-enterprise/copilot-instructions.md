## sparkops-enterprise

> Slack Notification Rule for User Input Requirements


# 📱 Slack Notification Rule for User Input

This rule automatically sends Slack notifications when Cascade needs user input for file changes, terminal operations, or any manual intervention.

## Rule Activation

This rule is automatically triggered when ANY of these conditions occur:
- File changes have been accepted and system is ready to test
- Work has been completed and successfully pushed to git
- Terminal command needs user input (passwords, confirmations)
- Manual intervention is required for any operation
- Process is blocked waiting for user action

## Notification Triggers

### 1. Build/Test Ready Notifications
**Trigger**: When file changes have been accepted and the system is ready to proceed with testing
```bash
# Detection pattern: File changes accepted, build/test environment ready
# Notification: "🚀 Ready to test - Build environment prepared"
# Action: Send Slack message when all changes are accepted and testing can begin
```

### 2. Git Push Complete Notifications
**Trigger**: When work has been completed and successfully pushed to git
```bash
# Detection pattern: Git push completed successfully
# Notification: "✅ Work completed and pushed to git"
# Action: Send Slack message when changes are committed and pushed
```

### 3. Terminal Input Required
**Trigger**: When terminal command waits for user input
```bash
# Detection patterns:
# - Password prompts: "Password:", "Enter password"
# - Y/N confirmations: "Continue? [y/n]", "Are you sure? [y/n]"
# - Any input prompt: "Press Enter to continue", "Input required"

# Notification: "⚠️ Terminal input required - [command details]"
# Action: Send Slack message with command and input needed
```

### 4. Process Blocked Notifications
**Trigger**: When any process is blocked waiting for user action
```bash
# Detection patterns:
# - Hanging processes (>30 seconds)
# - Interactive prompts
# - Manual intervention required

# Notification: "🚫 Process blocked - needs manual intervention"
# Action: Send Slack message with process details
```

## Automatic Notification System

### Slack Integration Setup
The rule uses the configured Slack MCP server with these parameters:
- **Bot Token**: Configured in MCP settings
- **Default Channel**: Your DM channel
- **Message Format**: Structured with emojis and context
- **Priority Levels**: Based on urgency of input needed

### Message Templates

#### Git Push Complete Template
```
✅ **Work Completed & Pushed**

🎯 **Status**: Successfully pushed to git
📁 **Repository**: [repository name]
🌿 **Branch**: [branch name]
📝 **Commit**: [commit message/ID]
📊 **Files**: [number] files committed
⏰ **Time**: [timestamp]

💡 _All work has been completed and pushed to git successfully_
```

#### Build/Test Ready Template
```
� **Ready to Test**

✅ **File Changes**: Accepted and ready
� **Environment**: Build/test environment prepared
� **Files Modified**: [number] files processed
⏰ **Time**: [timestamp]
🎯 **Action**: Testing can now proceed automatically

💡 _All file changes have been accepted and Cascade is ready to continue with testing_
```

#### Terminal Input Template
```
⚠️ **Terminal Input Required**

💻 **Command**: [command that needs input]
🔑 **Input Type**: [password/confirmation/other]
📝 **Prompt**: [exact prompt text]
⏰ **Time**: [timestamp]

💡 _Cascade is waiting for your input to continue the command_
```

#### Process Blocked Template
```
🚫 **Process Blocked**

🔄 **Process**: [process name/command]
⏱️ **Duration**: [time blocked]
📍 **Location**: [working directory]
🎯 **Action**: Manual intervention required

💡 _Cascade is blocked and needs your help to continue_
```

## Implementation Details

### 1. Build/Test Ready Detection
```bash
# Monitor for file changes acceptance and build readiness
monitor_build_readiness() {
  # Wait for file changes to be accepted by user
  # Check if build/test environment is ready
  # Send notification when ready to proceed
  # Only notify after acceptance, not before
}
```

### 2. Git Push Detection
```bash
# Monitor for successful git push operations
monitor_git_push_complete() {
  # Detect git push completion
  # Extract commit details and repository info
  # Send success notification with commit details
  # Only notify on successful push
}
```

### 3. Terminal Input Detection
```bash
# Monitor terminal processes for input prompts
monitor_terminal_input() {
  # Scan for password prompts
  # Detect Y/N confirmations  
  # Identify any interactive prompts
  # Send notification with details
}
```

### 4. Process Monitoring
```bash
# Monitor for blocked processes
monitor_blocked_processes() {
  # Check for processes waiting > 30 seconds
  # Detect hanging operations
  # Send notification for manual intervention
}
```

## Notification Logic

### Priority Levels
1. **HIGH** (Immediate): Git push complete notifications
2. **HIGH** (Immediate): Build/test ready notifications
3. **MEDIUM** (5 min delay): Terminal input needed
4. **LOW** (10 min delay): Process monitoring alerts

### Rate Limiting
- **Max notifications**: 3 per hour
- **Cooldown period**: 15 minutes between similar notifications
- **Build ready batching**: Only notify once when all changes are accepted
- **Git push batching**: Only notify once per successful push operation

### Smart Filtering
- **Wait for acceptance**: Don't notify about pending file changes
- **Ready-state only**: Only notify when system is ready to proceed
- **Context awareness**: Different notifications for different scenarios

## Integration with Skills

### @workflow-monitor
- Detects when processes are blocked
- Monitors terminal input requirements
- Tracks git push completion and build readiness
- Triggers notifications based on conditions

### @self-healing
- Attempts automatic resolution first
- Only notifies when manual intervention is truly needed
- Provides context for the required action

### @quality-verification
- Monitors for file change acceptance completion
- Detects when build/test environment is ready
- Tracks git operations and completion status
- Coordinates with notification system for ready-state notifications

## Configuration Options

### Notification Settings
```json
{
  "slack_notifications": {
    "enabled": true,
    "default_channel": "@your_username",
    "priority_levels": {
      "git_push_complete": "HIGH",
      "build_ready": "HIGH",
      "terminal_input": "MEDIUM", 
      "process_blocked": "MEDIUM"
    },
    "rate_limits": {
      "max_per_hour": 3,
      "cooldown_minutes": 15
    },
    "filters": {
      "wait_for_acceptance": true,
      "ready_state_only": true,
      "git_success_only": true,
      "batch_similar": true,
      "context_aware": true
    }
  }
}
```

### Custom Message Templates
```json
{
  "message_templates": {
    "git_push_complete": "✅ Work completed and pushed to git",
    "build_ready": "🚀 Build environment ready - Testing can proceed",
    "terminal_input": "⚠️ Terminal needs input: {command}",
    "process_blocked": "🚫 Process blocked: {process}"
  }
}
```

## Testing and Verification

### Test Scenarios
1. **Build Ready**: Make file changes, accept them, verify notification when ready
2. **Git Push Complete**: Commit and push changes, verify success notification
3. **Terminal Input**: Run a command that needs user input
4. **Process Block**: Create a hanging process scenario
5. **Rate Limiting**: Send multiple notifications to verify limits

### Verification Steps
1. **Notification Received**: Check Slack for message
2. **Message Content**: Verify all required details are included
3. **Timing**: Check notification timing is appropriate
4. **Actionability**: Ensure message provides clear next steps

## Troubleshooting

### Common Issues
1. **No Notifications**: Check Slack bot permissions and token
2. **Too Many Notifications**: Adjust rate limiting settings
3. **Wrong Channel**: Verify default channel configuration
4. **Missing Context**: Update message templates

### Debug Mode
```bash
# Enable debug logging
export SLACK_NOTIFICATIONS_DEBUG=true

# Test notification manually
send_test_notification()
```

## Benefits

### Proactive Communication
- **No more wondering**: Always know when you're needed
- **Clear context**: Detailed information about what's needed
- **Actionable messages**: Specific next steps provided
- **Timely alerts**: Notifications sent at appropriate times

### Improved Workflow
- **Reduced delays**: Faster response to blocked operations
- **Better coordination**: Clear communication about needs
- **Less frustration**: No more guessing what's needed
- **Productivity gains**: Smoother workflow transitions

### Automation Enhancement
- **Smart detection**: Only notifies when truly needed
- **Context awareness**: Different messages for different situations
- **Self-healing first**: Attempts automatic resolution before notifying
- **Learning system**: Improves notification accuracy over time

---

**This rule ensures you're notified when the system is ready to test after file changes are accepted, keeping the workflow moving smoothly while waiting for your input only when truly needed.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/semajyad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
