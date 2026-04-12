## rproj-website

> Implement comprehensive security measures to protect against common web vulnerabilities and ensure secure data handling.

# Security Best Practices for Astro Applications

## Rule
Implement comprehensive security measures to protect against common web vulnerabilities and ensure secure data handling.

## Guidelines

### Environment Variables and Secrets Management
- Store all sensitive information in environment variables
- Never commit actual secrets to version control
- Use proper environment variable validation
- Follow the principle of least privilege for API tokens

### Input Validation and Data Security
- Validate all user input and external data using Zod schemas
- Sanitize content before rendering with `set:html` or `dangerouslySetInnerHTML`
- Implement proper HTTPS for all API communications
- Use Content Security Policy (CSP) headers

### Dependency and Build Security
- Keep dependencies up-to-date to patch vulnerabilities
- Avoid dynamic code execution with untrusted input
- Secure the build process and deployment pipeline
- Regular security audits of dependencies

## Examples

### Secure Environment Variables Setup
```typescript
// lib/config/env.ts
import { z } from 'zod';

const EnvSchema = z.object({
  // Public variables (prefixed with PUBLIC_)
  PUBLIC_SITE_URL: z.string().url(),
  PUBLIC_ANALYTICS_ID: z.string().optional(),
  
  // Private variables (server-side only)
  NOTION_API_KEY: z.string().min(1, 'Notion API key is required'),
  RESEND_API_KEY: z.string().min(1, 'Resend API key is required'),
  DATABASE_URL: z.string().url('Invalid database URL'),
  JWT_SECRET: z.string().min(32, 'JWT secret must be at least 32 characters'),
  
  // Optional but validated if present
  SENTRY_DSN: z.string().url().optional(),
  WEBHOOK_SECRET: z.string().min(16).optional(),
});

// Validate environment variables at startup
export const env = EnvSchema.parse(process.env);

// Type-safe environment variables
export type Env = z.infer<typeof EnvSchema>;

// Helper to check if we're in production
export const isProduction = process.env.NODE_ENV === 'production';

// Secure headers configuration
export const securityHeaders = {
  'X-Frame-Options': 'DENY',
  'X-Content-Type-Options': 'nosniff',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()',
} as const;
```

### Input Validation and Sanitization
```typescript
// lib/security/validation.ts
import { z } from 'zod';
import DOMPurify from 'isomorphic-dompurify';

// Secure form schemas with additional validation
export const SecureContactFormSchema = z.object({
  name: z.string()
    .min(1, 'Name is required')
    .max(100, 'Name too long')
    .regex(/^[a-zA-Z\s\-']+$/, 'Name contains invalid characters'),
  
  email: z.string()
    .email('Invalid email format')
    .max(254, 'Email too long') // RFC 5321 limit
    .toLowerCase(),
  
  subject: z.string()
    .min(1, 'Subject is required')
    .max(200, 'Subject too long')
    .transform(val => val.trim()),
  
  message: z.string()
    .min(10, 'Message too short')
    .max(2000, 'Message too long')
    .transform(val => val.trim()),
  
  // Honeypot field for bot detection
  website: z.string().max(0, 'Bot detected').optional(),
  
  // Rate limiting token
  token: z.string().min(1, 'Security token required'),
});

// Content sanitization
export function sanitizeHtml(content: string): string {
  return DOMPurify.sanitize(content, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: [],
    KEEP_CONTENT: true,
  });
}

// URL validation for redirects
export function validateRedirectUrl(url: string, allowedDomains: string[]): boolean {
  try {
    const parsedUrl = new URL(url);
    return allowedDomains.includes(parsedUrl.hostname);
  } catch {
    return false;
  }
}

// SQL injection prevention for dynamic queries
export function escapeSQL(value: string): string {
  return value.replace(/'/g, "''").replace(/;/g, '');
}
```

