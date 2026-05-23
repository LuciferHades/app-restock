# Warehouse Reconciliation ERP Platform — AI Blueprint Backup

เอกสารนี้ใช้เป็น **ไฟล์สำรองบทสนทนา + Blueprint Package** สำหรับส่งต่อให้ AI Agent / Developer / Codex เพื่อสร้างระบบ Web App แบบ clean, professional, ใช้งานจริง และไม่หลุด scope

---

## 0. Executive Summary

ระบบนี้ไม่ใช่ระบบนับสต๊อกธรรมดา แต่เป็น **Warehouse Reconciliation ERP Platform** ที่มีหน้าที่เชื่อมข้อมูล 5 ชั้น:

1. **Physical Count** = ของจริงที่นับได้
2. **Field Movement** = สิ่งที่หน้างานแจ้งว่าเกิดขึ้นจริง
3. **System In-Out** = movement ที่ ERP post จริง
4. **Inquiry** = snapshot ยอดคงเหลือจาก ERP
5. **Expected Stock / Stock Replay** = ยอดที่ระบบคำนวณจาก Anchor + Movement Ledger

เป้าหมายคือให้ระบบช่วยคิดแทนคน:

- เช็คเองว่าข้อมูลตรงหรือไม่ตรง
- บอกว่าไม่ตรงตรงไหน
- บอกว่าสินค้า/ลูกค้า/location/lot อะไรเกี่ยวข้อง
- เสนอ next action
- สร้าง issue/task ให้คนที่เกี่ยวข้อง
- แจ้งเตือนผ่าน In-app / Telegram / LINE Messaging API / Email
- Export รายงาน/ไฟล์ ERP template ได้
- มี audit log และ lock/close control

แนวคิดสำคัญ:

```text
ระบบไม่ควรให้คนไล่เช็คทุกแถว
ระบบควรเช็คให้ก่อน แล้วให้คนดูเฉพาะ exception / approval / export / close day
```

---

# 1. Current Decision Snapshot

## 1.1 คะแนนแนวคิดระบบปัจจุบัน

| ด้าน | คะแนนประเมิน |
|---|---:|
| Business Fit | 94–95 |
| Operation Realism | 92–95 |
| Data Logic | 90–92 |
| Auto Logic / Reconciliation | 90–92 |
| UX/UI Direction | 86–89 |
| Architecture Direction | 88–91 |
| Build Readiness | 72–76 |
| Production Readiness | 68–76 |

**สรุป:** แนวคิดระบบอยู่ระดับ 90+ แล้ว แต่ความพร้อมสร้างจริงยังรอการ lock schema จากข้อมูล Inquiry / In-Out จริง และ Alias Strategy v0

---

## 1.2 จุดติดที่ยังเหลือ

### P0 — ต้องแก้ก่อนทำระบบลึก

1. **Data Profiling จากไฟล์ Inquiry/In-Out จริง**
   - column จริงคืออะไร
   - DC/Location เรียกไม่ตรงกันยังไง
   - Customer เช่น MKMK / UPUP / UFUF อยู่ตรงไหน
   - Item / Year / Lot อยู่ในชื่อหรือแยก column
   - Qty / UOM format เป็นอย่างไร
   - Movement type มี code อะไร

2. **Alias Strategy**
   - แยก Customer Alias, Item Alias, Location Alias, DC Alias, UOM Alias
   - ห้ามรวมทุก alias ไว้ table เดียวแบบมั่ว
   - ต้องมี Alias Review Queue

3. **Anchor Snapshot + Stock Replay**
   - Inquiry ไม่จำเป็นต้องมีทุกวัน
   - Admin อาจ upload ข้ามวัน 4–5 วัน
   - ใช้ In-Out ย้อนหลัง replay stock position

4. **Database Decision**
   - MVP: Google Sheets / Apps Script ได้
   - ระบบจริง: Supabase/Postgres หรือ MySQL เหมาะกว่า
   - Supabase Free ใช้ได้สำหรับ MVP ถ้ามี retention 45 วัน + summary/archive

5. **User Approval + Permission Model**
   - สมัครแล้วต้องรออนุมัติ
   - Admin กำหนด role, permission, warehouse scope
   - Reject แล้วลบ request แต่เก็บ audit log

6. **Issue/Result Code + Explanation Structure**
   - ไม่ใช่แค่ QTY_DIFF
   - ต้องแสดง Field vs ERP vs Count vs Expected พร้อม explanation และ suggested action

---

## 1.3 20/80 Core ที่ควรทำก่อน

ทำ 20% นี้ก่อน จะได้ผล 80% ของระบบ:

1. Master Alias + Smart Input
2. Inquiry Anchor Snapshot
3. In-Out Import แบบ Chunk + Backfill
4. Movement Ledger + Stock Replay
5. Auto Reconciliation + Explanation Detail
6. Issue / Exception Center
7. User Approval + Permission
8. Control Center + Export + Notification

เลื่อนออกไปก่อน:

- Relocation Optimizer เต็มรูปแบบ
- Visual Warehouse Map
- Advanced FIFO
- Cross-Warehouse Full Workflow
- Offline Mode เต็มรูปแบบ
- Advanced AI Matching
- Full Custody Contract Workflow

---

# 2. File Package ที่ควรสร้างให้ AI Agent

