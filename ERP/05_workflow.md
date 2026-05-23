# 05_workflow.md

# Workflow Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Workflow / Process Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer Use  
**Primary Use:** ใช้กำหนด flow การทำงานจริงของระบบตั้งแต่สมัครผู้ใช้, upload ไฟล์, map alias, backfill, stock replay, reconciliation, issue, notification, export, close day และ retention

---

## 1. Purpose of This Document

เอกสารนี้กำหนด workflow หลักของ Warehouse Reconciliation ERP Platform แบบ end-to-end

เป้าหมายคือให้ AI Agent / Developer เข้าใจว่าแต่ละ process ต้องไหลอย่างไร ใครเริ่ม process ข้อมูลอะไรเข้า ระบบต้อง validate อะไร ผลลัพธ์คืออะไร และต้องเชื่อมต่อ module ไหน

ระบบนี้ไม่ใช่ CRUD ธรรมดา ดังนั้น workflow ต้องชัดเจนมาก เพราะข้อมูลมาจากหลายแหล่งและมีความจริงหลายชั้น:

```text
Physical Count
Field Movement
System In-Out
Inquiry Snapshot
Expected / Stock Replay
```

หลักสำคัญของ workflow ทั้งระบบ:

```text
Input
→ Permission Check
→ Validation
→ Staging / Processing
→ Business Rule
→ Result / Status
→ Issue / Task / Notification
→ Audit Log
→ Summary / Dashboard
```

---

## 2. Workflow Design Principles

ทุก workflow ต้องยึดหลักต่อไปนี้

### 2.1 Raw Before Decision

ข้อมูลที่ upload เข้าระบบต้องเข้า raw/staging ก่อนเสมอ

```text
Upload
→ Raw Staging
→ Mapping
→ Normalize
→ Validate
→ Confirm
→ Use in Engine
```

ห้าม:

```text
Upload
→ ใช้คำนวณทันทีโดยไม่มี preview/mapping
```

---

### 2.2 System Checks First

ระบบต้องเช็คก่อนว่ามีอะไรผิดหรือเสี่ยง

คนไม่ควรไล่ดูทุก record เอง

```text
System checks all records
→ Human reviews exceptions only
```

---

### 2.3 Every Critical Action Has Audit

ทุก action สำคัญต้องมี audit log เช่น:

- approve user
- reject user request
- import confirm
- alias map
- run stock replay
- run reconciliation
- close issue
- export
- lock day
- retention cleanup

---

### 2.4 Status-Driven Workflow

ข้อมูลทุกตัวที่มี lifecycle ต้องมี status ชัดเจน

ตัวอย่าง:

```text
PENDING_APPROVAL
ACTIVE
WAIT_SYSTEM_POST
MATCHED
PROBLEM
CLOSED
LOCKED
```

ห้ามใช้ boolean เดียวแทน process ที่ซับซ้อน เช่น `done = true` โดยไม่มี status ระหว่างทาง

---

### 2.5 Confidence-Aware Workflow

ระบบต้องรู้ว่าข้อมูลที่ใช้ตัดสินใจมั่นใจแค่ไหน

ตัวอย่าง:

```text
HIGH      = Field + System + Count ตรงกัน
MEDIUM    = Field + System ตรง แต่ไม่มี Count
LOW       = ข้อมูลบางแหล่งขาด
CRITICAL  = REJ / Lot / Qty ใหญ่ / Unknown / No Anchor
```

---

# 3. Workflow Index

เอกสารนี้กำหนด workflow หลักต่อไปนี้:

