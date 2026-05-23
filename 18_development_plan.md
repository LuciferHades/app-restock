# 18_development_plan.md

# Development Plan Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Development Roadmap / Build Sequence / Implementation Plan  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / Project Owner Use  
**Primary Use:** ใช้กำหนดลำดับการพัฒนาแบบมืออาชีพ ตั้งแต่ Foundation, Data Profiling, Schema Lock, Import, Alias, Stock Replay, Reconciliation, Issue, Control Center, Notification, Export, Retention ไปจนถึง Future Phase โดยเน้นให้ระบบเร็ว คลีน ใช้งานจริง และไม่บานเกิน MVP

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Development Plan สำหรับสร้าง Warehouse Reconciliation ERP Platform แบบเป็นลำดับขั้น

เป้าหมายคือให้ AI Agent / Developer รู้ว่า:

1. ต้องสร้างอะไรก่อน
2. อะไรเป็น dependency ของอะไร
3. อะไรห้ามทำก่อน
4. แต่ละ phase มี output อะไร
5. แต่ละ phase ต้องผ่าน acceptance อะไร
6. ต้อง test อะไรก่อนขึ้น phase ถัดไป
7. ต้องทำ data profiling ตอนไหน
8. ต้อง lock schema ตอนไหน
9. ต้องกันระบบช้า/bug/บานอย่างไร
10. ต้องส่งมอบอะไรให้ user ตรวจในแต่ละรอบ

หลักสำคัญ:

```text
Build Foundation Now
Profile Data in Parallel
Lock Schema v0
Then Build Reconciliation
```

ห้ามเริ่มจาก:

```text
Simulation / Relocation / Advanced FIFO / Visual Map / Advanced AI
```

ต้องเริ่มจาก:

```text
User + Permission + Import + Alias + Anchor + Ledger + Replay + Reconciliation + Issue
```

---

## 2. Development Philosophy

## 2.1 Build the Data Control Backbone First

ระบบนี้จะมั่นคงได้ต้องสร้าง backbone ก่อน:

```text
Auth
→ Permission
→ Audit
→ Import Staging
→ Column Mapping
→ Alias Review
→ Anchor Snapshot
→ Movement Ledger
→ Stock Replay
→ Reconciliation
→ Issue
→ Control Center
```

ถ้า backbone ยังไม่เสร็จ ห้ามไปสร้าง feature ชั้นบนที่พึ่งพาข้อมูลเหล่านี้

---

## 2.2 Vertical Slice Over Big Bang

อย่าสร้างทุก table ทุก page ทุก service พร้อมกันแบบ big bang

ให้ทำเป็น vertical slice ที่ใช้ได้จริงทีละชั้น:

```text
Upload small file
→ Preview
→ Map column
→ Save raw
→ Normalize one file type
→ Create alias queue
→ Confirm
→ Show batch detail
```

แล้วค่อยขยายไป replay/reconciliation

---

## 2.3 Manual First, Automation Later

ใน MVP ระบบควร:

```text
System suggests
Human confirms
System audits
```

ไม่ควร:

```text
System auto-decides without review
```

ใช้กับ:

- alias mapping
- pending inbound confirm
- accept difference
- close issue
- lock day
- retention execute
- ERP correction export

---

## 2.4 Performance Built In From Start

ห้ามรอให้ระบบช้าแล้วค่อยแก้

ต้องออกแบบตั้งแต่ต้น:

- import แบบ batch/chunk
- dashboard อ่าน summary/result
- table มี pagination/filter
- heavy job มี status
- retention 45 วัน
- indexes ตาม date/warehouse/status
- mobile ไม่โหลด raw data ใหญ่

---

## 2.5 Audit Built In From Start

ทุก phase ที่มี write action ต้องมี audit ตั้งแต่แรก

ห้ามทำระบบก่อนแล้วค่อยใส่ audit ทีหลัง เพราะจะย้อนตาม source ไม่ได้

---

# 3. Overall Roadmap

ระบบแบ่งเป็น 8 phase หลัก:

```text
Phase 0: Preparation & Data Profiling
Phase 1: Foundation Core
Phase 2: Import & Alias Backbone
Phase 3: Snapshot, Ledger & Stock Replay
Phase 4: Reconciliation & Issue Engine
Phase 5: Control Layer
Phase 6: Export, Notification & Retention
Phase 7: Hardening, QA & UAT
Phase 8: Future Advanced Modules
```

