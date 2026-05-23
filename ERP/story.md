# Story — Warehouse Reconciliation ERP Platform

## 0. จุดเริ่มต้นของความต้องการ

ระบบนี้เกิดจากปัญหาจริงของงานคลังน้ำตาลที่ไม่ได้เป็นแค่ “นับของให้ครบ” แต่เป็นปัญหาที่ซับซ้อนกว่านั้น คือข้อมูลระหว่าง **ของจริงในคลัง**, **สิ่งที่หน้างานแจ้ง**, และ **ข้อมูลใน ERP** ไม่ตรงกันตลอดเวลา

คลังมีข้อจำกัดสำคัญหลายอย่าง:

- สินค้าเป็นกองน้ำตาลขนาดใหญ่ หนัก และเข้าถึงยาก
- บางกองซ้อนกัน ทำให้ตรวจ Lot ทุกกองไม่ได้จริง
- ERP ไม่ได้เป็นข้อมูล real-time
- ข้อมูล Stock In-Out จากระบบมักดูย้อนหลังได้ และข้อมูลของวันนี้อาจยังไม่ครบเพราะระบบตัดรอบตอนกลางคืน
- Inquiry เป็นยอดคงเหลือ ณ เวลาที่ดึง แต่บางชุดไม่มี timestamp ที่เชื่อถือได้ หรือไม่มีเวลาชัดเจน
- Location ในระบบกับที่ของจริงอยู่ไม่ตรงกันบ่อย
- บางครั้งของออกไปแล้ว แต่ ERP ยังไม่ตัด
- บางครั้งย้ายของแล้ว แต่ไม่มีคนทำ transfer ในระบบ
- บางครั้งสินค้าชำรุดต้องย้ายไป REJ แต่ต้องตามต่อว่า ERP ตัดจริงไหม
- บางครั้งคลังอื่นฝากของไว้ หรือโอนมาฝาก ทำให้ยอดบัญชีดูเหมือนเกิน ทั้งที่มีที่มาที่ไป
- การแก้ไขปัจจุบันมักต้องไล่ Excel, ถามคน, ดู Line, ดูใบงาน และเทียบเองทีละรายการ

ดังนั้นระบบที่ต้องการจึงไม่ใช่แค่ **Stock Count App** แต่เป็นระบบที่ทำหน้าที่เป็นตัวกลางระหว่าง “โลกจริงของคลัง” กับ “โลกข้อมูลของ ERP”

ชื่อแนวคิดระบบคือ:

```text
Warehouse Reconciliation ERP Platform
```

ระบบนี้ต้องทำให้รู้ว่า:

```text
ของจริงอยู่ไหน
ERP บอกว่าอยู่ไหน
หน้างานแจ้งอะไร
ERP ตัดอะไรแล้ว
อะไรตรง
อะไรผิด
ใครต้องไปตรวจ
อะไรต้องย้ายในระบบ
อะไรต้องปรับยอด
อะไรต้องสอบสวน
อะไรปิดวันได้
อะไรยังห้ามปิด
```

---

## 1. เป้าหมายใหญ่ของระบบ

เป้าหมายหลักคือทำให้คลังสามารถบริหาร Inventory ได้แม่นยำขึ้น โดยไม่ต้องพึ่ง Excel และความจำของคนมากเกินไป

ระบบต้องช่วยให้ตอบคำถามเหล่านี้ได้:

1. ของจริงอยู่ Location ไหน
2. ERP คิดว่าของอยู่ Location ไหน
3. ของที่ระบบมี ยังอยู่จริงไหม
4. ของที่หน้างานเจอ มีอยู่ใน ERP ไหม
5. ยอด Qty ตรงไหม
6. Lot หรือ FIFO มีความเสี่ยงไหม
7. มี movement จริงไหม
8. หน้างานแจ้ง movement ครบไหม
9. ERP ตัด movement ครบไหม
10. REJ ถูกย้ายและ post เข้าระบบจริงไหม
11. ของที่ฝากไว้จากคลังอื่นมีหลักฐานไหม
12. Location ไหนเต็มหรือใกล้เต็ม
13. แผนรับเข้า/จ่ายออกทำให้ capacity เกินไหม
14. ควรให้ Checker ไปตรวจที่ไหน
15. ต้องสร้าง Transfer, Adjust, REJ, Investigation หรือไม่
16. ปิดวันหรือปิดรอบได้หรือยัง

หัวใจคือ:

```text
Physical Reality + Field Movement + ERP Data = Reconciliation Decision
```

---

## 2. หลักคิดกลางของระบบ

ระบบนี้ต้องแยก “ความจริง” ออกเป็นหลายชั้น ไม่เอาทุกอย่างมาปนกัน

### 2.1 Physical Count

คือความจริงจากของที่มองเห็นและนับได้จริง

```text
Physical Count = ของจริงที่หน้างานเห็น
```

ใช้ตอบว่า:

- ของอยู่ที่ไหนจริง
- จำนวนจริงเท่าไร
- ตรวจ Lot ได้หรือไม่ได้
- เป็นการนับแบบ bulk หรือ verify รายละเอียด

### 2.2 Field Movement

คือสิ่งที่หน้างานแจ้งว่าเกิดขึ้นจริง

```text
Field Movement = Operational Truth
```

เช่น:

