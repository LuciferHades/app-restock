# Warehouse Reconciliation ERP Platform — Structure Blueprint

## 0. ภาพรวมระบบ

ระบบนี้คือ Web App สำหรับบริหารคลังสินค้าและตรวจสอบความถูกต้องระหว่าง **ของจริง**, **ข้อมูลที่หน้างานแจ้ง**, และ **ข้อมูลใน ERP** โดยออกแบบให้ใช้งานจริงในคลังน้ำตาลที่มีข้อจำกัดเรื่องกองใหญ่, Lot ตรวจยาก, Location เคลื่อน, ERP ตัดรอบดึก และข้อมูลไม่ได้ Real-time

ระบบหลักแบ่งเป็น 2 ระบบใหญ่ แต่ใช้ Core ร่วมกัน

```text
Warehouse Reconciliation ERP Platform
│
├── Core Platform
│   ├── Master Data
│   ├── User / Role / Permission
│   ├── Warehouse Structure
│   ├── Snapshot Control
│   ├── Movement Engine
│   ├── Approval
│   ├── Investigation
│   ├── Export
│   ├── Archive / Retention
│   └── Audit Log
│
├── Module A: Restock / Stock Audit ERP
│   └── ใช้ตรวจรอบใหญ่, เคลียร์ยอด, แก้ Location, Adjust, Reconcile ERP
│
└── Module B: Daily Inventory Control ERP
    └── ใช้ควบคุม Inventory รายวัน, In-Out, Before/After Count, REJ, Planning
```

เป้าหมายหลักของระบบ

```text
1. รู้ว่าของจริงอยู่ที่ไหน
2. รู้ว่า ERP บอกว่าอยู่ที่ไหน
3. รู้ว่าใครแจ้งเข้า/ออกอะไร
4. รู้ว่า ERP ตัดครบหรือไม่
5. รู้ว่า Location / Qty / Lot ตรงหรือไม่
6. รู้ว่า REJ ถูกย้ายเข้าระบบจริงหรือไม่
7. รู้ว่าควร Transfer / Adjust / Investigate อะไร
8. รู้ว่าปิดวันหรือปิดรอบได้หรือยัง
9. ลดการไล่ Excel ด้วยมือ
10. ทำให้หัวหน้าตัดสินใจง่ายขึ้น
```

หลักคิดกลาง

```text
Physical Count = ความจริงจากของที่เห็น
Field Movement = ความจริงจากหน้างาน
System In-Out = ความจริงจาก ERP movement
Inquiry = Snapshot ของยอดคงเหลือในระบบ
Expected Stock = ค่าที่ระบบคำนวณเอง
```

---

# 1. มีกี่เรื่อง แบ่งออกได้กี่โมดูล

## 1.1 Core Platform Modules

```text
1. Master Data Module
2. Warehouse Structure Module
3. User / Role / Permission Module
4. Snapshot Control Module
5. Movement Engine Module
6. Reconciliation Engine Module
7. Approval Center Module
8. Investigation Module
9. Notification / Alert Module
10. Export Center Module
11. Archive / Retention Module
12. Audit Log Module
```

## 1.2 Business Modules

```text
13. Restock / Stock Audit Module
14. Daily Inventory Control Module
15. Field In-Out Notice Module
16. Before / After Count Module
17. REJ Damage Module
18. Internal Transfer / Split Stock Module
19. Cross-Warehouse Transfer / Custody Stock Module
20. Storage Capacity Module
21. Storage Planning & Simulation Module
22. Outbound Planning Module
23. Checker Task Module
24. Lot / FIFO Risk Module
25. Mobile Count Module
26. Dashboard Module
```

## 1.3 Module Map

```text
Core Platform
│
├── Master Data
├── Warehouse Structure
├── User / Permission
├── Snapshot Control
├── Movement Engine
├── Reconciliation Engine
├── Approval
├── Investigation
├── Export
├── Archive
└── Audit Log

Restock / Stock Audit ERP
│
├── Import Inquiry
├── System Stock Normalize
├── Mobile Physical Count
├── Analysis Result
├── Recount Queue
├── Transfer Proposal
├── Adjustment Proposal
├── Lot / FIFO Check
├── Investigation
└── Audit Round Close

Daily Inventory Control ERP
│
├── Opening Stock
├── Field In-Out Notice
├── Before / After Count
├── System In-Out Import
├── Expected Stock
├── Movement Reconciliation
├── REJ Notice
├── Daily Close
└── Daily Lock

Planning & Control
│
├── Capacity by Location
├── Storage Simulation
├── Outbound Plan
├── Checker Task
├── Cross-Warehouse Custody
└── Internal Split Transfer
```

---

# 2. แต่ละโมดูลทำอะไรได้บ้าง

## 2.1 Master Data Module

ใช้จัดการข้อมูลตัวเลือกกลางทั้งหมด ไม่ให้ฝัง hardcode ในหน้าเว็บ

### ทำอะไรได้

```text
เพิ่มข้อมูล
แก้ไขข้อมูล
ปิดใช้งาน
ค้นหา
จัดกลุ่ม
ตั้งค่า alias
ตั้งค่า default UOM
ตั้งค่า location type
ตั้งค่า lot rule
ตั้งค่า capacity
```

### ข้อมูลย่อย

```text
Location
Customer
Item
Item Alias
Item Type
Variant
Year
UOM
Lot
FIFO Date
Reason Code
Zone
Warehouse
DC
User
Role
Permission
```

### กฎสำคัญ

```text
ถ้าข้อมูลเคยถูกใช้งานแล้ว ห้ามลบจริง
ให้ใช้ active = FALSE
ถ้าหน้างานเพิ่มเอง ให้ status = WAITING_REVIEW
Admin ต้องอนุมัติก่อนเป็น master จริง
```

### สถานะ Master

```text
ACTIVE
INACTIVE
TEMP
WAITING_REVIEW
REJECTED
MERGED
```

---

## 2.2 Warehouse Structure Module

ใช้วางโครงคลัง

### โครงสร้าง

```text
DC
Warehouse
Zone
Location
Sub Location
Nearby Location
REJ Location
Temporary Location
```

### Location Type

```text
NORMAL
TEMP
REJ
LOADING
RECEIVING
OVERFLOW
BLOCKED
CUSTODY
QUARANTINE
```

