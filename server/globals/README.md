# Globals: structure, config, middlewares, utilities, conventions

## Folder layout (layer-first)

```text
index.php                 front controller → config/App/Kernel.php
.htaccess                 assets/ served directly; everything else → index.php; dotfiles denied
bin/                      migrate.php, seed.php (CLI only)
config/      App/ (bootstrap, Env, AppConfig, Kernel)  Database/  Routes/ (routes.php, middleware.php)
constants/   Common/ Users/ Stadiums/ Reservations/ Accounting/
controllers/ Base/ Home/ Users/ Regional/ Stadiums/ Reservations/ Accounting/ Settings/
dtos/        Common/ Users/ Regional/ Stadiums/ Reservations/ Accounting/ Settings/
enums/       Common/ Users/ Stadiums/ Reservations/ Accounting/ Settings/
exceptions/  Common/
globals/     Auth/ (CurrentUser)
middlewares/ Auth/ Security/ Errors/
migrations/  001…006 *.sql (flat, ordered)
models/      Base/ Users/ Regional/ Stadiums/ Reservations/ Accounting/ Settings/
repositories/ Base/ Users/ Regional/ Stadiums/ Reservations/ Accounting/ Settings/
seeds/       Users/ Regional/ Stadiums/ Settings/
services/    Common/ Users/ Regional/ Stadiums/ Reservations/ Accounting/ Settings/
utilities/   Http/ Responses/ Security/ Views/ Pagination/ Codes/ Dates/ Formatting/
validations/ Common/ Users/
views/       layouts/ partials/ (ui, forms, filters, layout, reservations) + one folder per page group
assets/      css/ (base, layout, components, pages)  javascripts/ (core, config, forms, ui, stadiums, reservations)
storage/logs/
tests/       run.php (unit runner)  Unit/<Layer>/…Test.php  http/api-suite.sh
```

Notifications and reviews follow the same layout: `models/Notifications`, `repositories/Notifications`, `services/Notifications`, `controllers/Notifications`, `enums/Notifications`, `constants/Notifications`, and the same for `Reviews`.

All text files use LF line endings (`.gitattributes`).

The autoloader maps `App\<Layer>\<Group>\<Class>` to `<layer>/<Group>/<Class>.php` (`config/App/bootstrap.php`).

## Request pipeline — `config/App/Kernel.php`

1. `Session::start()` — `HttpOnly`, `SameSite=Lax`, `Secure` on HTTPS, strict mode.
2. `SecurityHeaders` — CSP with a per-request nonce, `X-Frame-Options: DENY`, `nosniff`, referrer and permissions policies.
3. `AuthService::resolveSessionUser()` → `CurrentUser` (used by repositories for audit fields and by views).
4. `VerifyCsrf` — every non-GET request.
5. `Router::dispatch()` — runs route middleware (`auth`, `guest`, `role:…`, `can:…`) then the controller.
6. Any exception → `ErrorHandler` (JSON for `/api/*`, error page otherwise, 401 → sign-in redirect). Unexpected errors are logged to `storage/logs/app.log` without request bodies.

## Config — `config/`

| File | Purpose |
| --- | --- |
| `App/Env.php` | Reads `.env` once; typed `string/int/float/bool` getters |
| `App/AppConfig.php` | App name, environment, debug, timezone, currency, base path (auto-detected), session idle minutes, map defaults |
| `Database/Database.php` | One PDO per request (exceptions, assoc fetch, native prepares, utf8mb4, PHP timezone); `transaction(callable)` |
| `Routes/routes.php` | Every route; mirrored by `assets/javascripts/config/endpoints.js` |
| `Routes/middleware.php` | Middleware aliases |

## Middlewares — `middlewares/`

| Middleware | Alias | Behaviour |
| --- | --- | --- |
| `Auth/Authenticate` | `auth` | 401 when signed out (pages redirect to `/login?next=`) |
| `Auth/RedirectIfAuthenticated` | `guest` | Signed-in users skip sign-in pages |
| `Auth/RequirePermission` | `can:reservations.manage` | Permission check via `PermissionService` → 403 |
| `Security/VerifyCsrf` | global | 403 `CSRF_MISMATCH` |
| `Security/SecurityHeaders` | global | Security headers, CSP nonce |
| `Errors/ErrorHandler` | global | Exception → response |

## Utilities — `utilities/`

| Utility | Purpose |
| --- | --- |
| `Http/Request`, `Http/Router`, `Http/Redirect` | Request parsing (JSON or form), routing with `{code}` params, redirects |
| `Responses/ApiResponse` | The one JSON shape |
| `Security/Session`, `Security/Csrf`, `Security/PasswordHasher` | Session, CSRF token, bcrypt + strength rule |
| `Pagination/Paginator` | The one pagination utility (count + `LIMIT/OFFSET` in SQL, sort allowlist) |
| `Codes/CodeGenerator` | `PREFIX-XXXXXXXX` codes |
| `Dates/OpeningWindow` | Opening window of a day, including windows that cross midnight |
| `Formatting/Money`, `Formatting/AddressFormatter` | 3-decimal money; address line and Google Maps URLs |
| `Formatting/IdentityNormalizer` | Regex-based canonical forms: lower-case username/email, phone without separators, upper-case codes, collapsed whitespace |
| `Logging/Logger` | One line per event to `storage/logs/app.log` (used by the error handler and failed notification inserts) |
| `Views/View`, `Views/helpers.php` | Rendering and helpers: `e`, `url`, `asset`, `partial`, `icon`, `button`, `badge`, `can`, `attrs`, `money`, `format_date` |

Base persistence: `repositories/Base/BaseRepository.php` generates codes (retrying on a unique collision), fills `created_*`/`updated_*` from `CurrentUser`, soft-deletes, and exposes prepared-statement helpers. Validation: `validations/Common/Validator.php` (fluent rules, all errors at once) and `QueryFilters` (lenient list filters).

## Response shape

```json
{ "success": true, "data": { "message": "Booking confirmed.", "redirect": null }, "meta": { "page": 1 } }
{ "success": false, "error": { "code": "VALIDATION_FAILED", "message": "Please fix the highlighted fields.", "details": { "fields": { "email": "Enter a valid email address." } } } }
```

## Conventions

- Routes have no version segment (`/api/...`).
- Public URLs use `code`, never `id`.
- Every list goes through `Paginator`; every mutation through a DTO; every SQL through prepared statements.
- One class per file; files named after the class in PascalCase inside lowercase layer folders.
