# 17_out_of_scope.md

# Out of Scope & Scope Guard Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Out of Scope / MVP Boundary / Scope Guard Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / Product Owner Use  
**Primary Use:** ใช้กำหนดขอบเขตสิ่งที่ยังไม่ควรทำใน MVP, สิ่งที่ควรเลื่อนไป future phase, สิ่งที่ห้าม AI Agent ทำเองโดยไม่มี approval และหลักการกัน scope creep เพื่อให้ระบบเร็ว คลีน ครบจบในแกนหลักก่อน ไม่บานจนช้า/bug เยอะ

---

## 1. Purpose of This Document

เอกสารนี้กำหนด **Out of Scope** ของ Warehouse Reconciliation ERP Platform

ระบบนี้มีแนวคิดใหญ่และสามารถขยายไปได้หลายทิศทาง เช่น:

- warehouse map
- capacity simulation
- relocation optimizer
- FIFO
- custody contract
- AI matching
- OCR ป้ายหน้ากอง
- offline app
- native mobile app
- ERP integration real-time

แต่ถ้าทำทุกอย่างตั้งแต่แรก ระบบจะเสี่ยง:

- ช้า
- bug เยอะ
- dev ไม่จบ
- schema ยังไม่เสถียร
- reconciliation logic ยังไม่แม่น
- user สับสน
- maintenance ยาก
- AI Agent สร้างฟีเจอร์ลอยที่ไม่ได้ตอบปัญหาแกนหลัก

ดังนั้นเอกสารนี้ใช้เป็น **Scope Guard** เพื่อบอกว่า:

1. อะไรต้องทำใน MVP
2. อะไรยังไม่ทำใน MVP
3. อะไรทำเป็น basic placeholder ได้
4. อะไรห้ามทำจนกว่าจะมีข้อมูลจริงเพิ่ม
5. ถ้าจะเพิ่ม feature ต้องผ่านเงื่อนไขอะไร
6. AI Agent ต้องไม่สร้าง feature นอก scope เอง

หลักสำคัญ:

```text
MVP ต้องแก้ปัญหา Reconciliation ก่อน
อย่าเริ่มจาก Simulation, Visual Map, FIFO หรือ AI ขั้นสูง
```

---

## 2. Scope Philosophy

## 2.1 Build the Control Backbone First

ระบบต้องเริ่มจากแกนควบคุมข้อมูล:

```text
User / Permission
→ Import Staging
→ Column Mapping
→ Alias Review
→ Anchor Snapshot
→ Movement Ledger
→ Stock Replay
→ Reconciliation
→ Issue Center
→ Control Center
→ Export / Notification / Retention
```

ถ้าแกนนี้ยังไม่เสถียร ห้ามไปทำ feature ขั้นสูงที่ต้องพึ่งข้อมูล stock ที่แม่น

---

## 2.2 Avoid Smart Before Stable

ห้ามทำระบบ “ฉลาด” ก่อนที่ข้อมูลพื้นฐานจะนิ่ง

ตัวอย่าง:

```text
ห้ามทำ AI auto matching แบบฟันธง
ถ้า Alias Strategy ยังไม่ lock
```

```text
ห้ามทำ relocation optimizer
ถ้า Daily Stock Position ยังไม่น่าเชื่อถือ
```

```text
ห้ามทำ advanced FIFO
ถ้า lot/year ยังไม่ถูก normalize ชัดเจน
```

---

## 2.3 Manual Review Before Full Automation

ใน MVP ให้ใช้แนวคิด:

```text
System Suggests
Human Confirms
System Learns / Stores Rule
```

ไม่ใช่:

```text
System Auto-Decides Everything
```

โดยเฉพาะ:

- alias mapping
- pending inbound confirmation
- accept difference
- close issue
- ERP correction export
- lock/unlock day
- retention execute

---

## 2.4 Keep Future Hooks, Not Full Future System

อนุญาตให้ data model หรือ menu มี field/future hooks ที่ไม่ทำงานเต็ม เช่น:

- product_group
- capacity_qty
- location_type
- simulation_runs table placeholder
- confidence field
- source_ref field

แต่ห้ามทำ UI/engine เต็มจนกิน scope MVP

---

# 3. MVP In Scope Summary

เพื่อให้เห็นขอบเขตชัด สิ่งที่ **ต้องอยู่ใน MVP** คือ:

```text
1. User Approval + Permission + Warehouse Scope
2. Checker GPS Daily Session basic
3. Import Center: Inquiry / System In-Out
4. Column Mapping
5. Raw Staging
6. Alias Review
7. Inquiry Anchor Snapshot
8. In-Out Backfill 1–5+ วัน
9. Movement Ledger
10. Stock Replay
11. Data Coverage Check
12. Auto Reconciliation basic
13. Result Code + Severity + Confidence
14. Explanation Detail
15. Issue Center
16. Count Required Locations
17. Smart Inbound / Pending Inbound basic
18. REJ Notice basic
19. Control Center
20. Export Center basic
21. Notification Center basic
22. Daily Close / Lock basic
23. Retention 45 days basic
24. Audit Log
25. Loading / Error / Empty states
```

---

# 4. Out of Scope Categories

สิ่งที่ไม่อยู่ใน MVP แบ่งเป็นกลุ่มดังนี้:

```text
1. Advanced Warehouse Optimization
2. Advanced FIFO / Lot Strategy
3. Full Visual Warehouse Map
4. Full Custody / Contract Management
5. Cross-Warehouse Full Workflow
6. Native Mobile App / Full Offline
7. Advanced AI / OCR / Predictive Intelligence
8. Real-Time ERP Integration
9. Advanced Finance / Costing
10. Complex Approval Workflow
11. Enterprise Security / SSO
12. Advanced Analytics / Forecast Dashboard
13. Full Document Management
14. Hardware / IoT Integration
15. Multi-company / Multi-tenant Advanced Setup
```

---

# 5. Out of Scope Detail

# 5.1 Advanced Warehouse Optimization

## Not in MVP

- Relocation Optimizer เต็มรูปแบบ
- Available Location engine ขั้นสูง
- Auto move recommendation ที่สร้าง movement อัตโนมัติ
- Multi-scenario storage planning เต็มระบบ
- Capacity balancing แบบ algorithm ซับซ้อน
- Move small to large แบบ execute อัตโนมัติ
- Slotting optimization
- Space utilization forecasting เต็มรูปแบบ

## Why Not Now

Feature กลุ่มนี้ต้องใช้ข้อมูลที่ต้องแม่นก่อน:

- daily stock position
- location capacity
- product dimensions/pack
- lot/year/customer rules
- actual physical location
- current occupancy
- movement cost
- operation constraint

ถ้าข้อมูลยังไม่แม่น ระบบจะเสนอแผนย้ายผิด และทำให้ operation หน้างานพัง

## Allowed in MVP

ทำได้เฉพาะ basic placeholder:

```text
- location capacity field
- location occupancy summary basic
- manual note ว่า location เต็ม/ว่าง
- simulation module marked as future
```

## Future Entry Criteria

ค่อยทำเมื่อ:

- stock replay accuracy ผ่านเกณฑ์
- location master + capacity ถูกต้อง
- lot/customer/product group rule ชัด
- มีตัวอย่าง scenario จริง 20–50 เคส
- Supervisor ยืนยัน workflow relocation จริง

---

# 5.2 Advanced FIFO / Lot Strategy

## Not in MVP

- Advanced FIFO engine
- FEFO / expiry-based picking
- Auto lot allocation
- Lot rotation optimization
- FIFO violation dashboard ขั้นสูง
- Lot aging forecast
- lot split/merge control ขั้นสูง

## Why Not Now

ตอนนี้ปัญหาแกนหลักคือ:

```text
ชื่อ alias ไม่ตรง
location/DC ไม่ตรง
Inquiry ไม่มาทุกวัน
In-Out ต้อง backfill
Field vs ERP ยังต้อง reconcile
```

ถ้า lot/year ยัง normalize ไม่แม่น การทำ FIFO จะทำให้ผลลัพธ์ผิด

## Allowed in MVP

- เก็บ year / lot field ใน data model
- แสดง lot/year ใน issue detail ถ้ามี
- result_code LOT_DIFF basic
- warning ถ้า lot ไม่ตรงแบบง่าย

## Future Entry Criteria

- lot/year อยู่ใน canonical field ชัด
- sample lot movement ครบหลายวัน
- business rule FIFO ได้รับการยืนยัน
- มี UAT case สำหรับ FIFO violation

---

# 5.3 Full Visual Warehouse Map

## Not in MVP

- Interactive visual warehouse map
- Drag-and-drop location planning
- Heatmap occupancy แบบ realtime
- 2D/3D warehouse layout
- visual stack/pile representation
- map-based relocation simulation

## Why Not Now

