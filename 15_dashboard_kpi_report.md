# 15_dashboard_kpi_report.md

# Dashboard, KPI & Report Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Dashboard / KPI / Report / Decision Support Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / BI Analyst / UX Designer Use  
**Primary Use:** ใช้กำหนดตัวชี้วัด Dashboard, KPI logic, Report, Drilldown, Export และ Decision Framework เพื่อให้ระบบไม่ได้แค่แสดงตัวเลข แต่ช่วยบอกว่าอะไรผิด ใครต้องทำอะไร และปิดวันได้หรือยัง

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Dashboard, KPI และ Report ของ Warehouse Reconciliation ERP Platform

ระบบนี้มีข้อมูลหลายชั้น:

```text
Physical Count
Field Movement
System In-Out
Inquiry Snapshot
Expected Stock / Stock Replay
Reconciliation Result
Issue / Task
Export / Notification / Retention
```

ดังนั้น Dashboard ต้องไม่ใช่แค่หน้ารวมตัวเลข แต่ต้องเป็น **Decision Control Center** ที่ตอบคำถามหลักให้ผู้ใช้ได้ทันที:

1. วันนี้ข้อมูลครบไหม
2. ERP กับหน้างานตรงกันไหม
3. มี issue อะไรต้องแก้ก่อน
4. มี alias/pending inbound ที่ค้างไหม
5. REJ ถูก post ใน ERP แล้วหรือยัง
6. Admin upload/backfill ล่าสุดเมื่อไหร่
7. ปิดวันได้ไหม
8. มีอะไรเสี่ยงต่อการตัดสินใจผิด
9. ต้อง export อะไรต่อ
10. ระบบ retention/health ปกติไหม

หลักสำคัญ:

```text
Dashboard ต้องช่วยตัดสินใจและพาไป action
ไม่ใช่แค่โชว์ตัวเลขเยอะ ๆ
```

---

## 2. Dashboard Design Philosophy

## 2.1 Decision-First KPI

ทุก KPI ต้องมี purpose ชัดเจน:

```text
ตัวเลขนี้บอกอะไร?
ใครต้องดู?
ถ้าผิดปกติ ต้องทำอะไรต่อ?
กด drilldown ไปไหน?
```

ห้ามสร้าง KPI ที่ไม่มี decision action

ตัวอย่าง KPI ที่ดี:

```text
REJ Not Posted = 3
Decision: ERP Admin ต้องตรวจ post REJ
Action: Drilldown ไป Issue Center filtered REJ_NOT_POSTED
```

ตัวอย่าง KPI ที่ไม่ดี:

```text
Total Rows = 82,345
```

ถ้าไม่มี context หรือ action จะไม่ช่วยงานจริง

---

## 2.2 Exception-First Dashboard

ลำดับความสำคัญของ Dashboard:

```text
1. Critical / Blocker
2. Pending Action
3. Warning / Risk
4. Summary
5. Historical Trend
6. Raw Detail
```

Matched records ไม่ควรแสดงเยอะเกินไป เพราะไม่ใช่สิ่งที่ต้องแก้

---

## 2.3 Summary Table, Not Raw Query

Dashboard ต้องอ่านจาก summary/result tables ไม่ใช่ raw table ทั้งหมด

Allowed sources:

- daily_recon_summary
- daily_issue_summary
- recon_results summary
- issues
- alias_review_queue summary
- pending_inbounds summary
- import_summary
- retention_logs
- system_health_checks
- daily_close_status

Prohibited for dashboard default:

```text
query import_raw_rows ทั้งหมด
query movement_ledger full history ทุกครั้ง
query raw file rows แบบไม่ aggregate
```

---

## 2.4 Context Required

ทุก KPI ต้องมี context:

- business_date
- warehouse
- filter scope
- last_updated
- source_mode if relevant
- confidence/coverage status if relevant

ถ้าไม่มี context user จะตีความตัวเลขผิด

---

# 3. Dashboard Types

ระบบควรมี Dashboard หลัก 5 ประเภท

| Dashboard | Primary User | Purpose | MVP |
|---|---|---|---:|
| Control Center | Admin / Supervisor | ดูสถานะรวมและ action วันนี้ | Yes |
| Issue Dashboard | Supervisor / Admin / ERP Admin | วิเคราะห์ exception และ aging | Yes |
| Data Coverage Dashboard | Admin | ดูข้อมูลครบ/ขาดและ backfill | Yes |
| Operation Dashboard | Checker / Supervisor | งานนับ/movement/REJ | Basic |
| Maintenance Dashboard | Admin | retention, DB usage, job health | Yes |

