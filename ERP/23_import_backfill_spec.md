# 23_import_backfill_spec.md

# Import, Backfill & Data Coverage Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Import Pipeline / Backfill Workflow / Data Coverage / Replay Trigger Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / Data Engineer / System Analyst Use  
**Primary Use:** ใช้กำหนดระบบนำเข้าไฟล์ Inquiry, System In-Out, Field Count, Master Data, การ upload แบบ chunk, raw staging, column mapping, normalize, alias queue, confirm import, backfill ข้ามวัน, coverage check, stock replay และ reconciliation trigger เพื่อให้ระบบรองรับ admin ไม่ upload ทุกวันและยังตรวจ stock ได้ถูกต้อง

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Import และ Backfill Specification ของ Warehouse Reconciliation ERP Platform

ระบบนี้ต้องรองรับสถานการณ์จริงที่:

1. Admin อาจไม่ได้ upload Inquiry ทุกวัน
2. Admin อาจลา/หยุด แล้วต้อง upload In-Out ย้อนหลัง 1–5 วันหรือมากกว่า
3. Inquiry เป็น snapshot ไม่ใช่ transaction
4. System In-Out เป็น movement ที่ ERP post แล้ว
5. ข้อมูลจากไฟล์จริงอาจมี column ไม่เหมือนกันทุกครั้ง
6. ชื่อ customer/item/location/DC/UOM อาจไม่ตรง master
7. แจ้งเข้าอาจไม่รู้ข้อมูลครบจนกว่าจะได้ In-Out วันถัดไป
8. ระบบต้องรู้ว่าข้อมูลช่วงไหนครบพอสำหรับ replay/reconciliation

หลักสำคัญ:

```text
Import ไม่ใช่แค่ upload file
Import คือ data pipeline ที่ต้องควบคุม source, schema, mapping, alias, validation, confirmation, replay และ reconciliation อย่างมี audit
```

---

## 2. Import Philosophy

## 2.1 Raw First, Canonical Later

ทุกไฟล์ต้องถูกเก็บเป็น raw ก่อน แล้วค่อย normalize

Correct flow:

```text
File Upload
→ Import Batch
→ Raw Rows
→ Column Mapping
→ Normalize
→ Alias Matching
→ Confirm
→ Snapshot / Movement / Count / Master
```

ห้าม:

```text
Upload file → Insert เข้า stock/reconciliation ทันที
```

เหตุผล:

- ต้อง trace row กลับ source ได้
- ต้อง preview ก่อน confirm
- ต้องแก้ column mapping ได้
- ต้องส่ง unknown alias เข้า review
- ต้องป้องกัน file ซ้ำ
- ต้อง audit ได้

---

## 2.2 No Hardcoded Columns

ห้าม hardcode เช่น:

```text
column A = date
column B = customer
column C = qty
```

ต้องใช้ column mapping config:

```text
source_column_name → canonical_field
```

เพราะไฟล์จริงอาจเปลี่ยน header, เพิ่ม column, สลับ column หรือ export จาก ERP คนละ template

---

## 2.3 Import Confirmation Is a Control Point

ข้อมูลจากไฟล์จะถูกนำไปใช้คำนวณจริงเมื่อ confirm เท่านั้น

ก่อน confirm ต้องผ่าน:

- file validation
- required column mapping
- row validation
- alias validation/review
- duplicate warning
- preview summary
- permission check
- confirmation modal
- audit log

---

## 2.4 Backfill Is Normal Workflow, Not Exception

การ upload ย้อนหลังไม่ใช่เคสพิเศษ แต่เป็น workflow ปกติของระบบ

ระบบต้องรองรับ:

```text
Admin ไม่ upload 4–5 วัน
→ ดึง In-Out ย้อนหลัง
→ Upload date range
→ Coverage Check
→ Stock Replay
→ Reconciliation
→ Issue/Task
```

---

# 3. Import Types

ระบบรองรับ import types หลัก:

