# 01_product_brief.md

# Product Brief

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Product Brief / Business Requirement Foundation  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer Use  
**Primary Use:** ใช้เป็นเอกสารหลักเพื่อบอกว่า “ระบบนี้คืออะไร สร้างเพื่อแก้ปัญหาอะไร ใครใช้ และ MVP ต้องมีอะไรบ้าง” ก่อนเริ่มออกแบบโครงสร้างหรือเขียนโค้ด

---

## 1. Product Name

```text
Warehouse Reconciliation ERP Platform
```

ชื่อระบบภาษาไทยที่อธิบายได้ง่าย:

```text
ระบบควบคุมและตรวจสอบสต๊อกคลังเทียบ ERP
```

หรือ:

```text
ระบบพิสูจน์ความจริงของคลัง ระหว่างของจริง หน้างาน และ ERP
```

---

## 2. Product Vision

สร้างระบบ Web App สำหรับควบคุม ตรวจสอบ และ reconcile ข้อมูลคลังสินค้าระหว่าง:

1. **ของจริงในคลัง** ที่นับได้จากหน้างาน
2. **ข้อมูลที่หน้างานแจ้ง** เช่น เข้า ออก ย้าย เสียหาย REJ
3. **ข้อมูล ERP** เช่น Inquiry และ System In-Out
4. **ข้อมูลที่ระบบคำนวณเอง** จาก Anchor Snapshot + Movement Ledger

เป้าหมายคือให้ระบบช่วยคิดแทนคน:

- ช่วยเช็คว่าข้อมูลตรงหรือไม่ตรง
- ช่วยบอกว่าต่างตรงไหน
- ช่วยบอกว่าสินค้า/ลูกค้า/location/lot ไหนเกี่ยวข้อง
- ช่วยบอกว่าน่าจะผิดที่ฝั่งไหน
- ช่วยเสนอว่าต้องให้ใครทำอะไรต่อ
- ช่วยสร้าง issue/task/notification อัตโนมัติ
- ช่วย export ข้อมูลออกไปให้ ERP Admin หรือผู้บริหารได้
- ช่วยปิดวัน/ล็อควันอย่างมีหลักฐาน

แนวคิดหลัก:

```text
ระบบนี้ไม่ใช่ระบบกรอกข้อมูล
ระบบนี้คือระบบช่วยตัดสินใจและควบคุมความถูกต้องของคลัง
```

---

## 3. Why This Product Exists

ปัญหาปัจจุบันของคลังไม่ได้เกิดจาก “ไม่มีข้อมูล” แต่เกิดจากข้อมูลหลายแหล่งไม่ตรงกัน และต้องใช้คนไล่เช็คเอง

แหล่งข้อมูลที่เกี่ยวข้องมีหลายชุด:

```text
Inquiry
System In-Out
Field Movement
Physical Count
REJ Notice
Transfer
Pending Inbound
Location/Capacity
Alias/Master Data
```

แต่ละชุดมีความหมายไม่เหมือนกัน:

```text
Inquiry = Snapshot
System In-Out = ERP posted movement
Field Movement = หน้างานแจ้ง
Physical Count = ของจริง
Expected Stock = ระบบคำนวณ
```

ถ้าปนข้อมูลเหล่านี้ผิด จะเกิดปัญหา:

- ยอดเบิ้ล
- ยอดหาย
- ERP post ไม่ครบแต่ไม่มีใครรู้
- หน้างานแจ้งแล้วแต่ ERP ยังไม่ตัด
- ERP มี movement แต่หน้างานไม่แจ้ง
- Location ใน ERP กับของจริงไม่ตรง
- ของเข้าแต่ยังไม่รู้ว่าเป็นสินค้าอะไรครบ
- ชื่อย่อบนป้ายไม่ตรงกับชื่อใน ERP
- Admin หยุดหลายวันแล้วข้อมูลขาดช่วง
- ต้องนับย้อนหลังหลายวันแต่ไม่รู้ต้องนับตรงไหน

ระบบนี้จึงต้องถูกสร้างเพื่อ “จัดระเบียบความจริงของคลัง” ไม่ใช่แค่ทำฟอร์มบันทึก

---

## 4. Core Problem Statement

ระบบคลังปัจจุบันมีปัญหาหลักดังนี้

### 4.1 ERP ไม่ Real-Time

ERP อาจไม่ได้แสดงสถานะของวันนี้ทันที ข้อมูล System In-Out อาจต้องดึงวันถัดไปหรือดึงย้อนหลังหลายวัน

ผลกระทบ:

```text
หน้างานแจ้งออกวันนี้
แต่ ERP ยังไม่ post
Dashboard ถ้าเชื่อ ERP อย่างเดียวจะผิด
```

ระบบต้องรองรับสถานะ:

```text
WAIT_SYSTEM_POST
SYSTEM_MISSING
RECONSTRUCTED_FROM_LEDGER
```

---

### 4.2 Inquiry เป็น Snapshot ไม่ใช่ Transaction

Inquiry คือยอดคงเหลือ ณ จุดเวลาหนึ่ง ไม่ใช่ movement เข้า/ออก

ความเสี่ยง:

```text
ถ้าเอา Inquiry ไปบวก movement ซ้ำ → ยอดเบิ้ล
ถ้า upload Inquiry ซ้ำแบบ append → ยอดผิดทั้งระบบ
```

ระบบต้องใช้ Inquiry เป็น:

```text
Anchor Snapshot
```

ไม่ใช่:

```text
Movement Transaction
```

---

### 4.3 Admin อาจไม่ได้ Upload Inquiry ทุกวัน

Admin อาจ:

- ลา
- หยุด
- งานยุ่ง
- ไม่ได้ดึง Inquiry ทุกวัน
- ต้อง upload ข้ามวัน 4–5 วัน

ดังนั้นระบบห้ามบังคับว่าทุกวันต้องมี Inquiry

ระบบต้องรองรับ:

```text
Anchor Snapshot ล่าสุด
+ System In-Out ย้อนหลัง
+ Field Movement
= Stock Replay / Daily Stock Position
```

---

### 4.4 แจ้งเข้า ยากกว่าแจ้งออก

แจ้งออกมักมีของเดิมอยู่แล้ว จึงรู้ว่าออกจากกองไหน เป็นสินค้าอะไร

แต่แจ้งเข้าอาจเกิดกรณี:

- ของเข้ามาก่อน แต่ยังไม่รู้ชื่อสินค้าครบ
- ป้ายหน้ากองเขียนเป็นชื่อย่อ
- ยังไม่รู้ customer จริง
- ยังไม่รู้ lot หรือ year
- ต้องรอ In-Out วันถัดไปมาช่วยยืนยัน

ระบบต้องมี:

```text
Pending Inbound
Unknown Customer
Unknown Item
Wait ERP Confirm
Photo Evidence
Candidate Match from In-Out
```

---

### 4.5 ชื่อย่อ / Master Alias ไม่ตรงกับ ERP

หน้างานอาจเขียน:

```text
MKMK W150 69
UPUP ไฮโพลดิบ
RE T1 Brown 68
```

แต่ ERP อาจใช้ชื่อเต็ม เช่น:

```text
น้ำตาลทรายมิตรผล W150 (50KG.) ปี 68/69
```

ระบบต้องแยก alias ให้ถูก:

```text
MKMK / UPUP / UFUF = Customer Alias
W150 / RE T1 / SR CT1 / ไฮโพลดิบ = Item Alias
W8 / DC08 = DC/Warehouse Alias
A01 / W8-A01 = Location Alias
```

ห้ามรวมทุก alias ไว้ก้อนเดียวโดยไม่แยก type

---

### 4.6 Location และ DC ไม่ตรงกัน

ERP อาจใช้รหัส DC/Location แบบหนึ่ง แต่หน้างานเรียกอีกแบบหนึ่ง

ตัวอย่าง pattern:

```text
ERP DC = DC08
หน้างาน = W8
ระบบกลาง = WH_W8
```

```text
ERP Location = A01
หน้างาน = W8-A01 หรือ A1
ระบบกลาง = LOC_W8_A01
```

ระบบต้องมี:

- DC Alias
- Warehouse Alias
- Location Alias
- Canonical Location ID

---

### 4.7 REJ อาจแจ้งแล้ว แต่ ERP ยังไม่ Post

REJ คือ movement สำคัญ เพราะเป็นของเสีย/เสียหาย/แยกออกจาก stock ปกติ

ความเสี่ยง:

```text
หน้างานแจ้ง REJ แล้ว
แต่ ERP ไม่ post เข้า REJ location
ทำให้ของจริงกับ ERP ไม่ตรง
```

ระบบต้องจับ:

- REJ_NOT_POSTED
- REJ_QTY_DIFF
- REJ_LOCATION_DIFF
- REJ_LOT_DIFF

และต้องส่งงานต่อให้ ERP Admin หรือ Supervisor

---

### 4.8 คนไม่ควรต้องเช็คเองทุกแถว