| No. | Workflow | MVP | Main Actor |
|---:|---|---:|---|
| 1 | User Registration & Approval | Yes | User / Admin |
| 2 | Login & Session | Yes | All Users |
| 3 | Checker GPS Check-in | Yes | Checker |
| 4 | Import Inquiry | Yes | Admin |
| 5 | Import System In-Out | Yes | Admin / ERP Admin |
| 6 | Backfill In-Out ข้ามวัน | Yes | Admin |
| 7 | Column Mapping | Yes | Admin |
| 8 | Alias Review | Yes | Admin |
| 9 | Smart Inbound Input | Yes | Checker / Staff |
| 10 | Pending Inbound Confirm | Yes | Admin / Supervisor |
| 11 | Field Movement Notice | Yes | Checker / Staff |
| 12 | Physical Count / Recount | Yes | Checker |
| 13 | REJ Notice & Follow-up | Yes | Checker / ERP Admin |
| 14 | Movement Ledger Build | Yes | System |
| 15 | Stock Replay | Yes | System / Admin |
| 16 | Data Coverage Check | Yes | System |
| 17 | Auto Reconciliation | Yes | System |
| 18 | Issue Creation & Assignment | Yes | System / Supervisor |
| 19 | Issue Detail Review | Yes | Supervisor / Admin |
| 20 | Notification Dispatch | Yes | System |
| 21 | Export | Yes | Admin / ERP Admin |
| 22 | Daily Close / Lock | Yes | Admin / Supervisor |
| 23 | Retention 45 Days | Yes | System / Admin |
| 24 | Health Check | Recommended | System |
| 25 | Capacity / Simulation | Later | Planner |

---

# 4. Workflow 1 — User Registration & Approval

## 4.1 Purpose

ให้ผู้ใช้สมัครและตั้งรหัสได้ แต่ยังไม่สามารถใช้งานระบบจริงจนกว่า Admin จะอนุมัติและกำหนดสิทธิ์

## 4.2 Actors

- New User
- Admin
- System

## 4.3 Trigger

User เปิดหน้าสมัครบัญชี

## 4.4 Input

- username
- password
- full_name
- phone
- email optional
- requested_role optional
- requested_warehouse optional

## 4.5 Flow

```text
User submits registration
→ System validates username/email
→ System hashes password + salt
→ Create USER_ACCESS_REQUESTS
→ status = PENDING_APPROVAL
→ Notify Admin
→ User sees waiting approval message
```

## 4.6 Admin Approval Flow

```text
Admin opens User Approval Center
→ Select pending request
→ Review user data
→ Select role
→ Select permission
→ Select warehouse scope
→ Approve
→ Create USER_ACCOUNTS
→ Assign role/permission/scope
→ status = ACTIVE
→ Write audit log
→ Notify user/admin
```

## 4.7 Admin Reject Flow

```text
Admin opens request
→ Click Reject
→ Enter reason optional/required by config
→ Delete request from pending queue
→ Write audit log USER_REJECT_DELETE_REQUEST
→ Notify admin optional
```

## 4.8 Validation

- username required
- password required
- username unique
- pending request exists
- approver has CAN_MANAGE_USER
- approve requires role or permission
- operation-related role requires warehouse scope

## 4.9 Output

- Active user account if approved
- Deleted pending request + audit if rejected
- Notification
- Audit log

## 4.10 Failure Cases

| Case | Handling |
|---|---|
| username already exists | show error |
| approver has no permission | block action |
| missing role/permission | block approve |
| reject fails audit | block delete until audit write succeeds |

---

# 5. Workflow 2 — Login & Session

## 5.1 Purpose

ให้ active user เข้าใช้งานได้ตามสิทธิ์ และ block user ที่ยังไม่อนุมัติ/ถูกล็อค/หมดอายุ

## 5.2 Actors

- User
- System

## 5.3 Flow

```text
User enters username/password
→ System finds user
→ Check user status
→ Verify password hash
→ Create session token
→ Load role/permission/warehouse scope
→ Return user context
→ Write login audit
```

## 5.4 Status Rule

| User Status | Login Result |
|---|---|
| ACTIVE | Allowed |
| PENDING_APPROVAL | Block |
| LOCKED | Block |
| SUSPENDED | Block/Partial depending config |
| EXPIRED | Block |
| DELETED | Block |

## 5.5 Output

- session token
- user profile
- permission list
- warehouse scope

## 5.6 Audit

- USER_LOGIN_SUCCESS
- USER_LOGIN_FAILED
- USER_LOGOUT

---

# 6. Workflow 3 — Checker GPS Check-in

## 6.1 Purpose

ให้ Checker outsource check-in ที่ warehouse ก่อนทำงาน และจำสิทธิ์นับในวันนั้น

## 6.2 Actors

- Checker
- System

## 6.3 Trigger

Checker กดปุ่ม Check-in

## 6.4 Input

