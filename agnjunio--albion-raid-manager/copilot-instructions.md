## albion-raid-manager

> Documentation standards and guidelines for the project


# Documentation Standards

## Manual Documentation Management

This project uses Cursor Rules that are maintained by the AI Agent. Rules should be updated when major changes occur:

### When to Update Rules

- **New packages added** → Update [project-overview.mdc](mdc:.cursor/rules/project-overview.mdc)
- **New development commands** → Update [project-overview.mdc](mdc:.cursor/rules/project-overview.mdc) and [monorepo-standards.mdc](mdc:.cursor/rules/monorepo-standards.mdc)
- **Infrastructure changes** → Update [infrastructure-standards.mdc](mdc:.cursor/rules/infrastructure-standards.mdc)
- **Database schema changes** → Update [database-standards.mdc](mdc:.cursor/rules/database-standards.mdc)
- **API changes** → Update [api-standards.mdc](mdc:.cursor/rules/api-standards.mdc)
- **Bot changes** → Update [discord-bot-standards.mdc](mdc:.cursor/rules/discord-bot-standards.mdc)
- **Web app changes** → Update [react-standards.mdc](mdc:.cursor/rules/react-standards.mdc)
- **AI/ML changes** → Update [ai-package-standards.mdc](mdc:.cursor/rules/ai-package-standards.mdc)

### Documentation Files to Keep Updated

- **[apps/api/README.md](mdc:README.md)** - API project overview and quick start
- **[apps/bot/README.md](mdc:README.md)** - Bot project overview and quick start
- **[apps/web/README.md](mdc:README.md)** - Web project overview and quick start
- **[package.json](mdc:package.json)** - Scripts and dependencies
- **[turbo.json](mdc:turbo.json)** - Build configuration
- **[docker-compose.yml](mdc:docker-compose.yml)** - Infrastructure configuration
- **[packages/database/prisma/schema.prisma](mdc:packages/database/prisma/schema.prisma)** - Database schema

## Documentation Best Practices

### README.md Updates

- Keep development commands current
- Update project structure when packages change
- Document new features and breaking changes
- Maintain clear setup instructions

### Package.json Scripts

- Add new scripts for common operations
- Keep script names consistent and descriptive
- Group related scripts together

### Configuration Files

- Document all environment variables
- Keep configuration examples current
- Document breaking changes
- Maintain backward compatibility notes

### Database Schema

- Document all model relationships
- Keep enum values current
- Document migration strategies
- Maintain schema versioning

## Rule Maintenance

### Cursor Rules Structure

- **project-overview.mdc** - Always applied, general project info
- **technology-specific rules** - Applied to relevant file types
- **standards rules** - Applied to specific domains

### Manual Update Checklist

When making major changes, manually update:

- [ ] Update project overview with new structure
- [ ] Add new development commands to relevant rules
- [ ] Update technology-specific rules if needed
- [ ] Review and update related rules
- [ ] Test rule application
- [ ] Update documentation references

### Manual Validation

- **Review project structure** and ensure rules match current state
- **Verify file references** are correct and up-to-date
- **Test command examples** to ensure they work
- **Check rule consistency** across the project
- **Update rules** when major changes occur
- **Maintain rule quality** through manual review

---

description: Documentation standards and guidelines for the project
globs: ["**/*.md", "**/*.mdx", "**/README.md"]
alwaysApply: true

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agnjunio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
