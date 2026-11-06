# Test report: Studium (2026-11-07)

**Scope:** Server, web client (server-rendered views + JavaScript), documentation. Mobile does not exist (phase 07 skipped).
**Branch and commit:** `refactor/quality-notifications-reviews` from `main` @ `5eb70a0`
**Requirements sources:** user requests in the build session, `phases/`, `use-cases/`, `workflows/`, `security/`

## Builds and tests
| Part | Command | Result |
| --- | --- | --- |
| PHP lint | `php -l` on every `.php` file | Passed |
| JS syntax | `node --check` on every module | Passed |
| Migrations | `C:\xampp\php\php.exe bin/migrate.php` | 007, 008 applied |
| Unit tests (new) | `C:\xampp\php\php.exe tests/run.php` | 25 passed, 0 failed, 67 assertions |
| API suite (new) | `bash tests/http/api-suite.sh` | 83 passed, 0 failed |
| Booking/payment regression | curl smoke script | Expected statuses; error log empty |
| Browser | In-app browser: register page, bell, inbox, review post, guest locks, dark/light | Worked, no console errors from the app |

## Server and web client

### 1. Logic
| Severity | Location | Finding | Fix |
| --- | --- | --- | --- |
| High | `services/Reservations/ReservationService.php` (`changeStatus`, `confirm`, `reschedule`) | Status changes checked the transition without locking the row: a confirm and a cancel at the same moment could both succeed | Lock the reservation row, reload, and re-check the transition inside the transaction |
| Medium | `repositories/Reservations/ReservationRepository.php::expireStalePending` | Expiry recorded whichever user's request ran the sweep as `updated_by` | System update with `updated_by = NULL`; rows locked so two sweeps cannot expire the same booking twice |
| Medium | `assets/javascripts/forms/ajax-form.js` | A redirect to the same page with only a `#hash` (after posting a review) did not reload, leaving a stale form and spinner | `core/navigation.js::navigate()` reloads when only the hash changes; used by forms, action buttons, booking |

### 2. Documentation
| Severity | Location | Finding | Fix |
| --- | --- | --- | --- |
| Medium | docs repository | Notifications, reviews, case-insensitive uniqueness, guest locks, register `next`, steel palette, and tests were undocumented | Models, enums, constants, globals, services, controllers, use cases, security, client, migrations, setup, phases updated |
| Low | `server/globals/README.md` | Listed `RequireRole` middleware that no route used | Removed with the middleware |

### 3. Errors
| Severity | Location | Finding | Fix |
| --- | --- | --- | --- |
| — | all endpoints | 401, 403 (`FORBIDDEN`, `CSRF_MISMATCH`, `REVIEW_NOT_ALLOWED`), 404, 409, 422, 429 return the agreed shape; no stack traces or file paths in bodies (checked in the API suite) | OK |

### 4. Inconsistent logic or code
| Severity | Location | Finding | Fix |
| --- | --- | --- | --- |
| Medium | `ui/notification-bell.js`, `reviews/reviews-list.js` | Built API URLs from server-rendered data attributes while other modules used `config/endpoints.js`; reviews list bypassed the shared HTTP client | All JS API URLs come from `config/endpoints.js`; `core/http.js` gained `withMeta` and `keepalive` |
| Medium | `reviews/reviews-list.js` vs `views/partials/reviews/item.php` | "Show more" rendered reviews without the moderator's Remove button | Same markup including the Remove button |
| Low | `services/Reviews` | `ReviewEligibility` enum lived in the services layer | Moved to `enums/Reviews/` |

### 5. Clean code
| Severity | Location | Finding | Fix |
| --- | --- | --- | --- |
| Medium | 205 files | Mixed CRLF and LF line endings | Normalized to LF; `.gitattributes` enforces it (both repositories) |
| Low | `format.js`, `wheel-picker.js`, `ReviewService`, `ReviewRepository`, `middlewares/Auth/RequireRole.php`, `endpoints.js` | Dead code: `money`, `durationLabel`, `setPickerValue`, `findOwn`, `findVisibleForCustomer`, `RequireRole`, unused endpoints, unused `BookingUnit` import | Deleted |
| Low | `views/partials/reservations/table.php` | View created a service to read a setting | Controllers pass `cancelCutoffHours` |
| Low | `views/partials/forms/genericField.php`, `views/reservations/show.php` | Fully qualified class names inline | `use` imports |
| Low | `assets/css/components/buttons.css` | Hover shine was a transition, so it swept backwards on mouse-out; hover lift also applied on touch devices | One-shot keyframe shine, gradient glide, `(hover: hover)` media query |

