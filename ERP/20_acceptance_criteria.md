# 20_acceptance_criteria.md

# Acceptance Criteria Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Acceptance Criteria / Definition of Done / QA Gate Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / QA / Product Owner Use  
**Primary Use:** ใช้เป็นเกณฑ์ตรวจรับงานทั้งระบบ ตั้งแต่ Product, Data, UX, Backend, Security, Performance, Reconciliation, Export, Notification, Retention และ UAT เพื่อบอกได้ชัดเจนว่าสิ่งที่สร้าง “เสร็จจริง” หรือยัง

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Acceptance Criteria ของ Warehouse Reconciliation ERP Platform

ระบบนี้ถือว่าเสร็จไม่ได้จากแค่:

```text
เปิดหน้าได้
กดปุ่มได้
มี dashboard แล้ว
มีตารางแล้ว
```

แต่ต้องพิสูจน์ได้ว่า:

1. รับข้อมูลจริงได้
2. แยก Inquiry / In-Out / Field Movement / Count ได้ถูกต้อง
3. Map alias ได้โดยไม่ hardcode
4. Replay stock ข้ามวันได้
5. ตรวจความไม่ตรงได้พร้อม explanation
6. สร้าง issue/task ได้
7. แจ้งเตือนและ export ได้
8. ตรวจสิทธิ์และ audit ได้
9. Retention 45 วันไม่ลบข้อมูลสำคัญผิด
10. ใช้งานมือถือ/desktop ได้จริง
11. ระบบไม่ช้าเกินไปสำหรับงานประจำวัน

หลักสำคัญ:

```text
Done = ใช้งานจริงได้ + ตรวจย้อนหลังได้ + ไม่ทำให้ตัดสินใจผิด
```

---

## 2. Definition of Done

ระบบหรือ module จะถือว่าเสร็จเมื่อผ่านครบ 6 มิติ:

```text
1. Functional Done       = ทำงานตาม requirement
2. Data Done             = ข้อมูลถูกต้อง trace ได้
3. Validation Done       = กันข้อมูลผิดได้
4. Security Done         = สิทธิ์/audit ถูกต้อง
5. UX Done               = user ใช้งานเข้าใจ
6. Performance Done      = ไม่ช้าเกินงานจริง
```

ถ้าขาดมิติใดมิติหนึ่ง ห้ามถือว่า production-ready

---

## 3. Acceptance Levels

## 3.1 Level 1 — Prototype Ready

ใช้แสดง flow ได้ แต่ยังไม่ใช้ข้อมูลจริงเต็มรูปแบบ

ผ่านเมื่อ:

- UI หลักเปิดได้
- flow หลักเดินได้ด้วย sample data
- state loading/error/empty มีเบื้องต้น
- ยังอาจมี assumption อยู่

## 3.2 Level 2 — MVP Ready

ใช้กับข้อมูลจริงแบบควบคุมได้

ผ่านเมื่อ:

- รับไฟล์จริงได้
- import/alias/replay/reconcile ได้
- issue/detail/export/basic notification ได้
- permission/audit/retention พื้นฐานทำงาน
- test case สำคัญผ่าน

## 3.3 Level 3 — Production Ready

ใช้จริงต่อเนื่องได้

ผ่านเมื่อ:

- performance ผ่าน threshold
- permission/security ผ่าน
- retention ปลอดภัย
- UAT จาก role จริงผ่าน
- backup/health/monitoring มี
- critical bug = 0

---

# 4. Product Acceptance Criteria

## PAC-001 — Core Product Goal

ระบบต้องช่วยให้ user ไม่ต้องไล่เช็ค Excel/ERP/Line เองทุกแถว

ผ่านเมื่อ:

- [ ] ระบบรับข้อมูล Inquiry และ System In-Out ได้
- [ ] ระบบสร้าง expected stock จาก Anchor + Ledger ได้
- [ ] ระบบเทียบ Field/System/Count/Expected ได้
- [ ] ระบบแสดงรายการ exception ให้คนดูเฉพาะจุดที่ต้อง action
- [ ] ระบบบอก suggested action และ owner role ได้

---

## PAC-002 — Inquiry as Anchor Snapshot

Inquiry ต้องถูกใช้เป็น snapshot ไม่ใช่ transaction

ผ่านเมื่อ:

- [ ] Confirm Inquiry แล้วสร้าง inquiry_snapshot
- [ ] Confirm Inquiry แล้วสร้าง inquiry_snapshot_lines
- [ ] Inquiry ไม่สร้าง movement_ledger โดยตรง
- [ ] Upload Inquiry ซ้ำมี checksum warning หรือ snapshot policy
- [ ] Active snapshot ต่อ business_date/warehouse มี policy ชัดเจน

---

## PAC-003 — Admin Not Uploading Every Day

ระบบต้องรองรับ admin upload ข้ามวัน 1–5 วันหรือมากกว่า

ผ่านเมื่อ:

- [ ] เลือก date range สำหรับ backfill ได้
- [ ] Upload In-Out ย้อนหลังได้
- [ ] Coverage check บอกวันไหนขาดข้อมูลได้
- [ ] Stock replay ใช้ anchor snapshot ล่าสุดก่อนช่วงวันที่เลือกได้
- [ ] Reconciliation ทำกับวันที่ replay แล้วได้
- [ ] Count required locations ถูกสร้างจาก movement/issue ที่เกี่ยวข้องได้

