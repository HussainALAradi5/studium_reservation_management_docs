# Reservation

Table `reservations` — one booking of one stadium for one continuous time range.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | |
| `stadium_id` | `INT UNSIGNED` FK | no | Active stadium |
| `customer_id` | `INT UNSIGNED` FK | no | The customer who booked |
| `start_at` | `DATETIME` | no | Hourly: on the slot grid inside opening hours. Daily: opening time of the first day |
| `end_at` | `DATETIME` | no | `> start_at`; may pass midnight. Daily: closing time of the last day |
| `booking_unit` | `ENUM('hour','day')` | no | [BookingUnit](../../enums/README.md#bookingunit) |
| `unit_count` | `DECIMAL(6,2)` | no | Hours (may be 1.5) or days |
| `unit_price` | `DECIMAL(10,3)` | no | Snapshot of `price_per_hour` or `price_per_day` at booking time |
| `duration_minutes` | `INT UNSIGNED` | no | Real length of the range; hourly: multiple of `slot_minutes`, at most `reservation.max_hours` |
| `total_amount` | `DECIMAL(10,3)` | no | `unit_price × unit_count`, computed on the server only |
| `paid_amount` | `DECIMAL(10,3)` | no | Payments − refunds, updated in the same transaction as each payment |
| `status` | `ENUM` | no | [ReservationStatus](../../enums/README.md#reservationstatus) |
| `payment_status` | `ENUM` | no | [PaymentStatus](../../enums/README.md#paymentstatus) |
| `notes` | `VARCHAR(500)` | yes | Customer notes |
| `status_reason` | `VARCHAR(255)` | yes | Reason given on reject or cancel |

**Relations:** many-to-one Stadium, many-to-one User (customer), one-to-many Payment.

Time ranges are half-open `[start_at, end_at)`: a booking ending at 18:00 and one starting at 18:00 do not conflict. See [conflict resolution](../../../workflows/conflict-resolution.md) and the [lifecycle](../../../workflows/reservation-lifecycle.md).
