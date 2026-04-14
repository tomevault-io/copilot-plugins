## nestjs-infra-template

> This document provides strict rules for AI agents when working with configuration in this NestJS project.

# Configuration Management Rules for AI Agents

This document provides strict rules for AI agents when working with configuration in this NestJS project.

## 🎯 Core Principle

**Every domain has its own isolated configuration namespace. No domain can access another domain's configuration.**

---

## 📁 Configuration File Locations

### Rule 1: Infrastructure Config → `src/config/`

Place in `src/config/` if it's infrastructure-level:
- Database connection
- Redis connection  
- Logger settings
- JWT secrets
- API keys for infrastructure services

```
src/config/
├── env.ts                  # Environment accessor
├── database.config.ts      # Database config
├── redis.config.ts         # Redis config
├── logger.config.ts        # Logger config
└── jwt.config.ts           # JWT config
```

### Rule 2: Domain Config → `src/features/{domain}/config/`

Place in domain folder if it's domain-specific:
- Domain business rules
- Domain-specific API keys
- Domain feature flags
- Domain thresholds/limits

```
src/features/users/config/
└── users.config.ts

src/features/notifications/config/
└── notifications.config.ts

src/features/payments/config/
└── payments.config.ts
```

---

## 🏗️ Configuration File Structure

### Rule 3: All Configs Use `registerAs`

```typescript
import { registerAs } from '@nestjs/config';

export interface DomainConfig {
  // Type your configuration
  property1: string;
  property2: number;
}

export default registerAs(
  'domain-name',  // Unique namespace
  (): DomainConfig => ({
    property1: process.env.DOMAIN_PROPERTY1 || 'default',
    property2: parseInt(process.env.DOMAIN_PROPERTY2 || '0', 10),
  }),
);
```

### Rule 4: Configuration Template

```typescript
// src/features/users/config/users.config.ts
import { registerAs } from '@nestjs/config';

/**
 * Users domain configuration
 * Namespace: 'users'
 */
export interface UsersConfig {
  hashIdSalt: string;
  hashIdMinLength: number;
  maxLoginAttempts: number;
  lockoutDuration: number;
  sessionTimeout: number;
}

export default registerAs(
  'users',  // ✅ Unique namespace
  (): UsersConfig => ({
    hashIdSalt: process.env.USERS_HASHID_SALT || '',
    hashIdMinLength: parseInt(process.env.USERS_HASHID_MIN_LENGTH || '10', 10),
    maxLoginAttempts: parseInt(process.env.USERS_MAX_LOGIN_ATTEMPTS || '5', 10),
    lockoutDuration: parseInt(process.env.USERS_LOCKOUT_DURATION || '900', 10),
    sessionTimeout: parseInt(process.env.USERS_SESSION_TIMEOUT || '3600', 10),
  }),
);
```

**Required components:**
1. JSDoc comment with namespace
2. TypeScript interface
3. Export interface (for type checking)
4. Use `registerAs` with unique namespace
5. Environment variable names with domain prefix
6. Default values for non-critical configs
7. Type casting for numbers/booleans

---

## 🔑 Environment Variable Naming

### Rule 5: Use Domain Prefixes

Format: `{DOMAIN}_{PROPERTY_NAME}`

```bash
# ✅ CORRECT: Infrastructure (no prefix)
DB_HOST=localhost
DB_PORT=5432
REDIS_HOST=localhost
LOG_LEVEL=debug

# ✅ CORRECT: Domain-specific (with prefix)
USERS_HASHID_SALT=unique-salt
USERS_MAX_LOGIN_ATTEMPTS=5

NOTIFICATIONS_SMTP_HOST=smtp.example.com
NOTIFICATIONS_SMTP_PORT=587

PAYMENTS_STRIPE_API_KEY=sk_test_...
PAYMENTS_MAX_REFUND_DAYS=30
```

### Rule 6: Naming Convention

```bash
# Format:
{DOMAIN}_{FEATURE}_{PROPERTY}

# Examples:
USERS_PASSWORD_MIN_LENGTH=8
USERS_SESSION_TIMEOUT=3600
USERS_EMAIL_VERIFICATION_REQUIRED=true

PAYMENTS_STRIPE_API_KEY=sk_test_...
PAYMENTS_STRIPE_WEBHOOK_SECRET=whsec_...
PAYMENTS_REFUND_ENABLED=true

NOTIFICATIONS_EMAIL_ENABLED=true
NOTIFICATIONS_SMS_ENABLED=false
NOTIFICATIONS_RETRY_ATTEMPTS=3
```

---

## 📦 Registering Configuration

### Rule 7: Register in Module

```typescript
// src/features/users/users.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import usersConfig from './config/users.config';

@Module({
  imports: [
    ConfigModule.forFeature(usersConfig),  // ✅ Register domain config
  ],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

### Rule 8: Infrastructure Config in app.module

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import databaseConfig from './config/database.config';
import redisConfig from './config/redis.config';
import loggerConfig from './config/logger.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,  // Makes ConfigService available globally
      load: [databaseConfig, redisConfig, loggerConfig],  // Infrastructure configs
    }),
    // ... feature modules
  ],
})
export class AppModule {}
```

