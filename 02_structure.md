# 02_structure.md

# System Structure Blueprint

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** System Structure / Architecture Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer Use  
**Primary Use:** ใช้เป็นแผนผังโครงสร้างหลักของระบบ เพื่อให้ AI Agent / Developer เข้าใจว่าระบบต้องแบ่งเป็นชั้นอะไร มี module อะไร แต่ละ module เชื่อมกันอย่างไร และอะไรยังไม่ต้องทำใน MVP

---

## 1. Purpose of This Document

เอกสารนี้อธิบายโครงสร้างหลักของ Warehouse Reconciliation ERP Platform ในระดับ system architecture และ module architecture

เป้าหมายคือให้ AI Agent / Developer เห็นภาพรวมก่อนลงมือออกแบบ database, API, UI หรือ code

เอกสารนี้ตอบคำถามหลัก:

1. ระบบนี้แบ่งเป็นกี่ layer
2. แต่ละ layer รับผิดชอบอะไร
3. module หลักมีอะไรบ้าง
4. module ไหนอยู่ใน MVP
5. module ไหนเป็น phase หลัง
6. data flow หลักของระบบไหลอย่างไร
7. module เชื่อมต่อกันอย่างไร
8. จุดไหนต้อง flexible เพราะข้อมูลจริงยังอาจเปลี่ยน
9. จุดไหนห้าม hardcode
10. จุดไหนต้องมี audit/security/performance control

หลักสำคัญ:

```text
ระบบนี้ต้องถูกออกแบบเป็น platform ที่แยก core, data, operation, intelligence, UI และ maintenance ออกจากกัน
ไม่ใช่ระบบก้อนเดียวที่ทุก logic ปนกัน
```

---

## 2. System Identity

ระบบนี้คือ:

```text
Warehouse Reconciliation ERP Platform
```

ไม่ใช่:

```text
Stock Count App
Excel Online
Dashboard ดูยอด
ฟอร์มแจ้งเข้าออก
```

ระบบนี้มีหน้าที่เชื่อม 5 ความจริง:

```text
1. Physical Count       = ของจริงที่นับได้
2. Field Movement       = สิ่งที่หน้างานแจ้งว่าเกิดขึ้นจริง
3. System In-Out        = movement ที่ ERP post จริง
4. Inquiry              = snapshot ยอดคงเหลือจาก ERP
5. Expected / Replay    = ยอดที่ระบบคำนวณจาก Anchor + Movement Ledger
```

ระบบต้องให้ผลลัพธ์เป็น:

```text
Reconciliation Decision
```

เช่น:

- MATCHED
- SYSTEM_MISSING
- FIELD_MISSING
- QTY_DIFF
- LOCATION_DIFF
- REJ_NOT_POSTED
- UNKNOWN_ALIAS
- PENDING_INBOUND
- NEED_RECOUNT
- NEED_INVESTIGATION

---

## 3. High-Level System Layers

ระบบแบ่งเป็น 6 layer หลัก

```text
1. Control Core Layer
2. Operational Engine Layer
3. Data Engine Layer
4. Intelligence Engine Layer
5. Decision UI Layer
6. Maintenance & Governance Layer
```

ภาพรวม:

```text
User / File / ERP Data
        ↓
Control Core Layer
        ↓
Operational Engine Layer
        ↓
Data Engine Layer
        ↓
Intelligence Engine Layer
        ↓
Decision UI Layer
        ↓
Maintenance & Governance Layer
```

---

# 4. Layer 1 — Control Core Layer

## 4.1 Purpose

Control Core คือฐานควบคุมระบบทั้งหมด

หน้าที่หลัก:

- คุมผู้ใช้งาน
- คุมสิทธิ์
- คุม master data
- คุม alias
- คุม import
- คุม snapshot
- คุม lock/close
- คุม audit

ถ้า layer นี้ไม่ดี ระบบส่วนอื่นจะพัง เพราะข้อมูลจะไม่ถูกต้องตั้งแต่ต้น

---

## 4.2 Modules in Control Core