Visual map สวย แต่ไม่ใช่แกนแก้ปัญหาตอนนี้ ถ้าข้อมูล stock/location ยังไม่ตรง map จะกลายเป็นภาพที่หลอก user

## Allowed in MVP

- Location list/table
- location status badge
- count required locations list
- simple occupancy card
- future link placeholder

## Future Entry Criteria

- location master ถูกต้อง
- warehouse layout มี source ชัด
- location coordinates/zone พร้อม
- stock position เชื่อถือได้
- user ต้องการ map เพื่อ decision จริง ไม่ใช่แค่สวย

---

# 5.4 Full Custody / Contract Management

## Not in MVP

- Custody contract full workflow
- Customer contract rate/term
- billing by storage day
- customer statement
- external customer custody legal workflow
- multi-party approval contract
- storage fee calculation

## Why Not Now

ตอนนี้มีแนวคิดลูกค้านอก/ลูกค้าใน และกฎ lot/location ที่ต่างกัน แต่ยังไม่ควรทำ contract module เต็ม เพราะจะเพิ่ม business domain ใหม่ที่ไม่ใช่ reconciliation core

## Allowed in MVP

- customer_type = INTERNAL / EXTERNAL
- owner/customer field
- permission/location policy placeholder
- note ว่าสินค้านี้เป็นลูกค้านอกได้
- future custody module placeholder

## Future Entry Criteria

- ต้องมี requirement จากฝ่ายบัญชี/สัญญา
- มีตัวอย่าง customer custody จริง
- ต้องรู้ billing/report ที่ต้องการ
- ต้องแยก ownership/location policy ชัด

---

# 5.5 Cross-Warehouse Full Workflow

## Not in MVP

- Cross-warehouse transfer approval full cycle
- Inter-warehouse custody reconciliation
- Multi-WH routing
- transfer in transit tracking ขั้นสูง
- WH-to-WH billing/control

## Why Not Now

MVP ต้องทำให้ warehouse เดียวหรือ scope พื้นฐาน reconcile ได้ก่อน

## Allowed in MVP

- warehouse_id ทุก table
- user warehouse scope
- checker session per warehouse
- import by warehouse
- report by warehouse
- basic transfer record if data exists

## Future Entry Criteria

- แต่ละ warehouse ใช้ flow เดียวกันได้ stable
- inter-WH transfer data จาก ERP มี field ชัด
- มี test case cross-warehouse จริง

---

# 5.6 Native Mobile App / Full Offline Sync

## Not in MVP

- Native iOS/Android app
- Full offline-first database sync
- conflict resolution engine
- background sync
- push notification native
- device management

## Why Not Now

ต้องเริ่มด้วย responsive web app ก่อน เพราะเร็วกว่าและเปลี่ยน logic ได้ง่ายกว่า

## Allowed in MVP

- Mobile-first responsive web UI สำหรับ Checker
- large touch target
- photo upload
- GPS check-in via browser
- keep form data in memory if network fails
- retry submit basic

## Future Entry Criteria

- checker ใช้ web mobile แล้วพบข้อจำกัดจริง
- มี requirement offline ชัด เช่น no signal zone
- มี budget/time สำหรับ native maintenance
- มี conflict scenario ชัดเจน

---

# 5.7 Advanced AI / OCR / Predictive Intelligence

## Not in MVP

- AI auto matching แบบฟันธงโดยไม่ review
- OCR ป้ายหน้ากองแบบ production
- computer vision ตรวจสินค้า/กอง
- predictive anomaly detection ขั้นสูง
- chatbot assistant สำหรับถาม stock แบบ natural language
- auto root cause ด้วย AI โดยไม่มี rule proof
- recommendation engine ที่แก้ข้อมูลเอง

## Why Not Now

ตอนนี้ยังต้องสร้างฐาน rule-based + audit ให้มั่นคงก่อน AI ขั้นสูงจะปลอดภัย

## Allowed in MVP

- smart text parser rule-based
- alias suggestion with confidence
- explanation generated from deterministic rules
- manual review queue
- AI-ready fields เช่น confidence, suggested_action

## Future Entry Criteria

- มี historical data เพียงพอ
- result จาก rule engine แม่นแล้ว
- มี ground truth สำหรับ training/evaluation
- มี human review loop
- มี audit ว่า AI แนะนำอะไรและใคร approve

---

# 5.8 Real-Time ERP Integration

## Not in MVP

- Real-time ERP API integration
- direct ERP posting
- automated ERP adjustment
- two-way sync
- webhook integration
- SAP/Oracle/ERP connector full production

