# 03_module_spec.md

# Module Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Module Specification / Functional Module Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer Use  
**Primary Use:** ใช้กำหนดรายละเอียดแต่ละโมดูลของระบบ ว่าแต่ละโมดูลทำอะไร ใครใช้ ใช้ข้อมูลอะไร เชื่อมกับโมดูลไหน มี rule อะไร มี output อะไร และมี risk อะไร

---

## 1. Purpose of This Document

เอกสารนี้ลงรายละเอียดระดับ module ของ Warehouse Reconciliation ERP Platform

เป้าหมายคือให้ AI Agent / Developer เข้าใจว่าแต่ละ module:

1. มีหน้าที่อะไร
2. ใครเป็นผู้ใช้หลัก
3. รับ input อะไร
4. สร้าง output อะไร
5. เชื่อมโยงกับ module ไหน
6. มี business rule อะไร
7. ต้อง validate อะไร
8. ต้องเขียน audit log เมื่อไหร่
9. มี risk อะไรถ้าออกแบบผิด
10. อยู่ใน MVP หรือเป็น future phase

หลักสำคัญ:

```text
หนึ่งโมดูลต้องมีความรับผิดชอบชัดเจน
ห้ามเอา import, alias, reconciliation, issue, export, notification ปนกันใน logic เดียว
```

---

## 2. Module Overview

ระบบแบ่ง module หลักเป็น 6 กลุ่ม

```text
1. Control Core Modules
2. Operational Modules
3. Data Engine Modules
4. Intelligence Modules
5. Decision UI Modules
6. Maintenance & Governance Modules
```

---

## 3. Module Summary Table

| Group | Module | MVP | Primary User | Main Purpose |
|---|---|---:|---|---|
| Control Core | Auth Module | Yes | All Users | Login/session/security |
| Control Core | User Approval Module | Yes | Admin | อนุมัติผู้ใช้ใหม่ |
| Control Core | Permission Module | Yes | Admin/System | คุมสิทธิ์และ warehouse scope |
| Control Core | Master Data Module | Yes | Admin | จัดการ master หลัก |
| Control Core | Alias Management Module | Yes | Admin | map ชื่อไม่ตรงให้เป็น master |
| Control Core | Import Staging Module | Yes | Admin/Staff | รับไฟล์แบบ staging ก่อน normalize |
| Control Core | Column Mapping Module | Yes | Admin | map column ไฟล์จริงเป็น canonical field |
| Control Core | Snapshot Control Module | Yes | Admin/System | เก็บ Inquiry เป็น Anchor Snapshot |
| Operational | Field Movement Module | Yes | Checker/Staff | แจ้งเข้า/ออก/ย้าย/REJ |
| Operational | Pending Inbound Module | Yes | Checker/Admin | รับเข้าที่ข้อมูลยังไม่ครบ |
| Operational | Physical Count Module | Yes | Checker | บันทึกของจริงที่นับได้ |
| Operational | REJ Module | Yes | Checker/ERP Admin | จัดการของเสีย/เสียหาย |
| Operational | Checker Task Module | Yes | Checker/Supervisor | สร้างและติดตามงานนับ/ตรวจ |
| Operational | Checker GPS Session Module | Yes | Checker/Admin | check-in ตาม warehouse |
| Data Engine | Movement Ledger Module | Yes | System | รวม movement ทุก source |
| Data Engine | Stock Replay Module | Yes | System/Admin | คำนวณ stock ย้อนหลัง |
| Data Engine | Daily Stock Position Module | Yes | System | ยอดคงเหลือคำนวณรายวัน |
| Data Engine | Data Coverage Check Module | Yes | Admin/System | ตรวจข้อมูลครบ/ขาด |
| Intelligence | Rule Engine | Yes | System | ตรวจ rule พื้นฐาน |
| Intelligence | Reconciliation Engine | Yes | System | เทียบข้อมูลและสร้าง result |
| Intelligence | Explanation Engine | Yes | System/User | อธิบายว่าทำไมไม่ตรง |
| Intelligence | Issue Engine | Yes | System/Supervisor | สร้าง issue/task |
| Decision UI | Control Center | Yes | Admin/Supervisor | หน้ารวมสถานะสำคัญ |
| Decision UI | Issue Center | Yes | Supervisor/Admin | ดู exception และ action |
| Decision UI | Export Center | Yes | Admin/ERP Admin | ส่งออกข้อมูล |
| Decision UI | Notification Center | Yes | Admin | ตั้งค่าแจ้งเตือน |
| Maintenance | Audit Log Module | Yes | Admin/System | ตรวจย้อนหลัง |
| Maintenance | Retention Module | Yes | System/Admin | ลบ/archive เกิน 45 วัน |
| Future | Capacity Module | Later | Planner/Admin | ตรวจพื้นที่ว่าง |
| Future | Simulation Module | Later | Planner | หา location แนะนำ |
| Future | Relocation Optimizer | Later | Planner/Supervisor | เสนอการย้ายของ |
| Future | Custody Module | Later | Admin | ของฝาก/ข้ามคลัง |

---

# 4. Control Core Modules

---

## 4.1 Auth Module

### Module ID

```text
AUTH
```

### Purpose

จัดการตัวตนผู้ใช้ การ login, logout, session, token และสถานะบัญชีผู้ใช้

### Primary Users

- Checker
- Admin
- Supervisor
- ERP Admin
- Planner
- Viewer

### Core Responsibilities

