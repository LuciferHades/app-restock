# 10_data_model.md

# Data Model Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Data Model / Entity Relationship / Table Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / Database Designer Use  
**Primary Use:** ใช้กำหนดโครงสร้างข้อมูลหลักของระบบ ทั้ง master, transaction, import staging, movement ledger, stock replay, reconciliation, issue, export, notification, audit และ retention เพื่อให้ระบบสร้างได้จริงโดยไม่ปนความหมายของข้อมูลแต่ละแหล่ง

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Data Model ของ Warehouse Reconciliation ERP Platform

ระบบนี้มีข้อมูลหลายแหล่งที่มีความหมายไม่เหมือนกัน:

```text
Inquiry              = ERP Snapshot
System In-Out        = ERP Posted Movement
Field Movement       = หน้างานแจ้งว่าเกิดขึ้น
Physical Count       = ของจริงที่นับได้
Expected Stock       = ระบบคำนวณจาก Anchor + Ledger
Reconciliation Result = ผลตรวจว่าตรง/ไม่ตรง
Issue                = งานที่คนต้องแก้หรือตัดสินใจ
```

ดังนั้น Data Model ต้องแยกข้อมูลแต่ละชั้นอย่างชัดเจน ห้ามเอาทุกอย่างไปรวมเป็นตารางเดียว เพราะจะทำให้:

- ยอดเบิ้ล
- ยอดหาย
- audit ย้อนหลังไม่ได้
- replay stock ข้ามวันไม่ได้
- dashboard ช้า
- retention 45 วันลบผิด
- reconciliation อธิบายไม่ได้

หลักสำคัญ:

```text
Raw data ต้องเก็บเป็น raw/staging ก่อน
Canonical data ต้อง normalize จาก mapping/alias
Movement ทุกชนิดที่มีผลต่อ stock ต้องเข้า Movement Ledger
Stock Position ต้องสร้างจาก Anchor Snapshot + Ledger
Issue ต้องเกิดจาก Result ที่อธิบายได้
```

---

## 2. Data Model Principles

## 2.1 Separate Truth Layers

ห้ามปนความจริงแต่ละชั้น

| Truth Layer | Table Group | Meaning |
|---|---|---|
| Raw Truth | import_batches / import_raw_rows | ข้อมูลดิบจากไฟล์ก่อนตีความ |
| Master Truth | customers / items / locations | master ที่ระบบใช้จริง |
| Alias Truth | customer_aliases / item_aliases | mapping จากชื่อดิบไป canonical |
| ERP Snapshot Truth | inquiry_snapshots | ยอดคงเหลือ ERP ณ วัน/เวลา |
| ERP Movement Truth | system_inout_records | movement ที่ ERP post แล้ว |
| Field Truth | field_movements | หน้างานแจ้ง movement |
| Physical Truth | physical_counts | ของจริงที่นับได้ |
| Calculation Truth | daily_stock_positions | ยอดที่ระบบคำนวณ |
| Decision Truth | recon_results / issues | ผลตรวจและสิ่งที่ต้อง action |
| Governance Truth | audit_logs / export_logs / retention_logs | ตรวจย้อนหลังและควบคุม |

---

## 2.2 Raw First, Normalize Later, Canonical Last

Flow data ที่ถูกต้อง:

```text
File Upload
→ import_batches
→ import_raw_rows
→ column mapping
→ alias mapping
→ normalized records
→ canonical tables / ledger / snapshot
```

ห้าม:

```text
File Upload
→ insert เข้า stock/movement จริงทันที
```

---

## 2.3 Every Important Record Must Have Source

ข้อมูลที่มาจากไฟล์หรือ user action ต้อง trace กลับได้

ควรมี field เช่น:

- source_type
- source_ref_id
- batch_id
- raw_row_id
- created_by
- created_at

---

## 2.4 Do Not Delete Critical History Blindly

ระบบมี retention 45 วัน แต่ห้ามลบข้อมูลสำคัญโดยไม่มี summary/audit

ลบ raw ได้เมื่อ:

- normalized แล้ว
- ไม่มี open issue
- summary/archive แล้ว
- retention log ถูกเขียนแล้ว

---

## 2.5 Use Status, Not Only Boolean

ทุก entity ที่มี lifecycle ต้องใช้ status ไม่ใช่ boolean เดียว

ตัวอย่าง:

```text
Import Batch: UPLOADED / MAPPED / NORMALIZED / CONFIRMED / FAILED
Issue: OPEN / ASSIGNED / IN_REVIEW / RESOLVED / CLOSED
Day: OPEN / PRELIM_CLOSED / RECONSTRUCTED / FINAL_CLOSED / LOCKED
```

---

# 3. Naming Convention

## 3.1 Table Naming

ใช้ snake_case

ตัวอย่าง:

```text
users
user_access_requests
import_batches
movement_ledger
daily_stock_positions
recon_results
```

---

## 3.2 Primary Key Naming

ใช้รูปแบบ:

```text
<table_singular>_id
```

ตัวอย่าง:

```text
user_id
batch_id
movement_id
ledger_id
issue_id
```

---

## 3.3 Foreign Key Naming

ใช้ชื่อ key จาก table ต้นทาง

ตัวอย่าง:

```text
warehouse_id
location_id
customer_id
item_id
```

---

## 3.4 Timestamp Fields

ทุก table สำคัญควรมี:

```text
created_at
created_by
updated_at
updated_by
```

ถ้าเป็น system-generated record ให้ created_by = SYSTEM หรือ system_user_id

---

## 3.5 Status Fields

ใช้:

```text
status
active_flag
```

- `status` ใช้บอก lifecycle
- `active_flag` ใช้เปิด/ปิดการใช้งาน

---

# 4. Entity Groups

ระบบแบ่ง table เป็นกลุ่มหลัก:

```text
1. User & Permission
2. Master Data
3. Alias & Review
4. Import & Staging
5. ERP Snapshot / Inquiry
6. ERP Movement / System In-Out
7. Operational Field Data
8. Movement Ledger
9. Stock Replay / Daily Position
10. Reconciliation Result
11. Issue / Task / Investigation
12. Export
13. Notification
14. Audit / Security
15. Retention / Maintenance
16. Summary / Dashboard
17. Future Capacity / Simulation
```

---

# 5. User & Permission Tables

## 5.1 users

### Purpose

เก็บบัญชีผู้ใช้ที่ได้รับอนุมัติแล้ว

### Fields

