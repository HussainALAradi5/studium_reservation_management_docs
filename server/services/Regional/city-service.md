# CityService — `services/Regional/CityService.php`

### `list(CityQuery $query): PageResult`
Paginated cities with a stadium count per city (one sub-select, no N+1).

### `listActive(): City[]`
Active cities for selects and filters.

### `findByCode(string $code): City`
**Exceptions:** 404 when missing or soft-deleted.

### `findActiveByCode(string $code): City`
**Purpose:** City for a new or moved stadium address. **Exceptions:** 404; 409 with `fields.cityCode` when inactive.

### `create(CityFormDto $dto): City`
**Logic:** name unique per country among non-deleted cities → insert. **Exceptions:** 409 `DUPLICATE_VALUE` on `name`.

### `update(string $code, CityFormDto $dto): City`
Same uniqueness check excluding itself. **Exceptions:** 404, 409.

### `delete(string $code): void`
**Logic:** refuses when any address uses the city, otherwise soft-deletes. **Exceptions:** 404, 409 `CITY_IN_USE`.
