# 19_ai_agent_task.md

# AI Agent Task Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** AI Coding Agent Instruction / Implementation Task Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / Codex Use  
**Primary Use:** ใช้เป็นคำสั่งหลักให้ AI Coding Agent สร้าง Web App ตามเอกสารระบบ โดยควบคุมลำดับงาน architecture, coding rules, validation, audit, performance, UX state และ acceptance checklist เพื่อให้ระบบ clean, professional, maintainable และไม่หลุด scope

---

## 1. Purpose of This Document

เอกสารนี้กำหนดงานและกฎสำหรับ AI Coding Agent ที่จะนำชุดเอกสารทั้งหมดไปสร้างระบบจริง

AI Agent ต้องเข้าใจว่าโปรเจกต์นี้ไม่ใช่ CRUD app ธรรมดา แต่เป็นระบบที่มี data pipeline และ operation control หลายชั้น:

```text
Import Staging
→ Column Mapping
→ Alias Review
→ Anchor Snapshot
→ Movement Ledger
→ Stock Replay
→ Reconciliation
→ Issue / Task
→ Control Center
→ Export / Notification / Retention
```

ดังนั้น AI Agent ต้องสร้างระบบแบบ:

- clean architecture
- service-based
- data-driven
- permission-safe
- audit-ready
- responsive
- performance-aware
- explainable reconciliation
- MVP scope controlled

หลักสำคัญ:

```text
AI Agent ต้องสร้างระบบตาม docs เท่านั้น
ถ้าข้อมูลไม่พอ ให้สร้าง TODO / Question / Assumption ไม่ใช่เดา logic เอง
```

---

## 2. Mandatory Reading Order

ก่อนเขียนโค้ด AI Agent ต้องอ่านเอกสารตามลำดับนี้:

```text
01_product_brief.md
02_structure.md
17_out_of_scope.md
18_development_plan.md
10_data_model.md
11_api_backend_spec.md
12_business_rules.md
13_validation_rules.md
14_audit_log_security.md
07_design.md
08_design_system.md
09_responsive_spec.md
15_dashboard_kpi_report.md
16_error_empty_loading_state.md
20_acceptance_criteria.md
```

ถ้ามีเวลาอ่านครบ ให้ดูทุกไฟล์ใน `/docs`

---

## 3. Document Authority Order

ถ้าเอกสารขัดแย้งกัน ให้ยึดลำดับความสำคัญนี้:

```text
1. 01_product_brief.md
2. 02_structure.md
3. 12_business_rules.md
4. 10_data_model.md
5. 11_api_backend_spec.md
6. 13_validation_rules.md
7. 14_audit_log_security.md
8. 05_workflow.md
9. 07_design.md
10. 18_development_plan.md
11. 17_out_of_scope.md
12. Other supporting docs
```

ถ้ายังขัดแย้ง ให้หยุดและสร้าง `NEEDS_DECISION` note แทนการเดา

---

## 4. AI Agent Role

AI Agent ต้องทำหน้าที่เป็น:

```text
Senior Full-Stack Developer
System Architect
Data Engineer
UX Engineer
Security-Aware Developer
QA-Oriented Builder
Warehouse Reconciliation Implementer
```

ไม่ใช่แค่เขียน UI ให้เหมือน mockup

ต้องคิดครบ:

```text
Data → Validation → Permission → Process → Output → Audit → UX State → Performance
```

---

# 5. Non-Negotiable Rules

## 5.1 Coding Rules

