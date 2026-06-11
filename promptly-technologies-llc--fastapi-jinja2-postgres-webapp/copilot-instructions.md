## routers-tests

> Since this is a FastAPI web application, test logic for API endpoints often involves checking status codes. Remember, when making a request to an API endpoint, you should specify the `follow_redirects` parameter. With `follow_redirects=False`, the response code will often be `303`; otherwise it will be the response code of the route we've redirected to. We mostly  use `follow_redirects=False` so as to test routes in isolation, but there may be test cases where following the redirect is more appropriate.

# Setting test expectations regarding HTTP status codes

Since this is a FastAPI web application, test logic for API endpoints often involves checking status codes. Remember, when making a request to an API endpoint, you should specify the `follow_redirects` parameter. With `follow_redirects=False`, the response code will often be `303`; otherwise it will be the response code of the route we've redirected to. We mostly  use `follow_redirects=False` so as to test routes in isolation, but there may be test cases where following the redirect is more appropriate.

When checking status codes, think carefully to make sure the expected status code is the most appropriate to the situation.

---
> Source: [Promptly-Technologies-LLC/fastapi-jinja2-postgres-webapp](https://github.com/Promptly-Technologies-LLC/fastapi-jinja2-postgres-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
