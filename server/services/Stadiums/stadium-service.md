# StadiumService — `services/Stadiums/StadiumService.php`

### `browse(StadiumQuery $query): PageResult`
Public catalogue: active stadiums in active cities; filters `q` (name, area, street), `city`, `sport`; sort name / price / newest.

### `listManaged(User $user, StadiumQuery $query): PageResult`
Management list within `AccessPolicy::scopeFor($user)` (admins: all, managers: assigned), any status.

### `listForSelect(User $user): Stadium[]`
Stadiums in scope for filters, expense form, and reports.

### `findVisible(?User $user, string $code): Stadium`
Active stadiums for everyone; inactive ones only for whoever manages them. **Exceptions:** 404.

### `findManageable(User $user, string $code): Stadium`
**Exceptions:** 404 when missing or out of scope (no 403, so codes of other stadiums are not confirmed).

### `create(StadiumFormDto $dto): Stadium`
**Purpose:** Admin creates a stadium. **Logic:** 1) active city; 2) manager code must be an active manager (or empty); 3) one transaction inserts the address and the stadium.

| Case | Result |
| --- | --- |
| Valid | Stadium + address created |
| Inactive city | 409 with `fields.cityCode` |
| Manager code is not an active manager | 422 on `managerCode` |
| Latitude without longitude | 422 (DTO) |

### `update(User $actor, string $code, StadiumFormDto $dto): Stadium`
**Logic:** manageable stadium → city (an unchanged inactive city is allowed) → if switching active → inactive, no future bookings → only admins change `manager_id` → address + stadium updated in one transaction. Price changes never touch existing reservations (they keep a price snapshot). **Exceptions:** 404, 409 `STADIUM_HAS_BOOKINGS`, 422.

### `toggleActive(User $actor, string $code): Stadium`
Deactivation requires no pending/confirmed bookings ending in the future. **Exceptions:** 404, 409 `STADIUM_HAS_BOOKINGS` (`details.count`).

### `countActive(): int`
