# 12_business_rules.md

# Business Rules Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Business Rules / Decision Logic / Control Rule Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / System Analyst Use  
**Primary Use:** ใช้กำหนดกฎธุรกิจที่ระบบต้องยึดในการ import, alias, movement, stock replay, reconciliation, issue, permission, notification, export, close day และ retention เพื่อให้ระบบตัดสินใจได้ถูกต้องและตรวจย้อนหลังได้

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Business Rules ของ Warehouse Reconciliation ERP Platform

Business Rules คือกฎที่บอกว่า:

1. ระบบต้องเชื่อข้อมูลไหนอย่างไร
2. ข้อมูลไหนเป็น snapshot หรือ transaction
3. ข้อมูลไหนใช้คำนวณ stock ได้
4. ข้อมูลไหนต้องรอ review
5. เมื่อเกิด mismatch ต้องสร้าง result/issue แบบไหน
6. วันไหนปิดได้หรือไม่ได้
7. ข้อมูลเกิน 45 วันลบอะไรได้หรือไม่ได้
8. สิทธิ์ user ทำอะไรได้หรือไม่ได้
9. Notification ต้องส่งเมื่อไหร่และไม่ควร spam อย่างไร
10. Dashboard ต้องอ่านข้อมูลจากไหน

หลักสำคัญ:

```text
ระบบนี้ต้องไม่ใช่แค่บันทึกข้อมูล
แต่ต้องมีกฎควบคุมว่าข้อมูลไหนเชื่อได้แค่ไหน และต้องทำอะไรต่อเมื่อข้อมูลไม่ตรง
```

---

## 2. Business Rule Philosophy

## 2.1 Truth Separation Rule

ระบบต้องแยกความจริงแต่ละชั้นออกจากกัน

```text
Physical Count = ของจริงที่นับได้
Field Movement = หน้างานแจ้งว่าเกิดขึ้น
System In-Out = ERP post แล้ว
Inquiry = ERP snapshot
Expected Stock = ระบบคำนวณจาก Anchor + Ledger
```

ห้ามปนความหมาย เช่น:

```text
เอา Inquiry ไปเป็น movement
เอา Field Movement ไปถือว่า ERP post แล้ว
เอา Pending Inbound ไปถือว่าเป็น item confirmed แล้ว
```

---

## 2.2 System Checks Before Human Review

ระบบต้องตรวจทุก record ก่อน แล้วให้คนดูเฉพาะ exception

```text
System checks all
Human reviews exceptions
```

ห้ามออกแบบให้คนต้องไล่ดูทุก row เองเหมือน Excel

---

## 2.3 Explainable Result Rule

ทุกผลที่ไม่ใช่ MATCHED ต้องมี:

```text
result_code
severity
confidence
explanation
likely_cause
suggested_action
owner_role
```

ห้ามสร้าง issue ที่มีแค่ code โดยไม่มีคำอธิบาย

---

## 2.4 Configurable Rule Principle

กฎที่อาจเปลี่ยนตามไฟล์จริงหรือ policy ต้อง config ได้ ไม่ hardcode

เช่น:

- column mapping
- alias mapping
- matching key
- notification rule
- retention policy
- close readiness gate
- warehouse scope
- REJ location

---

# 3. Rule Categories

Business Rules แบ่งเป็นกลุ่มหลักดังนี้:

```text
1. Data Truth Rules
2. Import Rules
3. Inquiry Snapshot Rules
4. System In-Out Rules
5. Field Movement Rules
6. Pending Inbound Rules
7. Alias Rules
8. Movement Ledger Rules
9. Stock Replay Rules
10. Reconciliation Rules
11. Issue Rules
12. Physical Count Rules
13. REJ Rules
14. User / Permission Rules
15. Checker GPS Session Rules
16. Export Rules
17. Notification Rules
18. Close / Lock Rules
19. Retention Rules
20. Dashboard / KPI Rules
21. Performance Rules
22. Future Simulation Rules
```

---

# 4. Core Rule Summary

