# 24_current_remaining_work.md

# Current Remaining Work Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Remaining Work / Open Decisions / Next Action / Build Readiness Checklist  
**Version:** v1.0  
**Status:** Ready for Product Owner / AI Agent / Developer / Data Analyst Use  
**Primary Use:** ใช้สรุปว่าสิ่งที่ยังเหลือก่อนเริ่ม build จริงมีอะไรบ้าง ต้องตัดสินใจอะไร ต้องเตรียมข้อมูลอะไร ต้องทำ Data Profiling อะไร และต้องเริ่มงานลำดับไหน เพื่อกันแชทหายและใช้เป็น checkpoint ก่อนส่งให้ AI Agent สร้างระบบ

---

## 1. Purpose of This Document

เอกสารนี้คือไฟล์สรุป **งานที่ยังเหลือทั้งหมด ณ ปัจจุบัน** หลังจากวาง Blueprint ของ Warehouse Reconciliation ERP Platform แล้ว

เป้าหมายคือให้รู้ชัดว่า:

1. ตอนนี้ระบบคิดมาถึงระดับไหนแล้ว
2. อะไรพร้อมให้ AI Agent / Developer สร้างได้
3. อะไรยังต้องตัดสินใจก่อนเขียนระบบจริง
4. อะไรต้องใช้ข้อมูลจริงจากไฟล์ Inquiry / In-Out
5. อะไรเป็น P0 ที่ห้ามข้าม
6. อะไรเป็น P1/P2 ที่ทำทีหลังได้
7. ถ้าเริ่มสร้างเลย ต้องเริ่มจากอะไร
8. ถ้าอยากลด bug และความช้า ต้องกันตรงไหนก่อน

หลักสำคัญ:

```text
ระบบพร้อมในเชิงแนวคิดและเอกสารระดับสูงแล้ว
แต่ยังไม่ควรสร้าง reconciliation logic ลึกจนกว่าจะทำ Data Profiling และ Lock Schema v0
```

---

# 2. Current Overall Status

## 2.1 Status Summary

| Area | Current Status | Readiness |
|---|---|---:|
| Product Concept | ชัดเจนมาก | 90%+ |
| System Architecture | ชัดเจนระดับ blueprint | 88–92% |
| UX/UI Direction | ชัดเจน แต่ต้องทำ prototype | 85–90% |
| Data Model | มี blueprint แล้ว แต่ต้อง lock จากไฟล์จริง | 75–82% |
| API / Backend Spec | มี spec ครบระดับ build ได้ | 85–90% |
| Business Rules | ครบและเหมาะกับ operation | 90%+ |
| Validation Rules | ครบ แต่ต้อง tune ตามไฟล์จริง | 85–90% |
| Security / Audit | ครบและควรใช้ตั้งแต่ต้น | 90%+ |
| Import / Backfill Logic | ชัดเจน แต่ต้อง test ด้วยไฟล์จริง | 80–88% |
| Reconciliation Logic | Concept ชัด แต่ matching key ยังต้อง profiling | 70–78% |
| Build Readiness | พร้อมเริ่ม foundation ได้ | 75–80% |
| Production Readiness | ยังต้องผ่าน data profiling + UAT | 65–75% |

---

## 2.2 Current Professional Assessment

ระบบนี้มีแนวคิดที่ถูกทางมาก เพราะไม่ได้เริ่มจาก dashboard สวย ๆ แต่เริ่มจาก data truth layer:

```text
Physical Count
Field Movement
System In-Out
Inquiry Snapshot
Expected Stock / Stock Replay
```

จุดแข็งหลักตอนนี้:

1. แยก Inquiry เป็น Snapshot ไม่ใช่ Transaction
2. รองรับ Admin ไม่ upload ทุกวันด้วย Backfill + Stock Replay
3. มี Alias Review แทนการ hardcode ชื่อย่อ
4. มี Issue/Explanation แทนการให้คนไล่เช็คเอง
5. มี Permission/Audit/Retention ตั้งแต่ต้น
6. มี Out of Scope ชัดเจน ลดโอกาสระบบบาน