1. ห้ามเริ่มเขียน feature โดยไม่ดู docs
2. ห้ามเพิ่ม feature นอก scope
3. ห้าม hardcode column mapping
4. ห้าม hardcode alias เช่น MKMK/UPUP/UFUF ใน business logic
5. ห้าม hardcode permission เฉพาะ frontend
6. ห้ามให้ dashboard query raw full history
7. ห้ามสร้าง issue ที่ไม่มี explanation
8. ห้ามลบข้อมูลที่ผูกกับ open issue
9. ห้ามแก้ locked day โดยตรง
10. ห้าม expose password/token/secret ไป frontend
11. ห้าม direct ERP posting ใน MVP
12. ห้าม auto-map low confidence alias
13. ห้าม auto-close issue ที่ confidence ต่ำ
14. ห้ามทำ simulation/relocation/advanced FIFO ใน MVP
15. ห้ามใช้ mock data ใน production logic

---

## 5.2 Required For Every Write Action

ทุก write action ต้องมี:

```text
1. Auth/session check
2. User status check
3. Permission check
4. Warehouse scope check if related
5. Backend validation
6. Business rule check
7. Audit log if critical
8. Error handling
9. Success/failure response
```

Examples:

- approve user
- reject user
- confirm import
- map alias
- create field movement
- submit count
- close issue
- export report
- run retention

---

## 5.3 Required For Every Page

ทุก page ต้องมี:

```text
Loading state
Empty state
Error state
Permission denied state
Success feedback for write action
Responsive behavior
```

---

## 5.4 Required For Every Long Job

งานหนักต้องมี job status:

```text
QUEUED
RUNNING
SUCCESS
WARNING
FAILED
CANCELLED
```

และต้องเก็บ:

- started_at
- finished_at
- rows_processed
- warning_count
- error_count
- message

Applies to:

- normalize batch
- stock replay
- reconciliation
- export
- retention
- health check

---

# 6. Target Architecture

## 6.1 Recommended Folder Structure

AI Agent ควรสร้างโครงสร้างประมาณนี้:

```text
src/
  app/
    routes/
    layouts/
    providers/
  components/
    ui/
    data-display/
    forms/
    feedback/
    navigation/
  features/
    auth/
    users/
    permissions/
    master-data/
    import-center/
    alias/
    checker-session/
    field-operations/
    stock-replay/
    reconciliation/
    issues/
    dashboard/
    export/
    notification/
    retention/
    audit/
    health/
  services/
    AuthService
    PermissionService
    AuditLogService
    ImportService
    AliasService
    MasterDataService
    SnapshotService
    MovementLedgerService
    StockReplayService
    ReconciliationService
    IssueService
    ExportService
    NotificationService
    RetentionService
    HealthCheckService
  data/
    repositories/
    db/
    migrations/
    seeds/
  validation/
    schemas/
    validators/
  jobs/
    normalizeBatch
    stockReplay
    reconciliation
    exportJob
    retentionJob
    healthCheck
  utils/
    dates
    qty
    uom
    textNormalize
    ids
    errors
    pagination
  types/
    domain
    api
    permissions
  config/
    permissions
    status
    resultCodes
    notificationEvents
    importFields
  tests/
    unit/
    integration/
    e2e/
```

ถ้า framework ที่ใช้มี convention ต่างกัน ให้ปรับได้ แต่ต้องรักษา separation:

```text
UI layer ≠ Service layer ≠ Data layer ≠ Validation ≠ Jobs
```

---

## 6.2 Required Layers

### UI Layer

- Pages
- Components
- Forms
- Tables
- State display
- Permission-aware navigation

### Service Layer

- Business logic
- Permission assertion
- Validation orchestration
- Audit writing
- Job triggering

### Data Layer

- Repository/query functions
- Database schema/migrations
- Storage/file access

### Job Layer

- Batch processing
- Replay
- Reconciliation
- Retention
- Export

### Config Layer

- permissions
- statuses
- result codes
- import canonical fields
- notification events
- retention policy

---

# 7. Build Order

AI Agent ต้องทำตามลำดับนี้ ยกเว้นมี instruction ใหม่จาก project owner