| Rule ID | Rule | Severity if Violated |
|---|---|---|
| BR-001 | Inquiry เป็น Snapshot ไม่ใช่ Transaction | Critical |
| BR-002 | ห้าม append Inquiry ซ้ำโดยไม่มี snapshot policy | Critical |
| BR-003 | Field Movement และ System In-Out ต้องแยกกัน | Critical |
| BR-004 | Stock Position ต้องมาจาก Anchor + Movement Ledger | Critical |
| BR-005 | ถ้าไม่มี Inquiry ทุกวัน ให้ใช้ Stock Replay | High |
| BR-006 | Unknown Alias ต้องเข้า Alias Review | High |
| BR-007 | Pending Inbound ต้อง confirm ก่อน Final Close | High |
| BR-008 | Issue ที่ยัง open ห้ามถูก retention delete | Critical |
| BR-009 | LOCKED day ห้ามแก้ตรง | Critical |
| BR-010 | Checker ต้อง check-in ก่อนนับ | High |
| BR-011 | Export ทุกครั้งต้องมี export log | High |
| BR-012 | Notification ต้องมี cooldown/send once | Medium |
| BR-013 | Dashboard ต้องอ่าน summary/result ไม่อ่าน raw full history | High |
| BR-014 | ทุก critical write ต้องมี audit log | Critical |
| BR-015 | Low confidence ห้าม auto-map/auto-close | High |

---

# 5. Data Truth Rules

## BR-DT-001 — Truth Layer Separation

### Rule

ข้อมูลแต่ละประเภทต้องอยู่ใน table และ logic ของตัวเอง

### Details

```text
Inquiry → inquiry_snapshots / inquiry_snapshot_lines
System In-Out → system_inout_records
Field Movement → field_movements
Physical Count → physical_counts
Expected Stock → daily_stock_positions
Result → recon_results
Issue → issues
```

### Reason

ป้องกันการคำนวณซ้ำหรือเข้าใจผิดว่าข้อมูลคนละชนิดเป็นสิ่งเดียวกัน

### Violation Impact

- stock เบิ้ล
- stock หาย
- reconcile ผิด
- audit ย้อนกลับไม่ได้

---

## BR-DT-002 — Source Traceability

### Rule

ข้อมูลสำคัญทุก record ต้อง trace กลับ source ได้

### Required Fields

- source_type
- source_ref_id
- batch_id if from import
- raw_row_id if from file
- created_by
- created_at

### Applies To

- system_inout_records
- field_movements
- movement_ledger
- recon_results
- issues
- export_jobs

---

## BR-DT-003 — Confidence Required

### Rule

ข้อมูลที่ผ่านการ match/normalize/reconcile ต้องมี confidence

### Confidence Values

```text
HIGH
MEDIUM
LOW
CRITICAL
```

### Usage

- Alias mapping
- Pending inbound match
- Stock replay
- Reconciliation result
- Issue severity

### Important

Low confidence ห้ามทำ auto-close หรือ auto-map โดยไม่มี human review

---

# 6. Import Rules

## BR-IMP-001 — Import Must Start as Staging

### Rule

ทุกไฟล์ต้องเข้า import staging ก่อนนำไปใช้จริง

### Correct Flow

```text
Upload
→ IMPORT_BATCH
→ IMPORT_RAW_ROWS
→ Column Mapping
→ Normalize
→ Alias Matching
→ Confirm
```

### Prohibited Flow

```text
Upload
→ Insert เข้า stock หรือ ledger ทันที
```

### Severity

Critical

---

## BR-IMP-002 — Import Must Have Batch ID

### Rule

ทุกไฟล์ที่ upload ต้องมี batch_id

### Reason

ใช้ trace:

- upload โดยใคร
- file ไหน
- row ไหน
- error อะไร
- confirm แล้วหรือยัง

---

## BR-IMP-003 — Preview Before Confirm

### Rule

Admin ต้องเห็น preview headers/sample rows ก่อน confirm import

### Required Preview

- file name
- file type
- row count
- headers
- sample rows
- duplicate checksum warning
- required mapping status
- unknown alias count

---

## BR-IMP-004 — Required Column Mapping

### Rule

ไฟล์ต้อง map column ที่จำเป็นก่อน normalize

### Inquiry Required Fields

- business_date หรือ snapshot date
- warehouse/DC/location raw
- item/customer/description raw
- qty

### System In-Out Required Fields

- movement_date หรือ posting_date
- movement_type
- warehouse/DC/location raw
- item/customer raw
- qty
- uom optional but recommended

---

## BR-IMP-005 — Duplicate File Warning

### Rule

ถ้า file_checksum ซ้ำ ต้องแจ้งเตือนหรือ block ตาม policy

### Possible Policies

```text
WARN_ONLY
BLOCK_DUPLICATE
ALLOW_WITH_REASON
```

### Recommended MVP

```text
WARN_ONLY + require confirm reason if same file/date
```

---

## BR-IMP-006 — Import Confirmation Must Be Audited

### Rule

เมื่อ confirm import ต้องเขียน audit log

### Audit Event

```text
IMPORT_CONFIRM
```

