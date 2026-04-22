## cursor-rules

> Security patterns, vulnerabilities, and protection strategies


# Security Patterns & Protection

> **Instructions**: Document security requirements, patterns, and vulnerabilities specific to your project.

## Authentication & Authorization

### Authentication Patterns
- **Primary Auth**: [e.g., Supabase Auth, Auth0, custom JWT]
- **Multi-Factor**: Required for admin functions and sensitive operations
- **Session Management**: [e.g., 24-hour expiry, refresh tokens, secure cookies]
- **Password Policy**: [e.g., 12+ chars, complexity requirements, breach checking]

### Authorization Patterns
- **Role-Based Access Control (RBAC)**: [Define roles and permissions]
- **Resource-Level Permissions**: [Who can access what data]
- **API Endpoint Protection**: [Authentication required for all endpoints]
- **Admin Functions**: [Two-person approval for critical operations]

### Example Implementation
```typescript
// Authentication middleware
export function requireAuth(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'Authentication required' });
  
  // Verify JWT and extract user context
  const user = verifyToken(token);
  req.user = user;
  next();
}

// Authorization middleware
export function requireRole(role: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user?.roles?.includes(role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}
```

## Input Validation & Sanitization

### Validation Rules
- **All user inputs** must be validated on both client and server
- **SQL injection prevention**: Use parameterized queries only
- **XSS prevention**: Sanitize all user-generated content
- **File uploads**: Validate file types, scan for malware, limit size
- **API rate limiting**: Prevent abuse and DoS attacks

### Validation Patterns
```typescript
// Input validation with Zod
const userSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(12).regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/),
  role: z.enum(['user', 'admin', 'moderator'])
});

// Sanitization
function sanitizeHtml(input: string): string {
  return DOMPurify.sanitize(input, { 
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong'],
    ALLOWED_ATTR: []
  });
}
```

## Data Protection

### Encryption Requirements
- **Data at Rest**: [e.g., AES-256 encryption for sensitive data]
- **Data in Transit**: [e.g., TLS 1.3 for all communications]
- **Database Encryption**: [e.g., Transparent Data Encryption (TDE)]
- **Key Management**: [e.g., AWS KMS, Azure Key Vault, or similar]

### Sensitive Data Handling
- **PII Classification**: [Define what constitutes PII in your domain]
- **Data Minimization**: [Collect only necessary data]
- **Data Retention**: [Define retention periods for different data types]
- **Data Anonymization**: [Remove identifiers for analytics]

### Example Data Protection
```typescript
// Encrypt sensitive data before storage
async function encryptSensitiveData(data: any): Promise<string> {
  const key = await getEncryptionKey();
  return await encrypt(JSON.stringify(data), key);
}

// Hash passwords with salt
async function hashPassword(password: string): Promise<string> {
  const salt = await bcrypt.genSalt(12);
  return await bcrypt.hash(password, salt);
}
```

## API Security

### API Protection Patterns
- **HTTPS Only**: All API endpoints must use HTTPS
- **CORS Configuration**: [Define allowed origins and methods]
- **API Versioning**: [Version your APIs for security updates]
- **Request Validation**: [Validate all request parameters and body]
- **Response Sanitization**: [Remove sensitive data from responses]

### API Security Headers
```typescript
// Security headers middleware
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

## Vulnerability Management

### Common Vulnerabilities to Prevent
- **OWASP Top 10**: [Address all OWASP vulnerabilities]
- **Dependency Vulnerabilities**: [Regular security scanning]
- **Code Injection**: [Prevent all forms of code injection]
- **Broken Authentication**: [Secure session management]
- **Sensitive Data Exposure**: [Encrypt and protect sensitive data]

### Security Testing
- **Static Analysis**: [Use tools like ESLint security plugins]
- **Dependency Scanning**: [npm audit, Snyk, etc.]
- **Penetration Testing**: [Regular security assessments]
- **Code Review**: [Security-focused code reviews]

### Example Security Checks
```typescript
// Prevent NoSQL injection
function sanitizeMongoQuery(query: any): any {
  // Remove potentially dangerous operators
  const dangerousOps = ['$where', '$regex', '$ne', '$gt', '$lt'];
  return Object.keys(query).reduce((clean, key) => {
    if (!dangerousOps.includes(key)) {
      clean[key] = query[key];
    }
    return clean;
  }, {});
}

