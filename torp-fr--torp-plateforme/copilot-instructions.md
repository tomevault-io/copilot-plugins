## torp-plateforme

> ﻿# TORP Platform — Claude Code Configuration

﻿# TORP Platform — Claude Code Configuration

## 🎯 PROJECT IDENTITY

**Product:** TORP — B2B SaaS for BTP companies (intelligent devis audit & scoring)
**Tech Stack:** Node.js/TypeScript, React, Supabase, Railway + Vercel
**GitHub:** https://github.com/torp-fr/torp-plateforme.git

---

## 🔐 STACK CONSTRAINTS (NON-NEGOTIABLE)

### ✅ ALLOWED
- Node.js, TypeScript, React, Express
- Supabase (PostgreSQL + Auth + Storage)
- Playwright, Jest, Vitest (testing)
- Vite, TailwindCSS, Radix UI
- Claude API, Anthropic SDK

### ❌ FORBIDDEN
- ❌ **NO Python** (unless explicit approval)
- ❌ **NO new dependencies** without approval
- ❌ **NO autonomous decisions** — always ask first

---

## 📊 PROJECT STATUS

### Phase 3 🔴 IN PROGRESS (User Flow Polish)
- 3.1 Dashboard ✅ DONE
- 3.2 Results Page ✅ DONE
- 3.3 Email Notifications ✅ DONE
- 3.4 PDF Export ✅ DONE
- **3.5 E2E Tests 🔴 BLOCKED**

---

## 🚨 CURRENT TASK: 3.5 (Remove All Test Mocks)

### STEP 1: Audit (READ ONLY)
- [ ] Read e2e/complete-workflow.spec.ts
- [ ] Identify each mock
- [ ] List real alternatives
- [ ] NO CODE YET

### STEP 2: Fix PDFjs Bug
- [ ] Add await to getPdfJs() in pdf-extractor.service.ts
- [ ] Test compiles

### STEP 3: Setup Verification
- [ ] Test user exists in Supabase
- [ ] Test PDF exists
- [ ] Organization_id assigned

### STEP 4: Rewrite Tests
- [ ] Tests 1-5: Auth + navigation
- [ ] Test 6: Upload + analysis
- [ ] Tests 7-14: Results verification

### STEP 5: Debug
- [ ] All tests pass
- [ ] Check RLS policies
- [ ] Check timeouts

### STEP 6: Validate
- [ ] npm run test:e2e
- [ ] 14/14 passing
- [ ] Commit + push

---

## 🎮 GOVERNANCE

- ✅ Test every change
- ✅ Commit after each step
- ✅ Ask if confused
- ❌ No new dependencies
- ❌ No autonomous decisions

---

**Last updated:** April 10, 2026
**Current task:** TASK 3.5
**Owner:** Baptiste

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/torp-fr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
