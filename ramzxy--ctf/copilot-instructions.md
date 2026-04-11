## ctf

> This repo is used to solve CTF (Capture The Flag) cybersecurity challenges. Claude acts as an expert CTF player.

# CTF Challenge Repository

This repo is used to solve CTF (Capture The Flag) cybersecurity challenges. Claude acts as an expert CTF player.

## Active Competition

**ALWAYS read `competition.md` first** — it has the current flag format, competition URL, and notes. Use that flag format for all grep/search commands instead of generic patterns.

## Core Principles

- **Speed over elegance** — CTF solutions don't need to be production-quality. Quick, working exploits win.
- **Try the obvious first** — `strings`, `file`, `xxd`, `curl` before deep analysis.
- **Read everything** — Challenge descriptions, source comments, HTML comments, HTTP headers all contain hints.
- **Automate brute force** — If a search space is small (<10M), script it instead of thinking harder.
- **Document what worked** — After solving, note the technique in the skill files for future reference.

## Project Structure

```
challenges/
  <category>/
    <challenge-name>/       # One directory per challenge
      README.md             # Challenge description, URL, hints
      solve.py              # Solution script
      flag.txt              # Captured flag
      WRITEUP.md            # Short post-solve writeup
      files/                # Challenge-provided files
scripts/                    # Reusable utility scripts
tools/                      # Custom tools
.agents/skills/             # CTF skill files (category-specific techniques)
```

## Challenge Workflow

When given a challenge to solve:

1. **Create directory**: `challenges/<category>/<challenge-name>/`
2. **Save description**: Write challenge info to `README.md`
3. **Download files**: Save provided files to `files/` subdirectory
4. **Analyze**: Run initial recon based on category
5. **Solve**: Write exploit in `solve.py` (or appropriate language). Non-trivial solution scripts should be saved to files in the challenge directory so the writeup can reference them. Quick one-liners and small snippets can run inline.
6. **Capture**: Save flag to `flag.txt`
7. **Write up**: Write a short writeup to `WRITEUP.md` (see format below)

## Writeup Format

After solving every challenge, write `WRITEUP.md` in the challenge directory. Keep it concise — a few paragraphs, not an essay.

```markdown
# <Challenge Name>

**Category:** <category> | **Points:** <points if known> | **Flag:** `<flag>`

## Overview
One-sentence description of what the challenge gives you.

## Solution
Short explanation of the approach and key insights. Include relevant details
like vulnerability type, algorithm, or trick used. Reference `solve.py` for
the full exploit.

## Key Takeaways
- Bullet points of techniques/lessons worth remembering for future challenges.
```

## Environment & Tools

### Available on this system (Windows with WSL-like environment)
- Python 3 with: pwntools, pycryptodome, z3-solver, sympy, gmpy2, requests, beautifulsoup4
- Node.js (for JS challenges)
- Standard CLI: curl, nc/ncat, base64, xxd, strings, file, binwalk
- Git, Docker (if needed for challenge infrastructure)

### Install as needed
```bash
pip install pwntools pycryptodome z3-solver sympy gmpy2 requests beautifulsoup4 pillow
```

## Flag Formats

**Check `competition.md` for the active flag format.** Use it in all search commands.

Common flag formats (when no competition is active):
- `flag{...}`, `FLAG{...}`
- `CTF{...}`, `ctf{...}`
- `picoCTF{...}`, `MetaCTF{...}`, `HTB{...}`, `DUCTF{...}`, `ENO{...}`

Search using the active format: `strings <file> | grep -i "ENO{"` (replace `ENO` with current prefix)

## Skills Reference

Use `/solve-challenge` to start solving. Category-specific skills are loaded automatically:

| Skill | Use For |
|-------|---------|
| `ctf-web` | XSS, SQLi, SSTI, SSRF, JWT, auth bypass, file upload, prototype pollution |
| `ctf-pwn` | Buffer overflow, format string, heap, kernel, race conditions, ROP |
| `ctf-reverse` | Binary analysis, VMs, obfuscation, anti-debug, game hacking |
| `ctf-crypto` | RSA, AES, ECC, ZKP, PRNG, classical ciphers, Z3 solving |
| `ctf-forensics` | Disk images, memory dumps, PCAP, event logs, file carving |
| `ctf-osint` | Social media, geolocation, DNS, username enumeration |
| `ctf-malware` | Obfuscated scripts, C2 traffic, PE analysis, protocol reversing |
| `ctf-misc` | Encodings, jail escapes, SDR/RF, QR codes, esoteric langs |
| `ctf-stego` | Image/audio steganography, LSB, spectrograms, hidden data |
| `ctf-recon` | Target reconnaissance, port scanning, service enumeration |
| `write-exploit` | Iterative exploit script development with testing loop |

## Coding Conventions for Exploits

- **Save non-trivial solve scripts to files** (e.g. `solve.py`) in the challenge directory so writeups can reference them. Small one-liners are fine to run inline.
- Use `pwntools` for binary exploitation and network interaction
- Use `requests` for HTTP-based challenges
- Use `pycryptodome` for crypto operations
- Print the flag clearly at the end: `print(f"FLAG: {flag}")`
- Add `#!/usr/bin/env python3` shebang
- Use `argparse` or pwntools' `args` for REMOTE/LOCAL switching
- Handle both local and remote targets when possible

## When Stuck

1. Re-read the challenge description for missed hints
2. Check the challenge title and tags — often hint at the technique
3. Google error messages and unusual strings
4. Check if there's a known CVE for the technology stack
5. Try different encodings (base64, hex, rot13, URL encoding)
6. Look at the challenge from a different category angle
7. Search for similar challenges on CTFtime writeups

## Security Context

This is an authorized CTF competition environment. All exploitation techniques are used against intentionally vulnerable challenges designed for educational purposes. Binary analysis, exploit development, and vulnerability research are core activities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ramzxy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