```text
/docs
  00_ai_persona.md
  01_product_brief.md
  02_structure.md
  03_module_spec.md
  04_user_role_permission.md
  05_workflow.md
  06_information_architecture.md
  07_design.md
  08_design_system.md
  09_responsive_spec.md
  10_data_model.md
  11_api_backend_spec.md
  12_business_rules.md
  13_validation_rules.md
  14_audit_log_security.md
  15_dashboard_kpi_report.md
  16_error_empty_loading_state.md
  17_out_of_scope.md
  18_development_plan.md
  19_ai_agent_task.md
  20_acceptance_criteria.md
  21_database_retention_backup.md
  22_notification_center.md
  23_import_backfill_spec.md
  24_current_remaining_work.md
```

ถ้าใช้ GAS + Google Sheets เพิ่ม:

```text
  25_gas_architecture.md
  26_sheet_database_schema.md
  27_deployment_guide.md
```

ถ้าใช้ Supabase เพิ่ม:

```text
  28_supabase_architecture.md
  29_supabase_schema_policy.md
  30_supabase_cron_retention.md
```

---

# 00_ai_persona.md

## Role

AI Agent ต้องทำหน้าที่เป็น:

- Senior Product Architect
- Senior Full-Stack Developer
- UX/UI Designer
- Data Analyst
- System Analyst
- Workflow Analyst
- Warehouse Operation Consultant
- ERP Reconciliation Architect

## Core Thinking

AI ต้องคิดแบบ:

- Data-Driven Thinking
- System Thinking
- Critical Thinking
- Forecast Thinking
- Operation Thinking
- UX/UI Thinking
- Audit Thinking
- Security Thinking
- Performance Thinking

## Rules

1. ห้ามเขียนโค้ดก่อนอ่าน docs
2. ห้ามสร้างฟีเจอร์ที่ไม่มีใน docs
3. ห้าม hardcode column mapping
4. ห้าม hardcode alias
5. ห้าม hardcode permission เฉพาะใน UI
6. ห้ามคำนวณ dashboard จาก raw table โดยตรง
7. ทุก write action สำคัญต้องมี validation + audit log
8. ทุก issue ต้องมี result_code, severity, confidence, explanation, suggested_action
9. ทุก import ต้องเข้า staging/preview ก่อน normalize
10. ทุก raw data ต้องมี batch_id/source_id
11. ห้ามลบ official data โดยไม่มี retention log
12. ถ้าข้อมูลไม่พอ ต้องระบุ assumption/risk ไม่เดาเอง

---

# 01_product_brief.md

## Product Vision

สร้างระบบ Warehouse Reconciliation ERP Platform ที่ช่วยเชื่อมข้อมูลของจริง, หน้างาน, และ ERP เพื่อให้รู้ว่า stock อยู่ที่ไหน จำนวนเท่าไร ERP post ครบไหม และต้องให้ใครทำอะไรต่อ

## Problem Statement

คลังมีปัญหา:

- ERP ไม่ real-time
- Inquiry เป็น snapshot และอาจไม่ได้ upload ทุกวัน
- In-Out ดึงย้อนหลังได้ แต่ต้อง replay stock
- หน้างานแจ้งผ่านปากเปล่า/Line/ป้ายหน้ากอง
- ชื่อย่อไม่ตรงกับ ERP เช่น MKMK, UPUP, UFUF, W150, RE T1
- Location/DC ไม่ตรงกัน
- แจ้งออกพอรู้ของเดิม แต่แจ้งเข้ามักไม่รู้ข้อมูลครบ
- REJ แจ้งแล้วอาจไม่ถูก post ใน ERP
- ของฝาก/ลูกค้านอก/ย้ายเศษทำให้ owner location กับ physical location ไม่ตรง

## Business Goals

1. ลดการไล่ Excel/Line/ERP เอง
2. ให้ระบบเช็คความตรง/ไม่ตรงอัตโนมัติ
3. ให้คนดูเฉพาะ exception
4. ปิดวัน/ปิดรอบได้มั่นใจ
5. Export รายงาน/ไฟล์ ERP ได้
6. Audit ย้อนหลังได้
7. รองรับ admin ไม่ upload ทุกวัน
8. รองรับระบบลบ/เก็บข้อมูลเกิน 45 วันอัตโนมัติ

## MVP Scope

- User Approval + Permission
- Import Center
- Alias Center
- Inquiry Anchor Snapshot
- In-Out Backfill
- Movement Ledger
- Stock Replay
- Auto Reconciliation
- Issue Center
- Explanation Detail
- Control Center
- Basic Export
- Notification Center basic
- Retention 45 วัน

## Out of MVP

- Relocation Optimizer เต็มรูปแบบ
- Advanced FIFO
- Cross-warehouse full custody workflow
- Full visual warehouse map
- Offline full sync

---

# 02_structure.md

## System Overview

ระบบแบ่งเป็น 6 layer:

```text
1. Control Core
   - User / Permission
   - Master Data
   - Alias
   - Import
   - Snapshot
   - Lock
   - Audit

2. Operational Engine
   - Field Movement
   - Physical Count
   - REJ
   - Transfer
   - Checker Task
   - GPS Session

3. Data Engine
   - Movement Ledger
   - Anchor Snapshot
   - Stock Replay
   - Daily Stock Position
   - Data Coverage Check

4. Intelligence Engine
   - Rule Engine
   - Reconciliation Engine
   - Explanation Engine
   - Issue Engine
   - Simulation Engine later

5. Decision UI
   - Control Center
   - Exception Center
   - Detail View
   - Backfill Wizard
   - Export Center
   - Notification Center

6. Maintenance Layer
   - Retention 45 วัน
   - Health Check
   - Backup Export
   - Notification Log
```