```text
1. Project structure
2. Environment/config
3. Design system base components
4. App shell + route guards
5. Auth/session
6. User approval
7. Role/permission/warehouse scope
8. Audit log
9. Master data base
10. Import staging
11. Column mapping
12. Alias review
13. Inquiry anchor snapshot
14. System In-Out records
15. Movement ledger
16. Checker GPS session
17. Field movement basic
18. Pending inbound basic
19. Physical count basic
20. Stock replay
21. Data coverage check
22. Reconciliation engine
23. Issue center/detail
24. Control center/dashboard summary
25. Export center
26. Notification center
27. Retention 45 days
28. Health check
29. QA/test/acceptance checklist
```

ห้ามข้ามไปทำ:

```text
simulation
relocation optimizer
advanced FIFO
visual map
OCR
native app
direct ERP integration
```

---

# 8. Phase-Based Task Detail

# 8.1 Phase 1 — Foundation Core Tasks

## TASK-001 — Initialize Project Structure

### Goal

สร้าง project ที่แยก layer ชัดเจน พร้อมพัฒนาต่อ

### Requirements

- create folder structure
- setup lint/format if environment supports
- create base config files
- create environment variable example
- create README basic

### Acceptance

- project run ได้
- folder separation ชัด
- no feature logic in root/components มั่ว

---

## TASK-002 — Build Design System Base

### Components

- Button
- Input
- Select
- Textarea
- Checkbox/Toggle
- Card
- Badge
- StatusBadge
- SeverityBadge
- ConfidenceBadge
- DataTable
- FilterBar
- EmptyState
- ErrorState
- LoadingSkeleton
- ConfirmModal
- DetailDrawer
- ProgressPanel
- Toast/Banner

### Acceptance

- components reusable
- supports loading/disabled/error state
- mobile touch target ok
- colors/status consistent with docs

---

## TASK-003 — App Shell + Navigation

### Requirements

Desktop:

- sidebar
- top bar
- user menu
- warehouse/date context

Mobile:

- checker bottom nav
- compact header

### Acceptance

- menu respects permission
- route guard exists
- direct URL without permission blocked by backend/API layer and UI state

---

## TASK-004 — Auth / Session

### Requirements

- register user
- login
- logout
- verify session
- session expiry
- password hash/salt
- user status check

### Acceptance

- pending user cannot login
- locked/suspended user cannot login
- session can be revoked
- no plain password stored

---

## TASK-005 — User Approval Center

### Requirements

- list pending requests
- approve with role/permission/scope
- reject and delete request
- audit required

### Acceptance

- approve creates active user
- reject removes queue item but audit remains
- permission required backend

---

## TASK-006 — Permission + Warehouse Scope

### Requirements

- roles
- permissions
- role_permissions
- user_roles
- permission overrides
- warehouse scope
- effective permission resolver

### Acceptance

- backend blocks missing permission
- warehouse scope filters data
- DENY override wins if conflict

---

## TASK-007 — Audit Log

### Requirements

- writeAudit function
- list audit logs
- entity audit trail
- audit severity

### Acceptance

- critical writes write audit
- audit includes actor/action/entity/old/new/reason/timestamp

---

# 8.2 Phase 2 — Import & Alias Tasks

## TASK-101 — Import Batch Model/API

### Requirements

- create batch
- file type
- checksum
- business date range
- warehouse
- status lifecycle

### Acceptance

- duplicate checksum warning
- status starts UPLOADED
- audit IMPORT_BATCH_CREATED

---

## TASK-102 — File Preview

### Requirements

- parse headers
- show sample rows
- row count
- detect empty/header missing

### Acceptance

- user sees preview before confirm
- invalid file blocked

---

## TASK-103 — Chunk Upload Raw Rows

### Requirements

- upload rows in chunk
- save raw_json
- row_no
- parse_status

### Acceptance

- large files do not rely on one huge request
- progress visible

---

## TASK-104 — Column Mapping

### Requirements

- map source column to canonical field
- required mapping validation
- saved template
- header changed warning

### Acceptance

- normalize blocked if required mapping missing
- no hardcoded column indexes

---