```text
1. Auth Module
2. User Approval Module
3. Permission Module
4. Master Data Module
5. Alias Management Module
6. Import Staging Module
7. Column Mapping Module
8. Snapshot Control Module
9. Lock / Close Control Module
10. Audit Log Module
```

---

## 4.3 Auth Module

### Purpose

จัดการ login, session, password, token และสถานะผู้ใช้งาน

### Responsibilities

- register user
- login
- logout
- verify session
- reset password
- lock account
- session expiry

### Important Rules

```text
สมัครแล้วใช้งานไม่ได้ทันที
ต้องรอ Admin อนุมัติ
password ต้อง hash + salt
session/token ต้องหมดอายุ
```

### Output

- active session
- user profile
- permission context

---

## 4.4 User Approval Module

### Purpose

ให้ Admin อนุมัติ user ที่สมัครใหม่ และกำหนดสิทธิ์ก่อนใช้งานจริง

### Workflow

```text
User สมัคร / ตั้งรหัส
→ USER_ACCESS_REQUESTS status = PENDING_APPROVAL
→ Admin review
→ Approve หรือ Reject
```

### Approve

```text
เลือก role
เลือก permission
เลือก warehouse scope
สร้าง USER_ACCOUNTS
status = ACTIVE
write audit log
```

### Reject

```text
Reject with reason
ลบ request จาก queue
เก็บ audit log
```

### Connected Modules

- Auth Module
- Permission Module
- Audit Log Module
- Notification Center

---

## 4.5 Permission Module

### Purpose

คุมสิทธิ์แบบละเอียด ไม่ใช่แค่ role เดียว

### Permission Concept

```text
Role = template
Permission = สิทธิ์จริง
Warehouse Scope = ใช้ได้ที่คลังไหน
Checker Session = สิทธิ์นับเฉพาะวันที่ check-in
```

### Core Permission Examples

- CAN_LOGIN
- CAN_IMPORT_INQUIRY
- CAN_IMPORT_INOUT
- CAN_MANAGE_ALIAS
- CAN_CREATE_FIELD_MOVEMENT
- CAN_COUNT
- CAN_CREATE_REJ
- CAN_RUN_RECONCILIATION
- CAN_REVIEW_ISSUE
- CAN_EXPORT
- CAN_CLOSE_DAY
- CAN_MANAGE_NOTIFICATION
- CAN_MANAGE_USER

### Important Rules

```text
Permission ต้องตรวจทั้ง frontend และ backend
UI hide menu ไม่พอ
ทุก API สำคัญต้องเช็ค permission อีกครั้ง
```

---

## 4.6 Master Data Module

### Purpose

เก็บข้อมูลกลางที่ถูกต้อง เช่น customer, item, warehouse, location, UOM

### Core Master Data

- MASTER_CUSTOMER
- MASTER_ITEM
- MASTER_WAREHOUSE
- MASTER_LOCATION
- MASTER_UOM
- MASTER_REASON
- MASTER_PRODUCT_GROUP

### Important Rules

```text
Master ที่ถูกใช้แล้วห้ามลบจริง
ให้ใช้ active/inactive
ถ้าค่ามาจากหน้างานและยังไม่ยืนยัน ให้เป็น temporary หรือ waiting review
```

---

## 4.7 Alias Management Module

### Purpose

แก้ปัญหาชื่อในหน้างาน, ERP, ป้ายหน้ากอง, Excel ไม่ตรงกัน

### Alias Types

```text
CUSTOMER_ALIAS
ITEM_ALIAS
LOCATION_ALIAS
DC_ALIAS
WAREHOUSE_ALIAS
UOM_ALIAS
LOT_PATTERN
```

### Important Example

```text
MKMK / UPUP / UFUF = Customer Alias
W150 / RE T1 / SR CT1 / ไฮโพลดิบ = Item Alias
W8 / DC08 = Warehouse/DC Alias
A01 / W8-A01 / Location 1 = Location Alias
```

### Workflow

```text
Unknown text detected
→ Create Alias Review Queue
→ Suggest match with confidence
→ Admin maps / creates new / rejects
→ System remembers next time
```

### Connected Modules

- Import Staging
- Smart Input
- Reconciliation Engine
- Pending Inbound
- Data Quality Check

