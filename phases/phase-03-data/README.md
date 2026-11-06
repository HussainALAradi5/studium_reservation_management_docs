# Phase 03: Data

**Status:** Done
**Depends on:** Phase 02

## Goal
Persisted models, migrations, indexes, and the development seed.

## Scope
| Layer | Folders | Items |
| --- | --- | --- |
| Models | `models/<Group>/` | 9 models extending BaseEntity |
| Migrations | `migrations/001…006` | 11 tables |
| Seeds | `seeds/<Group>/`, `bin/seed.php` | Permissions, settings, one user per role, cities, stadiums |

## Tasks
- [x] Foreign keys and indexes for overlap, scope, and reports
- [x] Soft delete columns
- [x] Seed refuses production

## Acceptance criteria
- Migrations run on an empty database; the seed creates every role

## Verification
| Check | Command | Result |
| --- | --- | --- |
| Migrations | `C:\xampp\php\php.exe bin/migrate.php` | 6 migrations applied, 2026-11-07 |
| Seed | `C:\xampp\php\php.exe bin/seed.php` | admin, manager, customer + 6 cities + 3 stadiums, 2026-11-07 |
