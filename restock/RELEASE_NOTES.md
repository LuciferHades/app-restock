# Restock Reconciliation - Release Notes v1.0.0

## Overview
This is the first production-ready release of the Restock Reconciliation Google Apps Script (GAS) application. It implements a complete end-to-end mobile-first physical inventory counting and baseline reconciliation workflow, fully integrated with Google Sheets as the canonical database.

## Key Features
- **Excel Upload & Data Staging:** Client-side SheetJS chunked uploading directly into Google Sheets, mitigating GAS execution time limits.
- **Strict Validation Engine:** Mandatory field validation, case-insensitive mapping, alias resolution, and ambiguous cross-customer matching blocks.
- **Mobile Count Session:** Touch-friendly, offline-draft-capable mobile UI with dynamic search and item card display (Expected Qty explicitly hidden to prevent bias).
- **Baseline Reconciliation Engine:** Day T processing generating `TRANSFER_PLAN` for exact ItemKey mismatches and tracking unresolved exceptions.
- **T+1 Movement Impact Tracking:** Re-evaluation of Day T counts against Day T+1 `INOUT` movements to generate targeted `RECOUNT_LIST` scopes.
- **Multi-Sheet Export:** Automated compilation of an 8-sheet formatted Excel workbook pushed directly to Google Drive.

## Technical Highlights
- 100% Google Apps Script + Vanilla JS/HTML.
- Heavily optimized for performance: O(1) dictionary lookups, `Set` matching, and pure batch operations (`getValues`/`setValues`).
- Adherence to data contracts: 14 strict canonical table structures defined and enforced.
