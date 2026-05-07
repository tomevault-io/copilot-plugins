## security-requirements

> Security requirements for tools and guardrails


# Mandatory Security Practices

1. **Path traversal prevention**: Validate paths against whitelist; no `..`; resolve symlinks before access.
2. **Code execution sandboxing**: Subprocess with timeouts and resource limits; never `eval()` or `exec()` on untrusted input.
3. **PII detection**: Scan output for emails, phone numbers, SSNs, credit cards, API keys.
4. **Secret detection**: Scan for AWS keys, GitHub tokens, passwords, connection strings, JWT secrets.
5. **Prompt injection defense**: Validate external inputs for instruction override attempts.
6. **Dangerous pattern blocking**: Block `eval`, `exec`, `os.system`, `subprocess.call(shell=True)`, `__import__`, `pickle.loads`, `yaml.load` without SafeLoader.
7. **Audit logging**: Log file operations, code executions, and tool invocations with timestamps.

# Security Guardrail Integration

Wrap tool operations: pre-execution validate inputs (run_security_guardrails); on fail raise SecurityViolation; post-execution scan output for secrets and redact if needed.

```python
def secure_tool_wrapper(func):
    def wrapper(*args, **kwargs):
        security_result = run_security_guardrails(args, kwargs)
        if security_result.status == "fail":
            raise SecurityViolation(security_result.message)
        result = func(*args, **kwargs)
        output_scan = scan_output_for_secrets(result)
        if output_scan.has_findings:
            result = redact_secrets(result)
        return result
    return wrapper
```

---
> Source: [RickZee/ai-team](https://github.com/RickZee/ai-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
