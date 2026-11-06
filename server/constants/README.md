# Constants

`constants/<Group>/` — every repeated value lives here; services, controllers, and views never use magic strings or numbers.

| File | Contents |
| --- | --- |
| `Common/ErrorCodes.php` | Every `error.code` returned by the API (see [services](../services/README.md#exceptions-used)) |
| `Common/RegexPatterns.php` | All validation regexes (person name, username, E.164 phone, ISO 3166 country code, place name, street, building/block number, stadium name, payment reference, record code, HH:MM, ISO date, password letter/digit). `html()` converts a pattern to an HTML `pattern` attribute so the browser and the server check the same rule |
| `Common/Pagination.php` | Default page size 10, max 50, sort directions |
| `Common/CodePrefixes.php` | Record code prefixes (`USR`, `CTY`, `ADR`, `STD`, `RSV`, `PAY`, `EXP`, `SET`, `RPM`), code length 8, unambiguous alphabet |
| `Users/AuthRules.php` | 5 failed sign-ins per 15 minutes, password minimum 8, session keys |
| `Stadiums/StadiumRules.php` | Slot lengths 30/60/90/120, capacity 2–200, description max 2000, default country `BH` |
| `Reservations/ReservationRules.php` | Minimum hourly booking 30 min, suggestion search 7 days, month pattern, expiry reason, datetime format |
| `Accounting/AccountingRules.php` | 3 decimals, max amount, max report range 366 days |
| `Reviews/ReviewRules.php` | Rating 1–5, comment 3–1000, 5 reviews per page |
| `Notifications/NotificationRules.php` | Bell shows 8, inbox pages of 15, date label format |

`RegexPatterns` also holds `PHONE_SEPARATORS` and `WHITESPACE_RUN` (used by `IdentityNormalizer` for case- and format-insensitive uniqueness) and `TEMPLATE_PLACEHOLDER` (notification templates). `CodePrefixes` adds `NTF` (notifications) and `REV` (reviews); `ErrorCodes` adds `REVIEW_NOT_ALLOWED` and `REVIEW_ALREADY_EXISTS`.

Admin-tunable booking limits are **settings**, not constants: see [SettingsKey](../enums/README.md#settingskey).
