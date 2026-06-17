## devpipe

> Unified developer productivity CLI that compresses text by removing low-information tokens (LLM surprisal scoring) and generates technical specifications via Groq API.

# devpipe

Unified developer productivity CLI that compresses text by removing low-information tokens (LLM surprisal scoring) and generates technical specifications via Groq API.

## Trigger
Use this skill when the user asks to:
- Compress text or code to reduce token count
- Remove low-information tokens from text
- Generate a technical specification from a prompt
- Generate a spec then compress the output

## Instructions

### Compress text

```bash
# From file
devpipe compress path/to/file.txt --keep-ratio 0.7 --stats

# From stdin
echo "long text" | devpipe compress --keep-ratio 0.7

# Auto-detect optimal keep ratio
devpipe compress input.txt --auto --stats

# Keep exact number of tokens
devpipe compress input.txt --target-tokens 200

# Use Groq API for compression instead of local model
devpipe compress input.txt --groq --stats
```

### Generate a technical spec

Requires `GROQ_API_KEY` environment variable.

```bash
devpipe generate "A real-time collaborative editor" -o spec.md
devpipe generate "Auth system with OAuth2" --json --stats
```

### Generate then compress

```bash
# Generate a spec and compress the output
devpipe generate-compress "Payment processing microservice" --stats

# With custom keep ratio
devpipe generate-compress "Auth system" --keep-ratio 0.5 --stats

# Auto-detect optimal compression ratio
devpipe generate-compress "CLI tool" --auto

# Use Groq API for compression step
devpipe generate-compress "API gateway" --groq

# Output to file as JSON
devpipe generate-compress "Auth system" -o spec.md --json
```

### Installation

If `devpipe` is not installed:
```bash
cd ~/devpipe2 && cargo install --path .
```

## Notes
- Compression uses a local quantized Llama model via candle (Metal GPU on macOS).
- First run downloads a ~700 MB GGUF model from HuggingFace (cached locally).
- Spec generation uses Groq API (free tier, no credit card needed).
- On macOS 26+, you may need: `export CXXFLAGS="-isystem $(xcrun --show-sdk-path)/usr/include/c++/v1"`

---
> Source: [Amir-Zecharia/devpipe](https://github.com/Amir-Zecharia/devpipe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