### 6. Requirements met
| Requirement | Source | Status | Proof |
| --- | --- | --- | --- |
| Reserve / rent stadiums by hour or day | User request | Met | `services/Reservations/*`, API suite |
| Booking locked for guests with "You must register first" | User request | Met | `views/stadiums/show.php`, API suite "guest sees booking lock" |
| Steel-blue palette instead of teal/green | User request | Met | `assets/css/base/tokens.css` |
| Navbar and Back button on register / sign-in | User request | Met | `views/layouts/auth.php`, `partials/layout/guestNav.php` |
| Code, username, email, phone unique, case-insensitive (regex) | User request | Met | `utilities/Formatting/IdentityNormalizer.php`, `RegexPatterns::PHONE_SEPARATORS`, API suite |
| Notifications like e_clinic (bell, unread badge, inbox, kinds) | User request | Met | `services/Notifications`, `ui/notification-bell.js` |
| Comments and rating; hidden before sign-in; only after renting | User request | Met | `services/Reviews/ReviewService.php::eligibility`, API suite |
| Rate each stadium once; hide the form; exception if forced | User request | Met | `REVIEW_ALREADY_EXISTS` + `UNIQUE(stadium_id, customer_id)` |
| Test as the skill says and fix | User request | Met | This report |

### 7. Optimization
| Item | Location | Before | After | Change |
| --- | --- | --- | --- | --- |
| Page queries (manager) | `/dashboard`, `/reservations`, `/stadiums`, `/stadiums/{code}`, `/accounting/reports`, `/notifications` | — | 12, 7, 5, 6, 8, 4 queries; 32–55 ms | Measured: constant per page, no N+1 |
| Stadium rating in lists | `stadiums.rating_average/count` | — | 0 extra queries per card | Denormalized, recomputed on write |
| Bell polling | `ui/notification-bell.js` | — | 1 indexed `COUNT` per minute, paused while hidden | Index `(user_id, read_at, created_at)` |
| CSS delivery | `views/partials/layout/head.php` | 10 stylesheet requests | not changed | Low: acceptable locally; bundle before production |

### 8. Phases
| Phase | Recorded status | Actual | Finding |
| --- | --- | --- | --- |
| 08 Quality | In progress | Done | Updated with verification |
| — | — | Notifications, reviews, UI refinements belonged to no phase | Added phase 10 |
| 01–08 | Done | Delivered in one pull request (#1) instead of one per phase | Recorded; this batch ships as its own pull request |
| 09 Documentation and release | In progress | In progress | Release verification not run |

### 9. Roles and permissions (API suite, signed in as each role)
| Endpoint or screen | Role | Expected | Actual |
| --- | --- | --- | --- |
| `/settings`, `/users`, `/cities`, `/manage/stadiums`, `/accounting/*` | customer | 403 | 403 |
| `/settings`, `/users`, `/cities`, `/manage/stadiums/create` | manager | 403 | 403 |
| `POST /api/reservations` | manager | 403 | 403 |
| `POST /api/stadiums/{code}/reviews` | manager / customer who never played | 403 | 403 |
| `GET /api/stadiums/{code}/reviews` | guest | 401 | 401 |
| `GET /reservations/{other customer's code}` | customer | 404 | 404 |
| `POST /api/reservations/{other}/cancel` | customer | 404 | 404 |
| Edit / toggle another manager's stadium | manager | 404 | 404 |
| Confirm / pay on another manager's booking | manager | 404 | 404 |
| Mark another user's notification read | customer | 404 | 404 |
| Remove a review of another manager's stadium | manager | 404 | 404 |
| Register with `role: admin`; profile update with `role` | customer | stays customer | stays customer |

### 10. Exceptions exist
| Service function | Failure case | Expected exception | Actual | Documented | Tested |
| --- | --- | --- | --- | --- | --- |
| `ReviewService::create` | Already rated (incl. removed) / race | `ConflictException` 409 `REVIEW_ALREADY_EXISTS` | Thrown | Yes | API suite |
| `ReviewService::create` | Never played / not a customer | `AppException` 403 `REVIEW_NOT_ALLOWED` | Thrown | Yes | API suite |
| `ReviewService::delete` | Not moderator of the stadium | `NotFoundException` 404 | Thrown | Yes | API suite |
| `NotificationService::markRead` | Not yours / missing | `NotFoundException` 404 | Thrown | Yes | API suite |
| `NotificationService::send` | Insert fails | Logged, booking kept | Logged via `Logger` (not swallowed) | Yes | — |
| `ReservationService::*` status changes | Concurrent change | `ConflictException` 409 `INVALID_TRANSITION` | Thrown under lock | Yes | Logic reviewed; unit tests for transitions |
| `UserService::assertUnique` | Case/format duplicate | `ConflictException` 409 `DUPLICATE_VALUE` | Thrown | Yes | API suite + DB constraint check |

## Summary
| Check | Server | Web | Mobile | Docs |
| --- | --- | --- | --- | --- |
| Logic | 1 high, 1 medium fixed | 1 medium fixed | — | — |
| Documentation | — | — | — | 1 medium, 1 low fixed |
| Errors | OK | OK | — | — |
| Inconsistent | 1 low fixed | 2 medium fixed | — | — |
| Clean code | 3 low fixed | 1 medium, 2 low fixed | — | 1 medium fixed |
| Requirements | All met | All met | — | — |
| Optimization | OK (measured) | 1 low open (CSS bundling) | — | — |
| Phases | — | — | — | 1 medium fixed, 1 noted |
| Roles and permissions | OK | OK | — | — |
| Exceptions exist | OK | — | — | OK |

**Overall:** Ready for review. Open items: release verification (phase 09) and CSS bundling before production.
