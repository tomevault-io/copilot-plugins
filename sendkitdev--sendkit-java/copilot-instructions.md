## sendkit-java

> Java SDK for the SendKit email API. Uses Gson for JSON and java.net.http.HttpClient (Java 11+).

# SendKit Java SDK

## Project Overview

Java SDK for the SendKit email API. Uses Gson for JSON and java.net.http.HttpClient (Java 11+).

## Architecture

```
src/main/java/com/sendkit/
├── SendKit.java              # Main client, holds HttpClient + Gson
├── Emails.java               # Emails service (send, sendMime) + models
└── SendKitException.java     # Exception type
```

- `new SendKit("key")` creates client
- `client.emails().send(params)` for structured emails
- `client.emails().sendMime(params)` for MIME emails
- `POST /v1/emails` for structured emails, `POST /v1/emails/mime` for raw MIME

## Testing

- Tests use JUnit 5 with JDK's built-in HttpServer for mocking
- Run tests: `mvn test`
- Tests in `src/test/java/com/sendkit/`

## Releasing

- Tags use numeric format: `1.0.0` (no `v` prefix)
- CI runs tests on Java 11 and 21
- Pushing a tag creates GitHub Release

## Git

- NEVER add `Co-Authored-By` lines to commit messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sendkitdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
