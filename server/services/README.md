# Services

Business logic lives in `services/<Group>/`. Services take DTOs and models, never HTTP objects, and throw `AppException` subclasses that the error handler maps to the [JSON error shape](../globals/README.md#response-shape). Every function below is public; private helpers are described inside the function that uses them.

| Group | Service | Doc |
| --- | --- | --- |
| Common | `AccessPolicy` | [Common/access-policy.md](Common/access-policy.md) |
| Users | `AuthService`, `UserService`, `PermissionService` | [Users/](Users/users.md) |
| Regional | `CityService` | [Regional/city-service.md](Regional/city-service.md) |
| Stadiums | `StadiumService` | [Stadiums/stadium-service.md](Stadiums/stadium-service.md) |
| Reservations | `ConflictService`, `ReservationService` | [Reservations/](Reservations/reservations.md) |
| Accounting | `PaymentService`, `ExpenseService`, `ReportService` | [Accounting/](Accounting/accounting.md) |
| Settings | `SettingsService` | [Settings/settings-service.md](Settings/settings-service.md) |

## Exceptions used

| Exception | HTTP | Error code |
| --- | --- | --- |
| `ValidationException` | 422 | `VALIDATION_FAILED` (with `details.fields`) |
| `UnauthorizedException` | 401 | `UNAUTHENTICATED` |
| `ForbiddenException` | 403 | `FORBIDDEN` |
| `NotFoundException` | 404 | `NOT_FOUND` |
| `ConflictException` | 409 | `CONFLICT`, `DUPLICATE_VALUE`, `RESERVATION_CONFLICT`, `INVALID_TRANSITION`, `CANCEL_CUTOFF_PASSED`, `CITY_IN_USE`, `STADIUM_HAS_BOOKINGS`, `MANAGER_HAS_STADIUMS`, `SELF_ACTION_FORBIDDEN` |
| `TooManyRequestsException` | 429 | `TOO_MANY_ATTEMPTS` |
| `AppException` (direct) | 403 / 422 | `CSRF_MISMATCH`, `RESERVATION_IN_PAST`, `OUTSIDE_OPENING_HOURS`, `AMOUNT_EXCEEDS_BALANCE` |