## Main Modules

| Module | Purpose |
|---|---|
| User Approval | สมัคร/อนุมัติ/ให้สิทธิ์/ล็อคผู้ใช้ |
| Permission | จัดการ role, permission, warehouse scope |
| Checker GPS Session | checker check-in ต่อ warehouse ต่อวัน |
| Import Center | upload Inquiry/In-Out/Count แบบ preview/chunk |
| Alias Center | map ชื่อย่อ/ชื่อ ERP/ชื่อหน้างานให้เป็น master เดียว |
| Anchor Snapshot | เก็บ Inquiry เป็นจุดอ้างอิง stock |
| Movement Ledger | รวมทุก movement ที่มีผลต่อ stock |
| Stock Replay | reconstruct stock ข้ามวันจาก Anchor + Ledger |
| Reconciliation Engine | เทียบ Field/System/Count/Expected |
| Issue Center | รายการผิด/เสี่ยง/ต้อง action |
| Explanation Detail | บอกว่าไม่ตรงยังไงและต้องทำอะไร |
| Notification Center | ตั้งค่า Telegram/LINE/Email/In-app |
| Export Center | export selected/filter rows, report, ERP template |
| Lock Control | Preliminary/Final Close/Locked |
| Retention | ลบ/archive raw เกิน 45 วัน |

## Data Flow

```text
Upload File
→ Import Batch
→ Raw Rows
→ Column Mapping
→ Alias Mapping
→ Normalized Records
→ Movement Ledger / Snapshot
→ Stock Replay
→ Reconciliation Result
→ Issue / Task / Notification
→ Summary / Dashboard / Export
→ Close / Lock / Archive
```

---

# 03_module_spec.md

| Module | Purpose | Main User | Main Action | Data Used | Output | Risk |
|---|---|---|---|---|---|---|
| User Approval | อนุมัติคนสมัคร | Admin | Approve/Reject | USER_ACCESS_REQUESTS | USER_ACCOUNTS | ถ้าไม่มี audit อาจตรวจไม่ได้ |
| Import Center | รับไฟล์จริง | Admin/Staff | Upload/Preview/Confirm | Excel | IMPORT_BATCH/RAW | ถ้า hardcode column จะพัง |
| Alias Center | แก้ชื่อไม่ตรง | Admin | Map/Create/Reject | Unknown text | Alias master | ถ้า alias รวมกันหมดจะมั่ว |
| Stock Replay | คำนวณย้อนหลัง | System | Run Replay | Anchor + Ledger | Daily Position | ถ้า In-Out ขาด confidence ต่ำ |
| Reconciliation | ตรวจตรง/ไม่ตรง | System | Auto check | Field/System/Count | Results/Issues | ถ้า match key ผิดจะ false issue |
| Issue Center | คุม exception | Supervisor/Admin | Review/Assign/Close | Reconciliation Result | Task/Action | ถ้าไม่มี owner issue จะค้าง |
| Notification | แจ้งเตือน | Admin/System | Send/Retry | Rules/Templates | Logs | Spam ถ้าไม่มี cooldown |
| Export | ส่งออกข้อมูล | Admin/ERP Admin | Export selected/filter | Results/Issues | File/Log | ถ้าไม่มี export spec จะเปลี่ยนบ่อย |
| Retention | ลบ/archive อัตโนมัติ | System/Admin | Cleanup | active/raw data | Retention log | ลบ open issue ไม่ได้ |

---

# 04_user_role_permission.md

## Roles

- Admin
- Supervisor
- Planner
- ERP Admin
- Checker
- Viewer
- External Warehouse later

## Permission Concept

```text
Role = template
Permission = สิทธิ์จริง
Warehouse Scope = ใช้ได้ที่คลังไหน
Checker Session = สิทธิ์นับในวันนั้นหลัง check-in
```

## Core Permissions

| Permission | Description |
|---|---|
| CAN_LOGIN | เข้าใช้งานระบบ |
| CAN_MANAGE_USER | อนุมัติ/แก้/ล็อค user |
| CAN_MANAGE_PERMISSION | จัดการสิทธิ์ |
| CAN_IMPORT_INQUIRY | upload Inquiry |
| CAN_IMPORT_INOUT | upload System In-Out |
| CAN_MANAGE_ALIAS | map alias |
| CAN_CREATE_FIELD_MOVEMENT | แจ้ง movement |
| CAN_COUNT | นับจริง |
| CAN_CREATE_REJ | แจ้ง REJ |
| CAN_RUN_RECONCILIATION | run auto check |
| CAN_REVIEW_ISSUE | ดู/แก้ issue |
| CAN_APPROVE | อนุมัติ |
| CAN_EXPORT | export |
| CAN_CLOSE_DAY | close day |
| CAN_UNLOCK_DAY | unlock day |
| CAN_MANAGE_NOTIFICATION | ตั้งค่าแจ้งเตือน |

## Rule

- Permission ต้องเช็คทั้ง frontend และ backend
- Checker ต้องมี active session ก่อน count
- Reject user request ให้ลบ request แต่เก็บ audit log
- User หนึ่งคนมีหลาย role และ permission override ได้

---

# 05_workflow.md

## Workflow 1: User Registration & Approval

```text
User สมัคร / ตั้งรหัส
→ USER_ACCESS_REQUESTS status = PENDING_APPROVAL
→ Admin เปิด User Approval Center
→ Approve หรือ Reject
```

### Approve