1. สมัครผู้ใช้ใหม่
2. ตรวจ username/email ซ้ำ
3. hash password + salt
4. login
5. logout
6. verify session
7. reset password
8. lock account
9. expire session
10. บันทึก login audit

### Inputs

- username
- password
- email
- phone
- full_name
- device information
- login timestamp

### Outputs

- session token
- user profile
- permission context
- login status
- audit log

### Main Tables

- users
- user_access_requests
- login_sessions
- audit_logs

### Business Rules

| Rule | Description |
|---|---|
| AUTH-001 | ผู้สมัครใหม่ต้องอยู่สถานะ PENDING_APPROVAL ก่อนใช้งานจริง |
| AUTH-002 | Password ต้องเก็บเป็น hash + salt เท่านั้น |
| AUTH-003 | Pending user ห้าม login เข้าระบบจริง |
| AUTH-004 | Locked/Suspended user ห้ามเข้าใช้งาน |
| AUTH-005 | Session ต้องมีวันหมดอายุ |
| AUTH-006 | Login fail ต้องถูก log |

### Validation

- username required
- password required
- email format valid if provided
- username unique
- account status must be ACTIVE for login
- session token valid and not expired

### Audit Events

- USER_REGISTER
- USER_LOGIN_SUCCESS
- USER_LOGIN_FAILED
- USER_LOGOUT
- USER_PASSWORD_RESET
- USER_LOCKED
- USER_UNLOCKED

### Connected Modules

- User Approval Module
- Permission Module
- Checker GPS Session Module
- Audit Log Module

### Risk if Wrong

- คนที่ยังไม่อนุมัติอาจเข้าใช้งานได้
- password ไม่ปลอดภัย
- user ที่ถูกล็อคยังเข้าได้
- audit ตรวจไม่ได้ว่าใครทำอะไร

### MVP Requirement

Required in MVP

---

## 4.2 User Approval Module

### Module ID

```text
USER_APPROVAL
```

### Purpose

ให้ Admin อนุมัติหรือปฏิเสธผู้สมัครใหม่ และกำหนดสิทธิ์ก่อนใช้งานจริง

### Primary User

- Admin

### Core Responsibilities

1. แสดงรายการ user ที่รออนุมัติ
2. ดูรายละเอียดคำขอสมัคร
3. Approve user
4. กำหนด role
5. กำหนด permission
6. กำหนด warehouse scope
7. Reject request
8. ลบ request ที่ reject จาก queue
9. เก็บ audit log หลัง reject/approve
10. แจ้งเตือน Admin เมื่อมี user รออนุมัติ

### Inputs

- user_access_request
- selected roles
- selected permissions
- warehouse scope
- rejection reason

### Outputs

- active user account
- rejected/deleted request
- audit log
- notification

### Main Tables

- user_access_requests
- users
- user_roles
- user_permission_overrides
- user_warehouse_scope
- audit_logs
- notification_logs

### Workflow

```text
User registers
→ USER_ACCESS_REQUESTS = PENDING_APPROVAL
→ Admin reviews
→ Approve or Reject
```

### Approve Flow

```text
Select role
→ Select permissions
→ Select warehouse scope
→ Create user account
→ Mark request approved
→ Audit log
→ Notification
```

### Reject Flow

```text
Reject with reason
→ Delete request from pending queue
→ Keep audit log
→ Notification optional
```

### Business Rules

| Rule | Description |
|---|---|
| UA-001 | Pending user ใช้งานระบบจริงไม่ได้ |
| UA-002 | Approve ต้องกำหนดอย่างน้อย 1 role หรือ 1 permission |
| UA-003 | Reject ต้องเก็บ audit แม้ request ถูกลบจาก queue |
| UA-004 | User หนึ่งคนมีหลาย role ได้ |
| UA-005 | Permission override ใช้เสริม/ลดสิทธิ์จาก role ได้ |

### Validation

- request exists
- request status = PENDING_APPROVAL
- approver has CAN_MANAGE_USER
- at least one access scope selected
- reject reason required if reject policy enabled

### Audit Events

- USER_APPROVED
- USER_REJECT_DELETE_REQUEST
- USER_PERMISSION_ASSIGNED
- USER_WAREHOUSE_SCOPE_ASSIGNED

### Connected Modules

- Auth Module
- Permission Module
- Notification Center
- Audit Log Module

### Risk if Wrong

- คนที่ไม่ควรใช้ระบบอาจเข้าถึงข้อมูลคลัง
- user ไม่มี warehouse scope แต่เข้าไปนับได้
- reject แล้วไม่มีหลักฐาน

### MVP Requirement

Required in MVP

---

## 4.3 Permission Module

### Module ID

```text
PERMISSION
```

### Purpose

คุมสิทธิ์แบบละเอียดทั้งระดับ feature, action, record และ warehouse scope

### Primary Users

- Admin
- System

### Core Responsibilities

1. จัดการ roles
2. จัดการ permissions
3. map role → permission
4. map user → role
5. map user → permission override
6. map user → warehouse scope
7. ตรวจสิทธิ์ทุก API สำคัญ
8. ตรวจสิทธิ์ก่อนแสดงเมนู
9. ตรวจสิทธิ์ก่อน export/import/close/lock

### Permission Model

```text
Role = สิทธิ์กลุ่มเริ่มต้น
Permission = สิทธิ์จริงที่ใช้ตรวจ action
Warehouse Scope = คลัง/พื้นที่ที่ user เข้าถึงได้
Session Rule = เงื่อนไขเพิ่มเติม เช่น checker ต้อง check-in
```

### Core Permissions