ปัญหาสำคัญคือคนต้องไล่เช็คเองว่า:

- ตรงไหม
- ไม่ตรงเพราะอะไร
- ต้องดู ERP หรือ Field
- ต้องนับใหม่ไหม
- ต้องให้ใครแก้

ระบบต้องเปลี่ยนจาก:

```text
Human checks everything
```

เป็น:

```text
System checks everything
Human reviews exceptions only
```

---

## 5. Target Users

### 5.1 Admin

หน้าที่:

- อนุมัติ user
- จัดการ permission
- upload Inquiry/In-Out
- map alias
- ดู Control Center
- export
- ตั้ง notification
- ดู retention/maintenance

สิ่งที่ Admin ต้องการ:

- ไม่ต้องแก้ Sheet/DB ตรง
- เห็นข้อมูลที่ต้อง action
- รู้ว่า upload สำเร็จหรือ fail
- รู้ว่า alias ไหนยังไม่ map
- รู้ว่า close day ได้ไหม

---

### 5.2 Supervisor

หน้าที่:

- ดู issue/exception
- assign task
- approve/reject
- สั่ง recount
- close issue
- ช่วยปิดวัน

สิ่งที่ Supervisor ต้องการ:

- เห็นเฉพาะปัญหา ไม่ใช่ raw data ทั้งหมด
- รู้ว่า issue ไหน critical
- รู้ว่าใครต้องทำอะไรต่อ
- เห็น explanation ก่อนตัดสินใจ

---

### 5.3 Checker

หน้าที่:

- check-in ที่ warehouse
- นับของจริง
- แจ้งเข้า/ออก
- แจ้ง REJ
- ถ่ายรูปป้าย/goods evidence
- ทำ task ที่ได้รับมอบหมาย

สิ่งที่ Checker ต้องการ:

- มือถือใช้ง่าย
- ฟอร์มสั้น
- ปุ่มใหญ่
- กรอกน้อย
- พิมพ์ชื่อย่อจากป้ายได้
- ไม่ต้องเลือก dropdown ยาว
- บันทึก pending ได้ถ้าไม่รู้ข้อมูลครบ

---

### 5.4 ERP Admin

หน้าที่:

- ตรวจรายการที่ ERP ยังไม่ post
- post transfer / adjustment / REJ
- confirm posted
- export/import template ERP

สิ่งที่ ERP Admin ต้องการ:

- เห็นรายการที่ต้อง post
- เห็น SYSTEM_MISSING / REJ_NOT_POSTED
- export ได้ตาม template
- mark posted ได้

---

### 5.5 Planner

หน้าที่ใน phase หลัง:

- ดู capacity
- simulation การรับเข้า
- วางแผน location
- เสนอ relocation

ใน MVP ยังไม่ใช่ focus หลัก

---

### 5.6 Viewer / Management

หน้าที่:

- ดู summary
- ดูสถานะ stock/reconciliation
- ดูรายงาน

สิ่งที่ต้องการ:

- ภาพรวมที่เชื่อถือได้
- ไม่ต้องดู raw data
- เห็น issue aging / risk / close status

---

## 6. Product Goals

### Goal 1: ลดการเช็ค manual

ระบบต้องเช็คให้ก่อน:

- Field vs System
- Expected vs Count
- REJ vs ERP
- Pending Inbound vs In-Out
- Anchor vs Replay

### Goal 2: รองรับ admin ไม่ upload ทุกวัน

ระบบต้องรองรับ:

- Inquiry เป็น Anchor Snapshot
- In-Out Backfill
- Stock Replay
- Data Coverage Check

### Goal 3: ทำให้ชื่อไม่ตรงกันจัดการได้

ระบบต้องมี:

- Alias Center
- Alias Review Queue
- Smart Input
- Canonical Master

### Goal 4: ให้คนดูเฉพาะ exception

ระบบต้องสร้าง:

- Issue Center
- Result Code
- Severity
- Confidence
- Suggested Action

### Goal 5: ใช้งานง่ายจริง

ระบบต้องมี:

- Mobile Checker UI
- Smart input
- Pending inbound
- Detail explanation
- Control Center

### Goal 6: ตรวจย้อนหลังได้

ระบบต้องมี:

- Audit Log
- Export Log
- Notification Log
- Retention Log
- Lock/Close Control

### Goal 7: ทำงานเร็วและไม่บวม

ระบบต้องมี:

- active data 45 วัน
- summary/archive
- retention job
- dashboard อ่าน summary
- chunk upload
- import staging

---

