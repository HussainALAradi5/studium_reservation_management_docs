# Phase 02: Foundation

**Status:** Done
**Depends on:** Phase 01

## Goal
Shared building blocks: config, constants, enums, base entity, utilities, middlewares.

## Scope
| Layer | Folders | Items |
| --- | --- | --- |
| Config | `config/App`, `config/Database`, `config/Routes` | Env loader, AppConfig, Kernel, PDO, routes |
| Shared | `constants/`, `enums/`, `exceptions/`, `utilities/`, `validations/Common`, `middlewares/` | Error codes, regex patterns, paginator, code generator, validator, CSRF, security headers, error handler |

## Tasks
- [x] Layer-first autoloader
- [x] One response shape
- [x] Regex patterns shared by server and HTML `pattern`
- [x] CSRF and security headers on every request

## Acceptance criteria
- Every page and API returns through the Kernel with the shared error shape

## Verification
| Check | Command | Result |
| --- | --- | --- |
| PHP lint (all files) | `php -l` over every `.php` | No syntax errors, 2026-11-07 |