Future:

| Dashboard | Purpose |
|---|---|
| Capacity Dashboard | occupancy/location risk |
| Simulation Dashboard | scenario planning |
| Forecast Dashboard | stock risk / workload |

---

# 4. Control Center Dashboard

## 4.1 Purpose

Control Center คือหน้าแรกของ Admin/Supervisor ใช้ดูภาพรวมประจำวัน และตัดสินใจว่า:

```text
ต้องแก้อะไรก่อน
ต้องให้ใครทำอะไร
ปิดวันได้ไหม
ระบบ health ปกติไหม
```

---

## 4.2 Required Filters

Control Center ต้องมี filter หลัก:

- business_date
- warehouse
- product_group optional
- customer optional
- issue_status optional

Default:

```text
business_date = today
warehouse = user default scope / all allowed warehouses
```

---

## 4.3 Control Center KPI Cards

### KPI-CC-001 — Close Readiness

| Field | Detail |
|---|---|
| KPI Name | Close Readiness |
| Purpose | บอกว่าปิดวันได้หรือยัง |
| Formula | readiness gate result from daily_close_status / coverage check |
| Values | READY / READY_WITH_WARNING / NOT_READY |
| User | Admin / Supervisor |
| Action | View Close Readiness Detail |
| Drilldown | Daily Close Readiness page |
| Severity | Red if NOT_READY, Yellow if warning, Green if ready |

Decision:

```text
ถ้า NOT_READY → ห้าม final close และต้องดู blocker
ถ้า READY_WITH_WARNING → ปิดเบื้องต้นได้แต่ต้องเห็น risk
ถ้า READY → ปิดวันได้ตาม permission
```

---

### KPI-CC-002 — Critical Issue Count

| Field | Detail |
|---|---|
| KPI Name | Critical Issues |
| Purpose | จำนวน issue ด่วนที่ block การปิดวันหรือเสี่ยงสูง |
| Formula | count(issues where severity='CRITICAL' and status not in CLOSED/RESOLVED) |
| User | Admin / Supervisor |
| Action | Open critical issue list |
| Drilldown | Issue Center filtered severity=CRITICAL |

Decision:

```text
ถ้า > 0 → ต้องแก้ก่อนปิด final
```

---

### KPI-CC-003 — Open Issue Count

| Field | Detail |
|---|---|
| KPI Name | Open Issues |
| Purpose | ปริมาณ exception ที่ยังไม่จบ |
| Formula | count(issues where status in OPEN, ASSIGNED, IN_REVIEW, WAIT_ERP_ADMIN, WAIT_CHECKER_COUNT) |
| User | Admin / Supervisor |
| Drilldown | Issue Center filtered open statuses |

Sub-breakdown:

- Critical
- High
- Medium
- Low
- Waiting ERP
- Waiting Checker
- Waiting Admin

---

### KPI-CC-004 — Matched Count

| Field | Detail |
|---|---|
| KPI Name | Matched Count |
| Purpose | จำนวนรายการที่ระบบเทียบแล้วตรง |
| Formula | count(recon_results where result_code='MATCHED') |
| User | Admin / Supervisor |
| Drilldown | Reconciliation Results filtered MATCHED |
| Display Rule | ไม่ควรเด่นกว่า issue |

Decision:

```text
ใช้ดูความสมบูรณ์โดยรวม แต่ไม่ใช่ primary action
```

---

### KPI-CC-005 — Match Rate

| Field | Detail |
|---|---|
| KPI Name | Match Rate |
| Purpose | สัดส่วนรายการที่ตรงจากทั้งหมด |
| Formula | matched_count / total_reconciled_count * 100 |
| User | Admin / Supervisor / Management |
| Warning | ถ้า coverage ต่ำ ต้องแสดง confidence warning |

Formula:

```text
Match Rate = MATCHED / (MATCHED + NON_MATCHED + WARNING_RESULTS)
```

Important:

```text
ถ้าข้อมูล coverage ไม่ครบ ห้ามแสดง Match Rate แบบมั่นใจสูง
```

---

### KPI-CC-006 — Unknown Alias Count

| Field | Detail |
|---|---|
| KPI Name | Unknown Alias |
| Purpose | จำนวนชื่อที่ระบบยัง map master ไม่ได้ |
| Formula | count(alias_review_queue where status='PENDING') |
| User | Admin |
| Action | Map alias |
| Drilldown | Alias Review filtered PENDING |

