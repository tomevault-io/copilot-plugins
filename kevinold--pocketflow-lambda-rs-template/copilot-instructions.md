## pocketflow-lambda-rs-template

> This is a Rust codebase for building AI agents deployed as AWS Lambda functions.

# PocketFlow Lambda Rust Template - AI Agent Instructions

This is a Rust codebase for building AI agents deployed as AWS Lambda functions.

## References

Always reference `.cursor/rules/rust.mdc` for Rust codebase best practices, conventions, and patterns.

Reference `.cursor/rules/pocketflow/*` files for PocketFlow framework best practices, conventions, and patterns.

## Overview

This template provides a production-ready structure for building AI agent workflows using:
- **PocketFlow**: Minimalist LLM workflow framework
- **AWS Lambda**: Serverless compute with custom Rust runtime
- **AWS Secrets Manager**: Secure secrets management
- **AWS SAM**: Infrastructure as Code

## Project Structure

```
your-project/
├── src/
│   ├── main.rs           # Lambda entry point with secrets management
│   ├── flow.rs           # Flow definitions (workflow orchestration)
│   ├── nodes.rs          # Node implementations (processing units)
│   └── utils/            # Utility functions (external integrations)
│       └── mod.rs
├── .cursor/
│   └── rules/
│       ├── rust.mdc      # Rust best practices
│       └── pocketflow/   # PocketFlow documentation
├── Cargo.toml            # Rust dependencies
├── Justfile              # Build automation
├── template.yaml         # AWS SAM CloudFormation template
├── .env.example          # Local development environment variables
├── .env.local.dev        # Dev secrets (deployed to AWS Secrets Manager)
└── .env.local.prod       # Prod secrets (deployed to AWS Secrets Manager)
```

## Features

- **Dual Mode Operation**: Works as both CLI tool and Lambda function
- **Secrets Management**: Automatic AWS Secrets Manager integration
- **Environment Isolation**: Separate dev/prod configurations
- **Type-Safe Workflows**: Strongly-typed nodes and state machines
- **Error Handling**: Built-in error routing and recovery
- **Observability**: Structured logging with tracing
- **Testing**: Unit test examples included

## Getting Started

### 1. Customize the Template

Search for "REPLACE:" comments throughout the codebase and update:
- Project name in `Cargo.toml`, `Justfile`, `template.yaml`
- AWS region and configuration
- Your specific domain logic

Search for "CUSTOMIZE:" comments for areas to extend:
- Add your secrets to `AppSecrets` struct
- Implement your nodes in `nodes.rs`
- Create your flows in `flow.rs`
- Add utilities in `utils/`

### 2. PocketFlow Patterns

Follow these core patterns from PocketFlow:

**Node Design**:
- Read from Context in `execute()`
- Write to Context in `post_process()`
- Return appropriate State for flow control

**Flow Design**:
- Use functional composition
- Keep nodes focused (single responsibility)
- Handle errors with dedicated error nodes

**Context Usage**:
- Use Context as shared store for data passing
- Avoid side effects in nodes
- Keep Context updates in post_process()

### 3. Development Workflow

1. **Design First** (`.cursor/rules/pocketflow/guide_for_pocketflow.mdc`):
   - Define requirements
   - Design flows (high-level)
   - Identify utilities needed
   - Plan node interactions

2. **Implement**:
   - Create utility functions
   - Implement nodes
   - Build flows
   - Add tests

3. **Deploy**:
   ```bash
   just deploy-dev    # Deploy to dev
   just deploy-prod   # Deploy to prod
   ```

## Important Notes

- **Always read PocketFlow documentation** before implementing
- **Keep nodes pure**: Avoid side effects in execute()
- **Use Context for state**: Don't use mutable class fields
- **Test utilities independently**: Each should have its own tests
- **Follow functional patterns**: Immutable data, pure functions

## Common Patterns

### Adding a New Node

1. Define in `nodes.rs`:
```rust
pub struct MyNode {
    // Configuration only, no mutable state
}

#[async_trait]
impl Node for MyNode {
    type State = MyState;

    async fn execute(&self, context: &Context) -> Result<Value> {
        // Read from context
        // Do work
        // Return result
    }

    async fn post_process(&self, context: &mut Context, result: &Result<Value>) -> Result<ProcessResult<MyState>> {
        // Write to context
        // Return next state
    }
}
```

2. Connect in `flow.rs`:
```rust
let flow = build_flow!(
    start: ("my_node", my_node),
    nodes: [...],
    edges: [
        ("my_node", "next_node", MyState::Success),
    ]
);
```

### Adding a Utility

1. Create `src/utils/my_utility.rs`:
```rust
pub async fn my_utility(param: String) -> Result<String> {
    // Implement external integration
}

#[cfg(test)]
mod tests {
    #[tokio::test]
    async fn test_my_utility() {
        // Test your utility
    }
}
```

2. Export in `src/utils/mod.rs`:
```rust
pub mod my_utility;
pub use my_utility::my_utility;
```

## Deployment

The template includes complete AWS deployment setup:

- **Secrets**: Stored in AWS Secrets Manager
- **Lambda**: Custom Rust runtime (provided.al2023)
- **Logs**: CloudWatch with retention policies
- **Monitoring**: CloudWatch Insights queries included

See README.md for detailed deployment instructions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinold) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