### Secure API Routes
```typescript
// src/pages/api/secure-contact.ts
import type { APIRoute } from 'astro';
import { SecureContactFormSchema } from '../../lib/security/validation';
import { env, securityHeaders } from '../../lib/config/env';
import { rateLimit } from '../../lib/security/rate-limit';

export const POST: APIRoute = async ({ request, clientAddress }) => {
  // Apply security headers
  const headers = new Headers({
    'Content-Type': 'application/json',
    ...securityHeaders,
  });

  try {
    // Rate limiting
    const rateLimitResult = await rateLimit(clientAddress, 'contact-form', {
      windowMs: 15 * 60 * 1000, // 15 minutes
      maxRequests: 5, // 5 requests per window
    });

    if (!rateLimitResult.success) {
      return new Response(JSON.stringify({
        success: false,
        message: 'Too many requests. Please try again later.',
        retryAfter: rateLimitResult.retryAfter,
      }), {
        status: 429,
        headers,
      });
    }

    // Validate request body
    const body = await request.json();
    const validation = SecureContactFormSchema.safeParse(body);

    if (!validation.success) {
      return new Response(JSON.stringify({
        success: false,
        message: 'Invalid form data',
        errors: validation.error.errors.map(err => ({
          field: err.path.join('.'),
          message: err.message,
        })),
      }), {
        status: 400,
        headers,
      });
    }

    const { name, email, subject, message, token } = validation.data;

    // Verify security token (CSRF protection)
    const isValidToken = await verifySecurityToken(token, clientAddress);
    if (!isValidToken) {
      return new Response(JSON.stringify({
        success: false,
        message: 'Invalid security token',
      }), {
        status: 403,
        headers,
      });
    }

    // Process the secure form submission
    await processContactForm({
      name,
      email,
      subject,
      message,
      clientAddress,
      timestamp: new Date().toISOString(),
    });

    return new Response(JSON.stringify({
      success: true,
      message: 'Message sent successfully',
    }), {
      status: 200,
      headers,
    });

  } catch (error) {
    console.error('Contact form error:', error);

    // Don't expose internal errors
    return new Response(JSON.stringify({
      success: false,
      message: 'Internal server error',
    }), {
      status: 500,
      headers,
    });
  }
};

// Security token verification
async function verifySecurityToken(token: string, clientAddress: string): Promise<boolean> {
  try {
    // Implement your token verification logic
    // This could be JWT, HMAC, or session-based
    const payload = await verifyJWT(token, env.JWT_SECRET);
    return payload.clientAddress === clientAddress;
  } catch {
    return false;
  }
}
```

### Rate Limiting Implementation
```typescript
// lib/security/rate-limit.ts
interface RateLimitConfig {
  windowMs: number;
  maxRequests: number;
}

interface RateLimitResult {
  success: boolean;
  retryAfter?: number;
}

// In-memory store (use Redis in production)
const requestCounts = new Map<string, { count: number; resetTime: number }>();

export async function rateLimit(
  identifier: string,
  endpoint: string,
  config: RateLimitConfig
): Promise<RateLimitResult> {
  const key = `${identifier}:${endpoint}`;
  const now = Date.now();
  const windowStart = now - config.windowMs;

  // Clean up old entries
  for (const [k, v] of requestCounts.entries()) {
    if (v.resetTime < now) {
      requestCounts.delete(k);
    }
  }

  const current = requestCounts.get(key);

  if (!current || current.resetTime < now) {
    // First request in window or window expired
    requestCounts.set(key, {
      count: 1,
      resetTime: now + config.windowMs,
    });
    return { success: true };
  }

  if (current.count >= config.maxRequests) {
    // Rate limit exceeded
    return {
      success: false,
      retryAfter: Math.ceil((current.resetTime - now) / 1000),
    };
  }

  // Increment count
  current.count++;
  return { success: true };
}
```

### Content Security Policy
```astro
---
// layouts/SecureLayout.astro
import { env } from '../lib/config/env';

const cspDirectives = [
  "default-src 'self'",
  "script-src 'self' 'unsafe-inline' https://analytics.google.com",
  "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
  "font-src 'self' https://fonts.gstatic.com",
  "img-src 'self' data: https:",
  "connect-src 'self' https://api.notion.com",
  "frame-ancestors 'none'",
  "base-uri 'self'",
  "form-action 'self'",
].join('; ');

const securityHeaders = {
  'Content-Security-Policy': cspDirectives,
  'X-Frame-Options': 'DENY',
  'X-Content-Type-Options': 'nosniff',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()',
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
};

// Set headers in Astro response
if (Astro.response) {
  Object.entries(securityHeaders).forEach(([key, value]) => {
    Astro.response.headers.set(key, value);
  });
}
---

<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  
  <!-- Security meta tags -->
  <meta http-equiv="X-Content-Type-Options" content="nosniff">
  <meta http-equiv="X-Frame-Options" content="DENY">
  
  <slot name="head" />
</head>
<body>
  <slot />
</body>
</html>
```

