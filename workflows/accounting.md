# Accounting

## Amounts

- Currency comes from `APP_CURRENCY` (default `BHD`, 3 decimals). Amounts are `DECIMAL(10,3)` in MySQL and are rounded to 3 decimals in PHP before storing.
- `total_amount = unit_price × unit_count`, where `unit_price` is the stadium's `price_per_hour` (hourly) or `price_per_day` (daily) at booking time. The server computes it; a price sent by the client is ignored.

## Payments and refunds

Recorded by the stadium's manager or an admin from the reservation page (`PaymentService::recordTransaction`). In one transaction:

1. Lock the reservation row (`FOR UPDATE`).
2. Validate:

| Rule | Error |
| --- | --- |
| Payment on a `rejected` reservation | 409 `INVALID_TRANSITION` |
| Payment amount > outstanding (`total_amount − paid_amount`) | 422 `AMOUNT_EXCEEDS_BALANCE` |
| Refund amount > `paid_amount` | 422 `AMOUNT_EXCEEDS_BALANCE` |
| `paid_at` in the future | 422 `VALIDATION_FAILED` |

3. Insert the `payments` row.
4. Update `paid_amount` (`+amount` for payment, `−amount` for refund) and `payment_status`.

## Payment status

| Condition | `payment_status` |
| --- | --- |
| `paid_amount = 0` and at least one refund exists | `refunded` |
| `paid_amount = 0` | `unpaid` |
| `0 < paid_amount < total_amount` | `partial` |
| `paid_amount ≥ total_amount` | `paid` |

A cancelled or rejected reservation with `paid_amount > 0` is flagged **Refund due**.

## Expenses

Managers log expenses per stadium (category, amount, date, description). Admins can log for any stadium.

## Report (`ReportService::buildSummary`)

Scope: admin → all stadiums or one; manager → their stadiums or one of them. Period: `from`–`to` dates (default: current month).

| Figure | Definition |
| --- | --- |
| Gross revenue | Sum of `payment` amounts with `paid_at` in the period |
| Refunds | Sum of `refund` amounts with `paid_at` in the period |
| Net revenue | Gross revenue − refunds |
| Expenses | Sum of expenses in the period, also split by category |
| Net profit | Net revenue − expenses |
| Booked value | Sum of `total_amount` of `confirmed` + `completed` reservations starting in the period |
| Outstanding | Sum of `total_amount − paid_amount` for `confirmed` + `completed` reservations starting in the period |
| Bookings by status | Count per status of reservations starting in the period |
| Utilization | Booked hours (`confirmed` + `completed`) ÷ open hours of the scoped stadiums in the period |
| Per stadium | Net revenue, expenses, profit, bookings for each stadium |
| Daily trend | Net revenue per day, rendered as a CSS bar chart |

Revenue is cash-based (money actually received in the period); booked value and outstanding show the accrual side.