// Validate file uploads
function validateFileUpload(file: Express.Multer.File): boolean {
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
  const maxSize = 5 * 1024 * 1024; // 5MB
  
  return allowedTypes.includes(file.mimetype) && file.size <= maxSize;
}
```

## Monitoring & Incident Response

### Security Monitoring
- **Failed Login Attempts**: [Log and alert on suspicious activity]
- **Unusual API Usage**: [Monitor for abuse patterns]
- **Data Access Logs**: [Track who accesses what data]
- **System Anomalies**: [Monitor for unusual system behavior]

### Incident Response
- **Security Incident Plan**: [Define response procedures]
- **Escalation Procedures**: [Who to contact for security issues]
- **Forensic Requirements**: [Preserve evidence for investigation]
- **Communication Plan**: [How to communicate security incidents]

### Example Security Monitoring
```typescript
// Log security events
function logSecurityEvent(event: string, details: any, userId?: string) {
  const logEntry = {
    timestamp: new Date().toISOString(),
    event,
    details,
    userId,
    ip: req.ip,
    userAgent: req.get('User-Agent')
  };
  
  // Send to security monitoring system
  securityLogger.warn(logEntry);
}

// Rate limiting
const rateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});
```

## Compliance & Standards

### Security Standards
- **ISO 27001**: [Information security management]
- **SOC 2**: [Security, availability, processing integrity]
- **PCI DSS**: [Payment card industry data security]
- **GDPR**: [General data protection regulation]
- **HIPAA**: [Health insurance portability and accountability]

### Security Controls
- **Access Controls**: [Who can access what systems]
- **Audit Logging**: [Log all security-relevant events]
- **Backup Security**: [Secure backup and recovery procedures]
- **Change Management**: [Secure change control processes]

## Security Code Patterns

### Secure Coding Practices
- **Principle of Least Privilege**: [Minimal necessary permissions]
- **Defense in Depth**: [Multiple layers of security]
- **Fail Secure**: [System fails in secure state]
- **Input Validation**: [Validate all inputs]
- **Output Encoding**: [Encode outputs to prevent injection]

### Example Secure Patterns
```typescript
// Secure database queries
async function getUserById(id: string): Promise<User | null> {
  // Use parameterized queries to prevent SQL injection
  const result = await db.query(
    'SELECT id, email, name FROM users WHERE id = $1',
    [id]
  );
  return result.rows[0] || null;
}

// Secure error handling
function handleError(error: Error, context: string) {
  // Log error details for debugging
  logger.error({ error: error.message, context });
  
  // Return generic error to user (don't expose internals)
  return { error: 'An unexpected error occurred' };
}
```

## Security Checklist

### Pre-Deployment Security
- [ ] All inputs validated and sanitized
- [ ] Authentication and authorization implemented
- [ ] Sensitive data encrypted
- [ ] Security headers configured
- [ ] Dependencies scanned for vulnerabilities
- [ ] Security tests passing
- [ ] Error handling doesn't leak information
- [ ] Logging configured for security events

### Ongoing Security
- [ ] Regular security updates applied
- [ ] Dependencies kept up to date
- [ ] Security monitoring active
- [ ] Incident response plan tested
- [ ] Security training completed
- [ ] Penetration testing scheduled
- [ ] Compliance requirements met

---

## Maintenance Notes

**Last Updated**: [Date]
**Owner**: [Security team or lead developer]
**Review Schedule**: Monthly security review

This file should be updated when:
- New security threats are identified
- Compliance requirements change
- Security incidents occur
- New security tools are adopted
- Security architecture evolves

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harrysayers7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
