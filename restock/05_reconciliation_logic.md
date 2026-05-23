# Reconciliation Logic

## Processing Pipeline
The reconciliation engine follows a structured sequence to ensure data accuracy and rule compliance:
1. **Data Ingestion**: Load the Excel workbook and validate the presence of required sheets.
2. **Normalization**: Standardize text (trimming, case sensitivity) and location formats.
3. **Key Construction**: Build `ItemKey` and `StockLocationKey` for all records.
4. **Aggregation**: Sum quantities by `StockLocationKey` for both Inquiry and Physical Count.
5. **Comparison**: Calculate `VarianceQty = PhysicalQty - InquiryQty`.
6. **Classification**: Assign a status to each record based on the variance and business rules.
7. **Evidence Linking**: Attach relevant In-Out transactions to unexplained variances.
8. **Scoring**: Assign a confidence score to each identified action or status.

## Status Classification Logic

| Status | Condition | Action/Meaning |
| :--- | :--- | :--- |
| **MATCHED** | `VarianceQty = 0` | System and Physical counts are identical. |
| **LOCATION_MISMATCH** | Total Item Qty matches, but location Qty differs. | Stock is in the wrong place; needs transfer. |
| **TRANSFER_NEEDED** | Negative variance in Loc A matches Positive in Loc B for same `ItemKey`. | Generate transfer instruction. |
| **PENDING_TODAY_MOVEMENT** | Variance exists and `inout_end_date < count_date`. | Wait for T+1 In-Out data. |
| **SUBSTITUTION_REVIEW** | Same product, different customer variance. | Manual review required; auto-offset blocked. |
| **RE_T1_BLOCKED** | Product is "RE T1" and customers differ. | Strict block on auto-substitution. |
| **MAPPING_ERROR** | Item cannot be mapped to `ItemKey`. | Exclude from reconciliation; fix master data. |
| **POSSIBLE_LOSS_DRAFT** | `Physical < Inquiry` on Day T. | Potential loss; pending T+1 confirmation. |
| **POSSIBLE_OVER_DRAFT** | `Physical > Inquiry` on Day T. | Potential gain; pending T+1 confirmation. |

## Confidence Scoring
The system assigns a confidence score (0-100) to help the user prioritize actions:
- **95**: Exact transfer matches within the same item/customer.
- **85-95**: Discrepancies clearly explained by T+1 movement data.
- **40-60**: Variances likely due to pending today's movements.
- **20-50**: Substitution reviews requiring human judgment.
- **0**: Mapping errors or data corruption.
