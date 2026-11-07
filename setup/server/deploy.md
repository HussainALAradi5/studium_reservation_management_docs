# Deployment

**Live:** <https://studium-bh.infinityfreeapp.com> — InfinityFree free plan, PHP 8, MySQL host `sql212.infinityfree.com`, database `if0_43381176_studium`. Deployed 2026-11-07 by importing `bin/export-sql.php --with-admin` output in phpMyAdmin and uploading the release package through the InfinityFree File Manager.

### Live smoke test (2026-11-07)

| Check | Result |
| --- | --- |
| `/`, `/stadiums`, `/login`, `/register` | 200 |
| `/dashboard` signed out | Redirect to sign-in |
| `/api/notifications` signed out | 401 |
| `/.env` | Blocked by the host (empty response) |
| `/config/…`, `/bin/…`, `/migrations/…`, `/storage/logs/…` | 404 |
| HTTPS, HSTS (180 days), CSP, `X-Frame-Options: DENY` | Present |
| Stylesheets | Single `assets/build/app.min.css` |
| Database | Landing page reads stadiums and cities without errors |

Studium needs **PHP 8.2+, MySQL 5.7+/MariaDB 10.4+, and Apache with `.htaccess` (mod_rewrite)**. Static hosts cannot run it.

| Host | Fits? | Notes |
| --- | --- | --- |
| GitHub Pages | No | Static files only (no PHP, no database) |
| Vercel | No (without a rewrite) | No native PHP, no MySQL, no persistent files for sessions/logs |
| Render | Yes, via Docker | Free web service sleeps after 15 min idle; no free MySQL, so add an external one (e.g. Aiven free MySQL) |
| InfinityFree | Yes, as-is | Free PHP + MySQL + Apache; no command line, so import the SQL with phpMyAdmin |
| Shared hosting (Hostinger, Namecheap) | Yes, as-is | Paid, faster, SSH on most plans |
| VPS (DigitalOcean, Hetzner) | Yes | Full control; you maintain the server |

## Production settings (`.env` on the server)

| Key | Value |
| --- | --- |
| `APP_ENV` | `production` (hides debug details, uses the CSS bundle, blocks the dev seed) |
| `APP_DEBUG` | `false` |
| `APP_TRUST_PROXY` | `true` when a proxy terminates HTTPS (InfinityFree, Cloudflare, Render) so session cookies are `Secure` and HSTS is sent |
| `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASS` | From the host's MySQL panel |
| `APP_BASE_PATH` | Empty (auto-detected) unless the app lives in a sub-folder that detection gets wrong |

Real environment variables (Render, Docker) take precedence over `.env`.

## Database

- **Command line available:** `php bin/migrate.php`, then `php bin/install.php`. The install grants default permissions only to roles without any, adds missing settings, and creates the first admin only while none exists; it is safe to re-run. Admin details come from `INSTALL_ADMIN_*` environment variables or prompts.
- **No command line:** locally run `php bin/export-sql.php --with-admin`, import `deploy/studium-install.sql` into the empty database with phpMyAdmin, then delete the file (it holds the admin's password hash). `deploy/` is git-ignored.

## Assets

Production serves `assets/build/app.min.css` (one request). Run `php bin/build-assets.php` after any CSS change; the unit test `AssetsTest` fails while the bundle is stale.

## Upload checklist (shared hosting)

1. Upload everything except `.env.example`, `deploy/`, `tests/`, `screenshots/`, `node_modules/`, `.git/` so `index.php` and `.htaccess` are in the web root (`htdocs/` on InfinityFree).
2. Create `.env` on the server with the production values; it is denied to browsers by `.htaccess`.
3. Make `storage/logs/` writable.
4. Open the site, sign in as the admin, and add cities, managers, and stadiums.

## Release verification (2026-11-07, local, production mode)

| Check | Result |
| --- | --- |
| Unit tests (`tests/run.php`) | 28 passed, 81 assertions |
| API suite in `APP_ENV=production` | 83 passed, 0 failed, empty error log |
| Install SQL imported into an empty database | 8 migrations, permissions (manager 6, customer 2), 7 settings |
| `bin/install.php` twice on a fresh database | Admin created once (identity normalized); second run changes nothing |
| `bin/seed.php` with `APP_ENV=production` | Refused, exit code 1 |
| Production page head | One stylesheet (`app.min.css`, 48.9 KB from 57.3 KB in 10 files) |
| Screenshots (desktop/mobile, light/dark) | In the app repository `screenshots/` |
| Docker image for Render | Not built (Docker not installed locally) |
