# Payment

Table `payments` — money received for, or refunded from, a reservation. Append-only ledger: rows are never edited or deleted; a mistake is corrected with an opposite entry.

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| base columns | see [BaseEntity](../Base/base-entity.md) | | `created_by` = the manager/admin who recorded it |
| `reservation_id` | `INT UNSIGNED` FK | no | |
| `type` | `ENUM('payment','refund')` | no | [TransactionType](../../enums/README.md#transactiontype) |
| `method` | `ENUM('cash','card','bank_transfer','online')` | no | [PaymentMethod](../../enums/README.md#paymentmethod) |
| `amount` | `DECIMAL(10,3)` | no | > 0; payment ≤ outstanding balance; refund ≤ paid amount |
| `paid_at` | `DATETIME` | no | When the money moved; not in the future |
| `reference` | `VARCHAR(80)` | yes | Receipt / transaction number |
| `notes` | `VARCHAR(255)` | yes | |

**Relations:** many-to-one Reservation. See [accounting rules](../../../workflows/accounting.md).