---

## PAC-004 — System Explains Mismatch

ระบบต้องไม่บอกแค่ “ไม่ตรง” แต่ต้องอธิบายได้

ผ่านเมื่อทุก non-matched result มี:

- [ ] result_code
- [ ] severity
- [ ] confidence
- [ ] explanation
- [ ] likely_cause
- [ ] suggested_action
- [ ] owner_role
- [ ] source/evidence link

---

# 5. User, Auth & Permission Acceptance Criteria

## UAC-001 — User Registration

ผ่านเมื่อ:

- [ ] User สมัครด้วย username/password/full_name ได้
- [ ] Password ถูก hash + salt ก่อนบันทึก
- [ ] User ใหม่อยู่สถานะ PENDING_APPROVAL
- [ ] Pending user login ไม่ได้
- [ ] มี audit USER_REGISTER
- [ ] Admin เห็นรายการ pending user

---

## UAC-002 — User Approval

ผ่านเมื่อ:

- [ ] Admin approve pending request ได้
- [ ] Admin เลือก role ได้
- [ ] Admin เลือก warehouse scope ได้
- [ ] Approved user กลายเป็น ACTIVE
- [ ] Approved user login ได้
- [ ] มี audit USER_APPROVE

---

## UAC-003 — User Reject

ผ่านเมื่อ:

- [ ] Admin reject pending request ได้
- [ ] Reject แล้ว request หายจาก pending queue
- [ ] มี audit USER_REJECT_DELETE_REQUEST
- [ ] ถ้ามี reason policy ต้องบังคับกรอก reason
- [ ] Reject ไม่สร้าง active user

---

## UAC-004 — Permission Enforcement

ผ่านเมื่อ:

- [ ] Frontend ซ่อนเมนูตาม permission
- [ ] Backend block API ถ้าไม่มี permission
- [ ] Direct API call โดยไม่มี permission ถูก block
- [ ] Permission denied แสดงข้อความเข้าใจง่าย
- [ ] Sensitive permission change มี audit severity HIGH/CRITICAL

---

## UAC-005 — Warehouse Scope

ผ่านเมื่อ:

- [ ] User เห็นเฉพาะ warehouse ใน scope
- [ ] Import warehouse ที่ไม่มี scope ไม่ได้
- [ ] Count warehouse ที่ไม่มี scope ไม่ได้
- [ ] Export data นอก scope ไม่ได้
- [ ] Issue Center filter/visibility เคารพ warehouse scope

---

# 6. Checker GPS Session Acceptance Criteria

## CGS-001 — Check-in

ผ่านเมื่อ:

- [ ] Checker check-in warehouse ได้
- [ ] ระบบเก็บ business_date, warehouse_id, lat, lng, accuracy, timestamp
- [ ] Checker มี active warehouse ได้ 1 แห่งต่อวัน
- [ ] ถ้า check-in warehouse ใหม่ session เดิมถูก suspend
- [ ] มี audit CHECKER_CHECKIN และ CHECKER_SESSION_SUSPENDED ถ้ามี

---

## CGS-002 — Count Requires Active Session

ผ่านเมื่อ:

- [ ] Checker ไม่มี active session แล้ว submit count ไม่ได้
- [ ] Count location ต้องอยู่ใน warehouse ของ active session
- [ ] Session expired/suspended แล้ว submit ไม่ได้
- [ ] UI พาไปหน้า Check-in เมื่อไม่มี session

---

# 7. Import Center Acceptance Criteria

## IMP-001 — File Upload & Preview

ผ่านเมื่อ:

- [ ] เลือก file_type ได้ เช่น INQUIRY / SYSTEM_INOUT / MASTER_DATA
- [ ] Upload xlsx/csv ได้ตาม policy
- [ ] Invalid file type ถูก block
- [ ] Empty file ถูก block
- [ ] Preview headers ได้
- [ ] Preview sample rows ได้
- [ ] แสดง row count ได้
- [ ] แสดง duplicate checksum warning ได้

---

## IMP-002 — Import Batch & Raw Rows

ผ่านเมื่อ:

- [ ] ทุก upload สร้าง import_batch
- [ ] import_batch มี batch_id, file_type, file_name, checksum, uploaded_by, status
- [ ] raw rows ถูกเก็บใน import_raw_rows พร้อม batch_id, row_no, raw_json
- [ ] Chunk upload รองรับไฟล์ใหญ่ได้
- [ ] Raw rows trace กลับ batch ได้

---

## IMP-003 — Column Mapping

ผ่านเมื่อ:

- [ ] User map source column → canonical field ได้
- [ ] Required mapping missing แล้ว normalize ไม่ได้
- [ ] Mapping template ถูก save/reuse ได้
- [ ] Header changed warning แสดงได้
- [ ] ไม่มี hardcoded column index ใน business logic

---

## IMP-004 — Normalize Batch

ผ่านเมื่อ:

