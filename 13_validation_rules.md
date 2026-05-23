# 13_validation_rules.md

# Validation Rules Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Validation Rules / Data Quality / Input Control Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / QA / System Analyst Use  
**Primary Use:** ใช้กำหนดกฎตรวจสอบข้อมูลก่อนบันทึก ก่อน normalize ก่อนคำนวณ ก่อน reconcile ก่อน export ก่อน close day และก่อน retention เพื่อให้ระบบไม่รับข้อมูลผิดเข้าไปสร้างผลลัพธ์ผิดทั้งระบบ

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Validation Rules ของ Warehouse Reconciliation ERP Platform

ระบบนี้มีข้อมูลจากหลายแหล่ง:

```text
User input
Excel / CSV upload
ERP Inquiry
ERP System In-Out
Field Movement
Physical Count
REJ Notice
Alias Mapping
Export Request
Notification Config
Retention Job
```

ถ้ารับข้อมูลผิดตั้งแต่ต้น ระบบจะคำนวณผิดตามไปทั้งหมด เช่น:

- column map ผิด → item/location/qty ผิด
- alias ผิด → reconcile ผิด
- Inquiry ซ้ำ → stock เบิ้ล
- In-Out ขาดวัน → replay ผิด
- Count ไม่มี scope → เทียบ expected ไม่ได้
- user ไม่มีสิทธิ์แต่บันทึกได้ → audit/security พัง
- retention ลบ source ของ open issue → ตรวจย้อนหลังไม่ได้

เป้าหมายของ validation คือ:

1. ป้องกันข้อมูลผิดก่อนเข้าระบบ
2. ตรวจความครบของข้อมูลก่อนใช้คำนวณ
3. บอก user ชัดเจนว่าผิดอะไร
4. แยก error ที่ block action กับ warning ที่ให้ confirm ได้
5. ทำให้ระบบตัดสินใจได้โดยไม่ต้องให้คนไล่เช็คเองทุกแถว
6. สร้าง data quality signal ให้ dashboard/close readiness/reconciliation ใช้ต่อ

หลักสำคัญ:

```text
Validation ไม่ใช่แค่ required field
Validation คือระบบควบคุมคุณภาพข้อมูลก่อนตัดสินใจ
```

---

## 2. Validation Philosophy

## 2.1 Validate at Multiple Layers

ระบบต้องตรวจหลายชั้น:

```text
Frontend Validation
→ Backend Validation
→ Data Quality Validation
→ Business Rule Validation
→ Reconciliation Readiness Validation
→ Close / Retention Validation
```

Frontend validation ช่วย UX แต่ไม่ใช่ความปลอดภัยหลัก

Backend validation คือ source of truth

---

## 2.2 Fail Fast, Explain Clearly

ถ้าข้อมูลผิดแบบ critical ต้อง block ทันที และบอก user ชัดเจน

ตัวอย่าง:

```text
ไม่สามารถ confirm import ได้ เพราะยังไม่ได้ map column qty
```

ไม่ควรบอกแค่:

```text
Error
```

---

## 2.3 Warning Is Not Error

ต้องแยก:

| Level | Meaning | Action |
|---|---|---|
| INFO | ข้อมูลประกอบ | ไม่ block |
| WARNING | เสี่ยง แต่ไปต่อได้ถ้ามีเหตุผล | require confirm / downgrade confidence |
| ERROR | ผิด ต้องแก้ก่อน | block action |
| CRITICAL | เสี่ยงกระทบ stock/security/audit | block action + notify/admin |

---

## 2.4 Validation Must Produce Machine-Readable Code

ทุก validation fail ต้องมี code สำหรับ frontend / log / test

ตัวอย่าง:

```json
{
  "code": "MISSING_REQUIRED_MAPPING",
  "message": "ยังไม่ได้ map column ที่จำเป็น",
  "details": {
    "missing_fields": ["qty", "business_date"]
  }
}
```

---

## 2.5 Validation Must Be Auditable When Critical

Validation fail ทั่วไปไม่จำเป็นต้อง audit ทุกครั้ง แต่ critical action ที่ถูก block หรือ override ต้องมี log/audit

ตัวอย่างที่ควร log:

- duplicate Inquiry แต่ user override
- close day แบบ ready with warning
- accept diff
- retention skipped open issue
- locked day edit attempt
- permission denied on critical action

---

# 3. Validation Categories

Validation Rules แบ่งเป็นกลุ่มหลัก:

```text
1. Common Field Validation
2. Authentication & User Validation
3. Permission & Scope Validation
4. Checker Session Validation
5. Import File Validation
6. Column Mapping Validation
7. Raw Row / Normalize Validation
8. Alias Validation
9. Master Data Validation
10. Inquiry Snapshot Validation
11. System In-Out Validation
12. Field Movement Validation
13. Pending Inbound Validation
14. Physical Count Validation
15. REJ Validation
16. Movement Ledger Validation
17. Stock Replay Validation
18. Reconciliation Validation
19. Issue / Task Validation
20. Export Validation
21. Notification Validation
22. Close / Lock Validation
23. Retention Validation
24. Dashboard / Summary Validation
25. Performance / Job Validation
```

---

# 4. Common Field Validation

## VAL-COM-001 — Required Field

### Rule

field ที่จำเป็นต้องมีค่าและไม่เป็น empty string/null/undefined

### Applies To

- username
- business_date
- warehouse_id
- location_id where required
- qty
- file_type
- batch_id
- result_code
- issue_id

### Error Code

```text
REQUIRED_FIELD_MISSING
```

### Severity

ERROR

---

## VAL-COM-002 — Date Format

### Rule

date ต้อง parse ได้เป็น date ที่ถูกต้อง และไม่สลับ พ.ศ./ค.ศ. โดยไม่รู้ตัว

### Accepted Internal Format

```text
YYYY-MM-DD
```

### Must Detect

- `17/05/2026`
- `05/17/2026`
- `2569-05-17`
- Excel serial date
- blank date

### Error Code

```text
INVALID_DATE
```

### Warning Code

```text
DATE_FORMAT_AMBIGUOUS
```

### Severity

ERROR หรือ WARNING ตามความมั่นใจ

---

## VAL-COM-003 — Date Range