```text
เลือก role/permission/warehouse scope
→ สร้าง USER_ACCOUNTS
→ status = ACTIVE
→ audit log
→ notification user/admin
```

### Reject

```text
Reject with reason
→ ลบ request จากคิว
→ เก็บ audit log
```

---

## Workflow 2: Import Inquiry / In-Out

```text
Upload file
→ Client-side parse
→ Preview headers/rows
→ Column Mapping
→ Chunk Upload
→ Save Raw Rows
→ Normalize
→ Alias Mapping
→ Unknown เข้า Alias Review
→ Confirm Import
→ Run job ต่อ
```

---

## Workflow 3: Admin ข้ามวัน / Backfill

```text
เลือกช่วงวันที่ขาด 1–5 วัน
→ Upload In-Out ย้อนหลัง
→ Coverage Check
→ หา Anchor Snapshot ล่าสุดก่อนช่วง
→ Stock Replay
→ Reconciliation รายวัน
→ สร้าง Count Required Locations
→ สร้าง Issue/Task
→ Update Reconciliation Calendar
```

---

## Workflow 4: Pending Inbound

```text
ของเข้า แต่ยังไม่รู้ข้อมูลครบ
→ Checker/Staff กรอกข้อความจากป้าย + location + qty + photo
→ status = PENDING_INBOUND_VERIFY
→ วันถัดไป In-Out เข้า
→ ระบบเสนอ candidate match
→ Admin/Supervisor confirm
→ convert to canonical item/customer/location
```

---

## Workflow 5: Auto Reconciliation

```text
Normalized Data
→ Build Matching Key
→ Compare Field vs System
→ Compare Expected vs Count
→ Compare REJ vs ERP
→ Generate Result Code
→ Calculate Severity + Confidence
→ Create Issue/Task
→ Notify
```

---

# 06_information_architecture.md

## Main Navigation

### Admin Desktop

- Control Center
- Import Center
- Backfill Wizard
- Alias Review
- User Approval
- User & Permission
- Issue Center
- Export Center
- Notification Center
- Lock / Close Control
- Audit Log
- Maintenance / Retention

### Checker Mobile

- Check-in
- งานของฉัน
- แจ้งเข้า
- แจ้งออก
- นับจริง
- แจ้ง REJ
- ประวัติล่าสุด

### Supervisor

- Control Center
- Issue Queue
- Pending Approval
- REJ Aging
- Count Required Locations
- Daily Close Readiness

### ERP Admin

- Export Pending
- ERP Post Confirmation
- SYSTEM_MISSING
- REJ_NOT_POSTED

---

# 07_design.md

## UX Principles

1. Mobile-first for Checker
2. Exception-only for Supervisor
3. Summary-first for Admin
4. Detail-on-demand
5. Progressive disclosure
6. Smart input, not long dropdown
7. Clear status and next action
8. Prevent wrong data before saving

## Key Pages

### Control Center

Cards:

- Matched
- Issue
- Critical
- Unknown Alias
- Pending Inbound
- REJ Not Posted
- Export Pending
- Close Status
- Cleanup Status

### Smart Inbound Input

ช่องหลัก:

```text
พิมพ์ข้อความจากป้าย เช่น "MKMK W150 69"
```

ระบบแสดง preview:

```text
Customer: MKMK
Item: W150
Year: 69
Confidence: 95%
```

ถ้าไม่รู้:

```text
บันทึกเป็น Pending Inbound
แนบรูปป้าย
รอ In-Out ยืนยัน
```

### Issue Detail

ต้องแสดง:

- result_code
- severity
- confidence
- customer/item/year/lot/location
- field qty
- system qty
- before count
- after count
- expected qty
- diff
- explanation
- likely cause
- suggested action
- action buttons

---

# 08_design_system.md

## Color Tokens

| Token | Use |
|---|---|
| Blue | primary action / info |
| Green | matched / success |
| Yellow | warning / pending |
| Red | critical / failed |
| Gray | neutral / inactive |
| Purple | custody / external |

## Status Badge

| Status | Color |
|---|---|
| MATCHED | Green |
| PENDING | Yellow |
| WARNING | Yellow/Orange |
| CRITICAL | Red |
| UNKNOWN | Gray |
| LOCKED | Dark Gray |

## UI Pattern

- Card-based dashboard
- Table with filters
- Detail drawer/modal
- Sticky action bar on mobile
- Large touch target for checker
- Loading, empty, error states always required

---

# 09_responsive_spec.md

## Breakpoints

- Mobile: < 768px
- Tablet: 768–1024px
- Desktop: > 1024px

## Mobile

- Bottom navigation for Checker
- One main action per screen
- Large buttons
- Minimal fields
- Camera/photo support
- Last used location/customer/item

## Desktop

- Sidebar navigation
- Dashboard cards
- Full tables
- Advanced filters
- Import/export tools

## Tablet

- Supervisor review
- Issue card grid
- Quick approve / assign task

---

# 10_data_model.md

## Core Tables

### users

- user_id
- username
- email
- hashed_password
- salt
- full_name
- phone
- status
- created_at
- updated_at

### user_access_requests

- request_id
- username
- email
- full_name
- phone
- requested_role
- status
- requested_at
- approved_by
- approved_at
- rejected_by
- rejected_at
- rejection_reason

### roles / permissions

- role_id
- role_name
- permission_code

### warehouses / locations

- warehouse_id
- warehouse_code
- warehouse_name
- location_id
- location_code
- location_name
- capacity_qty
- active

### alias tables

- customer_aliases
- item_aliases
- location_aliases
- dc_aliases
- uom_aliases