Breakdown:

- Customer Alias
- Item Alias
- Location Alias
- DC Alias
- UOM Alias

Decision:

```text
ถ้า Unknown Alias > 0 → Admin ต้อง review ก่อน recon/close เชื่อถือได้
```

---

### KPI-CC-007 — Pending Inbound Count

| Field | Detail |
|---|---|
| KPI Name | Pending Inbound |
| Purpose | ของเข้าที่ข้อมูลยังไม่ครบ |
| Formula | count(pending_inbounds where status not in CONFIRMED/REJECTED/CLOSED) |
| User | Admin / Supervisor |
| Action | Confirm inbound / wait In-Out |
| Drilldown | Pending Inbound Review |

Decision:

```text
ถ้ามี pending inbound → final close อาจถูก block หรือ warning
```

---

### KPI-CC-008 — REJ Not Posted Count

| Field | Detail |
|---|---|
| KPI Name | REJ Not Posted |
| Purpose | REJ ที่หน้างานแจ้งแล้วแต่ ERP ยังไม่ post |
| Formula | count(issues where issue_type='REJ_NOT_POSTED' and status open) |
| User | Supervisor / ERP Admin |
| Action | Send/assign to ERP Admin |
| Drilldown | Issue Center filtered REJ_NOT_POSTED |

Decision:

```text
ถ้า > 0 → ERP Admin ต้องตรวจ post/adjust
```

---

### KPI-CC-009 — System Missing Count

| Field | Detail |
|---|---|
| KPI Name | System Missing |
| Purpose | Field แจ้ง movement แต่ไม่พบใน ERP In-Out |
| Formula | count(recon_results/issues where result_code='SYSTEM_MISSING' and open) |
| User | Supervisor / ERP Admin |
| Action | ตรวจ ERP post |
| Drilldown | Issue Center filtered SYSTEM_MISSING |

---

### KPI-CC-010 — Quantity Difference Count

| Field | Detail |
|---|---|
| KPI Name | QTY Difference |
| Purpose | จำนวนรายการที่ยอดต่าง |
| Formula | count(result_code='QTY_DIFF' open) |
| User | Supervisor |
| Action | Assign recount / send ERP |
| Drilldown | Issue Center filtered QTY_DIFF |

Optional value:

```text
Total absolute diff qty
```

แต่ต้องแยกตาม UOM/product group เพื่อไม่รวมหน่วยมั่ว

---

### KPI-CC-011 — Location Difference Count

| Field | Detail |
|---|---|
| KPI Name | Location Difference |
| Purpose | ของอยู่ผิด location หรือ ERP/Field ใช้ location ไม่ตรง |
| Formula | count(result_code='LOCATION_DIFF' open) |
| User | Supervisor / Checker |
| Action | Verify location / recount |
| Drilldown | Issue Center filtered LOCATION_DIFF |

---

### KPI-CC-012 — Count Required Locations

| Field | Detail |
|---|---|
| KPI Name | Count Required Locations |
| Purpose | location ที่ควรให้นับ/verify |
| Formula | count distinct locations from issues requiring count/recount |
| User | Supervisor / Checker |
| Action | Create checker tasks |
| Drilldown | Count Required Locations page |

Criteria examples:

- QTY_DIFF
- LOCATION_DIFF
- moving location with no recent count
- pending inbound at location
- REJ pending

---

### KPI-CC-013 — Data Coverage Status

| Field | Detail |
|---|---|
| KPI Name | Data Coverage |
| Purpose | ข้อมูลช่วงวันที่เลือกครบพอไหม |
| Formula | data_coverage_checks.overall_status |
| Values | COVERAGE_OK / WARNING / CRITICAL |
| User | Admin |
| Drilldown | Data Coverage Detail / Backfill Wizard |

Coverage components:

- Anchor snapshot exists
- In-Out complete
- Alias pending
- Pending inbound
- REJ pending
- Count missing for moving location

---

### KPI-CC-014 — Last Import Status

| Field | Detail |
|---|---|
| KPI Name | Last Import |
| Purpose | ดูว่าไฟล์ล่าสุดเข้าเมื่อไหร่และสำเร็จไหม |
| Formula | latest import_batches by file_type/date/scope |
| User | Admin |
| Drilldown | Import Batch Detail |

Display:

- latest Inquiry upload
- latest In-Out upload
- failed import count
- warning count