### ใช้ทำอะไร

```text
รู้ว่า Location ไหนอยู่โซนไหน
รู้ว่า Location ไหนใกล้กัน
รู้ว่า Location ไหนเป็น REJ
รู้ว่า Location ไหนรับของได้
รู้ว่า Location ไหนห้ามใช้
รู้ว่า Location ไหนเต็ม
```

---

## 2.3 User / Role / Permission Module

ใช้ควบคุมสิทธิ์

### Roles หลัก

```text
Checker
Supervisor
Planner
Admin
ERP Admin
External Warehouse
Viewer
```

### สิทธิ์หลัก

```text
Checker
- นับจริง
- แจ้ง In-Out
- แจ้ง REJ
- รับ task
- เพิ่ม temp master

Supervisor
- ตรวจผล
- อนุมัติ Transfer
- อนุมัติ REJ บางกรณี
- ส่ง Investigation
- ปิด task

Planner
- วางแผน In/Out
- Simulation
- Assign Checker
- กำหนด Location รับเข้า

Admin
- Import Inquiry
- Import System In-Out
- จัดการ Master
- ตั้งค่า Capacity
- Archive
- Export

ERP Admin
- รับใบแก้ ERP
- Post Transfer
- Post Adjust
- Post REJ
- Mark Posted in ERP

External Warehouse
- ส่งคำขอโอน
- ส่งคำขอฝากของ
- รอปลายทางกดรับ

Viewer
- ดูรายงาน
- Export ตามสิทธิ์
- ห้ามแก้
```

---

## 2.4 Snapshot Control Module

ใช้ควบคุมข้อมูล Inquiry ที่ดึงมาจาก ERP

### ปัญหาที่แก้

```text
Inquiry ไม่มีวันที่เวลา
Upload ซ้ำแล้วข้อมูลเบิ้ล
ดึงคนละเวลาแต่ใช้เป็นวันเดียวกัน
เอา Inquiry มาบวกซ้ำกับ Movement
```

### ทำอะไรได้

```text
สร้าง snapshot_id
กำหนด business_date
กำหนด snapshot_time
บันทึก uploaded_at
ป้องกัน active snapshot ซ้ำ
Replace snapshot
Archive snapshot เก่า
คำนวณ checksum
```

### กฎ

```text
1 business_date มี active inquiry ได้ 1 ชุด
Upload ซ้ำต้อง Replace / Archive / Cancel
ห้าม Append ทับ
Inquiry เป็น Snapshot เท่านั้น
Inquiry ไม่ใช่ Transaction
```

---

## 2.5 Movement Engine Module

หัวใจของระบบทั้งหมด

### Movement Type

```text
IN
OUT
TRANSFER
REJ
RETURN
ADJUST
COUNT
SPLIT_TRANSFER
CUSTODY_IN
CUSTODY_OUT
PLAN_IN
PLAN_OUT
```

### แนวคิด

ทุกการเคลื่อนไหวต้องเป็น Event

```text
ห้าม update stock ตรง ๆ โดยไม่มี movement log
```

### ข้อมูลที่ต้องมีทุก Movement

```text
movement_id
business_date
movement_type
from_location
to_location
customer
item
variant
year
lot_code
fifo_date
uom
qty
source
reported_by
reported_at
status
```

---

## 2.6 Reconciliation Engine Module

ใช้เปรียบเทียบข้อมูลหลายแหล่ง

### เทียบอะไรกับอะไร

```text
Inquiry vs Physical Count
Field In-Out vs System In-Out
Expected Stock vs Latest Inquiry
Expected Stock vs Physical Count
REJ Notice vs System In-Out
Transfer Proposal vs ERP Posted
Plan vs Actual Movement
```

### ผลลัพธ์หลัก

```text
MATCHED
SYSTEM_MISSING
FIELD_MISSING
QTY_DIFF
LOCATION_DIFF
ITEM_DIFF
LOT_DIFF
DUPLICATE_POSTED
POSTED_EXTRA
PHYSICAL_MOVEMENT_DIFF
REJ_NOT_POSTED
REJ_UNREPORTED
NEED_RECOUNT
NEED_INVESTIGATION
```

---

## 2.7 Approval Center Module

ใช้ให้หัวหน้าหรือ Admin อนุมัติ

### อนุมัติอะไรได้บ้าง

```text
Transfer Proposal
Adjustment Proposal
Temp Master
REJ Exception
Capacity Over
Multi-Lot Accept
Cross-Warehouse Receive
Investigation Result
Daily Close
Audit Round Close
```

### สถานะ

```text
DRAFT
PENDING_APPROVAL
APPROVED
REJECTED
HOLD
EXPORTED
POSTED_IN_ERP
CLOSED
```

---

## 2.8 Investigation Module

ใช้กับปัญหาที่ตัดสินไม่ได้ทันที

### สาเหตุที่ต้อง Investigation

```text
ระบบมี แต่ของไม่เจอ
หน้างานแจ้ง แต่ ERP ไม่ตัด
ERP ตัด แต่หน้างานไม่แจ้ง
Qty ไม่ตรง
Location ไม่ตรง
Lot ไม่ตรง
REJ ไม่เข้า ERP
Transfer ค้าง
ของฝากจากคลังอื่น
```

### Issue Type

```text
LIKELY_SHIPPED
LIKELY_TRANSFERRED
NOT_POSTED
COUNT_ERROR
WRONG_LOCATION
QTY_DIFF
LOT_ISSUE
REJ_ISSUE
CUSTODY_STOCK
UNKNOWN
```

---

## 2.9 Notification / Alert Module

ใช้แจ้งเตือน

### Trigger

```text
รับเข้าเกิน capacity
รับเข้า location ที่มีหลาย lot
REJ รอ post
System In-Out ไม่ตรง Field Notice
Upload Inquiry ซ้ำ
Issue ค้างเกิน SLA
Daily Close ไม่ผ่าน
Cross-Warehouse Transfer รอรับ
```

### ผู้รับ

```text
Checker
Supervisor
Planner
Admin
ERP Admin
```

---

## 2.10 Export Center Module

ใช้สร้างไฟล์หรือเอกสารให้ไปแก้ ERP

### Export หลัก