### Important Rules

```text
ห้าม hardcode alias
ห้ามรวม alias ทุกชนิดโดยไม่แยก type
ห้าม auto-map low confidence โดยไม่ review
```

---

## 4.8 Import Staging Module

### Purpose

รับไฟล์ Inquiry / In-Out / Count / Master โดยไม่เอาเข้า canonical table ทันที

### Flow

```text
Upload
→ Client-side parse
→ Preview
→ Column Mapping
→ Chunk Upload
→ Raw Staging
→ Normalize
→ Alias Matching
→ Confirm Import
```

### Core Data

- IMPORT_BATCH
- IMPORT_RAW_ROWS
- IMPORT_COLUMN_MAPPING
- IMPORT_ERROR_LOG

### Important Rules

```text
Raw data ต้องเก็บ batch_id
ห้ามตีความข้อมูลทันทีโดยไม่มี mapping
ห้ามลบ raw ก่อน normalize/confirm
```

---

## 4.9 Column Mapping Module

### Purpose

ให้ระบบรองรับไฟล์ที่ column เปลี่ยนหรือชื่อ header ไม่เหมือนกัน

### Example

```text
ERP Column: Posting Date → canonical: posting_date
ERP Column: DC → canonical: warehouse_raw
ERP Column: Material Description → canonical: item_raw_text
ERP Column: Qty → canonical: qty
```

### Important Rules

```text
ห้าม hardcode column index
ห้าม assume ว่า Inquiry/In-Out ทุกไฟล์เหมือนกันเสมอ
```

---

## 4.10 Snapshot Control Module

### Purpose

ควบคุม Inquiry ที่เป็น snapshot ไม่ใช่ transaction

### Main Concept

```text
Inquiry = Anchor Snapshot
```

### Responsibilities

- create snapshot_id
- store checksum
- prevent duplicate append
- support replace/archive/cancel
- mark active snapshot

### Connected Modules

- Stock Replay
- Data Coverage Check
- Reconciliation Calendar

---

## 4.11 Lock / Close Control Module

### Purpose

ควบคุมสถานะของวันทำงานและการแก้ข้อมูลหลังปิดวัน

### Daily Status

```text
OPEN
PRELIM_CLOSED
RECONSTRUCTED
FINAL_CLOSED
LOCKED
```

### Rules

```text
LOCKED แล้วห้ามแก้ movement/count/import ตรง
ถ้าจะแก้ต้องผ่าน Investigation / Correction
```

---

## 4.12 Audit Log Module

### Purpose

เก็บหลักฐานว่าใครทำอะไร เมื่อไร ก่อน/หลังเป็นอะไร

### Must Log

- user approve/reject
- permission change
- import upload/confirm
- alias map
- run replay
- run reconciliation
- issue close
- export
- lock day
- retention delete
- notification rule change

---

# 5. Layer 2 — Operational Engine Layer

## 5.1 Purpose

Operational Engine คือ layer ที่จัดการกิจกรรมจริงของคลัง เช่น แจ้งเข้า แจ้งออก นับจริง REJ transfer และ task

---

## 5.2 Modules

```text
1. Field Movement Module
2. Physical Count Module
3. Pending Inbound Module
4. REJ Module
5. Transfer Module
6. Checker Task Module
7. Checker GPS Session Module
```

---

## 5.3 Field Movement Module

### Purpose

เก็บสิ่งที่หน้างานแจ้งว่าเกิดขึ้นจริง เช่น IN, OUT, MOVE, REJ

### Important Fields

- movement_id
- business_date
- movement_type
- warehouse_id
- location_id
- customer_id
- item_id
- year
- lot
- qty
- uom
- source_text
- status

### Status

```text
DRAFT
REPORTED
WAIT_SYSTEM_POST
MATCHED
PROBLEM
INVESTIGATING
CLOSED
CANCELLED
```

### Connected Modules

- Movement Ledger
- Reconciliation Engine
- Issue Center
- Notification Center

---

## 5.4 Physical Count Module

### Purpose

เก็บของจริงที่นับได้

### Count Modes