- รับเข้า
- จ่ายออก
- ย้าย Location
- REJ
- ฝากของ
- แบ่งกอง
- รับของจากคลังอื่น

### 2.3 System In-Out

คือ movement ที่ ERP ตัดจริง

```text
System In-Out = ERP Movement Truth
```

แต่ไม่ควรถือว่าเป็น real-time truth เพราะ ERP อาจตัดช้า โดยเฉพาะข้อมูลวันนี้ที่อาจยังไม่ครบจนกว่าจะผ่านรอบตัดช่วงกลางคืน

### 2.4 Inquiry

คือยอดคงเหลือ ณ เวลาที่ดึงจาก ERP

```text
Inquiry = Snapshot Truth
```

สำคัญมาก: Inquiry ไม่ใช่ transaction

ห้ามเอา Inquiry มาบวกซ้ำกับ movement แบบมั่ว ๆ เพราะจะทำให้ยอดเบิ้ล

### 2.5 Expected Stock

คือยอดที่ระบบคำนวณเองจาก Opening Stock + Movement

```text
Expected Stock = Calculation Truth
```

สูตรหลัก:

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

---

## 3. ระบบหลักแบ่งออกเป็น 2 ระบบใหญ่

ระบบทั้งหมดต้องแยกเป็น 2 ระบบใหญ่ แต่ใช้ Core Platform ร่วมกัน

```text
Warehouse Reconciliation ERP Platform
│
├── Core Platform
│
├── Module A: Restock / Stock Audit ERP
│
└── Module B: Daily Inventory Control ERP
```

---

## 4. Module A — Restock / Stock Audit ERP

### 4.1 ความเป็นมา

Module นี้เกิดจากปัญหาตอนตรวจสต๊อกรอบใหญ่หรือช่วงเคลียร์ยอด ที่ต้องเทียบว่า ERP กับของจริงตรงกันไหม

ปัญหาที่พบ:

- ERP บอกว่าของอยู่ Location หนึ่ง แต่ของจริงอยู่อีก Location
- ERP มีของ แต่หน้างานหาไม่เจอ
- หน้างานเจอของ แต่ ERP ไม่มีที่ Location นั้น
- จำนวนไม่ตรง
- ของไม่ได้หาย แต่อยู่ผิด Location
- บางรายการอาจออกไปแล้ว แต่ ERP ยังไม่ตัด
- บางรายการอาจย้ายจริงแล้ว แต่ไม่มี transfer ในระบบ
- Lot/FIFO มีผล แต่ตรวจจริงได้ไม่ครบทุกกอง

### 4.2 เป้าหมาย

Module นี้มีไว้เพื่อ:

```text
ตรวจสต๊อกรอบใหญ่
เคลียร์ยอด
แก้ Location ใน ERP
เสนอ Transfer
เสนอ Adjust
ส่ง Recount
ส่ง Investigation
ปิดรอบ Audit
```

### 4.3 Flow หลัก

```text
Import Inquiry
↓
Normalize System Stock
↓
Mobile Physical Count
↓
Analysis / Reconciliation
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

### 4.4 สิ่งที่ระบบต้องทำได้

ระบบต้องแยกผลลัพธ์เป็นกลุ่ม:

```text
MATCHED
LOCATION_MISMATCH
QTY_DIFF
SYSTEM_ONLY
PHYSICAL_ONLY
LOT_MISMATCH
FIFO_WARNING
NEED_RECOUNT
NEED_INVESTIGATION
TRANSFER_PROPOSED
ADJUST_PROPOSED
```

ตัวอย่าง:

```text
ERP:
W8-A01 = 3,360
W8-B01 = 60
W8-C02 = 1,290

หน้างานนับ:
W8-B04 = 4,710

