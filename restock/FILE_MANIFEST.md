# File Manifest - Restock Reconciliation Tool

## Directory Structure

```
restock_reconciliation_gas/
├── Backend Services (.gs files)
│   ├── Code.gs
│   ├── Config.gs
│   ├── Utils.gs
│   ├── SheetService.gs
│   ├── UploadService.gs
│   ├── ValidationService.gs
│   ├── MappingService.gs
│   ├── ReconciliationEngine.gs
│   ├── TPlusOneEngine.gs
│   ├── CountSessionService.gs
│   ├── ExportService.gs
│   ├── LogService.gs
│   └── appsscript.json
├── Frontend Files (.html files)
│   ├── index.html
│   ├── css.html
│   ├── js-app.html
│   ├── js-upload.html
│   ├── js-count-mobile.html
│   └── js-results.html
├── Documentation
│   ├── README.md
│   ├── README_GAS.md
│   ├── HANDOFF_ANTIGRAVITY.md
│   ├── PROJECT_STATUS.md
│   ├── NEXT_STEPS.md
│   ├── FILE_MANIFEST.md
│   └── ANTIGRAVITY_PROMPT.md
└── /docs/ (Specifications)
    ├── 00_master_context.md
    ├── 01_product_scope.md
    ├── 02_structure.md
    ├── 03_business_rules.md
    ├── 04_data_contract.md
    ├── 05_reconciliation_logic.md
    ├── 06_t_plus_1_recount_logic.md
    ├── 07_ui_ux_design.md
    ├── 08_technical_architecture.md
    ├── 09_api_and_state.md
    ├── 10_export_output_spec.md
    ├── 11_test_cases.md
    ├── 12_implementation_plan.md
    ├── 13_agent_rules.md
    └── 14_assumptions_and_open_questions.md
```

## Backend Services (.gs files)

### Active Files (Phase 1)

| File | Size | Purpose | Status | Phase |
|------|------|---------|--------|-------|
| Code.gs | ~2KB | Main entry point, doGet(), API wrappers | ✅ Active | 1 |
| Config.gs | ~1KB | Configuration constants, sheet names, statuses | ✅ Active | 1 |
| Utils.gs | ~1KB | Helper functions (include, IDs, logging) | ✅ Active | 1 |
| SheetService.gs | ~3KB | Batch Sheets operations, database setup | ✅ Active | 1 |
| appsscript.json | ~1KB | GAS project manifest | ✅ Active | 1 |

### Stub Files (Phases 2-7)

| File | Size | Purpose | Status | Phase |
|------|------|---------|--------|-------|
| UploadService.gs | ~1KB | Excel upload, chunk handling | 🔲 Stub | 2 |
| ValidationService.gs | ~1KB | Data validation, error logging | 🔲 Stub | 3 |
| MappingService.gs | ~1KB | ItemKey, StockLocationKey building | 🔲 Stub | 3 |
| ReconciliationEngine.gs | ~2KB | Day T reconciliation, status classification | 🔲 Stub | 5 |
| TPlusOneEngine.gs | ~1KB | Day T+1 movement analysis | 🔲 Stub | 6 |
| CountSessionService.gs | ~1KB | Mobile count sessions | 🔲 Stub | 4 |
| ExportService.gs | ~1KB | Excel workbook export | 🔲 Stub | 7 |
| LogService.gs | ~1KB | Validation error logging | 🔲 Stub | 3 |

## Frontend Files (.html files)

### Active Files (Phase 1)

| File | Size | Purpose | Status | Phase |
|------|------|---------|--------|-------|
| index.html | ~3KB | Main app shell, navigation, page containers | ✅ Active | 1 |
| css.html | ~4KB | Mobile-first responsive CSS | ✅ Active | 1 |
| js-app.html | ~2KB | App routing, state management | ✅ Active | 1 |

### Stub Files (Phases 2-7)

| File | Size | Purpose | Status | Phase |
|------|------|---------|--------|-------|
| js-upload.html | ~1KB | Upload page UI | 🔲 Stub | 2 |
| js-count-mobile.html | ~2KB | Mobile count UI | 🔲 Stub | 4 |
| js-results.html | ~2KB | Results display UI | 🔲 Stub | 5 |

## Documentation Files

| File | Purpose | Audience |
|------|---------|----------|
| README.md | Project overview | All |
| README_GAS.md | Deployment guide | Developers |
| HANDOFF_ANTIGRAVITY.md | Handoff to Antigravity | Antigravity team |
| PROJECT_STATUS.md | Current state summary | All |
| NEXT_STEPS.md | Phase checklists | Developers |
| FILE_MANIFEST.md | This file - file purposes | Developers |
| ANTIGRAVITY_PROMPT.md | Ready-to-paste prompt | Antigravity team |

