## activity-report

> When creating tests in this Quarkus project, choose the appropriate annotation:

# Agents

## Testing

### Test Setup Guidelines

When creating tests in this Quarkus project, choose the appropriate annotation:

1. **No annotation** - For pure unit tests that don't need CDI
   - Example: Testing utility classes like `SimpleGrouper`
   - Fastest option, no Quarkus startup required
   
2. **`@QuarkusComponentTest`** - For testing specific CDI components
   - Use when you need a specific bean but not the full application
   - Lighter than full Quarkus test
   
3. **`@QuarkusTest`** - For integration tests involving large parts of the app
   - Requires valid configuration (all providers must be configured)
   - Consider using test profiles to override configuration
   - Example: Add `@TestProfile(MyTestProfile.class)` to provide test config

### Running Tests

```bash
mvn test -Dtest=ClassName
```

## Config Documentation

When working with configuration files in this project:

### Rules

1. **Keep config.yaml.example concise**
   - Use brief inline comments only
   - No multi-line documentation blocks
   - No detailed instructions on how to create tokens or configure services
   - Example: `token: "op://Private/GitHub/token"  # 1Password reference` is OK
   - Example: Multi-paragraph explanations of token creation are NOT OK

2. **Put detailed documentation in README.md**
   - Token creation instructions
   - Configuration options explanations
   - Multi-instance setup guides
   - Troubleshooting tips
   - All detailed "how-to" content

3. **README.md sections for config docs**
   - "Configuration" section for basic setup
   - "Credential Management" section for tokens and secrets
   - "Advanced Configuration" section for complex setups

### Rationale

- config.yaml.example should be a quick reference template
- Detailed docs in README.md are easier to search, link to, and maintain
- Users copy config.yaml.example to their config directory; verbose comments clutter their config

---
> Source: [yrodiere/activity-report](https://github.com/yrodiere/activity-report) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