## TASK-105 — Normalize Batch v0

### Requirements

- parse rows using mapping
- validate date/qty/uom
- detect aliases
- create error logs
- create normalized staging records where possible

### Acceptance

- invalid rows reported
- unknown alias queued
- batch status updated

---

## TASK-106 — Alias Review Queue

### Requirements

- create queue item for unknown alias
- list/filter queue
- suggest match with confidence
- map to existing
- create master from alias
- reject

### Acceptance

- alias type required
- type mismatch blocked
- low confidence cannot auto-map
- audit alias actions

---

# 8.3 Phase 3 — Data Engine Tasks

## TASK-201 — Inquiry Anchor Snapshot

### Requirements

- confirm Inquiry batch
- create inquiry_snapshots
- create inquiry_snapshot_lines
- handle active snapshot conflict

### Acceptance

- Inquiry does not create movement ledger
- duplicate/same date policy enforced
- snapshot traceable to batch/raw rows

---

## TASK-202 — System In-Out Records

### Requirements

- confirm System In-Out batch
- create system_inout_records
- normalize movement_type
- validate qty/location/date

### Acceptance

- records trace to batch/raw row
- movement type canonical

---

## TASK-203 — Movement Ledger

### Requirements

- create ledger from System In-Out
- create ledger from Field Movement
- create ledger from REJ/Pending Inbound Confirm later
- source_type/source_ref_id required

### Acceptance

- every ledger row traceable
- stock_effect_sign valid
- no silent edit after replay

---

## TASK-204 — Checker GPS Daily Session

### Requirements

- check-in warehouse
- store gps lat/lng/accuracy
- one active warehouse per day
- suspend old session when new WH login

### Acceptance

- count blocked without active session
- session warehouse must match count warehouse

---

## TASK-205 — Field Movement Basic

### Requirements

- IN/OUT/TRANSFER basic form/API
- qty positive
- location validation
- create movement_ledger

### Acceptance

- locked day blocks direct edit
- checker requires session

---

## TASK-206 — Pending Inbound Basic

### Requirements

- smart text input
- label text + location + qty + photo optional
- parse preview
- save pending if incomplete/unknown

### Acceptance

- pending inbound minimal data required
- unknown alias queued

---

## TASK-207 — Physical Count Basic

### Requirements

- submit count
- task/location scope
- checker session required
- photo optional

### Acceptance

- count qty >= 0
- count outside session warehouse blocked

---

## TASK-208 — Data Coverage Check

### Requirements

Check:

- anchor exists
- in-out complete
- alias pending
- pending inbound
- REJ pending
- moving location count missing

### Acceptance

- coverage status OK/WARNING/CRITICAL
- details explain missing data

---

## TASK-209 — Stock Replay Job

### Requirements

- find anchor snapshot
- load ledger range
- calculate daily stock position
- save daily_stock_positions
- confidence calculation

### Acceptance

- replay has job status/log
- missing anchor handled
- daily positions generated

---

# 8.4 Phase 4 — Reconciliation & Issue Tasks

## TASK-301 — Matching Key Config

### Requirements

- configurable matching rules
- Field vs System
- Expected vs Count
- REJ vs System
- Pending Inbound vs In-Out

### Acceptance

- no hardcoded single key
- config can evolve after profiling

---

## TASK-302 — Reconciliation Job

### Requirements

- load required truth layers
- match candidates
- compare qty/location/item/lot
- generate recon_results

### Acceptance

- job logs status/counts
- low coverage downgrades confidence

---

## TASK-303 — Result Code Engine

### Required Result Codes

```text
MATCHED
SYSTEM_MISSING
FIELD_MISSING
QTY_DIFF
LOCATION_DIFF
ITEM_DIFF
LOT_DIFF
UNKNOWN_ALIAS
PENDING_INBOUND
REJ_NOT_POSTED
NEED_RECOUNT
NEED_INVESTIGATION
```

### Acceptance