### Rule

date_from ต้องไม่มากกว่า date_to

### Applies To

- Backfill Wizard
- Import In-Out date range
- Reconciliation run
- Export filters
- Retention dry run

### Error Code

```text
INVALID_DATE_RANGE
```

---

## VAL-COM-004 — Quantity Numeric

### Rule

qty ต้องเป็นตัวเลขที่ parse ได้

### Must Handle

- comma separator เช่น `1,000`
- decimal เช่น `1000.50`
- empty value
- text mixed เช่น `1,000 BAG`
- negative sign

### Internal Rule

Field Movement / Count / REJ ควรเก็บ qty เป็น positive แล้วใช้ movement_type บอกทิศทาง

### Error Code

```text
INVALID_QTY
```

---

## VAL-COM-005 — Quantity Range

### Rule

qty ต้องอยู่ในช่วงที่สมเหตุสมผล

### Basic Rule

```text
qty > 0 for movement
qty >= 0 for physical count
```

### Warning Rule

ถ้า qty สูงผิดปกติเมื่อเทียบกับ capacity/history ให้ warning

### Error Codes

```text
QTY_MUST_BE_POSITIVE
QTY_OUT_OF_REASONABLE_RANGE
```

---

## VAL-COM-006 — UOM Valid

### Rule

UOM ต้อง map ได้เป็น canonical UOM หรือเข้า UOM Alias Review

### Examples

```text
BAG
KG
TON
ถุง
ตัน
```

### Error / Warning

- ถ้า UOM required และไม่รู้จัก → ERROR
- ถ้ามี default UOM จาก item และ UOM missing → WARNING

### Codes

```text
UNKNOWN_UOM
MISSING_UOM_WITH_DEFAULT_ASSUMED
```

---

## VAL-COM-007 — ID Exists

### Rule

foreign key ที่ส่งเข้ามาต้องมีอยู่จริงและ active ถ้าต้องใช้งาน

### Applies To

- warehouse_id
- location_id
- customer_id
- item_id
- role_id
- permission_id
- issue_id

### Error Code

```text
REFERENCE_NOT_FOUND
```

---

## VAL-COM-008 — Status Transition Valid

### Rule

ห้ามเปลี่ยน status ข้ามขั้นผิด flow

### Example

```text
Issue OPEN → CLOSED allowed with reason
Issue CLOSED → ASSIGNED not allowed unless REOPENED first
Import UPLOADED → CONFIRMED not allowed without MAPPED/NORMALIZED
```

### Error Code

```text
INVALID_STATUS_TRANSITION
```

---

# 5. Authentication & User Validation

## VAL-AUTH-001 — Username Required and Unique

### Rule

username ต้องมีและไม่ซ้ำกับ users หรือ pending requests

### Error Codes

```text
USERNAME_REQUIRED
USERNAME_ALREADY_EXISTS
USERNAME_PENDING_APPROVAL_EXISTS
```

---

## VAL-AUTH-002 — Password Required and Policy

### Rule

password ต้องมี และผ่าน policy ขั้นต่ำ

### Recommended MVP Policy

- length >= 8
- not empty
- confirm password match

### Production Recommended

- length >= 10
- มีตัวอักษรและตัวเลข
- rate limit login fail

### Error Codes

```text
PASSWORD_REQUIRED
PASSWORD_TOO_SHORT
PASSWORD_CONFIRM_NOT_MATCH
```

---

## VAL-AUTH-003 — Password Must Be Hashed Before Save

### Rule

ห้ามบันทึก plain password ลง database

### Error Code

```text
PASSWORD_HASH_REQUIRED
```

### Severity

CRITICAL

---

## VAL-AUTH-004 — User Must Be Active to Login

### Rule

เฉพาะ user status = ACTIVE เท่านั้นที่ login ใช้งานจริงได้

### Block Status

```text
PENDING_APPROVAL
LOCKED
SUSPENDED
EXPIRED
DELETED
```

### Error Codes

```text
PENDING_APPROVAL
USER_LOCKED
USER_SUSPENDED
USER_EXPIRED
USER_NOT_ACTIVE
```

---

## VAL-AUTH-005 — Approve User Request Validation

### Rule

ก่อน approve user ต้องตรวจ:

- request exists
- request status = PENDING_APPROVAL
- approver has CAN_MANAGE_USER
- role/permission selected
- warehouse scope selected if operation role

### Error Codes

```text
USER_REQUEST_NOT_FOUND
USER_REQUEST_NOT_PENDING
ROLE_OR_PERMISSION_REQUIRED
WAREHOUSE_SCOPE_REQUIRED
```

---

## VAL-AUTH-006 — Reject User Request Validation

### Rule

ก่อน reject ต้องตรวจ:

- request exists
- status = PENDING_APPROVAL
- actor has CAN_MANAGE_USER
- reason required if config enabled

### Critical Rule

ต้องเขียน audit ก่อน/พร้อมการลบ request

### Error Code

```text
REJECT_REASON_REQUIRED
AUDIT_REQUIRED_BEFORE_DELETE
```

---

# 6. Permission & Scope Validation

## VAL-PERM-001 — Backend Permission Required

### Rule

ทุก API สำคัญต้องตรวจ permission ที่ backend

### Applies To

- import
- confirm import
- map alias
- create movement
- count
- run replay
- run reconciliation
- close issue
- export
- lock day
- retention

### Error Code

```text
PERMISSION_DENIED
```

### Severity

CRITICAL for write actions

---

## VAL-PERM-002 — Warehouse Scope Required

### Rule

ทุก action ที่เกี่ยวกับ warehouse ต้องตรวจว่า user มี scope

### Applies To

- view issues
- import warehouse data
- count
- movement
- close day
- export warehouse report

### Error Code

```text
WAREHOUSE_SCOPE_DENIED
```

---

## VAL-PERM-003 — Permission Override Conflict

### Rule

ถ้า user มี override ALLOW และ DENY สำหรับ permission เดียวกัน ต้องมี priority ชัดเจน

### Recommended Rule

```text
DENY wins over ALLOW
```

### Error/Warning Code

```text
PERMISSION_OVERRIDE_CONFLICT
```

---

# 7. Checker Session Validation

