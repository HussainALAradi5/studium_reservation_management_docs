# Phase 05: Client foundation

**Status:** Done
**Depends on:** Phase 04

## Goal
Vanilla CSS design system, generic partials, and the JS core (HTTP client, AJAX forms, dialogs, toasts).

## Scope
| Layer | Folders | Items |
| --- | --- | --- |
| CSS | `assets/css/` | Tokens, glass surfaces, dark/light, animations |
| Partials | `views/partials/` | customButton, badge, genericForm, genericField, dateField, genericFilter, table, card, dialog |
| JS | `assets/javascripts/core`, `forms`, `ui`, `config` | http, button, toast, dialog, ajax-form, wheel picker, theme, filters |

## Tasks
- [x] Endpoint map mirrors server routes
- [x] Button spinner while processing
- [x] Wheel picker (iOS/Android)

## Acceptance criteria
- Client signs in against the running server

## Verification
| Check | Command | Result |
| --- | --- | --- |
| Browser check | In-app browser | Dark/light, wheel picker, filters work; no console errors, 2026-11-07 |
