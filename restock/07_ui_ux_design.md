# UX/UI Design

## Design Philosophy
The interface is designed for a single professional user, focusing on clarity, speed, and actionable data. It uses a clean, minimal theme with a white and blue color palette, avoiding the heavy visual load of traditional ERP systems.

## Main Application Pages

### 1. Upload Page
The entry point for the application. It features a prominent drag-and-drop area for the "ข้อมูล.xls/xlsx" file. Once uploaded, it displays file metadata and a high-level validation summary.

### 2. Validation Result Page
If the uploaded file has issues, this page provides a detailed breakdown. It categorizes errors into missing sheets, missing columns, mapping errors, and invalid data types, allowing the user to fix the source file before proceeding.

### 3. Reconciliation Dashboard
The central hub for analyzing results. It uses KPI cards to summarize the state of the warehouse:
- **Matched**: Items with zero variance (hidden by default).
- **Transfer Needed**: Items requiring movement to correct locations.
- **Pending Today Movement**: Items awaiting T+1 data.
- **Exceptions**: Mapping errors and substitution reviews.
- **Draft Variances**: Possible losses or gains.

### 4. Transfer Plan Page
A dedicated view for actionable stock movements. It lists the customer, product, and specific locations involved, along with a confidence score and the reason for the suggested transfer.

### 5. T+1 Recount Page
Displays the targeted list of items and locations that must be recounted. It prioritizes items based on movement volume and variance magnitude.

### 6. Export Page
A simple interface to download the results. Users can choose to download the full result workbook or specific subsets like the Transfer Plan or Recount List.

## Visual and Interaction Rules

| Element | Rule |
| :--- | :--- |
| **Default View** | Hide `MATCHED` rows to focus on exceptions. |
| **Color Coding** | **Red** for blocks/errors, **Yellow** for pending/review, **Green** for confirmed actions. |
| **Transparency** | Every row must display a clear "Reason" and "Confidence Score." |
| **Navigation** | Sidebar-based navigation for quick switching between modules. |