### Must Include

- actor_id
- batch_id
- file_type
- row_count
- warning_count
- error_count
- confirm policy

---

# 7. Inquiry Snapshot Rules

## BR-INQ-001 — Inquiry Is Snapshot

### Rule

Inquiry คือยอดคงเหลือ ณ วัน/เวลา ไม่ใช่ movement เข้า/ออก

### Prohibited

ห้ามใช้ Inquiry เป็นรายการบวก/ลบ stock โดยตรง

### Correct Usage

```text
Inquiry = Anchor Snapshot
Stock Replay = Anchor + Movement Ledger
```

---

## BR-INQ-002 — Active Snapshot Policy Required

### Rule

ถ้า business_date เดียวกันมี Inquiry มากกว่า 1 batch ต้องมี policy ว่าตัวไหน active

### Allowed Policies

```text
REPLACE_OLD_WITH_NEW
ARCHIVE_OLD_USE_NEW
KEEP_BOTH_MANUAL_ACTIVE
CANCEL_NEW_UPLOAD
```

### Prohibited

```text
Append ซ้ำแบบเงียบ ๆ โดยไม่บอกว่า active snapshot ไหน
```

---

## BR-INQ-003 — Snapshot Must Have Checksum

### Rule

Inquiry snapshot ต้องมี checksum เพื่อจับซ้ำ

### Purpose

- ป้องกัน upload ซ้ำ
- ตรวจ version
- trace file ที่ใช้ปิดวัน

---

## BR-INQ-004 — Snapshot Is Anchor for Replay

### Rule

Stock Replay ต้องเลือก Anchor Snapshot ที่เหมาะสมที่สุดก่อนช่วงวันที่ replay

### Example

```text
Replay วันที่ 10–15
ต้องหา Inquiry ล่าสุดก่อนหรือเท่ากับวันที่ 10
แล้วใช้ In-Out วันที่ 11–15 replay ต่อ
```

---

# 8. System In-Out Rules

## BR-SIO-001 — System In-Out Is ERP Posted Movement

### Rule

System In-Out คือ movement ที่ ERP post แล้ว

### Usage

- สร้าง system_inout_records
- สร้าง movement_ledger source_type = SYSTEM_INOUT
- ใช้เทียบ Field Movement
- ใช้ Stock Replay

---

## BR-SIO-002 — In-Out Backfill Must Be Supported

### Rule

ระบบต้องรองรับการ upload In-Out ย้อนหลัง 1–5 วันหรือมากกว่า

### Reason

Admin อาจลา/หยุด และ Inquiry อาจไม่ได้ upload ทุกวัน

---

## BR-SIO-003 — Movement Type Must Be Normalized

### Rule

movement_type จาก ERP ต้อง normalize เป็น canonical movement type

### Canonical Types

```text
IN
OUT
TRANSFER
REJ
ADJUSTMENT
RETURN
UNKNOWN
```

### Unknown Rule

ถ้า movement type ไม่รู้จัก ต้องเข้า mapping/review ไม่ควรเดาเอง

---

## BR-SIO-004 — Qty Sign Rule Must Be Explicit

### Rule

ระบบต้องรู้ว่า qty จาก In-Out เป็น positive/negative หรือใช้ movement_type แยก sign

### Possible Patterns

```text
Pattern A: Qty positive + movement_type tells direction
Pattern B: Qty negative for OUT
Pattern C: Transfer has from/to with positive qty
```

### Requirement

ต้องได้จาก Data Profiling และเก็บไว้ใน config/mapping

---

# 9. Field Movement Rules

## BR-FM-001 — Field Movement Is Not ERP Post

### Rule

Field Movement คือสิ่งที่หน้างานแจ้ง ไม่ใช่หลักฐานว่า ERP post แล้ว

### Status After Create

```text
WAIT_SYSTEM_POST
```

จนกว่า System In-Out จะ match

---

## BR-FM-002 — Every Stock-Effect Field Movement Must Enter Ledger

### Rule

movement ที่มีผลต่อ stock ต้องสร้าง movement_ledger

### Applies To

- IN
- OUT
- TRANSFER
- REJ
- ADJUSTMENT
- PENDING_INBOUND_CONFIRM

---

## BR-FM-003 — Locked Day Cannot Accept Direct Movement

### Rule

ถ้าวันนั้น status = LOCKED ห้ามเพิ่ม/แก้ Field Movement โดยตรง

### Required Flow

```text
Create Correction / Investigation
→ Approve
→ Apply adjustment with audit
```

---