| Import Type | Description | Result After Confirm | MVP |
|---|---|---|---:|
| INQUIRY | ERP stock snapshot | inquiry_snapshot + lines | Yes |
| SYSTEM_INOUT | ERP posted movement | system_inout_records + movement_ledger | Yes |
| FIELD_COUNT | physical count file/manual count import | physical_counts | Optional Basic |
| MASTER_DATA | customer/item/location/uom master seed | master tables/alias seed | Yes Basic |
| ALIAS_SEED | alias seed file | alias tables | Optional |
| REJ_IMPORT | REJ reference if ERP exports separately | system movement or REJ match | Future/Optional |

---

## 3.1 INQUIRY Import

### Meaning

Inquiry คือ snapshot ของยอดคงเหลือ ณ วันที่/เวลาหนึ่ง

### Used For

- Anchor Snapshot
- Stock Replay starting point
- Daily close reference

### Must Not Do

Inquiry ห้ามสร้าง movement ledger โดยตรง

### After Confirm

```text
create inquiry_snapshots
create inquiry_snapshot_lines
set active snapshot policy
update coverage status
optional trigger replay/reconciliation
```

---

## 3.2 SYSTEM_INOUT Import

### Meaning

System In-Out คือ movement ที่ ERP post แล้ว

### Used For

- Movement Ledger
- Stock Replay
- Field vs System reconciliation
- Pending Inbound matching
- REJ posting check

### After Confirm

```text
create system_inout_records
create movement_ledger source_type=SYSTEM_INOUT
update coverage by business_date
trigger stock replay if configured
trigger reconciliation if configured
```

---

## 3.3 FIELD_COUNT Import

### Meaning

ไฟล์นับจริง หรือ bulk physical count import

### Used For

- Expected vs Count reconciliation
- Recount task result
- Close readiness

### After Confirm

```text
create physical_counts
link to task/location if possible
trigger reconciliation if configured
```

MVP สามารถทำ manual count form ก่อน แล้ว field count import เป็น optional

---

## 3.4 MASTER_DATA Import

### Meaning

ไฟล์ตั้งต้นสำหรับ master เช่น customer/item/location/UOM

### Used For

- master seed
- alias seed
- normalize pipeline

### After Confirm

```text
create/update master data according policy
create alias seed optional
write audit
```

---

# 4. Import Pipeline Overview

## 4.1 Standard Import Flow

```text
1. Select Import Type
2. Select Warehouse / Date Range
3. Upload File
4. Parse File on Client or Server
5. Create Import Batch
6. Preview Header / Sample Rows
7. Save Column Mapping
8. Upload Raw Rows by Chunk
9. Validate Raw Rows
10. Normalize Batch
11. Detect Alias / Queue Unknown
12. Show Validation Summary
13. Resolve Blocking Errors
14. Confirm Import
15. Create Domain Records
16. Trigger Replay/Reconciliation if needed
17. Update Dashboard Summary
18. Write Audit / Notification
```

---

## 4.2 Import Status Lifecycle

```text
DRAFT
UPLOADED
PREVIEWED
MAPPED
RAW_UPLOADED
VALIDATING
NORMALIZING
NORMALIZED
NORMALIZED_WITH_WARNING
BLOCKED_BY_ERROR
BLOCKED_BY_ALIAS
CONFIRMING
CONFIRMED
CANCELLED
FAILED
SUPERSEDED
```

### Status Meaning

| Status | Meaning |
|---|---|
| DRAFT | created but file not processed |
| UPLOADED | file metadata uploaded |
| PREVIEWED | header/sample shown |
| MAPPED | column mapping saved |
| RAW_UPLOADED | raw rows saved |
| VALIDATING | validation running |
| NORMALIZING | normalize job running |
| NORMALIZED | canonical staging ready |
| NORMALIZED_WITH_WARNING | usable but has warnings |
| BLOCKED_BY_ERROR | cannot confirm until fixed |
| BLOCKED_BY_ALIAS | unknown alias blocks confirm |
| CONFIRMED | data applied to domain tables |
| CANCELLED | user cancelled before confirm |
| FAILED | processing failed |
| SUPERSEDED | replaced by newer batch/policy |

---

## 4.3 Import Batch Required Fields

Table: `import_batches`

