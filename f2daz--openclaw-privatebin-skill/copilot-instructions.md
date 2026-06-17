## openclaw-privatebin-skill

> Read and create PrivateBin pastes via CLI. Use when a user shares a PrivateBin URL (privatebin.*, ?pasteid=*#*), asks to create a paste, or needs to share text securely. Handles client-side decryption without a browser.


# PrivateBin

Read and create encrypted pastes via `pbcli`.

## Reading a Paste

```bash
pbcli "<full-privatebin-url-with-key>"
```

The URL **must include** the `#key` fragment. Output is the decrypted plaintext.

If the paste is password-protected:

```bash
pbcli -p "<password>" "<url>"
```

## Creating a Paste

```bash
echo "content here" | pbcli https://privatebin.example/ --expire 1week --json
```

Or pipe a file:

```bash
cat file.txt | pbcli https://privatebin.example/ --expire 1week --json
```

Options:
- `--expire`: `5min`, `10min`, `1hour`, `1day`, `1week`, `1month`, `1year`, `never`
- `--format`: `plaintext` (default), `syntax`, `markdown`
- `--burn`: burn after reading
- `-p <password>`: password-protect the paste
- `--json`: return JSON with paste URL, delete URL, and key

## Wrapper Script

Use `scripts/pb.sh` for convenience:

```bash
# Read
scripts/pb.sh get "<url>"

# Create
scripts/pb.sh put "text to paste"
scripts/pb.sh put --burn --expire 1hour "secret text"
cat file.txt | scripts/pb.sh put
```

---
> Source: [f2daz/openclaw-privatebin-skill](https://github.com/f2daz/openclaw-privatebin-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
