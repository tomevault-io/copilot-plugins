## voicescribestudio

> - UI Layer (Gradio components in app/components/)

# VoiceScribe Studio - Cursor Rules

## Project Architecture Patterns

### Multi-Tier Architecture
- UI Layer (Gradio components in app/components/)
- Core Logic (app/main.py, app/utils/)
- LLM Layer (Primary DeepSeek, Fallback Claude)
- TTS Layer (Elevenlabs via elevenlabs_client.py)
- Token Tracking Layer (SQLite database)
- Reference Processing Layer (Document, Web, YouTube)
- Future MCP Layer (Model Context Protocol servers)

### Voice Loading Pattern
- Use consolidated `load_all_voices()` function in elevenlabs_client.py as the single source of truth
- Rely on hardcoded preset voices for reliability
- Also attempt to fetch additional voices from API with timeout to prevent delays
- Handle API failures gracefully with fallback to preset voices
- Maintain Dan Teacher voices at top of the list
- Use backward-compatible aliases to maintain API consistency

### Component Organization
- Keep UI components in app/components/
- Keep utility functions and clients in app/utils/
- Keep reference handlers in app/utils/reference_handlers/
- Documentation in docs/ directory
- Test framework in app/tests/

## Development Patterns

### LLM Integration
- Always use Primary/Fallback pattern for script generation
- Use token_tracker.track_generation() for ALL LLM API calls
- Set is_test=True for all test-related API calls
- Include both successful and failed API attempts in token tracking
- Implement robust error handling with graceful degradation

### Token Management
- Use the singleton token_tracker for consistent tracking
- Ensure test and production tracking are clearly separated
- Always include template attribution in token tracking
- Provide cost estimates before potentially expensive operations

### Template System
- Always use template-specific guidance from get_template_guidance()
- Ensure template markers don't appear in final output
- Track which templates are used for analytics purposes

### UI Construction
- Use consistent element styling across components
- Include clear visual feedback for user actions
- Implement loading indicators for all API calls
- Use tab-based organization for different functionalities
- Maintain separation between public and admin interfaces

### Error Handling
- Implement multiple fallback strategies for content extraction
- Provide user-friendly error messages
- Log detailed errors for debugging purposes
- Return graceful defaults when operations fail

### Voice Generation
- Prioritize voice loading from main.py for consistency
- Always clean scripts of problematic characters (↗↘)
- Use the humanize feature for professional voiceover markup

## Documentation Requirements

- Keep memory-bank files updated with latest component details
- Document all new features in both code and user-facing instructions
- Maintain README.md with current features and setup instructions
- Create dedicated documentation files for complex features
- Update CHANGELOG.md when implementing significant changes

## Future Integration Guidelines

- Prepare for MCP server integration for specialized tasks
- Plan for translation feature with template-specific glossaries
- Design for media enhancement capabilities (music, visuals)
- Consider premium tier feature separation 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MisterV111) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
