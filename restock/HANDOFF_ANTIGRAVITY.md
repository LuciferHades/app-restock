# Restock Reconciliation Tool - Handoff to Antigravity

## Project Goal

Build a mobile-first warehouse stock reconciliation tool using Google Apps Script. The tool helps warehouse staff count physical stock, compare against system records (Inquiry), and generate transfer/recount plans with minimal manual intervention.

## Architecture Decision

**Final Stack:**
- Backend: Google Apps Script (.gs services)
- Database: Google Sheets
- Storage: Google Drive
- Frontend: HTML/CSS/Vanilla JS (responsive mobile UI)
- Deployment: GAS Web App (no localhost, no external backend)

**Why we pivoted from FastAPI to GAS:**
- Mobile-first requirement: GAS Web App deploys instantly to mobile browsers
- No backend hosting needed: Runs entirely in Google ecosystem
- No local dependencies: Users don't need to install/run servers
- Batch Sheets operations: Built-in for handling 50k+ rows efficiently
- Offline-capable: localStorage for draft saving before sync

## Current Phase Completed

**Phase 1: GAS Skeleton + Mobile UI (✅ Complete)**

**Deliverables:**
- 13 .gs backend files (services + main entry)
- 7 .html frontend files (mobile-responsive UI)
- 2 documentation files
- appsscript.json manifest
- All 16 Google Sheets database schemas defined

**What's Implemented:**
- ✅ doGet() web app entry point
- ✅ include() HTML template composition
- ✅ Mobile-first responsive layout (48px+ buttons)
- ✅ 5-page navigation (Upload, Count, Results, T+1, Export)
- ✅ App state management (currentJob, currentSession, localStorage)
- ✅ Sync status indicator
- ✅ Loading/error/success toasts
- ✅ SheetService with batch operations (getValues/setValues)
- ✅ setupDatabase() creates all 16 sheets with headers
- ✅ 16 backend API wrapper functions (stubs)
- ✅ Business rules in comments

**What's NOT Implemented Yet:**
- ❌ Excel parsing (SheetJS) - Phase 2
- ❌ Chunk upload logic - Phase 2
- ❌ Validation rules - Phase 3
- ❌ Item/location mapping - Phase 3
- ❌ Reconciliation engine - Phase 5
- ❌ T+1 movement analysis - Phase 6
- ❌ Export workbook generation - Phase 7
- ❌ Reconciliation logic
- ❌ Transfer plan generation
- ❌ Mobile count session UI logic

## What Must NOT Be Done Yet

1. **Do NOT implement Phase 2 until Phase 1B QA passes**
   - Phase 1B: Verify skeleton works before SheetJS integration
   - Acceptance: Web app loads, setupDatabase runs, all sheets created

2. **Do NOT add reconciliation logic to Phase 2**
   - Reconciliation is Phase 5, after validation (Phase 3) and mapping (Phase 3)

3. **Do NOT implement features outside scope**
   - No approval workflows
   - No role-based access control
   - No ERP integration
   - No real-time sync
   - No capacity simulation
   - No cross-warehouse custody

4. **Do NOT duplicate business logic**
   - Reference /docs as source of truth for all business rules
   - Implement once in GAS backend
   - Frontend only calls backend APIs

## Business Rules Summary

**ItemKey Construction:**
- Format: `customer_code|product_grade|pack_size|crop_year|lot`
- Used to identify exact items across locations
- Must be consistent across all sheets

**StockLocationKey Construction:**
- Format: `warehouse|location|ItemKey`
- Used to identify unique stock locations
- Enables location mismatch detection

**RE T1 Cross-Customer Blocking:**
- Use SubstitutionCandidateKey (without customer) to detect cross-customer scenarios
- Block substitution attempts for RE T1 and Pure Sugar T1
- Only block if same product appears across different customers

**Reconciliation Statuses (8 total):**
- MATCHED: Inquiry qty = Physical qty
- LOCATION_MISMATCH: Same item, different location
- TRANSFER_NEEDED: Variance can be resolved by transfer
- PENDING_TODAY_MOVEMENT: Waiting for Day T movements
- RE_T1_BLOCKED: Cross-customer substitution blocked
- MAPPING_ERROR: Item cannot be mapped
- POSSIBLE_LOSS_DRAFT: Shortage (draft, not final)
- POSSIBLE_OVER_DRAFT: Overage (draft, not final)

