# Phase 10: Notifications, reviews, and UI refinements

**Status:** Done
**Depends on:** Phase 06

## Goal
In-app notifications (bell + inbox), stadium reviews (rate once after playing, members-only comments), case-insensitive uniqueness, locked booking for guests, steel-blue palette, navbar on auth pages.

## Scope
| Layer | Folders | Items |
| --- | --- | --- |
| Server | `models|repositories|services|controllers/{Notifications,Reviews}`, migrations 007–008 | NotificationService, ReviewService |
| Client | `ui/notification-bell.js`, `reviews/reviews-list.js`, partials | Bell, inbox, stars, rating input, locks |

## Tasks
- [x] Notifications for every booking, payment, and review event
- [x] One review per customer and stadium, enforced in service and database
- [x] IdentityNormalizer for username, email, phone, and codes
- [x] Guest locks for booking and reviews

## Acceptance criteria
- Use cases UC-NTF-01..03 and UC-REV-01..03 work end to end

## Verification
| Check | Command | Result |
| --- | --- | --- |
| API suite (notifications, reviews, uniqueness) | `bash tests/http/api-suite.sh` | Passed, 2026-11-07 |
| Browser | Bell dropdown, review post, guest locks | Worked, 2026-11-07 |