- [ ] Normalize อ่านจาก mapping ไม่ใช่ hardcode
- [ ] Date parsing ตรวจ error ได้
- [ ] Qty parsing ตรวจ error ได้
- [ ] Unknown alias เข้า alias_review_queue ได้
- [ ] Error rows มี import_error_logs
- [ ] Batch status update ถูกต้อง
- [ ] Normalize job มี status/progress/error count

---

## IMP-005 — Confirm Import

ผ่านเมื่อ:

- [ ] Confirm import ได้เฉพาะ batch ที่ผ่าน validation ตาม policy
- [ ] Confirm Inquiry สร้าง snapshot
- [ ] Confirm System In-Out สร้าง system_inout_records + ledger
- [ ] Confirm import มี confirmation modal
- [ ] Confirm import มี audit IMPORT_CONFIRM
- [ ] Confirm import trigger replay/reconciliation ได้ตาม policy

---

# 8. Alias Acceptance Criteria

## ALI-001 — Alias Type Separation

ผ่านเมื่อ alias แยกตาม type:

- [ ] CUSTOMER_ALIAS
- [ ] ITEM_ALIAS
- [ ] LOCATION_ALIAS
- [ ] DC_ALIAS
- [ ] UOM_ALIAS
- [ ] LOT_PATTERN optional

และ:

- [ ] Customer alias map ไป customer เท่านั้น
- [ ] Item alias map ไป item เท่านั้น
- [ ] Type mismatch ถูก block

---

## ALI-002 — Alias Review Queue

ผ่านเมื่อ:

- [ ] Unknown raw text สร้าง queue item ได้
- [ ] Queue item มี alias_type, raw_text, source_batch_id, source_row_no, confidence, status
- [ ] Admin filter queue ตาม type/status/confidence ได้
- [ ] Admin map to existing ได้
- [ ] Admin create new master ได้
- [ ] Admin reject alias ได้
- [ ] ทุก action มี audit

---

## ALI-003 — Smart Input Alias

ผ่านเมื่อ:

- [ ] User พิมพ์ข้อความป้าย เช่น MKMK W150 69 ได้
- [ ] ระบบ parse customer_alias/item_alias/year ได้ถ้ารู้จัก
- [ ] ระบบแสดง confidence ได้
- [ ] Unknown token เข้า review ได้
- [ ] Low confidence ไม่ auto-map
- [ ] Product group อื่นไม่ถูกบังคับใช้ pattern น้ำตาล

---

# 9. Master Data Acceptance Criteria

## MDM-001 — Master Data Basic

ผ่านเมื่อมี master data สำหรับ:

- [ ] Warehouse
- [ ] Location
- [ ] Customer
- [ ] Item
- [ ] Product Group
- [ ] UOM
- [ ] Reason / REJ reason basic

---

## MDM-002 — Master Data Quality

ผ่านเมื่อ:

- [ ] Master code unique
- [ ] Location ผูก warehouse
- [ ] Item ผูก product_group
- [ ] Used master ไม่ถูก hard delete
- [ ] Inactive master ไม่ถูกเลือกเป็น default ใน flow ใหม่

---

# 10. Movement & Ledger Acceptance Criteria

## MOV-001 — Field Movement

ผ่านเมื่อ:

- [ ] Create IN/OUT/TRANSFER basic ได้
- [ ] Qty ต้อง positive
- [ ] Location required ตาม movement type
- [ ] Checker ต้องมี active session ถ้าเป็นคนบันทึก
- [ ] Locked day block direct edit
- [ ] สร้าง movement_ledger source_type=FIELD_NOTICE ได้
- [ ] มี audit FIELD_MOVEMENT_CREATED

---

## MOV-002 — System In-Out Movement

ผ่านเมื่อ:

- [ ] System In-Out normalize movement_type เป็น canonical ได้
- [ ] Date/posting date valid
- [ ] Qty/sign rule valid ตาม mapping config
- [ ] Location requirement ถูกต้องตาม movement type
- [ ] สร้าง movement_ledger source_type=SYSTEM_INOUT ได้

---

## MOV-003 — Movement Ledger

ผ่านเมื่อ:

- [ ] ทุก ledger row มี source_type
- [ ] ทุก ledger row มี source_ref_id
- [ ] movement_type canonical
- [ ] stock_effect_sign ถูกต้อง
- [ ] ledger trace กลับ source record ได้
- [ ] ledger ที่ใช้ replay แล้วไม่ถูกแก้เงียบ ๆ

---

# 11. Pending Inbound Acceptance Criteria

## PIN-001 — Create Pending Inbound

ผ่านเมื่อ:

- [ ] บันทึก pending inbound ได้เมื่อข้อมูลเข้าไม่ครบ
- [ ] ต้องมี business_date, warehouse, location, qty, text_on_label หรือ photo
- [ ] ระบบ parse label text และแสดง preview ได้
- [ ] Unknown alias เข้า review ได้
- [ ] Status เริ่มเป็น PENDING_INBOUND_VERIFY หรือ WAIT_SYSTEM_INOUT

---

## PIN-002 — Confirm Pending Inbound

ผ่านเมื่อ:

- [ ] ระบบเสนอ candidate จาก In-Out ได้เมื่อมีข้อมูล
- [ ] Candidate มี match_score/reason
- [ ] Low confidence ต้อง manual review
- [ ] Confirm แล้วสร้าง canonical movement/ledger ได้
- [ ] Confirm แล้ว pending status = CONFIRMED
- [ ] มี audit PENDING_INBOUND_CONFIRMED

---

# 12. Physical Count Acceptance Criteria

## CNT-001 — Submit Count

ผ่านเมื่อ:

- [ ] Checker submit physical count ได้
- [ ] ต้องมี business_date, warehouse, location, qty_counted, count_type
- [ ] qty_counted >= 0
- [ ] Active checker session required
- [ ] Count location ต้องตรงกับ session warehouse
- [ ] Count สามารถ link task/issue ได้
- [ ] มี audit PHYSICAL_COUNT_SUBMITTED

---

## CNT-002 — Recount Task

ผ่านเมื่อ:

- [ ] Supervisor/Admin สร้าง recount task จาก issue ได้
- [ ] Task มี task_type, priority, warehouse/location, status
- [ ] Checker เห็น task ของตัวเองบน mobile
- [ ] Submit count แล้ว task status update ได้

---

# 13. REJ Acceptance Criteria

## REJ-001 — Create REJ Notice

ผ่านเมื่อ:

- [ ] REJ ต้องมี reason
- [ ] REJ ต้องมี from_location และ rej_location
- [ ] REJ qty > 0
- [ ] REJ location ต้อง valid หรือ type=REJ
- [ ] หลัง create status = WAIT_SYSTEM_POST
- [ ] มี audit REJ_NOTICE_CREATED

---

## REJ-002 — REJ Reconciliation

ผ่านเมื่อ:

- [ ] ระบบเทียบ REJ Notice กับ System In-Out ได้
- [ ] ถ้า ERP post แล้ว status update/matched ได้
- [ ] ถ้าไม่พบ ERP post ตาม policy สร้าง REJ_NOT_POSTED issue ได้
- [ ] Owner role ของ REJ_NOT_POSTED = ERP_ADMIN
- [ ] Notification event ถูกสร้างได้ถ้า rule active

---

# 14. Stock Replay Acceptance Criteria

## SRP-001 — Data Coverage

ผ่านเมื่อ coverage check แสดงสถานะของ:

- [ ] Anchor snapshot
- [ ] In-Out by date
- [ ] Unknown alias
- [ ] Pending inbound
- [ ] REJ pending
- [ ] Count required / moving location
- [ ] Overall status OK/WARNING/CRITICAL

---

## SRP-002 — Run Stock Replay

ผ่านเมื่อ:

- [ ] User/System run replay ด้วย date range ได้
- [ ] ระบบหา anchor snapshot ก่อนหรือเท่ากับช่วงวันที่เลือกได้
- [ ] ระบบโหลด movement ledger ในช่วงได้
- [ ] ระบบสร้าง daily_stock_positions ได้
- [ ] daily_stock_positions มี opening/in/out/transfer/rej/ending_expected
- [ ] มี confidence/source_mode
- [ ] Missing anchor ถูก block หรือ warning ตาม policy
- [ ] Replay job มี log/status/error count

---

## SRP-003 — Backfill Replay

ผ่านเมื่อ:

- [ ] Admin upload In-Out ย้อนหลัง 1–5 วันได้
- [ ] ระบบ replay stock ตามช่วงได้
- [ ] ระบบบอกวันที่ข้อมูลขาดได้
- [ ] ระบบสร้าง issue/task หลัง replay ได้ถ้ามี mismatch

---

# 15. Reconciliation Acceptance Criteria

## REC-001 — Reconciliation Job

ผ่านเมื่อ:

- [ ] Run reconciliation by date range/warehouse ได้
- [ ] Job มี QUEUED/RUNNING/SUCCESS/WARNING/FAILED
- [ ] Job มี rows_checked, matched_count, issue_count, warning_count, error_count
- [ ] Low coverage downgrade confidence
- [ ] Matching key ใช้ config ไม่ hardcode ถาวร

---

## REC-002 — Result Codes

ผ่านเมื่อสร้าง result_code ได้อย่างน้อย:

- [ ] MATCHED
- [ ] SYSTEM_MISSING
- [ ] FIELD_MISSING
- [ ] QTY_DIFF
- [ ] LOCATION_DIFF
- [ ] ITEM_DIFF
- [ ] LOT_DIFF
- [ ] UNKNOWN_ALIAS
- [ ] PENDING_INBOUND
- [ ] REJ_NOT_POSTED
- [ ] NEED_RECOUNT
- [ ] NEED_INVESTIGATION

---

## REC-003 — Explanation Quality

ผ่านเมื่อ issue/detail แสดงได้ว่า:

- [ ] ระบบเทียบข้อมูลแหล่งไหนกับแหล่งไหน
- [ ] Field qty คืออะไร
- [ ] System qty คืออะไร
- [ ] Count qty คืออะไร ถ้ามี
- [ ] Expected qty คืออะไร ถ้ามี
- [ ] Diff เท่าไร
- [ ] ทำไมระบบคิดว่าไม่ตรง
- [ ] ควรทำอะไรต่อ
- [ ] ใครเป็น owner

---

## REC-004 — Test Cases

ต้องผ่าน test case อย่างน้อย:

- [ ] Field vs System matched
- [ ] Field OUT 1,000 แต่ System OUT 950 → QTY_DIFF
- [ ] Field movement มี แต่ System ไม่มี → SYSTEM_MISSING
- [ ] System มี แต่ Field ไม่มี → FIELD_MISSING
- [ ] Location ไม่ตรง → LOCATION_DIFF
- [ ] Unknown customer/item → UNKNOWN_ALIAS
- [ ] Pending inbound ยังไม่ confirm → PENDING_INBOUND
- [ ] REJ แจ้งแล้ว ERP ไม่ post → REJ_NOT_POSTED
- [ ] Count ไม่ตรง expected → NEED_RECOUNT / QTY_DIFF

---

# 16. Issue Center Acceptance Criteria

## ISS-001 — Issue Creation

ผ่านเมื่อ:

- [ ] Critical/high non-matched result สร้าง issue ได้
- [ ] Issue link กับ recon_result
- [ ] Issue มี issue_type, severity, confidence, owner_role, status
- [ ] Owner role mapping ถูกต้อง
- [ ] Issue ไม่ซ้ำแบบ uncontrolled เมื่อ run reconciliation ซ้ำ

---

## ISS-002 — Issue Center UI

ผ่านเมื่อ:

- [ ] แสดง issue list ได้
- [ ] Filter by date/warehouse/status/severity/owner/result_code ได้
- [ ] Desktop เป็น table ได้
- [ ] Mobile เป็น card list ได้
- [ ] Summary cards แสดง critical/open/waiting ERP/waiting checker ได้
- [ ] Drilldown ไป Issue Detail ได้

---

## ISS-003 — Issue Detail UI

ผ่านเมื่อ:

- [ ] มี header result_code/severity/confidence/status
- [ ] มี stock identity: customer/item/year/lot/location
- [ ] มี comparison panel
- [ ] มี explanation panel
- [ ] มี suggested action
- [ ] มี source evidence / related records
- [ ] มี audit timeline
- [ ] Action button แสดงตาม permission

---

## ISS-004 — Issue Action

ผ่านเมื่อ:

- [ ] Assign issue ได้
- [ ] Create recount task ได้
- [ ] Add comment ได้
- [ ] Close issue ต้องมี reason
- [ ] Accept difference ต้องมี permission + reason
- [ ] Closed issue แก้ต่อไม่ได้ถ้าไม่ reopen
- [ ] มี audit สำหรับ assign/close/accept/reopen

---

# 17. Control Center & Dashboard Acceptance Criteria

## DSH-001 — Control Center

ผ่านเมื่อแสดง card อย่างน้อย:

- [ ] Close Readiness
- [ ] Critical Issues
- [ ] Open Issues
- [ ] Unknown Alias
- [ ] Pending Inbound
- [ ] REJ Not Posted
- [ ] Data Coverage
- [ ] Last Import
- [ ] Backfill/Replay Status
- [ ] Export Pending
- [ ] Retention Status
- [ ] DB Usage / Health

---

## DSH-002 — Dashboard Data Source

ผ่านเมื่อ:

- [ ] Dashboard อ่านจาก summary/result tables
- [ ] Dashboard ไม่ query import_raw_rows full history
- [ ] Dashboard ไม่ query movement_ledger full history ทุกครั้ง
- [ ] ทุก KPI มี business_date/warehouse context
- [ ] ทุก card มี last_updated หรือ data freshness
- [ ] Stale data warning แสดงได้

---

## DSH-003 — Drilldown

ผ่านเมื่อทุก card สำคัญ drilldown ได้:

- [ ] Critical Issues → Issue Center severity=CRITICAL
- [ ] Unknown Alias → Alias Review PENDING
- [ ] Pending Inbound → Pending Inbound Review
- [ ] REJ Not Posted → Issue Center issue_type=REJ_NOT_POSTED
- [ ] Data Coverage → Coverage Detail / Backfill Wizard
- [ ] Last Import → Import Batch Detail
- [ ] Retention Status → Retention Log

---

## DSH-004 — Close Readiness

ผ่านเมื่อ:

- [ ] Close readiness คำนวณ READY / READY_WITH_WARNING / NOT_READY ได้
- [ ] Open critical issue ทำให้ NOT_READY
- [ ] Missing anchor ทำให้ NOT_READY หรือ warning ตาม policy
- [ ] Pending inbound block/warn ตาม policy
- [ ] REJ_NOT_POSTED critical ทำให้ NOT_READY
- [ ] UI แสดง blocker list และ suggested action

---

# 18. Export Acceptance Criteria

## EXP-001 — Export Center

ผ่านเมื่อ export ได้อย่างน้อย:

- [ ] Issue List
- [ ] Daily Reconciliation
- [ ] REJ Report
- [ ] Pending Inbound Report
- [ ] Alias Review Report
- [ ] ERP Correction Template basic
- [ ] Audit Report basic
- [ ] Retention Report

---

## EXP-002 — Export Governance

ผ่านเมื่อ:

- [ ] Export ต้องมี permission
- [ ] Export ต้องมี filter หรือ selected_ids
- [ ] Export preview แสดง row_count/filters/columns/file_format
- [ ] Export ทุกครั้งสร้าง export_job/export_log
- [ ] Export file พร้อม download ได้
- [ ] Export error มี log
- [ ] ERP Template export มี permission เฉพาะและ audit HIGH/CRITICAL

---

# 19. Notification Acceptance Criteria

## NOTI-001 — Notification Center

ผ่านเมื่อ:

- [ ] สร้าง notification channel ได้
- [ ] สร้าง notification rule ได้
- [ ] สร้าง notification template ได้
- [ ] Test send ได้
- [ ] มี notification logs
- [ ] Token/secret ไม่ถูก expose ไป frontend

---

## NOTI-002 — Supported Channels

ผ่านเมื่อรองรับ abstraction สำหรับ:

- [ ] In-app
- [ ] Telegram
- [ ] LINE Messaging API
- [ ] Email

หมายเหตุ:

```text
LINE Notify ไม่ควรเป็นช่องทางหลักสำหรับระบบใหม่ ให้ใช้ LINE Messaging API
```

---

## NOTI-003 — Anti-Spam

ผ่านเมื่อ:

- [ ] Rule มี severity filter
- [ ] Rule มี cooldown_minutes
- [ ] Rule มี send_once_per_issue option
- [ ] Failed send ถูก log
- [ ] Notification failure แสดงใน Maintenance/Log ได้

---

# 20. Retention & Maintenance Acceptance Criteria

## RET-001 — 45-Day Retention

ผ่านเมื่อ:

- [ ] Retention cutoff default = today - 45 days
- [ ] มี DRY_RUN mode
- [ ] มี EXECUTE mode
- [ ] ก่อนลบมี summary/archive ตาม policy
- [ ] Matched/closed raw/staging เก่าลบ/archive ได้
- [ ] Open issue linked data ไม่ถูกลบ
- [ ] Critical audit log ไม่ถูกลบ
- [ ] Export log ไม่ถูกลบแบบผิด policy
- [ ] Retention log แสดง deleted/skipped/reason ได้

---

## RET-002 — Retention Safety

ต้องผ่าน test:

- [ ] open issue source data ไม่ถูกลบ
- [ ] pending inbound unresolved ไม่ถูกลบ
- [ ] REJ_NOT_POSTED unresolved ไม่ถูกลบ
- [ ] SYSTEM_MISSING unresolved ไม่ถูกลบ
- [ ] closed/matched data เกิน cutoff ถูกลบ/archive ได้
- [ ] retention failed มี notification/admin log

---

## HLT-001 — System Health

ผ่านเมื่อ health check แสดง:

- [ ] DB usage / storage usage
- [ ] table row counts basic
- [ ] last import time
- [ ] last replay time
- [ ] last reconciliation time
- [ ] last retention time
- [ ] failed jobs
- [ ] pending alias count
- [ ] pending inbound count
- [ ] open critical issues
- [ ] health status OK/WARNING/CRITICAL

---

# 21. Audit & Security Acceptance Criteria

## SEC-001 — Password & Session

ผ่านเมื่อ:

- [ ] Password ไม่เก็บ plain text
- [ ] Session token มี expiry
- [ ] Logout revoke session ได้
- [ ] Locked/suspended user login ไม่ได้
- [ ] Pending user login ไม่ได้

---

## SEC-002 — Audit Log

ผ่านเมื่อ critical action มี audit:

- [ ] USER_APPROVE
- [ ] USER_REJECT_DELETE_REQUEST
- [ ] PERMISSION_CHANGE
- [ ] IMPORT_CONFIRM
- [ ] ALIAS_MAP
- [ ] RUN_STOCK_REPLAY
- [ ] RUN_RECONCILIATION
- [ ] ISSUE_CLOSE
- [ ] EXPORT_CREATE
- [ ] DAILY_CLOSE
- [ ] DAILY_LOCK
- [ ] DAILY_UNLOCK
- [ ] RETENTION_DELETE
- [ ] NOTIFICATION_RULE_CHANGE

Audit ต้องมี:

- [ ] actor_id
- [ ] action
- [ ] entity_type
- [ ] entity_id
- [ ] old_value/new_value where relevant
- [ ] reason where required
- [ ] timestamp
- [ ] severity

---

## SEC-003 — Secret Handling

ผ่านเมื่อ:

- [ ] Notification tokens ใช้ token_ref หรือ server-side secret
- [ ] Raw token ไม่ส่งกลับ frontend
- [ ] Raw token ไม่ลง audit log
- [ ] Export/photo URL ไม่ public ถ้าเป็น sensitive
- [ ] Service role / database secret ไม่อยู่ client-side

---

## SEC-004 — Locked Day

ผ่านเมื่อ:

- [ ] Day status LOCKED แล้วแก้ movement/count/import/recon ตรงไม่ได้
- [ ] Unlock ต้องมี permission
- [ ] Unlock ต้องมี reason
- [ ] Lock/unlock มี audit CRITICAL
- [ ] Correction/investigation flow placeholder หรือ policy ชัดเจน

---

# 22. UX/UI Acceptance Criteria

## UX-001 — Checker Mobile

ผ่านเมื่อ:

- [ ] Checker ใช้มือถือ check-in ได้
- [ ] Checker เห็น active warehouse/session
- [ ] Checker เห็น My Tasks
- [ ] Checker แจ้งเข้า/แจ้งออก/นับจริงได้ด้วย form ที่สั้น
- [ ] ปุ่มใหญ่ touch target >= 44px
- [ ] ไม่มี table ใหญ่ใน mobile workflow หลัก
- [ ] Network error ไม่ล้างข้อมูลที่กรอกทันที

---

## UX-002 — Admin Desktop

ผ่านเมื่อ:

- [ ] Admin ใช้ sidebar/navigation ได้
- [ ] Import Center มี stepper/preview/mapping/validation
- [ ] Alias Review ใช้ง่ายด้วย queue/detail/action
- [ ] Control Center เห็น status รวมชัด
- [ ] Export/Notification/Retention ใช้จาก UI ได้ ไม่ต้องแก้ sheet/db ตรง

---

## UX-003 — Supervisor / ERP Admin

ผ่านเมื่อ:

- [ ] Supervisor เห็น issue/action-first ไม่ใช่ raw data
- [ ] ERP Admin เห็นรายการที่ต้อง post/export
- [ ] Issue Detail อ่านเข้าใจโดยไม่ต้องเปิด Excel เอง
- [ ] Suggested action ชัดเจน
- [ ] Owner/status/age แสดงชัด

---

## UX-004 — Required States

ทุก page สำคัญต้องมี:

- [ ] Loading state
- [ ] Empty state
- [ ] Error state
- [ ] Permission denied state
- [ ] Success feedback
- [ ] Validation error
- [ ] Confirmation modal สำหรับ critical action

---

# 23. Responsive Acceptance Criteria

## RSP-001 — Mobile

ผ่านเมื่อที่ 375px / 414px:

- [ ] ไม่มี horizontal scroll ใน workflow หลัก
- [ ] Bottom nav ใช้งานได้
- [ ] Form เป็น single column
- [ ] Sticky action bar ไม่บัง content
- [ ] Issue list เป็น cards
- [ ] Smart inbound/count ใช้งานได้

---

## RSP-002 — Tablet

ผ่านเมื่อที่ 768px:

- [ ] Dashboard cards 2 columns หรือ layout อ่านง่าย
- [ ] Issue review ใช้ cards/detail drawer ได้
- [ ] Modals/drawers ไม่ล้นจอ
- [ ] Filter ใช้ drawer/chips ได้

---

## RSP-003 — Desktop

ผ่านเมื่อที่ 1024px / 1440px:

- [ ] Sidebar + main content ใช้งานได้
- [ ] Tables มี filter/sort/pagination
- [ ] Import mapping ใช้ง่าย
- [ ] Issue detail comparison อ่านชัด
- [ ] Export preview ใช้งานได้

---

# 24. Performance Acceptance Criteria

## PERF-001 — Dashboard Performance

ผ่านเมื่อ:

- [ ] Control Center โหลดจาก summary/result table
- [ ] ไม่ query raw rows ทั้งหมด
- [ ] ใช้ date/warehouse/status filter
- [ ] มี pagination ใน issue/export/audit list

Target guideline:

```text
Dashboard initial load ควรตอบสนองได้ในระดับไม่รู้สึกค้างสำหรับข้อมูล MVP
```

---

## PERF-002 — Import Performance

ผ่านเมื่อ:

- [ ] Large file ใช้ chunk upload หรือ job processing
- [ ] Normalize มี job status
- [ ] Error rows ไม่ทำให้ทั้งหน้า crash
- [ ] Preview จำกัด sample rows ไม่ render ทั้งไฟล์

---

## PERF-003 — Reconciliation Performance

ผ่านเมื่อ:

- [ ] Matching ใช้ lookup/index ไม่ใช่ nested loop O(n²) แบบไม่จำเป็น
- [ ] Run reconciliation มี job status
- [ ] Re-run ช่วงเดิมไม่สร้าง duplicate issue uncontrolled
- [ ] Summary refresh เฉพาะ affected date/warehouse ได้หรือมี plan

---

## PERF-004 — Data Size Control

ผ่านเมื่อ:

- [ ] Retention 45 วันทำงาน
- [ ] Summary/archive มี
- [ ] Raw/staging เก่าถูก cleanup ตาม policy
- [ ] Health check เตือน DB usage ได้

---

# 25. Testing Acceptance Criteria

## TEST-001 — Unit Tests

ต้องมี test สำหรับ:

- [ ] Permission resolver
- [ ] Warehouse scope check
- [ ] Password hash verification
- [ ] Date parsing
- [ ] Qty parsing
- [ ] Alias type validation
- [ ] Result code engine
- [ ] Close readiness gate
- [ ] Retention skip open issue

---

## TEST-002 — Integration Tests

ต้องมี test สำหรับ:

- [ ] register → approve → login
- [ ] reject user → audit remains
- [ ] upload → preview → mapping → raw rows
- [ ] unknown alias → alias queue → map alias
- [ ] inquiry confirm → snapshot only
- [ ] inout confirm → ledger
- [ ] replay → daily stock position
- [ ] reconciliation → recon result → issue
- [ ] issue close with reason
- [ ] export → export log
- [ ] retention dry run → execute → skip open issue