Common fields:

- alias_id
- alias_text
- canonical_id
- alias_type
- confidence
- active
- created_by
- created_at

### import_batches

- batch_id
- file_type
- file_name
- business_date_from
- business_date_to
- uploaded_by
- uploaded_at
- row_count
- checksum
- status

### import_raw_rows

- raw_row_id
- batch_id
- row_no
- raw_json
- parse_status
- error_message

### inquiry_snapshots

- snapshot_id
- batch_id
- warehouse_id
- business_date
- snapshot_time
- uploaded_at
- uploaded_by
- active_flag
- checksum
- status

### movement_ledger

- ledger_id
- business_date
- movement_datetime
- source_type
- source_ref_id
- movement_type
- from_location_id
- to_location_id
- customer_id
- item_id
- year
- lot
- qty
- uom
- confidence
- status

### daily_stock_positions

- business_date
- warehouse_id
- location_id
- customer_id
- item_id
- year
- lot
- opening_qty
- in_qty
- out_qty
- transfer_in_qty
- transfer_out_qty
- rej_qty
- ending_expected_qty
- source_mode
- confidence
- last_rebuilt_at

### recon_results

- result_id
- business_date
- source_ref_id
- result_code
- severity
- confidence
- explanation
- likely_cause
- suggested_action
- owner_role
- status

### issues

- issue_id
- result_id
- issue_type
- severity
- owner_role
- assigned_to
- status
- due_at
- closed_at

### export_jobs / export_logs

- export_id
- export_type
- filters_json
- row_count
- file_url
- status
- exported_by
- exported_at

### audit_logs

- log_id
- actor_id
- action
- entity_type
- entity_id
- old_value
- new_value
- reason
- created_at

### retention_logs

- retention_id
- cutoff_date
- deleted_rows
- skipped_open_issues
- status
- message
- created_at

---

# 11_api_backend_spec.md

## API Pattern

Every response:

```json
{
  "success": true,
  "data": {},
  "message": "",
  "code": "",
  "meta": {}
}
```

Every error:

```json
{
  "success": false,
  "code": "VALIDATION_ERROR",
  "message": "ข้อมูลไม่ถูกต้อง",
  "details": {}
}
```

## Core APIs

### Auth

- registerUser
- approveUser
- rejectUser
- login
- logout
- verifySession
- updatePermission

### Import

- createImportBatch
- uploadRawChunk
- previewImport
- saveColumnMapping
- normalizeBatch
- confirmImport

### Alias

- listAliasQueue
- mapAlias
- createMasterFromAlias
- rejectAlias

### Stock Replay

- runStockReplay(date_from, date_to)
- getDailyStockPosition
- getCoverageCheck

### Reconciliation

- runReconciliation(date_from, date_to)
- getReconResults
- getIssueDetail
- createInvestigation

### Export

- createExportJob
- exportSelectedRows
- exportFilteredRows
- getExportLog

### Notification

- saveChannel
- saveRule
- saveTemplate
- testSend
- getNotificationLogs

### Maintenance

- runRetentionJob
- getHealthStatus
- getRetentionLogs

---

# 12_business_rules.md

## Core Rules

| Rule ID | Rule |
|---|---|
| BR-001 | Inquiry เป็น Snapshot ไม่ใช่ Transaction |
| BR-002 | 1 business_date มี active inquiry ได้หลาย batch เฉพาะถ้า archive/replace policy ชัดเจน |
| BR-003 | ห้าม append Inquiry ซ้ำแบบไม่ตั้ง batch/snapshot |
| BR-004 | Field Movement และ System In-Out ต้องแยกกัน |
| BR-005 | Stock position ต้องมาจาก Anchor + Movement Ledger |
| BR-006 | ถ้าไม่มี Inquiry ทุกวัน ให้ใช้ Stock Replay |
| BR-007 | Unknown alias ต้องเข้า Alias Review |
| BR-008 | Pending Inbound ต้อง confirm จาก In-Out หรือ Admin ก่อนปิด final |
| BR-009 | Issue ที่ยัง open ห้ามถูกลบจาก retention |
| BR-010 | LOCKED day ห้ามแก้ตรง ต้องผ่าน Correction/Investigation |
| BR-011 | Checker ต้อง check-in ก่อนนับ |
| BR-012 | Export ทุกครั้งต้องมี export log |
| BR-013 | Notification ต้องมี cooldown/send-once option |
| BR-014 | Dashboard ต้องอ่าน summary/result ไม่อ่าน raw full history |

---

# 13_validation_rules.md

## Import Validation

- file type allowed
- header exists
- row count > 0
- required column mapped
- qty numeric
- date valid
- duplicate batch checksum warning
- unknown alias count

## Alias Validation

- alias text required
- alias type required
- canonical target required if mapping
- duplicate alias warning
- confidence recorded

## Movement Validation

- movement_type required
- qty > 0
- at least one location required
- customer/item may be unknown only for Pending Inbound
- locked date cannot accept direct edit

## User Validation

- username unique
- email unique
- password hash required before active
- approval required before login
- permission required for every backend action

---

# 14_audit_log_security.md

## Actions requiring audit

- USER_REGISTER
- USER_APPROVE
- USER_REJECT_DELETE_REQUEST
- USER_LOCK
- PERMISSION_CHANGE
- IMPORT_UPLOAD
- IMPORT_CONFIRM
- ALIAS_MAP
- ALIAS_CREATE_MASTER
- RUN_STOCK_REPLAY
- RUN_RECONCILIATION
- ISSUE_ASSIGN
- ISSUE_CLOSE
- EXPORT_CREATE
- DAILY_CLOSE
- DAILY_LOCK
- RETENTION_DELETE
- NOTIFICATION_RULE_CHANGE

