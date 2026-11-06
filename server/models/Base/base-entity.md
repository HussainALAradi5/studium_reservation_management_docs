# BaseEntity

Abstract parent of every persisted model (`models/Base/BaseEntity.php`). The repository base (`repositories/Base/BaseRepository.php`) fills these fields automatically; services never set them by hand.

| Column | Type | Null | Set by | Notes |
| --- | --- | --- | --- | --- |
| `id` | `INT UNSIGNED AUTO_INCREMENT` | no | database | Primary key, internal only |
| `code` | `VARCHAR(20)` | no | `CodeGenerator` | Unique public identifier used in URLs (`STD-7KQ2M9XA`) |
| `created_at` | `DATETIME` | no | repository insert | Server time |
| `updated_at` | `DATETIME` | no | repository insert/update | Server time |
| `created_by` | `INT UNSIGNED` | yes | repository insert | Signed-in user id; null for self-registration and seeds |
| `updated_by` | `INT UNSIGNED` | yes | repository update | Signed-in user id |
| `deleted_at` | `DATETIME` | yes | repository soft delete | Non-null rows are hidden by default |
| `deleted_by` | `INT UNSIGNED` | yes | repository soft delete | Who deleted |

## Code prefixes

| Model | Prefix | Example |
| --- | --- | --- |
| User | `USR` | `USR-4H8K2M1Q` |
| City | `CTY` | `CTY-9P3Q7Z2D` |
| Address | `ADR` | `ADR-2B6N8C4F` |
| Stadium | `STD` | `STD-7KQ2M9XA` |
| Reservation | `RSV` | `RSV-5T1W3E8R` |
| Payment | `PAY` | `PAY-6Y2U9H4P` |
| Expense | `EXP` | `EXP-2A7S5D3G` |

Codes use 8 characters from an unambiguous alphabet (no `0/O/1/I`) generated with `random_int`; the `UNIQUE(code)` constraint is the final guarantee.
