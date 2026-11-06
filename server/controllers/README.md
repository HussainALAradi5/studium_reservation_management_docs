# Controllers and endpoints

All routes are defined in `config/Routes/routes.php`. Pages render HTML; `/api/*` returns the [JSON shape](../globals/README.md#response-shape) and is called with `fetch` from `assets/javascripts/`. Mutations need the CSRF token (`X-CSRF-Token` header). "Permission" is the `can:` middleware ([permission list](../enums/README.md#permission)); "Scope" is the record-level check done in the service.

Request bodies are JSON with camelCase fields. Error codes per status are listed in [services](../services/README.md#exceptions-used).

## HomeController — `controllers/Home/HomeController.php`

| Method and path | Service function | Why | Access | Response |
| --- | --- | --- | --- | --- |
| `GET /` | `StadiumService::browse`, `CityService::listActive` | Landing page with search and featured stadiums | public | HTML |
| `GET /dashboard` | `ReservationService::listUpcoming`, `countByStatus`, `ReportService::buildSummary` (manager/admin), `UserService::countByRole` (admin) | Role-aware dashboard | signed in | HTML |

## AuthController — `controllers/Users/AuthController.php`

| Method and path | Service function | Why | Access | Request DTO | Response | Errors |
| --- | --- | --- | --- | --- | --- | --- |
| `GET /login`, `GET /register` | — | Forms | guests (signed-in users go to `/dashboard`) | — | HTML | — |
| `POST /api/auth/login` | `AuthService::login` | Sign in | public | `LoginDto {identifier, password, next?}` | `{message, redirect}` | 401, 403 CSRF, 422, 429 |
| `POST /api/auth/register` | `AuthService::register` | Customer sign-up | public | `RegisterDto {fullName, username, email, phone, password, passwordConfirmation}` | `{message, redirect}` 201 | 409, 422 |
| `POST /api/auth/logout` | `AuthService::logout` | Sign out | signed in | — | `{message, redirect}` | — |

`next` is accepted only as an internal path (no open redirects).

## ProfileController — `controllers/Users/ProfileController.php`

| Method and path | Service function | Access | Request DTO | Errors |
| --- | --- | --- | --- | --- |
| `GET /profile` | — | signed in | — | — |
| `PUT /api/profile` | `UserService::updateProfile` | signed in | `ProfileDto {fullName, email, phone, gender?, birthDate?}` | 409, 422 |
| `PUT /api/profile/password` | `UserService::changePassword` | signed in | `ChangePasswordDto {currentPassword, newPassword, newPasswordConfirmation}` | 422 |

## UserController — `controllers/Users/UserController.php` (permission `users.manage`)

| Method and path | Service function | Request DTO | Errors |
| --- | --- | --- | --- |
| `GET /users?q&role&status&page` | `UserService::list` | `UserQuery` | — |
| `GET /users/create`, `GET /users/{code}/edit` | `UserService::findByCode` | — | 404 |
| `POST /api/users` | `UserService::create` | `UserFormDto {fullName, username, email, phone, role, gender?, birthDate?, isActive, password, passwordConfirmation}` | 409, 422 |
| `PUT /api/users/{code}` | `UserService::update` | `UserFormDto` (password optional) | 404, 409, 422 |
| `POST /api/users/{code}/toggle` | `UserService::toggleActive` | — | 404, 409 |

## CityController — `controllers/Regional/CityController.php` (permission `cities.manage`)

| Method and path | Service function | Request DTO | Response | Errors |
| --- | --- | --- | --- | --- |
| `GET /cities?q&page` | `CityService::list` | `CityQuery` | HTML | — |
| `POST /api/cities` | `CityService::create` | `CityFormDto {name, countryCode, isActive}` | `{message, city}` 201 | 409, 422 |
| `PUT /api/cities/{code}` | `CityService::update` | `CityFormDto` | `{message, city}` | 404, 409, 422 |
| `DELETE /api/cities/{code}` | `CityService::delete` | — | `{message}` | 404, 409 `CITY_IN_USE` |

## StadiumController — `controllers/Stadiums/StadiumController.php`

| Method and path | Service function | Why | Access | Request | Response | Errors |
| --- | --- | --- | --- | --- | --- | --- |
| `GET /stadiums?q&city&sport&sort&page` | `StadiumService::browse` | Catalogue | public | `StadiumQuery` | HTML | — |
| `GET /stadiums/{code}` | `StadiumService::findVisible` | Details, map, booking widget | public | — | HTML | 404 |
| `GET /api/stadiums/{code}/availability?date` | `ConflictService::availability` | Hourly slot grid | public | `date` (YYYY-MM-DD) | `{date, slotMinutes, pricePerHour, currency, hours, slots[]}` | 404, 422 |
| `GET /api/stadiums/{code}/calendar?month` | `ConflictService::calendar` | Day states for full-day rentals | public | `month` (YYYY-MM) | `{month, pricePerDay, days[]}` | 404, 422 |
| `GET /api/stadiums/{code}/quote?bookingUnit&date&startTime&durationMinutes&days` | `ConflictService::quote` | Server price + availability of a selection | public | `BookingRequestDto` | `{…slot, unitPrice, total, totalLabel, isAvailable, conflicts, suggestions}` | 404, 422 |
| `GET /manage/stadiums` | `StadiumService::listManaged` | Management list | `stadiums.manage` + scope | `StadiumQuery` (+ `status`) | HTML | — |
| `GET /manage/stadiums/create` | — | New stadium form | `stadiums.create` | — | HTML | — |
| `GET /manage/stadiums/{code}/edit` | `StadiumService::findManageable` | Edit form with map picker | `stadiums.manage` + scope | — | HTML | 404 |
| `POST /api/stadiums` | `StadiumService::create` | Create | `stadiums.create` | `StadiumFormDto` | `{message, redirect}` 201 | 409, 422 |
| `PUT /api/stadiums/{code}` | `StadiumService::update` | Edit (manager change admin-only) | `stadiums.manage` + scope | `StadiumFormDto` | `{message, redirect}` | 404, 409, 422 |
| `POST /api/stadiums/{code}/toggle` | `StadiumService::toggleActive` | Open / close for bookings | `stadiums.manage` + scope | — | `{message}` | 404, 409 `STADIUM_HAS_BOOKINGS` |

`StadiumFormDto`: `name, description?, sportType, surfaceType, capacity, pricePerHour, pricePerDay?, openingTime, closingTime, slotMinutes, isActive, managerCode?, cityCode, street, building?, block?, area?, latitude?, longitude?`.

## ReservationController — `controllers/Reservations/ReservationController.php`

| Method and path | Service function | Access | Request DTO | Response | Errors |
| --- | --- | --- | --- | --- | --- |
| `GET /reservations?q&status&stadium&from&to&page` | `ReservationService::list` | signed in, scoped | `ReservationQuery` | HTML | — |
| `GET /reservations/{code}` | `findViewable`, `PaymentService::listForReservation` | signed in, scoped | — | HTML | 404 |
| `POST /api/reservations` | `create` | `reservations.book` | `CreateReservationDto {stadiumCode, bookingUnit, date, startTime?, durationMinutes?, days?, notes?}` | `{message, redirect, code}` 201 | 403, 404, 409 `RESERVATION_CONFLICT`, 422 |
| `POST /api/reservations/{code}/confirm` | `confirm` | `reservations.manage` + scope | — | `{message}` | 404, 409 |
| `POST /api/reservations/{code}/reject` | `reject` | `reservations.manage` + scope | `StatusChangeDto {reason}` (required) | `{message}` | 404, 409, 422 |
| `POST /api/reservations/{code}/complete` | `complete` | `reservations.manage` + scope | — | `{message}` | 404, 409 |
| `POST /api/reservations/{code}/cancel` | `cancel` | owner / manager / admin | `StatusChangeDto {reason}` (required for staff) | `{message}` | 404, 409 `CANCEL_CUTOFF_PASSED`, 422 |
| `POST /api/reservations/{code}/reschedule` | `reschedule` | staff, or owner while pending | `BookingRequestDto` | `{message}` | 404, 409, 422 |

## PaymentController — `controllers/Accounting/PaymentController.php`

| Method and path | Service function | Access | Request DTO | Errors |
| --- | --- | --- | --- | --- |
| `POST /api/reservations/{code}/payments` | `PaymentService::recordTransaction` | `payments.record` + scope | `PaymentFormDto {type, method, amount, paidAt, reference?, notes?}` | 404, 409, 422 `AMOUNT_EXCEEDS_BALANCE` |

## ExpenseController / ReportController — `controllers/Accounting/`

| Method and path | Service function | Access | Request | Errors |
| --- | --- | --- | --- | --- |
| `GET /accounting/expenses?stadium&category&from&to&page` | `ExpenseService::list` | `expenses.manage` | `ExpenseQuery` | — |
| `POST /api/expenses` | `ExpenseService::create` | `expenses.manage` + scope | `ExpenseFormDto {stadiumCode, category, amount, expenseDate, description}` | 404, 422 |
| `DELETE /api/expenses/{code}` | `ExpenseService::delete` | `expenses.manage` + scope | — | 404 |
| `GET /accounting/reports?from&to&stadium` | `ReportService::buildSummary` | `reports.view` + scope | `ReportQuery` | 404, 422 |

## SettingsController — `controllers/Settings/SettingsController.php` (permission `settings.manage`)

| Method and path | Service function | Request DTO | Errors |
| --- | --- | --- | --- |
| `GET /settings` | `SettingsService::all`, `PermissionService::matrix` | — | — |
| `PUT /api/settings` | `SettingsService::update` | `SettingsFormDto` — one field per key, e.g. `reservation_max_hours` | 422 |
| `PUT /api/settings/permissions/{role}` | `PermissionService::updateRole` | `RolePermissionsDto {permissions: string[]}` | 409 (admin), 422 |