| Field | Type | Required | Description |
|---|---|---:|---|
| user_id | uuid/text | Yes | primary key |
| username | text | Yes | unique username |
| email | text | No | optional, unique if used |
| phone | text | No | phone number |
| full_name | text | Yes | ชื่อผู้ใช้ |
| hashed_password | text | Yes | password hash |
| salt | text | Yes | password salt |
| status | text | Yes | ACTIVE / LOCKED / SUSPENDED / EXPIRED / DELETED |
| last_login_at | datetime | No | login ล่าสุด |
| created_at | datetime | Yes | วันที่สร้าง |
| created_by | text | No | admin/system |
| updated_at | datetime | No | แก้ไขล่าสุด |
| updated_by | text | No | ผู้แก้ไข |

### Rules

- ห้ามเก็บ plain password
- Pending user ยังไม่อยู่ใน users หรืออยู่แต่ status ต้องไม่ active ตาม implementation
- Locked/Suspended user ห้าม login

---

## 5.2 user_access_requests

### Purpose

เก็บคำขอสมัครที่รอ Admin อนุมัติ

### Fields

| Field | Type | Required | Description |
|---|---|---:|---|
| request_id | uuid/text | Yes | primary key |
| username | text | Yes | requested username |
| email | text | No | email |
| phone | text | No | phone |
| full_name | text | Yes | full name |
| hashed_password | text | Yes | hash password ที่ตั้งไว้ |
| salt | text | Yes | salt |
| requested_role | text | No | role ที่ขอ |
| requested_warehouse | text | No | warehouse ที่ขอ |
| status | text | Yes | PENDING_APPROVAL / APPROVED / REJECTED |
| requested_at | datetime | Yes | วันที่สมัคร |
| approved_by | text | No | admin approver |
| approved_at | datetime | No | เวลา approve |
| rejected_by | text | No | admin rejecter |
| rejected_at | datetime | No | เวลา reject |
| rejection_reason | text | No | เหตุผล reject |

### Rules

- Reject แล้ว request อาจถูกลบจาก queue แต่ต้องมี audit log
- Approve ต้องสร้าง user/role/permission/scope

---

## 5.3 roles

| Field | Type | Description |
|---|---|---|
| role_id | uuid/text | primary key |
| role_code | text | ADMIN / SUPERVISOR / CHECKER / ERP_ADMIN / VIEWER |
| role_name | text | display name |
| description | text | คำอธิบาย |
| active_flag | boolean | ใช้งานอยู่ไหม |
| created_at | datetime |  |
| updated_at | datetime |  |

---

## 5.4 permissions

| Field | Type | Description |
|---|---|---|
| permission_id | uuid/text | primary key |
| permission_code | text | เช่น CAN_IMPORT_INOUT |
| permission_name | text | display name |
| module_code | text | module ที่เกี่ยวข้อง |
| description | text | คำอธิบาย |
| active_flag | boolean | ใช้งานอยู่ไหม |

---

## 5.5 role_permissions

| Field | Type | Description |
|---|---|---|
| role_permission_id | uuid/text | primary key |
| role_id | uuid/text | FK roles |
| permission_id | uuid/text | FK permissions |
| active_flag | boolean | active |
| created_at | datetime |  |
| created_by | text |  |

---

## 5.6 user_roles

| Field | Type | Description |
|---|---|---|
| user_role_id | uuid/text | primary key |
| user_id | uuid/text | FK users |
| role_id | uuid/text | FK roles |
| active_flag | boolean | active |
| assigned_by | text | admin |
| assigned_at | datetime |  |

---

## 5.7 user_permission_overrides

### Purpose

เปิด/ปิดสิทธิ์เฉพาะ user เพิ่มจาก role

| Field | Type | Description |
|---|---|---|
| override_id | uuid/text | primary key |
| user_id | uuid/text | FK users |
| permission_id | uuid/text | FK permissions |
| override_type | text | ALLOW / DENY |
| reason | text | เหตุผล |
| active_flag | boolean | active |
| assigned_by | text | admin |
| assigned_at | datetime |  |

---

## 5.8 user_warehouse_scope

### Purpose

จำกัดว่าผู้ใช้เห็นหรือทำงานกับ warehouse/location ไหนได้

| Field | Type | Description |
|---|---|---|
| scope_id | uuid/text | primary key |
| user_id | uuid/text | FK users |
| scope_type | text | ALL_WAREHOUSES / WAREHOUSE / ZONE / LOCATION_GROUP / OWN_TASK_ONLY |
| warehouse_id | uuid/text | nullable if all |
| zone_id | uuid/text | optional |
| location_group_id | uuid/text | optional |
| active_flag | boolean | active |
| assigned_by | text | admin |
| assigned_at | datetime |  |

---

## 5.9 login_sessions

| Field | Type | Description |
|---|---|---|
| session_id | uuid/text | primary key |
| user_id | uuid/text | FK users |
| session_token_hash | text | token hash |
| device_id | text | optional |
| ip_address | text | optional |
| user_agent | text | optional |
| expires_at | datetime | token expiry |
| status | text | ACTIVE / EXPIRED / REVOKED |
| created_at | datetime |  |
| last_seen_at | datetime |  |

---

## 5.10 checker_daily_sessions

### Purpose

ให้ checker check-in ที่ warehouse ต่อวันก่อนนับ

| Field | Type | Description |
|---|---|---|
| checker_session_id | uuid/text | primary key |
| checker_user_id | uuid/text | FK users |
| business_date | date | วันที่ทำงาน |
| warehouse_id | uuid/text | warehouse active |
| checkin_lat | number | latitude |
| checkin_lng | number | longitude |
| checkin_accuracy | number | GPS accuracy |
| checkin_at | datetime | เวลา check-in |
| checkout_at | datetime | optional |
| device_id | text | optional |
| status | text | ACTIVE / SUSPENDED_BY_OTHER_WH_LOGIN / EXPIRED / CHECKED_OUT / MANUAL_SUSPENDED |
| active_flag | boolean | active |
| suspended_reason | text | nullable |
| last_seen_at | datetime |  |

### Rules

- 1 user มี active warehouse ได้ 1 ที่ต่อ business_date
- ถ้า check-in warehouse ใหม่ ต้อง suspend session เดิม

---

# 6. Master Data Tables

## 6.1 warehouses

| Field | Type | Description |
|---|---|---|
| warehouse_id | uuid/text | primary key |
| warehouse_code | text | canonical code เช่น WH_W8 |
| warehouse_name | text | ชื่อคลัง |
| dc_code | text | canonical DC if applicable |
| description | text |  |
| active_flag | boolean | active |
| created_at | datetime |  |
| updated_at | datetime |  |

---

## 6.2 locations

| Field | Type | Description |
|---|---|---|
| location_id | uuid/text | primary key |
| warehouse_id | uuid/text | FK warehouses |
| location_code | text | canonical location code |
| location_name | text | display name |
| zone_code | text | optional |
| location_type | text | NORMAL / REJ / STAGING / HOLD / EXTERNAL |
| capacity_qty | number | capacity base uom |
| capacity_uom | text | UOM ของ capacity |
| active_flag | boolean | active |
| created_at | datetime |  |
| updated_at | datetime |  |