### Secure Authentication Utilities
```typescript
// lib/security/auth.ts
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';
import { env } from '../config/env';

export interface UserPayload {
  id: string;
  email: string;
  role: string;
}

// JWT token generation
export function generateToken(payload: UserPayload): string {
  return jwt.sign(payload, env.JWT_SECRET, {
    expiresIn: '24h',
    issuer: env.PUBLIC_SITE_URL,
    audience: env.PUBLIC_SITE_URL,
  });
}

// JWT token verification
export function verifyToken(token: string): UserPayload {
  try {
    return jwt.verify(token, env.JWT_SECRET, {
      issuer: env.PUBLIC_SITE_URL,
      audience: env.PUBLIC_SITE_URL,
    }) as UserPayload;
  } catch (error) {
    throw new Error('Invalid token');
  }
}

// Password hashing
export async function hashPassword(password: string): Promise<string> {
  const saltRounds = 12;
  return bcrypt.hash(password, saltRounds);
}

// Password verification
export async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

// Secure random token generation
export function generateSecureToken(length: number = 32): string {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  let result = '';
  const randomArray = new Uint8Array(length);
  crypto.getRandomValues(randomArray);
  
  for (let i = 0; i < length; i++) {
    result += chars.charAt(randomArray[i] % chars.length);
  }
  
  return result;
}
```

### Secure Component with XSS Prevention
```astro
---
// components/SecureContent.astro
import { sanitizeHtml } from '../lib/security/validation';

interface Props {
  content: string;
  allowHtml?: boolean;
  maxLength?: number;
}

const { content, allowHtml = false, maxLength = 1000 } = Astro.props;

// Truncate content if too long
const truncatedContent = content.length > maxLength 
  ? content.substring(0, maxLength) + '...' 
  : content;

// Sanitize if HTML is allowed
const safeContent = allowHtml 
  ? sanitizeHtml(truncatedContent)
  : truncatedContent;
---

{allowHtml ? (
  <div set:html={safeContent} />
) : (
  <div>{safeContent}</div>
)}
```

### Security Audit Checklist
```typescript
// scripts/security-audit.ts
import { execSync } from 'child_process';

interface SecurityCheck {
  name: string;
  check: () => Promise<boolean>;
  severity: 'low' | 'medium' | 'high' | 'critical';
}

const securityChecks: SecurityCheck[] = [
  {
    name: 'Dependencies vulnerability scan',
    check: async () => {
      try {
        execSync('npm audit --audit-level=moderate', { stdio: 'pipe' });
        return true;
      } catch {
        return false;
      }
    },
    severity: 'high',
  },
  {
    name: 'Environment variables validation',
    check: async () => {
      const requiredVars = ['NOTION_API_KEY', 'RESEND_API_KEY', 'JWT_SECRET'];
      return requiredVars.every(varName => process.env[varName]);
    },
    severity: 'critical',
  },
  {
    name: 'HTTPS enforcement',
    check: async () => {
      return process.env.NODE_ENV === 'production' 
        ? process.env.PUBLIC_SITE_URL?.startsWith('https://') ?? false
        : true;
    },
    severity: 'high',
  },
];

export async function runSecurityAudit(): Promise<void> {
  console.log('Running security audit...\n');
  
  const results = await Promise.all(
    securityChecks.map(async check => ({
      ...check,
      passed: await check.check(),
    }))
  );

  const failed = results.filter(result => !result.passed);
  
  if (failed.length === 0) {
    console.log('✅ All security checks passed!');
  } else {
    console.log('❌ Security issues found:');
    failed.forEach(check => {
      console.log(`  - ${check.name} (${check.severity})`);
    });
    
    const criticalIssues = failed.filter(check => check.severity === 'critical');
    if (criticalIssues.length > 0) {
      process.exit(1);
    }
  }
}
```

### Bad Security Practices
```typescript
// ❌ Bad - Hardcoded secrets
const API_KEY = 'sk-1234567890abcdef';

// ❌ Bad - No input validation
export const POST: APIRoute = async ({ request }) => {
  const data = await request.json();
  // Direct database insertion without validation
  await db.insert(data);
};

// ❌ Bad - Unsafe HTML rendering
<div set:html={userContent} />

// ❌ Bad - No rate limiting
export const POST: APIRoute = async ({ request }) => {
  // Process unlimited requests
};
```

### Good Security Practices
```typescript
// ✅ Good - Environment variables
const API_KEY = env.NOTION_API_KEY;

// ✅ Good - Input validation
export const POST: APIRoute = async ({ request }) => {
  const validation = schema.safeParse(await request.json());
  if (!validation.success) {
    return new Response('Invalid input', { status: 400 });
  }
  await db.insert(validation.data);
};

// ✅ Good - Sanitized HTML
<div set:html={sanitizeHtml(userContent)} />

// ✅ Good - Rate limiting
export const POST: APIRoute = async ({ request, clientAddress }) => {
  const rateLimitResult = await rateLimit(clientAddress, 'api', config);
  if (!rateLimitResult.success) {
    return new Response('Rate limited', { status: 429 });
  }
};
```

## Security Monitoring
- Implement logging for security events
- Monitor for unusual patterns in API usage
- Set up alerts for failed authentication attempts
- Regular security audits and dependency updates
- Use tools like Snyk or npm audit for vulnerability scanning

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Resonant-Projects)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Resonant-Projects)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
