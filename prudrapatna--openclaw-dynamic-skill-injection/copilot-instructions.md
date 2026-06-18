## openclaw-dynamic-skill-injection

> Monitor channels/topics and dynamically load relevant skills based on context, auto-inject skill instructions into agent prompts, and enable real-time capability switching. Use when agents need adaptive capabilities based on conversation context, topic detection, or channel-specific requirements. Inspired by X/Twitter video pattern for real-time skill loading.


# Dynamic Skill Injection System

## Quick Start

1. **Configure topic mapping:** Edit `topic_mapping.yaml`
2. **Add skill metadata:** Edit `skill_metadata.yaml`  
3. **Enable for agents:** Set `skills.dynamic_injection.enabled: true` in agent config
4. **Test:** Send context-specific messages to see skills auto-load

## Core Components

### Topic Detection
- NLP analysis of conversation context
- Channel-specific topic mapping
- Real-time monitoring

### Skill Registry  
- YAML configuration mapping topics → skills
- Skill metadata and dependencies
- Priority and conflict resolution

### Dynamic Loading
- Auto-inject skill instructions into prompts
- Load/unload skills in real-time
- Manage skill dependencies

## Configuration Files

### 1. Topic Mapping (`topic_mapping.yaml`)
See [TOPIC_MAPPING.md](references/TOPIC_MAPPING.md) for complete guide.

### 2. Skill Metadata (`skill_metadata.yaml`)  
See [SKILL_METADATA.md](references/SKILL_METADATA.md) for schema.

### 3. Agent Integration
Add to agent config in `openclaw.json`:

```json
{
  "agentId": "agents-dwight",
  "skills": {
    "dynamic_injection": {
      "enabled": true,
      "mapping_path": "~/.openclaw/skills/dynamic-skill-injection/topic_mapping.yaml",
      "metadata_path": "~/.openclaw/skills/dynamic-skill-injection/skill_metadata.yaml"
    }
  }
}
```

## Use Cases

### Medical AI Context
When discussing "CGMacros", "insulin resistance", or "V7 model":
- Auto-loads: `cgmacros_dataloader`, `energy_based_inference`, `run_probe`
- Enables: Data loading, System 2 inference, evaluation

### Social Media Context  
When in `#kelly` channel or discussing "viral posts":
- Auto-loads: `tweet-ideas`, `typefully`, `headlines`
- Enables: Content drafting, scheduling, optimization

### Engineering Context
When in `#ross` channel or discussing "code review":
- Auto-loads: `superpowers-coding`, `fix-sentry-issues`
- Enables: Code analysis, debugging, refactoring

## Implementation

### Context Monitoring
Use `scripts/monitor_context.py` for topic detection:

```python
from monitor_context import ContextMonitor

monitor = ContextMonitor("topic_mapping.yaml")
topics = monitor.detect_topics("Evaluate V7 on CGMacros", channel="#dwight")
skills = monitor.get_skills_for_topics(topics)
```

### Skill Loading
Use `scripts/load_skills.py` for prompt injection:

```python
from load_skills import SkillLoader

loader = SkillLoader("skill_metadata.yaml")
injected_prompt = loader.inject_into_prompt(skills, base_prompt)
```

## Testing

Run `scripts/test_dynamic_injection.py`:

```bash
cd ~/.openclaw/skills/dynamic-skill-injection
python3 scripts/test_dynamic_injection.py
```

## Advanced Features

### 1. Cross-Agent Coordination
Skills can be shared across agents when topics overlap.

### 2. Skill Dependencies  
Complex dependency graphs with automatic resolution.

### 3. Performance Optimization
Lazy loading and caching for frequently used skills.

### 4. Conflict Resolution
Priority-based resolution when multiple skills conflict.

## References

- **Topic Mapping**: See [TOPIC_MAPPING.md](references/TOPIC_MAPPING.md)
- **Skill Metadata**: See [SKILL_METADATA.md](references/SKILL_METADATA.md)
- **Implementation Guide**: See [IMPLEMENTATION.md](references/IMPLEMENTATION.md)
- **Testing Guide**: See [TESTING.md](references/TESTING.md)

## Troubleshooting

**Problem:** Skills not loading
**Solution:** Check topic mapping and skill metadata paths

**Problem:** Performance issues  
**Solution:** Enable caching in skill loader

**Problem:** Skill conflicts
**Solution:** Adjust priority in skill metadata

## Next Steps

1. **Test with Dwight:** Medical AI context
2. **Test with Kelly:** Social media context  
3. **Test with Ross:** Engineering context
4. **Monitor performance:** Adjust as needed
5. **Add more skills:** Expand topic coverage

---
> Source: [prudrapatna/openclaw-dynamic-skill-injection](https://github.com/prudrapatna/openclaw-dynamic-skill-injection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