- checker_user_id
- warehouse_id
- GPS lat/lng
- GPS accuracy
- device_id optional
- business_date

## 6.5 Flow

```text
Checker login
→ Open Check-in page
→ Select warehouse
→ System requests GPS permission
→ User grants location
→ System records lat/lng/accuracy
→ Optional: check warehouse radius
→ If user has active session at another WH, suspend old session
→ Create new CHECKER_DAILY_SESSION
→ status = ACTIVE
→ Checker can count/task in this warehouse today
→ Audit log
```

## 6.6 Validation

- user has CAN_CHECKIN_WAREHOUSE
- user status ACTIVE
- warehouse in user scope
- GPS required if config enabled
- if outside radius, block or warning based on config

## 6.7 Output

- active checker session
- old session suspended if needed
- audit log

## 6.8 Failure Cases

| Case | Handling |
|---|---|
| GPS denied | show manual fallback or block by config |
| outside radius | warning/block |
| user no checker permission | block |
| warehouse not in scope | block |

---

# 7. Workflow 4 — Import Inquiry

## 7.1 Purpose

รับไฟล์ Inquiry จาก ERP เป็น Anchor Snapshot โดยไม่ถือว่าเป็น movement

## 7.2 Actors

- Admin
- System

## 7.3 Trigger

Admin upload Inquiry file

## 7.4 Input

- Inquiry Excel/CSV
- file_type = INQUIRY
- business_date
- snapshot_time optional
- warehouse/DC optional

## 7.5 Flow

```text
Admin opens Import Center
→ Select file type = Inquiry
→ Upload file
→ Client parses file
→ Preview headers and sample rows
→ Create IMPORT_BATCH
→ Check checksum duplicate
→ Column Mapping
→ Chunk upload raw rows
→ Save IMPORT_RAW_ROWS
→ Validate required fields
→ Alias mapping for warehouse/location/customer/item
→ Unknown alias goes to Alias Review
→ Admin confirms import
→ Create INQUIRY_SNAPSHOT
→ Mark as active/archive/replace based on policy
→ Update Data Coverage
→ Write audit log
```

## 7.6 Important Rule

```text
Inquiry = Snapshot
ห้ามเอา Inquiry เป็น movement
```

## 7.7 Snapshot Duplicate Policy

If same business_date already has active Inquiry:

Options:

1. Replace old snapshot
2. Archive old snapshot
3. Cancel new upload
4. Keep both but only one active

ต้องไม่มี silent append

## 7.8 Validation

- file type valid
- row count > 0
- required columns mapped
- checksum checked
- business_date required
- unknown alias count shown
- active snapshot conflict handled

## 7.9 Output

- IMPORT_BATCH
- IMPORT_RAW_ROWS
- INQUIRY_SNAPSHOT
- normalized stock rows optional
- audit log
- coverage update

## 7.10 Failure Cases

| Case | Handling |
|---|---|
| duplicate checksum | warn/block based policy |
| missing required columns | block confirm |
| unknown location/customer/item | send to Alias Review |
| same date active snapshot exists | require replace/archive/cancel decision |

---

# 8. Workflow 5 — Import System In-Out

## 8.1 Purpose

รับ movement ที่ ERP post แล้ว เพื่อใช้เทียบ Field Movement และใช้ replay stock ข้ามวัน

## 8.2 Actors

- Admin
- ERP Admin
- System

## 8.3 Trigger

Upload In-Out file

## 8.4 Input

- In-Out Excel/CSV
- date range
- warehouse/DC
- movement rows

## 8.5 Flow

```text
User opens Import Center
→ Select file type = SYSTEM_INOUT
→ Upload file
→ Client parses file
→ Preview headers/sample rows
→ Create IMPORT_BATCH
→ Column Mapping
→ Chunk upload raw rows
→ Normalize rows
→ Alias mapping
→ Detect unknown alias
→ Create SYSTEM_INOUT normalized records
→ Create Movement Ledger entries source_type = SYSTEM_INOUT
→ Update Data Coverage
→ Trigger Stock Replay if date range selected
→ Trigger Reconciliation job
→ Write audit log
```

## 8.6 Validation

- movement date or posting date must be mapped
- qty valid
- movement type parseable
- location/DC mapped or queued
- customer/item mapped or queued
- duplicate source_ref handling

