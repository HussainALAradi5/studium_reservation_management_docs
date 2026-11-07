# Phase 09: Documentation and release

**Status:** In progress
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
- [ ] Choose host and domain (owner)
- [ ] Deploy and smoke-test the live site

## Acceptance criteria
- Docs match the code; release checks pass; the site answers on its public URL

## Verification
| Check | Command | Result |
| --- | --- | --- |
| Docs vs code | Written from the implemented signatures | Done 2026-11-07 |
| Release verification | See `setup/server/deploy.md` | Passed locally in production mode, 2026-11-07 |
| Live deployment | — | Not done (waiting for host choice) |