---

# 4. Phase Summary

| Phase | Name | Goal | Priority |
|---:|---|---|---|
| 0 | Preparation & Data Profiling | เข้าใจไฟล์จริงและ lock schema v0 | P0 |
| 1 | Foundation Core | user, permission, audit, layout | P0 |
| 2 | Import & Alias Backbone | upload, mapping, raw, alias review | P0 |
| 3 | Snapshot, Ledger & Replay | Inquiry anchor, In-Out ledger, daily stock | P0 |
| 4 | Reconciliation & Issue | auto check, explanation, issue/task | P0 |
| 5 | Control Layer | dashboard, close readiness, coverage | P1 |
| 6 | Export, Notification, Retention | report, alert, cleanup 45 days | P1 |
| 7 | Hardening & UAT | speed, security, edge cases | P0 before production |
| 8 | Future Advanced | simulation/fifo/visual map/AI | Future |

---

# 5. Phase 0 — Preparation & Data Profiling

## 5.1 Goal

ทำความเข้าใจข้อมูลจริงก่อนสร้าง logic ลึก เพื่อไม่ให้ hardcode ผิด

## 5.2 Inputs Required

ต้องขอ sample data จากงานจริง:

```text
1. Inquiry 3–5 ไฟล์ จากคนละวัน
2. System In-Out 3–5 วัน รวมวันที่ย้อนหลัง
3. ตัวอย่างป้ายหน้ากอง 10–20 ตัวอย่าง
4. ตัวอย่างชื่อย่อ customer/item เช่น MKMK, UPUP, UFUF, W150, RE T1
5. ตัวอย่าง location/DC ที่ไม่ตรงกัน
6. ตัวอย่าง REJ
7. ตัวอย่าง pending inbound
8. ตัวอย่าง export ที่ ERP Admin อยากได้
```

## 5.3 Tasks

### Task 0.1 — File Inventory

เก็บรายการไฟล์ตัวอย่าง:

- file name
- file type
- source
- business date
- warehouse/DC
- row count
- notes

### Task 0.2 — Header Profiling

วิเคราะห์:

- columns ทั้งหมด
- required columns
- columns ที่ชื่อเปลี่ยนระหว่างไฟล์
- columns ที่อาจเป็น date/qty/location/item/customer
- columns ที่ว่างเยอะ

### Task 0.3 — Date / Time Profiling

วิเคราะห์:

- business date อยู่ไหน
- posting date อยู่ไหน
- movement date อยู่ไหน
- date format
- พ.ศ./ค.ศ.
- time มีไหม

### Task 0.4 — Customer / Item Profiling

วิเคราะห์:

- customer อยู่ column แยกหรือใน description
- MKMK / UPUP / UFUF ปรากฏอย่างไร
- item เช่น W150 / RE T1 / SR CT1 อยู่ตรงไหน
- year/lot/pack อยู่ตรงไหน
- product group อื่น pattern ต่างจากน้ำตาลอย่างไร

### Task 0.5 — Location / DC Profiling

วิเคราะห์:

- DC กับ warehouse เหมือน/ต่างกันอย่างไร
- location raw มี pattern อะไร
- location หน้างานกับ ERP ใช้ชื่อไม่ตรงกันอย่างไร
- มี owner location vs physical location หรือไม่

### Task 0.6 — Qty / UOM Profiling

วิเคราะห์:

- qty มี comma/decimal/negative ไหม
- UOM มีอะไรบ้าง
- UOM อยู่ column แยกหรือใน text
- qty sign ใช้ movement type หรือ negative sign

### Task 0.7 — Movement Type Profiling

วิเคราะห์:

- IN/OUT/TRANSFER/REJ แสดงด้วย code อะไร
- transfer มี from/to หรือไม่
- REJ มี reason/location ปลายทางไหม

### Task 0.8 — Matching Key Candidate

หา candidate fields สำหรับ reconcile:

- date
- warehouse/DC
- location
- customer
- item
- year
- lot
- qty
- source_ref
- document no
- truck no optional

### Task 0.9 — Alias Seed v0

สร้าง alias seed เบื้องต้น:

- customer aliases
- item aliases
- location aliases
- DC aliases
- UOM aliases

### Task 0.10 — Test Case Set v0

สร้าง test case 10–20 เคส:

- matched
- qty diff
- system missing
- field missing
- location diff
- unknown alias
- pending inbound
- REJ not posted
- duplicate inquiry
- backfill 5 days

## 5.4 Outputs

Phase 0 ต้องได้:

```text
1. Data Profiling Report
2. Canonical Field Contract v0
3. Column Mapping Template v0
4. Alias Seed v0
5. Matching Key Candidate v0
6. Data Quality Risk List
7. Reconciliation Test Case v0
8. Export Format Requirement v0
```

## 5.5 Acceptance Criteria

- [ ] รู้ว่า Inquiry มี column อะไรและ field ไหนสำคัญ
- [ ] รู้ว่า In-Out มี movement type อย่างไร
- [ ] รู้ว่า customer/item/location/DC/UOM raw text ต้อง map อย่างไร
- [ ] มี canonical field contract v0
- [ ] มี alias seed v0
- [ ] มี test case สำหรับ reconciliation อย่างน้อย 10 เคส
- [ ] ระบุ risk ที่ยังไม่แน่ชัดได้

---

# 6. Phase 1 — Foundation Core

## 6.1 Goal

สร้างฐานระบบให้ปลอดภัยและ maintainable ก่อนทำ data engine

## 6.2 Modules

- App Shell
- Design System Component Basic
- Auth / Session
- User Registration
- User Approval
- Role / Permission
- Warehouse Scope
- Audit Log
- Basic Master Data
- Basic Error/Loading/Empty State

## 6.3 Tasks

### Task 1.1 — Project Structure

สร้างโครงสร้าง project เช่น:

```text
/src
  /app or /pages
  /components
  /features
  /services
  /data
  /validation
  /utils
  /types
  /config
  /jobs
  /tests
/docs
```

### Task 1.2 — App Shell

สร้าง layout:

- desktop sidebar
- top bar
- mobile bottom nav for checker
- permission-aware menu
- page header
- route guard

### Task 1.3 — Design System Basic Components

สร้าง component:

- Button
- Input
- Select
- Card
- StatusBadge
- SeverityBadge
- ConfidenceBadge
- DataTable
- FilterBar
- DetailDrawer
- ConfirmModal
- EmptyState
- ErrorState
- LoadingSkeleton
- ProgressPanel

### Task 1.4 — Auth / Session

สร้าง:

- register
- login
- logout
- verify session
- session expiry
- password hash/salt
- blocked pending/locked user

### Task 1.5 — User Approval

สร้าง flow:

```text
Register
→ PENDING_APPROVAL
→ Admin approve/reject
→ Assign role/permission/scope
```

Reject rule:

```text
Reject แล้วลบ request จาก queue แต่ต้องเก็บ audit
```

### Task 1.6 — Permission Service

สร้าง logic:

- effective permission
- role permissions
- permission override
- warehouse scope
- backend permission guard
- frontend menu guard

### Task 1.7 — Audit Log Service

สร้าง audit infrastructure:

- writeAudit
- listAuditLogs
- getEntityAuditTrail

### Task 1.8 — Master Data Basic

สร้าง table/page basic สำหรับ:

- warehouse
- location
- customer
- item
- product group
- UOM

### Task 1.9 — Global Validation/Error Format

สร้าง response/error pattern:

```json
{
  "success": false,
  "code": "VALIDATION_ERROR",
  "message": "",
  "details": {}
}
```

## 6.4 Acceptance Criteria

- [ ] สมัคร user ได้และ status = PENDING_APPROVAL
- [ ] pending user login ไม่ได้
- [ ] Admin approve user พร้อม role/scope ได้
- [ ] Admin reject user แล้ว request หายจาก queue แต่ audit ยังอยู่
- [ ] Password ไม่เก็บ plain text
- [ ] Permission ตรวจ backend ได้
- [ ] Warehouse scope ตรวจได้
- [ ] Menu แสดงตาม permission
- [ ] Audit log บันทึก critical action ได้
- [ ] Layout responsive พื้นฐานได้

## 6.5 Do Not Build in Phase 1

- Reconciliation engine
- Stock replay
- Simulation
- Advanced dashboard
- Export official ERP template

---

# 7. Phase 2 — Import & Alias Backbone

## 7.1 Goal

สร้างระบบรับไฟล์แบบปลอดภัย ไม่ hardcode column และจัดการชื่อไม่ตรงด้วย alias review

## 7.2 Modules

