# Restock Reconciliation Tool - Google Apps Script

Mobile-first warehouse stock reconciliation using Google Apps Script.

## Architecture

- **Backend**: Google Apps Script (.gs services)
- **Database**: Google Sheets (16 sheets)
- **Storage**: Google Drive
- **Frontend**: HTML / Vanilla CSS / Vanilla JS (responsive mobile UI)
- **Deployment**: GAS Web App

## Project Structure (26 files)

**Backend (.gs files):**
- `appsscript.json` — GAS project manifest
- `Code.gs` — Main entry point & doGet()
- `Config.gs` — Configuration constants & sheet names
- `Utils.gs` — Helper functions (include, generateJobId, logging)
- `SheetService.gs` — Google Sheets batch read/write operations
- `UploadService.gs` — Excel upload handling *(Phase 2 stub)*
- `ValidationService.gs` — Data validation *(Phase 3 stub)*
- `MappingService.gs` — Item/location key construction *(Phase 3 stub)*
- `ReconciliationEngine.gs` — Day T reconciliation *(Phase 5 stub)*
- `TPlusOneEngine.gs` — Day T+1 movement analysis *(Phase 6 stub)*
- `CountSessionService.gs` — Mobile count session management *(Phase 4 stub)*
- `ExportService.gs` — Excel export *(Phase 7 stub)*
- `LogService.gs` — Centralised validation logging *(Phase 3 stub)*

**Frontend (.html files):**
- `index.html` — Main app shell & navigation
- `css.html` — Mobile-first responsive CSS (vanilla)
- `js-app.html` — App routing, state management, localStorage
- `js-upload.html` — Upload page *(Phase 2 stub)*
- `js-count-mobile.html` — Mobile count UI *(Phase 4 stub)*
- `js-results.html` — Results display *(Phase 5 stub)*

**Documentation:**
- `README.md` — Project overview (this file)
- `README_GAS.md` — Deployment guide
- `HANDOFF_ANTIGRAVITY.md` — Agent handoff document
- `FILE_MANIFEST.md` — File purposes
- `NEXT_STEPS.md` — Phase checklists
- `PROJECT_STATUS.md` — Current implementation status

## Current Phase: 1B QA

Phase 1 skeleton is complete. All services are stubs pending Phase 2–7.

### What works (skeleton):
- ✅ Web app loads and renders
- ✅ 5-page navigation (Upload, Count, Results, T+1, Export)
- ✅ App state management (currentJob, currentSession, localStorage)
- ✅ Sync status indicator
- ✅ Loading overlay and error/success toasts
- ✅ setupDatabase() creates all 16 sheets with headers
- ✅ All backend API wrapper functions exist (stubs)

### Not yet implemented:
- ❌ Excel parsing (Phase 2)
- ❌ Data validation (Phase 3)
- ❌ Mobile count session (Phase 4)
- ❌ Reconciliation engine (Phase 5)
- ❌ T+1 analysis (Phase 6)
- ❌ Excel export (Phase 7)

## Deployment

See `README_GAS.md` for step-by-step instructions.

### Quick summary:
1. Create Google Apps Script project at script.google.com
2. Copy all `.gs` and `.html` files to the project
3. Replace `appsscript.json` manifest
4. Create a Google Sheet; set its ID via `PropertiesService.getUserProperties().setProperty('SPREADSHEET_ID', '<id>')`
5. Run `setupDatabase()` once to create all 16 sheets
6. Deploy as Web App → Execute as: Your account, Access: Anyone
7. Open the Web App URL on mobile browser

## Performance Principles

- Batch read with `getValues()` — no row-by-row SpreadsheetApp calls
- Batch write with `setValues()` / `appendRow` only for single rows
- Build lookup maps before reconciliation — no nested loops
- Chunk uploads at 500 rows per chunk for large Excel files

## Business Rules (summary)

- Inquiry is a snapshot at Day T — not a running balance
- Day T result is always draft — no final conclusions before T+1
- Transfer only allowed for exact same ItemKey (same customer, grade, spec)
- RE T1 / Pure Sugar T1 cross-customer substitution is strictly blocked
- All calculations use Qty field only — TonQty is informational
- Items with mapping errors are excluded from reconciliation

## Development Phases

| Phase | Status | Description |
|-------|--------|-------------|
| 1 | ✅ Done | Skeleton + mobile UI + service stubs |
| 1B | ✅ Done | Skeleton QA — all wiring verified |
| 2 | ⬜ Next | Upload + SheetJS chunk import |
| 3 | ⬜ | Validation + Mapping |
| 4 | ⬜ | Count session mobile UI |
| 5 | ⬜ | Reconciliation engine |
| 6 | ⬜ | T+1 recount |
| 7 | ⬜ | Export |
