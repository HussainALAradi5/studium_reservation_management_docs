# Studium — Stadium Reservation Management (documentation)

Documentation repository for the Studium stadium reservation system. The application code lives in a separate repository:

| Repository | Path | Role |
| --- | --- | --- |
| Application (server + web client) | [HussainALAradi5/studium_reservation_management](https://github.com/HussainALAradi5/studium_reservation_management) — local: `C:\xampp\htdocs\studium_reservation_management` | PHP 8.2 server-rendered app, JSON API for AJAX, vanilla CSS and JS |
| Documentation (this repo) | [HussainALAradi5/studium_reservation_management_docs](https://github.com/HussainALAradi5/studium_reservation_management_docs) — local: `D:\IT\htdocs_projects\studium_reservation_management_docs` | Design, models, services, controllers, use cases, phases |

## What the system does

- Customers browse stadiums by city and sport, see live availability, and book time slots.
- Managers run the stadiums assigned to them: confirm or reject bookings, reschedule, record payments and refunds, log expenses, and read accounting reports.
- Admins manage users, cities, and every stadium, and see global accounting.
- Every stadium has an address (city + street) and a map location picked on a free OpenStreetMap map, with Google Maps links for directions.
- Double bookings are prevented by a locked, transactional overlap check; a conflicting request gets alternative free slots back.

## Stack

| Layer | Technology |
| --- | --- |
| Runtime | XAMPP: Apache 2.4 + PHP 8.2 |
| Database | MySQL / MariaDB 10.4 (InnoDB, utf8mb4) |
| Server | Plain PHP, layer-first folders, PSR-4 style autoloader, PDO |
| Web client | Server-rendered PHP views, vanilla CSS design system, vanilla ES modules using `fetch` (AJAX) |
| Maps | Leaflet 1.9.4 + OpenStreetMap tiles (free, no key); Google Maps URL links |

## Navigation

| Section | Contents |
| --- | --- |
| [phases/](phases/README.md) | Project phases, status, verification |
| [server/models/](server/models/README.md) | Entities, base entity, ERD, relationships |
| [server/enums/](server/enums/README.md) | Enumerations |
| [server/constants/](server/constants/README.md) | Constants and error codes |
| [server/globals/](server/globals/README.md) | Config, middlewares, utilities, conventions |
| [server/services/](server/services/README.md) | Every service function |
| [server/controllers/](server/controllers/README.md) | Every page and API endpoint |
| [client/](client/README.md) | CSS design system and JavaScript modules |
| [workflows/](workflows/README.md) | Reservation lifecycle, conflict resolution, accounting |
| [use-cases/](use-cases/README.md) | Use cases per actor |
| [security/](security/README.md) | Roles, permission matrix, controls |
| [integrations/maps.md](integrations/maps.md) | Map and location integration |
| [migrations/](migrations/README.md) | Migration order and schema notes |
| [setup/server/](setup/server/README.md) | Local setup on XAMPP |
