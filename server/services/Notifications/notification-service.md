# NotificationService — `services/Notifications/NotificationService.php`

In-app notifications. Business services call the `notify*` helpers **after** their transaction commits; a failed insert is logged (`storage/logs/app.log`) and never undoes the booking or payment that triggered it.

| Function | Purpose | Exceptions |
| --- | --- | --- |
| `list(User $user, NotificationQuery $query): PageResult` | The user's inbox, newest first; filters unread / kind | — |
| `unreadCount(User $user): int` | Bell badge | — |
| `markRead(User $user, string $code): void` | Marks one of the user's own notifications read | 404 when missing or someone else's |
| `markAllRead(User $user): int` | Marks every unread one read | — |
| `notifyCustomer(Reservation $reservation, NotificationTemplate $template, array $params = [], bool $skipActor = true): void` | Reservation event → customer. Expiry passes `$skipActor = false` because the sweep may run during that customer's own request | — |
| `notifyStaff(Reservation $reservation, NotificationTemplate $template, array $params = []): void` | Reservation event → stadium manager, or every active admin when the stadium has no manager | — |
| `notifyOtherParty(User $actor, Reservation $reservation, NotificationTemplate $template, array $params = []): void` | Customer acted → staff; staff acted → customer | — |
| `notifyStadiumStaff(?int $managerId, string $stadiumCode, NotificationTemplate $template, array $params): void` | Stadium-level event (new review) | — |

**Rules:** the user who performed the action is never notified about it; the body is rendered once with `NotificationTemplate::body()`; links are `/reservations/{code}` or `/stadiums/{code}#reviews`.

## Where notifications are sent

| Action | Service | Template → recipient |
| --- | --- | --- |
| Customer books | `ReservationService::create` | `reservation.requested` (or `reservation.booked` with auto-confirm) → staff |
| Confirm | `ReservationService::confirm` | `reservation.confirmed` → customer |
| Reject | `ReservationService::reject` | `reservation.rejected` (with reason) → customer |
| Cancel | `ReservationService::cancel` | `reservation.cancelled` (with reason) → other party |
| Reschedule | `ReservationService::reschedule` | `reservation.rescheduled` → other party |
| Complete | `ReservationService::complete` | `reservation.completed` → customer |
| Pending expired | `ReservationService::expireStalePending` | `reservation.expired` → customer |
| Payment / refund | `PaymentService::recordTransaction` | `payment.recorded` / `payment.refunded` → customer |
| New review | `ReviewService::create` | `review.created` → staff |
