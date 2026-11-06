# Security

## Roles

| Role | Created by | Seeded dev user |
| --- | --- | --- |
| `admin` | seed or another admin | `admin` |
| `manager` | admin | `manager` |
| `customer` | self-registration or admin | `customer` |

Seed passwords come from `SEED_PASSWORD` in `.env`; see [setup](../setup/server/README.md). The seed refuses to run when `APP_ENV=production`.

## Two layers

1. **Permissions (which actions):** every protected route declares `can:<permission>` in `config/Routes/routes.php`. `RequirePermission` checks the role's grants in `role_permissions`; a missing grant → 403 (customers cannot reach manager or admin endpoints). Admins edit grants on `/settings`; the admin role always keeps all of them.
2. **Scope (which records):** services call `AccessPolicy` (below). Out-of-scope records → 404.

## Permission matrix (defaults)

`own` = stadiums where `manager_id` is the user, or reservations where `customer_id` is the user.

| Action | admin | manager | customer |
| --- | --- | --- | --- |
| Manage users | all | — | — |
| Manage cities | all | read | read |
| Create stadium, assign manager | all | — | — |
| Edit stadium details/location/hours | all | own stadiums | — |
| Activate / deactivate stadium | all | own stadiums | — |
| Browse active stadiums, availability | yes | yes | yes (also guests) |
| Create reservation | — | — | yes |
| View reservations | all | own stadiums | own |
| Confirm / reject / complete | all | own stadiums | — |
| Cancel | all | own stadiums | own (rules in lifecycle) |
| Reschedule | all | own stadiums | own, pending only |
| Record payment / refund | all | own stadiums | — |
| Expenses | all | own stadiums | — |
| Accounting report | all | own stadiums | — |
| Settings and role permissions | all | — | — |
| Edit own profile / password | yes | yes | yes |

Expressions as implemented in `services/Common/AccessPolicy.php`:

- `canManageStadium(user, stadium) = isAdmin(user) OR (isManager(user) AND stadium.managerId == user.id)`
- `canViewReservation(user, r) = canManageStadium(user, r.stadium) OR (isCustomer(user) AND r.customerId == user.id)`
- `canCancelReservation(user, r) = canManageStadium(user, r.stadium) OR (r.customerId == user.id AND (r.status == pending OR (r.status == confirmed AND hoursUntil(r.startAt) >= CANCEL_CUTOFF_HOURS)))`

Out-of-scope records return **404** (not 403) so codes of other people's records are not confirmed.

## Controls

| Threat | Control | Where |
| --- | --- | --- |
| SQL injection | PDO prepared statements only; sort columns from allowlists | `repositories/` |
| XSS | Every view value escaped with `e()`; JS builds DOM with `textContent` | `utilities/Views/View.php`, `assets/javascripts/core/dom.js` |
| CSRF | Per-session token in `<meta>`; required on every POST/PUT/DELETE (header `X-CSRF-Token` or `_token` field); failure → 403 `CSRF_MISMATCH` | `middlewares/Security/VerifyCsrf.php` |
| Session fixation / hijack | `session_regenerate_id` on login; `HttpOnly`, `SameSite=Lax`, `Secure` when HTTPS; idle timeout | `utilities/Security/Session.php` |
| Brute force | 5 failed logins per username+IP in 15 min → 429 | `services/Users/AuthService.php`, `login_attempts` table |
| Password storage | bcrypt (`password_hash`), min 8 chars with letter + digit | `utilities/Security/PasswordHasher.php` |
| Mass assignment | DTOs copy only known fields; role/price/status never read from customer input | `dtos/` |
| IDOR | Every lookup by code goes through `AccessPolicy`; 404 when out of scope | `services/` |
| Clickjacking, sniffing | `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, CSP | `middlewares/Security/SecurityHeaders.php` |
| Direct file access | `.htaccess` routes everything except `assets/` to `index.php`; blocks dotfiles | `.htaccess` |
| Error leaks | Stack traces only when `APP_DEBUG=true`; generic message otherwise; errors logged | `middlewares/Errors/ErrorHandler.php` |
| Secrets | `.env` ignored by git, `.env.example` committed | repo root |