## 8.7 Output

- import batch
- normalized in-out rows
- movement ledger entries
- coverage update
- reconciliation job queued

## 8.8 Failure Cases

| Case | Handling |
|---|---|
| movement type unknown | queue mapping/review |
| location unknown | alias review |
| duplicate ERP row | mark duplicate candidate |
| date missing | block normalize |

---

# 9. Workflow 6 — Backfill In-Out ข้ามวัน

## 9.1 Purpose

รองรับกรณี Admin ไม่ได้ upload Inquiry/In-Out ทุกวัน เช่น ลา/หยุด แล้วต้องดึงย้อนหลัง 1–5 วัน

## 9.2 Actors

- Admin
- System

## 9.3 Trigger

Admin เปิด Backfill Wizard และเลือกช่วงวันที่

## 9.4 Input

- date_from
- date_to
- System In-Out file(s)
- optional Inquiry anchor

## 9.5 Flow

```text
Admin opens Backfill Wizard
→ Select missing date range
→ Upload System In-Out files
→ Import + Normalize
→ Coverage Check
→ Find latest Anchor Snapshot before date_from
→ If no anchor, show MISSING_ANCHOR
→ Build Movement Ledger for date range
→ Run Stock Replay
→ Generate Daily Stock Position
→ Run Reconciliation per day
→ Create Count Required Locations
→ Create Issues/Tasks
→ Update Reconciliation Calendar
→ Show Backfill Summary
→ Notify if completed/failed
```

## 9.6 Coverage Check

ตรวจ:

- มี anchor snapshot ก่อนช่วงไหม
- In-Out ครบทุกวันไหม
- มี unknown alias ไหม
- มี pending inbound ไหม
- มี REJ pending ไหม
- มี location ที่ movement แต่ยังไม่ count ไหม

## 9.7 Output

- daily stock positions
- coverage status
- issue/task list
- count required locations
- backfill summary

## 9.8 Status

```text
BACKFILL_PENDING
BACKFILL_RUNNING
BACKFILL_COMPLETED
BACKFILL_COMPLETED_WITH_WARNING
BACKFILL_FAILED
```

---

# 10. Workflow 7 — Column Mapping

## 10.1 Purpose

ให้ Admin map column จากไฟล์จริงเป็น canonical field โดยไม่ต้องแก้โค้ด

## 10.2 Actors

- Admin
- System

## 10.3 Flow

```text
Import file parsed
→ System displays headers
→ System suggests mapping if existing template
→ Admin reviews mapping
→ Admin maps required fields
→ System validates required fields
→ Save mapping template
→ Continue normalize
```

## 10.4 Required Fields by File Type

### Inquiry

- business_date
- warehouse/DC/location field
- item/customer field or description
- qty
- uom optional

### System In-Out

- movement_date or posting_date
- movement_type
- warehouse/DC
- location/from-to location
- item/customer
- qty
- uom
- source_ref optional

## 10.5 Failure Cases

| Case | Handling |
|---|---|
| required field missing | block normalize |
| duplicated canonical mapping | warning/block |
| header changed | show warning |

---

# 11. Workflow 8 — Alias Review

## 11.1 Purpose

ให้ Admin map unknown alias ให้เป็น canonical master หรือสร้าง master ใหม่

## 11.2 Actors

- Admin
- System

## 11.3 Trigger

ระบบพบ unknown text จาก import หรือ smart input

## 11.4 Flow

```text
Unknown alias detected
→ Create ALIAS_REVIEW_QUEUE
→ System suggests possible master with confidence
→ Admin opens Alias Review
→ Admin chooses action:
   1. Map to existing master
   2. Create new master
   3. Reject
→ System updates alias table
→ Reprocess affected rows optional
→ Audit log
```

## 11.5 Alias Types

- CUSTOMER_ALIAS
- ITEM_ALIAS
- LOCATION_ALIAS
- DC_ALIAS
- UOM_ALIAS
- LOT_PATTERN

## 11.6 Validation

- alias type required
- raw_text required
- canonical target required when mapping
- duplicate alias warning
- low confidence requires manual approval

## 11.7 Output

- alias mapping
- created master optional
- affected rows reprocessed
- audit log

---

# 12. Workflow 9 — Smart Inbound Input