---

### KPI-CC-015 — Backfill Status

| Field | Detail |
|---|---|
| KPI Name | Backfill Status |
| Purpose | ดูงาน replay ย้อนหลังล่าสุด |
| Formula | latest backfill/replay job status |
| User | Admin |
| Drilldown | Backfill Job Detail / Replay Log |

Values:

- Not needed
- Running
- Completed
- Completed with warning
- Failed

---

### KPI-CC-016 — Export Pending Count

| Field | Detail |
|---|---|
| KPI Name | Export Pending |
| Purpose | จำนวน export job ที่ยังไม่พร้อม/ยังไม่ได้ส่ง |
| Formula | count(export_jobs where status in QUEUED/RUNNING/READY and not downloaded/sent depending type) |
| User | Admin / ERP Admin |
| Drilldown | Export Center |

---

### KPI-CC-017 — Notification Failed Count

| Field | Detail |
|---|---|
| KPI Name | Notification Failed |
| Purpose | แจ้งเตือนส่งไม่สำเร็จ |
| Formula | count(notification_logs where status='FAILED' in selected period) |
| User | Admin |
| Drilldown | Notification Logs |

---

### KPI-CC-018 — Retention Status

| Field | Detail |
|---|---|
| KPI Name | Retention Status |
| Purpose | ดู cleanup 45 วันทำงานปกติไหม |
| Formula | latest retention_jobs.status |
| User | Admin |
| Drilldown | Retention Log |

Display:

- last run time
- deleted rows
- skipped open issues
- status

---

### KPI-CC-019 — DB Usage / Storage Risk

| Field | Detail |
|---|---|
| KPI Name | DB Usage |
| Purpose | คุม free tier / storage limit |
| Formula | system_health_checks.db_usage_percent |
| User | Admin |
| Drilldown | System Health |

Warning thresholds:

```text
< 70% = OK
70–85% = Warning
> 85% = Critical
```

---

# 5. Issue Dashboard

## 5.1 Purpose

Issue Dashboard ใช้วิเคราะห์ exception และ workload ของผู้รับผิดชอบ

Primary users:

- Supervisor
- Admin
- ERP Admin

---

## 5.2 Issue Dashboard KPIs

| KPI | Formula | Decision |
|---|---|---|
| Open Issues | count open issues | workload รวม |
| Critical Issues | count open critical | ต้องแก้ก่อน |
| Aging Over SLA | count open issue where age > SLA | ติดค้างเกินกำหนด |
| Waiting ERP | count status=WAIT_ERP_ADMIN | ERP Admin ต้องทำ |
| Waiting Checker Count | count status=WAIT_CHECKER_COUNT | ต้อง assign/check count |
| Unknown Alias Issues | count issue_type=UNKNOWN_ALIAS | Admin map alias |
| Pending Inbound Issues | count issue_type=PENDING_INBOUND | confirm inbound |
| REJ Not Posted | count issue_type=REJ_NOT_POSTED | ERP post |
| QTY_DIFF | count issue_type=QTY_DIFF | recount/ERP check |
| LOCATION_DIFF | count issue_type=LOCATION_DIFF | verify location |

---

## 5.3 Issue Aging Metrics

### KPI-ISS-001 — Average Issue Age

Formula:

```text
Average age = avg(now - issue.created_at) for open issues
```

Display by:

- severity
- owner_role
- issue_type
- warehouse

---

### KPI-ISS-002 — SLA Breach Count

Formula:

```text
count(open issues where now > due_at)
```

Default SLA suggestion:

| Severity | SLA |
|---|---|
| CRITICAL | same day / 4 hours |
| HIGH | 1 business day |
| MEDIUM | 2–3 business days |
| LOW | 5 business days |

MVP สามารถเริ่มด้วย simple age threshold ก่อน

---

## 5.4 Issue Drilldown Required Fields

Issue Dashboard drilldown ต้องไป Issue Center พร้อม filter:

- issue_type
- severity
- status
- owner_role
- assigned_to
- business_date
- warehouse

---

# 6. Data Coverage Dashboard

## 6.1 Purpose

ใช้ดูว่าข้อมูลที่ระบบใช้คำนวณ/reconcile ครบพอหรือไม่ โดยเฉพาะกรณี admin upload ข้ามวัน

---

## 6.2 Coverage KPIs