| Field | Required | Description |
|---|---:|---|
| batch_id | Yes | unique id |
| file_type | Yes | INQUIRY / SYSTEM_INOUT / FIELD_COUNT / MASTER_DATA |
| file_name | Yes | original file name |
| file_size | Yes | file size |
| file_checksum | Yes | duplicate detection |
| warehouse_id | Conditional | required for warehouse-specific imports |
| business_date_from | Conditional | required for date range import |
| business_date_to | Conditional | required for date range import |
| uploaded_by | Yes | actor id |
| uploaded_at | Yes | timestamp |
| row_count | Yes | total rows |
| header_json | Yes | file headers |
| status | Yes | lifecycle status |
| warning_count | Yes | count |
| error_count | Yes | count |
| unknown_alias_count | Yes | count |
| confirmed_by | No | actor id |
| confirmed_at | No | timestamp |
| confirm_policy_json | No | policy used |

---

# 5. File Upload & Preview

## 5.1 Supported File Types

MVP allowed:

```text
.xlsx
.csv
```

Optional later:

```text
.xls
json
```

Reject:

- executable files
- unknown extensions
- password-protected file if parser cannot read
- empty file

---

## 5.2 Preview Requirements

Before mapping/confirm user must see:

- file name
- file type
- file size
- checksum duplicate warning
- detected sheet name if xlsx
- row count
- header list
- sample rows, e.g. first 20 rows
- detected date range if possible
- suggested file type confidence
- suggested column mapping if available

---

## 5.3 Preview UI Pattern

```text
Step 1: Upload
Step 2: Preview
Step 3: Column Mapping
Step 4: Validate / Normalize
Step 5: Confirm
```

Preview should not render all rows at once

For large files:

```text
show sample rows only
show row count
show progress for chunk upload
```

---

# 6. Column Mapping Specification

## 6.1 Canonical Field Concept

Column mapping converts source headers to system fields:

```text
source column: "Posting Date"
canonical field: posting_date
```

Mapping must be saved by:

- file_type
- warehouse optional
- template name
- header signature/checksum

---

## 6.2 Common Canonical Fields

| Canonical Field | Description |
|---|---|
| business_date | date used for operation/control |
| snapshot_date | inquiry snapshot date |
| posting_date | ERP posting date |
| movement_date | physical/ERP movement date |
| movement_time | movement time if available |
| warehouse_raw | raw warehouse text/code |
| dc_raw | raw DC text/code |
| location_raw | raw location text/code |
| from_location_raw | source location |
| to_location_raw | destination location |
| customer_raw | raw customer text/code |
| item_raw | raw item text/code |
| description_raw | raw combined description |
| year_raw | year/season raw |
| lot_raw | lot raw |
| qty_raw | quantity raw |
| uom_raw | unit raw |
| movement_type_raw | movement type code/text |
| source_ref | document/order/reference |
| remark_raw | note |

---

## 6.3 INQUIRY Required Mapping

Required at minimum:

```text
snapshot_date or business_date
warehouse_raw or dc_raw
qty_raw
item_raw or description_raw
```

Recommended:

```text
location_raw
customer_raw
year_raw
lot_raw
uom_raw
source_ref
```

If location/customer not available, system must downgrade confidence or require product-specific policy

---

## 6.4 SYSTEM_INOUT Required Mapping

Required at minimum:

```text
posting_date or movement_date
movement_type_raw
warehouse_raw or dc_raw
qty_raw
item_raw or description_raw
```

Required by movement type:

| Movement Type | Required Location |
|---|---|
| IN | to_location_raw or location_raw |
| OUT | from_location_raw or location_raw |
| TRANSFER | from_location_raw and to_location_raw |
| REJ | from_location_raw and rej/to_location_raw |
| ADJUSTMENT | location_raw |

Recommended:

```text
customer_raw
year_raw
lot_raw
uom_raw
source_ref
```

---

## 6.5 FIELD_COUNT Required Mapping

```text
business_date
warehouse_raw or dc_raw
location_raw
qty_raw
```

Recommended:

