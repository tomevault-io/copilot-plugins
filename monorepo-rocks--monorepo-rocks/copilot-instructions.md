## python-inline-scripts

> ALWAYS read this guide when running inline python scripts. This should be used when the user says "use inline python" or "run inline python

<python-inline-scripts>

<title>Python Inline Scripts with uv</title>

<IMPORTANT>
- NEVER add stdlib modules like json, os, re, etc. to dependencies OR THE SCRIPT WILL FAIL
- ALWAYS assume Python 3.12+
</IMPORTANT>

<example>
uv run --no-project -q --script - < <(cat <<'EOF'
# /// script
# # ONLY add dependencies array if you need PyPi packages
# dependencies = [
#   "httpx"
# ]
# requires-python = ">=3.12"
# ///

import httpx
print(httpx.get("https://api.github.com/zen").text)
EOF
)
</example>

</python-inline-scripts>

---
> Source: [monorepo-rocks/monorepo-rocks](https://github.com/monorepo-rocks/monorepo-rocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