| KPI | Formula | Decision |
|---|---|---|
| Anchor Status | has active Inquiry before range | replay ได้ไหม |
| In-Out Coverage | days with In-Out / days in range | data missing ไหม |
| Unknown Alias Count | pending alias | normalize/recon เสี่ยงไหม |
| Pending Inbound Count | pending inbound open | close ได้ไหม |
| REJ Pending Count | REJ WAIT_SYSTEM_POST | ERP post ครบไหม |
| Moving Location Without Count | location movement but no count | ต้องนับไหม |
| Replay Confidence | derived from coverage | เชื่อ stock position ได้แค่ไหน |

---

## 6.3 Coverage Calendar

ควรแสดงเป็น calendar/list รายวัน:

| Date | Inquiry Anchor | In-Out | Alias | Pending Inbound | REJ | Count | Status |
|---|---|---|---|---|---|---|---|
| 2026-05-01 | OK | OK | 0 | 0 | 0 | OK | OK |
| 2026-05-02 | Anchor prev | Missing | 2 | 1 | 0 | Missing | Warning |
| 2026-05-03 | Anchor prev | OK | 0 | 0 | 1 | OK | Warning |

Status:

- OK
- Warning
- Critical
- No Activity

---

## 6.4 Coverage Status Logic

```text
CRITICAL if missing anchor and no approved estimate
CRITICAL if open critical issues block close
WARNING if missing in-out day but replay can estimate
WARNING if pending alias exists
WARNING/CRITICAL if pending inbound affects stock
OK if all required sources present or resolved
```

---

# 7. Operation Dashboard

## 7.1 Purpose

ใช้สำหรับ Supervisor/Checker ดูงานหน้างานและ status movement/count

MVP อาจทำ basic โดยแสดง task และ count required locations

---

## 7.2 Operation KPIs

| KPI | Formula | User | Decision |
|---|---|---|---|
| My Open Tasks | count tasks assigned to checker and open | Checker | ทำงานอะไรต่อ |
| Count Tasks Open | count checker_tasks type COUNT/RECOUNT open | Supervisor | assign/check |
| Count Submitted Today | count physical_counts today | Supervisor | งานนับคืบหน้า |
| Movement Reported Today | count field_movements today | Supervisor | มี movement หน้างานเท่าไร |
| Pending REJ | count rej_notices WAIT_SYSTEM_POST | Supervisor/ERP | ตาม post |
| Checker Active Sessions | count active checker sessions | Admin/Supervisor | ใคร active ที่คลังไหน |

---

## 7.3 Checker Mobile Dashboard

Checker Home ต้องแสดง:

- Active warehouse session
- My open tasks
- Quick actions
- Recent submissions
- Warning if no session

KPI สำหรับ Checker ต้องไม่เยอะเกินไป

---

# 8. Maintenance Dashboard

## 8.1 Purpose

ใช้ให้ Admin ดูสุขภาพระบบ ลดการต้องมาคอยเช็คเอง

---

## 8.2 Maintenance KPIs

| KPI | Formula | Decision |
|---|---|---|
| DB Usage | current storage usage | free tier risk |
| Last Import | max uploaded_at | data stale ไหม |
| Last Replay | latest replay job | engine ทำงานไหม |
| Last Reconciliation | latest recon job | result update ไหม |
| Last Retention | latest retention job | cleanup ทำงานไหม |
| Failed Jobs | count failed jobs last 24h/7d | ต้องแก้ระบบ |
| Notification Failed | failed notification count | channel มีปัญหาไหม |
| Open Critical Issues | issue blocker | business risk |
| Pending Alias | queue backlog | data quality risk |

---

## 8.3 Health Status Logic

```text
OK = no failed critical job, DB usage < threshold, retention recent
WARNING = DB usage high, pending alias high, job delayed
CRITICAL = retention failed, DB usage critical, replay/recon failed, open critical issue high
```

---

# 9. Report Specifications

ระบบควรมี report หลักดังนี้

| Report | Primary User | MVP | Purpose |
|---|---|---:|---|
| Daily Reconciliation Report | Admin/Supervisor | Yes | สรุป matched/issue รายวัน |
| Issue Aging Report | Supervisor | Yes | ดู issue ค้าง |
| Pending Inbound Report | Admin/Supervisor | Yes | ดู inbound ที่ยังไม่ confirm |
| Alias Review Report | Admin | Yes | ดู alias pending/mapped |
| REJ Report | ERP Admin/Supervisor | Yes | ดู REJ status/posting |
| ERP Correction Template | ERP Admin | Basic | ส่งให้ ERP post/adjust |
| Export Log Report | Admin | Yes | ตรวจ export |
| Retention Report | Admin | Yes | ตรวจ cleanup |
| User Approval Report | Admin | Basic | ดู approve/reject history |
| Audit Log Report | Admin | Basic | ตรวจ action สำคัญ |