```text
customer_raw
item_raw or description_raw
year_raw
lot_raw
uom_raw
counted_by_raw
count_time
```

---

## 6.6 MASTER_DATA Required Mapping

Depends on master type:

### Customer Master

```text
customer_code
customer_name
customer_type optional
```

### Item Master

```text
item_code
item_name
product_group
uom optional
pack_size optional
```

### Location Master

```text
warehouse_code
location_code
location_name
capacity optional
```

### Alias Seed

```text
alias_type
alias_text
canonical_code or canonical_name
confidence optional
```

---

# 7. Raw Staging

## 7.1 Raw Row Storage

Table: `import_raw_rows`

Required fields:

| Field | Description |
|---|---|
| raw_row_id | id |
| batch_id | import batch |
| row_no | original row number |
| raw_json | full raw row |
| parse_status | PENDING / PARSED / ERROR |
| error_code | optional |
| error_message | optional |
| created_at | timestamp |

---

## 7.2 Why Raw Rows Matter

Raw rows are needed to:

- debug import errors
- show row-level detail
- re-normalize after alias mapping
- prove source of system records
- audit import result
- support data profiling

Raw rows can be deleted/archive after retention policy if safe

---

## 7.3 Chunk Upload

Large file upload should be chunked:

```text
chunk_index
total_chunks
rows_in_chunk
batch_id
checksum optional per chunk
```

Rules:

- prevent duplicate chunk insert
- allow retry failed chunk
- update progress
- do not confirm until all chunks uploaded

---

# 8. Normalize Pipeline

## 8.1 Normalize Steps

```text
Load batch + mapping
→ Load raw rows
→ Parse canonical fields
→ Normalize date
→ Normalize qty
→ Normalize UOM
→ Normalize movement type
→ Resolve warehouse/DC
→ Resolve location
→ Resolve customer
→ Resolve item/year/lot
→ Create normalized staging records
→ Create alias review queue for unknowns
→ Create import error logs
→ Update batch summary
```

---

## 8.2 Text Normalization

Before alias matching:

- trim spaces
- normalize case
- remove duplicated spaces
- normalize Thai/English punctuation
- standardize KG/kg/Kg
- strip unnecessary brackets if configured
- keep original raw text separately

Important:

```text
Do not destroy raw text
Store normalized_text separately
```

---

## 8.3 Quantity Normalization

Handle:

- `1,000`
- `1000.00`
- `1,000 BAG`
- negative qty if ERP uses sign
- blank qty
- text qty

Internal canonical:

```text
qty = numeric positive value where possible
movement direction from movement_type/sign config
```

---

## 8.4 Date Normalization

Handle:

- YYYY-MM-DD
- DD/MM/YYYY
- MM/DD/YYYY if detected
- Excel serial date
- Buddhist year
- missing time

Ambiguous date must create warning or block depending policy

---

## 8.5 Movement Type Normalization

Map ERP movement code/text to canonical:

```text
IN
OUT
TRANSFER
REJ
ADJUSTMENT
RETURN
UNKNOWN
```

Unknown movement type must go to mapping review or error

---

# 9. Alias Handling During Import

## 9.1 Alias Detection

System must detect unknown:

- customer
- item
- location
- DC/warehouse
- UOM
- lot pattern optional

When unknown:

```text
create alias_review_queue
link source_batch_id
link source_row_no
set confidence
set status=PENDING
```

---

## 9.2 Blocking vs Non-Blocking Alias

Not all unknown aliases block import equally

| Alias Type | Blocking? | Reason |
|---|---:|---|
| warehouse/DC | Usually Yes | cannot scope stock |
| location | Often Yes | cannot place stock |
| item | Usually Yes | cannot calculate stock identity |
| customer | Depends | may be unknown pending inbound/customer external |
| UOM | Depends | if default item UOM exists may warn |
| lot/year | Usually Warning | depending product group |

MVP recommended:

```text
Block confirm if warehouse/location/item unknown for normal import
Allow pending inbound path for incomplete inbound data
```

---

## 9.3 Alias Reprocess

After Admin maps alias:

