## ok-keycloak

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`ok_keycloak` (composer: `oliverkroener/ok-keycloak`) signs TYPO3 frontend and backend users in
through Keycloak using the OpenID Connect authorization code flow. It talks to the Keycloak
endpoints over plain HTTP through TYPO3's `RequestFactory` and has **no third-party dependencies**.

**Extension key**: `ok_keycloak`
**Namespace**: `OliverKroener\OkKeycloak\`
**Version**: 5.0.0
**TYPO3**: 13.4, 14.x — v13-only APIs are avoided, v12 compatibility shims are deliberately absent
**PHP**: ^8.2 (requires `sodium`)

## Development context

This package lives at `packages/ok_keycloak/` in the TYPO3 13 monorepo at `/home/oliver/typo3-13`,
registered through the `packages/*` path repository. DDEV runs the stack
(project `typo3-13`, docroot `public`, https://typo3-13.ddev.site).

```bash
ddev composer update oliverkroener/ok-keycloak --no-interaction
ddev exec vendor/bin/typo3 database:updateschema   # after ext_tables.sql changes
ddev exec vendor/bin/typo3 cache:flush             # after config / class / DI changes
ddev exec vendor/bin/typo3 site:sets:list          # confirm the site set is registered
ddev exec vendor/bin/typo3 site:list               # site identifiers, needed for OK_KEYCLOAK_<SITE>_* vars
ddev mysql -e "SELECT * FROM tx_okkeycloak_configuration\G"
```

`cache:flush` is the cheapest real smoke test: it compiles the DI container, so a broken
`Services.yaml`, a bad `#[AsEventListener]` or an unresolvable constructor dependency fails here.

### Quality checks

There is **no test suite**, and the project ships no PHPStan or php-cs-fixer. Install them into a
throwaway directory rather than adding dev dependencies to the project:

```bash
cd /home/oliver/typo3-13
mkdir -p .tools-okkeycloak && cd .tools-okkeycloak
cat > composer.json <<'JSON'
{
    "require": {
        "phpstan/phpstan": "^2.1",
        "friendsofphp/php-cs-fixer": "^3.68",
        "typo3/coding-standards": "^0.8"
    },
    "config": { "allow-plugins": false }
}
JSON
```

PHPStan needs the project autoloader, because the extension is only resolvable through it:

```neon
# .tools-okkeycloak/phpstan.neon
parameters:
    level: 5
    paths:
        - /var/www/html/packages/ok_keycloak/Classes
    bootstrapFiles:
        - /var/www/html/vendor/autoload.php
    scanDirectories:
        - /var/www/html/vendor/typo3
```

```bash
ddev exec "cd /var/www/html/.tools-okkeycloak && composer install --no-interaction"
ddev exec "cd /var/www/html/.tools-okkeycloak && php -d memory_limit=1G vendor/bin/phpstan analyse -c phpstan.neon --no-progress"
# then, and only then, the fixer - point its finder at Classes/ plus the root *.php files
ddev exec "cd /var/www/html/.tools-okkeycloak && php vendor/bin/php-cs-fixer fix --config=.php-cs-fixer.php --dry-run --diff --using-cache=no"
rm -rf /home/oliver/typo3-13/.tools-okkeycloak
```

The php-cs-fixer config is `\TYPO3\CodingStandards\CsFixerConfig::create()` with the finder
pointed at the extension. Both currently pass clean (level 5, 0 of 26 files needing fixes) — keep
them that way.

Other checks:

```bash
ddev exec "cd /var/www/html/packages/ok_keycloak && composer validate --no-check-publish --no-check-lock"
ddev exec vendor/bin/yaml-lint packages/ok_keycloak/Configuration/Services.yaml packages/ok_keycloak/Configuration/Sets/Keycloak/*.yaml
xmllint --noout Resources/Private/Language/*.xlf Configuration/FlexForms/*.xml Resources/Public/Icons/*.svg
make docs        # renders Documentation/ via the TYPO3 render-guides Docker image
```

Label keys drift easily; this catches it:

```bash
comm -23 \
  <(grep -rhoE "locallang_be_module\.xlf:[A-Za-z0-9_.]+" Resources/Private Classes | sed 's/.*xlf://' | sort -u) \
  <(grep -oE 'trans-unit id="[^"]+"' Resources/Private/Language/locallang_be_module.xlf | sed 's/.*id="//;s/"//' | sort -u)
```

### Exercising the login flow without a Keycloak server

Most of the flow can be verified offline. Insert a configuration row whose `server_url` points at
an unreachable host (this also exercises the discovery fallback), then fetch `/typo3/` and read the
rendered authorize URL: the button, the resolved configuration, the decrypted secret, endpoint
derivation, the signed state and the PKCE challenge are all covered by that one request.

The client secret column is encrypted, so seed it by running `EncryptionService::encrypt()` in a
throwaway CLI script — it only needs `$GLOBALS['TYPO3_CONF_VARS']['SYS']['encryptionKey']` from
`config/system/settings.php`, no TYPO3 bootstrap. To confirm the callback would work, re-derive
`hash_hmac('sha256', 'pkce:' . nonce, encryptionKey)` from the state in the URL and check that its
SHA-256 matches the `code_challenge`. **Delete the seeded row afterwards** — an enabled row puts a
dead button on the real backend login screen.

## Architecture

### Configuration resolution — the core idea

`Service/ConfigurationService` resolves **each field separately** and takes the first layer with a
non-empty value, so the client secret can come from the environment while the redirect URI comes
from the database.

| Context | Order |
|---|---|
| Frontend | environment → site settings (`okKeycloak.*`) → database record of the site |
| Backend | database record (by uid → by site root → global row `site_root_page_id = 0`) → environment |

Environment names are `OK_KEYCLOAK_<SITE_IDENTIFIER>_<FIELD>` first, then `OK_KEYCLOAK_<FIELD>`.

Every value from every layer runs through `Service/EnvironmentValueResolver`, which understands
`%env(NAME)%` and `%env(NAME:-default)%`. An unresolvable placeholder becomes an empty string **on
purpose**, so the next layer wins instead of a literal `%env(...)%` reaching Keycloak.

**Why site sets matter here**: site settings are readable from the `site` request attribute. The
OAuth callback is handled in a middleware that runs *before* TypoScript exists, so TypoScript
could never serve as a runtime layer. With the site set, the controller and the middleware resolve
configuration through exactly the same path.

`getConfigurationWithSources()` returns `value` + `source` per field and backs the module's Status
tab.

**Confidential vs public clients**: `ConfigurationService::BOOLEAN_FIELDS` adds `publicClient`
(`OK_KEYCLOAK_PUBLIC_CLIENT`, `okKeycloak.publicClient`, column `public_client`), resolved through
the same layers but as a boolean — with one difference: the database layer holds a real bool, so an
unchecked box is authoritative instead of falling through the way an empty string does. A public
client has no secret, so `findMissingFields()` (the replacement for the old all-or-nothing
`isComplete()`) stops requiring one, `KeycloakOAuthService` omits `client_secret` from the token
exchange entirely, and the repository discards a stored secret when the flag is saved. PKCE `S256`
is always sent and is what secures the exchange in that case.

**Nothing fails silently**: a configuration that is missing a required field used to be dropped
without a trace, which is indistinguishable from a broken login screen. `findMissingFields()` names
the fields; the login provider logs them, and the module's Backend tab renders them per row.

### Authentication flow

1. `LoginController` (frontend) or `KeycloakLoginProvider` (backend) builds the authorize URL via
   `KeycloakOAuthService::buildAuthorizeUrl()`, with PKCE `S256` and an HMAC-signed `state`
   carrying `type`, `returnUrl`, `siteRootPageId`, `configUid`, `nonce` and `exp`.

   **The signing key is domain separated** — `hash_hmac('sha256', 'ok_keycloak/oauth-state',
   $encryptionKey)`, not the encryption key itself. `ok_azure_login`, which this extension was
   modelled on, signs an identically shaped payload with the same algorithm keyed on the plain
   encryption key. With the same key the two states are interchangeable, both callback middlewares
   match on `code` + `state` alone, and whichever runs first (Azure did) accepts the other's
   callback and fails the exchange against the wrong provider — the symptom is a redirect to
   `/typo3?azure_login_error=exchange_failed` after a successful Keycloak login. Any further OAuth
   extension added here has to pick its own domain string.
2. Keycloak redirects back with `code` + `state`.
3. `Middleware/KeycloakOAuthMiddleware` (registered before the authentication middleware in both
   stacks) validates the state, **recomputes the PKCE verifier** as
   `hash_hmac('sha256', 'pkce:' . nonce, encryptionKey)` — this is why the flow needs no session —
   exchanges the code at the token endpoint, reads `/userinfo`, and publishes the claims as the
   `keycloak_user` request attribute (plus `keycloak_configuration`).
4. It injects the login trigger fields (`login_status`/`username`/`userident` for BE,
   `logintype`/`user`/`pass` for FE) and updates `$GLOBALS['TYPO3_REQUEST']`.
5. `EventListener/KeycloakRequestTokenListener` supplies a valid `RequestToken` so CSRF protection
   does not reject a login that originated at the identity provider.
6. `Authentication/KeycloakAuthService` looks the user up by the configured claim, optionally
   provisions one, and returns 200.
7. The middleware redirects to the return URL, **carrying the `Set-Cookie` headers** of the auth
   chain and downgrading `SameSite=Strict` to `Lax` — a Strict cookie would not survive the
   cross-site hop through Keycloak.

**Failure has to be routed back by hand.** The middleware replaces the auth chain's response with
a redirect, so core's own "login failed" flash message dies with it. Every failure exit therefore
goes through `buildErrorUrl()`, which for backend logins targets `/typo3/login` (plain `/typo3`
makes `BackendUserAuthenticator` redirect an anonymous request itself and drops the query string)
and **pins `loginProvider=keycloak-login`**. The pin is not optional: the backend remembers the
provider you picked only in `be_lastLoginProvider`, a `SameSite=Strict` cookie the browser
withholds after the cross-site hop, so the screen otherwise falls back to the primary provider -
and `ok_azure_login` registers at the same `sorting => 75`, so which one that is comes down to
package order. The wrong template then renders and swallows the error. `KeycloakLoginProvider`
reads `keycloak_error` back off the query and the template maps it to a message.

### Endpoint resolution

`Service/KeycloakEndpointService` derives the endpoints from `{serverUrl}/realms/{realm}/protocol/
openid-connect/*` and tries `.well-known/openid-configuration` first, cached in the `hash` cache
for an hour. Discovery failures are cached as `unavailable` and fall back silently.

### Secret blinding

`Service/SecretBlindingService` masks a value and dispatches
`Event/BeforeSecretIsBlindedEvent`, so listeners can change the masking. `ViewHelpers/
BlindSecretViewHelper` is the only path through which a secret may reach the module's markup.
Secret input fields are write-only, and cloning copies the *encrypted* value server-side.

### User provisioning

`Service/UserProvisioningService` is off by default and configured per site **in the module only**
— never through the environment or site settings, because it decides who gets access. Created
users get a random password hash; created backend users never get `admin = 1`. Group lists are
filtered to positive integers before they are stored.

The lookup column follows the claim: `email` matches `email`, anything else matches `username`.

### Backend module

`web_okkeycloak` at `/module/web/keycloak`, admin only, page-tree navigation. Routes:
`_default` (frontend edit), `save`, `backendList`, `backendEdit`, `backendSave`, `backendDelete`,
`status`. Table `tx_okkeycloak_configuration` is plain QueryBuilder — no Extbase model, no TCA.

### Backend login screen

`ext_localconf.php` unsets the core provider `1433416747` and registers `KeycloakLoginProvider`,
because the login screen renders one provider at a time and the template has to carry the Keycloak
buttons *and* the username/password form.

v13 still declares `render()` in `LoginProviderInterface` but calls `modifyView()` when it exists,
so both are implemented; `render()` throws, exactly as the core provider does. `modifyView()` has
to reach through the `@internal` `FluidViewAdapter` to extend the template paths — the generic
`ViewInterface` offers no way to do it. That call is guarded, and falls back to the core template
rather than locking anyone out of the backend.

## Registration points

| File | Purpose |
|---|---|
| `ext_localconf.php` | plugins, `addService()` auth service, login provider |
| `Configuration/Sets/Keycloak/` | site set: `config.yaml`, `settings.definitions.yaml`, `setup.typoscript`, `page.tsconfig` |
| `Configuration/Services.yaml` | DI; `KeycloakLoginProvider`, `KeycloakAuthService` and the ViewHelpers must stay **public** |
| `Configuration/Backend/Modules.php` | module and its routes |
| `Configuration/Backend/Routes.php` | public `/typo3/keycloak/callback` |
| `Configuration/RequestMiddlewares.php` | the OAuth middleware in both stacks |
| `Configuration/Icons.php`, `JavaScriptModules.php` | icons, JS import map |
| `Configuration/TCA/Overrides/tt_content.php` | the two content elements |

**Why those services are public**: `GeneralUtility::makeInstance()` and Fluid's ViewHelperResolver
both go through `$container->has()`, which is false for private Symfony services — a private
service would be constructed without its dependencies.

## Conventions

- Bilingual XLIFF (EN + `de.`) for all four language files; keys must stay in sync
- JavaScript is native ES modules under `Resources/Public/JavaScript/`, no build step
- One `clone-config.js` serves both edit forms; the target redirect field comes from
  `data-clone-target-redirect`
- `PublicClientToggle.html` carries the public-client switch *and* the client secret field for
  both edit forms, so the two always stay adjacent; `public-client-toggle.js` hides and disables
  the secret field while the switch is on (disabled, so a stray value is never submitted)
- Fluid partials under `Resources/Private/Partials/Backend/` carry the tabs, warnings, status
  table and group select
- Selection state for multi-selects is computed in PHP, not in Fluid
- `Resources/Private/Partials/.gitkeep` exists because the site set declares a `partialRootPaths`
  entry for that directory
- `tx_okkeycloak_configuration` has no TCA, so its records are invisible to the List module —
  inspect them with `ddev mysql`

## Parent repository

See `/home/oliver/typo3-13` for the DDEV setup and the sibling extensions. `ok_azure_login`
(in the TYPO3 12 project) is the Microsoft Entra equivalent this extension was modelled on.

---
> Source: [oliverkroener/ok_keycloak](https://github.com/oliverkroener/ok_keycloak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
