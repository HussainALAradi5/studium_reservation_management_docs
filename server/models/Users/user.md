# User

Table `users` — any person who signs in. One table with a `role` column, because the three roles share the same identity fields and differ only in permissions.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | |
| `role` | `ENUM('admin','manager','customer')` | no | [UserRole](../../enums/README.md#userrole); self-registration always creates `customer` |
| `full_name` | `VARCHAR(120)` | no | 3–120 characters |
| `username` | `VARCHAR(40)` | no | Unique, 3–40, letters, digits, `.`, `_` |
| `email` | `VARCHAR(160)` | no | Unique, valid email, stored lower-case |
| `phone` | `VARCHAR(20)` | no | Unique, E.164 style (`+97333123456`), 8–15 digits |
| `password_hash` | `VARCHAR(255)` | no | bcrypt via `password_hash`; never sent to views or JSON |
| `gender` | `ENUM('male','female')` | yes | [Gender](../../enums/README.md#gender) |
| `birth_date` | `DATE` | yes | In the past |
| `is_active` | `TINYINT(1)` | no | Inactive users cannot sign in |
| `last_login_at` | `DATETIME` | yes | Set on successful login |

**Relations:** manages many Stadiums (`stadiums.manager_id`, managers only); books many Reservations (`reservations.customer_id`).

**Rules:**
- An admin cannot deactivate themself or change their own role.
- A manager who still manages stadiums cannot be given another role until the stadiums are reassigned.
