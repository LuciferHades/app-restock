# Test Cases

## Functional Test Scenarios

| ID | Scenario | Input Condition | Expected Outcome |
| :--- | :--- | :--- | :--- |
| **TC01** | **Exact Match** | Inquiry Qty = Physical Qty | Status = `MATCHED`. |
| **TC02** | **Location Mismatch** | Inquiry: Loc A=100, B=0; Physical: Loc A=70, B=30 | Status = `LOCATION_MISMATCH`; Transfer 30 from A to B. |
| **TC03** | **Pending Movement** | Day T variance exists; In-Out today not yet uploaded | Status = `PENDING_TODAY_MOVEMENT`. |
| **TC04** | **T+1 Impact (OUT)** | In-Out Day T shows OUT from Loc A | Loc A added to `Recount List`. |
| **TC05** | **T+1 Impact (XFER)** | In-Out Day T shows Transfer from A to B | Both A and B added to `Recount List`. |
| **TC06** | **RE T1 Restriction** | Cust A RE T1 -50; Cust B RE T1 +50 | Status = `RE_T1_BLOCKED`; No auto-offset. |
| **TC07** | **Mapping Failure** | Raw item name not in `ITEM_MASTER` | Status = `MAPPING_ERROR`; Excluded from totals. |
| **TC08** | **Duplicate Data** | Identical In-Out record appears twice | Flag `DUPLICATE_MOVEMENT_WARNING`. |
| **TC09** | **Missing Data** | Physical Count sheet missing `Qty` column | Validation fails; processing stops. |
| **TC10** | **Vague Movement** | Movement record missing specific location | All locations for that `ItemKey` added to `Recount List`. |

## Performance and Edge Cases
- **Large Dataset**: Process 50,000+ In-Out rows. System must complete within 10 seconds.
- **Special Characters**: Item names or locations containing symbols (e.g., `/`, `-`, `#`). System must handle without crashing.
- **Empty Sheets**: Optional sheets like `SUB_RULE` being empty. System must continue processing.
- **Zero Quantities**: Records with `Qty = 0`. System should ignore or flag based on context.
