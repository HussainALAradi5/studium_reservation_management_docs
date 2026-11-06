# Phase 04: Server logic

**Status:** Done
**Depends on:** Phase 03

## Goal
DTOs, repositories, services, controllers, and permission-guarded routes.

## Scope
| Layer | Folders | Items |
| --- | --- | --- |
| Logic | `dtos/`, `repositories/`, `services/`, `controllers/` | Auth, users, cities, stadiums, reservations (hourly + daily), conflicts, payments, expenses, reports, settings, permissions |

## Tasks
- [x] Locked overlap check with suggestions
- [x] Hourly and daily rentals
- [x] Settings keys
- [x] Role permissions with `can:` middleware
- [x] Ledger with refunds

## Acceptance criteria
- Every endpoint returns the documented DTOs and errors (see `server/controllers/`)

## Verification
| Check | Command | Result |
| --- | --- | --- |
| HTTP smoke test (auth, permissions, booking, conflicts, payments, cancel, settings) | curl script against Apache | All expected statuses, empty error log, 2026-11-07 |