จุดที่ยังห้ามประมาท:

```text
Matching Key ของ Reconciliation ยังต้องใช้ข้อมูลจริงยืนยัน
Alias Strategy ต้อง seed จากไฟล์จริง
Location/DC mapping ต้องเห็น pattern จริง
Qty sign / movement type ต้อง verify จาก In-Out จริง
```

---

# 3. P0 Remaining Work — ต้องทำก่อนสร้าง Reconciliation จริง

P0 คือสิ่งที่ถ้าไม่ทำก่อน ระบบจะเสี่ยงคำนวณผิดหรือ build แล้วต้องรื้อ

---

## P0-001 — Data Profiling จากไฟล์ Inquiry / In-Out จริง

### Why Critical

ระบบนี้จะดีหรือพังขึ้นกับการเข้าใจไฟล์จริง เพราะ logic สำคัญทั้งหมดพึ่งพา:

- column จริง
- date จริง
- movement type จริง
- location/DC จริง
- customer/item raw text จริง
- qty/UOM จริง
- source reference จริง

### Required Inputs

ต้องมีไฟล์ตัวอย่างอย่างน้อย:

```text
Inquiry 3–5 ไฟล์ จากคนละวัน
System In-Out 3–5 วัน รวมวันที่ย้อนหลัง
ตัวอย่างป้ายหน้ากอง 10–20 ตัวอย่าง
ตัวอย่างชื่อย่อ customer/item
ตัวอย่าง location/DC ที่ไม่ตรงกัน
ตัวอย่าง REJ
ตัวอย่าง Pending Inbound
ตัวอย่าง export/report ที่ ERP Admin ใช้จริง
```

### What To Analyze

#### Header / Column

- column ทั้งหมดมีอะไรบ้าง
- column ไหนเป็น date
- column ไหนเป็น DC/location
- column ไหนเป็น customer/item
- column ไหนเป็น description รวม
- column ไหนเป็น qty/uom
- column ไหนเป็น movement type
- column ไหนเป็น source reference

#### Date / Time

- Inquiry date คือ snapshot date หรือ business date
- In-Out ใช้ posting date หรือ movement date
- มี movement time ไหม
- date เป็น พ.ศ. หรือ ค.ศ.
- Excel serial date มีไหม
- upload ย้อนหลังแล้ว date ในไฟล์ยังตรงไหม

#### Customer / Item

- MKMK / UPUP / UFUF อยู่ column ไหน
- ชื่อลูกค้าอยู่แยกหรือปนกับสินค้า
- W150 / RE T1 / SR CT1 อยู่ตรงไหน
- year เช่น 68/69 หรือ 69 อยู่ในชื่อหรือ column
- lot มีจริงไหม
- product group อื่น เช่น ไม้ แป้ง โมลาส เทกอง ใช้ pattern อะไร

#### Location / DC

- DC ใน ERP เทียบ warehouse ยังไง
- location จากหน้างานกับ ERP ไม่ตรงแบบไหน
- มี owner location กับ physical location หรือไม่
- location มี alias หลายชื่อไหม

#### Qty / UOM

- qty มี comma ไหม
- decimal ไหม
- negative sign ไหม
- OUT ใช้ qty ติดลบหรือ movement type บอกทิศทาง
- UOM อยู่ column แยกหรืออยู่ในชื่อสินค้า

#### Movement Type

- IN / OUT / TRANSFER / REJ ใช้ code อะไร
- transfer มี from/to ไหม
- REJ แสดงด้วย movement type หรือ location/reason
- adjustment มีไหม

### Output Required

ต้องได้:

```text
Data Profiling Report
Canonical Field Contract v0
Column Mapping Template v0
Movement Type Mapping v0
Qty Sign Rule v0
Location/DC Mapping v0
Alias Seed v0
Matching Key Candidate v0
Reconciliation Test Cases v0
Data Quality Risk List
```

### Acceptance Criteria

