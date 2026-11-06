# Stadium use cases

### UC-STD-01 Admin creates a stadium
- **Actor:** Admin
- **Preconditions:** At least one active city.
- **Main flow:** Stadium form: name, sport, surface, capacity, price, hours, slot length, manager, address (city, street, building, block, area) and a map pin → `POST /api/stadiums` → address and stadium inserted in one transaction.
- **Failure flows:** 422 field errors (e.g. latitude without longitude, manager is not a manager); inactive city → 422.
- **Result:** Stadium visible to customers when active.

### UC-STD-02 Manager edits a stadium and its location
- **Actor:** Manager of the stadium, or admin
- **Main flow:** Edit form → move the map marker / change fields → `PUT /api/stadiums/{code}` → stadium and address updated in one transaction.
- **Failure flows:** Not their stadium → 404; manager tries to change `manager_id` → ignored (only admins can reassign).
- **Result:** New price applies to new bookings only.

### UC-STD-03 Activate or deactivate a stadium
- **Main flow:** `POST /api/stadiums/{code}/toggle`.
- **Failure flows:** Future pending/confirmed bookings exist → 409 `STADIUM_HAS_BOOKINGS` with the count.

### UC-STD-04 Browse and filter stadiums
- **Actor:** Anyone
- **Main flow:** `/stadiums?city=&sport=&q=&page=` → paginated cards with price, city, sport, surface.
- **Result:** Only active stadiums in active cities.

### UC-STD-05 View a stadium and its availability
- **Actor:** Anyone (booking requires a customer sign-in)
- **Main flow:** `/stadiums/{code}` → details, map, Google Maps links → date picker loads `GET /api/stadiums/{code}/availability?date=` via AJAX → slot grid shows free / booked / past.
