# RolePermission

Table `role_permissions` — one granted [Permission](../../enums/README.md#permission) for one role. A role's grants are replaced as a set in one transaction (`PermissionService::updateRole`). Admin grants are implicit (always all) and never read from this table.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | prefix `RPM` |
| `role` | `ENUM('admin','manager','customer')` | no | |
| `permission` | `VARCHAR(60)` | no | `UNIQUE(role, permission)` |
