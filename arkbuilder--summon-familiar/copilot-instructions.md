## summon-familiar

> Use when user wants persistent code review or documentation assistance.

# Skill: summon-familiar

Give AI assistants persistent, personality-driven companions that watch your work and offer commentary across sessions.

## What It Does

Summon Familiar creates **time-bounded AI companions** that:
- Spawn as background processes with distinct personalities
- Observe your codebase and workflow
- Send theatrical commentary via JSON stdout
- Respond to commands
- Track state across their lifespan
- Depart with a summary when dismissed or expired

Unlike single AI queries, familiars maintain **continuity** — they remember what they've seen and develop a running commentary on your work.

## Installation

```bash
npm install -g @arkbuilder/summon-familiar
```

## When to Use

- **Code Review**: "Summon a familiar to watch my commits and roast my variable names"
- **Documentation**: "Spawn a docs sprite to nag me when README drifts from code"
- **Pair Programming**: "I want a companion that stays with me for this 4-hour session"
- **Accountability**: "Something to observe and occasionally comment on my progress"

## Usage

### Summon

```bash
familiar summon --type code-review --name Snarkles --duration 2h
```

**Parameters:**
- `--type` (required): `code-review` | `docs-helper`
- `--name` (optional): Custom name (auto-generated if omitted)
- `--duration` (optional): `30m`, `2h`, `4h30m` (default: 4h, max: 24h)

**Returns:** JSON emergence message via stdout

### Command

```bash
familiar command Snarkles "review the auth module"
```

**Returns:** JSON response message via stdout

### Dismiss

```bash
familiar dismiss Snarkles
```

Immediately stops the familiar. Triggers departure message.

## Output Format

All communication is JSON via stdout:

```json
// Emergence (once on summon)
{"type": "emergence", "familiarName": "Snarkles", "content": "*The terminal flickers...*"}

// Observation (periodic, random)
{"type": "observation", "content": "Snarkles: Noticed a TODO comment..."}

// Response (to commands)
{"type": "response", "content": "Snarkles squints at the code..."}

// Departure (on expiry/dismissal)
{"type": "departure", "content": "*Snarkles prepares to depart...*"}
```

## Personalities

### code-review

**Role:** Watches git commits, comments on code quality  
**Voice:** Grouchy, sarcastic, secretly helpful  
**Observations:**
- Variable naming critiques
- console.log shaming
- TODO judgment
- Late-night commit commentary

**Example emergence:**
> *The terminal flickers. A small creature climbs out from between the lines of code.*
> 
> "I am Snarkles, summoned to judge thy commits. I see you have... work to do."

### docs-helper

**Role:** Watches README and docs, reminds about updates  
**Voice:** Earnest, organized, gently persistent  
**Observations:**
- Code/docs drift detection
- Empty section alerts
- README freshness checks

**Example emergence:**
> *A gentle glow emanates from your README.*
> 
> "Hello! I'm Lumina! I've been sent to help keep your documentation in sync!"

## Integration Examples

### With Codex CLI

```markdown
## Tool: summon-familiar

Use when user wants persistent code review or documentation assistance.

Spawn: familiar summon --type code-review --name <name> --duration <duration>
Capture stdout, parse JSON, relay emergence/observations/responses to user.
Dismiss: familiar dismiss <name> when user is done.
```

### With Claude Code

Add to your Claude Code configuration to make familiar available as a tool.

### With Custom AI Assistant

```javascript
const { spawn } = require('child_process');

function summonFamiliar(type, name, duration) {
  const familiar = spawn('familiar', [
    'summon', '--type', type, '--name', name, '--duration', duration
  ], { detached: true });
  
  familiar.stdout.on('data', (data) => {
    data.toString().split('\n').forEach(line => {
      if (!line.trim()) return;
      try {
        const msg = JSON.parse(line);
        if (['emergence', 'observation', 'response', 'departure'].includes(msg.type)) {
          console.log(msg.content); // Relay to user
        }
      } catch (e) {}
    });
  });
  
  return familiar;
}
```

## State & Storage

Familiar state persists in `~/.summon-familiar/`:
- State files: `{familiar-id}.json`
- Command queue: `commands/{id}-cmd.json`
- Dismissal signals: `commands/{id}-dismiss.json`

## Extending

Create custom personalities by extending the base class:

```javascript
class MyFamiliar {
  async onEmergence() { /* theatrical arrival */ }
  async onObservation() { /* periodic commentary */ }
  async onCommand(instruction) { /* handle user command */ }
  async onDeparture(contribution) { /* summary & farewell */ }
}
```

Place in `lib/personalities/` and register in the index.

## License

MIT

---
> Source: [arkbuilder/summon-familiar](https://github.com/arkbuilder/summon-familiar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