## Why Not Now

ปัจจุบัน requirement ยังอยู่ที่ upload Inquiry/In-Out และดึงย้อนหลังได้ การทำ ERP integration จริงจะเพิ่ม security, approval, mapping และ dependency สูงมาก

## Allowed in MVP

- Import System In-Out from Excel/CSV
- Export ERP correction template
- ERP Admin mark posted
- source_ref field
- integration-ready API structure

## Future Entry Criteria

- ERP system มี API/documentation
- data owner อนุมัติ
- security review ผ่าน
- export template ถูกใช้จริงจน stable
- ERP Admin process ชัดเจน

---

# 5.9 Advanced Finance / Costing

## Not in MVP

- cost of inventory difference
- financial adjustment journal
- storage cost calculation
- loss/damage financial impact
- customer billing
- GL integration

## Why Not Now

ระบบยังอยู่ที่ operational reconciliation ไม่ใช่ finance posting

## Allowed in MVP

- export report ให้ฝ่ายบัญชีใช้ต่อได้
- issue severity based on qty/value placeholder if value exists
- no accounting entry generation

## Future Entry Criteria

- มี unit cost / price / billing rule
- บัญชียืนยัน posting flow
- ERP finance integration requirement ชัด

---

# 5.10 Complex Approval Workflow

## Not in MVP

- multi-level approval chain
- dynamic approval matrix ตาม amount/value
- delegation workflow
- approval SLA escalation เต็มระบบ
- e-signature

## Why Not Now

MVP ต้องมี approval พื้นฐานก่อน เช่น user approval, close day, accept diff

## Allowed in MVP

- Admin approve/reject user
- Supervisor/Admin close issue
- reason required for critical action
- status + audit
- optional owner_role

## Future Entry Criteria

- มี role hierarchy จริง
- มี policy ว่าใคร approve อะไรตามระดับความเสี่ยง
- มี compliance requirement

---

# 5.11 Enterprise Security / SSO

## Not in MVP

- SSO/SAML/OAuth enterprise
- MFA mandatory
- device management
- IP allowlist
- advanced audit retention policy
- SOC2-level controls

## Why Not Now

MVP ควรทำ basic auth/session/permission/audit ให้ถูกก่อน

## Allowed in MVP

- password hash + salt
- session token
- role/permission
- warehouse scope
- audit log
- no raw secret exposure

## Future Entry Criteria

- องค์กรต้องการ security standard
- มีผู้ใช้เยอะขึ้น
- ระบบใช้จริงกับข้อมูล sensitive มากขึ้น

---

# 5.12 Advanced Analytics / Forecast Dashboard

## Not in MVP

- advanced forecast demand/supply
- stockout prediction
- workload forecast
- seasonality model
- risk scenario simulator เต็มรูปแบบ
- machine learning dashboard

## Why Not Now

ต้องมี historical clean data ก่อน forecast ถึงจะน่าเชื่อถือ

## Allowed in MVP

- basic aging
- basic count trend
- issue count trend
- storage DB usage trend
- simple warning threshold

## Future Entry Criteria

- มี clean history อย่างน้อย 2–3 เดือนขึ้นไป
- metric stable
- business action จาก forecast ชัดเจน

---

# 5.13 Full Document Management

## Not in MVP

- full document repository
- version control ของเอกสารสัญญา
- e-signature
- approval packet generation
- document OCR/search

## Allowed in MVP

- attach photo evidence
- store file_url for export
- import batch metadata
- audit link to source row/batch

---

# 5.14 Hardware / IoT Integration

## Not in MVP

- barcode scanner integration แบบ dedicated
- RFID
- weighing scale integration
- camera station automation
- gate sensor
- IoT stock measurement

## Allowed in MVP

- manual input
- mobile camera upload
- future field for scanned_code if needed

---

# 5.15 Multi-company / Multi-tenant Advanced Setup

## Not in MVP

- multiple companies with isolated tenants
- tenant billing
- tenant admin hierarchy
- per-tenant branding
- cross-tenant analytics

## Allowed in MVP

- warehouse scope
- role/permission
- customer_type

---

# 6. Feature Status Matrix

