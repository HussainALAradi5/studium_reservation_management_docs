# Setting

Table `settings` — one row per [SettingsKey](../../enums/README.md#settingskey). Read through `SettingsService`, which falls back to the key's default and clamps to its range.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | prefix `SET` |
| `setting_key` | `VARCHAR(80)` | no | Unique; a `SettingsKey` value |
| `setting_value` | `VARCHAR(255)` | no | Stored as text, validated as an integer in the key's range |