## 7. Non-Goals

ระบบนี้ใน MVP ไม่ได้ตั้งใจทำสิ่งเหล่านี้ทันที:

1. ไม่ใช่ ERP เต็มรูปแบบ
2. ไม่ใช่ WMS เต็มรูปแบบ
3. ไม่ใช่ระบบบัญชี
4. ไม่ใช่ระบบคุม forklift/task แบบ real-time tracking
5. ไม่ใช่ระบบ AI vision อ่านป้ายจากรูปอัตโนมัติใน MVP
6. ไม่ใช่ full offline mobile app
7. ไม่ใช่ advanced FIFO optimizer เต็มรูปแบบ
8. ไม่ใช่ visual warehouse map เต็มรูปแบบ
9. ไม่ใช่ full relocation optimizer ใน MVP
10. ไม่ใช่ cross-warehouse custody workflow เต็มรูปแบบใน MVP

---

## 8. MVP Scope

MVP ต้องมี feature ต่อไปนี้

### 8.1 User Approval + Permission

- user สมัครและตั้งรหัสได้
- ยังใช้งานไม่ได้จนกว่า Admin อนุมัติ
- Admin เลือก role / permission / warehouse scope
- Reject แล้วลบ request แต่เก็บ audit log
- user มีหลาย role/permission ได้

### 8.2 Import Center

- upload Inquiry
- upload System In-Out
- preview file headers/rows
- column mapping
- chunk upload
- raw staging
- normalize
- import log

### 8.3 Alias Center

- Customer Alias
- Item Alias
- Location Alias
- DC Alias
- UOM Alias
- Alias Review Queue
- suggested match + confidence

### 8.4 Inquiry Anchor Snapshot

- Inquiry เป็น snapshot
- เก็บ snapshot_id
- business_date
- checksum
- uploaded_at
- active/archive/replace policy

### 8.5 In-Out Backfill

- upload In-Out ย้อนหลัง 1–5 วันหรือมากกว่า
- coverage check
- missing day warning
- import batch status

### 8.6 Movement Ledger

- รวม movement จากหลาย source
- source_type/source_ref
- movement_type
- customer/item/location/year/lot/qty/uom

### 8.7 Stock Replay

- ใช้ Anchor + Ledger
- สร้าง Daily Stock Position
- confidence ตาม coverage
- รองรับ reconstructed/estimated/final

### 8.8 Auto Reconciliation

- Field vs System
- Expected vs Count
- REJ vs ERP
- Pending Inbound vs In-Out
- result_code
- severity
- confidence
- explanation
- suggested_action

### 8.9 Issue Center

- แสดงเฉพาะ exception
- assign owner
- status workflow
- issue aging
- action buttons

### 8.10 Explanation Detail

ทุก issue ต้องแสดง:

- Field data
- System data
- Count data
- Expected data
- Difference
- Explanation
- Likely cause
- Suggested action

### 8.11 Control Center

หน้าแรกสำหรับ Admin/Supervisor:

- matched count
- issue count
- critical issue
- pending alias
- pending inbound
- REJ not posted
- export pending
- close readiness
- retention status

### 8.12 Export Center

- export selected rows
- export filtered rows
- daily report
- issue report
- ERP template basic
- export log

### 8.13 Notification Center Basic

- In-app notification
- Telegram
- LINE Messaging API later
- Email optional
- notification rule
- template
- log
- cooldown

### 8.14 Retention 45 Days

- active raw data 45 วัน
- archive summary
- skip open issues
- retention log
- health check

---

## 9. Future Scope

ทำหลัง MVP stable

### Phase Advanced 1: Capacity

- location capacity
- available capacity
- near capacity
- capacity exceeded

### Phase Advanced 2: Available Location Simulation

- policy-based lot rule
- customer type
- external/internal customer behavior
- candidate locations
- risk score

### Phase Advanced 3: Relocation Suggestion

- move small to large
- free location for inbound
- movement cost
- approval flow

### Phase Advanced 4: Custody / Cross-Warehouse

- stock owner
- physical keeper
- ERP owner
- custody report

### Phase Advanced 5: Advanced FIFO / Lot Risk

- multi-lot warning
- FIFO risk score
- lot verification task

---

## 10. Success Metrics

ระบบถือว่าสำเร็จถ้า:

### Operation Metrics

- ลดเวลาการเช็ค Field vs ERP
- ลดจำนวนรายการที่ต้องไล่เองใน Excel
- ลด unknown alias ซ้ำ
- ลด REJ ที่ค้างไม่ post
- ปิดวันได้เร็วขึ้น
- Admin ไม่ต้องแก้ Sheet/DB ตรง

