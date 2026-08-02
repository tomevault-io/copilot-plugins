## writebook

> Prefer using existing fixtures over creating new records in tests. For example, use `leaves(:welcome_page)` instead of `books(:handbook).press Page.new(body: "..."), title: "..."`.

# Agent Guidelines

## Testing Conventions

### Use Existing Fixtures
Prefer using existing fixtures over creating new records in tests. For example, use `leaves(:welcome_page)` instead of `books(:handbook).press Page.new(body: "..."), title: "..."`.

### URL Helpers in Tests
Use `_path` helpers instead of `_url` helpers in controller/integration tests unless you need to test across different hosts or explicitly need the full URL.

**Preferred:**
```ruby
get book_slug_path(books(:handbook))
```

**Avoid:**
```ruby
get book_slug_url(books(:handbook))
```

### Response Body Assertions
Use `assert_in_body` and `assert_not_in_body` to check if text is present or absent in the response body without DOM manipulation.

**Preferred:**
```ruby
assert_in_body "Expected content"
assert_not_in_body "Unexpected content"
```

**Avoid:**
```ruby
assert_includes response.body, "Expected content"
assert_not_includes response.body, "Unexpected content"
```

## Controller Conventions

### Simplified respond_to Blocks
When using `respond_to` with default rendering, omit the `{ render }` block as it's implied.

**Preferred:**
```ruby
def show
  respond_to do |format|
    format.html
    format.md
  end
end
```

**Avoid:**
```ruby
def show
  respond_to do |format|
    format.html { render }
    format.md { render }
  end
end
```

---
> Source: [basecamp/writebook](https://github.com/basecamp/writebook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