### Rules

- Location ต้องผูก warehouse
- REJ location ควรมี location_type = REJ

---

## 6.3 customers

| Field | Type | Description |
|---|---|---|
| customer_id | uuid/text | primary key |
| customer_code | text | canonical code |
| customer_name | text | canonical name |
| customer_type | text | INTERNAL / EXTERNAL / UNKNOWN / OTHER |
| active_flag | boolean | active |
| created_at | datetime |  |
| updated_at | datetime |  |

---

## 6.4 items

| Field | Type | Description |
|---|---|---|
| item_id | uuid/text | primary key |
| item_code | text | canonical item code |
| item_name | text | canonical item name |
| product_group_id | uuid/text | FK product_groups |
| grade | text | optional เช่น W150 / RE T1 |
| pack_size | text | optional เช่น 50KG |
| default_uom | text | default UOM |
| active_flag | boolean | active |
| created_at | datetime |  |
| updated_at | datetime |  |

### Rules

- สินค้าแต่ละ product group อาจมี identity field ต่างกัน
- น้ำตาลอาจใช้ item + year + lot + pack
- ไม้/แป้ง/โมลาส/เทกอง ต้องรองรับ template ต่างกันใน phase ต่อไป

---

## 6.5 product_groups

| Field | Type | Description |
|---|---|---|
| product_group_id | uuid/text | primary key |
| product_group_code | text | SUGAR / WOOD / FLOUR / MOLASSES / BULK / OTHER |
| product_group_name | text | display |
| identity_template | json/text | optional rule/template |
| active_flag | boolean | active |

---

## 6.6 uoms

| Field | Type | Description |
|---|---|---|
| uom_id | uuid/text | primary key |
| uom_code | text | BAG / KG / TON |
| uom_name | text | display name |
| base_uom | text | optional |
| conversion_factor | number | optional |
| active_flag | boolean | active |

---

## 6.7 reasons

| Field | Type | Description |
|---|---|---|
| reason_id | uuid/text | primary key |
| reason_code | text | เช่น REJ_DAMAGE |
| reason_name | text | display |
| reason_type | text | REJ / ADJUSTMENT / CORRECTION / HOLD |
| active_flag | boolean | active |

---

# 7. Alias & Review Tables

## 7.1 customer_aliases

| Field | Type | Description |
|---|---|---|
| alias_id | uuid/text | primary key |
| alias_text | text | เช่น MKMK |
| customer_id | uuid/text | FK customers |
| confidence | number | default confidence |
| source_type | text | MANUAL / IMPORT / SMART_INPUT |
| active_flag | boolean | active |
| created_by | text |  |
| created_at | datetime |  |
| updated_at | datetime |  |

---

## 7.2 item_aliases

| Field | Type | Description |
|---|---|---|
| alias_id | uuid/text | primary key |
| alias_text | text | เช่น W150 / RE T1 |
| item_id | uuid/text | FK items |
| product_group_id | uuid/text | optional |
| confidence | number |  |
| source_type | text | MANUAL / IMPORT / SMART_INPUT |
| active_flag | boolean |  |
| created_by | text |  |
| created_at | datetime |  |

---

## 7.3 location_aliases

| Field | Type | Description |
|---|---|---|
| alias_id | uuid/text | primary key |
| alias_text | text | raw location/DC text |
| location_id | uuid/text | FK locations |
| warehouse_id | uuid/text | FK warehouses |
| confidence | number |  |
| source_type | text |  |
| active_flag | boolean |  |
| created_at | datetime |  |

---

## 7.4 dc_aliases

| Field | Type | Description |
|---|---|---|
| alias_id | uuid/text | primary key |
| alias_text | text | เช่น DC08 / W8 |
| warehouse_id | uuid/text | FK warehouses |
| confidence | number |  |
| active_flag | boolean |  |
| created_at | datetime |  |

---

## 7.5 uom_aliases

| Field | Type | Description |
|---|---|---|
| alias_id | uuid/text | primary key |
| alias_text | text | ถุง / BAG / Bags |
| uom_id | uuid/text | FK uoms |
| conversion_factor | number | optional |
| active_flag | boolean | active |

---

## 7.6 alias_review_queue

### Purpose

เก็บ unknown alias ที่ต้องให้ Admin review

| Field | Type | Description |
|---|---|---|
| queue_id | uuid/text | primary key |
| alias_type | text | CUSTOMER / ITEM / LOCATION / DC / UOM / LOT_PATTERN |
| raw_text | text | raw unknown text |
| normalized_text | text | cleaned text |
| suggested_master_id | text | nullable |
| suggested_master_type | text | CUSTOMER / ITEM / LOCATION etc |
| confidence | number | 0–100 |
| source_type | text | IMPORT / SMART_INPUT / RECON |
| source_batch_id | uuid/text | nullable |
| source_row_no | number | nullable |
| occurrence_count | number | จำนวนครั้งที่เจอ |
| status | text | PENDING / MAPPED / CREATED_NEW / REJECTED / DEFERRED |
| reviewed_by | text | user_id |
| reviewed_at | datetime |  |
| review_note | text | optional |
| created_at | datetime |  |

### Rules

- Low confidence ห้าม auto-map
- ต้อง trace กลับ source batch/row ได้

---

# 8. Import & Staging Tables

## 8.1 import_batches

### Purpose

เก็บ metadata ของไฟล์ที่ upload ทุกครั้ง

| Field | Type | Description |
|---|---|---|
| batch_id | uuid/text | primary key |
| file_type | text | INQUIRY / SYSTEM_INOUT / FIELD_COUNT / MASTER_DATA |
| file_name | text | original file name |
| file_size | number | optional |
| file_checksum | text | ใช้จับ duplicate |
| business_date_from | date | date range |
| business_date_to | date | date range |
| warehouse_id | uuid/text | optional |
| uploaded_by | text | user_id |
| uploaded_at | datetime |  |
| row_count | number | จำนวน row |
| status | text | UPLOADED / MAPPED / NORMALIZED / CONFIRMED / FAILED / CANCELLED |
| error_count | number |  |
| warning_count | number |  |
| notes | text | optional |

---

## 8.2 import_raw_rows

### Purpose

เก็บ raw data ราย row ก่อน normalize

| Field | Type | Description |
|---|---|---|
| raw_row_id | uuid/text | primary key |
| batch_id | uuid/text | FK import_batches |
| row_no | number | row number |
| raw_json | json/text | raw row data |
| parse_status | text | PENDING / PARSED / ERROR / NORMALIZED |
| error_message | text | nullable |
| created_at | datetime |  |

### Retention

