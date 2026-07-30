## rails-instructions

> Rails Background Jobs Guide (Solid Queue)


# Rails Background Jobs Guide (Solid Queue)

## Core Philosophy

- Use database-backed queuing for simplicity and reliability
- Avoid external dependencies like Redis when possible
- Design jobs to be idempotent and retryable
- Keep background processing simple and focused
- Leverage Rails' built-in Active Job framework

## Solid Queue Principles

- Database-backed job storage using FOR UPDATE SKIP LOCKED
- No external dependencies required for basic operation
- Built-in support for priorities, concurrency limits, and recurring jobs
- Seamless integration with Active Job interface
- Production-ready performance with PostgreSQL or MySQL

## Configuration Strategy

- Use separate database for queue in high-volume production environments
- Configure workers based on job priorities and server resources
- Set appropriate polling intervals for job urgency
- Design thread counts based on database connection limits
- Monitor queue depth and processing times

## Queue Organization

- Design queues based on priority and processing requirements
- Use semantic queue names that indicate purpose
- Configure dedicated workers for high-priority queues
- Set appropriate concurrency limits per queue type
- Use wildcards sparingly - prefer explicit queue names

## Job Design Principles

- Make all jobs idempotent - safe to run multiple times
- Keep jobs small and focused on single tasks
- Pass simple arguments - prefer IDs over objects
- Design for eventual consistency
- Handle failures gracefully with proper error reporting

## Error Handling Strategy

- Configure retry behavior based on job importance
- Use exponential backoff for transient failures
- Discard jobs that cannot succeed after retries
- Log errors with sufficient context for debugging
- Monitor failed job patterns for system issues

## Concurrency Controls

- Use `limits_concurrency` to prevent resource exhaustion
- Define concurrency keys based on resource constraints
- Set appropriate duration for concurrency locks
- Group related jobs with concurrency controls
- Monitor for concurrency bottlenecks

## Recurring Jobs

- Define recurring jobs in configuration, not code
- Use cron-like syntax for scheduling
- Keep recurring job logic simple and fast
- Monitor recurring job execution patterns
- Handle overlapping executions appropriately

## Performance Optimization

- Use exact queue names for better polling performance
- Configure batch sizes based on job characteristics
- Set polling intervals according to latency requirements
- Monitor database load from job queries
- Index job tables appropriately

## Database Configuration

- Use separate database for Solid Queue in production
- Configure appropriate connection pool sizes
- Set transaction isolation levels correctly
- Monitor database performance metrics
- Plan for job table maintenance

## Worker Management

- Use Puma plugin for integrated process management
- Configure supervisor for automatic restarts
- Set appropriate number of worker processes
- Monitor worker health and performance
- Handle graceful shutdowns properly

## Job Arguments

- Pass primitive types and simple data structures
- Avoid passing full ActiveRecord objects
- Use GlobalID for model references
- Keep argument size reasonable
- Version job arguments for backwards compatibility

## Testing Strategy

- Test job logic separately from queueing
- Use perform_now in tests for synchronous execution
- Test retry and failure scenarios
- Verify idempotency with multiple executions
- Mock external dependencies appropriately

## Monitoring Best Practices

- Track job queue depth and latency
- Monitor job success/failure rates
- Alert on queue backlogs
- Track job execution times
- Monitor worker resource usage

## Deployment Considerations

- Use Kamal for container-based deployments
- Configure health checks for workers
- Implement zero-downtime deployments
- Handle long-running jobs during deploys
- Monitor deployment impact on queues

## Migration Strategy

- Plan migration from other queue backends carefully
- Run both systems in parallel during transition
- Migrate jobs gradually by type
- Monitor for issues during migration
- Have rollback plan ready

## Debugging Techniques

- Use Rails console to inspect job state
- Query job tables directly when needed
- Add structured logging to jobs
- Use APM tools for job performance
- Track job execution paths

## Lifecycle Hooks

- Use Solid Queue callbacks for monitoring
- Track job state transitions
- Implement custom instrumentation
- Monitor for stuck jobs
- Alert on abnormal patterns

## Best Practices

- Keep jobs simple and focused
- Design for failure and retry
- Monitor everything important
- Document job behavior clearly
- Test edge cases thoroughly

## Common Patterns

- Email delivery jobs with rate limiting
- Report generation with progress tracking
- Data import/export with chunking
- Webhook processing with retries
- Scheduled maintenance tasks

## Anti-Patterns to Avoid

- Long-running jobs without heartbeats
- Jobs that modify shared state unsafely
- Recursive job spawning without limits
- Jobs with external dependencies in transactions
- Overly complex job orchestration

Remember: Solid Queue brings reliability and simplicity to background processing. Use it to keep your application responsive while handling work asynchronously.

---
> Source: [levifig/rails-instructions](https://github.com/levifig/rails-instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