```text
BULK_COUNT
LOT_VERIFY
PARTIAL_VERIFY
RECOUNT
BEFORE_COUNT
AFTER_COUNT
```

### Important Rules

```text
Physical Count เป็นหลักฐานของจริง
แต่ต้องมี scope ชัดเจนว่านับ location/item/lot ไหน
```

### Connected Modules

- Reconciliation Engine
- Stock Replay
- Count Required Locations
- Issue Detail

---

## 5.5 Pending Inbound Module

### Purpose

รองรับกรณีของเข้าแต่ยังไม่รู้ข้อมูลครบ

### Use Case

- ของเข้ามาแล้ว
- ยังไม่รู้ customer/item/lot/year ชัด
- ป้ายหน้ากองมีแค่ชื่อย่อ
- ต้องรอ System In-Out วันถัดไปยืนยัน

### Minimal Input

- business_date
- warehouse/location
- qty
- uom
- text_on_label
- photo optional
- reported_by

### Status

```text
PENDING_INBOUND_VERIFY
WAIT_SYSTEM_INOUT
CANDIDATE_FOUND
CONFIRMED
REJECTED
NEED_INVESTIGATION
```

### Connected Modules

- Alias Center
- Import In-Out
- Reconciliation Engine
- Issue Center

---

## 5.6 REJ Module

### Purpose

จัดการของเสีย/เสียหายที่ต้องออกจาก stock ปกติและเข้า REJ location

### Flow

```text
Checker แจ้ง REJ
→ from normal location
→ to REJ location
→ wait ERP post
→ compare with System In-Out
```

### Result Codes

- REJ_MATCHED
- REJ_NOT_POSTED
- REJ_QTY_DIFF
- REJ_LOCATION_DIFF
- REJ_LOT_DIFF

### Connected Modules

- Movement Ledger
- Reconciliation Engine
- ERP Admin Export
- Notification Center

---

## 5.7 Transfer Module

### Purpose

รองรับการย้าย location ภายใน และใน phase หลังรองรับ split/owner vs physical location

### MVP Scope

- basic transfer notice
- from_location
- to_location
- qty
- reason
- status

### Future Scope

- owner_location
- physical_location
- custody stock
- relocation suggestion

---

## 5.8 Checker Task Module

### Purpose

สร้างงานให้ Checker นับ ตรวจ REJ ตรวจ location หรือ verify pending inbound

### Task Types

```text
COUNT
RECOUNT
VERIFY_INBOUND
VERIFY_REJ
VERIFY_LOCATION_DIFF
LOT_VERIFY
```

### Status

```text
OPEN
ASSIGNED
IN_PROGRESS
DONE
HOLD
CANCELLED
```

### Connected Modules

- Issue Center
- Count Required Locations
- Checker GPS Session
- Notification Center

---

## 5.9 Checker GPS Session Module

### Purpose

รองรับ checker outsource ที่มาไม่ซ้ำและต้อง check-in ต่อ warehouse ต่อวัน

### Rule

```text
1 user มี active warehouse ได้ 1 ที่ต่อวัน
ถ้า check-in warehouse ใหม่ session เดิมถูก suspend
```

### Status

```text
ACTIVE
SUSPENDED_BY_OTHER_WH_LOGIN
EXPIRED
CHECKED_OUT
MANUAL_SUSPENDED
```

### Connected Modules

- Permission Module
- Physical Count
- Checker Task
- Audit Log

---

# 6. Layer 3 — Data Engine Layer

## 6.1 Purpose

Data Engine คือ layer ที่สร้างความจริงเชิงคำนวณของระบบ

หน้าที่หลัก:

- รวม movement ทุก source
- สร้าง stock position รายวัน
- replay ข้ามวัน
- ตรวจ coverage
- เตรียมข้อมูลให้ reconciliation

---

## 6.2 Modules

```text
1. Movement Ledger Module
2. Anchor Snapshot Module
3. Stock Replay Module
4. Daily Stock Position Module
5. Data Coverage Check Module
6. Reconciliation Calendar Module
```

---

## 6.3 Movement Ledger Module

### Purpose