ระบบต้องสรุปว่า:
ของไม่ได้หาย แต่ Location ใน ERP ผิด
ควรย้ายในระบบจาก A01/B01/C02 ไป B04
```

### 4.5 หลักสำคัญของ Module A

ระบบไม่ได้มีไว้เพื่อบอกแค่ว่า “ยอดต่าง” แต่ต้องบอกว่า:

```text
ต่างเพราะอะไร
ต้องแก้อะไร
ใครต้องอนุมัติ
ERP ต้อง post อะไร
```

---

## 5. Module B — Daily Inventory Control ERP

### 5.1 ความเป็นมา

หลังเคลียร์ยอดแล้ว ผู้ใช้ต้องการทำระบบนับและควบคุม Inventory รายวัน เพื่อไม่ให้ปัญหากลับมาสะสมอีก

ข้อมูลที่ต้องใช้มี 2 ชุดจากระบบ:

1. Inquiry
2. Stock In-Out

และมีข้อมูลที่หน้างานต้องกรอกเพิ่ม:

1. Field In-Out Notice
2. Before / After Count
3. REJ Notice
4. Physical Count ของ Location ที่มี movement

### 5.2 Inquiry ในระบบรายวัน

Inquiry คือยอดคงเหลือ ณ เวลาที่ดึงจากระบบ

ปัญหา:

- เป็นข้อมูล snapshot
- บางครั้งไม่มีวันที่/เวลาในตัว
- ถ้าอัปโหลดซ้ำในวันเดียวกันอาจเบิ้ล
- ไม่ควร append ทับกัน

แนวทาง:

```text
ทุกครั้งที่ upload inquiry ต้องสร้าง snapshot_id
ต้องกำหนด business_date
ต้องกำหนด uploaded_at
ต้องมี active_flag
1 วันควรมี active inquiry ได้ 1 ชุด
ถ้า upload ซ้ำต้อง Replace / Archive / Cancel
```

### 5.3 System In-Out

System In-Out คือข้อมูล movement ที่ ERP ตัดจริง

ข้อจำกัด:

- ไม่ real-time
- ข้อมูลวันนี้อาจยังไม่พร้อม
- มักใช้ข้อมูลย้อนหลังของวันก่อน
- ระบบอาจตัดตอนกลางคืนประมาณ 23:59:59

ใช้เพื่อตรวจว่า:

```text
Field Movement ที่หน้างานแจ้ง
ตรงกับ ERP movement ที่ระบบตัดจริงไหม
```

### 5.4 Field In-Out Notice

หน้างานต้องแจ้ง movement จริง เช่น:

```text
IN
OUT
TRANSFER
REJ
RETURN
CUSTODY_IN
CUSTODY_OUT
```

ต้องแจ้ง:

```text
เข้า/ออกอะไร
จำนวนเท่าไร
Location ไหน
Customer ไหน
ชนิดอะไร
ปีอะไร
Lot อะไร
ใครแจ้ง
เวลาไหน
```

### 5.5 Before / After Count

ระบบต้องรองรับการนับก่อนและหลัง movement

ใช้พิสูจน์ว่าของจริงเปลี่ยนตามที่แจ้งจริงไหม

#### กรณี IN

```text
Before Count + IN Qty = After Count
```

#### กรณี OUT

```text
Before Count - OUT Qty = After Count
```

#### กรณี TRANSFER

```text
From Before - Qty = From After
To Before + Qty = To After
```

#### กรณี REJ

```text
From Before - Qty = From After
REJ Location + Qty
```

ถ้าไม่ตรง ให้ขึ้น:

```text
PHYSICAL_MOVEMENT_DIFF
```

### 5.6 Daily Flow

```text
Upload Inquiry ตั้งต้น
↓
Set Opening Stock
↓
หน้างานแจ้ง In-Out / REJ / Transfer
↓
นับ Before / After เฉพาะรายการสำคัญ
↓
ระบบคำนวณ Expected Stock
↓
นับเฉพาะ Location ที่มี Movement
↓
Upload System In-Out วันถัดไป
↓
เทียบ Field Movement vs System In-Out
↓
สรุปว่า Matched / Missing / Diff / Investigation
↓
Close Day
↓
Lock Day
```

### 5.7 เป้าหมาย Module B

ตอบให้ได้ว่า:

```text
วันนี้เข้าอะไร
วันนี้ออกอะไร
หน้างานแจ้งครบไหม
ERP ตัดครบไหม
ตัดเกินไหม
ตัดผิด Location ไหม
ตัดผิด Qty ไหม
ตัดผิด Lot ไหม
REJ เข้า ERP จริงไหม
Location ที่เคลื่อนไหวยอดตรงไหม
```

---

## 6. Field Movement vs System In-Out — Logic ที่ต้องมี

### 6.1 หน้างานแจ้ง แต่ ERP ไม่ตัด

```text
Field Movement มี
System In-Out ไม่มี
```

ผลลัพธ์:

```text
SYSTEM_MISSING
```

แปลว่า:

```text
หน้างานแจ้งว่าเกิด movement จริง แต่ ERP ยังไม่มีรายการ post
```

ต้องตรวจ:

- ห้องชั่งตัดหรือยัง
- ERP Admin post หรือยัง
- เอกสารค้างไหม
- movement อยู่ผิดวันไหม

### 6.2 ERP ตัด แต่หน้างานไม่แจ้ง

```text
System In-Out มี
Field Movement ไม่มี
```

ผลลัพธ์:

```text
FIELD_MISSING
```

แปลว่า:

```text
ERP มี movement แต่หน้างานไม่ได้แจ้งในระบบ
```

ต้องตรวจ:

- หน้างานลืมแจ้ง
- แจ้งใน Line แต่ไม่ได้บันทึกระบบ
- ERP ตัด movement ผิด
- เป็น movement ที่เกิดจากแหล่งอื่น

### 6.3 จำนวนไม่ตรง

```text
Field Qty != System Qty
```

ผลลัพธ์:

```text
QTY_DIFF
```

ต้องตรวจ Before / After Count

ถ้า Before / After ยืนยัน Field ถูก:

```text
ERP ตัดจำนวนผิด
```

ถ้า Before / After ไม่สัมพันธ์:

```text
หน้างานแจ้งผิดหรือนับผิด
```

### 6.4 Location ไม่ตรง

```text
Field Location != System Location
```

ผลลัพธ์:

```text
LOCATION_DIFF
```

ต้องตรวจ Physical Count ของ Location ที่เกี่ยวข้อง

ถ้าของจริงตรงกับ Field:

```text
ERP ตัด Location ผิด
```

ถ้าของจริงตรงกับ System:

```text
Field Notice ผิด
```

### 6.5 Lot ไม่ตรง

```text
Field Lot != System Lot
```

ผลลัพธ์:

```text
LOT_DIFF
```

ห้าม Auto Transfer ทันที

ต้องส่ง:

```text
LOT_VERIFY
หรือ
INVESTIGATION
```

### 6.6 ระบบตัดซ้ำ

ถ้ารายการเดียวกันถูก post มากกว่า 1 ครั้ง:

```text
DUPLICATE_POSTED
```

ต้องตรวจ:

- ORDER_NO
- Movement date
- Qty
- Location
- Item
- Lot
- Time window

---

## 7. REJ Damage Flow

### 7.1 ความเป็นมา

REJ คือการแจ้งสินค้าชำรุดหรือเสียหาย

ผู้ใช้ต้องการให้หน้างานแจ้งผ่านระบบ ไม่ใช่แค่แจ้งใน Line เพราะต้องเอาไปเทียบกับ ERP วันถัดไป

### 7.2 Flow

```text
Checker เลือก From Location
↓
เลือกสินค้า / ลูกค้า / ชนิด / ปี / Lot
↓
ใส่ Qty
↓
เลือก Damage Reason
↓
กดบันทึก
↓
ระบบสร้าง REJ Notice
↓
ระบบแสดงหน้าสรุปให้แคปส่ง Line
↓
ระบบถือว่า From Location ลด และ W8-REJ เพิ่ม
↓
วันถัดไปเทียบกับ System In-Out
```

### 7.3 REJ ต้องไปที่ไหน

ปลายทางต้องเป็น Location ที่มีคำว่า:

```text
REJ
```

เช่น:

```text
W8-REJ
```

### 7.4 ผลลัพธ์ที่ต้องแยก

```text
REJ_MATCHED
REJ_NOT_POSTED
REJ_UNREPORTED
REJ_QTY_DIFF
REJ_LOCATION_DIFF
REJ_ITEM_DIFF
REJ_LOT_DIFF
```

### 7.5 ถ้า REJ ไม่ตรง

ต้องบอกได้ว่าอาจผิดตรงไหน:

```text
ใส่จำนวนผิด
Location ผิด
ชนิดผิด
Lot ผิด
ERP ยังไม่ post
หน้างานลืมบันทึก
```

---

## 8. Lot และ FIFO

### 8.1 ความเป็นมา

สินค้าในคลังมี Lot ซึ่งอยู่ท้าย Item เช่น:

```text
690312
```

ผู้ใช้ตีความว่า:

```text
ปี 69
เดือน 03
วันที่ 12
```

ดังนั้นระบบต้องดึง Lot จาก Item ได้

### 8.2 ปัญหา

การนับ Lot ทุกกองทำไม่ได้จริง เพราะ:

- กองใหญ่
- หนัก
- ซ้อนกัน
- เข้าถึงยาก
- ไม่สามารถเปิดดูทุก Lot ได้ 90–100%

### 8.3 แนวทางที่ตกลง

ไม่บังคับ Lot ทุกครั้ง

ใช้แนวคิด:

```text
Progressive Accuracy
```

หรือ:

```text
นับละเอียดเท่าที่หน้างานทำได้จริง
```

### 8.4 Lot Mode

```text
BULK_COUNT
LOT_VERIFY
PARTIAL_VERIFY
ASSUMED_FIFO
```

### 8.5 Lot Status

```text
LOT_VERIFIED
LOT_PARTIAL
LOT_UNCHECKED
LOT_MISMATCH
```

### 8.6 FIFO Status

```text
FIFO_OK
FIFO_WARNING
FIFO_VIOLATION
NEED_LOT_VERIFY
```

### 8.7 Multi-Lot Warning

ถ้า Location มีหลาย Lot อยู่แล้ว แล้วมีการแจ้ง IN เข้า Location นั้น ระบบต้องเตือน

ตัวอย่าง:

```text
Location 9 มี 3 Lot อยู่แล้ว
มีการแจ้ง IN เข้า Location 9 อีก
```

ระบบต้องถาม:

```text
Location นี้มีหลาย Lot อยู่แล้ว ต้องการรับเข้า Location นี้หรือไม่?
```

ถ้ากดตกลง:

```text
บันทึกได้
Mark MULTI_LOT_WARNING
แจ้ง Admin ทันที
```

---

## 9. Internal Transfer / Split Stock

### 9.1 ความเป็นมา

มีกรณีที่ของจาก Location หนึ่งถูกย้ายไปวางอีก Location เป็นเศษหรือฝากไว้ชั่วคราว

ตัวอย่าง:

```text
ของ Location 9 บางส่วนไปวางที่ Location 6
```

ปัญหาคือ ถ้าบันทึกแค่ว่าอยู่ Location 6 จะทำให้เข้าใจผิดว่าเป็นของ Location 6 เอง

### 9.2 แนวคิดที่ต้องใช้

ต้องแยก:

```text
owner_location
physical_location
```

ตัวอย่าง:

```text
owner_location = W8-09
physical_location = W8-06
qty = 120
reason = เศษฝาก
```

### 9.3 การแสดงผล

#### มุมมอง Location 9

```text
อยู่จริงที่ W8-09 = 4,000
ฝากไว้ W8-06 = 120
รวมของ Location 9 = 4,120
```

#### มุมมอง Location 6

```text
ของ W8-06 เอง = 3,500
รับฝากจาก W8-09 = 120
รวมที่วางจริง W8-06 = 3,620
```

### 9.4 สถานะ

```text
INTERNAL_TRANSFER
SPLIT_STACK
TEMP_HOLD
RETURN_PENDING
CLOSED
```

---

## 10. Cross-Warehouse Transfer / Custody Stock

### 10.1 ความเป็นมา

มีกรณีคลังอื่นเอาของมาฝาก หรือโอนมาเกินแล้วไม่อยากขนกลับ จึงฝากไว้ที่คลังเรา

ปัญหาคือ ตอนบัญชีมานับ อาจเห็นยอดเกินและไม่รู้ว่าเป็นของใคร

### 10.2 แนวคิด

ต้องมีระบบ:

```text
Custody Stock
```

คือของที่:

```text
อยู่ที่เรา
แต่เจ้าของหรือความรับผิดชอบมาจากคลังอื่น
```

### 10.3 ข้อมูลที่ต้องเก็บ

```text
stock_owner
physical_keeper
erp_owner
responsibility_note
expected_return_date
source_warehouse
destination_warehouse
customer
item
year
lot
qty
location
```

### 10.4 กรณีที่ 1: คลังอื่นไม่ได้ใช้ระบบนี้

```text
Admin บันทึก Manual Deposit
Checker ไปตรวจรับ
ระบบแยกเป็น Custody Stock
```

### 10.5 กรณีที่ 2: คลังอื่นใช้ระบบเหมือนกัน

```text
ต้นทางสร้าง Transfer Request
ปลายทางกดรับ
Checker ตรวจรับ
ERP Admin post
```

### 10.6 สถานะ

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

## 11. Storage Capacity

### 11.1 ความเป็นมา

ผู้ใช้ต้องการ set และ edit capacity by location ว่าแต่ละ Location เก็บได้เต็มสุดกี่ถุง

ถ้ามีการรับเข้าเกิน capacity ต้องแจ้งทันที

### 11.2 สิ่งที่ต้องทำได้

```text
Set capacity by location
Edit capacity
ตั้ง max bags
ตั้ง warning threshold
ตั้ง allowed customer
ตั้ง allowed item type
ตั้ง allowed lot rule
ดู current qty
ดู reserved qty
ดู available capacity
```

### 11.3 Warning

```text
NEAR_CAPACITY
CAPACITY_EXCEEDED
REQUIRES_APPROVAL
```

### 11.4 ตัวอย่าง

```text
Location W8-A01
Max Capacity = 5,000 ถุง
Current = 4,600
Planned IN = 700