---

## 🎯 Accessing Configuration

### Rule 9: Type-Safe Access

```typescript
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { UsersConfig } from './config/users.config';

@Injectable()
export class UsersService {
  private readonly config: UsersConfig;

  constructor(private readonly configService: ConfigService) {
    // ✅ CORRECT: Type-safe access to own domain config
    this.config = this.configService.get<UsersConfig>('users')!;
  }

  someMethod() {
    const maxAttempts = this.config.maxLoginAttempts;
    const lockoutDuration = this.config.lockoutDuration;
  }
}
```

### Rule 10: Configuration Isolation

```typescript
// ✅ CORRECT: Access only your domain's config
@Injectable()
export class UsersService {
  private readonly config: UsersConfig;

  constructor(private configService: ConfigService) {
    this.config = this.configService.get<UsersConfig>('users')!;
  }
}

// ❌ WRONG: Don't access other domain configs
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {
    // ❌ Users service accessing payments config
    const stripeKey = this.configService.get<PaymentsConfig>('payments')!.stripeApiKey;
    
    // ❌ Users service accessing notifications config
    const smtpHost = this.configService.get<NotificationsConfig>('notifications')!.smtpHost;
  }
}

// ✅ CORRECT: Infrastructure configs are OK
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {
    // ✅ OK: Accessing infrastructure config
    const logLevel = this.configService.get<LoggerConfig>('logger')!.level;
  }
}
```

---

## ✅ Configuration Validation

### Rule 11: Validate at Startup

```typescript
import { registerAs } from '@nestjs/config';
import { IsString, IsInt, Min, Max, IsUrl, validateSync } from 'class-validator';
import { plainToClass } from 'class-transformer';

export class PaymentsConfig {
  @IsString()
  stripeApiKey: string;

  @IsString()
  stripeWebhookSecret: string;

  @IsString()
  currency: string;

  @IsInt()
  @Min(1)
  @Max(365)
  maxRefundDays: number;

  @IsUrl()
  webhookUrl: string;
}

export default registerAs('payments', (): PaymentsConfig => {
  const config = plainToClass(PaymentsConfig, {
    stripeApiKey: process.env.PAYMENTS_STRIPE_API_KEY || '',
    stripeWebhookSecret: process.env.PAYMENTS_STRIPE_WEBHOOK_SECRET || '',
    currency: process.env.PAYMENTS_CURRENCY || 'USD',
    maxRefundDays: parseInt(process.env.PAYMENTS_MAX_REFUND_DAYS || '30', 10),
    webhookUrl: process.env.PAYMENTS_WEBHOOK_URL || '',
  });

  // ✅ Validate configuration at startup
  const errors = validateSync(config, {
    skipMissingProperties: false,
  });

  if (errors.length > 0) {
    throw new Error(
      `Payments configuration validation failed:\n${errors.map((e) => Object.values(e.constraints || {})).join('\n')}`,
    );
  }

  return config;
});
```

---

## 🧪 Testing Configuration

### Rule 12: Test Configuration Loading

```typescript
// src/features/users/config/users.config.spec.ts
import { Test } from '@nestjs/testing';
import { ConfigModule, ConfigService } from '@nestjs/config';
import usersConfig, { UsersConfig } from './users.config';

describe('UsersConfig', () => {
  let configService: ConfigService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [
        ConfigModule.forFeature(usersConfig),
      ],
    }).compile();

    configService = module.get<ConfigService>(ConfigService);
  });

  it('should load users configuration', () => {
    const config = configService.get<UsersConfig>('users');
    expect(config).toBeDefined();
    expect(config?.maxLoginAttempts).toBeDefined();
  });

  it('should not access other domain configs', () => {
    // ✅ This should be undefined
    const notificationsConfig = configService.get('notifications');
    expect(notificationsConfig).toBeUndefined();
  });

  it('should parse numbers correctly', () => {
    const config = configService.get<UsersConfig>('users');
    expect(typeof config?.maxLoginAttempts).toBe('number');
    expect(typeof config?.lockoutDuration).toBe('number');
  });
});
```

---

## 🚫 Common Mistakes

### ❌ DON'T: Access other domain configs

```typescript
// ❌ WRONG
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {
    const paymentKey = this.configService.get<PaymentsConfig>('payments')!.stripeApiKey;
  }
}
```

### ❌ DON'T: Use process.env directly in services

```typescript
// ❌ WRONG
@Injectable()
export class UsersService {
  someMethod() {
    const salt = process.env.USERS_HASHID_SALT;  // ❌ Use config instead
  }
}

// ✅ CORRECT
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {
    this.config = this.configService.get<UsersConfig>('users')!;
  }

  someMethod() {
    const salt = this.config.hashIdSalt;  // ✅ From config
  }
}
```

### ❌ DON'T: Skip type definitions

