# Migrations

Plain SQL files in `migrations/`, applied in filename order by `bin/migrate.php`, which creates the database when missing and records each file in `schema_migrations`. Run it again any time: applied files are skipped.

| Order | File | Creates |
| --- | --- | --- |
| 001 | `001_create_users_table.sql` | `users`, `login_attempts` |
| 002 | `002_create_cities_and_addresses_tables.sql` | `cities`, `addresses` (FK → cities) |
| 003 | `003_create_stadiums_table.sql` | `stadiums` (FK → users, addresses) |
| 004 | `004_create_reservations_table.sql` | `reservations` (FK → stadiums, users) |
| 005 | `005_create_payments_and_expenses_tables.sql` | `payments` (FK → reservations), `expenses` (FK → stadiums) |
| 006 | `006_create_settings_and_role_permissions_tables.sql` | `settings`, `role_permissions` |

All tables use InnoDB and `utf8mb4_unicode_ci`. Foreign keys use `RESTRICT` (soft delete keeps history) except `stadiums.manager_id` → `SET NULL`.

Never edit an applied migration; add a new numbered file for schema changes.