## Specification Files (/docs/)

All 15 specification documents from the initial requirements analysis. Use as source of truth for business rules and requirements.

| Document | Purpose |
|----------|---------|
| 00_master_context.md | Project context and overview |
| 01_product_scope.md | Feature scope and boundaries |
| 02_structure.md | Data structure and schemas |
| 03_business_rules.md | Business logic and rules |
| 04_data_contract.md | Data format specifications |
| 05_reconciliation_logic.md | Day T reconciliation algorithm |
| 06_t_plus_1_recount_logic.md | Day T+1 logic |
| 07_ui_ux_design.md | UI/UX design guide |
| 08_technical_architecture.md | Technical architecture |
| 09_api_and_state.md | API and state management |
| 10_export_output_spec.md | Export format specifications |
| 11_test_cases.md | Test cases and scenarios |
| 12_implementation_plan.md | Implementation phases |
| 13_agent_rules.md | Agent behavior rules |
| 14_assumptions_and_open_questions.md | Assumptions and unknowns |

## File Dependencies

### Backend Dependencies

```
Code.gs
├── Config.gs (configuration constants)
├── Utils.gs (include, logging, helpers)
├── SheetService.gs (Sheets operations)
├── UploadService.gs (Phase 2)
├── ValidationService.gs (Phase 3)
├── MappingService.gs (Phase 3)
├── ReconciliationEngine.gs (Phase 5)
├── TPlusOneEngine.gs (Phase 6)
├── CountSessionService.gs (Phase 4)
├── ExportService.gs (Phase 7)
└── LogService.gs (Phase 3)
```

### Frontend Dependencies

```
index.html
├── css.html (styling)
├── js-app.html (routing, state)
├── js-upload.html (Phase 2)
├── js-count-mobile.html (Phase 4)
└── js-results.html (Phase 5)

js-app.html
├── google.script.run wrappers
│   ├── getAppConfig()
│   ├── setupDatabase()
│   ├── createUploadJob()
│   ├── uploadParsedChunk()
│   ├── finalizeUpload()
│   ├── validateJob()
│   ├── createCountSession()
│   ├── getCountLocations()
│   ├── getCountItemsByLocation()
│   ├── saveCountDraft()
│   ├── submitCountLocation()
│   ├── runBaselineReconciliation()
│   ├── uploadTPlusOneChunk()
│   ├── generateRecountList()
│   ├── getResults()
│   └── exportResultWorkbook()
```

## File Status Legend

| Symbol | Meaning |
|--------|---------|
| ✅ Active | Implemented and working |
| 🔲 Stub | Placeholder, needs implementation |
| ⚠️ Partial | Partially implemented |
| ❌ Deprecated | No longer used |

## Phase Implementation Map

| Phase | Backend Files | Frontend Files | Status |
|-------|---------------|----------------|--------|
| 1 | Code, Config, Utils, SheetService | index, css, js-app | ✅ Complete |
| 2 | UploadService | js-upload | 🔲 Stub |
| 3 | ValidationService, MappingService, LogService | (no UI) | 🔲 Stub |
| 4 | CountSessionService | js-count-mobile | 🔲 Stub |
| 5 | ReconciliationEngine | js-results | 🔲 Stub |
| 6 | TPlusOneEngine | (no UI) | 🔲 Stub |
| 7 | ExportService | (no UI) | 🔲 Stub |

## How to Use This Manifest

1. **To find a file:** Use the directory structure or search by filename
2. **To understand a file's purpose:** Check the Purpose column
3. **To know if a file is ready:** Check the Status column
4. **To understand dependencies:** See the File Dependencies section
5. **To know which phase a file belongs to:** Check the Phase column
6. **To find business rules:** See /docs/03_business_rules.md
7. **To find implementation details:** See NEXT_STEPS.md

## Deprecated/Archived Files

The following projects were deprecated and archived:
- `/archive/restock_reconciliation/` - Original Node.js backend (replaced by GAS)
- `/archive/restock_reconciliation_backend/` - FastAPI backend (replaced by GAS)

These are kept for reference only. Do not use them.

## File Sizes

Total size of active files: ~20KB (very small, suitable for GAS)
Total size with stubs: ~25KB
Total size with docs: ~500KB

## Notes

- All .gs files are Google Apps Script compatible
- All .html files use vanilla JS (no frameworks)
- CSS is mobile-first responsive (no Tailwind CDN needed)
- No external dependencies except SheetJS (Phase 2)
- All files follow GAS naming conventions
- No node_modules or package managers needed