- every result has result_code
- non-matched creates explanation

---

## TASK-304 — Explanation Engine

### Requirements

For every non-matched:

- explanation
- likely_cause
- suggested_action
- owner_role

### Acceptance

Issue detail understandable without Excel

---

## TASK-305 — Issue Engine

### Requirements

- create issue from critical/high results
- owner role mapping
- status lifecycle
- assign/close/reopen
- comments

### Acceptance

- issue links result/source
- close requires reason
- open issue blocks retention delete

---

## TASK-306 — Issue Center UI

### Requirements

- summary cards
- filters
- table/card list responsive
- severity/status/confidence badges
- drilldown detail

### Acceptance

- can filter by date/warehouse/result_code/severity/status/owner
- mobile uses cards, not giant table

---

## TASK-307 — Issue Detail UI

### Requirements

Show:

- result header
- stock identity
- comparison panel
- field movement
- system in-out
- physical count
- expected stock
- explanation
- suggested action
- audit trail

### Acceptance

- user can see what is wrong and what to do next

---

## TASK-308 — Count Required Locations

### Requirements

- derive from issues/coverage
- list location/reason/priority
- create checker tasks

### Acceptance

- supervisor can assign recount tasks

---

## TASK-309 — REJ Notice Basic

### Requirements

- create REJ notice
- reason required
- from/rej location
- status WAIT_SYSTEM_POST
- reconcile with system in-out

### Acceptance

- REJ_NOT_POSTED issue created when not matched

---

# 8.5 Phase 5 — Control Layer Tasks

## TASK-401 — Summary Tables / Aggregation

### Requirements

- daily_recon_summary
- daily_issue_summary
- import_summary
- health summary

### Acceptance

- dashboard does not query raw full history

---

## TASK-402 — Control Center

### Required Cards

- Close Readiness
- Critical Issues
- Open Issues
- Unknown Alias
- Pending Inbound
- REJ Not Posted
- Data Coverage
- Last Import
- Backfill Status
- Export Pending
- Retention Status
- DB Usage

### Acceptance

- every card has context and drilldown
- critical/pending shown first

---

## TASK-403 — Close Readiness Gate

### Checks

- open critical issues
- pending alias
- pending inbound
- missing anchor
- missing in-out
- REJ_NOT_POSTED
- failed reconciliation

### Acceptance

- returns READY/READY_WITH_WARNING/NOT_READY
- blockers list visible

---

## TASK-404 — Daily Close / Lock Basic

### Requirements

- OPEN
- PRELIM_CLOSED
- FINAL_CLOSED
- LOCKED
- reason for lock/unlock
- locked day prevents direct edit

### Acceptance

- final close blocks critical issue
- unlock requires permission/reason/audit

---

# 8.6 Phase 6 — Export / Notification / Retention Tasks

## TASK-501 — Export Center

### Export Types

- Issue List
- Daily Reconciliation
- REJ Report
- Pending Inbound
- Alias Review
- ERP Correction Template
- Audit Report
- Retention Report

### Acceptance

- preview before export
- log every export
- ERP template requires specific permission/audit

---

## TASK-502 — Notification Center

### Requirements

- channels
- rules
- templates
- logs
- test send
- cooldown
- send once per issue

### Channels

```text
IN_APP
TELEGRAM
LINE_MESSAGING
EMAIL
```

### Acceptance

- no raw token exposed
- notification failures logged

---

## TASK-503 — Retention 45 Days

### Requirements

- cutoff = today - 45 days
- dry run
- execute
- archive/summary before delete
- skip open issue
- log per table
- notify admin

### Acceptance

- open issue data not deleted
- retention log shows deleted/skipped

---

## TASK-504 — Health Check

### Requirements

- DB usage
- row counts
- last import
- last replay
- last reconciliation
- last retention
- failed jobs
- pending alias
- pending inbound
- open critical issues

### Acceptance

- health status OK/WARNING/CRITICAL

