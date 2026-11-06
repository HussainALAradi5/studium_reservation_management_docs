# Reservations services

See [conflict resolution](../../../workflows/conflict-resolution.md) and the [lifecycle](../../../workflows/reservation-lifecycle.md) for the rules these implement.

## ConflictService — `services/Reservations/ConflictService.php`

### `resolve(Stadium $stadium, BookingRequestDto $request): TimeSlotDto`
**Purpose:** Turns a request into an exact `[startAt, endAt)`.
- Hourly: `date + startTime` → `+durationMinutes`; `unitCount = minutes / 60` (may be 1.5).
- Daily: opening of the first day's window → closing of the last day's window (a stadium open 08:00–02:00 rented 2 days from 10 Nov = 10 Nov 08:00 → 12 Nov 02:00); `unitCount = days`.
**Exceptions:** 422 on `bookingUnit` when the stadium has no `price_per_day`.

### `validateSlot(Stadium $stadium, TimeSlotDto $slot): void`
| Check | Error |
| --- | --- |
| Start < now + `reservation.min_lead_minutes` | 422 `RESERVATION_IN_PAST` |
| Start > now + `reservation.max_advance_days` | 422 on `date` |
| Hourly: minutes not a multiple of `slot_minutes` | 422 on `durationMinutes` |
| Hourly: minutes > `reservation.max_hours` × 60 | 422 on `durationMinutes` |
| Hourly: outside the opening window (incl. after-midnight windows) | 422 `OUTSIDE_OPENING_HOURS` |
| Hourly: start not on the slot grid | 422 on `startTime` |
| Daily: stadium hourly-only, or days > `reservation.max_days` | 422 |

### `assertNoConflict(Stadium $stadium, TimeSlotDto $slot, ?int $excludeReservationId = null): void`
Must run inside a transaction after `StadiumRepository::lockById()`. Throws 409 `RESERVATION_CONFLICT` with `details.conflicts` (ranges only, no customer data) and `details.suggestions`.

### `priceFor(TimeSlotDto $slot, float $unitPrice): float` / `unitPriceFor(Stadium $stadium, BookingUnit $unit): float`
`total = unitPrice × unitCount`, rounded to 3 decimals; unit price is `price_per_hour` or `price_per_day`.

### `quote(Stadium $stadium, BookingRequestDto $request): array`
Read-only (no lock): resolve + validate + price + conflicts + suggestions. Used by the booking widget so prices always come from the server.

### `availability(Stadium $stadium, \DateTimeImmutable $date): array`
Slot grid of one opening window: `{startAt, endAt, start, end, state: free|booked|past}`. One query for the whole window.

### `calendar(Stadium $stadium, string $month): array`
Each day of the month: `{date, day, weekday, bookedMinutes, openMinutes, state: free|partial|full|past}`. One query for the month.

### `suggestAlternatives(Stadium $stadium, TimeSlotDto $requested, ?int $excludeReservationId = null): array`
Same unit and size. Hourly: same-day grid starts ordered by distance from the requested start, then the same time on the next 7 days. Daily: the next 7 start dates. Every candidate passes `validateSlot` and an in-memory overlap test against bookings loaded once. Stops at `reservation.suggestion_limit`.

### `describeSlot(TimeSlotDto $slot): array`
`{bookingUnit, date, startTime, durationMinutes, days, startAt, endAt, size, label}` — enough for the client to show it and re-submit it.

## ReservationService — `services/Reservations/ReservationService.php`

| Function | Purpose | Exceptions |
| --- | --- | --- |
| `list(User $user, ReservationQuery $query): PageResult` | Expire stale pending, then page within scope | — |
| `listUpcoming(User $user, int $limit): Reservation[]` | Dashboard | — |
| `countByStatus(User $user): array` | Dashboard counters | — |
| `findViewable(User $user, string $code): Reservation` | Scoped lookup | 404 |
| `create(User $customer, CreateReservationDto $dto): Reservation` | Book (below) | 404, 409, 422 |
| `confirm(User $actor, string $code): Reservation` | pending → confirmed with a locked re-check | 404, 409 `INVALID_TRANSITION`, 409 `RESERVATION_CONFLICT` |
| `reject(User $actor, string $code, StatusChangeDto $dto): Reservation` | pending → rejected, reason required | 404, 409, 422 |
| `cancel(User $actor, string $code, StatusChangeDto $dto): Reservation` | pending/confirmed → cancelled | 404, 409 `INVALID_TRANSITION`, 409 `CANCEL_CUTOFF_PASSED` |
| `complete(User $actor, string $code): Reservation` | confirmed → completed after `end_at` | 404, 409 |
| `reschedule(User $actor, string $code, BookingRequestDto $request): Reservation` | Move (may switch hourly ↔ daily) | 404, 409, 422 |
| `expireStalePending(): int` | Pending past start → cancelled "Expired" | — |
| `cancelCutoffHours(): int` | Setting value for views and policy | — |

### `create` in detail
1. Stadium must be active (`findVisible(null, …)`).
2. `ConflictService::resolve` → `validateSlot`.
3. Unit price from the stadium (never from input).
4. Transaction: lock stadium row → `assertNoConflict` → insert with `unit_price`, `total_amount`, `paid_amount = 0`, `payment_status = unpaid`, status `pending` (or `confirmed` when `reservation.auto_confirm` = 1).

### `reschedule` in detail
Allowed for managers/admins, and for the owner while pending. Keeps the original unit price when the unit stays the same; uses the current stadium price when switching unit. Recalculates `total_amount` and `payment_status` from the existing `paid_amount`.
