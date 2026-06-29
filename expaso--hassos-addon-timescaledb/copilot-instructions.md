## hassos-addon-timescaledb

> This document provides guidelines for AI coding agents working on this Home Assistant addon. Following these guidelines ensures consistent, maintainable, and high-quality code.

# Agent Guidelines for Home Assistant TimescaleDB Addon Development

## Overview

This document provides guidelines for AI coding agents working on this Home Assistant addon. Following these guidelines ensures consistent, maintainable, and high-quality code.

## Code Quality Standards

### General Principles

- **Write clean, readable code**: Prioritize clarity over cleverness
- **Follow existing patterns**: Match the style and structure of the existing codebase
- **Document your changes**: Add comments for complex logic, especially in shell scripts
- **Test thoroughly**: Verify changes work in the Home Assistant addon environment
- **Keep it simple**: Avoid unnecessary complexity or over-engineering

### Shell Script Best Practices

- Use `#!/usr/bin/with-contenv bashio` for addon scripts that need Home Assistant integration
- Always quote variables: `"${variable}"` instead of `$variable`
- Use `bashio::log.*` functions for logging (info, warning, error, debug)
- Check return codes and handle errors gracefully
- Use meaningful variable names in ALL_CAPS for constants
- Add comments explaining non-obvious logic

### Docker Best Practices

- Minimize layer count in Dockerfile
- Clean up package manager caches after installations
- Use specific version pins where stability is critical
- Document why specific versions are chosen
- Follow multi-stage build patterns when applicable

## Project Structure Understanding

### Key Components

1. **timescaledb/**: Main addon directory
   - `config.yaml`: Addon configuration (options, schema, ports)
   - `Dockerfile`: Container build instructions
   - `build.yaml`: Build configuration for different architectures

2. **rootfs/**: Container root filesystem overlay
   - `etc/s6-overlay/s6-rc.d/`: Service definitions using s6-overlay
   - `usr/share/timescaledb/`: Initialization scripts

3. **docker-dependencies/**: Pre-built extension binaries

### Service Management (s6-overlay)

- Each service has its own directory under `s6-rc.d/`
- Required files: `type`, `run`, and optionally `finish`
- Use `dependencies.d/` to control service startup order
- Services should be resilient and handle failures gracefully

## Home Assistant Addon Documentation

### Essential Resources

- **Official Addon Documentation**: https://developers.home-assistant.io/docs/add-ons
- **Addon Configuration**: https://developers.home-assistant.io/docs/add-ons/configuration
- **Bashio Library**: https://github.com/hassio-addons/bashio (for interacting with Home Assistant)
- **s6-overlay**: https://github.com/just-containers/s6-overlay (service supervision)

### Key Addon Concepts

#### Configuration (`config.yaml`)

```yaml
options: # User-configurable options
  key: value
schema: # Validation schema for options
  key: type
```

#### Reading Configuration in Scripts

```bash
#!/usr/bin/with-contenv bashio

# Read option value
VALUE=$(bashio::config 'option_name')

# Check if option exists
if bashio::config.exists 'option_name'; then
    # Option is set
fi

# Get with default value
VALUE=$(bashio::config 'option_name' 'default_value')
```

#### Logging

```bash
bashio::log.info "Informational message"
bashio::log.warning "Warning message"
bashio::log.error "Error message"
bashio::log.debug "Debug message (only shown in debug mode)"
```

#### Service Scripts

- **run**: Main service execution script (should not exit unless service stops)
- **finish**: Cleanup script (runs when service stops)
- **type**: Service type (usually `longrun` for daemons)

## Development Workflow

### Planning

1. **Understand the requirement**: Read the issue/request carefully
2. **Review existing code**: Check how similar features are implemented
3. **Plan your approach**: Think through the changes before coding
4. **Stick to the plan**: Don't introduce unrelated changes

### Implementation

1. **Make focused changes**: One feature or fix per commit/PR
2. **Preserve working functionality**: Don't break existing features
3. **Follow the existing architecture**: Use established patterns
4. **Add appropriate logging**: Help users debug issues
5. **Update documentation**: Modify README.md if user-facing changes are made

### Testing Considerations

- Test initialization scripts handle both fresh installs and upgrades
- Verify configuration options are properly validated
- Check that services start and stop cleanly
- Ensure PostgreSQL extensions load correctly
- Test across different architectures if possible (amd64, aarch64, armv7)

## PostgreSQL & TimescaleDB Specifics

### Extension Management

- Extensions are installed in initialization scripts
- Use `CREATE EXTENSION IF NOT EXISTS` to avoid errors
- Check compatibility with PostgreSQL version
- Document any version-specific requirements

### Configuration Tuning

- `timescaledb-tune` is used for automatic configuration
- Custom settings can be added via `postgresql.conf.auto` or similar
- Memory settings should respect container limits
- Consider Home Assistant's resource constraints

## Common Pitfalls to Avoid

❌ **Don't:**

- Hardcode paths that should be configurable
- Ignore errors or return codes
- Make breaking changes to configuration schema without migration
- Assume specific PostgreSQL versions without checking
- Leave debug code or commented-out blocks
- Use `echo` for logging (use `bashio::log.*` instead)
- Break backward compatibility without documenting

✅ **Do:**

- Validate user input from configuration
- Provide sensible defaults
- Handle edge cases (missing files, permissions, etc.)
- Use appropriate exit codes
- Keep services idempotent where possible
- Document environment variables and their purposes

## Code Review Checklist

Before completing your work, verify:

- [ ] Code follows existing style and patterns
- [ ] All shell scripts have proper shebangs
- [ ] Variables are properly quoted
- [ ] Error handling is in place
- [ ] Logging uses bashio functions
- [ ] Configuration changes are reflected in `config.yaml` schema
- [ ] Services have proper dependencies defined
- [ ] Documentation is updated if needed
- [ ] No hardcoded values that should be configurable
- [ ] Changes are tested or testable

## Getting Help

When stuck:

1. Review existing code for similar patterns
2. Check Home Assistant addon documentation
3. Examine bashio library capabilities
4. Look at PostgreSQL/TimescaleDB documentation
5. Ask specific questions about the architecture or requirements

## Summary

**Core Principles:**

- 📝 Write clean, documented code
- 🎯 Stick to the plan and scope
- 🏗️ Follow existing patterns
- 🔍 Test thoroughly
- 📚 Document changes
- 🤝 Respect Home Assistant addon conventions

Your work contributes to a critical infrastructure component for Home Assistant users. Quality and reliability are paramount.

---
> Source: [expaso/hassos-addon-timescaledb](https://github.com/expaso/hassos-addon-timescaledb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