| Feature | MVP Status | Reason |
|---|---|---|
| User Approval | In Scope | security foundation |
| Permission + Warehouse Scope | In Scope | prevent wrong access |
| Checker GPS Session | In Scope Basic | control field count |
| Import Staging | In Scope | data foundation |
| Column Mapping | In Scope | avoid hardcode schema |
| Alias Review | In Scope | solve name mismatch |
| Inquiry Anchor Snapshot | In Scope | replay stock |
| In-Out Backfill | In Scope | admin may upload late |
| Movement Ledger | In Scope | stock effect source |
| Stock Replay | In Scope | reconstruct daily position |
| Reconciliation | In Scope | core value |
| Issue Center | In Scope | human exception handling |
| Control Center | In Scope | decision UI |
| Export Basic | In Scope | operation/ERP handoff |
| Notification Basic | In Scope | alert owner |
| Retention 45 Days | In Scope | free/stability control |
| Capacity Simulation | Future | needs stable stock/capacity |
| Relocation Optimizer | Future | complex and risky |
| Visual Warehouse Map | Future | not core now |
| Advanced FIFO | Future | needs lot stability |
| Native App | Future | web mobile first |
| OCR | Future | need image dataset |
| Real-time ERP API | Future | high dependency/security |
| Finance Costing | Future | separate domain |

---

# 7. Explicit Do Not Build List

AI Agent / Developer ห้ามสร้างสิ่งต่อไปนี้ใน MVP โดยไม่ได้รับ approval:

```text
1. Relocation optimizer that auto-creates move tasks from capacity simulation
2. Visual drag/drop warehouse map
3. Full FIFO/FEFO engine
4. AI auto-match that updates master without review
5. OCR production flow
6. Native mobile app
7. Full offline database sync
8. Direct ERP posting
9. Finance journal/accounting module
10. Multi-level approval engine
11. Contract/billing module
12. Multi-tenant SaaS admin
13. Real-time IoT/RFID integration
14. Complex ML forecast dashboard
15. Any feature that requires schema not proven by real Inquiry/In-Out files
```

---

# 8. Placeholder Rules

บาง feature ในอนาคตสามารถเตรียม placeholder ได้ แต่ต้องไม่ทำ logic เต็ม

## 8.1 Allowed Placeholder

| Future Feature | Allowed Placeholder |
|---|---|
| Capacity Simulation | location.capacity_qty, occupancy summary basic |
| FIFO | year/lot field, LOT_DIFF result code |
| Visual Map | location list + zone field |
| Custody | customer_type INTERNAL/EXTERNAL |
| OCR | photo_url evidence only |
| ERP Integration | export template + source_ref |
| Forecast | summary tables with history |
| Offline | retry submit + keep form state |

---

## 8.2 Placeholder Must Be Clearly Marked

UI/Docs ต้องระบุว่าเป็น future/basic ไม่ใช่ feature สมบูรณ์

ตัวอย่าง:

```text
Capacity Simulation: Future Phase
```

```text
OCR: Not available in MVP, attach photo only
```

---

# 9. Scope Change Control

ถ้าต้องการเพิ่ม feature นอก scope ต้องตอบคำถามก่อน:

1. Feature นี้แก้ปัญหา P0 หรือไม่?
2. ต้องใช้ข้อมูลที่เรายังไม่มีหรือไม่?
3. กระทบ data model ไหน?
4. กระทบ workflow ไหน?
5. กระทบ permission/audit/retention หรือไม่?
6. มี sample data/test case แล้วหรือยัง?
7. ทำเป็น manual/basic ก่อน full automation ได้ไหม?
8. ถ้าไม่ทำตอนนี้ ระบบ MVP ยังใช้งานได้ไหม?
9. ถ้าทำตอนนี้ จะทำให้ core reconciliation ช้าลงไหม?
10. ใครเป็น owner และ acceptance criteria คืออะไร?

ถ้าตอบข้อ 1 ไม่ใช่ หรือข้อ 6 ยังไม่มี → ควรเลื่อนไป future phase

---

# 10. MVP Protection Rules for AI Agent

AI Agent ต้องทำตามกฎนี้:

```text
1. อ่าน Out of Scope ก่อนเสนอ feature ใหม่
2. ถ้า feature อยู่ใน out of scope ห้าม build
3. ถ้าจำเป็น ให้สร้าง TODO/Future Note แทน
4. ห้ามเปลี่ยน data model เพื่อรองรับ future feature มากเกินไป
5. ห้ามสร้าง UI menu ของ future feature เป็น active page
6. ห้ามสร้าง mock simulation แล้วให้ user เข้าใจว่าใช้งานจริงได้
7. ห้ามเพิ่ม ML/AI auto decision โดยไม่มี human approval
8. ห้าม direct ERP posting
9. ห้ามลบ/แก้ข้อมูล locked day โดยตรง
10. ห้ามทำ dashboard จาก raw full history เพื่อโชว์ feature เร็ว ๆ
```

