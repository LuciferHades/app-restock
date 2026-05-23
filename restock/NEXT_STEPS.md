# Next Steps - Phase Checklists

## Phase 1B: GAS Skeleton QA (Current)

**Goal:** Verify Phase 1 skeleton works before SheetJS integration

**Checklist:**
- [ ] Web app loads without errors
- [ ] setupDatabase() creates all 16 sheets with headers
- [ ] All backend wrapper functions exist in Code.gs
- [ ] All include() calls load correctly (css, js-app, js-upload, js-count-mobile, js-results)
- [ ] Mobile buttons are 48px+ height
- [ ] No horizontal overflow on mobile width
- [ ] Navigation between pages works
- [ ] localStorage persistence works
- [ ] Sync status indicator displays
- [ ] Loading/error/success toasts work
- [ ] Test on actual mobile browser if possible

**Acceptance Criteria:**
- Web app loads and renders
- setupDatabase runs without errors
- All 16 sheets created with correct headers
- All client wrappers have matching backend functions
- No missing include files
- No broken JS references
- Mobile layout verified

**Blockers:** None

---

## Phase 2: Upload + SheetJS Chunk Import

**Goal:** Implement Excel file upload with SheetJS parsing and chunked import

**Checklist:**
- [ ] Add SheetJS library to index.html (CDN or bundled)
- [ ] Implement handleUpload() in js-upload.html
  - [ ] Read file as ArrayBuffer
  - [ ] Parse with SheetJS (read all 5 sheets)
  - [ ] Validate sheet names (CONFIG, ITEM_MASTER, INQUIRY, INOUT, PHYSICAL_COUNT)
  - [ ] Validate required columns for each sheet
  - [ ] Split data into 500-row chunks
- [ ] Implement UploadService.gs
  - [ ] createUploadJob(metadata) - returns jobId
  - [ ] uploadParsedChunk(jobId, sheetName, rows, chunkIndex) - batch write to UPLOAD_RAW
  - [ ] finalizeUpload(jobId) - mark job as UPLOADED, move to VALIDATING
- [ ] Implement frontend progress tracking
  - [ ] Show upload progress bar
  - [ ] Display chunk count and current chunk
  - [ ] Handle upload errors with retry
- [ ] Test with sample Excel files
  - [ ] Test with 50k+ rows
  - [ ] Test with invalid file format
  - [ ] Test with missing sheets
  - [ ] Test with chunk upload resumption

**Acceptance Criteria:**
- Excel file uploads without errors
- All 5 sheets parsed correctly
- Data chunked into 500-row batches
- Chunks written to UPLOAD_RAW sheet
- Job status transitions to VALIDATING
- Progress bar shows real-time upload status
- Handles 50k+ rows efficiently

**Blockers:** None - Phase 1B must pass first

---

## Phase 3: Validation + Mapping

**Goal:** Validate uploaded data and build ItemKey/StockLocationKey mappings

**Checklist:**
- [ ] Implement ValidationService.gs
  - [ ] validateJob(jobId) - orchestrator
  - [ ] validateSheets() - check all 5 sheets exist
  - [ ] validateColumns() - check required columns per sheet
  - [ ] validateDataTypes() - check Qty is numeric, dates are valid
  - [ ] validateDuplicates() - check for duplicate INOUT rows
  - [ ] Return validation result with error codes (12 total)
- [ ] Implement MappingService.gs
  - [ ] buildItemKey(customerCode, productGrade, packSize, cropYear, lot) - returns composite key
  - [ ] buildStockLocationKey(warehouse, location, itemKey) - returns composite key
  - [ ] buildSubstitutionCandidateKey(productGrade, packSize, cropYear, lot) - without customer
  - [ ] mapItemName(rawName, itemMaster) - match raw item to master data
  - [ ] detectMappingErrors() - return unmappable items
- [ ] Implement LogService.gs
  - [ ] logValidationError(jobId, errorCode, severity, sheetName, rowNumber, message)
  - [ ] getValidationLog(jobId) - return all errors for job
- [ ] Implement js-results.html validation result display
  - [ ] Show validation errors by sheet
  - [ ] Show mapping error count
  - [ ] Show error details (row number, column, message)
  - [ ] Option to proceed or fix and re-upload
- [ ] Test validation
  - [ ] Test with missing sheets
  - [ ] Test with missing columns
  - [ ] Test with invalid data types
  - [ ] Test with duplicate movements
  - [ ] Test with unmappable items

**Acceptance Criteria:**
- Validation runs without errors
- All 12 error codes detected correctly
- ItemKey and StockLocationKey built correctly
- Mapping errors logged and displayed
- Validation log persists to VALIDATION_LOG sheet
- Job status transitions to RECONCILED after validation passes

