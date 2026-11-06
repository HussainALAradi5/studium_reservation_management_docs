# Expense

Table `expenses` — running costs of a stadium, used for profit reporting.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | |
| `stadium_id` | `INT UNSIGNED` FK | no | A stadium the actor manages (admin: any) |
| `category` | `ENUM` | no | [ExpenseCategory](../../enums/README.md#expensecategory) |
| `amount` | `DECIMAL(10,3)` | no | > 0 |
| `expense_date` | `DATE` | no | Not in the future |
| `description` | `VARCHAR(255)` | no | 3–255 |

**Relations:** many-to-one Stadium. Expenses can be soft-deleted by the stadium's manager or an admin.