## BR-FM-004 — Movement Qty Must Be Positive

### Rule

ใน Field Movement ให้ qty เป็น positive แล้วใช้ movement_type บอกทิศทาง

### Reason

ลด confusion จากการกรอกติดลบผิด

---

# 10. Pending Inbound Rules

## BR-PI-001 — Pending Inbound Allowed for Unknown Inbound Only

### Rule

ใช้ Pending Inbound เมื่อของเข้าแล้วแต่ข้อมูลยังไม่ครบจริง

### Example

- ยังไม่รู้ customer
- ยังไม่รู้ item เต็ม
- รู้แค่ label text เช่น MKMK W150 69
- รอ In-Out วันถัดไปยืนยัน

---

## BR-PI-002 — Pending Inbound Must Keep Evidence

### Rule

Pending Inbound ควรมีอย่างน้อย:

- text_on_label
- location
- qty
- uom
- reported_by
- business_date

และแนะนำให้มี photo_url

---

## BR-PI-003 — Pending Inbound Blocks Final Close if Unresolved

### Rule

Pending Inbound ที่ยังไม่ confirm ควร block FINAL_CLOSED หรืออย่างน้อยทำให้เป็น READY_WITH_WARNING ตาม policy

### Recommended MVP

```text
Block FINAL_CLOSED if pending inbound has qty and affects stock
Allow PRELIM_CLOSED with warning
```

---

## BR-PI-004 — Candidate Match Requires Review if Confidence Not High

### Rule

ถ้า candidate match จาก In-Out confidence ต่ำ/กลาง ต้องให้ Admin/Supervisor confirm

### Low Confidence Prohibited Action

ห้าม convert เป็น canonical movement อัตโนมัติ

---

# 11. Alias Rules

## BR-ALIAS-001 — Alias Must Be Typed

### Rule

Alias ต้องแยก type

### Types

```text
CUSTOMER_ALIAS
ITEM_ALIAS
LOCATION_ALIAS
DC_ALIAS
WAREHOUSE_ALIAS
UOM_ALIAS
LOT_PATTERN
```

### Prohibited

ห้ามเอา MKMK, W150, DC08, BAG ใส่ alias table เดียวแล้วไม่ระบุ type

---

## BR-ALIAS-002 — Unknown Alias Must Enter Review Queue

### Rule

ถ้าระบบเจอข้อความที่ map master ไม่ได้ ต้องสร้าง alias_review_queue

### Required Data

- alias_type
- raw_text
- source_type
- source_batch_id
- source_row_no
- confidence
- status = PENDING

---

## BR-ALIAS-003 — Low Confidence Cannot Auto-Map

### Rule

ถ้า confidence ต่ำกว่า threshold ห้าม map อัตโนมัติ

### Suggested Threshold

```text
>= 90 auto-suggest only, still review if critical
70–89 manual review recommended
< 70 manual review required
```

### MVP Recommendation

แม้ high confidence ก็ให้ Admin confirm ในช่วงแรกจนกว่า alias seed นิ่ง

---

## BR-ALIAS-004 — Sugar Pattern Is Not Universal

### Rule

Pattern เช่น MKMK W150 69 ใช้กับน้ำตาลได้ แต่ห้ามบังคับทุก product group ใช้ pattern นี้

### Product Groups Need Different Identity

- Sugar
- Wood
- Flour
- Molasses
- Bulk / เทกอง
- Other

---

## BR-ALIAS-005 — Alias Mapping Must Be Audited

### Rule

การ map/create/reject alias ต้องมี audit

### Events

- ALIAS_MAP
- ALIAS_CREATE_MASTER
- ALIAS_REJECT
- ALIAS_UPDATE

---

# 12. Movement Ledger Rules

## BR-LEDGER-001 — Ledger Is Source of Stock Effect

### Rule

Movement Ledger คือแหล่งรวม movement ที่มีผลต่อ stock สำหรับ replay

### Sources

```text
SYSTEM_INOUT
FIELD_NOTICE
REJ_NOTICE
TRANSFER
ADJUSTMENT
COUNT_CORRECTION
PENDING_INBOUND_CONFIRM
```

---

## BR-LEDGER-002 — Ledger Must Trace Source

### Rule

ledger ทุก row ต้องมี source_type และ source_ref_id

### Reason

ถ้า replay ผิด ต้องย้อนกลับได้ว่าเกิดจาก record ไหน

---

## BR-LEDGER-003 — Ledger Correction Must Not Overwrite Original Silently

### Rule

ถ้าต้องแก้ ledger หลังใช้งานแล้ว ห้ามแก้ค่าเดิมเงียบ ๆ

