# Export Output Specification

## Output Workbook Overview
The system generates a multi-sheet Excel workbook named `restock_result_{batch_id}_{date}.xlsx`. This file serves as the primary document for warehouse operations and system adjustments.

## Sheet Specifications

### 01_SUMMARY
Provides a high-level overview of the reconciliation results.
- **Columns**: `Metric`, `Value`, `Note`.
- **Key Metrics**: Total Items, Matched Items, Total Variance Qty, Number of Transfers, Number of Recounts.

### 02_TRANSFER_PLAN
Instructions for moving stock to correct locations.
- **Columns**: `ActionType`, `CustomerCode`, `ProductGrade`, `PackSize`, `CropYear`, `Lot`, `Qty`, `FromLocation`, `ToLocation`, `Reason`, `Confidence`.

### 03_T_PLUS_1_RECOUNT_LIST
The targeted list for Day T+1 physical verification.
- **Columns**: `Priority`, `CustomerCode`, `ProductGrade`, `PackSize`, `CropYear`, `Lot`, `Location`, `Reason`, `MovementType`, `MovementQty`, `DayT_Variance`.

### 04_MOVEMENT_IMPACT
Detailed log of transactions that occurred on the day of the count.
- **Columns**: `DocDate`, `DocNo`, `MovementType`, `CustomerCode`, `ProductGrade`, `PackSize`, `CropYear`, `Lot`, `FromLocation`, `ToLocation`, `ImpactedLocation`, `Qty`.

### 05_FINAL_ACTION_PLAN
The definitive list of adjustments after all evidence and recounts are processed.
- **Columns**: `FinalStatus`, `ActionType`, `CustomerCode`, `ProductGrade`, `CropYear`, `Lot`, `Qty`, `FromLocation`, `ToLocation`, `Reason`, `Confidence`.

### 06_MAPPING_ERROR
A list of items that could not be processed due to data issues.
- **Columns**: `SourceSheet`, `RowNumber`, `RawItemName`, `CustomerCode`, `ProductGrade`, `CropYear`, `ErrorType`, `ErrorMessage`.

### 07_INOUT_EVIDENCE
Supporting transaction data linked to specific variances.
- **Columns**: `Status`, `DocDate`, `DocNo`, `MovementType`, `CustomerCode`, `ProductGrade`, `CropYear`, `Lot`, `Location`, `Qty`, `RelatedVariance`.

## Formatting Rules
- **Headers**: Bold text with a light blue background.
- **Auto-filter**: Enabled for all sheets.
- **Conditional Formatting**: Red text for negative variances, green for matches.
- **Freeze Panes**: Top row frozen for all data sheets.