```text
ใบย้าย Location
ใบปรับยอด
ใบ REJ
ใบ Investigation
ใบ Movement Reconciliation
ใบ Daily Close
ใบ Audit Close
ใบ Cross-Warehouse Custody
```

### สถานะหลัง Export

```text
EXPORTED
SENT_TO_ERP
POSTED_IN_ERP
VERIFIED
CLOSED
```

---

## 2.11 Archive / Retention Module

ใช้กันระบบช้า

### Policy

```text
Active Data = 45 วัน
Archive Data = เกิน 45 วัน
Dashboard โหลดเฉพาะ Active
Archive อ่านอย่างเดียว
```

### ห้าม Archive ถ้ายัง

```text
OPEN
WAIT_RECOUNT
WAIT_INVESTIGATION
PENDING_APPROVAL
SYSTEM_MISSING
QTY_DIFF
REJ_NOT_POSTED
```

### Archive ได้เมื่อ

```text
MATCHED
CLOSED
POSTED_IN_ERP
RESOLVED
LOCKED
```

---

# 3. Business Modules แบบละเอียด

## 3.1 Module A: Restock / Stock Audit ERP

ใช้กับการตรวจสต๊อกรอบใหญ่ หรือรอบเคลียร์ยอด

### Flow

```text
Import Inquiry
↓
Normalize System Stock
↓
Mobile Count
↓
Analysis
↓
Recount Queue
↓
Transfer Proposal
↓
Adjustment Proposal
↓
Investigation
↓
Approval
↓
Export ERP Action
↓
Posted in ERP
↓
Archive Audit Round
```

### ใช้ตอบคำถาม

```text
ของจริงอยู่ไหน
ERP คิดว่าอยู่ไหน
Location ผิดไหม
Qty ผิดไหม
Lot ผิดไหม
ต้อง Transfer ไหม
ต้อง Adjust ไหม
ต้องสอบสวนไหม
```

### Submodules

```text
Audit Batch
Inquiry Import
System Stock
Physical Count
Analysis Result
Recount Queue
Transfer Proposal
Adjustment Proposal
Lot / FIFO Check
Investigation
Audit Close
```

---

## 3.2 Module B: Daily Inventory Control ERP

ใช้บริหาร Inventory รายวันหลังเคลียร์ยอด

### Flow

```text
Upload Inquiry
↓
Set Opening Stock
↓
Field In-Out Notice
↓
Before / After Count
↓
Upload System In-Out วันถัดไป
↓
Movement Reconciliation
↓
Expected Stock Calculation
↓
Count Moving Locations
↓
Investigation / Correction
↓
Close Day
↓
Daily Lock
↓
Archive
```

### ใช้ตอบคำถาม

```text
วันนี้เข้าอะไร
วันนี้ออกอะไร
หน้างานแจ้งครบไหม
ERP ตัดครบไหม
Location ที่มี movement ตรงไหม
REJ เข้า ERP จริงไหม
ปิดวันได้ไหม
```

---

## 3.3 Field In-Out Notice Module

ใช้ให้หน้างานแจ้งการเคลื่อนไหวจริง

### Movement ที่รองรับ

```text
IN
OUT
TRANSFER
REJ
RETURN
CUSTODY_IN
CUSTODY_OUT
```

### Fields หลัก

```text
movement_type
from_location
to_location
customer
item
year
lot_code
qty
uom
reported_by
reported_at
remark
```

### สถานะ

```text
DRAFT
REPORTED
WAIT_SYSTEM_POST
MATCHED
PROBLEM
CLOSED
```

---

## 3.4 Before / After Count Module

ใช้พิสูจน์ว่าของจริงเปลี่ยนตาม Movement หรือไม่

### สูตร

```text
IN:
Before + Qty = After

OUT:
Before - Qty = After

TRANSFER:
From Before - Qty = From After
To Before + Qty = To After

REJ:
From Before - Qty = From After
REJ Location + Qty
```

### ถ้าไม่ตรง

```text
PHYSICAL_MOVEMENT_DIFF
```

---

## 3.5 REJ Damage Module

ใช้แจ้งสินค้าเสียหาย

### Flow

```text
เลือก Location ต้นทาง
↓
เลือกสินค้า / ปี / Lot / Qty
↓
บันทึก REJ
↓
ระบบสร้าง REJ Notice
↓
ขึ้นหน้าสรุปให้แคปส่ง Line
↓
ระบบถือว่า From Location ลด และ W8-REJ เพิ่ม
↓
วันถัดไปเทียบ System In-Out
```

### Result

```text
REJ_MATCHED
REJ_NOT_POSTED
REJ_UNREPORTED
REJ_QTY_DIFF
REJ_LOCATION_DIFF
REJ_ITEM_DIFF
REJ_LOT_DIFF
```

---

## 3.6 Internal Transfer / Split Stock Module

ใช้กรณีเอาเศษจาก Location หนึ่งไปฝากอีก Location

### แนวคิดสำคัญ

ต้องแยก

```text
owner_location = Location ที่ของเป็นเจ้าของ
physical_location = Location ที่ของวางจริง
```

### ตัวอย่าง

```text
owner_location = W8-09
physical_location = W8-06
qty = 120
reason = เศษฝาก
```

### หน้าแสดงผล

ดู Location 9

```text
อยู่จริงที่ W8-09 = 4,000
ฝากไว้ W8-06 = 120
รวมของ Location 9 = 4,120
```

ดู Location 6

```text
ของ W8-06 เอง = 3,500
รับฝากจาก W8-09 = 120
รวมที่วางจริง W8-06 = 3,620
```

### Status

```text
INTERNAL_TRANSFER
SPLIT_STACK
TEMP_HOLD
RETURN_PENDING
CLOSED
```

---

## 3.7 Cross-Warehouse Transfer / Custody Stock Module

ใช้กรณีคลังอื่นเอาของมาฝาก หรือโอนข้ามคลัง

### กรณี 1: คลังอื่นไม่ได้ใช้ระบบนี้

```text
Admin บันทึก Manual Deposit
Checker ตรวจรับ
ระบบแยกเป็น Custody Stock
```

### กรณี 2: คลังอื่นใช้ระบบเหมือนกัน

```text
ต้นทางสร้าง Transfer Request
ปลายทางกดรับ
Checker ตรวจรับ
ERP Admin post
```

### Concept