## 12.1 Purpose

ให้หน้างานแจ้งของเข้าโดยพิมพ์ข้อความจากป้าย เช่น `MKMK W150 69` แทนการเลือก dropdown ยาว ๆ

## 12.2 Actors

- Checker
- Staff
- System

## 12.3 Input

- text_on_label
- qty
- uom
- warehouse/location
- photo optional
- business_date

## 12.4 Flow

```text
Checker opens แจ้งเข้า
→ Enter label text เช่น MKMK W150 69
→ System tokenizes text
→ Match customer alias
→ Match item alias
→ Parse year/lot if possible
→ Show preview with confidence
→ User confirms if correct
→ If confidence high and data complete, create Field Movement IN
→ If incomplete/unknown, create Pending Inbound
→ Audit log
```

## 12.5 Example

Input:

```text
MKMK W150 69
```

Parsed:

```text
Customer = MKMK
Item = W150
Year = 69
Confidence = 95%
```

## 12.6 Unknown Case

```text
Unknown item or customer
→ Create Pending Inbound
→ Create Alias Review Queue
→ status = WAIT_SYSTEM_INOUT
```

---

# 13. Workflow 10 — Pending Inbound Confirm

## 13.1 Purpose

ยืนยันรายการรับเข้าที่ยังไม่รู้ข้อมูลครบ โดยใช้ In-Out วันถัดไปหรือ Admin review

## 13.2 Actors

- Admin
- Supervisor
- System

## 13.3 Trigger

System In-Out import เสร็จ หรือ Admin เปิด Pending Inbound Review

## 13.4 Flow

```text
Pending Inbound exists
→ System searches candidate from System In-Out
→ Match by date/location/qty/text similarity/customer/item
→ Create candidate list with confidence
→ Admin/Supervisor reviews
→ Confirm match or edit mapping
→ Convert pending inbound to canonical movement
→ Add ledger entry
→ Resolve pending issue
→ Audit log
```

## 13.5 Candidate Matching Inputs

- business_date
- warehouse/location
- qty
- text_on_label
- customer alias
- item alias
- source_ref optional
- time window optional

## 13.6 Output

- confirmed inbound
- movement ledger entry
- alias updated optional
- issue closed or updated

---

# 14. Workflow 11 — Field Movement Notice

## 14.1 Purpose

บันทึก movement ที่หน้างานแจ้ง เช่น OUT, TRANSFER, REJ

## 14.2 Actors

- Checker
- Staff
- Supervisor

## 14.3 Flow

```text
User opens movement form
→ Select movement type
→ Enter/scan location
→ Enter smart item/customer if needed
→ Enter qty/uom
→ Attach photo optional
→ System validates permission/session
→ Save Field Movement
→ Add Movement Ledger source_type = FIELD_NOTICE
→ status = WAIT_SYSTEM_POST
→ Audit log
```

## 14.4 Validation

- movement_type required
- qty > 0
- location required
- locked day block
- checker session required if checker

## 14.5 Output

- field movement
- ledger entry
- wait system post status

---

# 15. Workflow 12 — Physical Count / Recount

## 15.1 Purpose

ให้ Checker นับของจริง และใช้ count เป็นหลักฐานเทียบ Expected Stock

## 15.2 Actors

- Checker
- Supervisor
- System

## 15.3 Flow

```text
System creates count task OR checker starts count
→ Checker must have active session
→ Open task/count page
→ Confirm location
→ Enter qty
→ Enter item/customer/lot if required
→ Attach photo optional
→ Submit count
→ System validates
→ Save Physical Count
→ Link to task/issue if any
→ Reconciliation may run/update result
→ Audit log
```

## 15.4 Count Types

- BULK_COUNT
- BEFORE_COUNT
- AFTER_COUNT
- RECOUNT
- LOT_VERIFY

## 15.5 Output

- physical count record
- task status update
- issue update
- reconciliation input

---

# 16. Workflow 13 — REJ Notice & Follow-up

## 16.1 Purpose

แจ้งของเสีย/เสียหายและตามจน ERP post

## 16.2 Actors

- Checker
- Supervisor
- ERP Admin
- System

## 16.3 Flow