ลบได้หลัง normalized/closed และเกิน 45 วัน ถ้าไม่มี open issue อ้างอิง

---

## 8.3 import_column_mappings

| Field | Type | Description |
|---|---|---|
| mapping_id | uuid/text | primary key |
| file_type | text | INQUIRY / SYSTEM_INOUT etc |
| template_name | text | optional |
| source_column_name | text | header จากไฟล์ |
| canonical_field | text | field กลาง |
| required_flag | boolean | required? |
| active_flag | boolean | active |
| created_by | text |  |
| created_at | datetime |  |
| updated_at | datetime |  |

---

## 8.4 import_error_logs

| Field | Type | Description |
|---|---|---|
| error_id | uuid/text | primary key |
| batch_id | uuid/text | FK import_batches |
| raw_row_id | uuid/text | nullable |
| error_code | text | MISSING_COLUMN / INVALID_QTY etc |
| error_message | text |  |
| severity | text | WARNING / ERROR / CRITICAL |
| created_at | datetime |  |

---

# 9. ERP Snapshot / Inquiry Tables

## 9.1 inquiry_snapshots

### Purpose

เก็บ Inquiry เป็น Anchor Snapshot

| Field | Type | Description |
|---|---|---|
| snapshot_id | uuid/text | primary key |
| batch_id | uuid/text | FK import_batches |
| warehouse_id | uuid/text | FK warehouses |
| business_date | date | วันที่ข้อมูล snapshot |
| snapshot_time | time/datetime | optional |
| uploaded_by | text | user_id |
| uploaded_at | datetime |  |
| checksum | text | duplicate detection |
| active_flag | boolean | active snapshot |
| status | text | ACTIVE / ARCHIVED / REPLACED / CANCELLED |
| replaced_by_snapshot_id | uuid/text | nullable |
| notes | text | optional |

### Rules

- Inquiry ไม่ใช่ movement
- 1 business_date อาจมีหลาย snapshot แต่ควรมี active policy ชัดเจน

---

## 9.2 inquiry_snapshot_lines

### Purpose

เก็บ line stock balance จาก Inquiry

| Field | Type | Description |
|---|---|---|
| snapshot_line_id | uuid/text | primary key |
| snapshot_id | uuid/text | FK inquiry_snapshots |
| raw_row_id | uuid/text | FK import_raw_rows nullable after retention |
| warehouse_id | uuid/text | canonical |
| location_id | uuid/text | canonical |
| customer_id | uuid/text | canonical or unknown |
| item_id | uuid/text | canonical |
| product_group_id | uuid/text | optional |
| year | text | optional |
| lot | text | optional |
| qty | number | stock qty |
| uom | text | UOM |
| raw_customer_text | text | original |
| raw_item_text | text | original |
| raw_location_text | text | original |
| confidence | text | HIGH/MEDIUM/LOW |
| status | text | NORMALIZED / UNKNOWN_ALIAS / ERROR |

---

# 10. ERP Movement / System In-Out Tables

## 10.1 system_inout_batches

Optional หากใช้ import_batches อยู่แล้ว อาจไม่ต้องแยก แต่แนะนำถ้าต้องเก็บ metadata เฉพาะ In-Out

| Field | Type | Description |
|---|---|---|
| inout_batch_id | uuid/text | primary key |
| batch_id | uuid/text | FK import_batches |
| business_date_from | date | range |
| business_date_to | date | range |
| warehouse_id | uuid/text | optional |
| status | text | NORMALIZED / FAILED / CONFIRMED |
| created_at | datetime |  |

---

## 10.2 system_inout_records

### Purpose

เก็บ movement ที่ ERP post แล้วจาก System In-Out

| Field | Type | Description |
|---|---|---|
| system_inout_id | uuid/text | primary key |
| batch_id | uuid/text | FK import_batches |
| raw_row_id | uuid/text | FK import_raw_rows nullable |
| business_date | date | วันที่ business movement |
| posting_date | date | วันที่ ERP post optional |
| movement_datetime | datetime | optional |
| source_ref | text | document/order/reference |
| movement_type | text | IN / OUT / TRANSFER / REJ / ADJUSTMENT |
| warehouse_id | uuid/text | canonical |
| from_location_id | uuid/text | nullable |
| to_location_id | uuid/text | nullable |
| location_id | uuid/text | nullable for simple movement |
| customer_id | uuid/text | canonical |
| item_id | uuid/text | canonical |
| product_group_id | uuid/text | optional |
| year | text | optional |
| lot | text | optional |
| qty | number | positive qty |
| uom | text | uom |
| raw_customer_text | text | original |
| raw_item_text | text | original |
| raw_location_text | text | original |
| normalize_status | text | NORMALIZED / UNKNOWN_ALIAS / ERROR |
| duplicate_key | text | optional |
| created_at | datetime |  |

### Rules

- System In-Out คือ ERP posted movement
- ต้องสร้าง movement_ledger source_type = SYSTEM_INOUT

---

# 11. Operational Field Data Tables

## 11.1 field_movements

### Purpose

เก็บ movement ที่หน้างานแจ้ง

| Field | Type | Description |
|---|---|---|
| movement_id | uuid/text | primary key |
| business_date | date | วันที่ operation |
| movement_datetime | datetime | เวลาที่แจ้ง/เกิด |
| reported_by | text | user_id |
| warehouse_id | uuid/text | warehouse |
| movement_type | text | IN / OUT / TRANSFER / REJ / ADJUSTMENT |
| from_location_id | uuid/text | optional |
| to_location_id | uuid/text | optional |
| location_id | uuid/text | optional simple movement |
| customer_id | uuid/text | nullable for pending |
| item_id | uuid/text | nullable for pending |
| product_group_id | uuid/text | optional |
| year | text | optional |
| lot | text | optional |
| qty | number | positive |
| uom | text |  |
| source_text | text | text from label/form |
| photo_url | text | optional |
| status | text | DRAFT / REPORTED / WAIT_SYSTEM_POST / MATCHED / PROBLEM / INVESTIGATING / CLOSED / CANCELLED |
| confidence | text | HIGH/MEDIUM/LOW |
| created_at | datetime |  |
| updated_at | datetime |  |

---

## 11.2 pending_inbounds

### Purpose

เก็บของเข้าที่ข้อมูลยังไม่ครบ

| Field | Type | Description |
|---|---|---|
| pending_inbound_id | uuid/text | primary key |
| business_date | date | วันที่เข้า |
| reported_by | text | user_id |
| warehouse_id | uuid/text |  |
| location_id | uuid/text |  |
| qty | number |  |
| uom | text |  |
| text_on_label | text | ข้อความป้าย |
| photo_url | text | optional |
| parsed_customer_alias | text | optional |
| parsed_item_alias | text | optional |
| parsed_year | text | optional |
| parsed_lot | text | optional |
| status | text | PENDING_INBOUND_VERIFY / WAIT_SYSTEM_INOUT / CANDIDATE_FOUND / CONFIRMED / REJECTED / NEED_INVESTIGATION |
| confirmed_movement_id | uuid/text | nullable |
| created_at | datetime |  |
| updated_at | datetime |  |