## VAL-CS-001 — Checker Must Check In Before Count

### Rule

Checker ต้องมี active checker_daily_session ก่อนนับหรือทำ task ที่ warehouse นั้น

### Error Code

```text
CHECKER_SESSION_REQUIRED
```

---

## VAL-CS-002 — One Active Warehouse per Day

### Rule

checker 1 คนมี active warehouse ได้ 1 แห่งต่อ business_date

### If Check-in New Warehouse

ระบบต้อง suspend session เดิมก่อนสร้าง session ใหม่

### Warning Message

```text
การ check-in ที่คลังใหม่จะพักสิทธิ์ของคลังเดิม
```

---

## VAL-CS-003 — GPS Required if Enabled

### Rule

ถ้า config เปิด GPS required ต้องมี lat/lng/accuracy

### Error Codes

```text
GPS_REQUIRED
GPS_INVALID
GPS_ACCURACY_TOO_LOW
```

### Note

GPS เป็น control เบื้องต้น ไม่ใช่ proof absolute

---

# 8. Import File Validation

## VAL-IMP-001 — File Type Allowed

### Rule

รองรับเฉพาะ file type ที่กำหนด

### Allowed Types

```text
xlsx
csv
```

### Import Types

```text
INQUIRY
SYSTEM_INOUT
FIELD_COUNT
MASTER_DATA
```

### Error Code

```text
INVALID_FILE_TYPE
```

---

## VAL-IMP-002 — File Not Empty

### Rule

file ต้องมี row มากกว่า 0 และมี header

### Error Codes

```text
EMPTY_FILE
HEADER_NOT_FOUND
```

---

## VAL-IMP-003 — File Size Limit

### Rule

file ต้องไม่เกิน limit ที่ระบบกำหนด หรือใช้ chunk upload

### Recommended MVP

- warning if > 5 MB
- chunk upload required if > threshold

### Error Code

```text
FILE_TOO_LARGE
```

---

## VAL-IMP-004 — Duplicate Checksum

### Rule

ถ้า file checksum ซ้ำ ต้องแจ้งเตือนหรือ block ตาม policy

### Policies

```text
WARN_ONLY
BLOCK_DUPLICATE
ALLOW_WITH_REASON
```

### Error/Warning Codes

```text
DUPLICATE_FILE_CHECKSUM
DUPLICATE_IMPORT_REQUIRES_REASON
```

---

## VAL-IMP-005 — Business Date Required for Import

### Rule

import ต้องมี business_date หรือ date range ตาม file type

### Inquiry

ต้องมี business_date/snapshot date

### In-Out

ต้องมี date_from/date_to หรือ detect จาก rows ได้

### Error Code

```text
IMPORT_DATE_REQUIRED
```

---

## VAL-IMP-006 — Import Batch Status Valid

### Rule

แต่ละ action ต้องใช้ batch status ที่ถูกต้อง

| Action | Allowed Status |
|---|---|
| upload chunk | UPLOADED |
| save mapping | UPLOADED / MAPPED |
| normalize | MAPPED |
| confirm | NORMALIZED / NORMALIZED_WITH_WARNING |
| cancel | UPLOADED / MAPPED / FAILED |

### Error Code

```text
IMPORT_INVALID_STATUS
```

---

# 9. Column Mapping Validation

## VAL-CM-001 — Required Mapping Exists

### Rule

file ต้อง map canonical field ที่จำเป็นก่อน normalize

### Inquiry Required Fields

```text
business_date or snapshot_date
warehouse_raw or dc_raw
location_raw optional depending ERP format
item_raw_text or item_code_raw
qty
```

### System In-Out Required Fields

```text
movement_date or posting_date
movement_type
warehouse_raw or dc_raw
location_raw or from/to location raw
item_raw_text or item_code_raw
qty
```

### Error Code

```text
MISSING_REQUIRED_MAPPING
```

---

## VAL-CM-002 — Duplicate Canonical Mapping

### Rule

canonical field เดียวไม่ควรถูก map จากหลาย source columns เว้นแต่ field นั้นอนุญาต merge

### Error/Warning Code

```text
DUPLICATE_CANONICAL_MAPPING
```

---

## VAL-CM-003 — Source Column Exists

### Rule

source_column_name ที่ mapping ต้องมีอยู่ใน file header จริง

### Error Code

```text
SOURCE_COLUMN_NOT_FOUND
```

---

## VAL-CM-004 — Header Changed Warning

### Rule

ถ้าใช้ mapping template เดิมแต่ header file เปลี่ยน ต้อง warning ให้ user review

### Warning Code

```text
IMPORT_HEADER_CHANGED
```

---

# 10. Raw Row / Normalize Validation

## VAL-NORM-001 — Row Parseable

### Rule

raw row ต้อง parse ได้ตาม mapping

### Error Codes

```text
ROW_PARSE_FAILED
INVALID_ROW_FORMAT
```

---

## VAL-NORM-002 — Required Value in Row

### Rule

row ที่ขาด field สำคัญต้อง mark error หรือ warning ตาม criticality

### Examples

- qty missing → ERROR
- item missing → ERROR/UNKNOWN_ALIAS depending source
- UOM missing but default exists → WARNING

### Error Code

```text
ROW_REQUIRED_VALUE_MISSING
```

---

## VAL-NORM-003 — Unknown Alias Handling

### Rule

ถ้า raw text map master ไม่ได้ ต้องเข้า alias_review_queue

### Error/Warning Code

```text
UNKNOWN_ALIAS
```

### Severity

WARNING ถ้าไม่ block normalize  
ERROR ถ้า field จำเป็นต่อ canonical record

---

## VAL-NORM-004 — Duplicate Source Row

### Rule

row ที่มี duplicate key เดียวกันควรถูก flag

### Possible Duplicate Key

```text
file_type + source_ref + movement_date + item + location + qty
```

ต้องปรับหลัง Data Profiling

### Warning Code

```text
DUPLICATE_SOURCE_ROW
```

---

# 11. Alias Validation

## VAL-ALIAS-001 — Alias Text Required

### Rule

alias_text ต้องมีค่า

### Error Code

```text
ALIAS_TEXT_REQUIRED
```

---

## VAL-ALIAS-002 — Alias Type Required

