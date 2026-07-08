## http

> PSR-18 HTTP client on cURL with handler pipeline (Transformers + Transport) and optional Retry. Contains its own PSR-7/PSR-17 implementation (`src/Message/Psr17Factory`) — no external PSR-7 dependencies.

# jardisadapter/http

PSR-18 HTTP client on cURL with handler pipeline (Transformers + Transport) and optional Retry. Contains its own PSR-7/PSR-17 implementation (`src/Message/Psr17Factory`) — no external PSR-7 dependencies.

## Usage essentials

- **User API:** `new HttpClient(requestFactory, streamFactory, responseFactory, uriFactory, config)`. `ClientConfig` is a readonly VO, `HttpClient` has zero business logic — only Pipeline orchestration.
- **Convenience methods set JSON automatically:** `post()`/`put()`/`patch()` serialize arrays to JSON and set `Content-Type` + `Accept`. `get()`/`delete()`/`head()` have no body. Optional last parameter is custom headers per request.
- **Pipeline is built from Config, only what is configured is instantiated:** Transformers (`BaseUrl`, `DefaultHeaders`, `BearerAuth`, `BasicAuth`) → Transport (`CurlTransport`, wrapped by `Retry` if `maxRetries > 0`). Bearer takes precedence over Basic.
- **No Exception on 4xx/5xx** — those are valid responses. Only `NetworkException` (DNS, Connect-Refused, Timeout) and `RequestException` (malformed URI) are thrown, both extend `HttpClientException`. The Retry wrapper retries 5xx + `HttpClientException` with exponential backoff (`retryDelayMs`).
- **Custom Transport injectable via Closure:** `transport: function (RequestInterface, ClientConfig): ResponseInterface`. Enables mocks without real HTTP calls — all Unit Tests in the package use this path.
- **No caching/logging in the package.** Implement cross-cutting concerns as a Decorator on `Psr\Http\Client\ClientInterface` in the caller project; never instantiate Handlers directly.

## Full reference

https://docs.jardis.io/en/adapter/http

---
> Source: [jardisAdapter/http](https://github.com/jardisAdapter/http) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