Projected = 5,300
Status = CAPACITY_EXCEEDED
```

ระบบต้องแจ้ง Planner/Admin/Supervisor ทันที

---

## 12. Storage Planning & Simulation

### 12.1 ความเป็นมา

ก่อนของเข้า ผู้ใช้ต้องการวางแผนว่าจะเก็บที่ไหน โดยดู:

- ลูกค้า
- ชนิดสินค้า
- ปี
- Lot
- Capacity
- Location ที่ว่าง
- Location ที่มีของเดิม
- แผน In-Out วันนี้
- ของที่จะเข้าพรุ่งนี้
- ความเสี่ยง Multi-Lot/FIFO

### 12.2 เป้าหมาย

ระบบต้องช่วยตอบว่า:

```text
ถ้าพรุ่งนี้ของนี้เข้า ควรลง Location ไหน
Location ไหนว่าง
Location ไหนเต็ม
Location ไหนมี Lot ปน
Location ไหนควรหลีกเลี่ยง
ถ้าเอาแผน OUT วันนี้ไปลบแล้วจะเหลือพื้นที่เท่าไร
ถ้าต้องแยกเป็น 2 กอง ทำได้ไหม
```

### 12.3 สูตร Projected Capacity

```text
Projected Capacity =
Current Inquiry
- Planned OUT วันนี้
+ Planned IN วันนี้/พรุ่งนี้
+ Pending Transfer In
- Pending Transfer Out
```

### 12.4 Output ที่ต้องได้

```text
Recommended Location
Alternative Location
Capacity Warning
Multi-Lot Warning
FIFO Risk
Admin Approval Required
Checker Task Required
```

### 12.5 ตัวอย่างการใช้งาน

Planner วางแผนว่า:

```text
พรุ่งนี้จะรับ MKMK ไฮโพลดิบ ปี 69 Lot 690312 จำนวน 4,000 ถุง
```

ระบบต้องคำนวณว่า:

```text
Location ไหนว่าง
Location ไหนมี Lot เดิม
Location ไหนเกิน capacity
Location ไหนเหมาะสุด
```

แล้ว Planner สามารถโทรคุยกับหน้างานว่า:

```text
ระบบแนะนำลง W8-09 ได้ 3,000 ถุง
เหลืออีก 1,000 ถุง อาจต้องลง W8-06
ต้องยอมให้เป็น 2 กองไหม
```

ถ้าตกลง ก็สร้างแผนและมอบหมาย Checker สำหรับวันถัดไป

---

## 13. Outbound Planning

### 13.1 ความเป็นมา

ก่อนจะเอาของออก ผู้ใช้ต้องวางแผนว่า:

```text
จะเอาออกจากที่ไหน
จำนวนเท่าไร
สินค้าอะไร
ปีอะไร
Lot อะไร
```

จากนั้นแจ้ง Checker ไปตรวจว่ามีของจริงไหม

### 13.2 Flow

```text
Planner สร้างแผน OUT
↓
เลือก Location / SKU / Lot / Qty
↓
Assign Checker
↓
Checker ไปตรวจจริง
↓
ถ้าตรง → READY_TO_OUT
ถ้าไม่ตรง → ISSUE_FOUND
```

### 13.3 สถานะ

```text
PLANNED
ASSIGNED
CHECKED
READY_TO_OUT
ISSUE_FOUND
CANCELLED
```

---

## 14. Checker Task

### 14.1 ความเป็นมา

เพื่อให้หน้างานไม่ต้องคิดเองว่าต้องไปตรวจอะไร ระบบต้องส่ง task ให้

### 14.2 Task Type

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

### 14.3 สถานะ

```text
OPEN
ASSIGNED
IN_PROGRESS
DONE
HOLD
CANCELLED
```

### 14.4 หลัก UX

มือถือของ Checker ควรเห็นแค่:

```text
งานของฉัน
เริ่มตรวจ
บันทึกผล
ส่งกลับหัวหน้า
```

---

## 15. Master Data

### 15.1 ความเป็นมา

ผู้ใช้ต้องการให้จัดการตัวเลือกได้เอง เช่น:

- Location
- Customer
- Item
- Type
- Year
- Lot
- UOM

เพราะถ้ามีสินค้าใหม่หรือ Location ใกล้เคียงที่ไม่มีในระบบ หน้างานไม่ควรติด workflow

### 15.2 แนวทาง

ให้เพิ่มจากหน้าเว็บได้ แต่ต้องมีสถานะ:

```text
TEMP
WAITING_REVIEW
```

Admin ต้องตรวจและอนุมัติก่อนเป็น master จริง

### 15.3 ห้ามลบจริง

ถ้าข้อมูลเคยถูกใช้แล้ว ห้ามลบจริง

ให้ใช้:

```text
active = FALSE
```

---

## 16. User Roles

### 16.1 Checker

หน้าที่:

```text
นับจริง
แจ้ง In-Out
แจ้ง REJ
รับ task
เพิ่ม temp master
ดูรายการล่าสุดของตัวเอง
Undo draft ล่าสุด
```

ห้าม:

```text
Approve
Export
Delete official records
```

### 16.2 Supervisor

หน้าที่:

```text
ดู dashboard
ตรวจ issue
อนุมัติ transfer
อนุมัติ recount result
สร้าง investigation
assign checker task
hold/reject issue
```

### 16.3 Planner

หน้าที่:

```text
วางแผน inbound
วางแผน outbound
run storage simulation
reserve capacity
assign checker task
request capacity exception
```

### 16.4 Admin

หน้าที่:

```text
Import Inquiry
Import System In-Out
Manage Master
Manage Capacity
Replace Snapshot
Archive Data
Export Reports
Unlock Day with Reason
```

### 16.5 ERP Admin

หน้าที่:

```text
View approved ERP actions
Mark exported
Mark posted in ERP
Upload posted confirmation
Close ERP-related issues
```

### 16.6 External Warehouse

หน้าที่:

```text
Create transfer request
Create custody deposit request
View own request
Cannot edit after destination received
```

### 16.7 Viewer

หน้าที่:

```text
Read-only dashboard
Read-only reports
No create
No edit
No approve
```

---

## 17. UI/UX Requirements

### 17.1 อุปกรณ์ที่ต้องรองรับ

```text
Mobile
iPad / Tablet
PC / Desktop
```

### 17.2 Mobile

ใช้สำหรับ:

```text
Checker
หน้างาน
การนับ
แจ้ง movement
แจ้ง REJ
รับ task
```

หลัก UX:

```text
ปุ่มใหญ่
ฟอร์มสั้น
ใช้มือเดียวได้
Lock Field
Auto UOM
Last Used
Quick Add
Calculator
Recent Count
Undo ล่าสุด
Offline / Weak signal support
```

Mobile navigation:

```text
นับจริง
งานของฉัน
แจ้ง In/Out
แจ้ง REJ
ประวัติล่าสุด
```

### 17.3 iPad / Tablet

ใช้สำหรับ:

```text
Supervisor
Checker lead
ดูงานและตรวจหน้างาน
```

ควรมี:

```text
Card grid
Task list
Form + Preview
Quick approve บางกรณี
```

### 17.4 PC / Desktop

ใช้สำหรับ:

```text
Admin
Planner
Supervisor
ERP Admin
```

ควรมี:

```text
Sidebar
Dashboard
Tables
Filters
Import/Export
Approval Center
Simulation
Master Data
```

### 17.5 UX สำคัญ

```text
ไม่โชว์ code ดิบให้หน้างาน
ใช้ภาษาไทย
ใช้ status color
Mobile ใช้ card ไม่ใช้ table ยาว
Desktop ใช้ table/filter ได้
Action สำคัญต้อง confirm
หลัง save ต้อง show summary
REJ ต้องมีหน้าสรุปให้แคปส่ง Line
```

---

## 18. Home / Role Dashboard

### 18.1 เป้าหมาย

เป็นหน้าแรกหลังเข้าแอพ

ต้องแสดงทางลัดตาม role:

```text
Checker
Supervisor
Planner
Admin
ERP Admin
External Warehouse
Viewer
```

### 18.2 Quick Actions

ตัวอย่าง:

```text
เริ่มนับ
งานของฉัน
แจ้ง In-Out
แจ้ง REJ
Before / After Count
Storage Planning
Capacity
Reconciliation
Approval
Import Inquiry
Import System In-Out
Export
Master Data
```

### 18.3 Status Cards

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

---

## 19. Error Handling และ Edge Cases

### 19.1 Upload Inquiry ซ้ำ

ถ้ามี active inquiry ของวันเดียวกันแล้ว:

```text
ห้าม append
ให้เลือก Replace / Archive Old / Cancel
สร้าง version ใหม่
เก็บ log
```

### 19.2 System In-Out ยังไม่พร้อม

ถ้าเป็นข้อมูลวันนี้และ ERP ยังไม่ตัด:

```text
แสดง WAIT_SYSTEM_POST
ยังไม่สรุปผิดทันที
ตรวจวันถัดไป
```

### 19.3 Field Movement มี แต่ System ไม่มี

```text
SYSTEM_MISSING
```

ส่ง Investigation / ERP Admin / ห้องชั่ง

### 19.4 System มี แต่ Field ไม่มี

```text
FIELD_MISSING
```

ถามหน้างาน / ตรวจ movement นอกระบบ

### 19.5 Qty Diff

ใช้ Before / After Count ตัดสินต่อ

### 19.6 Location Diff

ใช้ Physical Count ของ Location ที่เกี่ยวข้องตัดสินต่อ

### 19.7 Lot Diff

ห้าม Auto Transfer

ส่ง Lot Verify หรือ Investigation

### 19.8 REJ Not Posted

รอ ERP Admin ตรวจ

ถ้าเกิน SLA แจ้ง Supervisor

### 19.9 Capacity Exceeded

ห้ามรับเข้าปกติ

ต้องขอ approval หรือเลือก alternative location

### 19.10 Multi-Lot Warning

เตือนก่อนรับ

ถ้ายืนยันให้บันทึกได้ แต่แจ้ง Admin

### 19.11 Cross-Warehouse Deposit

ต้องแยก Custody Stock

ห้ามปนกับ stock ปกติ

### 19.12 Locked Day

ถ้าปิดวันแล้ว:

```text
ห้ามนับเพิ่ม
ห้ามแก้ movement ตรง
แก้ได้เฉพาะผ่าน Investigation
```

### 19.13 Duplicate Save

ต้องใช้:

```text
idempotency_key
disable save button
duplicate hash
```

### 19.14 สัญญาณมือถือหลุด

ต้องเก็บ:

```text
LOCAL_DRAFT
PENDING_SYNC
SYNC_FAILED
```

---

## 20. Data Retention

### 20.1 ความต้องการ

เพื่อให้ระบบไม่ช้า และกันการย้อนกลับไปนับย้อนหลังนานเกินไป

### 20.2 Policy

```text
Active Data = 45 วัน
Archive Data = เกิน 45 วัน
Dashboard โหลดเฉพาะ Active
Archive อ่านอย่างเดียว
```

### 20.3 ห้าม Archive ถ้า

```text
OPEN
WAIT_RECOUNT
WAIT_INVESTIGATION
PENDING_APPROVAL
SYSTEM_MISSING
QTY_DIFF
REJ_NOT_POSTED
```

### 20.4 Archive ได้เมื่อ

```text
MATCHED
CLOSED
POSTED_IN_ERP
RESOLVED
LOCKED
```

### 20.5 Daily Lock

ทุกวันควรมีสถานะ:

```text
OPEN
COUNTING
RECONCILING
CLOSED
LOCKED
```

เมื่อ LOCKED แล้ว ห้ามแก้ตรง

---

## 21. ข้อจำกัดสำคัญ

### 21.1 จาก ERP

```text
ไม่ real-time
System In-Out ได้ย้อนหลัง
ข้อมูลวันนี้อาจยังไม่ครบ
Inquiry ไม่มี timestamp จริง
Inquiry เป็นแค่ยอดคงเหลือ ณ เวลาดึง
```

### 21.2 จากของจริง

```text
กองน้ำตาลหนัก
ตรวจ Lot ทุกกองไม่ได้
บาง Location เข้าถึงยาก
กองซ้อนกัน
นับได้แค่ Qty รวมบางกรณี
```

### 21.3 จากคน

```text
ลืมแจ้ง movement
แจ้งผิด
แจ้งช้า
นับผิด
กดผิด
ใช้มือถือในสัญญาณไม่ดี
```

### 21.4 จาก Google Sheets / Apps Script

```text
ข้อมูลเยอะแล้วช้า
Query หนัก
Concurrent users จำกัด
Quota จำกัด
ต้องมี archive และ summary table
```

---

## 22. กฎธุรกิจสำคัญ

```text
1. Inquiry เป็น Snapshot ไม่ใช่ Transaction
2. Field Movement คือสิ่งที่หน้างานแจ้ง
3. System In-Out คือสิ่งที่ ERP ตัดจริง
4. Physical Count คือความจริงจากหน้างาน
5. Expected Stock คือค่าคำนวณ
6. ERP ไม่ใช่ realtime truth
7. Adjust เป็นทางเลือกสุดท้าย
8. REJ ต้อง track ตั้งแต่แจ้งจน ERP post
9. Lot ตรวจเท่าที่ทำได้จริง
10. Capacity ต้องเช็คก่อนรับเข้า
11. Cross-Warehouse ต้องมี Custody concept
12. ทุก action ต้องมี audit log
13. Daily Lock แล้วห้ามแก้ตรง
14. Archive ได้เฉพาะข้อมูลที่ปิดแล้ว
15. Dashboard ห้ามโหลดข้อมูลเก่าโดยตรง
```

---

## 23. สิ่งที่ระบบต้องไม่ทำ

```text
ห้ามเชื่อ ERP แบบ real-time
ห้ามใช้ Inquiry เป็น transaction
ห้ามบวก movement ซ้ำ
ห้าม append inquiry ซ้ำในวันเดียว
ห้ามบังคับให้ตรวจ Lot ทุกกอง
ห้าม adjust โดยไม่ recount/investigate
ห้ามลบข้อมูล audit trail
ห้ามให้ REJ เป็นหลุมดำ
ห้ามปิดวันถ้า issue สำคัญยังเปิด
ห้ามรวม logic Restock Audit กับ Daily Control จนปนกัน
```

---

## 24. ภาพสุดท้ายของระบบ

ระบบนี้สุดท้ายจะเป็น:

```text
Warehouse Reconciliation ERP Platform
```

มี 3 แกนใหญ่:

```text
Restock / Stock Audit ERP
Daily Inventory Control ERP
Storage Planning & Movement Control
```

โดยมี Core Platform กลาง:

```text
Master Data
User / Permission
Warehouse Structure
Snapshot Control
Movement Engine
Reconciliation Engine
Approval
Investigation
Export
Archive
Audit Log
```

ระบบนี้ไม่ใช่แค่ “นับของ”

แต่เป็นระบบที่ช่วยให้คลังรู้ว่า:

```text
ของจริงอยู่ไหน
ของเป็นของใคร
ของถูกฝากไว้หรือไม่
ของเสียหายถูกแยกไป REJ หรือยัง
ERP ตัดครบไหม
Location ตรงไหม
Qty ตรงไหม
Lot เสี่ยงไหม
Capacity พอไหม
Planner ควรลงของที่ไหน
Checker ต้องไปตรวจอะไร
Supervisor ต้องอนุมัติอะไร
Admin ต้องแก้ ERP อะไร
```

---

## 25. คำจำกัดความสุดท้าย

ระบบนี้คือ:

```text
ระบบที่ทำให้ Physical Reality, Operational Reality และ ERP Reality เข้าใกล้กันมากที่สุด
```

หรือพูดแบบใช้งานจริง:

```text
ระบบที่เปลี่ยนงานนับของ งานแจ้งเข้าออก งานแก้ Location งาน REJ และงานวางแผนคลัง
ให้กลายเป็น workflow ที่ตรวจสอบได้ ปิดวันได้ และแก้ ERP ได้อย่างมีหลักฐาน
```

---

## 26. Development Direction

### Phase 1 — Core Foundation

```text
Master Data
User / Role
Location Structure
Snapshot Control
Audit Log
Basic Dashboard
```

### Phase 2 — Restock / Stock Audit

```text
Import Inquiry
Mobile Count
Analysis Result
Recount Queue
Transfer Proposal
Adjustment Proposal
Investigation
```

### Phase 3 — Daily Inventory Control

```text
Opening Stock
Field In-Out
Before / After Count
System In-Out Import
Expected Stock
Movement Reconciliation
Daily Close
```

### Phase 4 — REJ / Transfer / Capacity

```text
REJ Damage Flow
Internal Split Transfer
Capacity by Location
Multi-Lot Warning
Admin Alert
```

### Phase 5 — Planning

```text
Storage Simulation
Outbound Planning
Checker Task
Capacity Reservation
Plan vs Actual
```

### Phase 6 — Cross-Warehouse / Custody

```text
External Deposit
Cross-Warehouse Transfer
Destination Receive
Custody Stock
ERP Transfer Pending
```

### Phase 7 — Performance / Archive

```text
45-Day Active Data
Archive Sheets
Summary Tables
Dashboard Optimization
Offline Sync
```

---

## 27. Summary for Stakeholders

ระบบนี้เกิดจากปัญหาคลังที่ซับซ้อนกว่าการนับสต๊อกทั่วไป เพราะข้อมูลใน ERP ไม่ real-time, ของจริงเคลื่อนที่, Lot ตรวจยาก, Movement อาจไม่ถูก post, REJ ต้องตามต่อ, และบางครั้งมีของฝากจากคลังอื่น

ดังนั้นระบบที่ต้องการคือ Web App ที่ทำให้ทุก movement มีหลักฐาน ทุกการนับมีที่มา ทุก discrepancy มี workflow และทุกการแก้ ERP มี approval

ผลลัพธ์ที่ต้องการคือ:

```text
ลด Excel
ลดการถามย้อนหลัง
ลดของหายปลอม
ลด Location เพี้ยน
ลด ERP ตัดผิด
เพิ่มความมั่นใจตอนปิดวัน
เพิ่มความชัดเจนตอนปิดรอบ
ทำให้หัวหน้าตัดสินใจง่ายขึ้น
ทำให้หน้างานทำงานง่ายขึ้น
```

ระบบนี้ต้องออกแบบให้ใช้งานจริง ไม่ใช่แค่ dashboard สวย

หลักสำคัญคือ:

```text
หน้างานต้องใช้ง่าย
หัวหน้าต้องเห็นปัญหาเร็ว
Admin ต้องแก้ ERP ได้ถูก
Planner ต้องรู้ว่าควรเก็บของที่ไหน
ระบบต้องตรวจสอบย้อนหลังได้
```