### Rule

alias ต้องมี type ชัดเจน

### Allowed Types

```text
CUSTOMER
ITEM
LOCATION
DC
WAREHOUSE
UOM
LOT_PATTERN
```

### Error Code

```text
ALIAS_TYPE_REQUIRED
```

---

## VAL-ALIAS-003 — Canonical Target Required When Mapping

### Rule

ถ้า action = map to existing ต้องมี canonical_id ที่ถูกต้องตาม alias_type

### Error Code

```text
CANONICAL_TARGET_REQUIRED
```

---

## VAL-ALIAS-004 — Alias Type Must Match Canonical Type

### Rule

CUSTOMER alias ต้อง map ไป customer เท่านั้น  
ITEM alias ต้อง map ไป item เท่านั้น

### Error Code

```text
ALIAS_CANONICAL_TYPE_MISMATCH
```

### Severity

CRITICAL

---

## VAL-ALIAS-005 — Duplicate Alias Warning

### Rule

alias_text เดียวกันใน type เดียวกันไม่ควร map ไปหลาย canonical target

### Warning/Error Code

```text
ALIAS_DUPLICATE
ALIAS_CONFLICTING_TARGET
```

---

## VAL-ALIAS-006 — Low Confidence Requires Review

### Rule

ถ้า confidence ต่ำกว่า threshold ต้อง manual review

### Recommended Threshold

```text
< 90 requires review in MVP
< 70 mandatory review and cannot bulk map
```

### Error Code

```text
LOW_CONFIDENCE_REQUIRES_REVIEW
```

---

## VAL-ALIAS-007 — Product Group Pattern Validation

### Rule

pattern ของสินค้าแต่ละกลุ่มต้องแยกกัน ไม่บังคับใช้ pattern น้ำตาลกับทุกสินค้า

### Warning Code

```text
PRODUCT_GROUP_PATTERN_UNCONFIRMED
```

---

# 12. Master Data Validation

## VAL-MD-001 — Master Code Unique

### Rule

master code ต้อง unique ตาม type

### Applies To

- customer_code
- item_code
- warehouse_code
- location_code
- uom_code

### Error Code

```text
MASTER_CODE_DUPLICATE
```

---

## VAL-MD-002 — Master Name Required

### Rule

master name ต้องมีค่า

### Error Code

```text
MASTER_NAME_REQUIRED
```

---

## VAL-MD-003 — Location Must Belong to Warehouse

### Rule

location ต้องผูก warehouse ที่มีอยู่จริง

### Error Code

```text
LOCATION_WAREHOUSE_REQUIRED
```

---

## VAL-MD-004 — Used Master Cannot Be Hard Deleted

### Rule

ถ้า master ถูกใช้งานแล้ว ห้าม delete จริง ให้ inactive

### Error Code

```text
MASTER_IN_USE_CANNOT_DELETE
```

---

## VAL-MD-005 — Item Product Group Required

### Rule

item ต้องมี product_group_id เพื่อให้ระบบเลือก identity/matching rule ถูก

### Error Code

```text
ITEM_PRODUCT_GROUP_REQUIRED
```

---

# 13. Inquiry Snapshot Validation

## VAL-INQ-001 — Inquiry Is Snapshot Only

### Rule

Inquiry import ต้องไม่สร้าง movement ledger โดยตรง

### Error Code

```text
INQUIRY_CANNOT_CREATE_MOVEMENT
```

### Severity

CRITICAL

---

## VAL-INQ-002 — Snapshot Date Required

### Rule

Inquiry ต้องมี business_date / snapshot_date

### Error Code

```text
SNAPSHOT_DATE_REQUIRED
```

---

## VAL-INQ-003 — Active Snapshot Conflict

### Rule

ถ้า business_date + warehouse มี active snapshot แล้ว ต้องเลือก policy

### Allowed Policy

```text
ARCHIVE_OLD_USE_NEW
REPLACE_OLD_WITH_NEW
KEEP_BOTH_MANUAL_ACTIVE
CANCEL_NEW_UPLOAD
```

### Error Code

```text
ACTIVE_SNAPSHOT_CONFLICT
```

---

## VAL-INQ-004 — Snapshot Line Balance Valid

### Rule

snapshot line ต้องมี qty ที่ valid และ canonical item/location/customer ตาม requirement

### Error Codes

```text
SNAPSHOT_LINE_INVALID_QTY
SNAPSHOT_LINE_UNKNOWN_ALIAS
```

---

## VAL-INQ-005 — Duplicate Snapshot Checksum

### Rule

checksum ซ้ำต้อง warning/block ตาม policy

### Code

```text
DUPLICATE_SNAPSHOT_CHECKSUM
```

---

# 14. System In-Out Validation

## VAL-SIO-001 — Movement Date or Posting Date Required

### Rule

System In-Out ต้องมี movement_date หรือ posting_date อย่างน้อย 1 ค่า

### Error Code

```text
SYSTEM_INOUT_DATE_REQUIRED
```

---

## VAL-SIO-002 — Movement Type Required and Known

### Rule

movement_type ต้อง map เป็น canonical type ได้

### Error Code

```text
UNKNOWN_MOVEMENT_TYPE
```

---

## VAL-SIO-003 — Location Rule by Movement Type

### Rule

location requirement ต่างกันตาม movement_type

| Movement Type | Required Location |
|---|---|
| IN | to_location or location |
| OUT | from_location or location |
| TRANSFER | from_location and to_location |
| REJ | from_location and rej/to_location |
| ADJUSTMENT | location |

### Error Code

```text
MOVEMENT_LOCATION_REQUIRED
```

---

## VAL-SIO-004 — Qty Sign Validation

### Rule

ต้อง validate ว่า qty sign สอดคล้องกับ movement_type/mapping config

### Example

ถ้า config = positive_qty_only แต่พบ negative qty ต้อง warning/error

### Code

```text
QTY_SIGN_CONFLICT
```

---

## VAL-SIO-005 — Duplicate ERP Reference

### Rule

ถ้า source_ref ซ้ำกับ key เดิม ต้อง flag duplicate

### Warning Code

```text
DUPLICATE_SYSTEM_INOUT_REF
```

---

# 15. Field Movement Validation