**Blockers:** Phase 2 must complete first

---

## Phase 4: Mobile Count Session

**Goal:** Implement mobile count session UI and location-first counting workflow

**Checklist:**
- [ ] Implement CountSessionService.gs
  - [ ] createCountSession(jobId) - create new session, return sessionId
  - [ ] getCountLocations(sessionId) - return list of locations from INQUIRY
  - [ ] getCountItemsByLocation(sessionId, location) - return items at location
  - [ ] saveCountDraft(sessionId, location, rows) - save to localStorage
  - [ ] submitCountLocation(sessionId, location, rows) - write to COUNT_RESULT
- [ ] Implement js-count-mobile.html
  - [ ] Location search input with autocomplete
  - [ ] Location list (clickable)
  - [ ] Item count table (ItemKey, Expected Qty, Count input)
  - [ ] Save Draft button (localStorage)
  - [ ] Submit button (write to Sheets)
  - [ ] Count summary (total items, total qty)
- [ ] Implement localStorage persistence
  - [ ] Save draft counts to localStorage before submit
  - [ ] Load draft on page reload
  - [ ] Clear draft after successful submit
- [ ] Implement page initialization
  - [ ] Add initCountSession() callback to app.goToPage('count')
  - [ ] Load locations on page enter
  - [ ] Restore draft if exists
- [ ] Test mobile UI
  - [ ] Test on mobile browser (iOS/Android)
  - [ ] Test location search
  - [ ] Test draft save/restore
  - [ ] Test submit to Sheets
  - [ ] Test with 100+ locations
  - [ ] Test with 1000+ items per location

**Acceptance Criteria:**
- Count session creates successfully
- Locations load and are searchable
- Items display with expected quantities
- Counts can be entered and saved
- Draft persists in localStorage
- Submit writes to COUNT_RESULT sheet
- Mobile layout works on actual mobile devices

**Blockers:** Phase 3 must complete first

---

## Phase 5: Reconciliation Engine

**Goal:** Implement Day T reconciliation logic and status classification

**Checklist:**
- [ ] Implement ReconciliationEngine.gs
  - [ ] runBaselineReconciliation(jobId) - orchestrator
  - [ ] aggregateInquiryByLocation() - group INQUIRY by StockLocationKey
  - [ ] aggregatePhysicalByLocation() - group PHYSICAL_COUNT by StockLocationKey
  - [ ] calculateVariance() - compute VarianceQty = PhysicalQty - InquiryQty
  - [ ] classifyStatus() - assign status based on variance and rules
  - [ ] calculateConfidenceScore() - assign confidence 0-100
  - [ ] detectLocationMismatch() - same item, different location
  - [ ] detectTransferNeeded() - variance within same item
  - [ ] detectPendingMovement() - check INOUT for today's movements
  - [ ] detectRE_T1_CrossCustomer() - use SubstitutionCandidateKey
  - [ ] generateTransferPlan() - suggest transfers for TRANSFER_NEEDED
- [ ] Implement status classification logic
  - [ ] MATCHED: InquiryQty == PhysicalQty (confidence 100)
  - [ ] LOCATION_MISMATCH: Same item, different location (confidence 85)
  - [ ] TRANSFER_NEEDED: Variance within same item (confidence 85)
  - [ ] PENDING_TODAY_MOVEMENT: Waiting for INOUT (confidence 50)
  - [ ] RE_T1_BLOCKED: Cross-customer substitution (confidence 0)
  - [ ] MAPPING_ERROR: Item not mapped (confidence 0)
  - [ ] POSSIBLE_LOSS_DRAFT: Shortage (confidence 40)
  - [ ] POSSIBLE_OVER_DRAFT: Overage (confidence 40)
- [ ] Implement transfer plan generation
  - [ ] Detect variance pairs (LOSS + OVER of same item)
  - [ ] Suggest transfer from OVER location to LOSS location
  - [ ] Block cross-customer transfers
  - [ ] Block RE T1 cross-customer transfers
- [ ] Write results to RECON_RESULT sheet
- [ ] Test reconciliation
  - [ ] Test with exact match (MATCHED)
  - [ ] Test with location mismatch
  - [ ] Test with variance
  - [ ] Test with pending movements
  - [ ] Test with RE T1 blocking
  - [ ] Test with mapping errors
  - [ ] Test transfer plan generation
  - [ ] Test with 50k+ items