- Import Center
- Import Batch
- Raw Rows
- Column Mapping
- Normalize Pipeline basic
- Alias Detection
- Alias Review Queue
- Alias Management

## 7.3 Tasks

### Task 2.1 — Import Batch Create

สร้าง API/UI:

- select file type
- upload file metadata
- checksum
- business date/date range
- warehouse
- batch status

### Task 2.2 — Client Parse + Preview

แสดง:

- file name
- file size
- row count
- headers
- sample rows
- duplicate checksum warning

### Task 2.3 — Chunk Upload Raw Rows

สร้าง:

- uploadRawChunk
- save import_raw_rows
- row_no
- raw_json
- chunk progress

### Task 2.4 — Column Mapping

สร้าง:

- mapping source column → canonical field
- required field validation
- saved template
- warning if header changed

### Task 2.5 — Normalize Batch Basic

สร้าง normalize สำหรับ:

- Inquiry basic
- System In-Out basic

แต่ใน phase นี้ยังไม่จำเป็นต้องสร้าง stock replay เต็ม

### Task 2.6 — Alias Detection

ถ้าเจอ raw text ที่ map ไม่ได้:

```text
Create alias_review_queue
status = PENDING
```

### Task 2.7 — Alias Review UI

สร้างหน้า:

- list queue
- filter by alias type
- suggested match
- confidence
- map to existing
- create new master
- reject

### Task 2.8 — Alias Tables

แยก alias table:

- customer_aliases
- item_aliases
- location_aliases
- dc_aliases
- uom_aliases

ห้ามรวม alias ทุกชนิดแบบไม่ระบุ type

### Task 2.9 — Import Batch Detail

แสดง:

- batch status
- validation summary
- error rows
- unknown alias count
- mapping status
- confirm availability

## 7.4 Acceptance Criteria

- [ ] Upload file แล้วสร้าง import batch ได้
- [ ] Preview headers/sample rows ได้
- [ ] Save raw rows แบบ chunk ได้
- [ ] Column mapping ไม่ hardcode
- [ ] Missing required mapping block normalize ได้
- [ ] Unknown alias เข้า queue ได้
- [ ] Admin map alias ได้
- [ ] Alias type mismatch ถูก block
- [ ] Import batch detail แสดง error/warning ได้
- [ ] ทุก import/mapping/alias action มี audit

## 7.5 Do Not Build in Phase 2

- Full reconciliation
- Advanced AI matching
- OCR
- Direct ERP import API

---

# 8. Phase 3 — Snapshot, Ledger & Stock Replay

## 8.1 Goal

สร้าง data engine สำหรับคำนวณ stock ข้ามวันจาก Inquiry Anchor + Movement Ledger

## 8.2 Modules

- Inquiry Snapshot
- System In-Out Records
- Movement Ledger
- Backfill Wizard
- Data Coverage Check
- Stock Replay Job
- Daily Stock Position

## 8.3 Tasks

### Task 3.1 — Inquiry Confirm as Anchor Snapshot

เมื่อ confirm Inquiry:

```text
create inquiry_snapshots
create inquiry_snapshot_lines
handle active snapshot policy
```

Rules:

- Inquiry ไม่สร้าง movement ledger
- duplicate snapshot ต้อง warning
- same date ต้อง replace/archive/cancel policy

### Task 3.2 — System In-Out Confirm as Movement

เมื่อ confirm In-Out:

```text
create system_inout_records
create movement_ledger source_type=SYSTEM_INOUT
```

### Task 3.3 — Movement Ledger Service

สร้าง ledger central:

- source_type
- source_ref_id
- movement_type
- stock effect sign
- qty
- customer/item/location/year/lot
- status
- confidence

### Task 3.4 — Field Movement to Ledger Basic

สร้าง basic field movement:

- IN
- OUT
- TRANSFER basic
- REJ basic later/phase 4

### Task 3.5 — Backfill Wizard

สร้าง flow:

```text
select date range
→ upload/choose In-Out batch
→ coverage check
→ run replay
→ run reconciliation later
```

### Task 3.6 — Data Coverage Check

ตรวจ:

- anchor snapshot exists
- In-Out coverage by day
- unknown alias pending
- pending inbound
- REJ pending
- moving location without count

### Task 3.7 — Stock Replay Job

สร้าง job:

```text
find anchor snapshot
load ledger range
calculate opening/in/out/transfer/rej/adjustment
save daily_stock_positions
calculate confidence
```