```text
Checker reports REJ
→ Enter from location/item/qty/reason
→ System validates
→ Create REJ Notice
→ Add Movement Ledger source_type = REJ_NOTICE
→ status = WAIT_SYSTEM_POST
→ When In-Out imported, Reconciliation checks ERP REJ movement
→ If matched, status = REJ_MATCHED
→ If not found, issue = REJ_NOT_POSTED
→ Notify ERP Admin
→ ERP Admin exports/posts/marks posted
→ Final verify and close
```

## 16.4 Output

- REJ notice
- ledger entry
- issue if not posted
- notification
- export if needed

---

# 17. Workflow 14 — Movement Ledger Build

## 17.1 Purpose

รวม movement ทุก source ให้เป็น ledger กลาง

## 17.2 Actors

- System

## 17.3 Sources

- SYSTEM_INOUT
- FIELD_NOTICE
- REJ_NOTICE
- TRANSFER
- ADJUSTMENT
- COUNT_CORRECTION
- PENDING_INBOUND_CONFIRM

## 17.4 Flow

```text
Source record created/normalized
→ Validate movement fields
→ Map to canonical customer/item/location/uom
→ Create ledger entry
→ Set source_type/source_ref_id
→ Update affected date/location/item index
```

## 17.5 Output

- movement ledger entry
- replay job candidate

---

# 18. Workflow 15 — Stock Replay

## 18.1 Purpose

คำนวณ stock position รายวันจาก Anchor Snapshot + Movement Ledger

## 18.2 Actors

- System
- Admin optional

## 18.3 Trigger

- Inquiry import
- In-Out import
- Backfill wizard
- Movement correction
- Manual run

## 18.4 Flow

```text
Select date range
→ Find latest Anchor Snapshot before date range
→ Load movement ledger in range
→ For each date/location/customer/item/year/lot:
   opening_qty
   + in_qty
   - out_qty
   + transfer_in
   - transfer_out
   - rej_qty
   = ending_expected_qty
→ Save Daily Stock Position
→ Assign source_mode/confidence
→ Write replay log
→ Update summary
```

## 18.5 Confidence

- HIGH = anchor + system + count confirmed
- MEDIUM = anchor + system movement complete
- LOW = missing some movement/count
- CRITICAL = no anchor / unknown inbound / REJ pending

---

# 19. Workflow 16 — Data Coverage Check

## 19.1 Purpose

ตรวจว่าข้อมูลช่วงวันที่นั้นครบพอสำหรับ replay/reconciliation/close หรือไม่

## 19.2 Flow

```text
Select date range
→ Check anchor snapshot
→ Check In-Out coverage per day
→ Check unknown alias
→ Check pending inbound
→ Check REJ pending
→ Check moving locations with no count
→ Return coverage status
```

## 19.3 Output

- coverage_status
- missing_items list
- warning/critical flags

---

# 20. Workflow 17 — Auto Reconciliation

## 20.1 Purpose

ให้ระบบเทียบข้อมูลและสร้าง result/issue อัตโนมัติ

## 20.2 Actors

- System
- Admin optional

## 20.3 Flow

```text
Load Field Movement
→ Load System In-Out
→ Load Daily Stock Position
→ Load Physical Count
→ Load REJ Notices
→ Match records by configurable matching key
→ Compare quantities/location/item/lot
→ Generate result_code
→ Calculate severity
→ Calculate confidence
→ Generate explanation
→ Create/Update issue if needed
→ Update summary
→ Notify based on rules
```

## 20.4 Result Codes

- MATCHED
- SYSTEM_MISSING
- FIELD_MISSING
- QTY_DIFF
- LOCATION_DIFF
- ITEM_DIFF
- LOT_DIFF
- REJ_NOT_POSTED
- UNKNOWN_ALIAS
- PENDING_INBOUND
- NEED_RECOUNT

## 20.5 Output

- recon_results
- issues
- tasks
- notifications
- summary

---

# 21. Workflow 18 — Issue Creation & Assignment

## 21.1 Purpose

สร้าง issue จาก result ที่ผิด/เสี่ยง และ assign ให้ผู้รับผิดชอบ

## 21.2 Flow

```text
Reconciliation result generated
→ If result is not MATCHED or warning/critical
→ Create issue
→ Determine owner_role
→ Determine suggested_action
→ Set due/SLA if configured
→ Notify owner
```