---

# 10. Report Detail

## 10.1 Daily Reconciliation Report

### Purpose

รายงานสรุปผลการ reconcile ต่อวัน/warehouse

### Filters

- date range
- warehouse
- product group
- customer
- item
- result_code
- severity
- status

### Columns

| Column | Description |
|---|---|
| business_date | วันที่ |
| warehouse | คลัง |
| location | location |
| customer | ลูกค้า |
| item | สินค้า |
| year | ปี |
| lot | lot |
| result_code | ผลตรวจ |
| severity | ระดับความสำคัญ |
| confidence | ความมั่นใจ |
| field_qty | qty หน้างาน |
| system_qty | qty ERP |
| count_qty | qty นับจริง |
| expected_qty | qty ที่ระบบคำนวณ |
| diff_qty | ส่วนต่าง |
| explanation | คำอธิบาย |
| suggested_action | action ที่ควรทำ |
| issue_status | status issue |

### Export Formats

- XLSX
- CSV
- PDF/Print View optional

---

## 10.2 Issue Aging Report

### Purpose

ดู issue ที่ค้างและใครรับผิดชอบ

### Columns

- issue_id
- issue_type
- severity
- owner_role
- assigned_to
- business_date
- warehouse
- location
- created_at
- age_hours/days
- due_at
- status
- suggested_action

### KPI Summary

- total open
- over SLA
- by owner_role
- by severity

---

## 10.3 Pending Inbound Report

### Purpose

ดูของเข้าที่ข้อมูลยังไม่ครบและต้อง confirm

### Columns

- pending_inbound_id
- business_date
- warehouse
- location
- qty
- uom
- text_on_label
- parsed_customer_alias
- parsed_item_alias
- parsed_year
- status
- candidate_count
- best_match_score
- reported_by
- created_at

### Actions

- open pending detail
- confirm candidate
- create alias review
- reject with reason

---

## 10.4 Alias Review Report

### Purpose

ดู alias pending/mapped เพื่อคุม data quality

### Columns

- alias_type
- raw_text
- normalized_text
- suggested_master
- confidence
- occurrence_count
- source_batch
- status
- reviewed_by
- reviewed_at

### Summary

- pending by alias type
- low confidence count
- top unknown raw text
- mapped today

---

## 10.5 REJ Report

### Purpose

ติดตามของเสีย/เสียหายและ ERP posting

### Columns

- rej_id
- business_date
- warehouse
- from_location
- rej_location
- customer
- item
- year
- lot
- qty
- uom
- reason
- status
- system_post_ref
- issue_id
- aging
- owner_role

### Important Filters

- status = WAIT_SYSTEM_POST
- issue_type = REJ_NOT_POSTED
- date range
- warehouse

---

## 10.6 ERP Correction Template

### Purpose

ส่งรายการที่ ERP ต้องตรวจ/post/adjust

### Primary User

ERP Admin

### Source

issues / recon_results ที่ owner_role = ERP_ADMIN หรือ export selected rows

### Columns Recommended

- issue_id
- business_date
- warehouse
- location
- customer
- item
- year
- lot
- movement_type
- field_qty
- system_qty
- diff_qty
- uom
- suggested_action
- source_ref
- note

### Security

- ต้องมี CAN_EXPORT_ERP_TEMPLATE
- ต้องมี export log
- severity audit = CRITICAL ถ้าเป็น official correction template

---

## 10.7 Export Log Report

### Purpose

ตรวจว่าใคร export อะไร เมื่อไหร่

### Columns

- export_id
- export_type
- file_format
- filters
- row_count
- status
- exported_by
- exported_at
- downloaded_at optional
- error_message

---

## 10.8 Retention Report

### Purpose

ตรวจ cleanup 45 วัน

### Columns

- retention_job_id
- cutoff_date
- started_at
- finished_at
- status
- table_name
- deleted_rows
- skipped_rows
- skipped_reason
- message

Summary:

- total deleted
- total skipped open issue
- failed tables
- last successful cleanup

---

## 10.9 User Approval Report

### Purpose

ตรวจประวัติ user สมัคร/อนุมัติ/reject

### Columns

