# Local setup (XAMPP)

## Requirements

- XAMPP with Apache 2.4 and PHP 8.2+ (`pdo_mysql` enabled — default in XAMPP)
- XAMPP's MySQL/MariaDB (10.4+)
- `mod_rewrite` on and `AllowOverride All` for `htdocs` (XAMPP defaults)

## Steps

1. Put the application in `C:\xampp\htdocs\studium_reservation_management`.
2. Start **Apache** and **MySQL** from the XAMPP Control Panel.
3. Copy `.env.example` to `.env` and set the database user/password (XAMPP default: `root`, empty password). Set `SEED_PASSWORD` to a development password (8+ characters with a letter and a digit).
4. Create the database and tables:

   ```bash
   C:\xampp\php\php.exe bin/migrate.php
   ```

5. Seed development data (refuses to run when `APP_ENV=production`):

   ```bash
   C:\xampp\php\php.exe bin/seed.php
   ```

   Creates default role permissions, settings, the users `admin`, `manager`, `customer` (password = `SEED_PASSWORD`), six Bahraini cities, and three sample stadiums assigned to `manager`.

6. Open <http://localhost/studium_reservation_management/>.

## Environment variables

| Key | Default | Purpose |
| --- | --- | --- |
| `APP_NAME` | Studium | Title |
| `APP_ENV` | development | `production` disables debug output and the seed |
| `APP_DEBUG` | true | Show exception messages (never in production) |
| `APP_TIMEZONE` | Asia/Bahrain | PHP and MySQL session timezone |
| `APP_CURRENCY` | BHD | Display currency (3 decimals) |
| `APP_BASE_PATH` | auto | URL prefix when auto-detection is wrong |
| `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASS` | localhost, 3306, studium_reservation_management, root, — | MySQL |
| `SESSION_IDLE_MINUTES` | 120 | Idle sign-out |
| `MAP_DEFAULT_LAT`, `MAP_DEFAULT_LNG`, `MAP_DEFAULT_ZOOM` | Bahrain | Map picker start |
| `MAP_TILE_URL` | OpenStreetMap | Tile server |
| `SEED_PASSWORD` | — | Development seed password |

## Tests

Unit tests (no packages needed):

```bash
C:\xampp\php\php.exe tests/run.php
```

API suite against the running app (roles, IDOR, errors, uniqueness, notifications, reviews), from Git Bash with Apache, MySQL, and the dev seed:

```bash
bash tests/http/api-suite.sh
```

## Logs

Unexpected errors: `storage/logs/app.log`. Apache errors: `C:\xampp\apache\logs\error.log`.