- CAN_LOGIN
- CAN_MANAGE_USER
- CAN_MANAGE_PERMISSION
- CAN_IMPORT_INQUIRY
- CAN_IMPORT_INOUT
- CAN_MANAGE_ALIAS
- CAN_CREATE_FIELD_MOVEMENT
- CAN_COUNT
- CAN_CREATE_REJ
- CAN_RUN_RECONCILIATION
- CAN_REVIEW_ISSUE
- CAN_APPROVE
- CAN_EXPORT
- CAN_CLOSE_DAY
- CAN_UNLOCK_DAY
- CAN_MANAGE_NOTIFICATION
- CAN_RUN_RETENTION

### Main Tables

- roles
- permissions
- role_permissions
- user_roles
- user_permission_overrides
- user_warehouse_scope

### Business Rules

| Rule | Description |
|---|---|
| PERM-001 | Permission ต้องตรวจ backend ทุกครั้ง |
| PERM-002 | การ hide menu ใน frontend ไม่ถือว่าเพียงพอ |
| PERM-003 | User มีหลาย role ได้ |
| PERM-004 | Warehouse scope ต้องตรวจใน action ที่เกี่ยวกับคลัง |
| PERM-005 | Checker ต้องมี active checker session ก่อน CAN_COUNT ใช้งานได้จริง |
| PERM-006 | คนสร้างบาง proposal ไม่ควร approve เองใน phase หลัง |

### Validation

- permission_code exists
- role exists
- user active
- scope valid
- action allowed for user
- warehouse allowed for user

### Audit Events

- ROLE_CREATED
- ROLE_UPDATED
- PERMISSION_ASSIGNED
- PERMISSION_REMOVED
- USER_SCOPE_CHANGED

### Connected Modules

- Auth Module
- User Approval Module
- Checker GPS Session
- All backend APIs

### Risk if Wrong

- สิทธิ์หลุด
- user เห็น/แก้ warehouse ที่ไม่เกี่ยวข้อง
- checker นับโดยไม่ check-in
- export ข้อมูลได้โดยไม่มีสิทธิ์

### MVP Requirement

Required in MVP

---

## 4.4 Master Data Module

### Module ID

```text
MASTER_DATA
```

### Purpose

จัดการข้อมูลกลางที่ใช้เป็น canonical reference ของระบบ

### Primary User

- Admin

### Core Responsibilities

1. จัดการ customer master
2. จัดการ item master
3. จัดการ warehouse/DC master
4. จัดการ location master
5. จัดการ UOM master
6. จัดการ product group master
7. จัดการ reason master เช่น REJ reason
8. เปิด/ปิด active status
9. ป้องกันการลบ master ที่ถูกใช้งานแล้ว

### Main Master Entities

- customers
- items
- product_groups
- warehouses
- locations
- uoms
- reasons

### Important Product Groups

- Sugar
- Wood
- Flour
- Molasses
- Bulk / เทกอง
- Other

### Business Rules

| Rule | Description |
|---|---|
| MD-001 | Master ที่ถูกใช้งานแล้วห้าม delete จริง ให้ inactive |
| MD-002 | Master ใหม่จาก unknown alias ต้องมี review/approve |
| MD-003 | Item identity ต้องรองรับหลาย product group |
| MD-004 | น้ำตาลใช้ customer + item alias + year + lot ได้ แต่สินค้าอื่นอาจใช้ template ต่างกัน |
| MD-005 | Location ต้องผูก warehouse/DC canonical |

### Validation

- code unique
- name required
- product group valid
- location belongs to warehouse
- UOM valid
- active status required

### Audit Events

- MASTER_CREATED
- MASTER_UPDATED
- MASTER_INACTIVATED
- MASTER_MERGED

### Connected Modules

- Alias Management
- Import Normalize
- Smart Input
- Reconciliation
- Export

### Risk if Wrong

- alias map ไป master ผิด
- item คนละประเภทถูกปนกัน
- location/DC normalize ผิด
- report และ reconciliation ผิดทั้งระบบ

### MVP Requirement

Required in MVP, but can start with minimal master + alias review

---

## 4.5 Alias Management Module

### Module ID

```text
ALIAS
```

### Purpose

จัดการชื่อที่ไม่ตรงกันระหว่าง ERP, ป้ายหน้างาน, ชื่อย่อ, Excel และ master canonical

### Primary User

- Admin

### Core Responsibilities

1. เก็บ alias แยกตาม type
2. ตรวจ unknown alias จาก import/smart input
3. สร้าง Alias Review Queue
4. เสนอ suggested match พร้อม confidence
5. ให้ Admin map alias ไป master
6. ให้ Admin สร้าง master ใหม่จาก alias
7. reject alias ที่ไม่ถูกต้อง
8. จำ alias สำหรับครั้งต่อไป

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

### Example

```text
MKMK / UPUP / UFUF = CUSTOMER_ALIAS
W150 / RE T1 / SR CT1 / ไฮโพลดิบ = ITEM_ALIAS
W8 / DC08 = DC_ALIAS
A01 / W8-A01 = LOCATION_ALIAS
BAG / ถุง = UOM_ALIAS
```

### Main Tables

- customer_aliases
- item_aliases
- location_aliases
- dc_aliases
- uom_aliases
- alias_review_queue

### Business Rules