**Acceptance Criteria:**
- Reconciliation runs without errors
- All 8 statuses classified correctly
- Confidence scores assigned correctly
- Transfer plans generated correctly
- RE T1 cross-customer blocking works
- Results written to RECON_RESULT sheet
- Job status transitions to RECONCILED
- MATCHED rows hidden by default in UI

**Blockers:** Phase 4 must complete first

---

## Phase 6: T+1 Recount Logic

**Goal:** Implement Day T+1 movement analysis and targeted recount list generation

**Checklist:**
- [ ] Implement TPlusOneEngine.gs
  - [ ] uploadTPlusOneChunk(jobId, rows, chunkIndex) - batch write to INOUT
  - [ ] generateRecountList(jobId) - orchestrator
  - [ ] buildMovementImpactSet() - identify impacted locations
  - [ ] detectMovementImpact() - apply movement type rules
    - [ ] IN movement: add ToLocation to recount set
    - [ ] OUT movement: add FromLocation to recount set
    - [ ] TRANSFER movement: add both locations to recount set
    - [ ] REJ movement: add FromLocation + W8-REJ to recount set
    - [ ] ADJUST movement: add affected location to recount set
  - [ ] handleMissingLocation() - if location missing, recount all known locations
  - [ ] generateRecountList() - return list of (Location, ItemKey) pairs
- [ ] Write results to MOVEMENT_IMPACT and RECOUNT_LIST sheets
- [ ] Implement js-results.html T+1 page
  - [ ] Show movement summary (IN, OUT, TRANSFER count)
  - [ ] Show recount list (Location, ItemKey, Reason)
  - [ ] Allow user to confirm and proceed
- [ ] Test T+1 logic
  - [ ] Test with IN movements
  - [ ] Test with OUT movements
  - [ ] Test with TRANSFER movements
  - [ ] Test with REJ movements
  - [ ] Test with missing locations
  - [ ] Test with 50k+ movements
  - [ ] Test recount list generation

**Acceptance Criteria:**
- T+1 movements upload successfully
- Movement impact set calculated correctly
- Recount list generated correctly
- Results written to MOVEMENT_IMPACT and RECOUNT_LIST sheets
- Recount list shows only impacted items/locations
- Job status transitions to WAITING_T_PLUS_1
- Handles 50k+ movements efficiently

**Blockers:** Phase 5 must complete first

---

## Phase 7: Export Service

**Goal:** Generate multi-sheet Excel workbooks with reconciliation results

**Checklist:**
- [ ] Implement ExportService.gs
  - [ ] exportResultWorkbook(jobId) - orchestrator
  - [ ] generateSummarySheet() - job metadata, error count, status
  - [ ] generateReconciliationSheet() - RECON_RESULT data
  - [ ] generateTransferPlanSheet() - TRANSFER_PLAN data
  - [ ] generateMovementImpactSheet() - MOVEMENT_IMPACT data
  - [ ] generateRecountListSheet() - RECOUNT_LIST data
  - [ ] generateFinalActionSheet() - FINAL_ACTION data
  - [ ] generateMappingErrorSheet() - VALIDATION_LOG mapping errors
  - [ ] generateValidationLogSheet() - all validation errors
  - [ ] applyFormatting() - headers, auto-filter, conditional formatting
  - [ ] createWorkbook() - combine sheets into single file
  - [ ] uploadToGoogleDrive() - save to Drive, return download URL
- [ ] Implement sheet formatting
  - [ ] Bold headers
  - [ ] Auto-filter on all sheets
  - [ ] Conditional formatting for status (green/yellow/red)
  - [ ] Column width auto-fit
  - [ ] Freeze header rows
- [ ] Implement export UI
  - [ ] Show export progress
  - [ ] Display download link
  - [ ] Show file size and sheet count
  - [ ] Option to email or download
- [ ] Test export
  - [ ] Test with small dataset (100 items)
  - [ ] Test with large dataset (50k+ items)
  - [ ] Verify all sheets created
  - [ ] Verify formatting applied
  - [ ] Verify download works
  - [ ] Open in Excel and verify data

**Acceptance Criteria:**
- Export workbook creates without errors
- All 8 sheets generated correctly
- Formatting applied (headers, filters, colors)
- File uploads to Google Drive
- Download link works
- Excel file opens correctly
- All data displays correctly
- Handles 50k+ items efficiently

**Blockers:** Phase 6 must complete first

---

## Deployment Checklist (After All Phases)

- [ ] All services implemented (no stubs)
- [ ] All tests passing
- [ ] Code reviewed
- [ ] Performance tested with 50k+ rows
- [ ] Mobile tested on actual devices
- [ ] GAS project deployed
- [ ] Web app URL generated
- [ ] Database sheets created
- [ ] Documentation updated
- [ ] User training completed
