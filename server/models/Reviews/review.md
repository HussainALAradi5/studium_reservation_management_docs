# Review

Table `reviews` — a customer's star rating and comment for a stadium they played at. **One per customer and stadium, forever:** there is no edit, and a review removed by a moderator still blocks a new one.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | prefix `REV` |
| `stadium_id` | `INT UNSIGNED` FK → `stadiums.id` | no | |
| `customer_id` | `INT UNSIGNED` FK → `users.id` | no | Must have a completed booking (or a confirmed one that has ended) at the stadium |
| `rating` | `TINYINT UNSIGNED` | no | 1–5 |
| `comment` | `VARCHAR(1000)` | no | 3–1000 characters, whitespace collapsed |

**Constraints:** `UNIQUE(stadium_id, customer_id)` — the database rejects a second review even if two requests race.

**Denormalized rating:** `stadiums.rating_average` (`DECIMAL(3,2)`) and `stadiums.rating_count` are recomputed in the same transaction whenever a review is added or removed, so lists never aggregate on the fly.

**Visibility:** comments are returned only to signed-in users; guests see the average and count.
