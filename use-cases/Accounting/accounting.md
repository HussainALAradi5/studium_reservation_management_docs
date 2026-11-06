# Accounting use cases

### UC-ACC-01 Record a payment
- **Actor:** Manager of the stadium, admin
- **Preconditions:** Reservation not `rejected`, balance > 0.
- **Main flow:** Reservation page → "Add payment" dialog (amount defaults to the balance, method, date, reference) → `POST /api/reservations/{code}/payments` with `type=payment` → reservation locked → payment inserted → `paid_amount`, `payment_status` updated → ledger and balance refresh in place.
- **Failure flows:** Amount > balance → 422 `AMOUNT_EXCEEDS_BALANCE`.

### UC-ACC-02 Record a refund
- **Main flow:** Same dialog with `type=refund`; amount defaults to the paid amount.
- **Failure flows:** Refund > paid amount → 422.
- **Result:** `payment_status` becomes `refunded` when everything was returned.

### UC-ACC-03 Log an expense
- **Actor:** Manager (own stadiums), admin
- **Main flow:** Expenses page → "New expense" dialog → `POST /api/expenses` → row added.
- **Failure flows:** Stadium not managed by the user → 404; future date → 422.

### UC-ACC-04 Delete an expense
- **Main flow:** Confirm → `DELETE /api/expenses/{code}` → soft-deleted, excluded from reports.

### UC-ACC-05 View the accounting report
- **Actor:** Manager (own stadiums), admin (all)
- **Main flow:** `/accounting/reports?from=&to=&stadium=` → KPI cards (net revenue, expenses, profit, outstanding, utilization), daily revenue bars, expenses by category, per-stadium table, bookings by status.
- **Failure flows:** `from` after `to` → validation message; range > 366 days → validation message.