## VAL-FM-001 — Movement Type Required

### Rule

movement_type ต้องมีและอยู่ใน allowed list

### Allowed

```text
IN
OUT
TRANSFER
REJ
ADJUSTMENT
RETURN
```

### Error Code

```text
FIELD_MOVEMENT_TYPE_REQUIRED
```

---

## VAL-FM-002 — Movement Qty Positive

### Rule

qty ต้องมากกว่า 0

### Error Code

```text
FIELD_MOVEMENT_QTY_INVALID
```

---

## VAL-FM-003 — Customer / Item Required Except Pending Inbound

### Rule

Field Movement ทั่วไปต้องมี customer/item  
ยกเว้น inbound ที่บันทึกเป็น Pending Inbound

### Error Code

```text
CUSTOMER_ITEM_REQUIRED_OR_PENDING_INBOUND
```

---

## VAL-FM-004 — Location Required

### Rule

movement ต้องมี location หรือ from/to ตาม movement_type

### Error Code

```text
FIELD_MOVEMENT_LOCATION_REQUIRED
```

---

## VAL-FM-005 — Locked Day Cannot Edit

### Rule

ห้ามเพิ่ม/แก้ movement ในวัน LOCKED โดยตรง

### Error Code

```text
LOCKED_DAY
```

---

## VAL-FM-006 — Checker Session Required if Checker Role

### Rule

ถ้า actor เป็น Checker ต้องมี active session ที่ warehouse นั้น

### Error Code

```text
CHECKER_SESSION_REQUIRED
```

---

# 16. Pending Inbound Validation

## VAL-PI-001 — Minimal Data Required

### Rule

Pending Inbound ต้องมีข้อมูลขั้นต่ำ

### Required

```text
business_date
warehouse_id
location_id
qty
uom or default_uom
text_on_label or photo_url
reported_by
```

### Error Code

```text
PENDING_INBOUND_MINIMUM_REQUIRED
```

---

## VAL-PI-002 — Pending Inbound Qty Positive

### Rule

qty ต้องมากกว่า 0

### Error Code

```text
PENDING_INBOUND_QTY_INVALID
```

---

## VAL-PI-003 — Confirm Requires Canonical Identity

### Rule

ก่อน confirm ต้องรู้ canonical customer/item/location/uom ตาม requirement

### Error Code

```text
PENDING_INBOUND_CANONICAL_REQUIRED
```

---

## VAL-PI-004 — Candidate Match Confidence

### Rule

ถ้า candidate confidence ต่ำ ต้อง manual review

### Error Code

```text
PENDING_INBOUND_LOW_CONFIDENCE
```

---

## VAL-PI-005 — Duplicate Pending Inbound Warning

### Rule

ถ้า same date/location/qty/text ใกล้เคียงกันมาก ต้อง warning

### Warning Code

```text
PENDING_INBOUND_DUPLICATE_CANDIDATE
```

---

# 17. Physical Count Validation

## VAL-PC-001 — Count Scope Required

### Rule

Physical Count ต้องมี scope ชัดเจน

### Required

```text
business_date
warehouse_id
location_id
qty_counted
count_type
counted_by
```

### Error Code

```text
COUNT_SCOPE_REQUIRED
```

---

## VAL-PC-002 — Count Qty Non-Negative

### Rule

qty_counted ต้องมากกว่าหรือเท่ากับ 0

### Error Code

```text
COUNT_QTY_INVALID
```

---

## VAL-PC-003 — Active Checker Session Required

### Rule

ถ้า checker เป็นผู้ submit count ต้องมี checker_session_id active

### Error Code

```text
CHECKER_SESSION_REQUIRED
```

---

## VAL-PC-004 — Count Location Must Match Session Warehouse

### Rule

location ที่นับต้องอยู่ใน warehouse ที่ checker check-in

### Error Code

```text
COUNT_LOCATION_OUT_OF_SESSION_WAREHOUSE
```

---

## VAL-PC-005 — Recount Must Link Task or Issue if Created from Issue

### Rule

ถ้า count_type = RECOUNT จาก issue ต้องมี task_id หรือ issue_id

### Warning/Error Code

```text
RECOUNT_TASK_LINK_REQUIRED
```

---

# 18. REJ Validation

## VAL-REJ-001 — REJ Reason Required

### Rule

REJ ต้องมี reason_id หรือ reason_text

### Error Code

```text
REJ_REASON_REQUIRED
```

---

## VAL-REJ-002 — From Location and REJ Location Required

### Rule

ต้องมี from_location และ rej_location

### Error Code

```text
REJ_LOCATION_REQUIRED
```

---

## VAL-REJ-003 — REJ Qty Positive

### Rule

qty ต้องมากกว่า 0

### Error Code

```text
REJ_QTY_INVALID
```

---

## VAL-REJ-004 — REJ Location Type Valid

### Rule

rej_location ต้องเป็น location_type = REJ หรืออยู่ใน configured REJ area

### Error Code

```text
INVALID_REJ_LOCATION
```

---

## VAL-REJ-005 — REJ Must Wait System Post

### Rule

หลัง create REJ status ต้องเป็น WAIT_SYSTEM_POST เว้นแต่ system in-out match ทันที

### Validation Code

```text
REJ_STATUS_INVALID
```

---

# 19. Movement Ledger Validation

## VAL-LEDGER-001 — Ledger Source Required

### Rule

ทุก ledger row ต้องมี source_type และ source_ref_id

### Error Code

```text
LEDGER_SOURCE_REQUIRED
```

---

## VAL-LEDGER-002 — Ledger Movement Type Required

### Rule

movement_type ต้องเป็น canonical stock effect type

### Allowed

```text
IN
OUT
TRANSFER_IN
TRANSFER_OUT
REJ
ADJUSTMENT_IN
ADJUSTMENT_OUT
```

### Error Code

```text
LEDGER_MOVEMENT_TYPE_INVALID
```

---

## VAL-LEDGER-003 — Stock Effect Sign Valid

### Rule

stock_effect_sign ต้องสัมพันธ์กับ movement_type

### Example

| Movement Type | Sign |
|---|---:|
| IN | +1 |
| OUT | -1 |
| TRANSFER_IN | +1 |
| TRANSFER_OUT | -1 |
| REJ | -1 |

