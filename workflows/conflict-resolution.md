# Conflict resolution

The goal: two people can never hold the same stadium at the same time, and a person who loses a slot is immediately offered good alternatives.

## 0. Hourly and daily rentals share one rule

A daily rental of N days is stored as one range from the opening time of the first day to the closing time of the last day; an hourly rental is stored as its exact range. Because both are plain `[start_at, end_at)` ranges, one overlap query protects both: a full day blocks every hourly slot inside it, and an hourly booking blocks any full day that contains it.

## 1. What counts as a conflict

Two reservations of the **same stadium** conflict when both are in a blocking status (`pending` or `confirmed`) and their half-open ranges overlap:

```sql
existing.start_at < :new_end_at AND existing.end_at > :new_start_at
```

Back-to-back bookings (`16:00–18:00` and `18:00–19:00`) do not conflict.

## 2. Validation before the overlap check

`ReservationService` rejects a request before checking overlap when:

| Check | Error code | HTTP |
| --- | --- | --- |
| Start is in the past or closer than `reservation.min_lead_minutes` | `RESERVATION_IN_PAST` | 422 |
| Start is more than `reservation.max_advance_days` ahead | `VALIDATION_FAILED` | 422 |
| Hourly: start/duration off the stadium's `slot_minutes` grid, or longer than `reservation.max_hours` | `VALIDATION_FAILED` | 422 |
| Hourly: outside the opening window (including windows that cross midnight) | `OUTSIDE_OPENING_HOURS` | 422 |
| Daily: stadium has no `price_per_day`, or more than `reservation.max_days` days | `VALIDATION_FAILED` | 422 |
| Stadium inactive or soft-deleted | `NOT_FOUND` | 404 |

## 3. Race-safe check (no double booking under concurrency)

Create, confirm, and reschedule run in **one database transaction**:

1. `SELECT id FROM stadiums WHERE id = ? FOR UPDATE` — locks the stadium row, so concurrent bookings of the same stadium run one after another (other stadiums are not blocked).
2. Run the overlap query (excluding the reservation itself on confirm/reschedule).
3. If a conflict exists → roll back and return 409 `RESERVATION_CONFLICT`.
4. Otherwise insert/update and commit.

Without the lock, two requests could both see "free" and both insert. With it, the second request waits, then sees the first booking and gets a conflict.

## 4. Suggestions returned on conflict

The 409 response carries alternatives so the client can offer one-click rebooking:

```json
{
  "success": false,
  "error": {
    "code": "RESERVATION_CONFLICT",
    "message": "This time overlaps another booking.",
    "details": {
      "conflicts": [{ "startAt": "2026-11-10 18:00", "endAt": "2026-11-10 20:00" }],
      "suggestions": [
        { "startAt": "2026-11-10 20:00", "endAt": "2026-11-10 21:00", "label": "Tue 10 Nov, 20:00 – 21:00" }
      ]
    }
  }
}
```

`ConflictService::suggestAlternatives()` searches ranges **with the same unit and size**:

1. Hourly: same day, ordered by distance from the requested start (closest first, earlier or later), then the same time on the following days.
2. Daily: the following start dates.
3. Searches `SUGGESTION_SEARCH_DAYS` (7) days ahead, skips anything `validateSlot` rejects, and stops at `reservation.suggestion_limit` results. Bookings are loaded once and tested in memory.

Conflicting ranges are shown **without** the other customer's identity (privacy).

## 5. Prevention in the client

- `GET /api/stadiums/{code}/availability?date=` — hourly slot grid (`free`, `booked`, `past`); the widget only lets the user pick contiguous free slots.
- `GET /api/stadiums/{code}/calendar?month=` — each day `free`, `partial`, `full`, or `past` for daily rentals.
- `GET /api/stadiums/{code}/quote?...` — server price and availability of the current selection, with suggestions when it overlaps.

Most conflicts never reach the booking request; the locked server check still decides.

## 6. Conflicts caused by managers

| Situation | Handling |
| --- | --- |
| Confirming a pending booking that now overlaps (data edited meanwhile) | Re-check under lock → 409 with suggestions; the manager reschedules or rejects |
| Rescheduling | Same full check, excluding the reservation itself |
| Deactivating a stadium with future bookings | 409 `STADIUM_HAS_BOOKINGS` with the count; cancel or move them first |
| Changing opening hours | Existing bookings stay valid; new bookings follow the new hours |