---

# 9. API Implementation Rules

## 9.1 Response Format

Every API must return:

```json
{
  "success": true,
  "data": {},
  "message": "",
  "code": "OK",
  "meta": {}
}
```

Error:

```json
{
  "success": false,
  "code": "VALIDATION_ERROR",
  "message": "ข้อมูลไม่ถูกต้อง",
  "details": {}
}
```

---

## 9.2 Error Codes

Use machine-readable codes from docs:

- PERMISSION_DENIED
- WAREHOUSE_SCOPE_DENIED
- CHECKER_SESSION_REQUIRED
- MISSING_REQUIRED_MAPPING
- UNKNOWN_ALIAS
- MISSING_ANCHOR
- COVERAGE_NOT_READY
- ISSUE_CLOSE_REASON_REQUIRED
- RETENTION_BLOCKED_OPEN_ISSUE

Add new code only if needed and document it

---

## 9.3 Backend Guard Pattern

Every write API should follow:

```text
loadRequestContext
assertAuthenticated
assertUserActive
assertPermission
assertWarehouseScope if applicable
validateInput
validateBusinessRule
performTransaction
writeAudit
returnSuccess
```

---

# 10. Data Implementation Rules

## 10.1 Source Traceability

Every important record must include source fields where relevant:

```text
source_type
source_ref_id
batch_id
raw_row_id
created_by
created_at
```

## 10.2 Raw vs Canonical

Do not overwrite raw values

Keep:

```text
raw_customer_text
raw_item_text
raw_location_text
```

alongside canonical IDs

## 10.3 Status Instead of Boolean

Use lifecycle status fields, not only booleans

Example:

```text
Import status: UPLOADED / MAPPED / NORMALIZED / CONFIRMED / FAILED
Issue status: OPEN / ASSIGNED / IN_REVIEW / CLOSED
```

## 10.4 Summary Tables

Dashboard and report summary must read from summary/result tables where possible

---

# 11. UI Implementation Rules

## 11.1 Responsive Rules

- Checker mobile first
- Admin desktop first
- Supervisor tablet/desktop
- Tables on desktop
- Cards on mobile
- Bottom navigation for checker
- Sticky action bar on mobile forms

## 11.2 Required UI States

Every page:

- loading
- empty
- error
- permission denied

Every form:

- validation error
- submit loading
- success
- failure
- disabled reason

Every critical action:

- confirmation modal

## 11.3 Issue Detail Requirement

Issue Detail must never show only code

It must show:

```text
What was compared
What value differs
Why system thinks it is wrong
Confidence
Suggested action
Owner
Source evidence
```

---

# 12. Testing Requirements

## 12.1 Unit Tests

Must test:

- permission resolver
- alias normalization
- qty/date parsing
- result code engine
- confidence logic
- close readiness logic
- retention skip logic

## 12.2 Integration Tests

Must test:

- register → approve → login
- import → mapping → normalize → alias queue
- inquiry confirm → snapshot
- inout confirm → ledger
- replay → daily stock position
- reconciliation → issue
- issue close with reason
- export log
- retention skip open issue

## 12.3 UX / E2E Test Cases

Must test:

- Checker check-in → count submit
- Admin import/backfill
- Admin alias review
- Supervisor issue review
- ERP Admin export template
- Admin retention dry run

---

# 13. Sample Test Case Set

AI Agent must create test cases for:

```text
1. User register pending approval
2. Pending user cannot login
3. Admin approve with scope
4. Reject user deletes request but audit exists
5. Import file missing required mapping blocks normalize
6. Unknown customer alias creates alias queue
7. Alias type mismatch blocks mapping
8. Inquiry confirm creates snapshot not ledger
9. In-Out confirm creates ledger
10. Missing anchor blocks replay or creates warning
11. Backfill 5 days creates daily stock position
12. QTY_DIFF creates issue with explanation
13. SYSTEM_MISSING owner = ERP_ADMIN
14. REJ_NOT_POSTED creates issue and notification event
15. Checker without session cannot submit count
16. Locked day blocks movement edit
17. Export creates log
18. Retention skips open issue source
19. Dashboard loads from summary source
20. Permission denied on direct API call
```