### Error Code

```text
LEDGER_SIGN_INVALID
```

---

## VAL-LEDGER-004 — Ledger Cannot Be Edited Silently

### Rule

ถ้า ledger ถูกใช้ replay/reconciliation แล้ว ห้ามแก้ค่าเดิมโดยไม่มี correction/supersede

### Error Code

```text
LEDGER_DIRECT_EDIT_NOT_ALLOWED
```

---

# 20. Stock Replay Validation

## VAL-SR-001 — Date Range Required

### Rule

run stock replay ต้องมี date_from/date_to และ date range valid

### Error Code

```text
REPLAY_DATE_RANGE_REQUIRED
```

---

## VAL-SR-002 — Anchor Snapshot Required or Risk Accepted

### Rule

ต้องมี anchor snapshot ก่อน replay range หรือมี policy ให้ estimated with warning

### Error/Warning Codes

```text
MISSING_ANCHOR
REPLAY_WITHOUT_ANCHOR_REQUIRES_APPROVAL
```

---

## VAL-SR-003 — Movement Ledger Coverage Check

### Rule

ต้องตรวจว่า ledger/in-out ครบในช่วงที่ replay

### Warning/Error Codes

```text
MISSING_SYSTEM_INOUT
PARTIAL_LEDGER_COVERAGE
```

---

## VAL-SR-004 — Replay Job Status Valid

### Rule

job เดียวกันห้าม run ซ้อน date/warehouse เดียวกันโดยไม่มี lock/idempotency

### Error Code

```text
REPLAY_JOB_ALREADY_RUNNING
```

---

## VAL-SR-005 — Negative Expected Stock Warning

### Rule

ถ้า ending_expected_qty ติดลบ ต้องสร้าง warning/issue ตาม policy

### Code

```text
NEGATIVE_EXPECTED_STOCK
```

### Severity

HIGH หรือ CRITICAL ตาม qty/product/location

---

# 21. Reconciliation Validation

## VAL-REC-001 — Coverage Ready Before Reconcile

### Rule

ก่อน run reconciliation ต้องมี coverage check

### If Coverage Critical

block หรือ run with LOW confidence ตาม policy

### Error Code

```text
COVERAGE_NOT_READY
```

---

## VAL-REC-002 — Matching Key Config Exists

### Rule

ต้องมี matching key config สำหรับ compare type

### Compare Types

```text
FIELD_VS_SYSTEM
EXPECTED_VS_COUNT
REJ_VS_SYSTEM
PENDING_INBOUND_VS_INOUT
ANCHOR_VS_REPLAY
```

### Error Code

```text
MATCHING_KEY_CONFIG_MISSING
```

---

## VAL-REC-003 — Result Code Required

### Rule

ทุก recon result ต้องมี result_code

### Error Code

```text
RESULT_CODE_REQUIRED
```

---

## VAL-REC-004 — Non-Matched Requires Explanation

### Rule

ถ้า result_code != MATCHED ต้องมี explanation, likely_cause, suggested_action

### Error Code

```text
RECON_EXPLANATION_REQUIRED
```

---

## VAL-REC-005 — Low Coverage Cannot Produce High Confidence

### Rule

ถ้า coverage LOW/CRITICAL ห้าม result confidence = HIGH

### Error Code

```text
CONFIDENCE_CONFLICT_WITH_COVERAGE
```

---

## VAL-REC-006 — Critical Result Must Create Issue

### Rule

ถ้า severity = CRITICAL ต้องสร้าง issue หรือมี reason ว่าทำไมไม่สร้าง

### Error Code

```text
CRITICAL_RESULT_REQUIRES_ISSUE
```

---

# 22. Issue / Task Validation

## VAL-ISS-001 — Issue Must Link Result

### Rule

issue ต้อง link recon result หรือ rule failure source

### Error Code

```text
ISSUE_SOURCE_REQUIRED
```

---

## VAL-ISS-002 — Issue Owner Required

### Rule

issue ต้องมี owner_role

### Error Code

```text
ISSUE_OWNER_REQUIRED
```

---

## VAL-ISS-003 — Closing Issue Requires Reason

### Rule

close issue ต้องมี close_reason หรือ resolution_code

### Error Code

```text
ISSUE_CLOSE_REASON_REQUIRED
```

---

## VAL-ISS-004 — Closed Issue Cannot Be Edited Without Reopen

### Rule

ถ้า issue CLOSED ต้อง reopen ก่อนแก้ action/status สำคัญ

### Error Code

```text
ISSUE_ALREADY_CLOSED
```

---

## VAL-ISS-005 — Accept Difference Requires Permission and Reason

### Rule

accept diff ต้องมี CAN_ACCEPT_DIFF และ reason

### Error Codes

```text
ACCEPT_DIFF_PERMISSION_REQUIRED
ACCEPT_DIFF_REASON_REQUIRED
```

---

## VAL-TASK-001 — Task Must Have Type and Scope

### Rule

checker task ต้องมี task_type และ scope เพียงพอ

### Required

- task_type
- business_date
- warehouse_id
- priority
- owner_role or assigned_to

### Error Code

```text
TASK_SCOPE_REQUIRED
```

---

# 23. Export Validation

## VAL-EXP-001 — Export Type Required

### Rule

export ต้องมี export_type

### Allowed

```text
ISSUE_LIST
DAILY_RECON
REJ_REPORT
ERP_TEMPLATE
COUNT_TASK
AUDIT
RETENTION
```

### Error Code

```text
EXPORT_TYPE_REQUIRED
```

---

## VAL-EXP-002 — Export Permission Required

### Rule

ต้องมี CAN_EXPORT หรือ permission เฉพาะ export type

### Error Code

```text
EXPORT_PERMISSION_DENIED
```

---

## VAL-EXP-003 — Export Scope Required

### Rule

export ต้องมี filter หรือ selected rows เพื่อป้องกัน export ข้อมูลเกินจำเป็น

### Error Code

```text
EXPORT_SCOPE_REQUIRED
```

---

## VAL-EXP-004 — Export Row Limit

### Rule

ถ้า row count เกิน limit ต้องใช้ async job หรือ warning

### Code

