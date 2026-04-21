## nest-typeorm-mysql-starter

> Guidelines for using the logging service in NestJS applications


# Logger Usage Guidelines

This project uses a Winston-based logging service that supports multiple log levels and file output. Follow these guidelines for consistent logging across the application.

## Log Levels

| Level   | Description                                                |
| ------- | ---------------------------------------------------------- |
| `debug` | Detailed information for debugging                         |
| `log`   | General operational information (info level)               |
| `warn`  | Warning messages for potentially harmful situations        |
| `error` | Error events that might still allow the application to run |

## Configuration

Logging behavior can be configured in `.env`:

```env
LOG_LEVEL=info              # Minimum log level (debug, info, warn, error)
LOG_TO_FILE=true            # Enable file logging
```

When `LOG_TO_FILE` is enabled, logs are written to:

- `logs/combined.log` - All logs (info and above)
- `logs/error.log` - Error logs only

## Best Practices

1. **Context** - Always set context in constructor using service/class name
2. **Messages** - Use clear, descriptive messages with relevant data in meta object
3. **Sensitive Data** - Never log passwords, tokens, or other sensitive information
4. **Error Details** - Include error message, stack trace, and relevant context
5. **Meta Data** - Use structured objects with relevant IDs and concise information

## Usage Example

```typescript
import { LogService } from 'src/core/logger/log-service';

@Injectable()
export class UserService {
  constructor(private readonly logger: LogService) {
    this.logger.setContext('UserService');
  }

  async createUser(userData: CreateUserDto) {
    this.logger.log('Creating new user', { email: userData.email });

    try {
      const user = await this.userRepository.save(userData);
      this.logger.log('User created successfully', { userId: user.id });
      return user;
    } catch (error) {
      this.logger.error('Failed to create user', {
        error: error.message,
        email: userData.email,
      });
      throw error;
    }
  }

  async processData(data: any) {
    this.logger.debug('Processing request', { requestId: data.id });
    this.logger.warn('Rate limit approaching', { current: 95, limit: 100 });
  }
}
```

## What NOT to Do

```typescript
// ❌ Don't log sensitive data
this.logger.log('User data', { password: user.password });

// ❌ Don't use vague messages without context
this.logger.log('Something happened');
this.logger.error('Error occurred');

// ❌ Don't forget to set context
this.logger.log('Operation completed');
```

## Integration with Interceptors

The project includes a built-in logger interceptor that automatically logs HTTP requests and responses. It's configured in the core module and doesn't require additional setup.

**Note**: The logger is a transient service, meaning each injection gets its own instance for proper context isolation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VitaliyDrapkin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