- request_id
- username
- full_name
- requested_role
- requested_warehouse
- requested_at
- status
- approved_by
- approved_at
- rejected_by
- rejected_at
- rejection_reason

---

## 10.10 Audit Log Report

### Purpose

ดู action สำคัญของระบบ

### Filters

- date range
- actor
- action
- entity_type
- entity_id
- severity

### Columns

- audit_id
- created_at
- actor
- action
- entity_type
- entity_id
- reason
- severity
- old_value summary
- new_value summary

---

# 11. KPI Formula Reference

## 11.1 Core Formulas

```text
Total Reconciled = count(recon_results in filter)
Matched Count = count(result_code = MATCHED)
Non-Matched Count = Total Reconciled - Matched Count
Match Rate = Matched Count / Total Reconciled * 100
Open Issues = count(issues where status not in CLOSED/RESOLVED)
Critical Issues = count(open issues where severity = CRITICAL)
Unknown Alias = count(alias_review_queue where status = PENDING)
Pending Inbound = count(pending_inbounds where status not in CONFIRMED/REJECTED/CLOSED)
REJ Not Posted = count(open issues where issue_type = REJ_NOT_POSTED)
System Missing = count(open issues where issue_type = SYSTEM_MISSING)
QTY Diff = count(open issues where issue_type = QTY_DIFF)
Location Diff = count(open issues where issue_type = LOCATION_DIFF)
```

---

## 11.2 Close Readiness Formula

```text
If open_critical_issues > 0 → NOT_READY
Else if missing_anchor critical → NOT_READY
Else if pending_inbound affects stock and policy block → NOT_READY
Else if REJ_NOT_POSTED critical → NOT_READY
Else if pending_alias > 0 or missing_inout partial → READY_WITH_WARNING
Else READY
```

Need configuration:

- pending inbound block/warn policy
- missing inout block/warn policy
- REJ aging threshold
- critical severity threshold

---

## 11.3 Coverage Status Formula

```text
Coverage OK if:
- anchor exists or approved estimate exists
- in-out present for required dates
- no critical unknown alias
- no blocking pending inbound
- no blocking REJ pending

Warning if:
- partial in-out missing but replay still possible
- pending alias not blocking
- count missing for moving location

Critical if:
- no anchor
- in-out missing and movement expected
- unknown alias blocks normalize
- pending inbound unresolved with stock impact
```

---

## 11.4 Issue Aging Formula

```text
Issue Age = now - issue.created_at
SLA Breach = issue.due_at is not null and now > issue.due_at
```

If due_at not set:

```text
Use default SLA by severity
```

---

# 12. Dashboard Drilldown Rules

ทุก card ต้องกด drilldown ได้

| Card | Drilldown |
|---|---|
| Critical Issue | Issue Center severity=CRITICAL status=open |
| Unknown Alias | Alias Review status=PENDING |
| Pending Inbound | Pending Inbound Review open |
| REJ Not Posted | Issue Center issue_type=REJ_NOT_POSTED |
| System Missing | Issue Center issue_type=SYSTEM_MISSING |
| QTY Diff | Issue Center issue_type=QTY_DIFF |
| Location Diff | Issue Center issue_type=LOCATION_DIFF |
| Data Coverage | Coverage Detail / Backfill Wizard |
| Last Import | Import Batch Detail |
| Backfill Status | Backfill Job Detail |
| Export Pending | Export Center status pending |
| Retention Status | Retention Log |
| DB Usage | System Health |

---

# 13. Dashboard UX Rules

## 13.1 Visual Priority

Dashboard order:

```text
Critical Blocker
→ Close Readiness
→ Pending Action
→ Data Coverage
→ Import/Backfill
→ Maintenance
→ Normal Summary
```

---

## 13.2 Color Rules

| State | Color |
|---|---|
| Critical / Not Ready | Red |
| Warning / Pending | Yellow/Orange |
| OK / Ready / Matched | Green |
| In Progress | Blue |
| No Activity / Closed | Gray |

Do not use red for normal negative-looking numbers unless it requires action

---

## 13.3 Empty State

If no issue:

```text
ยังไม่มี issue ที่ต้องแก้ในช่วงวันที่เลือก
```

If no pending alias:

```text
ไม่มี alias ที่รอ review
```

If no import:

```text
ยังไม่พบไฟล์ที่ upload สำหรับช่วงวันที่เลือก
```

---

## 13.4 Stale Data Warning

ถ้า dashboard summary ไม่ได้ update หลัง import/replay/reconciliation ล่าสุด ต้องแสดง warning:

