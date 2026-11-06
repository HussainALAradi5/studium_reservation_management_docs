# Phase 06: Client screens

**Status:** Done
**Depends on:** Phase 05

## Goal
Every use case works in the browser.

## Scope
| Layer | Folders | Items |
| --- | --- | --- |
| Pages | `views/<group>/` | Landing, auth, dashboard, stadiums (browse, details + booking, manage, form + map), reservations, accounting, users, cities, settings, profile |

## Tasks
- [x] Booking widget with hourly slots and full-day calendar
- [x] Cancel / reject with reason dialogs
- [x] Payment dialog
- [x] Permission matrix

## Acceptance criteria
- Each use case works end to end in the browser

## Verification
| Check | Command | Result |
| --- | --- | --- |
| Browser check | Stadium page: slot selection, quote, full-day overlap detection | Worked, 2026-11-07 |
| Mobile width | 375 px dashboard | No horizontal scroll, 2026-11-07 |
