## clserver

> Parenscript-specific coding patterns and best practices


# Parenscript Patterns and Best Practices

## JavaScript Object Access

- Use `(@ object property)` for property access
- Use `(@ object "property")` for string-based property access
- Use `(funcall (@ object method) args...)` for method calls

## Common Patterns

```lisp
;; HTTP server creation
(let ((server (funcall (@ *http* create-server) handle-request)))
  (server.listen *port* callback))

;; Object property setting
(setf (@ res status-code) 200)
(setf (@ res "content-type") "text/html")

;; Array/string access
(aref string index)
(@ pathname 0 6)  ; substring from 0 to 6
```

## Syntax Validation

- **Always test Parenscript with SBCL** before making changes
- Use `sbcl --eval "(ps (your-code-here))"` to test Parenscript syntax
- Validate JSON handling with `sbcl --eval "(ps (JSON.stringify data))"`
- Test complex Parenscript expressions individually
- **Never edit Lisp files without SBCL validation**

## HTML Generation

- Use `(str ...)` for string concatenation
- Include complete HTML documents with DOCTYPE
- Use inline CSS for styling
- Escape special characters properly

## Error Handling

- Implement proper HTTP status codes
- Use descriptive error messages
- Handle 404 cases for unknown routes

## Server Configuration

- Define ports as variables: `(defvar *port* 3000)`
- Use descriptive server names in console output
- Include helpful startup messages with example URLs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codegod100) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