```text
mark affected raw rows / normalized staging for reprocess
rerun normalize for affected batch or rows
update unknown_alias_count
allow confirm if no blockers
```

---

# 10. Import Validation Summary

Before confirm, system must show:

| Metric | Description |
|---|---|
| total_rows | all rows |
| valid_rows | rows ready |
| warning_rows | rows usable with warning |
| error_rows | rows blocking |
| unknown_alias_count | alias pending |
| duplicate_candidate_count | possible duplicates |
| date_range_detected | from data |
| movement_types_detected | list |
| qty_total_by_uom | summary by UOM |
| coverage_impact | date coverage effect |

---

## 10.1 Error Row Detail

Error rows must show:

- row_no
- field
- raw value
- error code
- message
- suggested fix

Examples:

```text
Row 24: Qty = "abc" → INVALID_QTY → แก้ค่า qty ในไฟล์หรือ exclude row
Row 31: Location = "W8-X" → UNKNOWN_LOCATION_ALIAS → map location alias
```

---

## 10.2 Warning Detail

Warnings:

- duplicate checksum
- date ambiguous
- UOM assumed from item
- low confidence alias suggestion
- missing optional lot/year
- movement date outside selected range

---

# 11. Confirm Import

## 11.1 Confirm Rules

Import can be confirmed only if:

- user has permission
- batch status = NORMALIZED or NORMALIZED_WITH_WARNING
- blocking errors = 0
- blocking alias = 0 or accepted policy
- warehouse scope valid
- confirmation modal accepted
- reason provided if warning override

---

## 11.2 Confirm INQUIRY

Flow:

```text
validate batch
→ validate snapshot conflict
→ create inquiry_snapshot
→ create inquiry_snapshot_lines
→ set active snapshot by policy
→ update coverage
→ audit IMPORT_CONFIRM
→ optional trigger replay/recon
```

Snapshot conflict policy:

```text
ARCHIVE_OLD_USE_NEW
REPLACE_OLD_WITH_NEW
KEEP_BOTH_MANUAL_ACTIVE
CANCEL_NEW_UPLOAD
```

---

## 11.3 Confirm SYSTEM_INOUT

Flow:

```text
validate batch
→ create system_inout_records
→ create movement_ledger entries
→ update inout coverage by date
→ audit IMPORT_CONFIRM
→ optional trigger stock replay
→ optional trigger reconciliation
```

---

## 11.4 Confirm FIELD_COUNT

Flow:

```text
validate batch
→ create physical_counts
→ link tasks/issues if possible
→ audit IMPORT_CONFIRM
→ trigger reconciliation if configured
```

---

## 11.5 Confirm MASTER_DATA

Flow:

```text
validate master data
→ create/update master records
→ create aliases if included
→ audit MASTER_IMPORT_CONFIRM
```

---

# 12. Backfill Workflow

## 12.1 Backfill Use Case

Backfill ใช้เมื่อ:

- Admin ไม่ได้ upload Inquiry ทุกวัน
- Admin ดึง In-Out ย้อนหลังได้
- ต้อง reconstruct stock จาก Anchor + Movement Ledger
- ต้องรู้ว่าช่วงวันที่ขาด movement อะไร
- ต้องสร้าง count tasks สำหรับ location ที่มีความเสี่ยง

---

## 12.2 Backfill Wizard Flow

```text
Step 1: Select Date Range
Step 2: Select Warehouse
Step 3: Check Existing Coverage
Step 4: Upload or Select System In-Out Batch
Step 5: Validate / Normalize / Confirm In-Out
Step 6: Run Coverage Check Again
Step 7: Run Stock Replay
Step 8: Run Reconciliation
Step 9: Generate Issues / Count Required Locations
Step 10: Show Backfill Summary
```

---

## 12.3 Backfill Date Range Rules

Required:

```text
date_from <= date_to
warehouse_id required
```

Recommended warning:

```text
date range > 7 days → show performance warning
```

MVP target:

```text
support 1–5 days comfortably
support more days as async job
```

---

## 12.4 Anchor Selection Rule

For replay date range:

```text
Find latest active Inquiry snapshot where snapshot_date <= date_from
```

