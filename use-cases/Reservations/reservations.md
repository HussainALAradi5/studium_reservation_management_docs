# Reservation use cases

### UC-RSV-01 Customer books a slot
- **Actor:** Customer
- **Preconditions:** Signed in, stadium active.
- **Main flow (hourly):** Pick a day in the date strip → select a start slot and extend to contiguous free slots → summary shows the server quote → "Book" → `POST /api/reservations` with `bookingUnit=hour` → server validates, locks the stadium, checks overlap, inserts → redirect to the reservation page.
- **Main flow (full day):** Switch to "Full day" (stadiums with a day price) → pick a start day on the month calendar (free / partly booked / full) → choose the number of days → quote → "Book" with `bookingUnit=day`.
- **Result:** `pending` reservation holding the range (`confirmed` immediately when auto-confirm is on); managers see it in their queue.

### UC-RSV-02 Booking hits a conflict
- **Actor:** Customer
- **Preconditions:** Someone else booked an overlapping time after the grid was loaded.
- **Main flow:** `POST /api/reservations` → 409 `RESERVATION_CONFLICT` with suggestions → a dialog lists alternative times → customer picks one → booking retried with that time.
- **Alternate flow:** No suggestion fits → customer closes the dialog; the grid reloads with fresh availability.
- **Result:** No double booking.

### UC-RSV-03 Manager confirms a booking
- **Actor:** Manager of the stadium, admin
- **Main flow:** Reservation page or list → "Confirm" → `POST /api/reservations/{code}/confirm` → overlap re-checked under lock → `confirmed`.
- **Failure flows:** Not `pending` → 409 `INVALID_TRANSITION`; overlap → 409 with suggestions.

### UC-RSV-04 Manager rejects a booking
- **Main flow:** "Reject" → dialog asks for a reason → `POST /api/reservations/{code}/reject` → `rejected`, slot released.

### UC-RSV-05 Cancel a booking
- **Actors:** Owner customer, manager, admin
- **Main flow:** "Cancel" → reason → `POST /api/reservations/{code}/cancel` → `cancelled`, slot released.
- **Failure flows:** Customer cancels a confirmed booking less than 2 h before start → 409 `CANCEL_CUTOFF_PASSED`.
- **Result:** If money was paid, the reservation shows "Refund due".

### UC-RSV-06 Reschedule
- **Actors:** Manager, admin; owner customer while `pending`
- **Main flow:** "Reschedule" → choose date, start time, duration → `POST /api/reservations/{code}/reschedule` → full validation and overlap check excluding itself → times and total updated (original price snapshot kept).
- **Failure flows:** Conflict → 409 with suggestions; paid amount larger than the new total → payment status becomes `paid` and the difference shows as refund due.

### UC-RSV-07 Complete
- **Actor:** Manager, admin
- **Preconditions:** `confirmed` and `end_at` passed.
- **Main flow:** "Mark completed" → `POST /api/reservations/{code}/complete`.

### UC-RSV-08 Pending expiry
- **Actor:** System
- **Main flow:** Before lists and dashboards load, pending reservations whose start time has passed are cancelled with reason "Expired".
