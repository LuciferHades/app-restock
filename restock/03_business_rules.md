# Business Rules

## Core Data Principles

| Rule | Principle | Description |
| :--- | :--- | :--- |
| **Rule 1** | **Inquiry is Snapshot** | Inquiry represents the system stock at a specific timestamp. It must not be treated as a transaction or appended to existing balances. |
| **Rule 2** | **Physical is Truth** | The Physical Count is the absolute reference for on-hand stock, provided it passes mapping validation. |
| **Rule 3** | **In-Out is Evidence** | Transaction logs are used to explain variances but cannot replace Inquiry as the stock baseline due to the lack of a verified opening balance. |
| **Rule 4** | **Draft Status** | Any result generated on Day T is a "Draft." Final conclusions on stock loss or gain require T+1 movement data. |

## Reconciliation & Transfer Rules

### Rule 5: Transfer Eligibility
A transfer plan can only be generated if the following attributes match exactly between the source and destination:
- **Customer**: Must be the same entity.
- **Product**: Identical grade and name.
- **Specifications**: Same PackSize, CropYear, and Lot (if applicable).
- **Variance**: A negative variance in one location must be offset by a positive variance of the same item in another location.

### Rule 6: Substitution Restrictions
Automatic offsetting of variances between different customers is strictly forbidden. Specifically, for products categorized as **RE T1** or **Pure Sugar T1**, any cross-customer discrepancy must be flagged as `RE_T1_BLOCKED` or `SUBSTITUTION_REVIEW` for manual intervention.

### Rule 7: Mapping Integrity
Items that fail the mapping process are categorized as `MAPPING_ERROR`. If an item name maps to multiple keys, it is flagged as `AMBIGUOUS_MAPPING`. Both categories are excluded from reconciliation calculations to prevent data corruption.

## T+1 and Movement Rules

### Rule 8: Recount Logic
The Recount List for Day T+1 is the union of the **Movement Impact Set** (locations with transactions on Day T) and the **Unresolved Variance Set** (discrepancies from Day T that couldn't be explained).

### Rule 9: Movement Impact Mapping
The system identifies impacted locations based on the transaction type:
- **IN**: Recount the `ToLocation`.
- **OUT**: Recount the `FromLocation`.
- **TRANSFER**: Recount both `FromLocation` and `ToLocation`.
- **REJ**: Recount `FromLocation` and the designated `W8-REJ` area.
- **ADJUST**: Recount all affected locations.

### Rule 10: Quantity Standard
All calculations and transfer instructions must use the **Qty** field as the primary unit. The **TonQty** field is for informational purposes only and must not be used for generating action plans.