---

# 11. Deferred Feature Backlog

Feature ที่เลื่อนไปอนาคตควรเก็บใน backlog แบบนี้:

| Feature | Phase | Dependency | Entry Criteria |
|---|---|---|---|
| Capacity Simulation | Phase 5 | stock position + capacity master | accuracy stable |
| Relocation Suggestion | Phase 5+ | capacity + movement cost | approved policy |
| Visual Warehouse Map | Phase 5 | layout master | location stable |
| Advanced FIFO | Phase 5 | lot/year reliable | FIFO rule approved |
| OCR Label Reader | Phase 6 | photo dataset | accuracy test |
| Real-Time ERP API | Phase 6 | ERP API access | security approval |
| Native Mobile App | Phase 6 | web usage proof | offline need confirmed |
| Advanced Forecast | Phase 6 | clean history | action use case clear |
| Custody Billing | Phase 6 | contract/rate data | accounting requirement |

---

# 12. Out of Scope Risk if Ignored

| Ignored Scope Guard | Likely Damage |
|---|---|
| Build simulation before stock replay stable | suggestion wrong, user loses trust |
| Build FIFO before lot normalized | false FIFO issue |
| Build visual map before location clean | beautiful but wrong map |
| Build AI auto match early | wrong alias corrupts master |
| Build direct ERP post early | financial/ERP risk |
| Build native app early | high maintenance, slow change |
| Build offline sync early | conflict bugs |
| Build dashboard from raw | slow, expensive, timeout |
| Build contract billing early | domain complexity explodes |
| Build approval matrix early | slows MVP without proven need |

---

# 13. What To Do Instead

แทนที่จะทำ feature ใหญ่ ให้ทำสิ่งนี้ก่อน:

## Instead of Relocation Optimizer

ทำ:

```text
Count Required Locations
Location Occupancy Basic
Manual Suggested Action
```

## Instead of Full FIFO

ทำ:

```text
Year/Lot display
LOT_DIFF basic
Lot missing warning
```

## Instead of Visual Map

ทำ:

```text
Location table
Zone filter
Critical location card
```

## Instead of OCR

ทำ:

```text
Smart text input
Photo evidence upload
Alias Review
```

## Instead of Direct ERP Integration

ทำ:

```text
Export ERP Template
ERP Admin mark posted
System In-Out import next day
```

## Instead of AI Auto Matching

ทำ:

```text
Suggested match with confidence
Human review
Audit mapping
```

## Instead of Native App

ทำ:

```text
Responsive Web App
Mobile bottom nav
Retry submit
```

---

# 14. Acceptance Criteria

Out of Scope ถือว่าถูกควบคุมได้ถ้า:

- [ ] MVP ไม่มี relocation optimizer เต็มรูปแบบ
- [ ] MVP ไม่มี advanced FIFO engine
- [ ] MVP ไม่มี visual warehouse map เต็มรูปแบบ
- [ ] MVP ไม่มี direct ERP posting
- [ ] MVP ไม่มี AI auto-map แบบไม่ review
- [ ] MVP ไม่มี native mobile app
- [ ] MVP ไม่มี full offline sync
- [ ] MVP ไม่มี contract/billing module
- [ ] Future features ถูกเก็บเป็น backlog/future note เท่านั้น
- [ ] Placeholder ไม่ทำให้ user เข้าใจผิดว่า feature พร้อมใช้งาน
- [ ] AI Agent ไม่เพิ่ม feature นอก docs เอง
- [ ] Build order เริ่มที่ Foundation → Data Engine → Reconciliation → Control
- [ ] ทุก scope change ต้องตอบ scope change control questions

---

# 15. Final Out of Scope Statement

```text
Out of Scope ของ Warehouse Reconciliation ERP Platform มีหน้าที่ป้องกันไม่ให้ MVP บานจนช้าและพัง โดยระบบต้องโฟกัสที่การนำเข้าข้อมูลจริง, alias, anchor snapshot, backfill, movement ledger, stock replay, reconciliation, issue, control center, export, notification, audit และ retention ก่อน ส่วน simulation, relocation, FIFO, visual map, OCR, native app, direct ERP integration, finance, contract และ advanced AI ต้องเลื่อนไป future phase จนกว่าข้อมูลจริงและ core reconciliation จะนิ่งพอ
```