---

# 14. Performance Requirements

## 14.1 Must Avoid

- loading all raw rows into dashboard
- nested O(n²) matching for large data
- no pagination in large tables
- giant synchronous import request
- photo export/load unoptimized

## 14.2 Must Implement

- pagination
- filtering by date/warehouse/status
- indexes on date/warehouse/status
- chunk upload
- job status for heavy processing
- summary tables
- retention cleanup

---

# 15. Security Requirements

AI Agent must ensure:

- password hash + salt
- no raw secret in frontend
- notification token uses token_ref
- backend permission check
- warehouse scope check
- audit for critical action
- export log
- locked day protection
- retention skip open issue

---

# 16. When Data Is Missing

ถ้าข้อมูลจริงยังไม่พอ ให้ AI Agent ต้องสร้าง note แบบนี้:

```text
NEEDS_DATA_PROFILING:
- Need sample Inquiry column names
- Need System In-Out movement type codes
- Need location/DC mapping examples
- Need UOM patterns
```

หรือ:

```text
ASSUMPTION:
- Assuming qty from Field Movement is positive and movement_type determines sign
RISK:
- If ERP In-Out uses negative qty, mapping config must handle sign conversion
```

ห้ามเดาแบบฝัง logic ถาวร

---

# 17. Do Not Implement Yet

AI Agent ห้าม implement ใน MVP:

```text
Relocation Optimizer
Advanced FIFO
Visual Warehouse Map
OCR label reader
Native mobile app
Full offline sync
Direct ERP posting
Finance costing
Contract/billing
Advanced ML forecast
Multi-tenant SaaS system
```

ถ้าต้องเตรียมไว้ ให้ทำแค่ placeholder field หรือ backlog note

---

# 18. Deliverable Format

หลังจบแต่ละ sprint/phase AI Agent ต้องส่ง summary:

```text
Completed:
- ...

Files Created/Modified:
- ...

Validation Added:
- ...

Audit Events Added:
- ...

Tests Added:
- ...

Known Assumptions:
- ...

Risks / TODO:
- ...

Next Step:
- ...
```

---

# 19. Final Checklist Before Claiming Done

ห้ามบอกว่าเสร็จ ถ้ายังไม่ผ่าน checklist นี้:

- [ ] Auth works
- [ ] User approval works
- [ ] Permission backend works
- [ ] Warehouse scope works
- [ ] Audit log works
- [ ] Import staging works
- [ ] Column mapping works
- [ ] Alias review works
- [ ] Inquiry snapshot works
- [ ] In-Out ledger works
- [ ] Stock replay works
- [ ] Reconciliation creates result/explanation
- [ ] Issue center/detail works
- [ ] Checker session blocks count if missing
- [ ] Control center loads from summary/result
- [ ] Export creates log
- [ ] Notification rule/log works
- [ ] Retention dry run/execute works and skips open issue
- [ ] Loading/error/empty states exist
- [ ] Mobile checker flow works
- [ ] Critical tests pass

---

# 20. Final AI Agent Task Statement

```text
AI Agent ต้องสร้าง Warehouse Reconciliation ERP Platform ตามเอกสารเท่านั้น โดยเริ่มจาก foundation, permission, audit, import staging, alias, anchor snapshot, movement ledger, stock replay, reconciliation, issue center, control center, export, notification และ retention ตามลำดับ ทุก write action ต้องมี validation และ audit ทุก page ต้องมี state ครบ ทุก result ต้องอธิบายได้ และถ้าข้อมูลจริงยังไม่พอต้องระบุ assumption/risk/TODO ไม่ใช่ hardcode หรือเดา logic เอง
```

