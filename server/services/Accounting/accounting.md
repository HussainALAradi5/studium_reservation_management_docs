# Accounting services

## PaymentService — `services/Accounting/PaymentService.php`

### `listForReservation(Reservation $reservation): Payment[]`
Ledger oldest first, with the name of who recorded each row.

### `recordTransaction(User $actor, string $reservationCode, PaymentFormDto $dto): Reservation`
**Logic (one transaction):** 1) reservation must be manageable by the actor (404 otherwise); 2) `SELECT … FOR UPDATE` on the reservation, reload it; 3) validate; 4) insert the payment row; 5) `paid_amount ± amount`; 6) `payment_status = PaymentStatus::fromAmounts(paid, total, hasRefunds)`.

| Case | Result |
| --- | --- |
| Payment ≤ balance on pending/confirmed/completed | Recorded |
| Payment on rejected/cancelled | 409 `INVALID_TRANSITION` |
| Payment > balance | 422 `AMOUNT_EXCEEDS_BALANCE` on `amount` |
| Refund ≤ paid (any status) | Recorded; `refunded` when paid reaches 0 |
| Refund > paid | 422 `AMOUNT_EXCEEDS_BALANCE` |

## ExpenseService — `services/Accounting/ExpenseService.php`

| Function | Purpose | Exceptions |
| --- | --- | --- |
| `list(User $user, ExpenseQuery $query): PageResult` | Scoped, filtered by stadium, category, dates | — |
| `create(User $actor, ExpenseFormDto $dto): void` | Stadium must be manageable | 404, 422 |
| `delete(User $actor, string $code): void` | Soft delete | 404 |

## ReportService — `services/Accounting/ReportService.php`

### `buildSummary(User $user, ReportQuery $query): array`
**Purpose:** The accounting report and dashboard summary. **Logic:** resolve stadiums in scope (or one manageable stadium) → four aggregate queries (cash per stadium, expenses per stadium, bookings per stadium and status, expenses per category) and one daily cash query → combine in PHP. Definitions are in [accounting.md](../../../workflows/accounting.md#report-reportservicebuildsummary).

Returns: `from, to, stadiums, selectedStadiumCode, grossRevenue, refunds, netRevenue, expenses, netProfit, bookedValue, outstanding, utilization, bookedHours, statusCounts, expensesByCategory, daily, perStadium`.

**Exceptions:** 404 for a stadium out of scope; 422 from `ReportQuery` when `from > to` or the range exceeds 366 days.