## 21.3 Owner Mapping

| Result | Owner Role |
|---|---|
| SYSTEM_MISSING | ERP Admin |
| FIELD_MISSING | Supervisor |
| QTY_DIFF | Supervisor |
| LOCATION_DIFF | Checker/Supervisor |
| REJ_NOT_POSTED | ERP Admin |
| UNKNOWN_ALIAS | Admin |
| PENDING_INBOUND | Admin/Supervisor |

---

# 22. Workflow 19 — Issue Detail Review

## 22.1 Purpose

ให้ user กดดูรายละเอียดว่าข้อมูลไม่ตรงอย่างไร และต้องทำอะไรต่อ

## 22.2 Flow

```text
User opens Issue Center
→ Select issue
→ System loads detail
→ Show Field vs System vs Count vs Expected
→ Show explanation/likely cause/suggested action
→ User selects action:
   Assign Recount
   Send to ERP Admin
   Map Alias
   Confirm Pending Inbound
   Accept with Reason
   Close Issue
→ Audit log
```

## 22.3 Required Display

- result_code
- severity
- confidence
- item/customer/location/year/lot
- field data
- system data
- count data
- expected data
- difference
- explanation
- suggested action
- audit trail

---

# 23. Workflow 20 — Notification Dispatch

## 23.1 Purpose

แจ้งเตือนผู้เกี่ยวข้องเมื่อเกิด event สำคัญ โดยไม่ spam

## 23.2 Flow

```text
Event occurs
→ NotificationService receives event_code + payload
→ Load active notification rules
→ Filter by severity/channel/role
→ Check cooldown/send_once_per_issue
→ Render template
→ Send via In-app/Telegram/LINE/Email
→ Write notification log
→ Retry if failed based on policy
```

## 23.3 Events

- USER_PENDING_APPROVAL
- UNKNOWN_ALIAS_CREATED
- PENDING_INBOUND_CREATED
- SYSTEM_MISSING
- QTY_DIFF
- REJ_NOT_POSTED
- EXPORT_READY
- RETENTION_FAILED
- DB_USAGE_WARNING

---

# 24. Workflow 21 — Export

## 24.1 Purpose

ส่งออกข้อมูลสำหรับ ERP Admin, Supervisor, Management หรือ audit

## 24.2 Actors

- Admin
- ERP Admin
- Supervisor optional

## 24.3 Flow

```text
User opens Export Center
→ Select export type
→ Select filters or selected rows
→ System validates permission
→ Generate export data from result/summary tables
→ Create file
→ Save export job/log
→ Provide download link
→ Optional notify recipient
```

## 24.4 Export Types

- Issue List
- Daily Reconciliation
- REJ Report
- ERP Correction Template
- Count Task
- Retention Report
- Audit Export

## 24.5 Rules

- Every export must have export log
- Export must include filter criteria
- Export official ERP template requires permission

---

# 25. Workflow 22 — Daily Close / Lock

## 25.1 Purpose

ปิดวันหรือ lock วันโดยมี data quality gate

## 25.2 Daily Status

```text
OPEN
PRELIM_CLOSED
RECONSTRUCTED
FINAL_CLOSED
LOCKED
```

## 25.3 Flow

```text
Admin/Supervisor opens Close Readiness
→ System runs data coverage check
→ Check open critical issues
→ Check pending alias
→ Check pending inbound
→ Check REJ_NOT_POSTED
→ Check pending export/approval
→ Return READY / READY_WITH_WARNING / NOT_READY
→ User confirms close if allowed
→ Update day status
→ Write audit log
```

## 25.4 Lock Rule

```text
LOCKED day cannot be edited directly
All changes must go through Investigation / Correction
```

---

# 26. Workflow 23 — Retention 45 Days

## 26.1 Purpose

ลบ/archive ข้อมูลเกิน 45 วันโดยไม่ทำให้หลักฐานและ issue หาย

## 26.2 Trigger

Nightly job หรือ Admin manual run

## 26.3 Flow

```text
Start retention job
→ Run health check
→ cutoff_date = today - 45 days
→ Create/archive daily/monthly summary
→ Find deletable raw/staging/closed records
→ Skip open issues/pending export/pending approval
→ Delete safe records
→ Write retention log
→ Notify Admin summary
```