If no anchor:

```text
status = MISSING_ANCHOR
block final close
allow estimated only with approval/policy
```

Example:

```text
Backfill 2026-05-10 to 2026-05-15
Use Inquiry snapshot <= 2026-05-10
Then apply ledger movements from 2026-05-11 to 2026-05-15
```

Actual inclusion of date_from ledger depends on whether anchor is beginning-of-day or end-of-day; must be defined in Data Profiling

---

## 12.5 Backfill Coverage Inputs

Coverage must check:

- anchor snapshot exists
- In-Out exists for each day in range
- In-Out confirmed status
- unknown alias blockers
- pending inbound in date range
- REJ waiting system post
- physical count availability for moving/high-risk locations
- failed previous replay/reconciliation jobs

---

# 13. Data Coverage Status

## 13.1 Coverage Status Codes

```text
COVERAGE_OK
MISSING_ANCHOR
MISSING_SYSTEM_INOUT
PARTIAL_SYSTEM_INOUT
PENDING_UNKNOWN_ALIAS
PENDING_UNKNOWN_INBOUND
NO_COUNT_FOR_MOVING_LOCATION
REJ_PENDING
REPLAY_REQUIRED
RECONCILIATION_REQUIRED
COVERAGE_WARNING
COVERAGE_CRITICAL
```

---

## 13.2 Coverage Severity Logic

### Critical

```text
MISSING_ANCHOR and no approved estimate
Unknown item/location alias blocking normalize
Open critical issue in range
REJ_NOT_POSTED critical
```

### Warning

```text
Partial In-Out missing
Pending inbound not final
Unknown optional alias
No count for moving location
Dashboard stale
```

### OK

```text
Anchor exists
In-Out complete or no movement expected
No blocking alias
No blocking pending inbound
No critical issue
Replay/reconciliation up to date
```

---

## 13.3 Coverage Calendar UI

Backfill Wizard should show per-day coverage:

| Date | Anchor | In-Out | Alias | Pending Inbound | REJ | Count | Replay | Recon | Status |
|---|---|---|---|---|---|---|---|---|---|
| 2026-05-10 | OK | OK | 0 | 0 | 0 | OK | Done | Done | OK |
| 2026-05-11 | Anchor Prev | Missing | 2 | 1 | 0 | Missing | Needed | Needed | Warning |
| 2026-05-12 | Anchor Prev | OK | 0 | 0 | 1 | OK | Done | Issue | Warning |

---

# 14. Stock Replay Trigger

## 14.1 Trigger Conditions

Stock Replay should be triggered when:

- Inquiry snapshot confirmed
- System In-Out confirmed
- movement ledger changed
- pending inbound confirmed
- REJ posted/matched
- correction applied
- Admin manually runs backfill

---

## 14.2 Replay Scope

Replay should run for affected:

- date range
- warehouse
- customer/item/location if optimization available

MVP can run per date range + warehouse

---

## 14.3 Replay Output

```text
daily_stock_positions
stock_replay_jobs
coverage status update
summary refresh
```

---

# 15. Reconciliation Trigger

## 15.1 Trigger Conditions

Reconciliation should be triggered after:

- stock replay success
- field movement created/updated
- physical count submitted
- REJ notice created/posted
- pending inbound confirmed
- alias reprocess completed
- Admin manually runs reconciliation

---

## 15.2 Reconciliation Scope

Default:

```text
date range + warehouse
```

Optimized later:

```text
affected locations/items/issues only
```

---

# 16. Backfill Job Model

## 16.1 backfill_jobs Table

| Field | Required | Description |
|---|---:|---|
| backfill_job_id | Yes | id |
| warehouse_id | Yes | warehouse |
| date_from | Yes | start |
| date_to | Yes | end |
| triggered_by | Yes | user/system |
| trigger_reason | No | reason |
| status | Yes | QUEUED/RUNNING/SUCCESS/WARNING/FAILED |
| coverage_status | Yes | OK/WARNING/CRITICAL |
| replay_job_id | No | linked replay |
| reconciliation_job_id | No | linked recon |
| created_issue_count | Yes | count |
| created_task_count | Yes | count |
| warning_count | Yes | count |
| error_count | Yes | count |
| started_at | Yes | timestamp |
| finished_at | No | timestamp |
| message | No | summary |