```text
stock_owner
physical_keeper
erp_owner
responsibility_note
expected_return_date
```

### Status

```text
REQUESTED
SENT
WAIT_DESTINATION_RECEIVE
DESTINATION_RECEIVED
ERP_TRANSFER_PENDING
POSTED
CLOSED
```

---

## 3.8 Storage Capacity Module

ใช้ตั้งค่าความจุแต่ละ Location

### ทำอะไรได้

```text
Set capacity by location
Edit capacity
ตั้ง allowed item
ตั้ง allowed customer
ตั้ง max bags
ตั้ง warning threshold
```

### Fields

```text
location
max_bag_capacity
current_qty
reserved_qty
available_capacity
allowed_item_type
allowed_customer
allowed_lot_rule
warning_threshold
```

### Warning

```text
NEAR_CAPACITY
CAPACITY_EXCEEDED
REQUIRES_APPROVAL
```

---

## 3.9 Storage Planning & Simulation Module

ใช้วางแผนรับเข้า/เก็บของล่วงหน้า

### เป้าหมาย

```text
ของที่จะเข้าพรุ่งนี้ควรลงที่ไหน
ตอนนี้เหลือพื้นที่เท่าไร
ถ้าวันนี้มีแผนออก จะว่างเท่าไร
ลง location นี้แล้วเกิน capacity ไหม
มีหลาย lot อยู่แล้วไหม
เก็บร่วมกับของเดิมได้ไหม
```

### สูตร

```text
Projected Capacity =
Current Inquiry
- Planned OUT วันนี้
+ Planned IN วันนี้/พรุ่งนี้
+ Pending Transfer In
- Pending Transfer Out
```

### Output

```text
Recommended Location
Alternative Location
Capacity Warning
Multi-Lot Warning
FIFO Risk
Admin Approval Required
```

---

## 3.10 Outbound Planning Module

ใช้วางแผนเอาออก

### Flow

```text
Planner สร้างแผน OUT
↓
เลือก Location / SKU / Lot / Qty
↓
Assign Checker
↓
Checker ตรวจจริง
↓
ถ้าตรง → READY_TO_OUT
ถ้าไม่ตรง → ISSUE_FOUND
```

### Status

```text
PLANNED
ASSIGNED
CHECKED
READY_TO_OUT
ISSUE_FOUND
CANCELLED
```

---

## 3.11 Checker Task Module

ใช้ส่งงานให้หน้างาน

### Task Type

```text
COUNT
RECOUNT
CHECK_BEFORE_IN
CHECK_AFTER_IN
CHECK_BEFORE_OUT
CHECK_AFTER_OUT
REJ_CHECK
TRANSFER_RECEIVE
OUTBOUND_VERIFY
CUSTODY_RECEIVE
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

---

## 3.12 Lot / FIFO Risk Module

ใช้ตรวจ Lot เท่าที่ทำได้จริง

### Lot Mode

```text
BULK_COUNT
LOT_VERIFY
PARTIAL_VERIFY
ASSUMED_FIFO
```

### Lot Status

```text
LOT_VERIFIED
LOT_PARTIAL
LOT_UNCHECKED
LOT_MISMATCH
```

### FIFO Status

```text
FIFO_OK
FIFO_WARNING
FIFO_VIOLATION
NEED_LOT_VERIFY
```

### Multi-Lot Warning

ถ้า Location มีหลาย Lot อยู่แล้ว แล้วจะ IN เพิ่ม

```text
ระบบเตือน
ถ้าผู้ใช้กดตกลง
ระบบบันทึกได้
แต่แจ้ง Admin ทันที
```

---

# 4. ข้อจำกัด

## 4.1 ข้อจำกัดจาก ERP

```text
ERP ไม่ Real-time
System In-Out ได้ย้อนหลัง
ข้อมูลวันนี้อาจยังไม่ครบ
ตัดรอบประมาณ 23:59:59
Inquiry ไม่มี timestamp จริง
Inquiry เป็นแค่ยอดคงเหลือ ณ เวลาดึง
```

## 4.2 ข้อจำกัดจากการนับจริง

```text
กองน้ำตาลหนัก
ตรวจ Lot ทุกกองไม่ได้
บาง Location เข้าถึงยาก
กองซ้อนกัน
หน้างานอาจนับได้แค่ Qty รวม
```

## 4.3 ข้อจำกัดจากคนใช้งาน

```text
ลืมแจ้ง movement
แจ้งผิด location
แจ้งผิด lot
แจ้งช้า
นับผิด
กดผิด
ใช้มือถือในพื้นที่สัญญาณไม่ดี
```

## 4.4 ข้อจำกัดจาก Google Sheets

```text
แถวเยอะแล้วช้า
Query หนักทำให้ Web App ช้า
Concurrent users จำกัด
Trigger และ Apps Script quota จำกัด
ข้อมูลซ้ำง่ายถ้าไม่คุม batch_id
```

## 4.5 ข้อจำกัดเชิงออกแบบ

```text
อย่าเชื่อ ERP เป็น realtime truth
อย่าบังคับ Lot ทุกกอง
อย่าให้ Dashboard โหลดข้อมูลเก่าทั้งหมด
อย่าให้แก้ย้อนหลังโดยไม่มี log
อย่ารวม Restock Audit กับ Daily Control จน logic ปนกัน
```

---

# 5. การเชื่อมโยงของข้อมูล

## 5.1 Truth Model

```text
Inquiry
= Snapshot Truth

System In-Out
= ERP Movement Truth

Field Movement
= Operational Truth

Physical Count
= Physical Truth

