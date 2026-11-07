# Phase 09: Documentation and release

**Status:** Done
**Depends on:** Phase 08

## Goal
Complete docs, verify the release, and put the site online.

## Scope
| Layer | Folders | Items |
| --- | --- | --- |
| Docs | this repository | Models, services, controllers, use cases, setup, deployment, test report |
| App | `README.md`, `screenshots/`, `bin/install.php`, `bin/export-sql.php`, `bin/build-assets.php` | Production install, CSS bundle |

## Tasks
- [x] Screenshots section (app README)
- [x] Release verification in production mode
- [x] Production install and phpMyAdmin SQL export
- [x] Host: InfinityFree, https://studium-bh.infinityfreeapp.com
- [x] Deploy and smoke-test the live site

## Acceptance criteria
- Docs match the code; release checks pass; the site answers on its public URL

## Verification
| Check | Command | Result |
| --- | --- | --- |
| Docs vs code | Written from the implemented signatures | Done 2026-11-07 |
| Release verification | See `setup/server/deploy.md` | Passed locally in production mode, 2026-11-07 |
| Live deployment | Browser checks on https://studium-bh.infinityfreeapp.com | Pages 200, protected paths blocked, HTTPS + HSTS + CSP active, 2026-11-07 |