| Rule | Description |
|---|---|
| ALIAS-001 | ห้ามรวม customer alias กับ item alias โดยไม่แยก type |
| ALIAS-002 | Unknown alias ต้องเข้า review queue |
| ALIAS-003 | Low confidence ห้าม auto-map |
| ALIAS-004 | Admin map ครั้งเดียว ระบบต้องจำครั้งต่อไป |
| ALIAS-005 | Alias ต้อง trace ได้ว่ามาจาก batch/row ไหน |
| ALIAS-006 | Product group อื่นไม่จำเป็นต้องใช้ pattern น้ำตาล |

### Validation

- alias_text required
- alias_type required
- canonical_id required if mapped
- duplicate alias warning
- confidence recorded
- source reference recorded

### Audit Events

- ALIAS_DETECTED
- ALIAS_MAPPED
- ALIAS_CREATED_MASTER
- ALIAS_REJECTED
- ALIAS_UPDATED

### Connected Modules

- Import Staging
- Smart Input
- Pending Inbound
- Reconciliation
- Data Profiling
- Issue Center

### Risk if Wrong

- MKMK อาจถูกตีเป็น item แทน customer
- W150 ปี 68/69 อาจปนกัน
- location/DC match ผิด
- false reconciliation จำนวนมาก

### MVP Requirement

Required in MVP and should be built early

---

## 4.6 Import Staging Module

### Module ID

```text
IMPORT_STAGING
```

### Purpose

รับไฟล์ข้อมูลจริง เช่น Inquiry, System In-Out, Count, Master โดยเก็บ raw ก่อน แล้วค่อย mapping/normalize

### Primary Users

- Admin
- Staff with import permission

### Core Responsibilities

1. upload file
2. client-side parse
3. preview headers
4. preview sample rows
5. create import batch
6. chunk upload raw rows
7. save raw_json
8. validate required mapping
9. normalize batch
10. create import log

### Import Types

```text
INQUIRY
SYSTEM_INOUT
FIELD_COUNT
MASTER_DATA
REJ_IMPORT optional
```

### Main Tables

- import_batches
- import_raw_rows
- import_column_mapping
- import_error_logs

### Business Rules

| Rule | Description |
|---|---|
| IMP-001 | ทุก import ต้องมี batch_id |
| IMP-002 | Raw data ต้องไม่เข้าตารางจริงทันที |
| IMP-003 | ต้อง preview ก่อน confirm |
| IMP-004 | ต้องมี column mapping |
| IMP-005 | ต้องตรวจ duplicate checksum |
| IMP-006 | ต้องเก็บ import log |
| IMP-007 | Large file ต้อง chunk upload |

### Validation

- file type allowed
- row count > 0
- header exists
- required field mapped
- qty parseable
- date parseable
- batch checksum warning if duplicate

### Audit Events

- IMPORT_UPLOADED
- IMPORT_PREVIEWED
- IMPORT_MAPPING_SAVED
- IMPORT_CONFIRMED
- IMPORT_FAILED

### Connected Modules

- Column Mapping
- Alias Management
- Snapshot Control
- Movement Ledger
- Data Coverage Check

### Risk if Wrong

- ข้อมูล raw หาย
- column ผิดทำให้ normalize ผิด
- import ซ้ำแล้วไม่ได้รู้ตัว
- reconciliation ผิดทั้ง batch

### MVP Requirement

Required in MVP

---

## 4.7 Column Mapping Module

### Module ID

```text
COLUMN_MAPPING
```

### Purpose

กำหนดว่า column จากไฟล์จริง map ไป canonical field ไหน

### Primary User

- Admin

### Core Responsibilities

1. อ่าน header จากไฟล์
2. แสดง mapping UI
3. save mapping template per file type
4. reuse mapping ครั้งถัดไป
5. validate required canonical fields
6. allow update mapping without code change

### Canonical Field Examples

- business_date
- posting_date
- movement_date
- warehouse_raw
- location_raw
- customer_raw
- item_raw
- lot_raw
- year_raw
- qty
- uom
- movement_type
- source_ref

### Business Rules

| Rule | Description |
|---|---|
| CM-001 | ห้าม hardcode column index |
| CM-002 | Mapping ต้องเก็บตาม file_type |
| CM-003 | Required fields ต้องถูก map ก่อน normalize |
| CM-004 | ถ้า header เปลี่ยน ต้องแจ้ง warning |

### Validation

- source_column exists in batch
- canonical_field valid
- required fields mapped
- duplicate canonical mapping warning

### Audit Events

- COLUMN_MAPPING_CREATED
- COLUMN_MAPPING_UPDATED
- COLUMN_MAPPING_USED

### Connected Modules

- Import Staging
- Normalize Engine
- Data Profiling

### Risk if Wrong

- อ่าน qty เป็น location
- อ่าน posting date เป็น business date
- mapping ผิดแต่ระบบ normalize ต่อ

### MVP Requirement

Required in MVP

---

# 5. Operational Modules

---

## 5.1 Field Movement Module

### Module ID

```text
FIELD_MOVEMENT
```

### Purpose

บันทึก movement ที่หน้างานแจ้งว่าเกิดขึ้นจริง เช่น เข้า ออก ย้าย REJ

### Primary Users

- Checker
- Staff
- Supervisor

### Movement Types

```text
IN
OUT
TRANSFER
REJ
ADJUST
RETURN
PENDING_INBOUND_CONFIRM
```

### Core Responsibilities

1. แจ้ง movement
2. เก็บข้อมูล customer/item/location/qty
3. รองรับ unknown/pending เฉพาะ inbound
4. เขียน movement ledger
5. wait for system post
6. link กับ count หรือ evidence

### Main Tables

- field_movements
- movement_ledger
- audit_logs

### Business Rules

