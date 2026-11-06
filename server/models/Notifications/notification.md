# Notification

Table `notifications` — one in-app message for one user (bell + inbox). Rows are written by `NotificationService` after the action that triggers them commits.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | prefix `NTF` |
| `user_id` | `INT UNSIGNED` FK → `users.id` | no | Recipient; `ON DELETE CASCADE` |
| `template` | `VARCHAR(60)` | no | [NotificationTemplate](../../enums/README.md#notificationtemplate) key, e.g. `reservation.confirmed` |
| `title` | `VARCHAR(120)` | no | Rendered once at send time |
| `body` | `VARCHAR(500)` | no | Rendered from the template's named placeholders |
| `related_type` | `VARCHAR(30)` | yes | `reservation` or `stadium` — decides the link |
| `related_code` | `VARCHAR(20)` | yes | Code of the related record |
| `read_at` | `DATETIME` | yes | Set when opened or by "Mark all read" |

**Indexes:** `(user_id, read_at, created_at)` for the unread count and inbox, `(related_type, related_code)`.

**Kind:** derived from the template (`booking`, `payment`, `review`, `alert`) and decides the icon and color; it is not stored.