## Security Rules

- Password must be hash + salt
- Tokens expire
- Permission check backend side
- Do not expose token/secret to frontend
- Notification token stored securely
- File URLs should not be public unless intended
- Use audit log for all critical changes

---

# 15_dashboard_kpi_report.md

## Control Center KPIs

| KPI | Formula | Decision |
|---|---|---|
| Matched Count | count result_code = MATCHED | งานปกติ |
| Critical Issue | count severity = CRITICAL and status open | ต้องแก้ด่วน |
| Unknown Alias | count alias_queue pending | ต้อง map master |
| Pending Inbound | count pending inbound open | ต้องรอ/confirm In-Out |
| REJ Not Posted | count REJ_NOT_POSTED | ส่ง ERP Admin |
| Close Readiness | gate result | ปิดวันได้ไหม |
| DB Usage | storage usage | เสี่ยง free limit |
| Retention Status | last job status | cleanup ทำงานไหม |

## Reports

- Daily Reconciliation Report
- Issue Aging Report
- Pending Inbound Report
- Alias Review Report
- REJ Report
- Export Log Report
- Retention Report
- User Approval Report

---

# 16_error_empty_loading_state.md

## Required UI States

### Loading

- Importing file
- Normalizing data
- Running stock replay
- Running reconciliation
- Exporting
- Sending notification

### Empty

- No issue
- No pending alias
- No pending inbound
- No export pending
- No user approval pending

### Error

- Import failed
- Unknown schema
- Permission denied
- Session expired
- Reconciliation failed
- Notification failed
- Retention failed

### Confirmation

- Approve user
- Reject and delete request
- Confirm import
- Map alias
- Close issue
- Lock day
- Run retention cleanup

---

# 17_out_of_scope.md

## Not in MVP

- Relocation Optimizer เต็มรูปแบบ
- Advanced FIFO engine
- Visual warehouse map
- Full custody contract management
- Cross-warehouse workflow full cycle
- Offline-first sync
- AI auto matching แบบฟันธง
- Full mobile app native

## Future Phase

- Capacity Simulation
- Move small to large suggestion
- Plan reservation
- Multi-scenario storage planning
- Photo OCR later
- Advanced forecast

---

# 18_development_plan.md

## Phase 1 — Foundation

Tasks:

- User Approval
- Permission
- Audit Log
- Master Alias
- Import Staging
- Column Mapping

Acceptance:

- สมัครแล้วรออนุมัติได้
- Admin approve/reject ได้
- Upload file แล้ว preview headers/rows ได้
- Unknown alias เข้า queue ได้

## Phase 2 — Data Engine

Tasks:

- Inquiry Anchor Snapshot
- In-Out Backfill Import
- Movement Ledger
- Stock Replay
- Daily Stock Position

Acceptance:

- Upload ข้ามวันได้
- Replay stock from anchor ได้
- Coverage check แสดงวันที่ข้อมูลขาดได้

## Phase 3 — Auto Check

Tasks:

- Reconciliation Engine
- Result Code
- Explanation Detail
- Issue Center
- Count Required Locations

Acceptance:

- ระบบบอก diff พร้อม explanation ได้
- สร้าง issue/task ได้

## Phase 4 — Control

Tasks:

- Control Center
- Notification Center
- Export Center
- Lock/Close Control
- Retention Job

Acceptance:

- เห็น status รวม
- ส่งแจ้งเตือน
- export ได้
- cleanup 45 วันมี log

## Phase 5 — Advanced

Tasks:

- Capacity
- Simulation
- Relocation Suggestion
- Custody
- Advanced FIFO

---

# 19_ai_agent_task.md

## AI Agent Development Rules

1. อ่าน docs ทั้งหมดก่อนเขียนโค้ด
2. เริ่มจาก Foundation ไม่ใช่ Simulation
3. ห้าม hardcode column mapping
4. ห้าม hardcode alias
5. ห้ามเพิ่ม feature นอก scope
6. ทุก module ต้องมี service layer
7. ทุก write action ต้องมี validation + audit
8. ทุก page ต้องมี loading/error/empty state
9. ทุก result ต้องมี explanation
10. Dashboard อ่าน summary/result เท่านั้น
11. ถ้าข้อมูลไม่พอ ให้สร้าง TODO/Question ไม่เดาเอง

## Build Order

1. File structure
2. Config / environment
3. Auth / User Approval
4. Permission
5. Audit log
6. Import staging
7. Alias center
8. Anchor snapshot
9. Movement ledger
10. Stock replay
11. Reconciliation
12. Issue center
13. Control center
14. Export center
15. Notification
16. Retention
17. Tests

---

# 20_acceptance_criteria.md

## Product Acceptance

- [ ] ระบบรับไฟล์ Inquiry/In-Out ได้
- [ ] ระบบไม่ต้องมี Inquiry ทุกวันก็ replay ได้
- [ ] ระบบ map alias ได้
- [ ] ระบบสร้าง pending inbound ได้
- [ ] ระบบบอก diff ได้พร้อม explanation
- [ ] ระบบสร้าง issue/task ได้
- [ ] ระบบ export ได้
- [ ] ระบบแจ้งเตือนได้
- [ ] ระบบ cleanup เกิน 45 วันได้โดยไม่ลบ open issue