---

## 11.3 pending_inbound_candidates

| Field | Type | Description |
|---|---|---|
| candidate_id | uuid/text | primary key |
| pending_inbound_id | uuid/text | FK pending_inbounds |
| system_inout_id | uuid/text | FK system_inout_records |
| match_score | number | 0–100 |
| match_reason | text | เช่น date/location/qty/text |
| status | text | PROPOSED / CONFIRMED / REJECTED |
| reviewed_by | text | nullable |
| reviewed_at | datetime | nullable |

---

## 11.4 physical_counts

### Purpose

เก็บของจริงที่นับได้

| Field | Type | Description |
|---|---|---|
| count_id | uuid/text | primary key |
| business_date | date | วันที่นับ |
| count_datetime | datetime | เวลานับ |
| counted_by | text | checker user_id |
| checker_session_id | uuid/text | FK checker_daily_sessions |
| task_id | uuid/text | optional |
| warehouse_id | uuid/text |  |
| location_id | uuid/text |  |
| customer_id | uuid/text | optional |
| item_id | uuid/text | optional |
| product_group_id | uuid/text | optional |
| year | text | optional |
| lot | text | optional |
| qty_counted | number | qty |
| uom | text |  |
| count_type | text | BULK_COUNT / BEFORE_COUNT / AFTER_COUNT / RECOUNT / LOT_VERIFY / PARTIAL_VERIFY |
| photo_url | text | optional |
| remark | text | optional |
| status | text | SUBMITTED / VERIFIED / REJECTED / CANCELLED |
| created_at | datetime |  |

---

## 11.5 rej_notices

| Field | Type | Description |
|---|---|---|
| rej_id | uuid/text | primary key |
| business_date | date | วันที่แจ้ง REJ |
| reported_by | text | user_id |
| warehouse_id | uuid/text |  |
| from_location_id | uuid/text |  |
| rej_location_id | uuid/text | destination REJ |
| customer_id | uuid/text |  |
| item_id | uuid/text |  |
| year | text | optional |
| lot | text | optional |
| qty | number |  |
| uom | text |  |
| reason_id | uuid/text | FK reasons |
| photo_url | text | optional |
| status | text | REPORTED / WAIT_SYSTEM_POST / REJ_MATCHED / REJ_NOT_POSTED / REJ_QTY_DIFF / CLOSED |
| created_at | datetime |  |
| updated_at | datetime |  |

---

# 12. Movement Ledger Tables

## 12.1 movement_ledger

### Purpose

เป็น ledger กลางของ movement ทุก source ที่มีผลต่อ stock

| Field | Type | Description |
|---|---|---|
| ledger_id | uuid/text | primary key |
| business_date | date | วันที่ business |
| movement_datetime | datetime | เวลา movement |
| source_type | text | SYSTEM_INOUT / FIELD_NOTICE / REJ_NOTICE / TRANSFER / ADJUSTMENT / COUNT_CORRECTION / PENDING_INBOUND_CONFIRM |
| source_ref_id | uuid/text | id ของ source record |
| movement_type | text | IN / OUT / TRANSFER_IN / TRANSFER_OUT / REJ / ADJUSTMENT_IN / ADJUSTMENT_OUT |
| warehouse_id | uuid/text |  |
| from_location_id | uuid/text | nullable |
| to_location_id | uuid/text | nullable |
| location_id | uuid/text | nullable simple movement |
| customer_id | uuid/text | nullable |
| item_id | uuid/text | nullable |
| product_group_id | uuid/text | optional |
| year | text | optional |
| lot | text | optional |
| qty | number | positive qty |
| uom | text |  |
| stock_effect_sign | number | +1 / -1 / 0 depending row design |
| confidence | text | HIGH / MEDIUM / LOW / CRITICAL |
| status | text | ACTIVE / CANCELLED / SUPERSEDED |
| created_at | datetime |  |

### Rules

- ทุก movement ที่มีผลต่อ stock ต้องเข้า ledger
- ledger ต้อง trace กลับ source ได้
- ห้ามแก้ ledger ย้อนหลังโดยไม่มี correction record

---

# 13. Stock Replay / Daily Position Tables

## 13.1 stock_replay_jobs

| Field | Type | Description |
|---|---|---|
| replay_job_id | uuid/text | primary key |
| date_from | date | range |
| date_to | date | range |
| warehouse_id | uuid/text | optional |
| triggered_by | text | user/system |
| trigger_type | text | IMPORT / BACKFILL / MANUAL / CORRECTION |
| status | text | QUEUED / RUNNING / SUCCESS / WARNING / FAILED |
| started_at | datetime |  |
| finished_at | datetime |  |
| rows_processed | number |  |
| warning_count | number |  |
| error_count | number |  |
| message | text |  |

---

## 13.2 daily_stock_positions

### Purpose

เก็บยอด stock position ที่ระบบคำนวณรายวัน

| Field | Type | Description |
|---|---|---|
| stock_position_id | uuid/text | primary key |
| business_date | date | วันที่ |
| warehouse_id | uuid/text |  |
| location_id | uuid/text |  |
| customer_id | uuid/text | nullable |
| item_id | uuid/text |  |
| product_group_id | uuid/text | optional |
| year | text | optional |
| lot | text | optional |
| opening_qty | number | opening |
| in_qty | number | inbound qty |
| out_qty | number | outbound qty |
| transfer_in_qty | number | transfer in |
| transfer_out_qty | number | transfer out |
| rej_qty | number | rej qty |
| adjustment_qty | number | +/- adjustment |
| ending_expected_qty | number | expected ending |
| uom | text |  |
| source_mode | text | FROM_INQUIRY / RECONSTRUCTED_FROM_LEDGER / ESTIMATED / COUNT_CONFIRMED |
| confidence | text | HIGH / MEDIUM / LOW / CRITICAL |
| anchor_snapshot_id | uuid/text | used anchor |
| replay_job_id | uuid/text | FK stock_replay_jobs |
| last_rebuilt_at | datetime |  |

### Rules

- Dashboard ควรอ่านจากตารางนี้/summary ไม่ใช่ raw ledger ทุกครั้ง
- Rebuild affected range เมื่อ movement/correction เปลี่ยน

---

## 13.3 data_coverage_checks