รวม movement ทุกแหล่งไว้ใน ledger กลาง

### Source Types

```text
SYSTEM_INOUT
FIELD_NOTICE
REJ_NOTICE
TRANSFER
ADJUSTMENT
COUNT_CORRECTION
PENDING_INBOUND_CONFIRM
```

### Why Important

ถ้าไม่มี ledger กลาง ระบบจะ replay ข้ามวันไม่ได้ และ reconciliation จะกระจัดกระจาย

---

## 6.4 Anchor Snapshot Module

### Purpose

เก็บ Inquiry เป็นจุดอ้างอิง stock

### Key Idea

```text
Inquiry ไม่ต้องมีทุกวัน
แต่ทุก Inquiry ที่มีต้องเป็น Anchor Snapshot
```

### Connected Modules

- Stock Replay
- Data Coverage Check
- Anchor Variance

---

## 6.5 Stock Replay Module

### Purpose

คำนวณยอด stock position จาก Anchor + Movement Ledger

### Formula

```text
Stock Position Date D = Anchor Stock Date A + Movement from A+1 to D
```

หรือย้อนกลับจาก anchor ถัดไป:

```text
Stock Position Date D = Anchor Stock Date B - Movement from D+1 to B
```

### Output

- DAILY_STOCK_POSITION
- source_mode
- confidence

### Source Modes

```text
FROM_INQUIRY
RECONSTRUCTED_FROM_LEDGER
ESTIMATED
COUNT_CONFIRMED
```

---

## 6.6 Data Coverage Check Module

### Purpose

บอกว่าข้อมูลช่วงวันที่นั้นครบพอไหม

### Coverage Status

- COVERAGE_OK
- MISSING_SYSTEM_INOUT
- MISSING_ANCHOR
- PENDING_UNKNOWN_INBOUND
- NO_COUNT_FOR_MOVING_LOCATION
- REJ_PENDING

### Connected Modules

- Backfill Wizard
- Control Center
- Daily Close
- Notification Center

---

## 6.7 Reconciliation Calendar Module

### Purpose

แสดงสถานะข้อมูลแต่ละวัน

### Calendar Status

```text
FINAL_CLOSED
RECONSTRUCTED
PRELIM_CLOSED
WAIT_SYSTEM_INOUT
CRITICAL_ISSUE
NO_ACTIVITY
```

---

# 7. Layer 4 — Intelligence Engine Layer

## 7.1 Purpose

Intelligence Engine คือสมองของระบบ ใช้ตรวจ logic, match, สร้าง issue, อธิบายปัญหา และเสนอ action

---

## 7.2 Modules

```text
1. Rule Engine
2. Reconciliation Engine
3. Explanation Engine
4. Issue Engine
5. Suggested Action Engine
6. Count Planning Engine
7. Simulation Engine later
```

---

## 7.3 Rule Engine

### Purpose

ตรวจ rule พื้นฐานก่อนบันทึก/ก่อนปิดวัน

### Examples

- Inquiry duplicate
- unknown alias
- locked day edit
- checker no active session
- issue open before close
- retention skip open issue

---

## 7.4 Reconciliation Engine

### Purpose

เทียบข้อมูลหลายชุดเพื่อหาผลลัพธ์

### Compare Pairs

```text
Field Movement vs System In-Out
Expected Stock vs Physical Count
REJ Notice vs ERP Movement
Pending Inbound vs In-Out
Anchor vs Replayed Stock
```

### Output

- result_code
- severity
- confidence
- explanation
- likely_cause
- suggested_action

---

## 7.5 Explanation Engine

### Purpose

แปลง result code เป็นคำอธิบายที่คนเข้าใจ

### Example

```text
Field แจ้ง OUT 1,000
ERP ตัด OUT 950
Before/After Count สนับสนุนว่าออกจริง 1,000
ระบบสรุปว่า ERP อาจ post ขาด 50
```

---

## 7.6 Issue Engine

### Purpose

สร้าง issue/task จาก reconciliation result

### Issue Types