## UX Acceptance

- [ ] Checker ใช้มือถือได้
- [ ] Admin เห็น Control Center ชัด
- [ ] Issue detail อธิบายได้ว่าไม่ตรงยังไง
- [ ] Alias Review ใช้ง่าย
- [ ] Backfill Wizard ใช้ง่าย

## Security Acceptance

- [ ] สมัครแล้วต้องรออนุมัติ
- [ ] Reject ลบ request แต่มี audit
- [ ] Password hash/salt
- [ ] Permission ตรวจ backend
- [ ] Export/Lock/Retention มี audit

---

# 21_database_retention_backup.md

## Retention Policy

Active raw data เก็บ 45 วัน

ลบได้:

- import_raw_rows ที่ normalized/closed แล้ว
- staging data
- matched/closed movement เก่า
- sent/skipped notification logs

ห้ามลบ:

- open issue
- pending approval
- pending export
- REJ_NOT_POSTED
- SYSTEM_MISSING
- correction log
- export log
- daily close summary
- critical audit log

## Nightly Job

```text
02:00 ทุกคืน
→ health check
→ create summary
→ cutoff = today - 45 days
→ skip open issue
→ archive summary
→ delete raw/closed records
→ write retention log
→ send notification
```

## Health Check

- DB usage
- table row count
- last import
- last reconciliation
- last cleanup
- open critical issue
- pending alias
- pending inbound

---

# 22_notification_center.md

## Channels

- In-app Alert
- Telegram
- LINE Messaging API
- Email

## Important note

LINE Notify ไม่ควรใช้เป็นระบบใหม่ เพราะต้องใช้ LINE Messaging API แทนในอนาคต

## Event Codes

- USER_PENDING_APPROVAL
- UNKNOWN_ALIAS_CREATED
- PENDING_INBOUND_CREATED
- SYSTEM_MISSING
- FIELD_MISSING
- QTY_DIFF
- LOCATION_DIFF
- REJ_NOT_POSTED
- CAPACITY_EXCEEDED
- MULTI_LOT_WARNING
- BACKFILL_COMPLETED
- DAILY_CLOSE_NOT_READY
- EXPORT_READY
- RETENTION_FAILED
- DB_USAGE_WARNING

## Tables

- notification_channels
- notification_rules
- notification_templates
- notification_logs

## Anti-spam

- severity filter
- cooldown_minutes
- send_once_per_issue
- daily digest
- escalation by SLA

---

# 23_import_backfill_spec.md

## Import Types

- Inquiry
- System In-Out
- Field Count
- Master Data

## Required Flow

```text
Upload
→ Client parse
→ Preview
→ Column mapping
→ Chunk upload
→ Raw staging
→ Normalize
→ Alias matching
→ Unknown queue
→ Confirm
→ Run replay/reconciliation
```

## Backfill Wizard

Use case:

- Admin หยุด/ลา
- ไม่มี Inquiry หลายวัน
- ดึง In-Out ย้อนหลัง 1–5 วัน

Flow:

```text
เลือก date range
→ upload in-out
→ coverage check
→ run stock replay
→ run reconciliation
→ create count tasks
→ show summary
```

## Coverage Status

- COVERAGE_OK
- MISSING_SYSTEM_INOUT
- MISSING_ANCHOR
- PENDING_UNKNOWN_INBOUND
- NO_COUNT_FOR_MOVING_LOCATION
- REJ_PENDING

---

# 24_current_remaining_work.md

## ตอนนี้เหลืออะไรบ้าง

### A. ต้องทำก่อนสร้าง reconciliation จริง

1. Data Profiling จากไฟล์ Inquiry/In-Out จริง
2. สรุป Canonical Field Contract
3. ทำ Alias Strategy v0
4. ทำ Column Mapping Config
5. ทำตัวอย่าง Reconciliation Test Case 10–20 เคส

### B. ต้องตัดสินใจก่อน build architecture

1. ใช้ Google Sheets เป็น MVP หรือใช้ Supabase เลย
2. เก็บรูป/ไฟล์ที่ไหน
3. Retention 45 วันจะลบ raw หรือ archive summary
4. ใช้ Telegram เป็นหลักหรือ LINE Messaging API
5. Export format จริงที่ ERP Admin ต้องใช้คืออะไร

### C. ต้องทำ prototype UI ก่อน dev หนัก

1. Smart Inbound Input
2. Alias Review
3. Backfill Wizard
4. Issue Detail
5. Control Center

### D. ต้องมี sample data

- Inquiry 3–5 ไฟล์
- In-Out 3–5 วัน
- ตัวอย่างป้ายหน้ากอง
- ตัวอย่างชื่อย่อสินค้า/ลูกค้า
- ตัวอย่าง location/DC ที่ไม่ตรง
- ตัวอย่าง REJ
- ตัวอย่าง pending inbound

### E. ต้องมี Test Case

- Inquiry duplicate
- In-Out backfill 5 วัน
- Unknown alias
- Pending inbound confirm
- Field vs System matched
- QTY_DIFF
- LOCATION_DIFF
- REJ_NOT_POSTED
- open issue ไม่ถูก retention delete
- user reject delete request with audit

---

# MASTER PROMPT: ให้ AI สร้างเอกสารทั้งหมด