---

## TEST-003 — UAT Scenarios

ต้องให้ role จริงหรือ tester ทดสอบ:

### Admin

- [ ] approve/reject user
- [ ] import Inquiry
- [ ] import In-Out backfill
- [ ] map alias
- [ ] run replay/reconciliation
- [ ] export report
- [ ] run retention dry run

### Checker

- [ ] check-in warehouse
- [ ] submit count
- [ ] smart inbound / pending inbound
- [ ] create REJ notice

### Supervisor

- [ ] review issue
- [ ] assign recount
- [ ] close issue with reason
- [ ] check close readiness

### ERP Admin

- [ ] view SYSTEM_MISSING/REJ_NOT_POSTED
- [ ] export ERP correction template
- [ ] mark/post follow-up if implemented

---

# 26. Release Gate Criteria

## Gate 1 — Foundation Ready

ผ่านเมื่อ:

- [ ] Auth works
- [ ] User approval works
- [ ] Permission backend works
- [ ] Warehouse scope works
- [ ] Audit log works
- [ ] Design system base works

---

## Gate 2 — Import & Alias Ready

ผ่านเมื่อ:

- [ ] File upload/preview works
- [ ] Column mapping works
- [ ] Raw staging works
- [ ] Normalize basic works
- [ ] Unknown alias queue works
- [ ] Alias review works

---

## Gate 3 — Data Engine Ready

ผ่านเมื่อ:

- [ ] Inquiry snapshot works
- [ ] System In-Out records work
- [ ] Movement ledger works
- [ ] Data coverage check works
- [ ] Stock replay works
- [ ] Daily stock position works

---

## Gate 4 — Reconciliation Ready

ผ่านเมื่อ:

- [ ] Reconciliation job works
- [ ] Result codes correct
- [ ] Explanation generated
- [ ] Issues created
- [ ] Issue Center/Detail works
- [ ] Core test cases pass

---

## Gate 5 — Operational MVP Ready

ผ่านเมื่อ:

- [ ] Control Center works
- [ ] Close readiness works
- [ ] Export works with logs
- [ ] Notification works with logs
- [ ] Retention works safely
- [ ] Mobile Checker flow works
- [ ] Critical bugs = 0

---

# 27. Critical Failure Conditions

ถ้ามีข้อใดข้อหนึ่ง ห้ามถือว่า production-ready:

- [ ] Password เก็บ plain text
- [ ] Pending user login ได้
- [ ] Backend permission ไม่ตรวจ
- [ ] Dashboard query raw full history จนช้า
- [ ] Inquiry ถูกใช้เป็น transaction
- [ ] In-Out ไม่เข้า movement ledger
- [ ] Unknown alias ถูก auto-map แบบไม่มี review
- [ ] Non-matched result ไม่มี explanation
- [ ] Issue close ได้โดยไม่มี reason
- [ ] Locked day แก้ตรงได้
- [ ] Retention ลบ open issue/source evidence
- [ ] Export ไม่มี log
- [ ] Token/secret expose ไป frontend
- [ ] Checker count ได้โดยไม่ check-in

---

# 28. Final Acceptance Checklist

ก่อนบอกว่า “เสร็จแล้ว” ต้องผ่าน checklist นี้:

```text
Foundation
[ ] Auth/session complete
[ ] User approval complete
[ ] Permission/backend guard complete
[ ] Warehouse scope complete
[ ] Audit log complete

Data Intake
[ ] Import staging complete
[ ] Column mapping complete
[ ] Raw row storage complete
[ ] Alias review complete
[ ] Master data basic complete

Data Engine
[ ] Inquiry anchor snapshot complete
[ ] System In-Out records complete
[ ] Movement ledger complete
[ ] Stock replay complete
[ ] Data coverage complete

Operation
[ ] Checker session complete
[ ] Field movement basic complete
[ ] Pending inbound complete
[ ] Physical count complete
[ ] REJ basic complete

Intelligence
[ ] Reconciliation complete
[ ] Result code complete
[ ] Explanation complete
[ ] Issue engine complete
[ ] Issue center/detail complete

Decision Layer
[ ] Control center complete
[ ] Close readiness complete
[ ] Export center complete
[ ] Notification center complete
[ ] Retention complete
[ ] Health check complete

Quality
[ ] Loading/error/empty states complete
[ ] Responsive mobile/tablet/desktop complete
[ ] Unit tests complete
[ ] Integration tests complete
[ ] UAT scenarios pass
[ ] Critical bugs = 0
```

---

# 29. Final Acceptance Statement

```text
Warehouse Reconciliation ERP Platform จะถือว่าเสร็จจริงก็ต่อเมื่อระบบสามารถรับ Inquiry/In-Out จริงผ่าน staging และ column mapping, map alias แบบมี review, ใช้ Inquiry เป็น Anchor Snapshot, ใช้ In-Out/Field/REJ เป็น Movement Ledger, replay stock ข้ามวัน, reconcile Field/System/Count/Expected พร้อม result_code, severity, confidence, explanation, suggested_action, สร้าง issue/task, แสดง Control Center, export/log, notification/log, retention 45 วันอย่างปลอดภัย และทุก critical action มี validation, permission และ audit log ครบ
```