### Task 3.8 — Daily Stock Position UI

แสดง:

- date
- warehouse
- location
- customer
- item
- year/lot
- opening
- in/out
- ending expected
- confidence
- source mode

## 8.4 Acceptance Criteria

- [ ] Inquiry confirm สร้าง anchor snapshot ได้
- [ ] Inquiry ไม่สร้าง movement ledger
- [ ] Same-date snapshot conflict ต้องเลือก policy
- [ ] In-Out confirm สร้าง system records + ledger ได้
- [ ] Ledger trace กลับ source ได้
- [ ] Backfill date range ได้
- [ ] Coverage check บอก missing anchor/In-Out/alias ได้
- [ ] Stock replay จาก anchor + ledger ได้
- [ ] Daily stock position ถูกสร้างได้
- [ ] Replay job มี status/log/error count

## 8.5 Do Not Build in Phase 3

- Full issue engine
- Advanced close day
- Advanced capacity planning

---

# 9. Phase 4 — Reconciliation & Issue Engine

## 9.1 Goal

ให้ระบบตรวจความตรง/ไม่ตรงอัตโนมัติ พร้อม explanation และสร้าง issue/task

## 9.2 Modules

- Reconciliation Job
- Matching Key Config
- Result Code Engine
- Explanation Engine
- Issue Engine
- Issue Center
- Issue Detail
- Checker Task
- Physical Count
- Pending Inbound Confirm
- REJ Notice Basic

## 9.3 Tasks

### Task 4.1 — Matching Key Config

สร้าง config สำหรับ match:

- Field vs System
- Expected vs Count
- REJ vs System
- Pending Inbound vs In-Out

### Task 4.2 — Reconciliation Job

สร้าง job:

```text
load field movement
load system inout
load daily stock position
load physical counts
load pending inbound
load REJ
match records
generate result
```

### Task 4.3 — Result Code Engine

สร้าง result codes:

- MATCHED
- SYSTEM_MISSING
- FIELD_MISSING
- QTY_DIFF
- LOCATION_DIFF
- ITEM_DIFF
- LOT_DIFF
- UNKNOWN_ALIAS
- PENDING_INBOUND
- REJ_NOT_POSTED
- NEED_RECOUNT

### Task 4.4 — Severity & Confidence Engine

คำนวณ:

- severity
- confidence
- owner_role
- suggested_action

### Task 4.5 — Explanation Engine

ทุก non-matched result ต้องมี:

- explanation
- likely_cause
- suggested_action

### Task 4.6 — Issue Engine

ถ้า result requires action:

```text
create/update issue
assign owner_role
set status
notify optional later
```

### Task 4.7 — Issue Center UI

สร้างหน้า:

- summary cards
- filters
- issue list/table
- severity/status badges
- drilldown detail

### Task 4.8 — Issue Detail UI

แสดง:

- field movement
- system in-out
- count
- expected stock
- diff
- explanation
- suggested action
- audit trail
- action buttons

### Task 4.9 — Physical Count / Recount Basic

สร้าง checker count:

- task-based count
- free count basic if allowed
- active checker session required
- count links to issue/task

### Task 4.10 — Pending Inbound Confirm

สร้าง:

- pending inbound list
- candidate matching from In-Out
- confirm mapping
- convert to canonical movement/ledger

### Task 4.11 — REJ Notice Basic

สร้าง:

- create REJ notice
- status WAIT_SYSTEM_POST
- compare with System In-Out
- create REJ_NOT_POSTED issue

## 9.4 Acceptance Criteria

- [ ] Run reconciliation ได้จาก date range
- [ ] MATCHED result ถูกสร้างได้
- [ ] QTY_DIFF result ถูกสร้างได้
- [ ] SYSTEM_MISSING result ถูกสร้างได้
- [ ] LOCATION_DIFF result ถูกสร้างได้
- [ ] REJ_NOT_POSTED issue ถูกสร้างได้
- [ ] ทุก non-matched มี explanation
- [ ] Issue มี owner_role/status/severity/confidence
- [ ] Issue detail แสดง Field/System/Count/Expected ได้
- [ ] Recount task ถูกสร้างจาก issue ได้
- [ ] Count require active checker session
- [ ] Pending inbound confirm ได้

## 9.5 Do Not Build in Phase 4

- Full ML anomaly
- Advanced FIFO
- Full capacity optimizer

