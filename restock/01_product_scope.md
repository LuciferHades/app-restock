# Product Scope

## In Scope
- **Single File Upload**: Support for `.xls` and `.xlsx` files containing multiple required sheets.
- **Data Validation**: Automated checks for file format, required sheets, columns, data types, and duplicates.
- **Item Mapping**: Normalization of raw item names into standardized `ItemKey` using a master list.
- **Location Normalization**: Standardizing warehouse and location formats for accurate matching.
- **Reconciliation Engine**: Comparing Inquiry vs. Physical Count at both item and location levels.
- **Mismatch Detection**: Identifying items that exist in the wrong locations but have correct total quantities.
- **Transfer Plan Generation**: Creating specific instructions to move stock to correct locations.
- **Evidence Lookup**: Linking In-Out transactions to explain variances.
- **T+1 Recount Logic**: Generating a focused list of items/locations to recount based on Day T movements.
- **Excel Export**: Providing multi-sheet results for further manual work.

## Out of Scope
- **Multi-user Support**: No role-based access control or user management.
- **Approval Workflow**: No multi-stage approval for transfers or adjustments.
- **ERP Integration**: No direct API calls to or from external ERP/WMS systems.
- **Real-time Sync**: No live data updates; all processing is batch-based via file upload.
- **Capacity Planning**: No warehouse space optimization or capacity simulation.
- **Cross-Warehouse**: Limited to single warehouse reconciliation per job.
- **REJ Flow**: No specialized handling for rejected goods beyond basic movement tracking.
- **Mobile/Scanning**: No mobile app, barcode scanning, OCR, or AI image recognition.

## User Type
- **Single User**: Designed for an individual operator or admin to use as a personal productivity tool.
- **Web-based**: Accessible via a standard web browser.

## Success Criteria
- **Efficiency**: Ability to process 50,000+ In-Out rows quickly without system timeout.
- **Accuracy**: Correct identification of "RE T1" substitution blocks and customer-specific rules.
- **Actionability**: Output clearly distinguishes between what needs to be transferred, recounted, or corrected in mapping.