```text
คุณคือ Senior Product Architect, Senior Full-Stack Developer, UX/UI Designer, Data Analyst, System Analyst, Workflow Analyst และ Warehouse Operations Consultant

ฉันต้องการสร้าง Warehouse Reconciliation ERP Platform แบบ clean, professional, data-driven, system-thinking, mobile-first และพร้อมพัฒนาต่อจริง

ก่อนเขียนโค้ด ให้คุณสร้างชุดเอกสารวางระบบทั้งหมดก่อน เพื่อให้ AI Agent หรือ Developer สามารถนำไปสร้าง Web App จริงได้อย่างเป็นระบบ

ระบบนี้ต้องรองรับ:
- Inquiry เป็น Anchor Snapshot ไม่ใช่ Transaction
- Admin อาจไม่ upload ทุกวัน และต้อง upload/backfill ข้ามวัน 1–5 วัน
- System In-Out ใช้เป็น Movement Ledger ย้อนหลัง
- Stock Replay เพื่อคำนวณ Daily Stock Position
- Field Movement / Physical Count / REJ / Transfer
- Master Alias: Customer, Item, Location, DC, UOM
- Smart Input จากป้าย เช่น MKMK W150 69
- Pending Inbound เมื่อตอนเข้าไม่รู้ข้อมูลครบ
- Auto Reconciliation พร้อม result_code, severity, confidence, explanation, suggested_action
- Issue / Exception Center
- User Approval + Permission + Warehouse Scope
- Checker GPS Daily Session
- Notification Center: In-app, Telegram, LINE Messaging API, Email
- Export Center
- Lock / Close Control
- Retention 45 วัน พร้อม archive/summary/audit

ให้สร้างเอกสาร Markdown แยกเป็นไฟล์:
00_ai_persona.md
01_product_brief.md
02_structure.md
03_module_spec.md
04_user_role_permission.md
05_workflow.md
06_information_architecture.md
07_design.md
08_design_system.md
09_responsive_spec.md
10_data_model.md
11_api_backend_spec.md
12_business_rules.md
13_validation_rules.md
14_audit_log_security.md
15_dashboard_kpi_report.md
16_error_empty_loading_state.md
17_out_of_scope.md
18_development_plan.md
19_ai_agent_task.md
20_acceptance_criteria.md
21_database_retention_backup.md
22_notification_center.md
23_import_backfill_spec.md
24_current_remaining_work.md

กฎสำคัญ:
1. ห้ามเขียนโค้ดก่อนสร้างเอกสารครบ
2. ห้ามเสนอ feature ลอยที่ไม่จำเป็น
3. ห้ามทำระบบซับซ้อนเกิน MVP
4. ห้าม hardcode column mapping
5. ห้าม hardcode alias
6. ห้ามให้ dashboard อ่าน raw data ทั้งหมด
7. ทุก action สำคัญต้องมี validation และ audit log
8. ทุก issue ต้องมี explanation และ next action
9. ทุก role ต้องมี permission ชัดเจน
10. ต้องระบุ out of scope ชัดเจน
```

---

# MASTER PROMPT: ให้ AI Agent เขียนโค้ดหลังเอกสารเสร็จ

```text
ตอนนี้มีเอกสารระบบครบแล้วใน /docs

ให้คุณทำหน้าที่เป็น AI Coding Agent และสร้าง Web App ตามเอกสารเท่านั้น

กฎสำคัญ:
1. อ่านเอกสารทุกไฟล์ก่อนเริ่ม
2. ห้ามเพิ่ม feature นอกเหนือจาก docs
3. ถ้าเจอข้อมูลขัดแย้ง ให้ยึดตามลำดับนี้:
   - 01_product_brief.md
   - 02_structure.md
   - 03_module_spec.md
   - 05_workflow.md
   - 10_data_model.md
   - 11_api_backend_spec.md
   - 07_design.md
4. สร้างระบบแบบ clean architecture
5. แยก component, service, data layer, validation, utils
6. ทุก action สำคัญต้องผ่าน backend validation
7. ทุก role permission ต้องตรวจทั้ง frontend และ backend
8. ทุก page ต้อง responsive
9. ทุก form ต้องมี loading, error, empty, success state
10. ห้ามใช้ mock data ใน production logic
11. ทุก function สำคัญต้องมี error handling
12. ทุก transaction สำคัญต้องเขียน audit log
13. หลังสร้างเสร็จให้ทำ checklist เทียบกับ 20_acceptance_criteria.md

ให้เริ่มจาก:
1. สร้าง file structure
2. สร้าง core layout
3. สร้าง design system component
4. สร้าง data layer
5. สร้าง auth/session
6. สร้าง role permission
7. สร้าง import staging
8. สร้าง alias center
9. สร้าง stock replay
10. สร้าง reconciliation engine
11. สร้าง issue center
12. สร้าง control center
13. สร้าง export/notification/retention
14. ทดสอบ responsive
15. สรุปสิ่งที่ทำเสร็จและสิ่งที่ยังไม่เสร็จ
```

---

# FINAL IMPLEMENTATION NOTE

ถ้าต้องเริ่มสร้างตอนนี้ ให้เริ่มจาก:

```text
Build Foundation Now
Profile Data in Parallel
Lock Schema v0
Then Build Reconciliation
```

ห้ามเริ่มจาก simulation/relocation/fifo ขั้นสูง

เริ่มที่:

1. User Approval
2. Import Staging
3. Column Mapping
4. Alias Review
5. Anchor Snapshot
6. Backfill In-Out
7. Movement Ledger
8. Stock Replay
9. Auto Reconciliation Detail
10. Issue Center

นี่คือทางที่เร็วที่สุดแต่ไม่พังภายหลัง