| Field | Type | Description |
|---|---|---|
| coverage_id | uuid/text | primary key |
| date_from | date |  |
| date_to | date |  |
| warehouse_id | uuid/text | optional |
| anchor_status | text | OK / MISSING |
| inout_status | text | OK / MISSING / PARTIAL |
| alias_status | text | OK / UNKNOWN_PENDING |
| pending_inbound_status | text | OK / PENDING |
| rej_status | text | OK / PENDING |
| count_status | text | OK / MISSING_FOR_MOVING_LOCATION |
| overall_status | text | COVERAGE_OK / WARNING / CRITICAL |
| details_json | json/text | details |
| checked_at | datetime |  |
| checked_by | text | system/user |

---

# 14. Reconciliation Tables

## 14.1 reconciliation_jobs

| Field | Type | Description |
|---|---|---|
| recon_job_id | uuid/text | primary key |
| date_from | date |  |
| date_to | date |  |
| warehouse_id | uuid/text | optional |
| triggered_by | text | user/system |
| trigger_type | text | IMPORT / BACKFILL / MANUAL / SCHEDULED |
| status | text | QUEUED / RUNNING / SUCCESS / WARNING / FAILED |
| started_at | datetime |  |
| finished_at | datetime |  |
| rows_checked | number |  |
| matched_count | number |  |
| issue_count | number |  |
| warning_count | number |  |
| error_count | number |  |
| message | text |  |

---

## 14.2 recon_results

### Purpose

เก็บผลการเทียบข้อมูลและคำอธิบาย

| Field | Type | Description |
|---|---|---|
| result_id | uuid/text | primary key |
| recon_job_id | uuid/text | FK reconciliation_jobs |
| business_date | date |  |
| warehouse_id | uuid/text |  |
| location_id | uuid/text | nullable |
| customer_id | uuid/text | nullable |
| item_id | uuid/text | nullable |
| product_group_id | uuid/text | optional |
| year | text | optional |
| lot | text | optional |
| source_ref_id | uuid/text | main source id |
| source_ref_type | text | FIELD / SYSTEM / COUNT / REJ / STOCK_POSITION |
| result_code | text | MATCHED / SYSTEM_MISSING / QTY_DIFF etc |
| severity | text | CRITICAL / HIGH / MEDIUM / LOW |
| confidence | text | HIGH / MEDIUM / LOW / CRITICAL |
| field_qty | number | nullable |
| system_qty | number | nullable |
| count_qty | number | nullable |
| expected_qty | number | nullable |
| diff_qty | number | nullable |
| uom | text |  |
| explanation | text | human-readable |
| likely_cause | text | reason |
| suggested_action | text | action |
| owner_role | text | ERP_ADMIN / SUPERVISOR / ADMIN / CHECKER |
| status | text | OPEN / NO_ISSUE / ISSUE_CREATED / CLOSED |
| created_at | datetime |  |

---

## 14.3 recon_result_links

### Purpose

เชื่อม result กับ source records หลายตัว

| Field | Type | Description |
|---|---|---|
| link_id | uuid/text | primary key |
| result_id | uuid/text | FK recon_results |
| source_type | text | FIELD_MOVEMENT / SYSTEM_INOUT / PHYSICAL_COUNT / STOCK_POSITION / REJ_NOTICE |
| source_id | uuid/text | record id |
| relation_type | text | PRIMARY / MATCHED_WITH / COMPARED_TO / EVIDENCE |

---

# 15. Issue / Task / Investigation Tables

## 15.1 issues

### Purpose

เก็บ exception ที่คนต้อง action

| Field | Type | Description |
|---|---|---|
| issue_id | uuid/text | primary key |
| result_id | uuid/text | FK recon_results |
| issue_type | text | SYSTEM_MISSING / QTY_DIFF / REJ_NOT_POSTED etc |
| severity | text | CRITICAL / HIGH / MEDIUM / LOW |
| confidence | text | HIGH / MEDIUM / LOW / CRITICAL |
| business_date | date |  |
| warehouse_id | uuid/text |  |
| owner_role | text | owner role |
| assigned_to | text | user_id nullable |
| status | text | OPEN / ASSIGNED / IN_REVIEW / WAIT_ERP_ADMIN / WAIT_CHECKER_COUNT / RESOLVED / APPROVED / CLOSED / REOPENED |
| due_at | datetime | optional SLA |
| closed_by | text | nullable |
| closed_at | datetime | nullable |
| close_reason | text | nullable |
| created_at | datetime |  |
| updated_at | datetime |  |

### Retention Rule

Open issue ห้ามถูกลบ

---

## 15.2 issue_comments

| Field | Type | Description |
|---|---|---|
| comment_id | uuid/text | primary key |
| issue_id | uuid/text | FK issues |
| comment_text | text |  |
| comment_type | text | NOTE / ACTION / SYSTEM |
| created_by | text | user/system |
| created_at | datetime |  |

---

## 15.3 checker_tasks

| Field | Type | Description |
|---|---|---|
| task_id | uuid/text | primary key |
| issue_id | uuid/text | nullable FK issues |
| task_type | text | COUNT / RECOUNT / VERIFY_INBOUND / VERIFY_REJ / VERIFY_LOCATION_DIFF / LOT_VERIFY |
| business_date | date |  |
| warehouse_id | uuid/text |  |
| location_id | uuid/text | optional |
| customer_id | uuid/text | optional |
| item_id | uuid/text | optional |
| year | text | optional |
| lot | text | optional |
| priority | text | CRITICAL / HIGH / MEDIUM / LOW |
| assigned_to | text | checker user_id nullable |
| owner_role | text | CHECKER / SUPERVISOR |
| status | text | OPEN / ASSIGNED / IN_PROGRESS / DONE / HOLD / CANCELLED |
| due_at | datetime | optional |
| created_by | text | system/user |
| created_at | datetime |  |
| completed_at | datetime | nullable |

---

## 15.4 investigations

Optional in MVP; issue detail may cover basic investigation

| Field | Type | Description |
|---|---|---|
| investigation_id | uuid/text | primary key |
| issue_id | uuid/text | FK issues |
| investigation_status | text | OPEN / IN_PROGRESS / RESOLVED / CLOSED |
| root_cause | text | nullable |
| corrective_action | text | nullable |
| assigned_to | text | nullable |
| created_at | datetime |  |
| closed_at | datetime | nullable |

---

# 16. Export Tables

## 16.1 export_jobs

| Field | Type | Description |
|---|---|---|
| export_id | uuid/text | primary key |
| export_type | text | ISSUE_LIST / DAILY_RECON / REJ_REPORT / ERP_TEMPLATE / COUNT_TASK / AUDIT / RETENTION |
| file_format | text | CSV / XLSX / PDF / PRINT_VIEW |
| filters_json | json/text | filters used |
| selected_ids_json | json/text | selected rows optional |
| row_count | number |  |
| file_url | text | generated file location |
| status | text | QUEUED / RUNNING / READY / FAILED / CANCELLED |
| exported_by | text | user_id |
| exported_at | datetime |  |
| error_message | text | nullable |

