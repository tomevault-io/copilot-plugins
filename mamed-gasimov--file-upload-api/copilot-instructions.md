## file-upload-api

> Strictly prevent AI from accessing, exposing, inferring, or modifying environment files or secrets


# 🔐 Environment & Secret Protection Policy

## 🚫 Absolute Prohibitions

The AI MUST NEVER:

- Read, open, parse, summarize, or reference `.env` files
- Read any `.env.*` file (e.g., `.env.local`, `.env.production`, `.env.staging`)
- Modify environment files
- Generate, expose, or echo environment variable values
- Infer secret values from context, logs, stack traces, configs, or examples
- Suggest committing `.env` files to version control
- Suggest printing environment variables for debugging
- Suggest logging secrets to console, files, or monitoring systems
- Request users to paste secret values into chat
- Store, cache, or remember secret values

## 🔎 Indirect Exposure Prevention

The AI MUST NOT:

- Extract secrets from:
  - Error messages
  - Debug output
  - Build logs
  - CI/CD logs
  - Stack traces
  - Serialized config objects
- Reconstruct partial secrets
- Guess likely API key formats
- Provide examples using realistic-looking private keys or tokens
- Output placeholders that resemble real credentials (e.g., `sk-live-...`, long base64 strings, JWT-like strings)

All example credentials must use clearly fake placeholders such as:

- `YOUR_API_KEY`
- `YOUR_SECRET_HERE`
- `REPLACE_WITH_TOKEN`

## 📘 Strictly Allowed Reference

The AI MAY reference `.env.example` only for:

- Variable names
- Expected structure
- Documentation guidance

The AI MUST:

- Treat `.env.example` as a schema reference only
- Never assume values match real environments
- Never generate filled-in example secrets

## 🛑 If Environment Files Are Detected

If any environment file is encountered:

1. Immediately ignore it.
2. Do not inspect contents.
3. Do not summarize.
4. Do not infer values.
5. Do not reference it in output.
6. Continue the task without using it.

## 🧠 Secure Coding Guidance

When environment variables are required in examples:

- Use `process.env.VARIABLE_NAME` (or equivalent) without values
- Use explicit placeholders
- Encourage secure storage practices
- Recommend secret managers when appropriate

## 🔐 Rationale

Environment files contain sensitive data such as:

- API keys
- Tokens
- Passwords
- Private keys
- Database credentials
- OAuth secrets
- Encryption keys

These must always be treated as confidential and never exposed, processed, or inferred by the AI.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mamed-gasimov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
