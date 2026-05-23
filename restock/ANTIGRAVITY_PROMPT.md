# Prompt for Antigravity - Restock Reconciliation Tool

Copy and paste this prompt to Antigravity to continue implementation.

---

## Your Task

Continue implementation of the **Restock Reconciliation Tool** - a Google Apps Script Web App for mobile-first warehouse stock reconciliation.

**Current State:** Phase 1 GAS skeleton complete. All backend stubs and frontend pages created. Ready for Phase 1B QA.

**Your Role:** Implement Phases 1B through 7 following the specifications and checklists provided.

## Source of Truth

1. **Business Rules:** `/docs/03_business_rules.md`
2. **Reconciliation Logic:** `/docs/05_reconciliation_logic.md`
3. **T+1 Logic:** `/docs/06_t_plus_1_recount_logic.md`
4. **Handoff Document:** `HANDOFF_ANTIGRAVITY.md`
5. **Project Status:** `PROJECT_STATUS.md`
6. **Phase Checklists:** `NEXT_STEPS.md`
7. **File Purposes:** `FILE_MANIFEST.md`

## Current Phase: Phase 1B - GAS Skeleton QA

**Goal:** Verify Phase 1 skeleton works before SheetJS integration.

**Acceptance Criteria:**
- Web app loads without errors
- setupDatabase() creates all 16 sheets with headers
- All backend wrapper functions exist in Code.gs
- All include() calls load correctly
- Mobile layout verified (48px+ buttons, no overflow)
- No broken JS references

**Checklist:**
- [ ] Verify Code.gs doGet() renders index.html
- [ ] Verify include() loads all 5 HTML partials (css, js-app, js-upload, js-count-mobile, js-results)
- [ ] Verify setupDatabase() creates all 16 sheets with correct headers
- [ ] Verify all 16 backend wrapper functions exist in Code.gs
- [ ] Verify mobile buttons are 48px+ height
- [ ] Verify no horizontal overflow on mobile width
- [ ] Verify navigation between pages works
- [ ] Verify localStorage persistence works
- [ ] Test on actual mobile browser if possible

**Do NOT proceed to Phase 2 until Phase 1B QA passes.**

## Next Phase: Phase 2 - Upload + SheetJS Chunk Import

**Goal:** Implement Excel file upload with SheetJS parsing and chunked import.

**Key Requirements:**
- Parse Excel file with SheetJS (read all 5 sheets: CONFIG, ITEM_MASTER, INQUIRY, INOUT, PHYSICAL_COUNT)
- Validate sheet names and required columns
- Split data into 500-row chunks
- Batch write chunks to UPLOAD_RAW sheet
- Show upload progress bar
- Handle 50k+ rows efficiently

**See NEXT_STEPS.md for complete Phase 2 checklist.**

## Important Rules

1. **Use /docs as source of truth** - All business logic is defined there
2. **Report delta only** - Don't restate full requirements, only report what changed
3. **Do not implement Phase 2 until Phase 1B QA passes** - Verify skeleton first
4. **Do not add features outside scope** - No approval workflows, no role control, no ERP integration
5. **Do not duplicate business logic** - Implement once in GAS backend, frontend only calls APIs
6. **Batch Sheets operations** - Never loop row-by-row, use getValues()/setValues()
7. **Mobile-first** - Test on actual mobile devices, not just desktop browser

## Key Business Rules to Remember

**ItemKey Format:** `customer_code|product_grade|pack_size|crop_year|lot`

**StockLocationKey Format:** `warehouse|location|ItemKey`

**RE T1 Cross-Customer Blocking:** Use SubstitutionCandidateKey (without customer) to detect and block cross-customer substitution for RE T1 and Pure Sugar T1 items.

**Reconciliation Statuses (8 total):**
- MATCHED (hide by default)
- LOCATION_MISMATCH
- TRANSFER_NEEDED
- PENDING_TODAY_MOVEMENT
- RE_T1_BLOCKED
- MAPPING_ERROR
- POSSIBLE_LOSS_DRAFT
- POSSIBLE_OVER_DRAFT

**Day T vs Day T+1:**
- Day T: Baseline reconciliation (draft only)
- Day T+1: Upload movements, generate targeted recount list
- No final loss/over before T+1 analysis

**Qty Only:** Use Qty field only, ignore TonQty. All calculations in base units.

## Project Structure

