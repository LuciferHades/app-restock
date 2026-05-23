# Assumptions and Open Questions

## Project Assumptions
The following assumptions have been made to define the initial scope and technical approach:
- **Single User Environment**: The system will run locally or on a private server for one person; no authentication or multi-tenancy is required.
- **File-Based Workflow**: All data exchange happens via the "ข้อมูล.xls/xlsx" file; no database persistence is needed beyond job artifacts.
- **Data Availability**: In-Out data for the count day is always available the following morning.
- **Primary Unit**: The `Qty` field is the only unit used for reconciliation and action plans.
- **Item Attributes**: Attributes like `Lot` are optional; if missing, the system treats the item as having no lot-specific requirements.
- **Snapshot Nature**: Inquiry data is a point-in-time snapshot and does not include transaction history.

## Open Questions for Clarification
To ensure the system perfectly matches the operational reality, the following questions should be addressed:
1. **Sheet Naming**: Are the sheet names in the Excel file exactly as specified (e.g., `INQUIRY`, `INOUT`), or do they vary?
2. **Column Headers**: What are the exact column names used in the source system for Inquiry, In-Out, and Physical Count?
3. **RE T1 Variations**: How many different ways is the "RE T1" product grade written in the raw data?
4. **Customer Identification**: Is the `CustomerCode` always present as a separate column, or is it sometimes embedded in the item name?
5. **Location Format**: What is the typical format for locations (e.g., `A-01-01`)? Are there any special characters to handle?
6. **Movement Types**: What are the exact strings used for movement types (e.g., `Issue`, `Receipt`, `Transfer`)?
7. **Export Requirements**: Does the final Excel output need to be compatible with any specific downstream system for automated adjustments?
8. **Lot Usage**: Is the `Lot` field consistently used for all products, or only for specific customers?
9. **Warehouse Scope**: Will a single file ever contain data for multiple warehouses that need to be reconciled separately?
10. **Language**: Should the UI and export reports be in English, Thai, or a combination of both?
