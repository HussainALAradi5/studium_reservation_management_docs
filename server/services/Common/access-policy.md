# AccessPolicy

`services/Common/AccessPolicy.php` — pure permission expressions (no database). Route permissions (`can:`) decide **which actions** a role may call; `AccessPolicy` decides **which records** a user may touch.

### `scopeFor(User $user): AccessScope`
**Purpose:** Row scope for list queries. **Logic:** admin → `AccessScope::all()`; manager → `AccessScope::manager(id)` (stadiums they manage); customer → `AccessScope::customer(id)` (their bookings). Repositories turn it into SQL via `AccessScope::toSql()`.

### `canManageStadium(User $user, ?int $stadiumManagerId): bool`
`isAdmin OR (isManager AND stadiumManagerId == user.id)`

### `canManageReservation(User $user, Reservation $reservation): bool`
`canManageStadium(user, reservation.stadiumManagerId)`

### `canViewReservation(User $user, Reservation $reservation): bool`
`canManageReservation OR (isCustomer AND reservation.customerId == user.id)`

### `isReservationOwner(User $user, Reservation $reservation): bool`
`isCustomer AND reservation.customerId == user.id`

### `canCancelReservation(User $user, Reservation $reservation, int $cutoffHours): bool`
`canManageReservation OR (owner AND (status == pending OR (status == confirmed AND minutesUntilStart >= cutoffHours × 60)))`. `$cutoffHours` comes from the `reservation.cancel_cutoff_hours` setting.

### `canRescheduleReservation(User $user, Reservation $reservation): bool`
`canManageReservation OR (owner AND status == pending)`

### `canEditStadium(User $user, Stadium $stadium): bool`
`canManageStadium(user, stadium.managerId)`