```text
EXPORT_ROW_LIMIT_EXCEEDED
```

---

## VAL-EXP-005 — Export Log Required

### Rule

export สำเร็จ/ล้มเหลวต้องมี export log

### Error Code

```text
EXPORT_LOG_REQUIRED
```

---

# 24. Notification Validation

## VAL-NOTI-001 — Channel Type Required

### Rule

notification channel ต้องมี channel_type

### Allowed

```text
IN_APP
TELEGRAM
LINE_MESSAGING
EMAIL
```

### Error Code

```text
NOTIFICATION_CHANNEL_TYPE_REQUIRED
```

---

## VAL-NOTI-002 — Token Must Not Be Exposed

### Rule

token/secret ห้ามส่งกลับ frontend แบบ raw

### Error Code

```text
SECRET_EXPOSURE_BLOCKED
```

### Severity

CRITICAL

---

## VAL-NOTI-003 — Rule Requires Event and Target

### Rule

notification rule ต้องมี event_code, target_role/channel/template

### Error Code

```text
NOTIFICATION_RULE_INCOMPLETE
```

---

## VAL-NOTI-004 — Cooldown Required

### Rule

ถ้า rule เป็น instant alert ต้องมี cooldown_minutes หรือ send_once_per_issue

### Error/Warning Code

```text
NOTIFICATION_COOLDOWN_REQUIRED
```

---

## VAL-NOTI-005 — Template Variables Valid

### Rule

template variables ต้องอยู่ใน payload ที่ event นั้นส่งมา

### Error Code

```text
NOTIFICATION_TEMPLATE_VARIABLE_INVALID
```

---

# 25. Close / Lock Validation

## VAL-CLOSE-001 — Readiness Gate Required

### Rule

ก่อน close day ต้อง run readiness gate

### Required Checks

- open critical issue
- pending alias
- pending inbound
- REJ_NOT_POSTED
- missing anchor
- missing in-out
- failed reconciliation job
- pending export if required

### Error Code

```text
CLOSE_READINESS_REQUIRED
```

---

## VAL-CLOSE-002 — Final Close Blocks Critical Issue

### Rule

ถ้ายังมี open critical issue ห้าม FINAL_CLOSE

### Error Code

```text
OPEN_CRITICAL_ISSUE_EXISTS
```

---

## VAL-CLOSE-003 — Pending Inbound Blocks or Warns

### Rule

pending inbound ที่ยังไม่ confirm ต้อง block final close หรือ downgrade เป็น READY_WITH_WARNING ตาม policy

### Code

```text
PENDING_INBOUND_EXISTS
```

---

## VAL-CLOSE-004 — Lock Requires Final Close or Admin Override

### Rule

LOCKED ควรทำหลัง FINAL_CLOSED หรือ Admin override พร้อม reason

### Error Code

```text
LOCK_REQUIRES_FINAL_CLOSE
```

---

## VAL-CLOSE-005 — Unlock Requires Admin and Reason

### Rule

unlock day ต้องมี CAN_UNLOCK_DAY และ reason

### Error Codes

```text
UNLOCK_PERMISSION_REQUIRED
UNLOCK_REASON_REQUIRED
```

---

# 26. Retention Validation

## VAL-RET-001 — Cutoff Date Required

### Rule

retention job ต้องมี cutoff_date หรือใช้ default today - 45 days

### Error Code

```text
RETENTION_CUTOFF_REQUIRED
```

---

## VAL-RET-002 — Dry Run Before Manual Execute

### Rule

manual retention ควรทำ DRY_RUN ก่อน EXECUTE

### Error/Warning Code

```text
RETENTION_DRY_RUN_RECOMMENDED
```

---

## VAL-RET-003 — Skip Open Issue Records

### Rule

ห้ามลบข้อมูลที่เกี่ยวกับ open issue หรือ pending action

### Error Code

```text
RETENTION_BLOCKED_OPEN_ISSUE
```

---

## VAL-RET-004 — Archive Summary Before Delete

### Rule

ก่อนลบ raw/detail ต้องสร้าง summary/archive ที่จำเป็น

### Error Code

```text
RETENTION_ARCHIVE_REQUIRED
```

---

## VAL-RET-005 — Retention Log Required

### Rule

ทุก retention job ต้องมี retention log ต่อ table/operation

### Error Code

```text
RETENTION_LOG_REQUIRED
```

---

# 27. Dashboard / Summary Validation

## VAL-DASH-001 — Dashboard Must Use Summary Source

### Rule

Control Center ห้าม query raw full history โดยตรง

### Allowed Sources

- daily_recon_summary
- daily_issue_summary
- issues
- recon_results summary
- import_summary
- retention_logs
- system_health_checks

### Error Code

```text
DASHBOARD_RAW_QUERY_NOT_ALLOWED
```

---

## VAL-DASH-002 — KPI Context Required

### Rule

KPI ต้องมี business_date, warehouse_id/filter context, last_updated

### Error Code

```text
KPI_CONTEXT_REQUIRED
```

---

## VAL-DASH-003 — Stale Data Warning

### Rule

ถ้า last_updated เก่ากว่า threshold ต้องแสดง stale warning

### Warning Code

```text
DASHBOARD_DATA_STALE
```

---

# 28. Performance / Job Validation

## VAL-JOB-001 — Heavy Job Must Have Status

### Rule

งานหนักต้องมี job record/status

### Applies To

- normalize batch
- stock replay
- reconciliation
- export large report
- retention
- health check

### Error Code

```text
JOB_STATUS_REQUIRED
```

---

## VAL-JOB-002 — Prevent Duplicate Running Job

### Rule

ห้าม run job ซ้อนใน date/warehouse/scope เดียวกันโดยไม่มี idempotency/override

### Error Code

```text
DUPLICATE_RUNNING_JOB
```

---

## VAL-JOB-003 — Job Must Record Error Count

### Rule

job ต้องเก็บ rows_processed, warning_count, error_count

### Error Code

```text
JOB_METRICS_REQUIRED
```

---

# 29. Validation Response Format

## 29.1 Single Error

```json
{
  "success": false,
  "code": "MISSING_REQUIRED_MAPPING",
  "message": "ยังไม่ได้ map column ที่จำเป็น",
  "details": {
    "missing_fields": ["qty", "business_date"]
  }
}
```