- [ ] รู้ required columns ของ Inquiry
- [ ] รู้ required columns ของ System In-Out
- [ ] รู้ว่า date ไหนต้องใช้กับ replay
- [ ] รู้ movement type codes จริง
- [ ] รู้ qty sign rule จริง
- [ ] รู้ customer/item/location alias patterns
- [ ] สร้าง test case จากข้อมูลจริงได้อย่างน้อย 10 เคส

---

## P0-002 — Canonical Field Contract v0

### Purpose

กำหนด field กลางที่ระบบจะใช้หลัง normalize เพื่อให้ทุกไฟล์ map มาเป็นภาษาเดียวกัน

### Required Canonical Fields

```text
business_date
snapshot_date
posting_date
movement_date
movement_datetime
warehouse_id
warehouse_raw
dc_id
dc_raw
location_id
location_raw
from_location_id
from_location_raw
to_location_id
to_location_raw
customer_id
customer_raw
item_id
item_raw
description_raw
product_group_id
year
lot
qty
uom
movement_type
source_ref
source_batch_id
source_row_id
confidence
```

### Open Questions

- business_date ควรอิง posting_date หรือ movement_date ใน In-Out
- Inquiry snapshot เป็นต้นวันหรือปลายวัน
- date_from ของ replay ต้องรวม movement วันแรกไหม
- lot จำเป็นทุก product group หรือไม่
- customer จำเป็นทุก item หรือบางกรณี unknown ได้

### Acceptance Criteria

- [ ] Canonical field list approved
- [ ] Required/optional by import type defined
- [ ] Data type ของแต่ละ field ชัดเจน
- [ ] Mapping จากไฟล์จริงทำได้
- [ ] AI Agent ใช้ contract นี้แทนการเดา schema

---

## P0-003 — Alias Strategy v0

### Purpose

ออกแบบ alias ให้รองรับชื่อย่อและชื่อไม่ตรงหลายประเภท โดยไม่ hardcode เฉพาะน้ำตาล

### Required Alias Types

```text
CUSTOMER_ALIAS
ITEM_ALIAS
PRODUCT_GROUP_ALIAS
LOCATION_ALIAS
DC_ALIAS
WAREHOUSE_ALIAS
UOM_ALIAS
LOT_PATTERN
```

### Known Pattern from Current Discussion

ตัวอย่างกลุ่มน้ำตาล:

```text
MKMK / UPUP / UFUF = Customer Alias
W150 / W100 / RE T1 / SR CT1 / ไฮโพลดิบ = Item Alias
69 / 68 = Year / season
```

ตัวอย่าง smart input:

```text
MKMK W150 69
```

Expected parse:

```json
{
  "customer_alias": "MKMK",
  "item_alias": "W150",
  "year": "69",
  "confidence": 95
}
```

### Important Rule

```text
Pattern น้ำตาลห้ามนำไปบังคับใช้กับไม้ แป้ง โมลาส เทกอง หรือสินค้าอื่นทั้งหมด
```

### Non-Sugar Strategy

#### Wood

Possible identity:

```text
material_type + size + grade + length + owner/customer
```

#### Flour

Possible identity:

```text
flour_type + brand + pack_size + lot + mfg_date
```

#### Molasses

Possible identity:

```text
molasses_type + tank/batch + customer + qty/uom
```

#### Bulk / เทกอง

Possible identity:

```text
bulk_type + pile_id/location + customer + estimated_qty
```

### Acceptance Criteria

- [ ] Alias tables แยกตาม type
- [ ] Alias Review Queue มี type, raw_text, suggested_master, confidence
- [ ] Low confidence ไม่ auto-map
- [ ] Admin map/create/reject ได้
- [ ] Mapping มี audit
- [ ] Product group strategy แยกได้

---

## P0-004 — Column Mapping Config v0

### Purpose

ป้องกันการ hardcode column และรองรับไฟล์ที่ header เปลี่ยน

### Required Mapping Templates

