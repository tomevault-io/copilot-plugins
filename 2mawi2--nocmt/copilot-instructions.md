## nocmt

> **Configuration Files (`.nocmt.json`):**

**Configuration Files (`.nocmt.json`):**
-   Local: `.nocmt.json` (project root)
-   Global: `~/.nocmt/config.json`
-   Local overrides global. CLI flags override both. See `config/config.go`.

**Example `.nocmt.json` Structure:**
```json
{
  "ignorePatterns": [
    "TODO",            // Preserves comments with "TODO"
    "^// WHY:"         // Preserves comments starting with "// WHY:" (regex)
  ],
  "fileIgnorePatterns": [
    "\\.test\\.js$",   // Ignores *.test.js files (regex)
    "dist/"            // Ignores files in any 'dist' directory
  ]
}
```

**Common CLI Usage (`main.go`):**
-   `nocmt`: Processes modified lines in Git-staged files (default).
-   `nocmt [path/to/file_or_dir]`: Process a specific file (output to stdout) or directory (modifies in place).
-   `nocmt --dry-run` or `-d`: Preview changes without writing.
-   `nocmt --staged` or `-s`: Explicitly process staged files.
-   `nocmt --all` or `-a`: Process all files in current dir recursively.
-   `nocmt --ignore "PATTERN1,PATTERN2"`: Comma-separated regex for comments to keep.
-   `nocmt --ignore-file "PATTERN"`: Regex for files/dirs to skip.
-   `nocmt --add-ignore "PATTERN"`: Add comment pattern to local `.nocmt.json`.
-   `nocmt --add-ignore-global "PATTERN"`: Add comment pattern to global config.
-   `nocmt --remove-directives` or `-r`: Remove compiler directives (they are preserved by default).
-   `nocmt install-hooks`: Install Git pre-commit hook.
-   `nocmt --version`: Show version.
-   `nocmt --verbose` or `-v`: Detailed output.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2mawi2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