---

# 10. Phase 5 — Control Layer

## 10.1 Goal

สร้าง UI สำหรับตัดสินใจรายวัน ไม่ใช่แค่ list data

## 10.2 Modules

- Control Center
- Close Readiness
- Reconciliation Calendar Basic
- Data Coverage Dashboard
- Count Required Locations
- Maintenance Mini Health

## 10.3 Tasks

### Task 5.1 — Summary Tables / Aggregation

สร้าง summary:

- daily_recon_summary
- daily_issue_summary
- import_summary
- system_health_checks basic

### Task 5.2 — Control Center Cards

Cards:

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

### Task 5.3 — Drilldown Links

ทุก card ต้อง drilldown ไปหน้าที่เกี่ยวข้องพร้อม filter

### Task 5.4 — Close Readiness Gate

ตรวจ:

- open critical issue
- pending alias
- pending inbound
- missing anchor
- missing in-out
- REJ_NOT_POSTED
- failed recon job

### Task 5.5 — Daily Close Basic

สร้าง status:

- OPEN
- PRELIM_CLOSED
- FINAL_CLOSED
- LOCKED

### Task 5.6 — Count Required Locations

สร้าง list location ที่ควรนับ:

- QTY_DIFF
- LOCATION_DIFF
- moving location no count
- REJ pending
- pending inbound

## 10.4 Acceptance Criteria

- [ ] Control Center โหลดเร็วจาก summary/result
- [ ] ไม่ query raw full history
- [ ] Critical/Pending/Warning เด่นก่อน matched
- [ ] ทุก card มี drilldown
- [ ] Close readiness บอก READY/WARNING/NOT_READY ได้
- [ ] Lock day block direct edit ได้
- [ ] Count required locations สร้าง task ได้

---

# 11. Phase 6 — Export, Notification & Retention

## 11.1 Goal

ทำให้ระบบส่งข้อมูลออก แจ้งเตือน และคุมข้อมูลเกิน 45 วันได้

## 11.2 Modules

- Export Center
- Export Log
- Notification Center
- Notification Rules/Templates/Channels
- Retention Job
- Health Check
- Audit Report

## 11.3 Tasks

### Task 6.1 — Export Center Basic

Export types:

- Issue List
- Daily Reconciliation
- REJ Report
- Pending Inbound
- Alias Review
- ERP Correction Template basic
- Retention Report
- Audit Report basic

### Task 6.2 — Export Preview + Log

ก่อน export ต้องแสดง:

- filters
- selected rows
- row count
- columns
- file format

ทุก export ต้องมี export log

### Task 6.3 — Notification Center Basic

สร้าง:

- notification_channels
- notification_rules
- notification_templates
- notification_logs

Channels:

- In-app
- Telegram
- LINE Messaging API
- Email

### Task 6.4 — Notification Dispatch

ส่ง event:

- USER_PENDING_APPROVAL
- UNKNOWN_ALIAS_CREATED
- PENDING_INBOUND_CREATED
- SYSTEM_MISSING
- QTY_DIFF
- REJ_NOT_POSTED
- BACKFILL_COMPLETED
- DAILY_CLOSE_NOT_READY
- EXPORT_READY
- RETENTION_FAILED
- DB_USAGE_WARNING

### Task 6.5 — Anti-Spam

ต้องมี:

- severity filter
- cooldown_minutes
- send_once_per_issue
- daily digest optional

### Task 6.6 — Retention 45 Days

สร้าง job:

```text
cutoff = today - 45 days
create summary/archive
skip open issue
delete/archive safe raw/staging/closed records
write retention log
notify admin
```

### Task 6.7 — Health Check

ตรวจ:

- DB usage
- row count
- last import
- last replay
- last reconciliation
- last retention
- failed jobs
- pending alias
- pending inbound
- open critical issues

## 11.4 Acceptance Criteria

- [ ] Export report ได้อย่างน้อย CSV/XLSX
- [ ] Export มี preview และ log
- [ ] ERP template export ต้องมี permission/audit
- [ ] Notification rule/channel/template สร้างได้
- [ ] Telegram/LINE/Email/In-app มี abstraction service
- [ ] Notification มี cooldown
- [ ] Retention dry run ได้
- [ ] Retention execute ไม่ลบ open issue
- [ ] Retention log บอก deleted/skipped ได้
- [ ] Health check แสดง status ได้