### Required Flow

```text
Cancel/Supersede original ledger
→ Add correction ledger
→ Audit reason
```

---

# 13. Stock Replay Rules

## BR-SR-001 — Stock Replay Uses Anchor + Ledger

### Rule

Stock Replay ต้องใช้:

```text
Anchor Snapshot + Movement Ledger = Daily Stock Position
```

---

## BR-SR-002 — Missing Anchor Downgrades or Blocks Replay

### Rule

ถ้าไม่มี Anchor Snapshot ก่อนช่วงวันที่ replay ต้องให้ status = MISSING_ANCHOR

### Policy

```text
Critical: block final close
Warning: allow estimated if admin accepts risk
```

### MVP Recommendation

Block final close ถ้าไม่มี anchor และไม่มี count confirmed

---

## BR-SR-003 — Missing In-Out Downgrades Confidence

### Rule

ถ้า In-Out ขาดบางวัน ต้อง downgrade confidence ของ Daily Stock Position

### Example

```text
In-Out missing 1 day in replay range
→ confidence = LOW
→ close readiness warning/critical
```

---

## BR-SR-004 — Replay Must Have Job Log

### Rule

ทุกการ run Stock Replay ต้องมี stock_replay_jobs log

### Required Fields

- date_from
- date_to
- warehouse_id
- triggered_by
- status
- rows_processed
- warning_count
- error_count

---

# 14. Reconciliation Rules

## BR-REC-001 — Reconciliation Must Compare Multiple Truth Layers

### Rule

Reconciliation ไม่ใช่แค่เทียบ qty ERP กับ count อย่างเดียว

### Compare Pairs

```text
Field Movement vs System In-Out
Expected Stock vs Physical Count
REJ Notice vs ERP Movement
Pending Inbound vs In-Out
Anchor vs Replayed Stock
```

---

## BR-REC-002 — Matching Key Must Be Configurable

### Rule

Matching key ต้องปรับได้หลัง Data Profiling

### Candidate Matching Fields

- business_date
- movement_date
- warehouse/DC
- location
- customer
- item
- year
- lot
- qty
- uom
- source_ref
- time window

### Reason

ไฟล์จริงอาจไม่มีบาง field หรือใช้ field คนละชื่อ

---

## BR-REC-003 — Result Code Required

### Rule

ทุก reconciliation result ต้องมี result_code

### Core Result Codes

```text
MATCHED
SYSTEM_MISSING
FIELD_MISSING
QTY_DIFF
LOCATION_DIFF
ITEM_DIFF
LOT_DIFF
REJ_NOT_POSTED
REJ_QTY_DIFF
UNKNOWN_ALIAS
PENDING_INBOUND
NEED_RECOUNT
NEED_INVESTIGATION
```

---

## BR-REC-004 — Explanation Required for Non-Matched

### Rule

ถ้า result_code ไม่ใช่ MATCHED ต้องมี explanation

### Explanation Must Include

- ระบบเทียบอะไรกับอะไร
- ค่าที่ต่างคืออะไร
- likely cause คืออะไร
- suggested action คืออะไร

---

## BR-REC-005 — Low Coverage Cannot Produce High Confidence

### Rule

ถ้า coverage ไม่ครบ ห้ามตั้ง confidence = HIGH

### Examples

- Missing In-Out
- Missing Anchor
- Unknown Alias
- Pending Inbound
- No Count for moving location

---

## BR-REC-006 — Critical Result Must Create Issue

### Rule

Result ที่ severity = CRITICAL ต้องสร้าง issue อัตโนมัติ

### Examples

- REJ_NOT_POSTED
- SYSTEM_MISSING high qty
- MISSING_ANCHOR close day
- UNKNOWN_ALIAS blocking normalize
- QTY_DIFF large

---

# 15. Issue Rules

## BR-ISS-001 — Issue Must Have Owner

### Rule

Issue ทุกตัวต้องมี owner_role

### Owner Mapping

| Result Code | Owner Role |
|---|---|
| SYSTEM_MISSING | ERP_ADMIN |
| FIELD_MISSING | SUPERVISOR |
| QTY_DIFF | SUPERVISOR |
| LOCATION_DIFF | CHECKER / SUPERVISOR |
| REJ_NOT_POSTED | ERP_ADMIN |
| UNKNOWN_ALIAS | ADMIN |
| PENDING_INBOUND | ADMIN / SUPERVISOR |
| NEED_RECOUNT | CHECKER / SUPERVISOR |

---