---

## 16.2 export_logs

| Field | Type | Description |
|---|---|---|
| export_log_id | uuid/text | primary key |
| export_id | uuid/text | FK export_jobs |
| action | text | CREATED / DOWNLOADED / SENT / FAILED |
| actor_id | text | user/system |
| message | text | optional |
| created_at | datetime |  |

---

# 17. Notification Tables

## 17.1 notification_channels

| Field | Type | Description |
|---|---|---|
| channel_id | uuid/text | primary key |
| channel_type | text | IN_APP / TELEGRAM / LINE_MESSAGING / EMAIL |
| channel_name | text | display name |
| token_ref | text | reference to secure token, not raw token |
| target_id | text | chat_id / group_id / email / user id |
| active_flag | boolean | active |
| created_by | text | admin |
| created_at | datetime |  |
| updated_at | datetime |  |

---

## 17.2 notification_rules

| Field | Type | Description |
|---|---|---|
| rule_id | uuid/text | primary key |
| event_code | text | USER_PENDING_APPROVAL / QTY_DIFF etc |
| rule_name | text | display name |
| severity_filter | text | CRITICAL,HIGH etc |
| target_role | text | ADMIN / ERP_ADMIN etc |
| channel_id | uuid/text | FK notification_channels |
| template_id | uuid/text | FK notification_templates |
| cooldown_minutes | number | anti-spam |
| send_once_per_issue | boolean | issue duplicate prevention |
| active_flag | boolean | active |
| created_by | text |  |
| created_at | datetime |  |

---

## 17.3 notification_templates

| Field | Type | Description |
|---|---|---|
| template_id | uuid/text | primary key |
| template_name | text | display name |
| event_code | text | event |
| title_template | text | title |
| body_template | text | body with variables |
| active_flag | boolean | active |
| created_at | datetime |  |

---

## 17.4 notification_logs

| Field | Type | Description |
|---|---|---|
| notification_log_id | uuid/text | primary key |
| event_code | text | event |
| issue_id | uuid/text | nullable |
| channel_type | text | IN_APP / TELEGRAM / LINE / EMAIL |
| target | text | target id/email |
| message_preview | text | preview |
| status | text | SENT / FAILED / SKIPPED / RETRYING |
| error_message | text | nullable |
| sent_at | datetime | nullable |
| retry_count | number |  |
| created_at | datetime |  |

### Retention

SENT/SKIPPED logs เกิน 45 วันลบได้ตาม policy ยกเว้น critical event

---

# 18. Audit / Security Tables

## 18.1 audit_logs

### Purpose

เก็บประวัติ action สำคัญทั้งหมด

| Field | Type | Description |
|---|---|---|
| audit_id | uuid/text | primary key |
| actor_id | text | user/system |
| actor_type | text | USER / SYSTEM |
| action | text | USER_APPROVED / IMPORT_CONFIRMED etc |
| entity_type | text | table/entity |
| entity_id | text | record id |
| old_value_json | json/text | nullable |
| new_value_json | json/text | nullable |
| reason | text | optional |
| ip_address | text | optional |
| user_agent | text | optional |
| severity | text | LOW / MEDIUM / HIGH / CRITICAL |
| created_at | datetime |  |

### Rules

- Critical audit ไม่ควรลบใน 45 วัน
- Retention delete ต้องมี audit/retention log

---

# 19. Retention / Maintenance Tables

## 19.1 retention_jobs

| Field | Type | Description |
|---|---|---|
| retention_job_id | uuid/text | primary key |
| cutoff_date | date | today - 45 days |
| triggered_by | text | SYSTEM / user_id |
| trigger_type | text | SCHEDULED / MANUAL |
| status | text | RUNNING / SUCCESS / WARNING / FAILED |
| started_at | datetime |  |
| finished_at | datetime |  |
| deleted_total_rows | number |  |
| skipped_open_issues | number |  |
| message | text |  |

---

## 19.2 retention_logs

| Field | Type | Description |
|---|---|---|
| retention_log_id | uuid/text | primary key |
| retention_job_id | uuid/text | FK retention_jobs |
| table_name | text | affected table |
| cutoff_date | date |  |
| deleted_rows | number |  |
| skipped_rows | number |  |
| skipped_reason | text | OPEN_ISSUE / PENDING_EXPORT etc |
| status | text | SUCCESS / WARNING / FAILED |
| message | text | optional |
| created_at | datetime |  |

---

## 19.3 system_health_checks

| Field | Type | Description |
|---|---|---|
| health_check_id | uuid/text | primary key |
| checked_at | datetime |  |
| db_usage_mb | number | optional |
| db_usage_percent | number | optional |
| open_critical_issues | number |  |
| pending_alias_count | number |  |
| pending_inbound_count | number |  |
| last_import_at | datetime |  |
| last_replay_at | datetime |  |
| last_recon_at | datetime |  |
| last_retention_at | datetime |  |
| status | text | OK / WARNING / CRITICAL |
| details_json | json/text | details |

---

# 20. Summary / Dashboard Tables

## 20.1 daily_recon_summary

| Field | Type | Description |
|---|---|---|
| summary_id | uuid/text | primary key |
| business_date | date |  |
| warehouse_id | uuid/text |  |
| matched_count | number |  |
| issue_count | number |  |
| critical_count | number |  |
| high_count | number |  |
| medium_count | number |  |
| low_count | number |  |
| system_missing_count | number |  |
| qty_diff_count | number |  |
| rej_not_posted_count | number |  |
| generated_at | datetime |  |

---

## 20.2 daily_issue_summary

| Field | Type | Description |
|---|---|---|
| summary_id | uuid/text | primary key |
| business_date | date |  |
| warehouse_id | uuid/text |  |
| open_issue_count | number |  |
| assigned_issue_count | number |  |
| closed_issue_count | number |  |
| aging_over_sla_count | number |  |
| pending_erp_count | number |  |
| pending_checker_count | number |  |
| pending_admin_count | number |  |
| generated_at | datetime |  |

---

## 20.3 location_occupancy_summary

Future/basic support

| Field | Type | Description |
|---|---|---|
| summary_id | uuid/text | primary key |
| business_date | date |  |
| warehouse_id | uuid/text |  |
| location_id | uuid/text |  |
| expected_qty | number |  |
| capacity_qty | number |  |
| occupancy_percent | number |  |
| status | text | OK / WARNING / EXCEEDED |
| generated_at | datetime |  |

---

## 20.4 import_summary

