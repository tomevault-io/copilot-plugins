## typed-errors

> Handler and usecase errors must be typed or wrapped with a reason


# Typed and wrapped errors

Never return a bare `err` across package boundaries without context — handlers, usecase, integration.

## Required

- Prefer package-level sentinel errors (`var ErrInvalidID = errors.New(...)`).
- Otherwise wrap with `github.com/pkg/errors`: `errors.Wrap(err, "get task")` / `errors.Wrapf(...)`.
- Handlers map via `respondError` (or equivalent), not `return err` after `parseID`.

## Bad

```go
payload, _ := json.Marshal(body)
```

```go
key, err := ParseJiraIssue(issue)
if err != nil {
  return "", err
}
```

## Good

```go
payload, err := json.Marshal(body)
if err != nil {
  return "", errors.Wrap(err, "marshal jira comment")
}
```

```go
key, err := ParseJiraIssue(issue)
if err != nil {
  return "", errors.Wrap(err, "parse jira issue")
}
```

---
> Source: [gonnafaraway/kaiban](https://github.com/gonnafaraway/kaiban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
