# Stadium

Table `stadiums` — a bookable field or court.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | |
| `manager_id` | `INT UNSIGNED` FK → `users.id` | yes | Must be an active user with role `manager` |
| `address_id` | `INT UNSIGNED` FK → `addresses.id` | no | Unique (one-to-one) |
| `name` | `VARCHAR(120)` | no | 3–120 |
| `description` | `TEXT` | yes | Up to 2000 characters |
| `sport_type` | `ENUM` | no | [SportType](../../enums/README.md#sporttype) |
| `surface_type` | `ENUM` | no | [SurfaceType](../../enums/README.md#surfacetype) |
| `capacity` | `SMALLINT UNSIGNED` | no | Players, 2–200 |
| `price_per_hour` | `DECIMAL(10,3)` | no | > 0, currency from `APP_CURRENCY` (default BHD, 3 decimals per ISO 4217) |
| `price_per_day` | `DECIMAL(10,3)` | yes | Full-day rental price; null = rented by the hour only |
| `opening_time` | `TIME` | no | e.g. `08:00` |
| `closing_time` | `TIME` | no | If ≤ opening time, the stadium closes after midnight (16:00 → 02:00) |
| `slot_minutes` | `SMALLINT UNSIGNED` | no | Booking grid step: 30, 60, 90 or 120 |
| `is_active` | `TINYINT(1)` | no | Inactive stadiums are hidden from customers and not bookable |

**Relations:** many-to-one User (manager); one-to-one Address; one-to-many Reservation; one-to-many Expense.

**Rules:**
- Only an admin creates a stadium and assigns or changes its manager; the assigned manager edits details, hours, price, and location.
- A stadium with future pending or confirmed reservations cannot be deactivated (409 `STADIUM_HAS_BOOKINGS`).
- Changing the price never changes existing reservations: each reservation stores a price snapshot.
