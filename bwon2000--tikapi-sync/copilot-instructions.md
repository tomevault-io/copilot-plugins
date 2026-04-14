## tikapi-sync

> > **📋 Source-of-Truth**: This file complements README.md's Database Safety Guidelines. All database operations must follow both documents.

# Database Safety Protection Mechanisms

> **📋 Source-of-Truth**: This file complements README.md's Database Safety Guidelines. All database operations must follow both documents.

## 🛡️ Multi-Layer Security Architecture

**Protected Invariants (from README.md):**
- Supabase tables: `influencer_data`, `tiktok_video_data`, `campaigns`, `campaigns_influencers`
- Storage bucket: `profile-images` (public read; service-role insert/update)
- RLS policies must remain unchanged unless explicitly approved

### Layer 1: Code-Level Safeguards
- **Location**: `scripts/db-query.js`, `scripts/db-inspect.js`
- **Protection**: Regex pattern detection for destructive queries
- **Bypass Risk**: HIGH - Code can be modified

### Layer 2: Guardian Validation
- **Location**: `scripts/db-guardian.js`
- **Protection**: File integrity monitoring, pattern validation, tamper detection
- **Bypass Risk**: MEDIUM - Requires multiple file modifications

### Layer 3: Safe Command Wrapper
- **Location**: `scripts/safe-db-wrapper.js`
- **Protection**: Whitelist-only command execution, blocked pattern detection
- **Bypass Risk**: MEDIUM - Can be bypassed with direct script calls

### Layer 4: Environment Controls
- **Protection**: Separate environment variables for read/write operations
- **Implementation**: Use different Supabase keys for different permission levels

## 🚨 Known Attack Vectors & Mitigations

### Attack Vector 1: Direct Code Modification
**Risk**: Cursor modifies safety scripts to remove protections
**Mitigation**: 
- File integrity monitoring in db-guardian.js
- Tamper evidence logging
- Regular checksum validation

### Attack Vector 2: Direct Supabase Client Usage
**Risk**: Bypass scripts entirely with direct Supabase calls
**Mitigation**:
- Environment key separation (read-only vs admin)
- Process monitoring for suspicious patterns
- Safe wrapper command validation

### Attack Vector 3: Node.js Eval/Exec Bypass
**Risk**: `node -e "destructive code"` or process.exec()
**Mitigation**:
- Command pattern blocking in safe wrapper
- Process argument monitoring
- Whitelist-only execution

### Attack Vector 4: Environment Variable Manipulation
**Risk**: Overriding SUPABASE_SERVICE_ROLE_KEY at runtime
**Mitigation**:
- Environment validation in guardian
- Key rotation monitoring
- Suspicious pattern detection

## 🔒 Ultimate Protection Strategy

### Option 1: Environment Key Separation
```bash
# Read-only operations (safe)
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_anon_key_here

# Admin operations (restricted)
SUPABASE_ADMIN_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your_service_key_here
```

### Option 2: Database-Level Protection
- Use Supabase Row Level Security (RLS)
- Create read-only database roles
- Implement stored procedures for safe operations

### Option 3: External Validation Service
- Run guardian checks on external server
- Require external approval for destructive operations
- Implement change request workflow

## 📋 Security Validation Checklist

**README.md Alignment:**
- [ ] Changes reference README "Planned Changes" Active Development item
- [ ] Protected invariants unchanged (tables, storage, RLS policies)
- [ ] Database operations follow README Database Safety Guidelines

**Multi-Layer Protection:**
- [ ] Guardian integrity check passes (`npm run db:validate`)
- [ ] No suspicious patterns detected
- [ ] Safety scripts unchanged
- [ ] Environment variables validated
- [ ] Tamper evidence reviewed
- [ ] Command whitelist enforced (`npm run db:safe`)

## 🆘 Incident Response

If safety bypass is detected:
1. **Immediate**: Stop all database operations
2. **Investigate**: Review .db-guardian-log.json for tamper evidence
3. **Restore**: Revert to known-good safety script versions
4. **Validate**: Run full integrity check
5. **Document**: Log incident and update protections

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bwon2000) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
