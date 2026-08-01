## bruteforce-database

> **C**ommunity **L**ed **A**rchive of **U**niversal **D**ictionaries for **E**thical Security

# CLAUDE - Project Philosophy & Principles

**C**ommunity **L**ed **A**rchive of **U**niversal **D**ictionaries for **E**thical Security

---

## The Vision

This is not just a collection of text files. This is a **living archive** maintained by the security community, for the security community.

Every wordlist tells a story. Every password represents a pattern in human behavior. Every username is a glimpse into digital identity. This repository preserves that knowledge and makes it accessible.

## Core Principles

### 1. Quality Over Quantity

We don't collect every wordlist that exists. We curate the **essential sets** that security professionals actually need:
- Password dictionaries for brute-force testing
- Username collections for account enumeration
- Subdomain lists for reconnaissance
- Forced-browsing paths for web application testing

**Standard:** Every wordlist must have clear provenance, documented purpose, and validated quality.

### 2. Ethical Use

This repository exists to **improve security**, not undermine it:
- Wordlists help security teams test their defenses
- Penetration testers use these for authorized assessments
- Researchers analyze password patterns to educate users
- Developers test authentication systems against common attacks

**Responsibility:** We trust our community to use these resources ethically and legally.

### 3. Transparency

Every entry in this repository should be traceable:
- Where did this wordlist come from?
- When was it collected?
- How many unique entries does it contain?
- What is its intended use case?

**Commitment:** We maintain a comprehensive manifest with full metadata.

### 4. Community First

This repository belongs to everyone who contributes:
- Contributors add new wordlists with proper attribution
- Maintainers ensure quality and organization
- Users provide feedback on effectiveness
- The community decides what belongs

**Promise:** Every contribution is valued, credited, and preserved.

### 5. Evolution, Not Stagnation

The threat landscape changes. Password patterns evolve. Our wordlists must keep pace:
- Regular updates from new breach datasets (ethically sourced)
- Modern password patterns (passphrases, emoji passwords, etc.)
- Emerging patterns in usernames and domains
- Continuous improvement through automation

**Goal:** Stay relevant in 2025 and beyond.

## Technical Philosophy

### Simplicity

No complex dependencies. No proprietary formats. Just plain text files that work everywhere:
```bash
# It should always be this simple
cat passwords.txt | hydra -L users.txt -P - ssh://target
```

### Automation

Quality should be automatic, not manual:
- CI/CD validates every contribution
- Scripts deduplicate and normalize
- Manifest generation is automatic
- Statistics update on every commit

### Metadata

Every wordlist deserves rich context:
```json
{
  "path": "passwords/common-passwords.txt",
  "entries": 10000,
  "unique": 9847,
  "source": "SecLists Project",
  "updated": "2025-11-16",
  "purpose": "Most common passwords from breach data",
  "encoding": "utf-8",
  "sha256": "..."
}
```

### Organization

Structure should guide users naturally:
```
passwords/           # Password dictionaries
├── common/         # Most frequently used
├── specialized/    # Targeted lists (years, keyboard patterns, etc.)
└── massive/        # Large comprehensive sets

usernames/          # Username collections
identities/         # Names, locations, etc.
infrastructure/     # Subdomains, directories, files
```

## Design Decisions

### Why Plain Text?

- Universal compatibility (every tool can read .txt)
- Easy to inspect, verify, and audit
- Simple to combine, filter, and customize
- Minimal storage with compression

### Why Python for Tooling?

- Readable by humans (code should be documentation)
- Available everywhere (no complex setup)
- Rich standard library (no heavy dependencies)
- Fast enough for our use case

### Why CI/CD?

- Catches problems before they reach users
- Ensures consistent quality
- Documents what we care about (validated in tests)
- Builds trust in the repository

## The Future

This repository should become:

1. **The Standard Reference** - When someone needs a wordlist, they come here first
2. **A Living Dataset** - Continuously updated with modern patterns
3. **A Learning Resource** - Documentation teaches security concepts
4. **A Community Hub** - Contributors collaborate and share knowledge

## For Contributors

When you add a wordlist, ask yourself:

- **Unique:** Does this provide value that existing lists don't?
- **Sourced:** Can I document where this came from?
- **Clean:** Is it deduplicated, validated, and properly formatted?
- **Described:** Will users understand what it's for and how to use it?

If yes to all four, you're ready to contribute.

## For Users

When you use these wordlists:

- **Test Legally:** Only against systems you own or have authorization to test
- **Contribute Back:** Found a great wordlist? Share it with the community
- **Report Issues:** Bad data? Let us know so we can fix it
- **Give Credit:** Reference this repository in your work

---

**Remember:** The goal is not to break security, but to build it stronger.

---

*"Simplicity is the ultimate sophistication." - Leonardo da Vinci*

*"Quality is not an act, it's a habit." - Aristotle*

---
> Source: [duyet/bruteforce-database](https://github.com/duyet/bruteforce-database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