- SYSTEM_MISSING
- FIELD_MISSING
- QTY_DIFF
- LOCATION_DIFF
- REJ_NOT_POSTED
- UNKNOWN_ALIAS
- PENDING_INBOUND
- NEED_RECOUNT

---

## 7.7 Suggested Action Engine

### Purpose

เสนอ next action จาก result code

| Result | Suggested Action |
|---|---|
| SYSTEM_MISSING | Send to ERP Admin |
| FIELD_MISSING | Ask Supervisor to verify field movement |
| QTY_DIFF | Check Before/After Count or Recount |
| LOCATION_DIFF | Assign location recount |
| REJ_NOT_POSTED | Notify ERP Admin |
| UNKNOWN_ALIAS | Send to Alias Review |
| PENDING_INBOUND | Wait/Match with In-Out |

---

## 7.8 Count Planning Engine

### Purpose

บอกว่า location ไหนควรนับ เมื่อมี movement ย้อนหลังหรือ issue

### Rule

```text
ไม่ต้องนับทั้งคลัง
นับเฉพาะ location ที่มี movement, REJ, transfer, diff, pending inbound, low confidence
```

---

## 7.9 Simulation Engine Later

### Purpose in Future

ใช้หา available location และ relocation suggestion

ยังไม่ทำเต็มใน MVP

---

# 8. Layer 5 — Decision UI Layer

## 8.1 Purpose

Decision UI คือหน้าจอที่ทำให้ user ตัดสินใจได้ง่าย

ต้องไม่ใช่แค่แสดงข้อมูล แต่ต้องบอกว่า:

- วันนี้มีอะไรต้องทำ
- อะไรผิด
- ใครต้องทำอะไรต่อ
- ปิดวันได้ไหม
- export อะไรค้าง
- cleanup ทำงานไหม

---

## 8.2 Main Pages

```text
1. Control Center
2. Import Center
3. Backfill Wizard
4. Alias Review
5. Smart Inbound Input
6. Issue Center
7. Issue Detail
8. Count Required Locations
9. Export Center
10. Notification Center
11. User Approval
12. User & Permission
13. Lock / Close Control
14. Maintenance / Retention
```

---

## 8.3 Control Center

### Purpose

หน้าแรกสำหรับ Admin/Supervisor ใช้ดูสถานะระบบรวม

### Cards

- Matched Count
- Issue Count
- Critical Issue
- Unknown Alias
- Pending Inbound
- REJ Not Posted
- Export Pending
- Close Readiness
- Retention Status
- DB Usage / Health

---

## 8.4 Import Center

### Purpose

upload file และ preview/mapping ก่อน confirm

### Must Have

- file type
- preview headers
- preview sample rows
- column mapping
- unknown alias summary
- confirm import
- import log

---

## 8.5 Backfill Wizard

### Purpose

รองรับกรณี admin ไม่ upload ทุกวัน

### Flow

```text
เลือก date range
→ upload In-Out
→ coverage check
→ run stock replay
→ run reconciliation
→ generate count tasks
→ show summary
```

---

## 8.6 Alias Review

### Purpose

จัดการ unknown alias

### Actions

- map to existing master
- create new master
- reject
- bulk map

---

## 8.7 Issue Center

### Purpose

ให้ user ดูเฉพาะ exception

### Filters

- date
- warehouse
- result_code
- severity
- confidence
- owner_role
- status

---

## 8.8 Issue Detail

### Purpose

อธิบายความไม่ตรงอย่างละเอียด

### Sections

- Header
- Item/Customer/Location
- Field Data
- System Data
- Count Data
- Expected Data
- Explanation
- Suggested Action
- Audit Trail

---

## 8.9 Export Center

### Purpose

ส่งออกข้อมูลหลายรูปแบบ

### Export Types

- Issue List
- Daily Reconciliation
- REJ Report
- ERP Correction Template
- Export Log
- Retention Report

---

## 8.10 Notification Center

### Purpose

ให้ Admin ตั้งค่าแจ้งเตือนเอง

### Tabs

- Channels
- Rules
- Templates
- Test Send
- Logs

---

# 9. Layer 6 — Maintenance & Governance Layer

## 9.1 Purpose