```text
INQUIRY_MAPPING_V0
SYSTEM_INOUT_MAPPING_V0
FIELD_COUNT_MAPPING_V0 optional
MASTER_DATA_MAPPING_V0
ALIAS_SEED_MAPPING_V0 optional
```

### Required by Import Type

#### Inquiry

```text
snapshot_date or business_date
warehouse_raw or dc_raw
item_raw or description_raw
qty_raw
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

#### System In-Out

```text
posting_date or movement_date
movement_type_raw
warehouse_raw or dc_raw
item_raw or description_raw
qty_raw
```

Recommended:

```text
location_raw or from/to location
customer_raw
year_raw
lot_raw
uom_raw
source_ref
```

### Acceptance Criteria

- [ ] Mapping save/reuse ได้
- [ ] Missing required mapping block normalize
- [ ] Header changed warning ได้
- [ ] AI Agent ไม่อ้าง column index ถาวร

---

## P0-005 — Reconciliation Test Case v0

### Purpose

ใช้ทดสอบว่า engine ตรวจถูก ไม่สร้าง false issue เยอะ และ explanation ใช้งานได้จริง

### Minimum Test Cases

```text
1. Field vs System matched
2. Field OUT 1,000 แต่ System OUT 950 → QTY_DIFF
3. Field movement มี แต่ System ไม่มี → SYSTEM_MISSING
4. System มี แต่ Field ไม่มี → FIELD_MISSING
5. Location ERP ไม่ตรงกับ location หน้างาน → LOCATION_DIFF
6. Unknown customer alias → UNKNOWN_ALIAS
7. Unknown item alias → UNKNOWN_ALIAS
8. Pending inbound ยังไม่ confirm → PENDING_INBOUND
9. REJ แจ้งแล้ว ERP ไม่ post → REJ_NOT_POSTED
10. Count ไม่ตรง expected stock → NEED_RECOUNT / QTY_DIFF
11. Inquiry upload ซ้ำวันเดียวกัน → SNAPSHOT_CONFLICT
12. Backfill 5 วันแล้ว In-Out ขาด 1 วัน → COVERAGE_WARNING
13. Open issue เกิน 45 วัน → retention skip
14. Checker ไม่มี session แล้ว submit count → block
15. Locked day แล้วแก้ movement → block
```

### Acceptance Criteria

- [ ] ทุก test case มี input data
- [ ] expected result_code ชัด
- [ ] expected explanation ชัด
- [ ] owner_role ชัด
- [ ] severity/confidence ชัด
- [ ] ใช้ทดสอบก่อน production ได้

---

# 4. P0 Open Decisions — ต้องตัดสินใจก่อน Build Architecture ลึก

---

## DEC-001 — Database Platform

### Options

| Option | Good For | Risk |
|---|---|---|
| Google Sheets + Apps Script | prototype เร็ว / user คุ้น | ช้า, schema หลวม, audit/permission จำกัด |
| Supabase / Postgres | web app จริง / scale MVP | ต้องออกแบบ schema/RLS/retention ดี |
| MySQL / Managed SQL | infra องค์กร | ต้องทำ auth/storage/job เองมากกว่า |

### Recommendation

```text
ถ้าจะสร้างจริงแบบ clean professional ให้เริ่ม Supabase/Postgres ได้เลย
ถ้ายังแค่ validate data flow อาจ prototype ด้วย Sheets ได้ แต่ต้องมี migration path
```

### Decision Required

- [ ] เลือก platform สำหรับ MVP จริง
- [ ] เลือก file storage
- [ ] เลือก scheduled job/cron mechanism
- [ ] เลือก auth approach

---

## DEC-002 — File / Photo / Export Storage

### Need To Decide

- upload source file เก็บไหม หรือเก็บเฉพาะ raw rows
- photo evidence เก็บที่ไหน
- export file เก็บกี่วัน
- private/public URL policy
- backup file เก็บที่ไหน

### Recommendation

```text
ใช้ private storage และให้ download ผ่าน permission-controlled API
```

---

## DEC-003 — Retention 45 วัน Policy

### Need To Decide

- ลบ raw เลยหรือ archive เป็น file ก่อน
- summary ต้องเก็บระดับไหน
- export log เก็บกี่เดือน
- audit log critical เก็บกี่เดือน
- photo evidence ที่ปิด issue แล้วเก็บกี่วัน

### MVP Recommendation

```text
Raw/staging active 45 วัน
Summary + issue/export/audit critical เก็บนานกว่า 45 วัน
Open issue source ห้ามลบ
Manual retention ต้องมี DRY_RUN ก่อน EXECUTE
```

---

## DEC-004 — Notification Channel Priority

### Options

```text
In-app = base channel
Telegram = quick alert group
LINE Messaging API = long-term LINE channel
Email = summary/export/system
```

### Important

```text
LINE Notify ไม่ควรเป็นฐานของระบบใหม่ ให้ design รองรับ LINE Messaging API
```

### Decision Required

- [ ] ใช้ Telegram ก่อนหรือ LINE Messaging API ก่อน
- [ ] ใครเป็น target group ของแต่ละ event
- [ ] event ไหนต้อง immediate
- [ ] event ไหนควร digest

---

## DEC-005 — Export Format for ERP Admin

### Need To Know

- ERP Admin ต้องการ column อะไร
- export เป็น XLSX/CSV/PDF
- selected rows หรือ filtered rows
- ใช้ source_ref อะไรกลับไป ERP
- REJ / SYSTEM_MISSING / QTY_DIFF ต้อง export format เดียวกันไหม

### Recommendation

ทำ export v0 จาก issue/recon result:

```text
issue_id
business_date
warehouse
location
customer
item
year
lot
movement_type
field_qty
system_qty
diff_qty
uom
suggested_action
source_ref
note
```

---

# 5. P1 Remaining Work — ควรทำหลัง P0 หรือทำคู่ขนานได้

---

## P1-001 — Prototype UI ก่อน dev หนัก

### Why

ลดความเสี่ยงสร้างระบบถูก logic แต่ user ใช้ยาก

### Prototype Pages

```text
1. Smart Inbound Input
2. Alias Review
3. Backfill Wizard
4. Issue Detail
5. Control Center
6. User Approval
7. Import Mapping Stepper
8. Retention Dry Run
```

### Acceptance Criteria

- [ ] Checker ใช้มือถือได้จริง
- [ ] Admin map alias ได้เร็ว
- [ ] Backfill Wizard ไม่ทำให้ user งง
- [ ] Issue Detail อ่านแล้วรู้ว่าต้องทำอะไร
- [ ] Control Center แสดง blocker ก่อน summary

---

## P1-002 — Summary Table Design

### Why

Dashboard จะช้าแน่ถ้า query raw data ทุกครั้ง

### Need To Design

```text
daily_recon_summary
daily_issue_summary
import_summary
coverage_summary
health_summary
retention_summary
```

### Acceptance Criteria

- [ ] Control Center อ่านจาก summary/result table
- [ ] Summary refresh หลัง import/replay/recon/issue update
- [ ] มี last_updated/stale warning

---

## P1-003 — Notification Rule v0

### Required Rules

```text
USER_PENDING_APPROVAL
UNKNOWN_ALIAS_CREATED
PENDING_INBOUND_CREATED
SYSTEM_MISSING
QTY_DIFF high severity
REJ_NOT_POSTED
DAILY_CLOSE_NOT_READY
EXPORT_READY
RETENTION_FAILED
DB_USAGE_CRITICAL
```

### Acceptance Criteria

- [ ] มี channel/rule/template/log
- [ ] มี cooldown
- [ ] มี send_once_per_issue
- [ ] secret ไม่ expose frontend

---

## P1-004 — Export Template v0

### Need To Build

- Issue List Export
- Daily Reconciliation Export
- REJ Report
- Pending Inbound Report
- Alias Review Report
- ERP Correction Template v0
- Retention Report
- Audit Report basic

### Acceptance Criteria

- [ ] export preview ก่อนสร้างไฟล์
- [ ] export log ทุกครั้ง
- [ ] ERP template มี permission เฉพาะ

---

# 6. P2 Remaining Work — เลื่อนไป Future Phase

P2 คือสิ่งที่มีประโยชน์ แต่ไม่ควรทำก่อน core reconciliation stable

---

## P2-001 — Capacity / Available Location Simulation

### Current Idea

ต้องรองรับ simulation ที่ยืดหยุ่น เช่น:

- Available Location
- เลือก ignore multi-lot rule ได้
- ลูกค้านอกอาจไม่สนใจ lot บางกรณี
- ถ้าเต็ม ให้ระบบดูว่าของกลุ่มเดียวกันย้ายไปไหนได้
- เน้นย้ายของน้อยไปหามาก
- อิง capacity ว่างพอไหม
- simulation ไม่สร้าง movement จริงจนกว่าจะ approve

### Why Future

ต้องมีข้อมูลที่แม่นก่อน:

```text
location capacity
current occupancy
customer/item/lot policy
daily stock position accurate
movement cost/constraint
```

### MVP Placeholder

```text
location.capacity_qty
location_type
customer_type INTERNAL/EXTERNAL
future simulation_runs table optional
```

---

## P2-002 — Advanced FIFO / Lot Aging

ทำทีหลังเมื่อ:

- lot/year reliable
- business rule FIFO clear
- UAT cases available

MVP ทำแค่:

```text
year/lot field
LOT_DIFF result code basic
lot missing warning
```

---

## P2-003 — Visual Warehouse Map

ทำทีหลังเมื่อ:

- location master clean
- coordinates/zone ready
- stock position reliable

MVP ทำแค่:

```text
location list
dashboard card
count required locations
```

---

## P2-004 — OCR / Advanced AI Matching

ทำทีหลังเมื่อ:

- มีรูปป้ายจำนวนมาก
- alias seed stable
- มี ground truth
- human review loop พร้อม

MVP ทำแค่:

```text
smart text input
photo evidence upload
alias review
```

---

# 7. Current Build-Allowed Scope

ตอนนี้สามารถเริ่ม build ได้ในส่วนที่ไม่ต้องรอไฟล์จริงลึกมาก:

```text
1. Project structure
2. Design system components
3. App shell / navigation
4. Auth / register / login
5. User approval / reject with audit
6. Role / permission / warehouse scope
7. Audit log infrastructure
8. Master data basic
9. Import batch shell
10. File preview UI
11. Column mapping UI structure
12. Alias review UI structure
13. Checker GPS session basic
14. Error/loading/empty state components
```

ยังไม่ควร build ลึกจน hardcode:

```text
1. Final normalize parser for real files
2. Final movement type mapping
3. Final stock replay inclusion rule
4. Final reconciliation matching key
5. Final ERP export template
6. Final alias seed for all product groups
```

---

# 8. Current Do Not Start Yet

ห้ามเริ่มสิ่งเหล่านี้ตอนนี้:

```text
Relocation optimizer full engine
Advanced FIFO full engine
Visual warehouse map full UI
OCR production
Native mobile app
Full offline sync
Direct ERP posting
Finance/costing module
Contract/custody billing
ML prediction dashboard
```

เหตุผล:

```text
ยังต้องพิสูจน์ data foundation และ reconciliation accuracy ก่อน
```

---

# 9. Data Questions That Must Be Answered

ก่อน lock schema/reconciliation ต้องตอบคำถามเหล่านี้:

## Inquiry

1. Inquiry มีวัน/เวลาของ snapshot ชัดไหม
2. Inquiry เป็นยอดปลายวันหรือตอนที่ export
3. Inquiry มี location/customer/item/lot ครบไหม
4. Inquiry มี row ซ้ำได้ไหม
5. Inquiry ใช้ DC หรือ warehouse code อะไร

## System In-Out

1. In-Out มี posting date กับ movement date แยกไหม
2. OUT qty เป็นบวกหรือลบ
3. movement_type codes คืออะไร
4. transfer มี from/to location ไหม
5. REJ แสดงเป็น movement type หรือ reason/location
6. source_ref/document number reliable ไหม

## Alias

1. MKMK / UPUP / UFUF คือ customer เสมอไหม
2. W150 / RE T1 / SR CT1 คือ item เสมอไหม
3. ปี 68/69 ต้องเก็บเป็น 69 หรือ full 68/69
4. สินค้าอื่นไม่มีตัวย่อ ต้องใช้ field ไหนแทน
5. location/DC mapping มี master อยู่แล้วไหม

## Count / Field

1. Checker นับตาม location หรือ item/customer ด้วย
2. หน้างานเขียนป้ายหน้ากองด้วยรูปแบบใด
3. แจ้งเข้าไม่รู้ข้อมูลครบบ่อยแค่ไหน
4. Pending inbound ควรรอ In-Out กี่วันก่อนเตือน
5. REJ ต้อง post ERP ภายในวันเดียวหรือวันถัดไป

## Retention

1. 45 วันนับจาก business_date หรือ created_at
2. raw file ต้อง archive ก่อนลบไหม
3. closed issue detail ต้องเก็บกี่วัน
4. audit critical ต้องเก็บกี่เดือน

---

# 10. Recommended Immediate Next Actions

## Step 1 — เก็บไฟล์จริง

ต้องเตรียม:

```text
Inquiry 3–5 ไฟล์
System In-Out 3–5 วัน
ตัวอย่างป้ายหน้ากอง 10–20 ตัวอย่าง
ตัวอย่างชื่อย่อ customer/item
ตัวอย่าง location/DC mismatch
ตัวอย่าง REJ
ตัวอย่าง pending inbound
ตัวอย่าง export ที่ ERP Admin ต้องการ
```

---

## Step 2 — ทำ Data Profiling Sprint

Duration suggested:

```text
1–3 วัน ถ้าไฟล์ครบและไม่ซับซ้อน
3–7 วัน ถ้า schema ไฟล์หลากหลายหรือข้อมูลสกปรก
```

Output:

```text
Data Profiling Report
Canonical Field Contract v0
Column Mapping Template v0
Alias Seed v0
Matching Key Candidate v0
Data Quality Risk List
Test Case v0
```

---

## Step 3 — Lock MVP Scope

ใช้เอกสาร:

```text
17_out_of_scope.md
18_development_plan.md
20_acceptance_criteria.md
28_mvp_scope_lock.md if created later
```

Lock ว่า MVP ทำแค่:

```text
Foundation + Import/Alias + Anchor/Ledger/Replay + Reconciliation/Issue + Control/Export/Notification/Retention
```

---

## Step 4 — Decide Platform

เลือกอย่างใดอย่างหนึ่ง:

### Option A — Supabase/Postgres MVP

เหมาะถ้า:

- ตั้งใจทำ web app จริง
- ต้องการ role/permission/audit ที่ดี
- ต้องการ scale ต่อ
- ต้องการ retention/job/query ดี

### Option B — Google Sheets / Apps Script Prototype

เหมาะถ้า:

- ต้องการ validate workflow เร็ว
- ข้อมูลยังน้อย
- user ยังต้องช่วยดู sheet
- ยอมรับว่าจะ migrate ภายหลัง

Recommended professional path:

```text
ถ้าความตั้งใจคือระบบจริง ให้เริ่ม Supabase/Postgres
ถ้ายังไม่มั่นใจ schema ให้ทำ data profiling/prototype บางส่วนก่อน แต่โครงคิดยังต้องออกแบบเป็น database จริง
```

---

## Step 5 — Build Foundation

เริ่ม build ได้ทันทีหลังเลือก platform:

```text
1. File structure
2. Design system
3. App shell
4. Auth/session
5. User approval
6. Permission/warehouse scope
7. Audit log
8. Master data basic
9. Import batch shell
10. Alias queue shell
```

---

# 11. Build Readiness Checklist

## Documentation Ready

- [x] Product Brief
- [x] Structure
- [x] Module Spec
- [x] Workflow
- [x] UX/UI Spec
- [x] Data Model
- [x] API Spec
- [x] Business Rules
- [x] Validation Rules
- [x] Audit/Security
- [x] Dashboard/KPI
- [x] Error/State
- [x] Out of Scope
- [x] Development Plan
- [x] AI Agent Task
- [x] Acceptance Criteria
- [x] Database/Retention
- [x] Notification
- [x] Import/Backfill
- [x] Current Remaining Work

## Data Readiness

- [ ] Inquiry sample files
- [ ] In-Out sample files
- [ ] Field/pile label samples
- [ ] REJ examples
- [ ] Location/DC mapping examples
- [ ] Export template examples
- [ ] Alias seed v0
- [ ] Test case v0

## Technical Readiness

- [ ] Platform selected
- [ ] Database/storage selected
- [ ] Auth approach selected
- [ ] Retention job mechanism selected
- [ ] Notification channel priority selected
- [ ] Export format v0 selected

## UX Readiness

- [ ] Smart Inbound prototype
- [ ] Alias Review prototype
- [ ] Backfill Wizard prototype
- [ ] Issue Detail prototype
- [ ] Control Center prototype

---

# 12. Risk Register Snapshot

| Risk | Severity | Likelihood | Mitigation |
|---|---:|---:|---|
| ไฟล์จริง schema ไม่ตรง assumption | High | High | Data Profiling before logic lock |
| Alias ผิดแล้ว match ผิด | High | High | Alias Review + confidence + audit |
| Inquiry ถูกใช้ผิดเป็น transaction | Critical | Medium | Business rule + test case |
| In-Out qty sign ผิด | High | Medium | Qty sign profiling + config |
| Location/DC mapping ผิด | High | High | Location alias + master mapping |
| Dashboard ช้า | High | Medium | Summary/result tables only |
| Retention ลบ evidence | Critical | Medium | Skip open issue + dry run + log |
| Notification spam | Medium | High | cooldown + digest + send once |
| Checker mobile ใช้ยาก | Medium | Medium | prototype mobile-first |
| Permission หลุด | Critical | Medium | backend guard + tests |
| Build บานไป simulation/fifo | High | Medium | Out of Scope + phase gate |

---

# 13. Final Recommendation

ถ้าต้องเริ่มสร้างระบบหลังจากเอกสารชุดนี้ ให้ใช้ลำดับนี้:

```text
1. ทำ Data Profiling จากไฟล์จริงทันที
2. Lock Canonical Field Contract v0
3. ทำ Alias Seed v0
4. เลือก Platform: Supabase/Postgres ถ้าจะ build จริง
5. สร้าง Foundation: Auth/User/Permission/Audit
6. สร้าง Import Staging + Column Mapping
7. สร้าง Alias Review
8. สร้าง Inquiry Anchor + In-Out Ledger
9. สร้าง Stock Replay
10. สร้าง Reconciliation + Issue Detail
11. สร้าง Control Center + Export + Notification + Retention
```

อย่าเริ่มจาก:

```text
Simulation
Relocation Optimizer
Advanced FIFO
Visual Map
OCR
Native App
```

---

# 14. Final Current Remaining Work Statement

```text
งานที่เหลือสำคัญที่สุดตอนนี้ไม่ใช่การเพิ่ม feature ใหม่ แต่คือการทำ Data Profiling จากไฟล์ Inquiry/In-Out จริง, lock Canonical Field Contract v0, สร้าง Alias Strategy v0, กำหนด Matching Key Candidate และตัดสินใจ platform/database ก่อนเริ่ม build reconciliation logic ลึก ส่วน foundation อย่าง Auth, User Approval, Permission, Audit, Import Shell, Alias Review UI และ Design System สามารถเริ่มสร้างได้ทันทีโดยไม่ต้องรอไฟล์จริงทั้งหมด หากยังยึดกฎว่าไม่ hardcode schema, ไม่ hardcode alias และทุก critical action ต้องมี validation/audit
```