**Day T vs Day T+1:**
- Day T: Baseline reconciliation (draft only)
- Day T+1: Upload movements, generate targeted recount list
- No final loss/over determination before T+1 analysis

**Qty Only:**
- Use Qty field only, ignore TonQty
- All calculations in base units

**Mapping Errors:**
- Rows that cannot be mapped to ITEM_MASTER must be excluded
- Logged in VALIDATION_LOG and MAPPING_ERROR sheets
- Never included in reconciliation calculations

## Mobile Counting Requirement

**Day T Workflow:**
1. Upload ข้อมูล.xlsx (CONFIG, ITEM_MASTER, INQUIRY, INOUT, PHYSICAL_COUNT)
2. System validates sheets and columns
3. Mobile user counts physical stock by location
4. System runs baseline reconciliation
5. View exceptions and transfer plan
6. Export draft result

**Day T+1 Workflow:**
1. Upload Day T In-Out movements
2. System generates targeted recount list (only impacted items/locations)
3. Mobile user recounts only impacted items
4. System generates final action plan
5. Export final result

**Mobile UI Requirements:**
- Large touch buttons (48px minimum height)
- Location-first workflow (search location, then count items)
- Save draft to localStorage before submit
- Show sync status indicator
- Hide MATCHED rows by default (exception-first view)
- No horizontal overflow
- Responsive on mobile/tablet widths

## Next Phase: Phase 1B QA

**Phase 1B Acceptance Criteria:**
- Web app loads without errors
- setupDatabase() creates all 16 sheets
- All backend wrapper functions exist
- All include() files load correctly
- Mobile layout verified (no overflow, 48px buttons)
- No broken JS references
- Ready to proceed to Phase 2

**Phase 1B Checklist:**
- [ ] Verify Code.gs doGet() renders index.html
- [ ] Verify include() loads all 5 HTML partials
- [ ] Verify setupDatabase() creates all 16 sheets with headers
- [ ] Verify all 16 backend wrapper functions exist
- [ ] Verify mobile buttons are 48px+ height
- [ ] Verify no horizontal overflow on mobile
- [ ] Verify navigation works
- [ ] Verify localStorage works
- [ ] Test on actual mobile browser if possible

## Key Files to Review

**Backend Entry Point:**
- Code.gs - Main doGet() and API wrappers

**Database:**
- SheetService.gs - Batch Sheets operations
- Config.gs - Configuration constants

**Frontend Shell:**
- index.html - Main app with navigation
- css.html - Mobile-first responsive CSS
- js-app.html - App routing and state

**Services (Stubs - Phase 2-7):**
- UploadService.gs - Excel upload (Phase 2)
- ValidationService.gs - Data validation (Phase 3)
- MappingService.gs - Item/location mapping (Phase 3)
- ReconciliationEngine.gs - Day T reconciliation (Phase 5)
- TPlusOneEngine.gs - Day T+1 analysis (Phase 6)
- ExportService.gs - Excel export (Phase 7)

**Frontend Pages (Stubs - Phase 2-7):**
- js-upload.html - Upload page (Phase 2)
- js-count-mobile.html - Count by location (Phase 4)
- js-results.html - Results display (Phase 5)

## Source of Truth

**For all requirements:**
- `/docs/` folder contains 15 specification documents
- Use `/docs/03_business_rules.md` for business logic
- Use `/docs/05_reconciliation_logic.md` for reconciliation algorithm
- Use `/docs/06_t_plus_1_recount_logic.md` for T+1 logic

**For implementation:**
- This handoff document
- PROJECT_STATUS.md - Current state
- NEXT_STEPS.md - Phase checklists
- FILE_MANIFEST.md - File purposes

## Deployment Instructions

1. Create Google Apps Script project at script.google.com
2. Copy all .gs files to project
3. Copy all .html files as HTML files in GAS
4. Update appsscript.json manifest
5. Create Google Sheet, copy ID to Config.gs
6. Deploy as Web App (Execute as: Your account, Access: Anyone)
7. Open Web App URL on mobile browser

## Support

For questions or issues:
1. Check GAS execution logs (Executions tab)
2. Review /docs for business rules
3. Check FILE_MANIFEST.md for file purposes
4. Review NEXT_STEPS.md for phase requirements