```
restock_reconciliation_gas/
├── Backend Services (.gs files - 13 total)
│   ├── Code.gs (main entry point, API wrappers)
│   ├── Config.gs (configuration)
│   ├── Utils.gs (helpers)
│   ├── SheetService.gs (Sheets operations)
│   ├── UploadService.gs (Phase 2)
│   ├── ValidationService.gs (Phase 3)
│   ├── MappingService.gs (Phase 3)
│   ├── ReconciliationEngine.gs (Phase 5)
│   ├── TPlusOneEngine.gs (Phase 6)
│   ├── CountSessionService.gs (Phase 4)
│   ├── ExportService.gs (Phase 7)
│   ├── LogService.gs (Phase 3)
│   └── appsscript.json
├── Frontend Files (.html files - 7 total)
│   ├── index.html (main shell)
│   ├── css.html (styles)
│   ├── js-app.html (routing)
│   ├── js-upload.html (Phase 2)
│   ├── js-count-mobile.html (Phase 4)
│   └── js-results.html (Phase 5)
└── Documentation
    ├── HANDOFF_ANTIGRAVITY.md (this handoff)
    ├── PROJECT_STATUS.md (current state)
    ├── NEXT_STEPS.md (phase checklists)
    ├── FILE_MANIFEST.md (file purposes)
```

## Database Schema (16 Sheets)

All sheets are created by setupDatabase(). See PROJECT_STATUS.md for full schema.

Key sheets:
- JOBS - Job tracking
- UPLOAD_RAW - Raw uploaded data chunks
- ITEM_MASTER - Item definitions
- INQUIRY - System stock snapshot
- INOUT - Movement history
- PHYSICAL_COUNT - Physical count results
- RECON_RESULT - Reconciliation results
- TRANSFER_PLAN - Transfer suggestions
- RECOUNT_LIST - T+1 targeted recount items

## Backend API Wrappers (16 total)

All defined in Code.gs, ready to implement:

**Upload Phase (2):**
- createUploadJob(metadata)
- uploadParsedChunk(jobId, sheetName, rows, chunkIndex)
- finalizeUpload(jobId)

**Validation Phase (3):**
- validateJob(jobId)

**Count Session Phase (4):**
- createCountSession(jobId)
- getCountLocations(sessionId)
- getCountItemsByLocation(sessionId, location)
- saveCountDraft(sessionId, location, rows)
- submitCountLocation(sessionId, location, rows)

**Reconciliation Phase (5):**
- runBaselineReconciliation(jobId)

**T+1 Phase (6):**
- uploadTPlusOneChunk(jobId, rows, chunkIndex)
- generateRecountList(jobId)

**Results & Export Phase (7):**
- getResults(jobId, filter)
- exportResultWorkbook(jobId)

**Config:**
- getAppConfig()
- setupDatabase()

## Performance Requirements

- Batch read/write with getValues()/setValues()
- No row-by-row SpreadsheetApp calls in loops
- Build maps/indexes before reconciliation
- Chunk upload large data (500 rows per chunk)
- Handle 50k+ rows efficiently
- No nested loops against In-Out data

## Mobile Requirements

- Responsive layout (tested on actual mobile devices)
- Large touch buttons (48px minimum height)
- Location-first counting workflow
- Save draft to localStorage
- Show sync status indicator
- Hide MATCHED rows by default
- No horizontal overflow

## Testing

See NEXT_STEPS.md for test cases for each phase.

Key test scenarios:
- Upload 50k+ rows
- Validate with missing sheets/columns
- Map items correctly
- Reconcile with exact matches
- Detect location mismatches
- Generate transfer plans
- Block RE T1 cross-customer
- Generate T+1 recount lists
- Export workbooks

## Deployment

1. Create Google Apps Script project at script.google.com
2. Copy all .gs files to project
3. Copy all .html files as HTML files in GAS
4. Update appsscript.json
5. Create Google Sheet, copy ID to Config.gs
6. Deploy as Web App
7. Open Web App URL on mobile browser

See README_GAS.md for detailed deployment steps.

## Questions?

1. Check /docs for business rules
2. Check HANDOFF_ANTIGRAVITY.md for project overview
3. Check PROJECT_STATUS.md for current state
4. Check NEXT_STEPS.md for phase requirements
5. Check FILE_MANIFEST.md for file purposes

## Reporting

When you complete each phase:
- Report delta only (what changed, not full summary)
- Report blockers if any
- Report ready/not ready for next phase
- Do not restate requirements
- Do not repeat previous work

---

**Ready to start Phase 1B QA?**

Begin by verifying the Phase 1B acceptance criteria listed above.