Expected Stock
= Calculation Truth
```

## 5.2 Flow เชื่อมข้อมูลรายวัน

```text
Upload Inquiry ตั้งต้น
↓
Set Opening Stock
↓
Field In-Out Notice
↓
Before / After Count
↓
Expected Stock Calculation
↓
Physical Count เฉพาะ Location ที่เคลื่อนไหว
↓
Upload System In-Out วันถัดไป
↓
Field vs System Reconciliation
↓
Investigation / Correction
↓
Close Day
```

## 5.3 Flow เชื่อมข้อมูล Restock Audit

```text
Import Inquiry
↓
Normalize System Stock
↓
Physical Count
↓
Compare System vs Actual
↓
Transfer / Adjust / Recount / Investigation
↓
Approval
↓
Export to ERP
↓
Posted in ERP
↓
Archive Audit Round
```

## 5.4 Expected Stock Formula

```text
Expected Stock =
Opening Stock
+ Field IN
- Field OUT
+ Transfer In
- Transfer Out
- REJ Out
+ REJ In
+ Custody In
- Custody Out
```

## 5.5 สิ่งที่ห้ามทำ

```text
ห้ามเอา Latest Inquiry มาบวก Field Movement ซ้ำ
ห้าม Append Inquiry ซ้ำในวันเดียว
ห้ามใช้ System In-Out เป็นของวันนี้ถ้ายังไม่ตัดรอบ
ห้าม Adjust โดยไม่ผ่าน Recount หรือ Investigation
ห้ามลบข้อมูลที่มี Audit Trail
```

## 5.6 Matching Logic

ใช้ key หลายตัว ไม่ใช้ Qty อย่างเดียว

```text
business_date
movement_type
location
item
customer
year
lot_code
uom
qty
time_window
source
```

### Matching Result

```text
EXACT_MATCH
PARTIAL_MATCH
NO_MATCH
CONFLICT
DUPLICATE
```

---

# 6. Database Schema: Google Sheets Structure

## 6.1 Core Sheets

```text
CONFIG
MASTER_LOCATION
MASTER_CUSTOMER
MASTER_ITEM
MASTER_ITEM_ALIAS
MASTER_VARIANT
MASTER_YEAR
MASTER_UOM
MASTER_LOT
MASTER_REASON
MASTER_ZONE
MASTER_USER
MASTER_ROLE
MASTER_PERMISSION
AUDIT_LOG
APPROVAL_LOG
EXPORT_LOG
ARCHIVE_LOG
```

## 6.2 Snapshot / ERP Import Sheets

```text
INQUIRY_SNAPSHOT
INQUIRY_RAW
SYSTEM_STOCK
SYSTEM_INOUT_BATCH
SYSTEM_INOUT_RAW
SYSTEM_INOUT_NORMALIZED
IMPORT_LOG
```

## 6.3 Restock / Audit Sheets

```text
AUDIT_BATCH
COUNT_TXN
ANALYSIS_RESULT
RECOUNT_QUEUE
TRANSFER_PROPOSAL
TRANSFER_PROPOSAL_LINE
ADJUSTMENT_PROPOSAL
INVESTIGATION_REQUEST
AUDIT_CLOSE
```

## 6.4 Daily Inventory Sheets

```text
DAILY_SESSION
OPENING_STOCK
FIELD_MOVEMENT_NOTICE
FIELD_MOVEMENT_LINE
MOVEMENT_COUNT
DAILY_EXPECTED_STOCK
MOVEMENT_RECON_RESULT
DAILY_CLOSE
```

## 6.5 REJ / Transfer / Planning Sheets

```text
REJ_NOTICE
REJ_NOTICE_LINE
INTERNAL_TRANSFER
SPLIT_STOCK
CROSS_WAREHOUSE_TRANSFER
CUSTODY_STOCK
LOCATION_CAPACITY
LOCATION_OCCUPANCY
PLANNED_INOUT
PLAN_LINE
CHECKER_TASK
ADMIN_ALERT
```

---

## 6.6 Sheet Schemas

### CONFIG

```text
key
value
description
updated_by
updated_at
```

### MASTER_LOCATION

```text
location_id
location_code
location_name
dc
warehouse
zone
location_type
parent_location
nearby_locations
max_bag_capacity
active
review_status
created_by
created_at
updated_by
updated_at
note
```

### MASTER_CUSTOMER

```text
customer_id
customer_code
customer_name
active
created_by
created_at
updated_by
updated_at
note
```

### MASTER_ITEM

```text
item_id
item_code
item_name
item_type
default_uom
active
created_by
created_at
updated_by
updated_at
note
```

### MASTER_ITEM_ALIAS

```text
alias_id
raw_item_name
customer_code
item_type
variant
year
lot_extract_rule
default_uom
active
created_by
created_at
note
```

### MASTER_LOT

```text
lot_id
lot_code
fifo_date
production_date
year
status
created_by
created_at
note
```

### MASTER_USER

```text
user_id
display_name
employee_code
role_id
active
default_warehouse
created_at
updated_at
```

### MASTER_ROLE

```text
role_id
role_name
description
active
```

### MASTER_PERMISSION

```text
permission_id
role_id
module_name
can_view
can_create
can_edit
can_approve
can_export
can_delete
```

---

## 6.7 Snapshot Schemas

### INQUIRY_SNAPSHOT

```text
snapshot_id
business_date
snapshot_time
uploaded_at
uploaded_by
source_file_name
row_count
checksum
version
active_flag
status
remark
```

### INQUIRY_RAW

```text
raw_id
snapshot_id
raw_item
raw_customer
raw_location
raw_qty
raw_uom
raw_date
raw_time
imported_at
source_row
```

### SYSTEM_STOCK

```text
stock_id
snapshot_id
business_date
system_location
customer_code
item_code
item_type
variant
year
lot_code
fifo_date
uom
sku_key
stock_key
system_qty
source_rows
normalized_status
```

### SYSTEM_INOUT_BATCH

```text
system_batch_id
business_date
movement_date_from
movement_date_to
uploaded_at
uploaded_by
source_file_name
row_count
checksum
status
```

### SYSTEM_INOUT_NORMALIZED

```text
system_movement_id
system_batch_id
business_date
movement_date
work_time
service_mode
movement_type
location
from_location
to_location
customer_code
item_code
item_name
item_type
variant
year
lot_code
uom
qty
ton_qty
order_no
work_log_status
raw_ref
normalized_status
```

---

## 6.8 Restock / Audit Schemas

### AUDIT_BATCH

```text
audit_batch_id
audit_name
business_date
snapshot_id
status
started_by
started_at
closed_by
closed_at
remark
```

### COUNT_TXN

```text
count_id
audit_batch_id
daily_session_id
actual_location
customer_code
item_type
variant
year
lot_code
lot_status
fifo_date
uom
count_qty
count_mode
count_round
counted_by
counted_at
device_id
remark
status
```

### ANALYSIS_RESULT

```text
analysis_id
audit_batch_id
daily_session_id
stock_key
sku_key
system_location
actual_location
system_qty
count_qty
diff_qty
lot_status
fifo_status
status_code
suggested_action
confidence
remark
created_at
```

### RECOUNT_QUEUE

```text
queue_id
source_ref_type
source_ref_id
location_focus
sku_key
lot_code
system_qty
count_qty
diff_qty
priority
reason
assigned_to
status
created_at
closed_at
```

### TRANSFER_PROPOSAL

```text
proposal_id
source_type
source_ref_id
group_key
sku_key
lot_code
fifo_date
from_location
to_location
qty
uom
reason_code
status
approved_by
approved_at
exported_at
posted_status
remark
```

### TRANSFER_PROPOSAL_LINE

```text
line_id
proposal_id
from_location
to_location
qty
uom
lot_code
remark
```

### ADJUSTMENT_PROPOSAL

```text
adjustment_id
source_type
source_ref_id
sku_key
location
lot_code
adjust_type
qty
uom
reason_code
status
approved_by
approved_at
exported_at
posted_status
remark
```

---

## 6.9 Daily Inventory Schemas

### DAILY_SESSION

```text
daily_session_id
business_date
opening_snapshot_id
status
opened_by
opened_at
closed_by
closed_at
locked_at
remark
```

### OPENING_STOCK

```text
opening_id
daily_session_id
business_date
snapshot_id
location
customer_code
item_code
item_type
variant
year
lot_code
fifo_date
uom
opening_qty
stock_key
```

### FIELD_MOVEMENT_NOTICE

```text
notice_id
daily_session_id
business_date
movement_type
from_location
to_location
status
reported_by
reported_at
approved_by
approved_at
source_channel
remark
```

### FIELD_MOVEMENT_LINE

```text
line_id
notice_id
customer_code
item_code
item_type
variant
year
lot_code
fifo_date
uom
qty
owner_location
physical_location
line_status
remark
```

### MOVEMENT_COUNT

```text
movement_count_id
notice_id
line_id
count_type
location
customer_code
item_type
variant
year
lot_code
uom
count_qty
counted_by
counted_at
status
remark
```

### DAILY_EXPECTED_STOCK

```text
expected_id
daily_session_id
business_date
location
customer_code
item_type
variant
year
lot_code
uom
opening_qty
field_in_qty
field_out_qty
transfer_in_qty
transfer_out_qty
rej_in_qty
rej_out_qty
custody_in_qty
custody_out_qty
expected_qty
latest_inquiry_qty
actual_count_qty
diff_expected_vs_inquiry
diff_expected_vs_count
status
```

### MOVEMENT_RECON_RESULT

```text
recon_id
daily_session_id
field_notice_id
field_line_id
system_movement_id
match_status
diff_type
field_qty
system_qty
field_location
system_location
field_lot
system_lot
confidence
suggested_action
investigation_id
created_at
```

### DAILY_CLOSE

```text
daily_close_id
daily_session_id
business_date
status
total_movements
matched_count
issue_count
open_issue_count
closed_by
closed_at
lock_status
remark
```

---

## 6.10 REJ / Transfer / Planning Schemas

### REJ_NOTICE

```text
rej_id
daily_session_id
source_notice_id
from_location
to_rej_location
status
reported_by
reported_at
confirmed_by
confirmed_at
system_post_status
remark
```

### REJ_NOTICE_LINE

```text
rej_line_id
rej_id
customer_code
item_type
variant
year
lot_code
uom
qty
damage_reason
line_status
```

### INTERNAL_TRANSFER

```text
internal_transfer_id
daily_session_id
from_owner_location
from_physical_location
to_physical_location
reason_code
status
created_by
created_at
approved_by
approved_at
remark
```

### SPLIT_STOCK

```text
split_id
internal_transfer_id
owner_location
physical_location
customer_code
item_type
variant
year
lot_code
uom
qty
status
```

### CROSS_WAREHOUSE_TRANSFER

```text
cross_transfer_id
source_warehouse
destination_warehouse
stock_owner
physical_keeper
movement_type
status
requested_by
requested_at
received_by
received_at
erp_post_status
remark
```

### CUSTODY_STOCK

```text
custody_id
cross_transfer_id
stock_owner
physical_keeper
location
customer_code
item_type
variant
year
lot_code
uom
qty
responsibility_note
expected_return_date
status
```

### LOCATION_CAPACITY

```text
capacity_id
location
max_bag_capacity
warning_threshold
allowed_customer
allowed_item_type
allowed_lot_rule
active
updated_by
updated_at
```

### LOCATION_OCCUPANCY

```text
occupancy_id
business_date
location
current_qty
reserved_qty
planned_in_qty
planned_out_qty
available_capacity
capacity_status
```

### PLANNED_INOUT

```text
plan_id
plan_type
business_date
planned_date
status
created_by
created_at
approved_by
approved_at
remark
```

### PLAN_LINE

```text
plan_line_id
plan_id
movement_type
location
from_location
to_location
customer_code
item_type
variant
year
lot_code
uom
qty
recommended_location
capacity_status
fifo_risk
multi_lot_warning
status
```

### CHECKER_TASK

```text
task_id
source_type
source_ref_id
task_type
location
assigned_to
priority
status
created_at
started_at
completed_at
result
remark
```

### ADMIN_ALERT

```text
alert_id
alert_type
source_type
source_ref_id
severity
message
assigned_to
status
created_at
resolved_at
```

---

# 7. User Roles & Permissions

## 7.1 Permission Matrix

| Role | Main Purpose |
|---|---|
| Checker | นับ, แจ้ง movement, แจ้ง REJ |
| Supervisor | ตรวจ, อนุมัติ, ส่งสอบสวน |
| Planner | วางแผนรับเข้า/ออก, Simulation |
| Admin | Master, Import, Export, Archive |
| ERP Admin | Post ERP, Confirm Posted |
| External Warehouse | ส่งคำขอฝาก/โอน |
| Viewer | ดูข้อมูล |

## 7.2 Checker Permissions

```text
View assigned tasks
Create count
Create field movement notice
Create REJ notice
Create temp location/item
View recent own records
Undo latest own draft
Cannot approve
Cannot export
Cannot delete official records
```

## 7.3 Supervisor Permissions

```text
View dashboard
View analysis
Approve transfer
Approve recount result
Create investigation
Assign checker task
Approve temp master บางประเภท
Hold / reject issue
Cannot alter import raw data
```

## 7.4 Planner Permissions

```text
Create inbound plan
Create outbound plan
Run storage simulation
Reserve capacity
Assign checker task
Request capacity exception
```

## 7.5 Admin Permissions

```text
Import Inquiry
Import System In-Out
Manage master data
Manage capacity
Replace snapshot
Archive data
Export reports
Unlock day with reason
```

## 7.6 ERP Admin Permissions

```text
View approved ERP actions
Mark exported
Mark posted in ERP
Upload posted confirmation
Close ERP-related issues
```

## 7.7 External Warehouse Permissions

```text
Create transfer request
Create custody deposit request
View own request
Cannot edit after destination received
```

## 7.8 Viewer Permissions

```text
Read-only dashboard
Read-only reports
No create
No edit
No approve
```

---

# 8. UI/UX & State Management

## 8.1 Main Navigation

### Mobile Navigation

```text
นับจริง
งานของฉัน
แจ้ง In/Out
แจ้ง REJ
ประวัติล่าสุด
```

### Desktop Navigation

```text
Dashboard
Restock Audit
Daily Inventory
Field Movement
Before/After Count
REJ
Transfer
Planning
Capacity
Investigation
Approval
Master Data
Import
Export
Archive
```

---

## 8.2 Mobile UX Principles

```text
ปุ่มใหญ่
ฟอร์มสั้น
ใช้มือเดียวได้
Lock Field ได้
Auto UOM
Last Used
Quick Add
Calculator
Recent Count
Undo ล่าสุด
รองรับสัญญาณไม่ดี
```

### Mobile Count Screen

```text
Location
Customer
Item
Year
Lot Optional
Qty
Remark
Save
```

### Mobile Movement Screen

```text
Movement Type
From / To Location
Item
Lot
Qty
Before Count
After Count
Save
```

### Mobile REJ Screen

```text
From Location
Item
Lot
Qty
Damage Reason
Save
Summary for Line Screenshot
```

---

## 8.3 Desktop UX Principles

```text
ใช้ดูภาพรวม
ใช้ตรวจ issue
ใช้ approve
ใช้ import/export
ใช้ simulation
ใช้ master data
```

### Dashboard Cards

```text
Matched
System Missing
Field Missing
Qty Diff
Location Diff
Lot Diff
REJ Pending
Capacity Warning
Pending Approval
Open Investigation
```

### Analysis Tabs

```text
ตรงแล้ว
Location ผิด
Qty ผิด
ระบบมีแต่ไม่เจอ
หน้างานเจอแต่ระบบไม่มี
Lot/FIFO Issue
Movement ไม่ตรง
REJ Issue
Capacity Issue
Investigation
```

---

## 8.4 State Management

### Global State

```text
current_user
current_role
active_daily_session
active_audit_batch
active_snapshot
selected_warehouse
selected_location
online_status
last_sync_at
```

### Entity State

```text
DRAFT
OPEN
REPORTED
ASSIGNED
IN_PROGRESS
WAIT_SYSTEM_POST
WAIT_RECOUNT
WAIT_INVESTIGATION
PENDING_APPROVAL
APPROVED
REJECTED
MATCHED
PROBLEM
EXPORTED
POSTED_IN_ERP
CLOSED
LOCKED
ARCHIVED
```

### Client-side State

```text
form_data
locked_fields
last_used_values
pending_local_records
validation_errors
selected_tab
selected_filters
```

### Offline / Weak Signal State

```text
LOCAL_DRAFT
PENDING_SYNC
SYNCED
SYNC_FAILED
CONFLICT
```

---

## 8.5 UX Rules

```text
ไม่โชว์ code ดิบให้หน้างาน
แสดงภาษาไทย
แสดง status ด้วยสี
ใช้ grouped card แทน table บนมือถือ
ใช้ table/filter บน desktop
ทุก action สำคัญต้องมี confirm
หลัง save ต้อง show summary
REJ ต้องมีหน้าสรุปให้แคปส่ง Line
```

---

# 9. Error Handling & Edge Cases

## 9.1 Upload Inquiry ซ้ำ

### Case

```text
มี active inquiry ของวันเดียวกันแล้ว
```

### Handling

```text
ห้าม append
ให้เลือก Replace / Archive Old / Cancel
สร้าง snapshot version ใหม่
เก็บ log ทุกครั้ง
```

---

## 9.2 System In-Out ยังไม่พร้อม

### Case

```text
ข้อมูลวันนี้ยังไม่ตัด เพราะ ERP ตัด 23:59:59
```

### Handling

```text
แสดง WAIT_SYSTEM_POST
ยังไม่สรุปผิดทันที
ให้ตรวจวันถัดไป
```

---

## 9.3 Field Movement มี แต่ System In-Out ไม่มี

### Result

```text
SYSTEM_MISSING
```

### Handling

```text
ส่ง Investigation
ถามห้องชั่ง / ERP Admin
ตรวจเอกสาร
รอ post
```

---

## 9.4 System In-Out มี แต่ Field Movement ไม่มี

### Result

```text
FIELD_MISSING
```

### Handling

```text
แจ้ง Supervisor
ถามหน้างาน
เช็คว่ามี movement นอกระบบแจ้งหรือไม่
```

---

## 9.5 Qty ไม่ตรง

### Result

```text
QTY_DIFF
```

### Handling

```text
ดู Before/After Count
ถ้า Count ยืนยัน Field ถูก → ERP ตัดผิด
ถ้า Count ไม่ยืนยัน → หน้างานแจ้งผิดหรือนับผิด
```

---

## 9.6 Location ไม่ตรง

### Result

```text
LOCATION_DIFF
```

### Handling

```text
ตรวจ Physical Count ของ Location ที่เกี่ยวข้อง
ถ้าของจริงตรง Field → ERP location ผิด
ถ้าของจริงตรง System → Field notice ผิด
```

---

## 9.7 Lot ไม่ตรง

### Result

```text
LOT_DIFF
```

### Handling

```text
ห้าม Auto Transfer
ส่ง Lot Verify หรือ Investigation
ถ้าเป็น Bulk Count ให้ mark LOT_UNCHECKED
```

---

## 9.8 REJ แจ้งแล้วแต่ระบบไม่ตัด

### Result

```text
REJ_NOT_POSTED
```

### Handling

```text
ส่ง ERP Admin ตรวจ
ค้าง WAIT_SYSTEM_POST
ถ้าเกิน SLA แจ้ง Supervisor
```

---

## 9.9 REJ ระบบตัดแต่ไม่มีคนแจ้ง

### Result

```text
REJ_UNREPORTED
```

### Handling

```text
ตรวจว่ามีใครแจ้งใน Line แต่ไม่บันทึกระบบหรือไม่
เปิด Investigation
```

---

## 9.10 Capacity เกิน

### Result

```text
CAPACITY_EXCEEDED
```

### Handling

```text
ห้ามรับเข้าแบบปกติ
ให้ Supervisor/Admin อนุมัติ exception
เสนอ alternative location
```

---

## 9.11 Multi-Lot Location

### Case

```text
รับเข้า Location ที่มีหลาย Lot อยู่แล้ว
```

### Handling

```text
เตือนผู้ใช้
ถ้ายืนยัน ให้บันทึก
Mark MULTI_LOT_WARNING
แจ้ง Admin ทันที
```

---

## 9.12 Transfer เศษไปฝาก Location อื่น

### Case

```text
ของ Location 9 ไปวางที่ Location 6
```

### Handling

```text
ใช้ owner_location และ physical_location
แสดงทั้งสองมุมมอง
ห้ามปนเป็นของ Location 6 เฉย ๆ
```

---

## 9.13 Cross-Warehouse ฝากของ

### Case

```text
คลังอื่นเอาของมาฝาก
```

### Handling

```text
บันทึก Custody Stock
แยก stock_owner กับ physical_keeper
Checker ต้องกดรับ
ถ้ามีระบบทั้งคู่ ปลายทางต้อง confirm
```

---

## 9.14 นับย้อนหลังหลังปิดวัน

### Handling

```text
ถ้า DAILY_SESSION = LOCKED
ห้ามนับเพิ่ม
แก้ได้เฉพาะผ่าน Investigation
```

---

## 9.15 แก้ข้อมูลย้อนหลัง

### Handling

```text
ห้าม edit ตรง
สร้าง edit request
เก็บ old value / new value
ต้องมี reason
ต้องมี approval
```

---

## 9.16 ข้อมูลเยอะแล้วระบบช้า

### Handling

```text
โหลดเฉพาะ Active 45 วัน
Archive ข้อมูลเก่า
สร้าง summary table
Dashboard ใช้ aggregate
```

---

## 9.17 Google Sheets Quota / Write Failure

### Handling

```text
บันทึก local draft
retry
แสดง SYNC_FAILED
ไม่ล้างฟอร์มจนกว่าจะ sync สำเร็จ
```

---

## 9.18 Duplicate Save

### Handling

```text
ใช้ idempotency_key
ปิดปุ่ม save หลังคลิก
ตรวจ duplicate ด้วย hash
```

---

## 9.19 สัญญาณมือถือหลุด

### Handling

```text
เก็บ localStorage
status = PENDING_SYNC
sync เมื่อ online
แสดงรายการค้าง sync
```

---

# 10. Key Business Rules

```text
1. Inquiry เป็น Snapshot ไม่ใช่ Transaction
2. Field Movement เป็นสิ่งที่หน้างานแจ้ง
3. System In-Out เป็นสิ่งที่ ERP ตัดจริง
4. Physical Count เป็นความจริงจากหน้างาน
5. Expected Stock เป็นค่าคำนวณ
6. ERP ไม่ใช่ realtime truth
7. Adjust ต้องเป็นทางเลือกสุดท้าย
8. REJ ต้อง track ตั้งแต่แจ้งจน ERP post
9. Lot ตรวจเท่าที่ทำได้จริง
10. Capacity ต้องเช็คก่อนรับเข้า
11. Cross-Warehouse ต้องมี custody concept
12. ทุก action ต้องมี audit log
13. Daily Lock แล้วห้ามแก้ตรง
14. Archive ได้เฉพาะข้อมูลที่ปิดแล้ว
15. Dashboard ห้ามโหลดข้อมูลเก่าโดยตรง
```

---

# 11. Recommended Development Phases

## Phase 1: Core Foundation

```text
Master Data
User / Role
Location Structure
Snapshot Control
Audit Log
Basic Dashboard
```

## Phase 2: Restock / Stock Audit

```text
Import Inquiry
Mobile Count
Analysis Result
Recount Queue
Transfer Proposal
Adjustment Proposal
Investigation
```

## Phase 3: Daily Inventory Control

```text
Opening Stock
Field In-Out
Before / After Count
System In-Out Import
Expected Stock
Movement Reconciliation
Daily Close
```

## Phase 4: REJ / Transfer / Capacity

```text
REJ Damage Flow
Internal Split Transfer
Capacity by Location
Multi-Lot Warning
Admin Alert
```

## Phase 5: Planning

```text
Storage Simulation
Outbound Planning
Checker Task
Capacity Reservation
Plan vs Actual
```

## Phase 6: Cross-Warehouse / Custody

```text
External Deposit
Cross-Warehouse Transfer
Destination Receive
Custody Stock
ERP Transfer Pending
```

## Phase 7: Performance / Archive

```text
45-Day Active Data
Archive Sheets
Summary Tables
Dashboard Optimization
Offline Sync
```

---

# 12. Final System Definition

ระบบสุดท้ายคือ

```text
Warehouse Reconciliation ERP Platform
```

แบ่งเป็น

```text
Restock / Stock Audit ERP
Daily Inventory Control ERP
Storage Planning & Movement Control
```

แก่นของระบบ

```text
Physical Count
+ Field Movement
+ ERP Inquiry
+ System In-Out
+ Capacity
+ Lot/FIFO
+ REJ
+ Custody Stock
=
Reconciliation Decision
```

เป้าหมายสุดท้าย

```text
นับง่าย
แจ้งง่าย
วางแผนง่าย
รู้ว่าอะไรอยู่ที่ไหน
รู้ว่าอะไรเป็นของใคร
รู้ว่า ERP ตัดครบไหม
รู้ว่าต้องแก้อะไร
รู้ว่าใครต้องรับผิดชอบ
ปิดวันได้มั่นใจ
ปิดรอบได้ตรวจสอบได้
```
