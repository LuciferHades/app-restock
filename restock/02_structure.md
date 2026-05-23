# System Structure

## Overview
The Restock Reconciliation Tool is structured as a modular web application designed for high-performance data processing. It follows a linear pipeline for Day T analysis and an iterative loop for T+1 refinement.

## Main Modules

| Module | Description |
| :--- | :--- |
| **Upload Module** | Handles file reception, extension validation, and job creation. |
| **Validation Module** | Ensures data integrity by checking for required sheets, columns, and valid data types. |
| **Mapping Module** | Standardizes raw data into internal keys (`ItemKey`, `LocationKey`). |
| **Reconciliation Engine** | The core logic that compares system snapshots against physical counts. |
| **T+1 Impact Engine** | Analyzes subsequent day movements to identify necessary recounts. |
| **Export Module** | Generates formatted Excel workbooks with actionable results. |
| **Result Dashboard** | Provides a visual summary of the reconciliation status and key metrics. |
| **Process Log** | Tracks job history and provides detailed error reporting for troubleshooting. |

## Module Details

### Upload Module
This module is responsible for receiving the "ข้อมูล.xls/xlsx" file. It generates a unique `job_id` for every upload and stores the file in a dedicated job directory to maintain a history of operations.

### Validation Module
Before processing, this module verifies that all mandatory sheets (CONFIG, ITEM_MASTER, INQUIRY, INOUT, PHYSICAL_COUNT) are present. It also checks for required columns, ensures quantities are numeric, and validates location formats to prevent downstream errors.

### Mapping Module
The Mapping Module translates raw item names into a standardized `ItemKey` based on the `ITEM_MASTER`. It normalizes attributes such as customer, product, pack size, crop year, and lot. Any items that cannot be mapped are flagged as `MAPPING_ERROR` and excluded from reconciliation to ensure data purity.

### Reconciliation Engine
This engine aggregates Inquiry and Physical Count data by `StockLocationKey`. It performs a multi-level comparison to detect exact matches, location mismatches, and variances that require further investigation or movement evidence from the In-Out logs.

### T+1 Movement Impact Engine
On the second day, this engine processes the Day T In-Out data. It identifies any "Movement Impact" where transactions occurred at specific locations during or after the count. It combines these impacted locations with unresolved variances from Day T to create a targeted `Recount List`.

## Module Connection Flow
1. **Upload** → **Validate** → **Mapping** → **Aggregate** → **Reconcile** → **Classify** → **Export (Draft)**
2. **Day T+1 Upload** → **Movement Impact Analysis** → **Recount List Generation** → **Final Action Plan**

## Constraints
- **No Raw Matching**: All reconciliation must be performed using standardized `ItemKey`, never raw names.
- **Customer Isolation**: Substitution logic is strictly prohibited across different customers.
- **Draft Finalization**: No "Loss" or "Over" status can be finalized without T+1 movement data.