## BR-ISS-002 — Issue Must Have Status

### Rule

Issue lifecycle ต้องใช้ status

### Statuses

```text
OPEN
ASSIGNED
IN_REVIEW
WAIT_ERP_ADMIN
WAIT_CHECKER_COUNT
RESOLVED
APPROVED
CLOSED
REOPENED
```

---

## BR-ISS-003 — Closing Issue Requires Resolution

### Rule

การ close issue ต้องมี close_reason หรือ resolution_code

### Reason

ป้องกันปิดปัญหาโดยไม่มีหลักฐาน

---

## BR-ISS-004 — Open Issue Blocks Retention Delete

### Rule

issue ที่ status ยังไม่ closed ห้ามถูกลบหรือทำให้ source หายจนดู detail ไม่ได้

---

## BR-ISS-005 — Accept Difference Requires Reason

### Rule

ถ้าต้อง accept difference โดยไม่แก้ตัวเลข ต้องบันทึก reason และ approver

### Audit Required

```text
ISSUE_ACCEPT_DIFF
```

---

# 16. Physical Count Rules

## BR-PC-001 — Count Must Have Scope

### Rule

Physical Count ต้องบอก scope ว่านับอะไร

### Required Scope

- business_date
- warehouse
- location
- qty
- count_type
- counted_by

Optional but recommended:

- customer
- item
- year
- lot
- photo

---

## BR-PC-002 — Checker Must Have Active Session

### Rule

Checker ต้องมี active checker_daily_session ที่ warehouse นั้นก่อน submit count

---

## BR-PC-003 — Count After Locked Day Requires Correction Flow

### Rule

ถ้าวันถูก LOCKED แล้ว ห้ามเพิ่ม/แก้ count ตรง

### Required Flow

```text
Correction Request
→ Approval
→ New count/correction record
→ Audit
```

---

## BR-PC-004 — Recount Task Should Link Issue

### Rule

ถ้า count เกิดจาก issue ต้อง link task_id หรือ issue_id

---

# 17. REJ Rules

## BR-REJ-001 — REJ Must Have Reason

### Rule

ทุก REJ ต้องมี reason_id หรือ reason_text

---

## BR-REJ-002 — REJ Must Have From and REJ Location

### Rule

ต้องรู้ว่า REJ มาจาก location ไหน และไป REJ location ไหน

---

## BR-REJ-003 — REJ Must Wait ERP Post

### Rule

หลังแจ้ง REJ ให้ status = WAIT_SYSTEM_POST จนกว่า System In-Out พบรายการที่ match

---

## BR-REJ-004 — REJ Not Posted Must Notify ERP Admin

### Rule

ถ้าไม่พบ ERP movement ภายใน policy window ให้สร้าง issue REJ_NOT_POSTED และแจ้ง ERP Admin

### Suggested Window

```text
Same day or next business day depending ERP process
```

---

# 18. User / Permission Rules

## BR-USER-001 — User Must Be Approved Before Login

### Rule

User ที่สมัครใหม่ต้องรอ Admin อนุมัติก่อนใช้งาน

---

## BR-USER-002 — Reject Request Deletes Queue but Keeps Audit

### Rule

ถ้า Admin reject user request สามารถลบจาก pending queue ได้ แต่ต้องมี audit log

---

## BR-PERM-001 — Backend Permission Check Required

### Rule

ทุก API สำคัญต้องตรวจ permission backend

### Important

Frontend hide menu ไม่ถือว่าปลอดภัยพอ

---

## BR-PERM-002 — Warehouse Scope Required

### Rule

ทุก action ที่เกี่ยวกับ warehouse ต้องตรวจ warehouse scope

### Applies To

- import
- count
- movement
- issue view
- export
- close day

---

## BR-PERM-003 — Password Must Be Hash + Salt

### Rule

ห้ามเก็บ plain password

---

# 19. Checker GPS Session Rules

## BR-GPS-001 — One Active Warehouse per Checker per Day

### Rule

Checker 1 คนมี active warehouse ได้ 1 ที่ต่อ business_date

### If Check-in New Warehouse

```text
Suspend old session
Create new active session
Audit both actions
```

---

## BR-GPS-002 — GPS Accuracy Must Be Stored

### Rule

ถ้าใช้ GPS ต้องเก็บ lat/lng/accuracy

### Note

GPS ใช้เป็น control เบื้องต้น ไม่ใช่ proof absolute เพราะ spoof ได้

---

## BR-GPS-003 — Count Requires Active Session

### Rule

ถ้าไม่มี active session ห้าม submit count

---