---

## 29.2 Multiple Validation Results

```json
{
  "success": false,
  "code": "VALIDATION_FAILED",
  "message": "พบข้อผิดพลาดในการตรวจสอบข้อมูล",
  "details": {
    "errors": [
      {
        "code": "INVALID_QTY",
        "field": "qty",
        "message": "Qty ต้องเป็นตัวเลข"
      }
    ],
    "warnings": [
      {
        "code": "UNKNOWN_ALIAS",
        "field": "item_raw_text",
        "message": "พบชื่อสินค้าที่ยังไม่รู้จัก จะถูกส่งเข้า Alias Review"
      }
    ]
  }
}
```

---

## 29.3 Validation Success With Warnings

```json
{
  "success": true,
  "code": "VALIDATION_PASSED_WITH_WARNING",
  "message": "ข้อมูลผ่านการตรวจสอบ แต่มีคำเตือน",
  "data": {
    "can_continue": true,
    "requires_confirmation": true
  },
  "meta": {
    "warning_count": 2
  }
}
```

---

# 30. Validation Severity Matrix

| Validation Failure | Severity | Block? | Notes |
|---|---|---:|---|
| missing username | ERROR | Yes | register/login |
| pending user login | ERROR | Yes | auth |
| permission denied | CRITICAL | Yes | security |
| missing required mapping | ERROR | Yes | import normalize |
| duplicate checksum | WARNING/ERROR | Depends | policy-based |
| unknown alias | WARNING/ERROR | Depends | if required canonical missing, block confirm |
| low confidence alias | ERROR | Yes for auto-map | manual review required |
| missing anchor | CRITICAL | Yes for final close | may allow estimated with approval |
| missing in-out day | HIGH | No/Yes | downgrade confidence or block close |
| open critical issue | CRITICAL | Yes for final close | close readiness |
| locked day edit | CRITICAL | Yes | correction flow only |
| retention open issue | CRITICAL | Yes | skip/delete blocked |
| notification cooldown missing | WARNING | No initially | but recommended required |
| export without log | CRITICAL | Yes | governance |

---

# 31. Where Validation Should Live

## 31.1 Frontend

Frontend validates for UX:

- required fields
- input format
- immediate feedback
- disable buttons
- show parse preview

But frontend validation is not trusted for security

---

## 31.2 Backend

Backend validates for integrity:

- permission
- warehouse scope
- status transition
- locked day
- duplicate
- business rules
- canonical mapping
- audit requirement

---

## 31.3 Job Layer

Job layer validates:

- batch completeness
- row parsing
- data coverage
- unknown alias
- replay feasibility
- reconciliation readiness

---

## 31.4 Database Constraint

Database should enforce where possible:

- primary key
- unique key
- not null core fields
- foreign key where practical
- indexes
- status enums if supported

Do not rely only on database constraints because error message to user will be poor if service validation is missing

---

# 32. Validation Test Cases

## 32.1 User

- สมัคร username ซ้ำ
- login ก่อน approve
- reject request แล้ว audit ยังอยู่
- active user login สำเร็จ
- locked user login ไม่ได้

## 32.2 Import

- upload empty file
- upload duplicate file
- missing required column mapping
- invalid qty format
- unknown alias generated queue
- confirm Inquiry with existing active snapshot

## 32.3 Alias

- map customer alias ไป item ต้อง block
- duplicate alias warning
- low confidence bulk map ต้อง block
- create master from alias success

## 32.4 Movement

- OUT qty negative ต้อง block
- transfer ไม่มี to_location ต้อง block
- checker ไม่มี session ต้อง block
- locked day movement ต้อง block

## 32.5 Pending Inbound

- save pending with label/qty/location success
- confirm pending without item/customer block
- candidate low confidence requires review

## 32.6 Count

- count without location block
- count qty 0 allowed
- count qty negative block
- count outside session warehouse block

## 32.7 Reconciliation

- missing anchor blocks final close
- low coverage cannot high confidence
- QTY_DIFF creates issue
- non-matched without explanation block

## 32.8 Retention

- open issue source not deleted
- closed raw older than 45 days deleted after summary
- retention dry run shows delete/skip counts
- retention execute writes log

---

# 33. Validation Acceptance Criteria

Validation layer ถือว่า complete ถ้า:

- [ ] ทุก API สำคัญมี backend validation
- [ ] ทุก form สำคัญมี frontend validation เพื่อ UX
- [ ] Validation response ใช้ code/message/details มาตรฐาน
- [ ] Import validate file/header/row/mapping/duplicate ได้
- [ ] Unknown alias เข้า review queue ได้
- [ ] Alias type mismatch ถูก block
- [ ] Inquiry snapshot conflict ถูกบังคับเลือก policy
- [ ] System In-Out validate movement/date/qty/location ได้
- [ ] Field Movement validate qty/location/session/locked day ได้
- [ ] Pending Inbound validate minimal evidence ได้
- [ ] Physical Count validate scope/session ได้
- [ ] REJ validate reason/location/qty ได้
- [ ] Stock Replay validate anchor/coverage ได้
- [ ] Reconciliation validate coverage/matching/result/explanation ได้
- [ ] Issue close ต้องมี reason
- [ ] Export ต้องมี permission/scope/log
- [ ] Notification ไม่ expose secret และมี cooldown
- [ ] Close day มี readiness gate
- [ ] Retention skip open issue และมี log
- [ ] Dashboard validate summary source/context/stale data

---

# 34. Final Validation Statement

```text
Validation Rules ของ Warehouse Reconciliation ERP Platform ต้องป้องกันข้อมูลผิดตั้งแต่จุดรับเข้า ตรวจสิทธิ์และ scope ทุก action สำคัญ ตรวจคุณภาพข้อมูลก่อน replay/reconciliation ตรวจความพร้อมก่อน close day และตรวจความปลอดภัยก่อน retention/export โดยทุก validation ต้องมี code, message, severity และ details ที่ทำให้ user หรือ AI Agent เข้าใจได้ว่าผิดอะไร ต้องแก้อย่างไร และระบบควร block, warning หรือให้ confirm ต่อได้หรือไม่
```