```text
ข้อมูล Dashboard อาจไม่ใช่ล่าสุด กรุณา run reconciliation หรือ refresh summary
```

---

# 14. Report Export Rules

## 14.1 Export Scope Required

ทุก report export ต้องมี:

- filters_json หรือ selected_ids
- row_count preview
- export_type
- exported_by
- exported_at

---

## 14.2 Export Preview

ก่อน export ควรแสดง:

- report type
- filters
- row count
- columns
- file format

---

## 14.3 Export Log Required

ทุก export ต้องสร้าง export_jobs/export_logs

---

## 14.4 Official ERP Template Rules

ERP template ต้อง:

- ตรวจ permission เฉพาะ
- มี selected rows หรือ filter ชัดเจน
- มี audit severity HIGH/CRITICAL
- ไม่รวม row ที่ status ไม่พร้อม ถ้า policy ไม่อนุญาต

---

# 15. Dashboard Data Sources

## 15.1 Recommended Summary Tables

| Data Source | Used For |
|---|---|
| daily_recon_summary | matched/issue counts |
| daily_issue_summary | issue by status/owner |
| issues | open issue detail counts |
| recon_results | result code counts |
| alias_review_queue | pending alias |
| pending_inbounds | pending inbound |
| rej_notices / issues | REJ status |
| import_batches / import_summary | import status |
| stock_replay_jobs | replay status |
| reconciliation_jobs | recon job status |
| export_jobs | export status |
| retention_jobs / retention_logs | cleanup status |
| system_health_checks | DB/job health |
| daily_close_status | close readiness |

---

## 15.2 Summary Refresh Triggers

Summary ควร rebuild เมื่อ:

- import confirmed
- alias mapped and affected rows reprocessed
- stock replay completed
- reconciliation completed
- issue status changed
- pending inbound confirmed
- REJ status changed
- export job created/finished
- retention job completed

---

# 16. KPI Data Quality Rules

## 16.1 KPI Confidence

KPI ที่มาจาก replay/reconciliation ต้องแสดง confidence ถ้า coverage ไม่ครบ

Example:

```text
Match Rate: 92%
Confidence: MEDIUM เพราะ In-Out วันที่ 2026-05-02 ยังขาด
```

---

## 16.2 Do Not Mix UOM Incorrectly

ถ้ารายงานรวม diff qty ต้องแยกตาม UOM หรือ convert เป็น base UOM ก่อน

ห้ามรวม:

```text
100 BAG + 2 TON = 102
```

---

## 16.3 Product Group Context

บาง KPI ต้องแยก product group เพราะ logic/alias ต่างกัน

ตัวอย่าง:

- Sugar
- Wood
- Flour
- Molasses
- Bulk

---

# 17. KPI Acceptance Criteria

Dashboard/KPI/Report ถือว่า complete ถ้า:

- [ ] Control Center แสดง critical/pending/action-first
- [ ] ทุก KPI มี formula และ decision purpose
- [ ] ทุก card drilldown ได้
- [ ] KPI มี date/warehouse context
- [ ] Dashboard อ่าน summary/result ไม่ query raw full history
- [ ] Unknown Alias / Pending Inbound / REJ Not Posted แสดงชัด
- [ ] Close Readiness บอก READY/WARNING/NOT_READY ได้
- [ ] Data Coverage แสดง anchor/in-out/alias/pending/rej/count status ได้
- [ ] Issue Dashboard แสดง aging/owner/severity ได้
- [ ] Report export มี preview/log/filter
- [ ] ERP Template export มี permission และ audit
- [ ] Retention/Health status แสดงได้
- [ ] KPI ที่ coverage ต่ำต้องแสดง confidence/warning
- [ ] ไม่รวม UOM ผิดใน aggregate qty

---

# 18. Final Dashboard Statement

```text
Dashboard, KPI และ Report ของ Warehouse Reconciliation ERP Platform ต้องทำหน้าที่เป็น Decision Control Center ที่แสดง exception, blocker, pending action, coverage, close readiness และ system health ก่อนข้อมูลปกติ โดยทุก KPI ต้องมีสูตร แหล่งข้อมูล context drilldown และ next action ชัดเจน เพื่อให้ Admin/Supervisor/ERP Admin ไม่ต้องไล่เช็ค Excel เอง แต่ใช้ Dashboard นำทางไปแก้ปัญหาที่สำคัญที่สุดก่อน
```