---

## 16.2 Backfill Job Steps

```text
QUEUED
→ CHECKING_COVERAGE
→ WAITING_IMPORT if required
→ NORMALIZING_INOUT
→ CONFIRMING_INOUT
→ RUNNING_REPLAY
→ RUNNING_RECONCILIATION
→ CREATING_TASKS
→ REFRESHING_SUMMARY
→ SUCCESS / WARNING / FAILED
```

---

## 16.3 Backfill Summary Output

After backfill complete show:

- date range
- warehouse
- anchor used
- In-Out days found/missing
- replay status
- reconciliation status
- matched count
- issue count
- count tasks created
- critical blockers
- warning list
- next actions

---

# 17. Duplicate Handling

## 17.1 Duplicate File

Use checksum:

```text
same file_checksum + file_type + warehouse + date range
```

Policy options:

```text
WARN_ONLY
BLOCK_DUPLICATE
ALLOW_WITH_REASON
```

MVP recommended:

```text
WARN_ONLY + reason required if confirming duplicate
```

---

## 17.2 Duplicate Snapshot

Same business_date + warehouse active snapshot exists

Must choose policy:

```text
ARCHIVE_OLD_USE_NEW
REPLACE_OLD_WITH_NEW
KEEP_BOTH_MANUAL_ACTIVE
CANCEL_NEW_UPLOAD
```

---

## 17.3 Duplicate In-Out Rows

Potential duplicate key candidates:

```text
source_ref
posting_date
movement_type
warehouse/location
item/customer/year/lot
qty
```

Actual key must be locked after Data Profiling

Duplicate should be warning or block depending source_ref quality

---

# 18. Permission & Audit

## 18.1 Required Permissions

| Action | Permission |
|---|---|
| Upload Inquiry | CAN_IMPORT_INQUIRY |
| Upload In-Out | CAN_IMPORT_INOUT |
| Upload Master | CAN_IMPORT_MASTER |
| Save Mapping | CAN_MANAGE_COLUMN_MAPPING or import permission |
| Normalize Batch | import permission |
| Confirm Import | CAN_CONFIRM_IMPORT |
| Run Backfill | CAN_IMPORT_INOUT + CAN_RUN_STOCK_REPLAY |
| Run Replay | CAN_RUN_STOCK_REPLAY |
| Run Reconciliation | CAN_RUN_RECONCILIATION |
| View Import Logs | CAN_VIEW_IMPORT_LOG |

---

## 18.2 Audit Events

Audit required:

```text
IMPORT_BATCH_CREATED
IMPORT_RAW_UPLOADED
COLUMN_MAPPING_SAVED
IMPORT_NORMALIZE_RUN
IMPORT_CONFIRM
IMPORT_CANCEL
DUPLICATE_IMPORT_OVERRIDE
BACKFILL_JOB_CREATED
BACKFILL_RUN
RUN_STOCK_REPLAY
RUN_RECONCILIATION
```

Audit must include:

- actor
- batch/job id
- file type
- warehouse/date range
- row counts
- warning/error count
- reason if override

---

# 19. Error & State Handling

## 19.1 Import Error States

- invalid file type
- empty file
- header not found
- required mapping missing
- chunk upload failed
- invalid date
- invalid qty
- unknown alias blocking
- duplicate file warning
- normalize failed
- confirm failed

---

## 19.2 Backfill Error States

- missing anchor
- missing in-out
- coverage critical
- replay failed
- reconciliation failed
- issue generation failed
- task creation failed

---

## 19.3 User-Friendly Messages

### Missing Anchor

```text
ไม่พบ Inquiry Snapshot สำหรับใช้เป็นจุดเริ่มต้นของการคำนวณย้อนหลัง
โปรด upload Inquiry ก่อนช่วงวันที่นี้ หรือยืนยันการคำนวณแบบ estimated ตาม policy
```

### Missing In-Out

