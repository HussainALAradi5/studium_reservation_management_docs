# SettingsService — `services/Settings/SettingsService.php`

### `int(SettingsKey $key): int`
Stored value clamped to the key's range; the key's default when missing or not numeric. Values are loaded once per request.

### `bool(SettingsKey $key): bool`
`int($key) === 1`.

### `all(): array`
Effective value of every key, for the settings form.

### `update(SettingsFormDto $dto): void`
Upserts every key in one transaction and clears the cache. Validation (each key against its own range) happens in the DTO → 422.