| Rule | Description |
|---|---|
| FM-001 | Field Movement ไม่ใช่ ERP post |
| FM-002 | ทุก movement ต้องมี movement_id |
| FM-003 | ทุก movement ที่มีผลต่อ stock ต้องเข้า ledger |
| FM-004 | Locked day ห้ามเพิ่ม/แก้ตรง |
| FM-005 | IN ที่ข้อมูลไม่ครบต้องเข้า Pending Inbound |

### Validation

- movement_type required
- qty > 0
- business_date required
- location required unless special case
- user permission required
- checker session required if checker action

### Output

- field movement record
- ledger entry
- wait-system-post status
- audit log

### Connected Modules

- Pending Inbound
- Movement Ledger
- Reconciliation
- Issue Center

### Risk if Wrong

- หน้างานแจ้งแล้วไม่ถูก ledger
- ERP post ไม่ครบแต่ระบบไม่รู้
- locked day ถูกแก้ย้อนหลัง

### MVP Requirement

Required in MVP

---

## 5.2 Pending Inbound Module

### Module ID

```text
PENDING_INBOUND
```

### Purpose

รองรับของเข้าเมื่อข้อมูลยังไม่ครบ เช่น ยังไม่รู้ customer/item/lot/year แต่ของเข้าจริงแล้ว

### Primary Users

- Checker
- Admin
- Supervisor

### Use Cases

- ป้ายหน้ากองมีแค่ชื่อย่อ
- ของเข้าแล้วแต่ยังไม่มี In-Out
- ยังไม่รู้ item เต็ม
- ยังไม่รู้ lot/year
- รอวันถัดไปดึง In-Out มายืนยัน

### Minimal Required Input

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

### Main Tables

- pending_inbounds
- pending_inbound_candidates
- alias_review_queue
- audit_logs

### Business Rules

| Rule | Description |
|---|---|
| PI-001 | Pending inbound ใช้ได้เฉพาะกรณีข้อมูลไม่ครบจริง |
| PI-002 | ต้องเก็บ text_on_label หรือ evidence |
| PI-003 | เมื่อ In-Out เข้า ต้องพยายามหา candidate match |
| PI-004 | Confirm แล้วต้องสร้าง canonical movement/ledger |
| PI-005 | Pending inbound ที่ยังไม่ confirm อาจ block final close |

### Validation

- location required
- qty > 0
- label text or photo recommended
- pending reason required

### Output

- pending inbound record
- alias queue if unknown
- candidate match after In-Out
- issue if unresolved

### Connected Modules

- Smart Input
- Alias Management
- In-Out Import
- Reconciliation
- Issue Center

### Risk if Wrong

- ของเข้าแล้วหายจากระบบ
- unknown ถูกเดาผิดเป็นสินค้าอื่น
- final close ทั้งที่ inbound ยังไม่ชัด

### MVP Requirement

Required in MVP

---

## 5.3 Physical Count Module

### Module ID

```text
PHYSICAL_COUNT
```

### Purpose

บันทึกของจริงที่นับได้จากหน้างาน

### Primary User

- Checker

### Count Types

```text
BULK_COUNT
BEFORE_COUNT
AFTER_COUNT
RECOUNT
LOT_VERIFY
PARTIAL_VERIFY
```

### Core Responsibilities

1. บันทึก qty ที่นับจริง
2. ระบุ location/item/customer/year/lot เท่าที่รู้
3. แนบ evidence optional
4. เชื่อมกับ task/issue
5. ใช้เทียบกับ expected stock

### Main Tables

- physical_counts
- checker_tasks
- audit_logs

### Business Rules

| Rule | Description |
|---|---|
| PC-001 | Count ต้องมี scope ว่านับอะไร |
| PC-002 | Checker ต้องมี active session |
| PC-003 | Count หลัง locked day ต้องผ่าน correction flow |
| PC-004 | Count ที่ผูก issue ต้อง update issue status |

### Validation

- location required
- qty >= 0
- count_type required
- checker session active
- business_date required

### Connected Modules

- Checker Task
- Reconciliation
- Issue Detail
- Stock Replay

### Risk if Wrong

- นับ location ผิดแต่ระบบเอาไปเทียบผิด
- count ไม่มี scope ทำให้ reconciliation ใช้ไม่ได้

### MVP Requirement

Required in MVP

---

## 5.4 REJ Module

### Module ID

```text
REJ
```

### Purpose

จัดการ movement ของเสีย/เสียหายที่ต้องย้ายจาก stock ปกติไป REJ location และต้องตรวจว่า ERP post แล้วหรือยัง

### Primary Users

- Checker
- Supervisor
- ERP Admin

### Core Responsibilities

1. แจ้ง REJ
2. ระบุ from location
3. ระบุ item/customer/year/lot/qty
4. ระบุ reason
5. add to movement ledger
6. wait ERP post
7. reconcile with System In-Out
8. create issue if not posted

### Status

```text
REPORTED
WAIT_SYSTEM_POST
REJ_MATCHED
REJ_NOT_POSTED
REJ_QTY_DIFF
REJ_LOCATION_DIFF
INVESTIGATING
CLOSED
```

### Main Tables

- rej_notices
- rej_notice_lines
- movement_ledger
- recon_results
- issues

### Business Rules

| Rule | Description |
|---|---|
| REJ-001 | REJ ต้องมี reason |
| REJ-002 | REJ ต้องมี from location |
| REJ-003 | REJ ต้องเข้า ledger |
| REJ-004 | REJ ต้องตามจน ERP post |
| REJ-005 | REJ_NOT_POSTED ต้องแจ้ง ERP Admin |

