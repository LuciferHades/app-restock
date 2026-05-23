# Data Contract

## Input File Specification
The system expects a single Excel file named **"ข้อมูล.xls"** or **"ข้อมูล.xlsx"**. This file must contain several mandatory sheets, each following a specific schema.

## Required Sheets and Columns

### 1. CONFIG
This sheet defines the parameters for the current reconciliation job.
- `batch_id`: Unique identifier for the batch.
- `business_date`: The date the reconciliation is performed.
- `inquiry_datetime`: The exact timestamp of the system snapshot.
- `count_date`: The date the physical count was conducted.
- `inout_end_date`: The last date included in the In-Out evidence log.
- `mode`: Operation mode (e.g., "DAY_T" or "DAY_T_PLUS_1").

### 2. ITEM_MASTER
The source of truth for item mapping and normalization.
- `item_key`: Unique identifier for the item.
- `customer_code`, `customer_name`: Customer details.
- `product_grade`, `product_name`: Product specifications.
- `pack_size`, `crop_year`, `lot`: Item attributes.
- `alias_name`: Alternative names used in raw data.
- `active_flag`: Status of the item.

### 3. INQUIRY
The system stock snapshot.
- `wh`, `location`: Storage details.
- `item_name`: Raw item name.
- `customer_code`, `product_grade`, `pack_size`, `crop_year`, `lot`: Attributes for mapping.
- `qty`: System quantity.

### 4. INOUT
The transaction log for evidence and impact analysis.
- `doc_date`, `doc_no`: Transaction identification.
- `movement_type`: Type of movement (IN, OUT, TRANSFER, etc.).
- `wh`, `from_location`, `to_location`: Movement path.
- `item_name`, `customer_code`, `product_grade`, `pack_size`, `crop_year`, `lot`: Item attributes.
- `qty`: Transaction quantity.

### 5. PHYSICAL_COUNT
The actual count from the warehouse floor.
- `wh`, `location`, `physical_pile`: Location details.
- `item_name`, `customer_code`, `product_grade`, `pack_size`, `crop_year`, `lot`: Item attributes.
- `qty`: Physical quantity.
- `remark`: Optional notes from the counter.

## Key Definitions

| Key Name | Composition |
| :--- | :--- |
| **ItemKey** | `customer_code` + `product_grade` + `pack_size` + `crop_year` + `lot` |
| **StockLocationKey** | `wh` + `location` + `ItemKey` |

## Validation Rules
- **Mandatory Fields**: `customer_code`, `product_grade`, `crop_year`, and `location` (for Inquiry/Count) must not be empty.
- **Data Types**: All `qty` fields must be numeric.
- **Uniqueness**: `batch_id` in CONFIG should be unique across jobs.