```text
ข้อมูล In-Out ยังไม่ครบในช่วงวันที่เลือก
ระบบสามารถแสดง warning และลด confidence ได้ แต่ผล stock replay อาจไม่สมบูรณ์
```

### Unknown Alias Blocking

```text
ยังมีชื่อที่ระบบไม่รู้จักและจำเป็นต่อการคำนวณ
โปรดไปที่ Alias Review เพื่อ map master ก่อน confirm import
```

---

# 20. Performance Requirements

## 20.1 Import Performance

- preview sample only, not full render
- chunk upload for large file
- async normalize for large batch
- store raw rows in batch
- batch status/progress visible

---

## 20.2 Backfill Performance

- run stock replay as job
- run reconciliation as job
- avoid O(n²) matching
- use indexes on date/warehouse/status/source_ref
- refresh summary after job completion

---

## 20.3 Data Retention

Import raw rows older than 45 days can be deleted/archive only if safe per retention policy

---

# 21. Test Cases

## 21.1 Import Test Cases

- Upload valid Inquiry
- Upload valid System In-Out
- Upload empty file → block
- Upload invalid file type → block
- Missing required mapping → block normalize
- Invalid qty row → error row
- Invalid date row → error row
- Unknown item alias → alias queue
- Duplicate file checksum → warning
- Confirm Inquiry with active snapshot conflict → policy required
- Confirm In-Out → system records + ledger

---

## 21.2 Backfill Test Cases

- Backfill 1 day with anchor + inout complete → success
- Backfill 5 days with anchor + inout complete → success
- Backfill missing anchor → critical/block final close
- Backfill missing In-Out day → warning/low confidence
- Backfill with unknown alias → blocked until alias mapped
- Backfill creates daily stock positions
- Backfill triggers reconciliation
- Backfill creates count required tasks

---

# 22. Acceptance Criteria

Import & Backfill ถือว่า complete ถ้า:

- [ ] Import รองรับ INQUIRY และ SYSTEM_INOUT อย่างน้อย
- [ ] Upload file แล้ว preview headers/sample rows ได้
- [ ] File checksum duplicate warning ได้
- [ ] Import batch และ raw rows ถูกสร้างพร้อม batch_id
- [ ] Chunk upload รองรับไฟล์ใหญ่ได้
- [ ] Column mapping ไม่ hardcode
- [ ] Required mapping missing แล้ว block normalize ได้
- [ ] Normalize parse date/qty/uom/movement type ได้
- [ ] Unknown alias เข้า Alias Review Queue ได้
- [ ] Blocking alias กัน confirm ได้
- [ ] Confirm Inquiry สร้าง Anchor Snapshot ไม่สร้าง Ledger
- [ ] Confirm In-Out สร้าง System In-Out + Movement Ledger
- [ ] Admin upload/backfill ย้อนหลัง 1–5 วันได้
- [ ] Coverage Check แสดง MISSING_ANCHOR / MISSING_INOUT / PENDING_ALIAS ได้
- [ ] Backfill run Stock Replay ได้
- [ ] Backfill run Reconciliation ได้
- [ ] Backfill สร้าง Issue/Count Required Locations ได้
- [ ] ทุก import/backfill action มี audit
- [ ] Import/Backfill มี loading/error/empty/progress states
- [ ] Dashboard summary refresh หลัง confirm/replay/recon ได้

---

# 23. Final Import & Backfill Statement

```text
Import และ Backfill ของ Warehouse Reconciliation ERP Platform ต้องเป็น pipeline ที่รับไฟล์ผ่าน batch/raw staging, preview, column mapping, validation, alias review และ confirm ก่อนนำข้อมูลไปใช้จริง โดย Inquiry ต้องกลายเป็น Anchor Snapshot ส่วน System In-Out ต้องกลายเป็น Movement Ledger และเมื่อ Admin upload ย้อนหลัง 1–5 วัน ระบบต้องทำ Coverage Check, Stock Replay, Reconciliation, Issue/Task และ Summary Update ได้อย่าง traceable, auditable และไม่ hardcode schema จากไฟล์จริง
```