| Field | Type | Description |
|---|---|---|
| summary_id | uuid/text | primary key |
| business_date | date |  |
| warehouse_id | uuid/text | optional |
| inquiry_import_count | number |  |
| inout_import_count | number |  |
| failed_import_count | number |  |
| unknown_alias_count | number |  |
| generated_at | datetime |  |

---

# 21. Close / Lock Tables

## 21.1 daily_close_status

| Field | Type | Description |
|---|---|---|
| close_status_id | uuid/text | primary key |
| business_date | date |  |
| warehouse_id | uuid/text |  |
| status | text | OPEN / PRELIM_CLOSED / RECONSTRUCTED / FINAL_CLOSED / LOCKED |
| readiness_status | text | READY / READY_WITH_WARNING / NOT_READY |
| coverage_id | uuid/text | FK data_coverage_checks nullable |
| open_critical_count | number |  |
| pending_alias_count | number |  |
| pending_inbound_count | number |  |
| rej_not_posted_count | number |  |
| closed_by | text | nullable |
| closed_at | datetime | nullable |
| locked_by | text | nullable |
| locked_at | datetime | nullable |
| reason | text | optional |
| updated_at | datetime |  |

---

# 22. Future Capacity / Simulation Tables

These tables are future phase, not full MVP.

## 22.1 location_capacity_rules

| Field | Type | Description |
|---|---|---|
| capacity_rule_id | uuid/text | primary key |
| location_id | uuid/text | FK locations |
| product_group_id | uuid/text | optional |
| max_qty | number | max capacity |
| uom | text |  |
| allow_multi_lot | boolean | policy |
| allow_external_customer | boolean | policy |
| active_flag | boolean |  |

---

## 22.2 simulation_runs

| Field | Type | Description |
|---|---|---|
| simulation_id | uuid/text | primary key |
| simulation_type | text | AVAILABLE_LOCATION / RELOCATION |
| input_json | json/text | scenario input |
| result_json | json/text | output |
| status | text | RUNNING / SUCCESS / FAILED |
| created_by | text |  |
| created_at | datetime |  |

---

# 23. Key Relationships

## 23.1 Import to Canonical

```text
import_batches 1 → many import_raw_rows
import_raw_rows → normalized inquiry/system_inout records
```

## 23.2 Inquiry to Stock Replay

```text
inquiry_snapshots 1 → many inquiry_snapshot_lines
inquiry_snapshots → used as anchor in daily_stock_positions
```

## 23.3 System In-Out to Ledger

```text
system_inout_records → movement_ledger(source_type=SYSTEM_INOUT)
```

## 23.4 Field Movement to Ledger

```text
field_movements → movement_ledger(source_type=FIELD_NOTICE)
```

## 23.5 REJ to Ledger

```text
rej_notices → movement_ledger(source_type=REJ_NOTICE)
```

## 23.6 Replay to Reconciliation

```text
movement_ledger + inquiry_snapshots → daily_stock_positions

daily_stock_positions + physical_counts + system_inout_records + field_movements → recon_results
```

## 23.7 Result to Issue

```text
recon_results → issues → checker_tasks / notifications / exports
```

---

# 24. Indexing Recommendations

For database implementation, index these fields:

## 24.1 Common Indexes

```text
business_date
warehouse_id
location_id
customer_id
item_id
status
created_at
```

## 24.2 Import

```text
import_batches(file_type, uploaded_at)
import_batches(file_checksum)
import_raw_rows(batch_id, row_no)
```

## 24.3 Ledger

```text
movement_ledger(business_date, warehouse_id)
movement_ledger(location_id, item_id, customer_id)
movement_ledger(source_type, source_ref_id)
```

## 24.4 Stock Position

```text
daily_stock_positions(business_date, warehouse_id)
daily_stock_positions(location_id, item_id, customer_id, year, lot)
```

## 24.5 Issues

```text
issues(status, severity)
issues(business_date, warehouse_id)
issues(owner_role, assigned_to)
```

## 24.6 Alias

```text
customer_aliases(alias_text)
item_aliases(alias_text)
location_aliases(alias_text)
alias_review_queue(status, alias_type)
```

---

# 25. Retention Rules by Table

| Table | Retention Rule |
|---|---|
| import_raw_rows | delete after 45 days if normalized/closed and no open issue reference |
| import_error_logs | delete/archive after 45 days if resolved |
| system_inout_records | delete/archive detail after 45 days if summarized and no open issue |
| field_movements | delete/archive after 45 days if closed/matched and no open issue |
| physical_counts | archive after 45 days if linked issue closed |
| movement_ledger | archive after 45 days if summarized and no open issue |
| daily_stock_positions | keep summary longer, detail may archive |
| recon_results | archive closed after 45 days, keep issue-linked records |
| issues | keep if open; closed may archive after policy |
| export_logs | keep longer than 45 days recommended |
| audit_logs | keep critical logs longer than 45 days |
| notification_logs | delete sent/skipped after 45 days unless critical |
| retention_logs | keep longer for system proof |

---

# 26. Data Model Acceptance Criteria

Data Model ถือว่า complete ถ้า:

- [ ] แยก raw / normalized / canonical / result / summary ชัดเจน
- [ ] Inquiry ถูกเก็บเป็น Anchor Snapshot ไม่ใช่ movement
- [ ] System In-Out ถูกเก็บเป็น ERP movement และเข้า ledger
- [ ] Field Movement ถูกแยกจาก System In-Out
- [ ] Physical Count ถูกเก็บเป็น physical truth
- [ ] Movement Ledger รวมทุก stock effect source
- [ ] Stock Replay สร้าง Daily Stock Position ได้
- [ ] Reconciliation Result มี result_code/severity/confidence/explanation
- [ ] Issue เชื่อมกับ result ได้
- [ ] Alias แยก type ไม่ปน customer/item/location/DC
- [ ] User/Permission/Warehouse Scope รองรับ granular access
- [ ] Export/Notification/Audit/Retention มี log
- [ ] Retention 45 วันไม่ลบ open issue หรือ critical audit
- [ ] Dashboard มี summary tables ไม่อ่าน raw full history

---

# 27. Final Data Model Statement

```text
Data Model ของ Warehouse Reconciliation ERP Platform ต้องถูกออกแบบให้แยกความจริงแต่ละชั้นอย่างชัดเจน: Raw file, Master, Alias, ERP Snapshot, ERP Movement, Field Movement, Physical Count, Movement Ledger, Stock Replay, Reconciliation Result, Issue และ Governance Log เพื่อให้ระบบตรวจสอบ stock ได้แม้ข้อมูลไม่ครบ ไม่ตรง หรือ upload ข้ามวัน และยังสามารถ audit, export, notify และลบ/archive ข้อมูลเกิน 45 วันได้อย่างปลอดภัย
```

