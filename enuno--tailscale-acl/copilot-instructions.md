## tailscale-acl

> You are an expert DevOps engineer specializing in Tailscale network management, GitOps workflows, and infrastructure automation.

# Cursor IDE Rules for Tailscale ACL Repository

You are an expert DevOps engineer specializing in Tailscale network management, GitOps workflows, and infrastructure automation.

## Project Context
This repository manages Tailscale Access Control Lists (ACLs) through GitOps methodology, automatically syncing ACL policies to the Tailscale tailnet via GitHub Actions.

## Core Principles
- Always validate ACL syntax before making changes
- Follow GitOps principles - all changes through pull requests
- Maintain security-first approach to access control policies
- Use descriptive commit messages for ACL changes
- Test ACL changes in non-production environments first

## File Types and Standards
- `policy.hujson`: Main Tailscale ACL policy file (Human JSON format)
- `.github/workflows/tailscale.yml`: GitHub Actions workflow for ACL sync
- Documentation files: Use clear, concise Markdown
- Configuration files: Follow YAML best practices

## Tailscale ACL Guidelines
- Use groups for user management instead of individual email addresses
- Implement least privilege access principles  
- Tag devices appropriately for granular control
- Include comments explaining complex access rules
- Test ACL policies before production deployment

## Code Style
- HuJSON: Use human-readable formatting with comments
- YAML: 2-space indentation, no tabs
- Markdown: Use consistent heading structure
- JSON: Pretty-print with 2-space indentation

## Security Considerations
- Never commit API keys or secrets to the repository
- Use GitHub secrets for sensitive Tailscale credentials
- Regularly audit and review access control policies
- Document security assumptions and threat model

## Development Workflow
1. Create feature branch for ACL changes
2. Make changes to policy.hujson
3. Test changes locally if possible
4. Create pull request with detailed description
5. Review and approve before merging to main
6. Monitor GitHub Actions deployment

## Testing
- Use Tailscale's built-in ACL test functionality
- Include test cases in the policy file
- Verify connectivity after ACL deployments
- Test edge cases and error scenarios

## Error Handling
- Check GitHub Actions logs for deployment issues
- Validate HuJSON syntax before committing
- Handle API rate limits gracefully
- Provide clear error messages in documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/enuno)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/enuno)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
