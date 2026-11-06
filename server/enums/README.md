# Enums

PHP 8.1 backed enums in `enums/<Group>/`. Each enum has `label()` for display and `values()` for validation; database `ENUM` columns use the same string values.

## UserRole
`enums/Users/UserRole.php`

| Value | Meaning |
| --- | --- |
| `admin` | Full access; manages users, cities, all stadiums |
| `manager` | Runs assigned stadiums, their reservations and accounting |
| `customer` | Books stadiums |

## Gender
`enums/Users/Gender.php` — `male`, `female`.

## SportType
`enums/Stadiums/SportType.php` — `football`, `futsal`, `basketball`, `volleyball`, `tennis`, `padel`, `multi_sport`.

## SurfaceType
`enums/Stadiums/SurfaceType.php` — `natural_grass`, `artificial_turf`, `hard_court`, `wooden`, `sand`.

## ReservationStatus
`enums/Reservations/ReservationStatus.php`

| Value | Blocks the slot | Terminal | Meaning |
| --- | --- | --- | --- |
| `pending` | yes | no | Requested by a customer, waiting for the manager |
| `confirmed` | yes | no | Accepted by manager/admin |
| `completed` | — | yes | Played (allowed only after `end_at`) |
| `rejected` | no | yes | Declined by manager/admin |
| `cancelled` | no | yes | Cancelled by customer/manager/admin, or expired while pending |

## BookingUnit
`enums/Reservations/BookingUnit.php` — `hour` (Hourly), `day` (Full day). `describe(count)` renders "1.5 hours" or "3 days".

## Permission
`enums/Users/Permission.php` — checked by the `can:` route middleware; grants are stored in `role_permissions`.

| Value | Default roles |
| --- | --- |
| `reservations.book` | customer |
| `reservations.manage` | manager |
| `payments.record` | manager |
| `stadiums.manage` | manager |
| `stadiums.create` | — |
| `expenses.manage` | manager |
| `reports.view` | manager |
| `cities.manage` | — |
| `users.manage` | — |
| `settings.manage` | — |
| `reviews.write` | customer |
| `reviews.moderate` | manager |

Admins always hold every permission (not editable, so nobody gets locked out).

## SettingsKey
`enums/Settings/SettingsKey.php` — each key carries its default, range, label, and hint.

| Key | Default | Range | Meaning |
| --- | --- | --- | --- |
| `reservation.auto_confirm` | 0 | 0–1 | Confirm free bookings instantly |
| `reservation.min_lead_minutes` | 15 | 0–1440 | Minimum notice before start |
| `reservation.max_advance_days` | 60 | 1–365 | How far ahead customers can book |
| `reservation.max_hours` | 4 | 1–24 | Longest hourly booking |
| `reservation.max_days` | 7 | 1–30 | Longest daily booking |
| `reservation.cancel_cutoff_hours` | 2 | 0–168 | Customer cancel cutoff for confirmed bookings |
| `reservation.suggestion_limit` | 5 | 1–10 | Alternatives offered on conflict |

## NotificationTemplate
`enums/Notifications/NotificationTemplate.php` — every in-app message: title, body pattern with named placeholders (`:stadium`, `:when`, `:customer`, `:amount`, `:reason`, `:rating`), and kind.

| Template | Recipient | Kind |
| --- | --- | --- |
| `reservation.requested` / `reservation.booked` (auto-confirm) | stadium manager (or all admins) | booking |
| `reservation.confirmed` | customer | booking |
| `reservation.rejected` | customer | alert |
| `reservation.cancelled` | the other party | alert |
| `reservation.rescheduled` | the other party | booking |
| `reservation.completed` | customer (invites a review) | booking |
| `reservation.expired` | customer | alert |
| `payment.recorded` / `payment.refunded` | customer | payment |
| `review.created` | stadium manager (or all admins) | review |

`body()` fills only the pattern's own placeholders, so text inside a value (e.g. a cancel reason containing `:late`) is never treated as a placeholder.

## NotificationKind
`enums/Notifications/NotificationKind.php` — `booking`, `payment`, `review`, `alert`; each has an icon and a color tone.

## ReviewEligibility
`enums/Reviews/ReviewEligibility.php` — `can_review`, `already_reviewed`, `not_played`, `not_a_customer`. The review form is rendered only for `can_review`; the other cases show their message.

## PaymentStatus
`enums/Reservations/PaymentStatus.php` — `unpaid`, `partial`, `paid`, `refunded`. Derived from `paid_amount` and `total_amount`; see [accounting](../../workflows/accounting.md).

## TransactionType
`enums/Accounting/TransactionType.php` — `payment`, `refund`.

## PaymentMethod
`enums/Accounting/PaymentMethod.php` — `cash`, `card`, `bank_transfer`, `online`.

## ExpenseCategory
`enums/Accounting/ExpenseCategory.php` — `maintenance`, `utilities`, `salaries`, `equipment`, `rent`, `marketing`, `other`.