### Validation

- qty > 0
- reason required
- from_location required
- to_location = REJ location or configured REJ area

### Connected Modules

- Field Movement
- Movement Ledger
- Reconciliation
- Issue Center
- Notification
- Export

### Risk if Wrong

- REJ กลายเป็นหลุมดำ
- ของจริงเสียหายแล้วแต่ ERP ไม่ตัด
- stock ปกติสูงเกินจริง

### MVP Requirement

Required in MVP if REJ is operationally important; otherwise basic support in MVP and expand later

---

## 5.5 Checker Task Module

### Module ID

```text
CHECKER_TASK
```

### Purpose

สร้างงานให้ checker ทำ เช่น นับใหม่ ตรวจ location ตรวจ pending inbound ตรวจ REJ

### Primary Users

- Checker
- Supervisor
- System

### Task Types

```text
COUNT
RECOUNT
VERIFY_INBOUND
VERIFY_LOCATION_DIFF
VERIFY_REJ
LOT_VERIFY
PHOTO_EVIDENCE
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

### Main Tables

- checker_tasks
- physical_counts
- issues

### Business Rules

| Rule | Description |
|---|---|
| CT-001 | Task ต้องมี owner หรือ role owner |
| CT-002 | Task ที่เกิดจาก issue ต้อง link issue_id |
| CT-003 | Done task ต้องมี result/evidence ตาม type |
| CT-004 | Checker ต้อง check-in ก่อนทำ task ที่ต้องนับ |

### Connected Modules

- Issue Engine
- Physical Count
- Checker GPS Session
- Notification

### MVP Requirement

Required in MVP for count required locations and issue follow-up

---

## 5.6 Checker GPS Session Module

### Module ID

```text
CHECKER_SESSION
```

### Purpose

ให้ checker outsource check-in ที่ warehouse ก่อนนับ และจำสิทธิ์ได้ 1 วัน

### Primary User

- Checker

### Core Responsibilities

1. checker login
2. request GPS
3. verify warehouse radius optional
4. create daily session
5. suspend previous warehouse session if check-in elsewhere
6. expire at end of business date

### Main Tables

- checker_daily_sessions
- users
- warehouses
- audit_logs

### Business Rules

| Rule | Description |
|---|---|
| CS-001 | 1 user มี active warehouse ได้ 1 ที่ต่อวัน |
| CS-002 | ถ้า check-in WH ใหม่ ให้ suspend session เดิม |
| CS-003 | Count ต้องใช้ active session |
| CS-004 | GPS accuracy ต้องถูกบันทึก |

### Validation

- user active
- user has checker permission
- warehouse valid
- GPS provided if required

### MVP Requirement

Required if checker outsource ต้องคุมสิทธิ์นับ

---

# 6. Data Engine Modules

---

## 6.1 Movement Ledger Module

### Module ID

```text
MOVEMENT_LEDGER
```

### Purpose

เป็น ledger กลางของทุก movement ที่มีผลต่อ stock

### Source Types

- SYSTEM_INOUT
- FIELD_NOTICE
- REJ_NOTICE
- TRANSFER
- ADJUSTMENT
- COUNT_CORRECTION
- PENDING_INBOUND_CONFIRM

### Main Tables

- movement_ledger

### Business Rules

| Rule | Description |
|---|---|
| ML-001 | Movement ที่มีผล stock ต้องเข้า ledger |
| ML-002 | Ledger ต้องมี source_type/source_ref_id |
| ML-003 | Ledger ต้องระบุ business_date |
| ML-004 | Ledger ใช้สำหรับ stock replay |

### Connected Modules

- Import In-Out
- Field Movement
- REJ
- Stock Replay
- Reconciliation

### MVP Requirement

Required in MVP

---

## 6.2 Stock Replay Module

### Module ID

```text
STOCK_REPLAY
```

### Purpose

คำนวณ daily stock position จาก Anchor Snapshot + Movement Ledger เพื่อรองรับ admin ไม่ upload Inquiry ทุกวัน

### Inputs

- anchor snapshot
- movement ledger
- date range
- warehouse/location/customer/item/year/lot

### Output

- daily_stock_positions
- source_mode
- confidence
- replay log

### Formula

```text
Stock Position D = Anchor A + Movement A+1 to D
```

### Source Mode

- FROM_INQUIRY
- RECONSTRUCTED_FROM_LEDGER
- ESTIMATED
- COUNT_CONFIRMED

### Business Rules

| Rule | Description |
|---|---|
| SR-001 | ถ้าไม่มี Inquiry ทุกวัน ต้อง replay จาก anchor |
| SR-002 | ถ้า In-Out ขาด ต้อง downgrade confidence |
| SR-003 | Replay job ต้องมี log |
| SR-004 | Replay date range ต้อง rebuild affected summary |

### Connected Modules

- Anchor Snapshot
- Movement Ledger
- Data Coverage Check
- Reconciliation
- Control Center

### MVP Requirement

Required in MVP

---

## 6.3 Data Coverage Check Module

### Module ID

```text
DATA_COVERAGE
```

### Purpose

บอกว่าข้อมูลช่วงวันที่เลือกครบหรือขาดอะไร

### Checks

- มี anchor snapshot ก่อนช่วงไหม
- มี System In-Out ครบทุกวันไหม
- มี unknown alias ไหม
- มี pending inbound ไหม
- มี REJ pending ไหม
- มี count สำหรับ moving location ไหม

### Coverage Status

- COVERAGE_OK
- MISSING_SYSTEM_INOUT
- MISSING_ANCHOR
- PENDING_UNKNOWN_INBOUND
- NO_COUNT_FOR_MOVING_LOCATION
- REJ_PENDING

### Connected Modules

- Backfill Wizard
- Stock Replay
- Control Center
- Daily Close
- Notification

### MVP Requirement

Required in MVP

---

# 7. Intelligence Modules

---

## 7.1 Rule Engine

### Module ID

```text
RULE_ENGINE
```

### Purpose

ตรวจ business rules และ validation rules กลางของระบบ

### Examples

- duplicate inquiry
- unknown alias
- locked day edit
- checker no active session
- pending issue before close
- retention skip open issue

### Business Rules

| Rule | Description |
|---|---|
| RULE-001 | Rule ต้อง centralized |
| RULE-002 | Rule result ต้องมี severity |
| RULE-003 | Rule fail critical ต้อง block action |
| RULE-004 | Rule warning ให้ user confirm/approve ได้ตาม policy |

### MVP Requirement

Required in MVP

---

## 7.2 Reconciliation Engine

### Module ID

```text
RECONCILIATION
```

### Purpose

เทียบข้อมูลหลาย truth layer เพื่อสร้าง result ว่าตรงหรือไม่ตรง

### Compare Pairs

```text
Field Movement vs System In-Out
Expected Stock vs Physical Count
REJ Notice vs ERP Movement
Pending Inbound vs In-Out
Anchor vs Replayed Stock
```

### Result Output

- result_code
- severity
- confidence
- explanation
- likely_cause
- suggested_action
- owner_role
- status

### Core Result Codes

- MATCHED
- SYSTEM_MISSING
- FIELD_MISSING
- QTY_DIFF
- LOCATION_DIFF
- ITEM_DIFF
- LOT_DIFF
- REJ_NOT_POSTED
- REJ_QTY_DIFF
- UNKNOWN_ALIAS
- PENDING_INBOUND
- NEED_RECOUNT
- NEED_INVESTIGATION

### Business Rules

| Rule | Description |
|---|---|
| RECON-001 | ห้ามฟันธง high confidence ถ้าข้อมูล coverage ต่ำ |
| RECON-002 | Result ต้องมี explanation |
| RECON-003 | Critical result ต้องสร้าง issue |
| RECON-004 | Matching key ต้อง configurable และปรับได้หลัง data profiling |

### MVP Requirement

Required in MVP

---

## 7.3 Explanation Engine

### Module ID

```text
EXPLANATION
```

### Purpose

แปลง result code และข้อมูลเปรียบเทียบเป็นคำอธิบายที่คนเข้าใจ

### Example

```text
หน้างานแจ้ง OUT 1,000 แต่ ERP ตัด OUT 950
Before/After Count สนับสนุนว่าออกจริง 1,000
ระบบจึงประเมินว่า ERP อาจ post ขาด 50
```

### Output

- explanation text
- likely cause
- suggested action
- owner role

### MVP Requirement

Required in MVP

---

## 7.4 Issue Engine

### Module ID

```text
ISSUE_ENGINE
```

### Purpose

สร้าง issue จาก reconciliation result หรือ rule failure

### Issue Status

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

### Business Rules

| Rule | Description |
|---|---|
| IE-001 | Critical result ต้องสร้าง issue |
| IE-002 | Issue ต้องมี owner_role |
| IE-003 | Issue ต้องมี status |
| IE-004 | Issue open ห้ามถูก retention delete |
| IE-005 | Closing issue ต้องมี reason/action result |

### MVP Requirement

Required in MVP

---

# 8. Decision UI Modules

---

## 8.1 Control Center Module

### Module ID

```text
CONTROL_CENTER
```

### Purpose

หน้ารวมสถานะระบบสำหรับ Admin/Supervisor

### Main Cards

- Matched Count
- Issue Count
- Critical Issue
- Unknown Alias
- Pending Inbound
- REJ Not Posted
- Export Pending
- Close Readiness
- Retention Status
- DB/Health Status

### Data Source

- summary tables
- issues
- recon_results
- retention_logs
- import_batches

### Business Rules

| Rule | Description |
|---|---|
| CC-001 | Control Center ห้าม query raw full history |
| CC-002 | ต้องมี drill down ไป detail |
| CC-003 | ต้องแสดง close readiness |

### MVP Requirement

Required in MVP

---

## 8.2 Issue Center Module

### Module ID

```text
ISSUE_CENTER
```

### Purpose

หน้าให้ผู้ใช้ดูและจัดการ exception

### Filters

- business date
- warehouse
- result code
- severity
- confidence
- status
- owner role
- assigned user

### Actions

- view detail
- assign
- create recount task
- send to ERP Admin
- accept with reason
- close
- export

### MVP Requirement

Required in MVP

---

## 8.3 Export Center Module

### Module ID

```text
EXPORT_CENTER
```

### Purpose

ส่งออกข้อมูลหลายรูปแบบและเก็บ log

### Export Types

- Issue List
- Daily Reconciliation
- REJ Report
- ERP Correction Template
- Count Task
- Export Log
- Retention Report

### Formats

- CSV
- Excel
- PDF/Print View
- ERP Template

### Business Rules

| Rule | Description |
|---|---|
| EXP-001 | Export ทุกครั้งต้องมี export log |
| EXP-002 | Export selected rows และ filtered rows ต้องรองรับ |
| EXP-003 | User ต้องมี CAN_EXPORT |
| EXP-004 | Export หลัง locked day ต้อง trace version ได้ |

### MVP Requirement

Basic export required in MVP

---

## 8.4 Notification Center Module

### Module ID

```text
NOTIFICATION_CENTER
```

### Purpose

ตั้งค่าแจ้งเตือนผ่าน In-app, Telegram, LINE Messaging API, Email

### Tabs

- Channels
- Rules
- Templates
- Test Send
- Logs

### Event Codes

- USER_PENDING_APPROVAL
- UNKNOWN_ALIAS_CREATED
- PENDING_INBOUND_CREATED
- SYSTEM_MISSING
- QTY_DIFF
- LOCATION_DIFF
- REJ_NOT_POSTED
- EXPORT_READY
- RETENTION_FAILED
- DB_USAGE_WARNING

### Business Rules

| Rule | Description |
|---|---|
| NOTI-001 | Notification ต้องไม่ hardcode ใน business service |
| NOTI-002 | ต้องมี cooldown |
| NOTI-003 | ต้องมี log |
| NOTI-004 | ต้องมี send once per issue option |
| NOTI-005 | Critical alert ส่งทันที, low alert อาจเป็น digest |

### MVP Requirement

Basic notification required in MVP

---

# 9. Maintenance & Governance Modules

---

## 9.1 Audit Log Module

### Module ID

```text
AUDIT_LOG
```

### Purpose

เก็บหลักฐานการกระทำสำคัญทั้งหมด

### Must Log

- User approval/reject
- Permission change
- Import confirm
- Alias mapping
- Stock replay
- Reconciliation run
- Issue close
- Export
- Notification rule change
- Retention delete
- Close/Lock day

### MVP Requirement

Required in MVP

---

## 9.2 Retention Module

### Module ID

```text
RETENTION
```

### Purpose

ลบ/archive ข้อมูลเกิน 45 วันโดยปลอดภัย

### Rules

```text
Archive summary before delete raw
Skip open issue
Skip pending approval/export
Keep critical audit/export log
Write retention log
Notify admin
```

### MVP Requirement

Required in MVP

---

## 9.3 Health Check Module

### Module ID

```text
HEALTH_CHECK
```

### Purpose

ให้ระบบเช็คตัวเองและแจ้งเมื่อมี risk

### Checks

- DB usage
- row count by table
- last import
- last replay
- last reconciliation
- last retention
- open critical issue
- pending alias
- pending inbound

### MVP Requirement

Recommended in MVP if using Supabase/free tier

---

# 10. Future Modules

---

## 10.1 Capacity Module

### Purpose

จัดการ capacity ของ location

### Future Scope

- max capacity
- current occupancy
- reserved qty
- planned in/out
- capacity warning

### Not MVP

ทำหลัง stock replay/reconciliation stable

---

## 10.2 Simulation Module

### Purpose

หา available location และ simulation รับเข้า

### Future Scope

- lot policy
- customer type policy
- capacity policy
- candidate scoring

### Not MVP

---

## 10.3 Relocation Optimizer Module

### Purpose

เสนอให้ย้ายของน้อยไปหาของมากเพื่อเปิดพื้นที่

### Future Scope

- move small to large
- movement cost
- freed location
- approval required

### Not MVP

---

## 10.4 Custody Module

### Purpose

รองรับของฝาก/ของข้ามคลัง

### Future Scope

- stock_owner
- physical_keeper
- ERP owner
- custody report

### Not MVP

---

# 11. Module Build Priority

## Phase 1 — Foundation

1. Auth
2. User Approval
3. Permission
4. Audit Log
5. Import Staging
6. Column Mapping
7. Alias Management

## Phase 2 — Data Engine

1. Snapshot Control
2. Movement Ledger
3. In-Out Backfill
4. Stock Replay
5. Daily Stock Position
6. Data Coverage Check

## Phase 3 — Intelligence

1. Rule Engine
2. Reconciliation Engine
3. Explanation Engine
4. Issue Engine
5. Checker Task

## Phase 4 — Decision & Control

1. Control Center
2. Issue Center
3. Export Center
4. Notification Center
5. Lock/Close Control
6. Retention

## Phase 5 — Advanced

1. Capacity
2. Simulation
3. Relocation Optimizer
4. Custody
5. Advanced FIFO

---

# 12. Module Acceptance Criteria

ระบบถือว่า module spec นี้ถูก implement ได้ดี ถ้า:

- [ ] แต่ละ module แยก responsibility ชัดเจน
- [ ] import ไม่เขียนตรง canonical table โดยไม่ staging
- [ ] alias แยก type ชัดเจน
- [ ] permission ตรวจ backend ทุก action สำคัญ
- [ ] movement ทุกชนิดเข้า ledger
- [ ] stock replay ใช้ anchor + ledger
- [ ] reconciliation result มี explanation
- [ ] issue มี owner/status/action
- [ ] notification มี rule/template/log/cooldown
- [ ] export มี export log
- [ ] retention ไม่ลบ open issue
- [ ] audit log มีครบทุก critical action

---

# 13. Final Module Statement

```text
ระบบนี้ต้องสร้างจาก module เล็กที่มีหน้าที่ชัดเจน โดยเริ่มจาก foundation: user, permission, import, alias, audit แล้วค่อยสร้าง data engine: anchor, ledger, replay จากนั้นจึงสร้าง intelligence: reconciliation, explanation, issue และสุดท้ายสร้าง decision UI: control center, export, notification, retention เพื่อให้ระบบทำงานมั่นคงและขยายต่อได้โดยไม่รื้อใหม่
```