```typescript
// ❌ WRONG: No type definition
export default registerAs('users', () => ({
  maxAttempts: parseInt(process.env.USERS_MAX_ATTEMPTS || '5'),
}));

// ✅ CORRECT: With type definition
export interface UsersConfig {
  maxAttempts: number;
}

export default registerAs('users', (): UsersConfig => ({
  maxAttempts: parseInt(process.env.USERS_MAX_ATTEMPTS || '5', 10),
}));
```

### ❌ DON'T: Forget domain prefixes

```bash
# ❌ WRONG: No prefix
MAX_LOGIN_ATTEMPTS=5
SMTP_HOST=smtp.example.com

# ✅ CORRECT: With prefix
USERS_MAX_LOGIN_ATTEMPTS=5
NOTIFICATIONS_SMTP_HOST=smtp.example.com
```

---

## 📋 Configuration Checklist

When creating new domain configuration:

- [ ] Created `config/{domain}.config.ts` in domain folder
- [ ] Defined TypeScript interface
- [ ] Used `registerAs` with unique namespace
- [ ] Environment variables have domain prefix
- [ ] Provided sensible defaults (where appropriate)
- [ ] Registered in domain module with `ConfigModule.forFeature()`
- [ ] Type-safe access in services
- [ ] Added validation (if critical config)
- [ ] Created unit tests
- [ ] Updated `.env.example`
- [ ] Documented in README (if needed)

---

## 📚 Examples by Domain

### Users Domain

```typescript
// src/features/users/config/users.config.ts
export interface UsersConfig {
  hashIdSalt: string;
  hashIdMinLength: number;
  maxLoginAttempts: number;
  lockoutDuration: number;
  passwordMinLength: number;
  passwordRequireSpecialChar: boolean;
  emailVerificationRequired: boolean;
  sessionTimeout: number;
}

export default registerAs('users', (): UsersConfig => ({
  hashIdSalt: process.env.USERS_HASHID_SALT || '',
  hashIdMinLength: parseInt(process.env.USERS_HASHID_MIN_LENGTH || '10', 10),
  maxLoginAttempts: parseInt(process.env.USERS_MAX_LOGIN_ATTEMPTS || '5', 10),
  lockoutDuration: parseInt(process.env.USERS_LOCKOUT_DURATION || '900', 10),
  passwordMinLength: parseInt(process.env.USERS_PASSWORD_MIN_LENGTH || '8', 10),
  passwordRequireSpecialChar: process.env.USERS_PASSWORD_REQUIRE_SPECIAL === 'true',
  emailVerificationRequired: process.env.USERS_EMAIL_VERIFICATION_REQUIRED !== 'false',
  sessionTimeout: parseInt(process.env.USERS_SESSION_TIMEOUT || '3600', 10),
}));
```

### Notifications Domain

```typescript
// src/features/notifications/config/notifications.config.ts
export interface NotificationsConfig {
  emailEnabled: boolean;
  smsEnabled: boolean;
  smtpHost: string;
  smtpPort: number;
  smtpUser: string;
  smtpPassword: string;
  fromEmail: string;
  fromName: string;
  retryAttempts: number;
  retryDelay: number;
}

export default registerAs('notifications', (): NotificationsConfig => ({
  emailEnabled: process.env.NOTIFICATIONS_EMAIL_ENABLED === 'true',
  smsEnabled: process.env.NOTIFICATIONS_SMS_ENABLED === 'true',
  smtpHost: process.env.NOTIFICATIONS_SMTP_HOST || 'localhost',
  smtpPort: parseInt(process.env.NOTIFICATIONS_SMTP_PORT || '587', 10),
  smtpUser: process.env.NOTIFICATIONS_SMTP_USER || '',
  smtpPassword: process.env.NOTIFICATIONS_SMTP_PASSWORD || '',
  fromEmail: process.env.NOTIFICATIONS_FROM_EMAIL || 'noreply@example.com',
  fromName: process.env.NOTIFICATIONS_FROM_NAME || 'Application',
  retryAttempts: parseInt(process.env.NOTIFICATIONS_RETRY_ATTEMPTS || '3', 10),
  retryDelay: parseInt(process.env.NOTIFICATIONS_RETRY_DELAY || '5000', 10),
}));
```

---

## 🎓 Summary for AI Agents

**Configuration Golden Rules:**

1. **Domain isolation** - Each domain has its own config namespace
2. **Use `registerAs`** - Always namespace your configs
3. **Type everything** - Export interfaces for type safety
4. **Prefix env vars** - Use domain prefixes (e.g., `USERS_`, `NOTIFICATIONS_`)
5. **No cross-domain access** - Access only your domain's config
6. **Infrastructure configs are global** - OK to access from any domain
7. **Validate critical configs** - Use class-validator for important settings
8. **Test configuration** - Write tests for config loading
9. **Never use process.env directly** - Always use ConfigService
10. **Document in .env.example** - Keep environment template updated

**Remember:** Configuration isolation is a key part of domain-centric architecture!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sharifli4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