## 26.4 Delete Allowed

- import_raw_rows normalized/closed
- staging data
- matched/closed movement old
- sent/skipped notification logs

## 26.5 Delete Not Allowed

- open issue
- pending approval
- pending export
- REJ_NOT_POSTED
- SYSTEM_MISSING
- export log
- critical audit log
- daily close summary

---

# 27. Workflow 24 — Health Check

## 27.1 Purpose

ให้ระบบเช็คตัวเองเพื่อลดการดูแล manual

## 27.2 Checks

- DB usage
- row count by table
- last import
- last stock replay
- last reconciliation
- last retention
- open critical issues
- pending alias
- pending inbound
- failed notification

## 27.3 Flow

```text
Scheduled health check
→ Collect metrics
→ Compare threshold
→ Create health status
→ Notify Admin if warning/critical
```

---

# 28. Workflow 25 — Capacity / Simulation Later

## 28.1 Purpose

รองรับ phase หลังเรื่อง available location, capacity, relocation

## 28.2 Not MVP

ยังไม่ทำเต็มใน MVP

## 28.3 Future Flow

```text
Planner enters inbound requirement
→ Select policy: strict lot / allow multi-lot / ignore lot with approval
→ System finds available locations
→ If no space, suggest relocation
→ Move small to large strategy
→ Check capacity
→ Return scenarios
→ Supervisor approve if risky
```

---

# 29. Cross-Workflow Status Map

| Entity | Main Statuses |
|---|---|
| User Request | PENDING_APPROVAL / APPROVED / REJECTED |
| User | ACTIVE / LOCKED / SUSPENDED / EXPIRED |
| Checker Session | ACTIVE / SUSPENDED / EXPIRED |
| Import Batch | UPLOADED / MAPPED / NORMALIZED / CONFIRMED / FAILED |
| Alias Queue | PENDING / MAPPED / CREATED_NEW / REJECTED |
| Pending Inbound | PENDING / CANDIDATE_FOUND / CONFIRMED / INVESTIGATING |
| Field Movement | REPORTED / WAIT_SYSTEM_POST / MATCHED / PROBLEM / CLOSED |
| REJ | REPORTED / WAIT_SYSTEM_POST / MATCHED / NOT_POSTED / CLOSED |
| Issue | OPEN / ASSIGNED / IN_REVIEW / RESOLVED / CLOSED |
| Day | OPEN / PRELIM_CLOSED / RECONSTRUCTED / FINAL_CLOSED / LOCKED |
| Export | DRAFT / READY / EXPORTED / SENT / VERIFIED |
| Retention Job | RUNNING / SUCCESS / WARNING / FAILED |

---

# 30. Workflow Acceptance Criteria

Workflow design ถือว่า complete ถ้าระบบสามารถ:

- [ ] User สมัครแล้วรออนุมัติ
- [ ] Admin approve/reject พร้อม audit
- [ ] Checker check-in ก่อนนับ
- [ ] Upload Inquiry เป็น Anchor Snapshot
- [ ] Upload In-Out แบบ backfill ได้
- [ ] Column mapping ไม่ hardcode
- [ ] Unknown alias เข้า review
- [ ] Pending inbound ถูกสร้างและ confirm ได้
- [ ] Movement ทุกชนิดเข้า ledger
- [ ] Stock replay ข้ามวันได้
- [ ] Reconciliation สร้าง result + explanation ได้
- [ ] Issue ถูกสร้างและ assign ได้
- [ ] Notification ส่งตาม rule/cooldown
- [ ] Export มี log
- [ ] Close day มี readiness gate
- [ ] Retention เกิน 45 วันไม่ลบ open issue

---

# 31. Final Workflow Statement

```text
Workflow ของระบบนี้ต้องทำให้ข้อมูลไหลจาก raw input ไปสู่ decision อย่างปลอดภัย โดยผ่าน staging, mapping, alias, ledger, replay, reconciliation, issue, notification, export และ audit ทุกขั้นตอน เพื่อให้คนไม่ต้องไล่เช็คข้อมูลเอง แต่ยังตรวจย้อนหลังและควบคุมความถูกต้องได้
```