# 20. Export Rules

## BR-EXP-001 — Every Export Must Have Log

### Rule

ทุก export ต้องมี export_jobs/export_logs

### Required Metadata

- export_type
- filters_json
- selected_ids
- row_count
- exported_by
- exported_at
- file_format

---

## BR-EXP-002 — Export Requires Permission

### Rule

export ต้องตรวจ CAN_EXPORT หรือ permission เฉพาะ export type

### Example

```text
ERP Template → CAN_EXPORT_ERP_TEMPLATE
Audit Export → CAN_VIEW_AUDIT_LOG + CAN_EXPORT
```

---

## BR-EXP-003 — Export Should Use Result/Summary Tables

### Rule

Export report ทั่วไปควรอ่านจาก result/summary ไม่ใช่ raw full history

### Exception

Audit/source export อาจอ่าน source table โดยตรงตาม permission

---

# 21. Notification Rules

## BR-NOTI-001 — Notification Must Use Rules/Template

### Rule

ห้าม hardcode notification กระจัดกระจายใน service

### Required Components

- notification_channels
- notification_rules
- notification_templates
- notification_logs

---

## BR-NOTI-002 — Cooldown Required

### Rule

Notification ต้องมี cooldown หรือ send_once_per_issue เพื่อป้องกัน spam

---

## BR-NOTI-003 — Critical Alert Sends Immediately

### Rule

Critical event ควรส่งทันทีถ้า rule active

### Examples

- REJ_NOT_POSTED
- RETENTION_FAILED
- MISSING_ANCHOR before close
- DB_USAGE_WARNING critical

---

## BR-NOTI-004 — Low Priority Can Be Digest

### Rule

Low/Medium event สามารถรวมเป็น daily digest ได้

---

# 22. Close / Lock Rules

## BR-CLOSE-001 — Close Day Requires Readiness Gate

### Rule

ก่อน close day ต้อง run readiness gate

### Gate Checks

- open critical issues
- pending alias
- pending inbound
- missing anchor
- missing In-Out
- REJ_NOT_POSTED
- pending export if required
- failed reconciliation job

---

## BR-CLOSE-002 — Ready Status

### Values

```text
READY
READY_WITH_WARNING
NOT_READY
```

### Meaning

| Status | Meaning |
|---|---|
| READY | ปิดได้ |
| READY_WITH_WARNING | ปิดเบื้องต้นได้ แต่มี warning |
| NOT_READY | ห้าม final close |

---

## BR-CLOSE-003 — Lock Day Blocks Direct Edit

### Rule

ถ้า day status = LOCKED ห้ามแก้ import/movement/count/recon โดยตรง

### Required Flow

```text
Investigation / Correction
→ approval
→ correction record
→ audit
```

---

## BR-CLOSE-004 — Unlock Requires Admin and Reason

### Rule

Unlock day ต้องเป็น Admin และต้องใส่ reason

### Audit Event

```text
DAILY_UNLOCK
```

---

# 23. Retention Rules

## BR-RET-001 — Active Raw Data Retention 45 Days

### Rule

Raw/staging data เก็บ active 45 วัน แล้ว archive/delete ตาม policy

---

## BR-RET-002 — Archive Summary Before Delete

### Rule

ก่อนลบ raw/detail ต้องสร้าง summary/archive ที่จำเป็น

---

## BR-RET-003 — Do Not Delete Open Issue Data

### Rule

ห้ามลบข้อมูลที่ยังผูกกับ open issue หรือ pending action

### Do Not Delete

- open issue
- pending approval
- pending export
- REJ_NOT_POSTED
- SYSTEM_MISSING
- correction log
- export log
- daily close summary
- critical audit log

---

## BR-RET-004 — Retention Must Have Dry Run

### Rule

Manual retention ควรมี DRY_RUN เพื่อดูว่าจะลบ/ข้ามอะไร

---

## BR-RET-005 — Retention Must Notify Admin

### Rule

หลัง retention job ต้องแจ้ง Admin summary โดยเฉพาะถ้า failed/warning

---

# 24. Dashboard / KPI Rules

## BR-DASH-001 — Dashboard Reads Summary/Result

### Rule

Control Center/Dashboard ต้องอ่านจาก summary/result tables

### Allowed Sources

- daily_recon_summary
- daily_issue_summary
- recon_results
- issues
- import_summary
- retention_logs
- system_health_checks

### Prohibited

```text
Query raw rows/full ledger ทุกครั้งที่เปิด dashboard
```

---

## BR-DASH-002 — KPI Must Have Decision Purpose

### Rule

