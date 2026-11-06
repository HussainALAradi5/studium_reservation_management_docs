# Address

Table `addresses` — where a stadium is: city, street, and optional map coordinates.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | |
| `city_id` | `INT UNSIGNED` FK → `cities.id` | no | Must be an active city |
| `street` | `VARCHAR(120)` | no | Road name or number, 2–120 |
| `building` | `VARCHAR(20)` | yes | Building number |
| `block` | `VARCHAR(20)` | yes | Block / district number |
| `area` | `VARCHAR(80)` | yes | Neighbourhood |
| `latitude` | `DECIMAL(10,7)` | yes | −90 … 90, set with the map picker |
| `longitude` | `DECIMAL(10,7)` | yes | −180 … 180, set with the map picker |

Latitude and longitude are both set or both empty.

**Relations:** many-to-one City; one-to-one Stadium. The stadium owns its address: both are created and updated together in one transaction.

**Display format:** `Bldg {building}, Road {street}, Block {block}, {area}, {city}` — empty parts are skipped (`utilities/Formatting/AddressFormatter.php`).