### Data Metrics

- % matched สูงขึ้น
- % unknown alias ลดลง
- % pending inbound ที่ confirm สำเร็จ
- จำนวน issue critical ลดลง
- coverage ของ In-Out ย้อนหลังครบขึ้น

### UX Metrics

- Checker แจ้งเข้า/ออกได้ภายในเวลาสั้น
- Admin map alias ได้โดยไม่ต้องแก้ database
- Supervisor เข้าใจ issue detail โดยไม่ต้องถามเพิ่ม
- ERP Admin export/post ได้จากระบบ

### System Metrics

- import ไม่ timeout
- dashboard โหลดเร็ว
- retention job สำเร็จ
- notification ไม่ spam
- open issue ไม่ถูกลบ

---

## 11. Product Risks

| Risk | Impact | Mitigation |
|---|---|---|
| ไฟล์จริง column ไม่ตรง assumption | สูง | Data Profiling + Column Mapping |
| Alias ผิด | สูง | Alias Review + confidence |
| Inquiry ซ้ำ | สูง | snapshot checksum + replace/archive policy |
| Admin ไม่ upload ทุกวัน | สูง | Anchor + Backfill + Replay |
| In-Out ขาดบางวัน | สูง | Coverage Check + confidence downgrade |
| User ไม่เข้าใจ issue code | กลาง | Explanation Detail |
| Dashboard ช้า | สูง | Summary tables only |
| ลบข้อมูล 45 วันผิด | สูง | Retention rule + skip open issue |
| Notification spam | กลาง | cooldown/digest/send once |
| Scope ใหญ่เกิน | สูง | MVP lock + out of scope |

---

## 12. Key Product Principles

### Principle 1: Trust No Raw Data Immediately

ข้อมูลที่เข้ามาต้องผ่าน staging, mapping, alias, validation ก่อนใช้คำนวณ

### Principle 2: Explain Every Exception

ถ้าระบบบอกว่าไม่ตรง ต้องบอกเหตุผลและ next action

### Principle 3: Humans Review Exceptions Only

คนไม่ควรไล่ดูรายการ matched ทั้งหมด

### Principle 4: Keep Logic Configurable

column, alias, notification, permission, retention ต้องแก้ผ่านระบบ ไม่ใช่แก้ code

### Principle 5: Archive Before Delete

ข้อมูลเกิน 45 วันต้องสรุป/เก็บ log ก่อนลบ raw

### Principle 6: Build Foundation Before Intelligence

ต้องสร้าง import, alias, user, audit, staging ก่อน reconciliation/simulation

---

## 13. Product Build Strategy

แนวทางสร้างที่ถูกต้อง:

```text
Build Foundation Now
Profile Data in Parallel
Lock Schema v0
Then Build Reconciliation
```

ลำดับแนะนำ:

1. User Approval
2. Permission
3. Import Staging
4. Column Mapping
5. Alias Review
6. Audit Log
7. Anchor Snapshot
8. In-Out Backfill
9. Movement Ledger
10. Stock Replay
11. Reconciliation
12. Issue Center
13. Control Center
14. Export
15. Notification
16. Retention

ห้ามเริ่มจาก:

- Relocation Optimizer
- Visual Warehouse Map
- Advanced FIFO
- Full Simulation

---

## 14. Definition of Done for Product Brief

Product Brief นี้ถือว่า complete เมื่อ:

- [x] ระบุ vision ชัดเจน
- [x] ระบุ problem จริงของ operation
- [x] ระบุ users หลัก
- [x] ระบุ MVP scope
- [x] ระบุ out of scope
- [x] ระบุ future scope
- [x] ระบุ success metrics
- [x] ระบุ product risk
- [x] ระบุ build strategy

---

## 15. Final Product Statement

```text
Warehouse Reconciliation ERP Platform คือระบบที่ช่วยให้คลังไม่ต้องเช็คข้อมูลเองทุกแถว แต่ให้ระบบรวบรวมข้อมูลจาก Inquiry, System In-Out, Field Movement และ Physical Count มาคำนวณ ตรวจจับความไม่ตรง สร้าง issue อธิบายสาเหตุ เสนอ action แจ้งเตือน export และ audit ได้ครบ เพื่อให้ทีมคลังควบคุม stock ได้แม้ ERP ไม่ real-time และ admin ไม่ได้ upload ทุกวัน
```

This product must be built as a decision support and reconciliation platform, not as a simple stock entry form.

