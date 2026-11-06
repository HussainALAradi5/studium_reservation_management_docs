# City

Table `cities` — the cities where stadiums exist. Managed by admins.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | |
| `name` | `VARCHAR(80)` | no | 2–80 characters, unique within a country |
| `country_code` | `CHAR(2)` | no | ISO 3166-1 alpha-2, default `BH` |
| `is_active` | `TINYINT(1)` | no | Inactive cities are hidden from new addresses and search filters |

**Relations:** one-to-many Address.

**Rules:** a city that has addresses cannot be deleted (409 `CITY_IN_USE`); deactivate it instead.