ดูแลระบบระยะยาวให้เร็ว มั่นคง และไม่ต้องให้คนคอยเช็คตลอด

---

## 9.2 Modules

```text
1. Retention Module
2. Health Check Module
3. Backup Export Module
4. Notification Log Module
5. Export Log Module
6. Audit Review Module
```

---

## 9.3 Retention Module

### Purpose

ลบ/archive ข้อมูลเกิน 45 วัน

### Rule

```text
Archive summary before delete raw
Skip open issues
Write retention log
Notify admin
```

---

## 9.4 Health Check Module

### Checks

- DB usage
- row count by table
- last import
- last reconciliation
- last cleanup
- open critical issues
- pending alias
- pending inbound

---

## 9.5 Backup Export Module

### Purpose

สำรองข้อมูลสำคัญออกนอกระบบเป็นระยะ

### Backup Items

- daily summary
- issue summary
- export log
- audit log critical
- alias master

---

# 10. End-to-End Data Flow

## 10.1 Import Inquiry Flow

```text
Admin upload Inquiry
→ Import Batch
→ Raw Rows
→ Column Mapping
→ Alias Mapping
→ Normalize
→ Create Anchor Snapshot
→ Update Data Coverage
→ Audit Log
```

## 10.2 Import In-Out Backfill Flow

```text
Admin select date range
→ upload In-Out
→ Import Batch
→ Normalize
→ Alias Mapping
→ Build System Movement Ledger
→ Stock Replay
→ Reconciliation
→ Issue/Task
→ Summary
```

## 10.3 Field Movement Flow

```text
Checker/Staff report movement
→ Validate permission/session
→ Save Field Movement
→ Add to Movement Ledger
→ Wait System Post
→ Reconciliation when In-Out arrives
```

## 10.4 Pending Inbound Flow

```text
Checker enters label text + qty + location
→ system tries parse alias
→ if unknown, save Pending Inbound
→ later In-Out import creates candidates
→ Admin confirms
→ canonical movement created
```

## 10.5 Reconciliation Flow

```text
Movement Ledger + Daily Stock Position + Counts
→ Reconciliation Engine
→ Result Code
→ Explanation
→ Issue/Task
→ Notification
→ Control Center Summary
```

## 10.6 Retention Flow

```text
Nightly job
→ health check
→ create summary/archive
→ find records older than 45 days
→ skip open issues
→ delete raw/closed records
→ write retention log
→ notify admin
```

---

# 11. Module Dependency Map

| Module | Depends On | Used By |
|---|---|---|
| Auth | User Accounts | All modules |
| Permission | Auth, Roles | All APIs |
| Import Center | Auth, Permission, Audit | Snapshot, Ledger |
| Alias Center | Master Data | Import, Smart Input, Reconciliation |
| Snapshot Control | Import | Stock Replay |
| Movement Ledger | Import, Field Movement | Stock Replay, Reconciliation |
| Stock Replay | Snapshot, Ledger | Daily Position, Reconciliation |
| Reconciliation | Ledger, Count, Daily Position | Issue Center |
| Issue Center | Reconciliation | Notification, Control Center |
| Notification | Issue, Rules | Admin/Supervisor/ERP Admin |
| Export | Issue, Results, Summary | ERP Admin/Admin |
| Retention | Summary, Issue, Audit | Maintenance |

---

# 12. MVP Module Inclusion

## Must Build in MVP

| Module | Required |
|---|---|
| Auth | Yes |
| User Approval | Yes |
| Permission | Yes |
| Import Center | Yes |
| Column Mapping | Yes |
| Alias Center | Yes |
| Anchor Snapshot | Yes |
| In-Out Backfill | Yes |
| Movement Ledger | Yes |
| Stock Replay | Yes |
| Reconciliation | Yes |
| Explanation Detail | Yes |
| Issue Center | Yes |
| Control Center | Yes |
| Export Basic | Yes |
| Notification Basic | Yes |
| Retention 45 Days | Yes |
| Audit Log | Yes |

## Not Required in MVP

