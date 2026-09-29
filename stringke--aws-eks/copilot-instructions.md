## aws-eks

> - `terraform init` - Initialize the working directory

# Terraform Project Guidelines

## Commands
- `terraform init` - Initialize the working directory
- `terraform plan` - Show execution plan
- `terraform apply` - Apply changes
- `terraform validate` - Validate configurations
- `terraform fmt` - Format configurations
- `terraform fmt -check` - Check if files are properly formatted

## Linting
- Use `terraform fmt` before committing code
- Validate with `terraform validate` before applying changes

## Style Guidelines
1. **Naming**: Use snake_case for resources, variables, and outputs
2. **Variables**: Include description, type, and default (when appropriate)
3. **Modules**: Group related resources; use community modules when available
4. **Structure**: Organize by function (aws-vpc.tf, aws-eks.tf, addons-*.tf)
5. **Formatting**: 2-space indentation, align = signs where appropriate
6. **Documentation**: Add comments for complex configurations
7. **State Management**: Use remote state with S3 backend
8. **Tags**: Always apply appropriate tags to resources

## Dependencies
- Explicitly declare dependencies using depends_on
- Specify version constraints for providers and modules

---
> Source: [StringKe/aws-eks](https://github.com/StringKe/aws-eks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