---

# 12. Phase 7 — Hardening, QA & UAT

## 12.1 Goal

ทำให้ระบบเสถียร เร็ว ปลอดภัย และผ่าน test case งานจริงก่อน production

## 12.2 Tasks

### Task 7.1 — Performance Audit

ตรวจ:

- dashboard query speed
- import large file
- replay date range
- reconciliation date range
- issue center filter
- export large report
- mobile page load

### Task 7.2 — Security Audit

ตรวจ:

- permission backend
- warehouse scope
- session expiry
- password hash
- no secret exposure
- audit critical action
- locked day protection

### Task 7.3 — Data Quality QA

ใช้ sample data จริงทดสอบ:

- duplicate Inquiry
- In-Out missing day
- unknown alias
- qty format issue
- date format issue
- location mismatch
- pending inbound
- REJ not posted

### Task 7.4 — Reconciliation Accuracy QA

ทดสอบ 10–20 cases:

- matched
- QTY_DIFF
- SYSTEM_MISSING
- FIELD_MISSING
- LOCATION_DIFF
- LOT_DIFF
- REJ_NOT_POSTED
- Pending Inbound Confirm

### Task 7.5 — UX UAT

ให้ user ทดสอบ:

- Checker mobile check-in/count/smart inbound
- Admin import/backfill/alias
- Supervisor issue review/close
- ERP Admin export/post queue

### Task 7.6 — Retention Safety Test

ทดสอบ:

- open issue source not deleted
- closed raw older 45 days deleted/archived
- retention logs complete
- notification on failure

### Task 7.7 — Regression Checklist

ทำ checklist ก่อน release:

- auth
- permission
- import
- alias
- replay
- recon
- issue
- export
- notification
- retention
- dashboard
- mobile responsive

## 12.3 Acceptance Criteria

- [ ] Critical bugs = 0
- [ ] Permission bypass = 0
- [ ] Reconciliation test cases pass
- [ ] Dashboard does not query raw full history
- [ ] Import large file does not timeout under expected size
- [ ] Retention does not delete open issue data
- [ ] Mobile Checker flow usable
- [ ] UAT feedback addressed or logged as backlog

---

# 13. Phase 8 — Future Advanced Modules

## 13.1 Goal

ทำหลัง MVP stable แล้วเท่านั้น

## 13.2 Future Modules

- Capacity Simulation
- Available Location Engine
- Relocation Suggestion
- Move small to large strategy
- Advanced FIFO / Lot aging
- Visual Warehouse Map
- OCR Label Reader
- Real-time ERP Integration
- Native Mobile App
- Advanced Forecast Dashboard
- Custody / Contract / Billing

## 13.3 Entry Criteria

ห้ามเริ่ม future module จนกว่า:

- core stock replay stable
- reconciliation accuracy accepted
- alias seed stable
- location master stable
- data coverage reliable
- user ใช้ issue center จริงแล้ว
- sample data เพียงพอ
- business rule ได้รับการยืนยัน

---

# 14. Critical Build Dependencies

## 14.1 Dependency Chain

```text
User/Permission/Audit
→ Import Batch/Raw
→ Column Mapping
→ Alias Review
→ Master Data
→ Inquiry Snapshot
→ System In-Out Records
→ Movement Ledger
→ Stock Replay
→ Daily Stock Position
→ Reconciliation Result
→ Issue
→ Control Center
→ Export/Notification/Retention
```

## 14.2 Do Not Break Dependency

ห้ามทำ:

```text
Reconciliation ก่อนมี alias/ledger/replay
Dashboard ก่อนมี summary/result
Retention ก่อนมี issue skip rule
Export official ก่อนมี log/audit
Close day ก่อนมี readiness gate
```

---

# 15. Recommended Sprint Plan

## Sprint 0 — Data & Setup

Duration: 3–7 days depending file availability

Deliverables:

- Data Profiling Report
- Canonical Field Contract v0
- Alias Seed v0
- Test Cases v0
- Project skeleton

## Sprint 1 — Foundation

Deliverables:

- Auth/Register/Login
- User Approval
- Permission
- Audit log
- App shell/design system basic

## Sprint 2 — Import & Alias

Deliverables:

- Import Center
- Preview
- Column Mapping
- Raw Rows
- Normalize basic
- Alias Review

## Sprint 3 — Data Engine

Deliverables:

