# Web client

Server-rendered PHP views styled with a vanilla CSS design system, enhanced by vanilla ES modules that talk to the `/api` endpoints with `fetch` (AJAX). No framework, no build step.

## Design system — `assets/css/`

| File | Contents |
| --- | --- |
| `base/tokens.css` | Steel-blue palette (`--primary-rgb: 70, 130, 180`) with glass, radius, shadow, and motion tokens; every glow and focus ring derives from `--primary-rgb`; light by default, dark via `prefers-color-scheme` or `[data-theme="dark"]` |
| `base/base.css` | Reset, typography, animated background blobs, layout helpers (`stack`, `row`, `grid-*`, `split`) |
| `components/animations.css` | Generic keyframes and utilities: `anim-fade-up`, `anim-scale-in`, `anim-shake`, `[data-animate]` scroll reveal, `[data-stagger]`, `skeleton`, `spinner`, reduced-motion support |
| `components/buttons.css` | customButton: variants, sizes, ripple, loading spinner. Hover = 2 px lift + gradient glide + one forward shine sweep (a keyframe, so it never plays backwards on mouse-out), only on devices with a real hover pointer |
| `components/surfaces.css` | Glass cards, stat cards, tone helpers, badges (dot, live pulse, icon, solid), alerts, responsive tables, pagination, empty states, avatars |
| `components/forms.css` | Fields with icons, errors with shake, switch, checkbox chips, segmented control, stepper, filter bar |
| `components/wheel-picker.css` | iOS and Android wheel picker sheet |
| `components/overlays.css` | Dialogs, toasts, conflict suggestions |
| `layout/shell.css` | Glass sidebar + topbar (drawer on mobile), guest navigation |
| `pages/pages.css` | Landing, auth, stadium cards and details, booking widget (date strip, slots, calendar, summary), report charts, settings matrix, payment ledger |

## Generic partials — `views/partials/`

| Partial | Purpose |
| --- | --- |
| `ui/customButton.php` (`button()` helper) | Every button/link-button: variant, size, icon, icon-only, block, pill, `loadingText`, disabled |
| `ui/badge.php` (`badge()` helper), `ui/statusBadges.php` | Colored badges; reservation status + payment status + "Refund due" |
| `forms/genericForm.php` | AJAX form from field definitions with sections, columns, cancel, submit spinner |
| `forms/genericField.php` | text, email, tel, password (show/hide), number, select, textarea, switch, hidden, and date/time types; regex `pattern` from `RegexPatterns` |
| `forms/dateField.php` | Wheel picker field for date / time / month / datetime |
| `filters/genericFilter.php` | Auto-submitting GET filter bar for any list |
| `ui/genericTable.php`, `ui/pagination.php`, `ui/genericCard.php`, `ui/statCard.php`, `ui/pageHeader.php`, `ui/emptyState.php`, `ui/dialog.php`, `ui/stadiumCard.php`, `ui/icons.php` | Building blocks |
| `reservations/table.php`, `reservations/actions.php` | Booking table and lifecycle buttons (confirm, reject, complete, cancel) shared by dashboard and lists |
| `ui/stars.php`, `forms/genericField.php` type `rating` | Read-only stars with average and count; accessible 1–5 star radio input |
| `reviews/section.php`, `reviews/item.php` | Reviews card: locked for guests, form only when eligible, moderator remove buttons |
| `layout/guestNav.php` | Guest navbar, shared by public pages and the sign-in / register layout (which also has a Back button) |
| `layout/notificationBell.php` | Topbar bell with unread badge and dropdown |

**Locked states:** guests see the booking widget blurred behind a lock ("You must register first" → Register / Sign in, returning to the stadium afterwards) and the review comments locked ("Sign in to read reviews"). The server enforces both (`can:reservations.book`, `auth` on the reviews API).

## JavaScript — `assets/javascripts/`

| Folder | Module | Purpose |
| --- | --- | --- |
| — | `app.js` | Entry; starts shared behaviour, lazy-loads page widgets |
| `config/` | `endpoints.js` | API paths mirroring `config/Routes/routes.php` |
| `core/` | `http.js` | The one HTTP client: JSON, CSRF header, `ApiError`, `latestOnly()` to abort stale requests |
| `core/` | `dom.js`, `format.js` | Safe element builder (`textContent` only), delegation, debounce; money and date formatting |
| `core/` | `button.js` | `setLoading()` (disable + spinner), `withLoading()`, ripple |
| `core/` | `toast.js`, `dialog.js` | Toasts; promise-based confirm dialog with optional/required reason |
| `forms/` | `ajax-form.js`, `serialize.js` | Generic AJAX submit, field errors, conflict suggestions, success modes |
| `forms/` | `wheel-picker.js` | iOS/Android wheel picker (style remembered per browser) |
| `forms/` | `field-helpers.js` | Password toggle, hourly/daily field switch |
| `ui/` | `theme.js`, `sidebar.js`, `filters.js`, `action-buttons.js`, `dialog-triggers.js`, `animate.js` | Dark/light toggle, drawer, filter auto-submit, one-click API actions with confirmation, create/edit dialogs, scroll reveal |
| `stadiums/` | `booking-widget.js` | Hourly slot grid + full-day calendar, server quotes, booking, conflict alternatives |
| `stadiums/` | `leaflet-loader.js`, `map-view.js`, `map-picker.js` | Free OpenStreetMap maps (see [maps](../integrations/maps.md)) |
| `reservations/` | `conflict-dialog.js` | Lists free alternatives from a 409 response |
| `ui/` | `notification-bell.js` | Badge polling (60 s, paused in hidden tabs, toast on new items), dropdown, mark read with `keepalive`, inbox links |
| `ui/` | `back-link.js` | Back buttons: history back when coming from this site, else the link |
| `reviews/` | `reviews-list.js` | "Show more reviews" via the API with the same markup as the server partial |
| `core/` | `navigation.js` | `navigate()` — follows API redirects and reloads when only the `#hash` changes |

`config/endpoints.js` is the only place JavaScript builds API URLs (stadiums, reservations, reviews, notifications); forms and action buttons use the URL the server rendered into their markup. `core/http.js` supports `withMeta` (paginated responses) and `keepalive`.

## Data attributes (contract between views and JS)

| Attribute | Used by |
| --- | --- |
| `data-ajax-form`, `data-method`, `data-on-success` | `ajax-form.js` |
| `data-action`, `data-method`, `data-confirm*`, `data-reason` | `action-buttons.js` |
| `data-open-dialog`, `data-fill`, `data-form-action`, `data-form-method`, `data-dialog-title`, `data-close-dialog` | `dialog-triggers.js` |
| `data-wheel-picker`, `data-mode`, `data-min`, `data-max`, `data-minute-step` | `wheel-picker.js` |
| `data-filter-form`, `data-filter-search` | `filters.js` |
| `data-booking-widget`, `data-config` | `booking-widget.js` |
| `data-map-view`, `data-map-picker` | map modules |
| `data-theme-toggle`, `data-nav-toggle`, `data-animate`, `data-stagger` | theme, sidebar, animations |