| Module | Reason |
|---|---|
| Full Relocation Optimizer | Too complex, after data stable |
| Advanced FIFO | Needs reliable lot data first |
| Full Custody Workflow | Future phase |
| Visual Warehouse Map | UX enhancement later |
| Full Offline Mode | Future mobile phase |
| AI OCR Photo Reading | Future automation |

---

# 13. Technical Architecture Options

## Option A: Google Apps Script + Google Sheets

### Pros

- เริ่มเร็ว
- ใช้กับทีมเล็กได้
- เชื่อม Google Drive/Sheets ง่าย

### Cons

- ข้อมูลเยอะจะช้า
- timeout ง่าย
- concurrent write จำกัด
- long-term production เสี่ยง

### Use Case

เหมาะกับ MVP/Prototype เท่านั้น

---

## Option B: Supabase / Postgres

### Pros

- เหมาะกับ database จริง
- มี auth, RLS, storage, cron
- รองรับ retention job
- query summary ได้เร็วกว่า Sheets

### Cons

- ต้องออกแบบ schema ดี
- Free plan มี limit
- ต้อง monitor DB size

### Use Case

เหมาะกับ MVP ที่ตั้งใจไป production ต่อ

---

## Option C: MySQL / PHP or Node Backend

### Pros

- คุมระบบเองได้มาก
- เหมาะกับ production ระยะยาว

### Cons

- setup/devops มากกว่า
- ต้อง host/server

---

## Recommended Direction

```text
ถ้าต้องเร็วมาก: GAS + Sheets สำหรับ foundation prototype
ถ้าต้องมั่นคง: Supabase/Postgres ตั้งแต่ต้น
```

---

# 14. System Boundaries

## System Owns

- User approval
- Alias mapping
- Import staging
- Snapshot control
- Movement ledger
- Stock replay
- Reconciliation result
- Issue/task
- Notification
- Export
- Retention
- Audit

## System Does Not Own in MVP

- ERP posting actual transaction
- Full accounting
- Full WMS execution
- Forklift tracking
- Automatic OCR reading
- Advanced optimization

---

# 15. Important Design Constraints

1. ห้าม hardcode column mapping
2. ห้าม hardcode alias
3. ห้ามให้ dashboard อ่าน raw data ทั้งหมด
4. ห้ามลบ data เกิน 45 วันถ้ายังมี open issue
5. ห้ามให้ pending user ใช้งานก่อน approve
6. ห้ามแก้ locked day ตรง
7. ห้ามให้ notification ส่งซ้ำโดยไม่มี cooldown
8. ห้ามให้ reconciliation ฟันธงถ้าข้อมูล coverage ต่ำ
9. ห้ามเริ่มจาก advanced simulation ก่อน core data stable
10. ห้ามปน Inquiry กับ Movement

---

# 16. Structure Acceptance Criteria

เอกสารโครงสร้างนี้ถือว่า complete ถ้าระบบที่สร้างตามเอกสารสามารถ:

- [ ] แบ่ง layer ชัดเจน
- [ ] แยก Control, Operation, Data, Intelligence, UI, Maintenance
- [ ] รองรับ Inquiry เป็น Anchor Snapshot
- [ ] รองรับ In-Out Backfill
- [ ] รองรับ Movement Ledger
- [ ] รองรับ Stock Replay
- [ ] รองรับ Alias Center
- [ ] รองรับ Auto Reconciliation
- [ ] รองรับ Issue Center
- [ ] รองรับ Control Center
- [ ] รองรับ Notification/Export/Retention
- [ ] ระบุสิ่งที่ไม่ทำใน MVP ชัดเจน

---

# 17. Final Structure Statement

```text
Warehouse Reconciliation ERP Platform ต้องถูกสร้างเป็น modular platform ที่แยกข้อมูลดิบ ข้อมูลกลาง logic การตรวจสอบ และหน้าตัดสินใจออกจากกันอย่างชัดเจน โดยมี Anchor Snapshot, Movement Ledger, Stock Replay และ Reconciliation Engine เป็นแกนหลัก เพื่อให้ระบบรองรับข้อมูลไม่ครบ ข้อมูลย้อนหลัง ชื่อไม่ตรง และการปิดวันได้อย่างมั่นคง
```