- Inquiry Snapshot
- System In-Out Records
- Movement Ledger
- Backfill Wizard basic
- Stock Replay
- Daily Stock Position

## Sprint 4 — Reconciliation

Deliverables:

- Reconciliation job
- Result codes
- Explanation engine
- Issue Center
- Issue Detail
- Count task basic

## Sprint 5 — Control

Deliverables:

- Control Center
- Close Readiness
- Data Coverage Dashboard
- Count Required Locations

## Sprint 6 — Governance

Deliverables:

- Export Center
- Notification Center
- Retention job
- Health check
- Audit report

## Sprint 7 — QA/UAT

Deliverables:

- Test pass
- Bug fix
- Performance tuning
- Production checklist

---

# 16. AI Agent Build Rules

AI Agent ต้องทำตามลำดับนี้:

1. อ่าน docs ทั้งหมดก่อนเขียนโค้ด
2. อ่าน `17_out_of_scope.md` ก่อนเสนอ feature
3. เริ่มจาก Phase 0/1 ไม่ใช่ advanced feature
4. ถ้าข้อมูลไม่พอ ให้สร้าง TODO/Question ไม่เดาเอง
5. ห้าม hardcode column mapping
6. ห้าม hardcode alias
7. ห้าม hardcode permission เฉพาะ UI
8. ทุก write action ต้องมี validation + audit
9. ทุก heavy job ต้องมี job status
10. Dashboard ต้องอ่าน summary/result
11. Retention ต้อง skip open issue
12. ทุก issue ต้องมี explanation
13. ทุก page ต้องมี loading/error/empty state
14. ทุก feature ต้องมี acceptance criteria

---

# 17. Technical Implementation Notes

## 17.1 If Using Supabase/Postgres

Recommended:

- tables ตาม data model
- RLS for user/warehouse scope if feasible
- service role only server-side
- cron/edge function for retention
- storage bucket private for exports/photos
- indexes on date/warehouse/status

## 17.2 If Using Google Sheets / Apps Script MVP

Recommended:

- sheet as database only for small MVP
- separate sheets by entity
- avoid heavy dashboard formula over raw rows
- use summary sheets
- retention script
- Apps Script web app service functions
- migrate path to Supabase later

## 17.3 Data Size Control

ตั้งแต่ MVP ต้องมี:

- active data 45 days
- raw cleanup
- summary/archive
- pagination
- chunk upload
- index/lookup maps in memory for jobs

---

# 18. Risk-Based Priority

## P0 — Must Build Correctly

- User approval / permission
- Import staging
- Column mapping
- Alias review
- Inquiry anchor
- In-Out ledger
- Stock replay
- Reconciliation result/explanation
- Issue center
- Audit log
- Retention skip open issue

## P1 — Should Build After P0 Stable

- Control center
- Notification center
- Export center
- Daily close readiness
- Health check

## P2 — Future

- Simulation
- Visual map
- Advanced FIFO
- Forecast
- OCR
- Real-time ERP API

---

# 19. Release Gates

## Gate 1 — Foundation Ready

Can pass if:

- user approval works
- permission works backend
- audit works
- app shell works

## Gate 2 — Import Ready

Can pass if:

- file preview works
- column mapping works
- raw rows saved
- alias queue works

## Gate 3 — Data Engine Ready

Can pass if:

- inquiry snapshot created
- in-out ledger created
- replay daily stock works
- coverage check works

## Gate 4 — Reconciliation Ready

Can pass if:

- test cases create correct result codes
- explanations generated
- issues created

## Gate 5 — Operational Ready

Can pass if:

- control center usable
- export works
- notification works
- retention safe
- UAT critical flows pass

---

# 20. Final Development Plan Statement

```text
Development Plan ของ Warehouse Reconciliation ERP Platform ต้องเริ่มจาก Data Profiling และ Foundation Core ก่อน แล้วค่อยสร้าง Import/Alias, Anchor Snapshot, Movement Ledger, Stock Replay, Reconciliation, Issue, Control Center, Export, Notification และ Retention ตามลำดับ ห้ามข้ามไปทำ simulation, relocation, FIFO, visual map หรือ AI ขั้นสูงก่อน เพราะระบบจะมั่นคงได้ก็ต่อเมื่อข้อมูลจริงถูก normalize, trace, replay และ reconcile ได้อย่างถูกต้องพร้อม audit และ permission control
```