KPI ทุกตัวต้องตอบว่า user ควรทำอะไรต่อ

### Example

| KPI | Decision |
|---|---|
| Unknown Alias | Admin ต้อง map alias |
| REJ Not Posted | ERP Admin ต้องตรวจ post |
| Close Readiness | Supervisor/Admin ตัดสินใจปิดวัน |
| DB Usage | Admin ตรวจ retention/limit |

---

## BR-DASH-003 — KPI Must Have Context

### Rule

ตัวเลข KPI ต้องมี context

Required context:

- business_date
- warehouse
- last_updated
- filters used

---

# 25. Performance Rules

## BR-PERF-001 — Large Import Uses Chunk Upload

### Rule

ไฟล์ใหญ่ต้อง upload/process เป็น chunk

---

## BR-PERF-002 — Heavy Jobs Run Async

### Applies To

- normalize batch
- stock replay
- reconciliation
- export large report
- retention

---

## BR-PERF-003 — Use Lookup Map for Matching

### Rule

Reconciliation/alias matching ต้องใช้ indexed lookup ไม่ loop raw table ซ้ำแบบ O(n²) เมื่อข้อมูลเยอะ

---

## BR-PERF-004 — Mobile Must Not Load Full Dataset

### Rule

Checker mobile ต้องโหลดเฉพาะ task/data ที่จำเป็น

---

# 26. Future Simulation Rules

These rules are future phase, not full MVP.

## BR-SIM-001 — Simulation Must Not Move Stock Automatically

### Rule

Simulation เสนอแผนได้ แต่ห้ามเปลี่ยน stock/movement จริงโดยอัตโนมัติ

## BR-SIM-002 — Capacity Policy Must Be Explicit

### Rule

Available Location simulation ต้องระบุ policy เช่น:

- allow multi-lot
- ignore lot for external customer
- allow same customer merge
- capacity threshold

## BR-SIM-003 — Relocation Suggestion Requires Approval

### Rule

การย้ายของตาม suggestion ต้องผ่าน approval ก่อนสร้าง movement จริง

---

# 27. Rule Conflict Resolution

เมื่อกฎขัดกัน ให้ยึด priority นี้:

```text
1. Security / Permission Rules
2. Audit / Compliance Rules
3. Data Truth Rules
4. Close / Lock Rules
5. Reconciliation Rules
6. UX Convenience Rules
7. Performance Optimization Rules
```

ตัวอย่าง:

```text
UX อยากให้แก้เร็ว แต่ day LOCKED แล้ว
→ ต้องยึด Lock Rule และ Correction Flow
```

```text
อยากลบข้อมูลเพราะเกิน 45 วัน แต่ issue ยัง open
→ ต้องยึด Retention Skip Open Issue
```

---

# 28. Business Rule Acceptance Criteria

Business Rules ถือว่า complete ถ้า:

- [ ] Inquiry ถูกบังคับเป็น Snapshot ไม่ใช่ Transaction
- [ ] Import ต้องผ่าน staging/preview/mapping ก่อน confirm
- [ ] Field Movement แยกจาก System In-Out
- [ ] Unknown Alias เข้า review queue
- [ ] Pending Inbound มี workflow confirm
- [ ] Movement Ledger รวมทุก stock-effect movement
- [ ] Stock Replay ใช้ Anchor + Ledger
- [ ] Reconciliation มี result_code/severity/confidence/explanation
- [ ] Issue มี owner/status/action
- [ ] Low confidence ไม่ auto-close/auto-map
- [ ] Checker ต้อง check-in ก่อน count
- [ ] Export มี log
- [ ] Notification มี cooldown/send once
- [ ] Close day มี readiness gate
- [ ] Locked day ห้ามแก้ตรง
- [ ] Retention 45 วันไม่ลบ open issue/critical audit
- [ ] Dashboard อ่าน summary/result ไม่อ่าน raw full history
- [ ] ทุก critical write มี audit log

---

# 29. Final Business Rules Statement

```text
Business Rules ของ Warehouse Reconciliation ERP Platform ต้องทำให้ระบบแยกความจริงของข้อมูลแต่ละแหล่ง ควบคุมการนำเข้าข้อมูล ตรวจ alias คำนวณ stock จาก Anchor + Ledger ตรวจความไม่ตรงด้วย result ที่อธิบายได้ สร้าง issue ให้คนรับผิดชอบ ปิดวันด้วย readiness gate และลบ/archive ข้อมูลอย่างปลอดภัย โดยทุก action สำคัญต้องมี permission, validation และ audit log
```

