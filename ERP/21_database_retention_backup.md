# 21_database_retention_backup.md

# Database, Retention & Backup Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Database Operation / Retention Policy / Backup & Archive Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / Database Admin / System Owner Use  
**Primary Use:** ใช้กำหนดแนวทางจัดการฐานข้อมูล, retention 45 วัน, archive/summary, backup, health check, storage control, delete safety และ recovery เพื่อให้ระบบเร็ว มั่นคง ใช้ free tier ได้ดี และไม่ลบข้อมูลสำคัญที่ยังต้อง audit/แก้ issue

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Database, Retention และ Backup Strategy สำหรับ Warehouse Reconciliation ERP Platform

ระบบนี้มีข้อมูลหลายชนิดที่มีอายุการใช้งานต่างกัน:

```text
Raw Import Rows          = ใช้ตรวจ/normalize ช่วงแรก แต่เก็บยาวทำให้ DB โต
Normalized Records       = ใช้ replay/reconciliation
Movement Ledger          = ใช้คำนวณ stock
Daily Stock Positions    = ใช้ dashboard/report
Recon Results            = ใช้อธิบายความตรง/ไม่ตรง
Issues                   = ใช้ action และ audit
Audit Logs               = ใช้ตรวจย้อนหลัง
Export Logs              = ใช้ governance
Notification Logs        = ใช้ตรวจการแจ้งเตือน
Retention Logs           = ใช้พิสูจน์ว่าระบบลบอะไรและข้ามอะไร
```

เป้าหมายคือ:

1. ให้ระบบเร็วแม้ข้อมูลสะสม
2. ใช้ storage อย่างคุมได้ โดยเฉพาะถ้าใช้ free tier
3. ลบ/archive raw data เกิน 45 วันได้อย่างปลอดภัย
4. ไม่ลบข้อมูลที่ยังมี open issue หรือ pending action
5. มี summary/archive เพื่อดูย้อนหลังได้โดยไม่ต้องเก็บ raw ทั้งหมด
6. มี audit/retention log ทุกครั้งที่ cleanup
7. มี backup/export สำหรับกู้คืนหรือย้ายระบบ
8. มี health check ให้ Admin ไม่ต้องคอยตรวจเองทุกวัน

หลักสำคัญ:

```text
ลบข้อมูลได้เฉพาะเมื่อระบบยังอธิบายผลลัพธ์ย้อนหลังได้ และไม่มี open issue ที่ต้องใช้ข้อมูลนั้นเป็น evidence
```

---

## 2. Database Strategy Philosophy

## 2.1 Keep Active Data Small

ข้อมูลที่ใช้ทำงานประจำวันควรอยู่ใน active database เฉพาะช่วงที่จำเป็น เช่น 45 วัน

Active database ควรเก็บ:

- งานที่ยัง open
- raw/staging ล่าสุด
- ledger/replay ล่าสุด
- issue ที่ยังไม่ปิด
- summary ที่ใช้ dashboard
- audit ที่สำคัญ

ไม่ควรเก็บ raw import rows หลายเดือนโดยไม่มี archive เพราะจะทำให้:

- dashboard ช้า
- import/reconciliation ช้า
- free tier เต็ม
- backup ใหญ่เกินจำเป็น
- retention ยากขึ้น

---

## 2.2 Summary Before Delete

ก่อนลบ raw/detail ต้องมั่นใจว่ามี summary ที่จำเป็นแล้ว

ตัวอย่าง:

```text
import_raw_rows เกิน 45 วัน
→ ต้องมี import summary, normalized result, recon result, issue status, audit log แล้ว
→ ถ้าไม่มี open issue อ้างอิง จึงลบ/archive raw ได้
```

---

## 2.3 Do Not Delete Evidence for Open Work

ข้อมูลที่ยังใช้แก้ปัญหา ห้ามลบ แม้เกิน 45 วัน

ห้ามลบถ้าเกี่ยวกับ:

- open issue
- pending inbound unresolved
- REJ_NOT_POSTED unresolved
- SYSTEM_MISSING unresolved
- pending export
- pending approval
- correction/investigation
- locked day dispute

---

## 2.4 Retention Must Be Explainable

ทุก retention job ต้องตอบได้ว่า:

```text
ลบ table ไหน
ลบกี่ rows
ข้ามกี่ rows
ข้ามเพราะอะไร
cutoff date คืออะไร
ใคร/ระบบ run
สำเร็จหรือ fail
```

---

## 2.5 Backup Is Not the Same as Retention

Retention = ลด active data และลบ/archive ตาม policy  
Backup = สำรองเพื่อกู้คืน/ย้ายระบบ

ต้องมีทั้งสองแนวคิด ไม่ใช่ลบข้อมูลแล้วไม่มีทางย้อนกลับเลย

---

# 3. Database Platform Options

## 3.1 Google Sheets / Apps Script MVP

เหมาะเมื่อ:

- ข้อมูลยังไม่ใหญ่มาก
- ต้อง prototype เร็ว
- user คุ้นกับ sheet
- ยังอยู่ช่วง data profiling

ข้อดี:

- เริ่มเร็ว
- ฟรีหรือค่าใช้จ่ายต่ำ
- export/manual review ง่าย

ข้อเสีย:

- performance จำกัด
- permission/security จำกัด
- audit/transaction จำกัด
- schema คุมยาก
- concurrent user อาจมีปัญหา
- long-running job อาจ timeout

คำแนะนำ:

```text
ใช้ได้เฉพาะ MVP/prototype ที่ข้อมูลไม่ใหญ่ และต้องแยก summary sheets เพื่อลดการคำนวณจาก raw
```

---

## 3.2 Supabase / Postgres

เหมาะเมื่อ:

- ต้องทำ web app จริง
- ต้องมี auth/role/scope
- ต้อง query/filter เยอะ
- ต้องใช้ audit/retention/job table
- ต้อง scale จาก MVP ไป production

ข้อดี:

- relational data model ดี
- index ได้
- RLS ได้
- storage ได้
- scheduled job/edge function ได้ตาม architecture
- เหมาะกับ ledger/reconciliation มากกว่า sheet

ข้อควรระวัง:

- free tier มี storage/compute limit
- ต้องออกแบบ retention ตั้งแต่แรก
- RLS policy ต้องระวัง
- service key ต้องอยู่ server-side เท่านั้น

คำแนะนำ:

```text
ถ้าต้องสร้างจริงและอยากให้ stable ระยะยาว ให้ใช้ Supabase/Postgres ตั้งแต่ต้น พร้อม retention 45 วัน
```

---

## 3.3 MySQL / Managed SQL

เหมาะเมื่อ:

- องค์กรมี infra อยู่แล้ว
- ทีม dev คุ้นเคย
- ต้องการควบคุม database เอง

ข้อดี:

- stable
- index/query ดี
- backup/migration คุมเองได้

ข้อเสีย:

- ต้องมี backend/hosting/backup เองมากกว่า
- ไม่มี built-in auth/storage เหมือน Supabase

---

# 4. Recommended MVP Database Approach

## 4.1 Recommended for Real Build

```text
Supabase/Postgres + Server-side API + Private Storage + Scheduled Retention Job
```

เหตุผล:

- data model ของระบบเป็น relational มาก
- movement ledger/replay/reconciliation ต้อง query หลาย table
- permission + warehouse scope สำคัญ
- audit/retention ต้อง reliable
- dashboard ต้องเร็วด้วย summary table/index

---

## 4.2 Recommended if Need Fast Prototype

```text
Google Sheets as temporary database + Apps Script service + strict sheet schema + summary sheets
```

แต่ต้องยอมรับว่า:

- ไม่ควรทำ reconciliation logic ใหญ่บน sheet formula
- ต้องมี migration path ไป database จริง
- ต้องไม่ให้ user แก้ schema/raw sheet โดยตรงหลังระบบเริ่มนิ่ง

---

# 5. Data Lifecycle Model

ข้อมูลแต่ละชนิดควรมี lifecycle:

```text
Created
→ Active / Processing
→ Confirmed / Used
→ Closed / Summarized
→ Retention Eligible
→ Archived / Deleted
→ Logged
```

---

## 5.1 Active Data

คือข้อมูลที่ระบบยังใช้ในการทำงานประจำวัน

ตัวอย่าง:

- import batch ล่าสุด
- raw rows ที่ยัง normalize ไม่เสร็จ
- unknown alias pending
- pending inbound
- open issue
- last 45 days movement ledger
- last 45 days recon result
- active daily stock positions

---

## 5.2 Closed Data

คือข้อมูลที่งานจบแล้ว แต่ยังอยู่ใน active database ภายใน retention period

ตัวอย่าง:

- matched recon result
- closed issue
- confirmed import batch
- sent notification logs
- completed export jobs

---

## 5.3 Archive Data

คือข้อมูลที่ไม่ต้องใช้ประจำ แต่ควรเก็บ summary/หลักฐาน

ตัวอย่าง:

- daily summary
- closed issue summary
- import batch summary
- retention summary
- export log
- critical audit log

---

## 5.4 Delete-Eligible Data

ข้อมูลที่ลบได้เมื่อ:

- เกิน cutoff date
- normalized/confirmed แล้ว
- ไม่มี open issue อ้างอิง
- ไม่มี pending action
- มี summary/archive แล้ว
- retention dry run ผ่าน

---

# 6. Retention Policy Overview

## 6.1 Default Retention Window

```text
Active raw/detail data retention = 45 days
cutoff_date = today - 45 days
```

เหตุผล:

- คุม storage
- เหมาะกับ free tier/MVP
- ยังพอครอบคลุมช่วงแก้ปัญหาหน้างานย้อนหลัง
- ลดภาระ dashboard/query

---

## 6.2 Tables That Can Be Cleaned After 45 Days

ลบ/archive ได้ถ้าปลอดภัย:

- import_raw_rows ที่ normalized/closed แล้ว
- import_error_logs ที่ resolved แล้ว
- staging normalized records ที่ confirmed แล้ว
- closed/matched field movement detail
- closed/matched system in-out detail
- closed physical count detail ถ้า issue ปิดแล้ว
- closed recon_results ที่ไม่มี open issue
- sent/skipped notification_logs ที่ไม่ critical
- temporary job logs ที่สำเร็จและมี summary แล้ว

---

## 6.3 Tables That Should Not Be Blindly Deleted

ห้ามลบแบบ blind delete:

- users
- roles
- permissions
- user access audit
- active master data
- active alias tables
- open issues
- issue_comments linked to open/critical issue
- audit_logs critical
- export_logs
- retention_logs
- daily_close_status
- daily summaries
- system health summary

---

## 6.4 Tables to Keep Longer Than 45 Days

ควรเก็บนานกว่า raw data:

| Table / Data | Recommended Retention |
|---|---|
| critical audit logs | 6–12 months or longer |
| export logs | 6–12 months |
| retention logs | 12 months |
| daily close status | 12 months |
| daily recon summary | 6–12 months |
| daily issue summary | 6–12 months |
| closed issues summary | 6–12 months |
| master/alias history | until superseded with audit |

สำหรับ MVP ถ้ายังจำกัด storage ให้เก็บ summary แบบ compact แทน raw detail

---

# 7. Retention Safety Rules

## RET-SAFE-001 — Open Issue Protection

ห้ามลบ source data ที่ issue ยัง open

ต้องตรวจ:

```text
issues.status not in CLOSED/RESOLVED
or issue_type unresolved
or result_id/source_ref linked to issue
```

ถ้าพบ ให้ skip และเขียน retention log:

```text
skipped_reason = OPEN_ISSUE
```

---

## RET-SAFE-002 — Pending Inbound Protection

ห้ามลบ pending_inbounds หรือ source candidate ถ้า status ยังไม่จบ

Block statuses:

```text
PENDING_INBOUND_VERIFY
WAIT_SYSTEM_INOUT
CANDIDATE_FOUND
NEED_INVESTIGATION
```

---

## RET-SAFE-003 — REJ Protection

ห้ามลบ REJ ที่ยังไม่ post/ยังเป็น issue

Block statuses:

```text
WAIT_SYSTEM_POST
REJ_NOT_POSTED
REJ_QTY_DIFF
INVESTIGATING
```

---

## RET-SAFE-004 — Alias Pending Protection

ห้ามลบ raw/source rows ที่ยังต้องใช้ review alias ถ้า alias_review_queue ยัง pending และ source ยังจำเป็นต่อการตัดสินใจ

ถ้าต้องลบ raw เพราะ storage จำกัด ต้องเก็บ compact evidence:

- raw_text
- source_batch_id
- row_no
- alias_type
- occurrence_count
- sample context

---

## RET-SAFE-005 — Locked Day Protection

ถ้าวันนั้น locked แล้ว ห้ามลบข้อมูล detail ที่ยังต้องใช้ตรวจ correction/dispute โดยไม่มี daily close summary และ audit trail

---

## RET-SAFE-006 — Export Pending Protection

ห้ามลบ data ที่ export job ยัง running/queued หรือ official export ยังไม่เสร็จ

---

# 8. Retention Job Design

## 8.1 Job Schedule

Recommended:

```text
Nightly retention job: 02:00 local time
```

เหตุผล:

- ลดผลกระทบเวลาทำงาน
- มีเวลาทำ summary/health check
- แจ้ง Admin ตอนเช้าได้

---

## 8.2 Job Modes

Retention ต้องมี 2 mode:

```text
DRY_RUN  = preview ว่าจะลบ/ข้ามอะไร แต่ไม่ลบจริง
EXECUTE  = ลบ/archive จริงตาม policy
```

Manual run ควรเริ่มจาก DRY_RUN ก่อนเสมอ

---

## 8.3 Retention Job Flow

```text
Start retention job
→ calculate cutoff_date = today - 45 days
→ run health check
→ identify candidate records by table
→ check open issue / pending action protection
→ create summary/archive if required
→ if DRY_RUN: return preview only
→ if EXECUTE: delete/archive safe records
→ write retention_logs per table
→ write audit RETENTION_DELETE
→ update system_health_checks
→ send notification summary
```

---

## 8.4 Candidate Selection Logic

Candidate records must meet:

```text
record_date < cutoff_date
AND status in closed/safe statuses
AND no open issue link
AND no pending export link
AND archive/summary exists if required
```

---

# 9. Retention Tables

## 9.1 retention_jobs

| Field | Type | Required | Description |
|---|---|---:|---|
| retention_job_id | uuid/text | Yes | primary key |
| cutoff_date | date | Yes | today - 45 days |
| run_mode | text | Yes | DRY_RUN / EXECUTE |
| triggered_by | text | Yes | SYSTEM or user_id |
| trigger_type | text | Yes | SCHEDULED / MANUAL |
| status | text | Yes | RUNNING / SUCCESS / WARNING / FAILED |
| started_at | datetime | Yes | start time |
| finished_at | datetime | No | end time |
| deleted_total_rows | number | Yes | total deleted |
| archived_total_rows | number | No | total archived |
| skipped_total_rows | number | Yes | skipped rows |
| skipped_open_issues | number | Yes | skipped because open issues |
| warning_count | number | Yes | warnings |
| error_count | number | Yes | errors |
| message | text | No | summary |

---

## 9.2 retention_logs

| Field | Type | Required | Description |
|---|---|---:|---|
| retention_log_id | uuid/text | Yes | primary key |
| retention_job_id | uuid/text | Yes | FK retention_jobs |
| table_name | text | Yes | affected table |
| cutoff_date | date | Yes | cutoff used |
| candidate_rows | number | Yes | rows found |
| archived_rows | number | No | archived rows |
| deleted_rows | number | Yes | deleted rows |
| skipped_rows | number | Yes | skipped rows |
| skipped_reason | text | No | OPEN_ISSUE / PENDING_EXPORT etc |
| status | text | Yes | SUCCESS / WARNING / FAILED |
| message | text | No | detail |
| created_at | datetime | Yes | timestamp |

---

## 9.3 archive_summaries

Optional but recommended if archive summary stored inside DB

| Field | Type | Description |
|---|---|
| archive_summary_id | uuid/text | primary key |
| archive_type | text | IMPORT / RECON / ISSUE / STOCK / RETENTION |
| date_from | date | range start |
| date_to | date | range end |
| warehouse_id | uuid/text | optional |
| summary_json | json/text | compact summary |
| source_tables_json | json/text | tables included |
| created_by | text | system/user |
| created_at | datetime | timestamp |

---

# 10. Table-Level Retention Policy

## 10.1 Import Tables

### import_batches

Keep metadata longer than raw rows

Policy:

```text
Keep batch metadata at least 6–12 months
Do not delete if linked to open issue or active snapshot
```

### import_raw_rows

Policy:

```text
Delete/archive after 45 days if:
- batch confirmed/closed
- rows normalized or error resolved
- no open issue/pending alias needs raw row
```

### import_error_logs

Policy:

```text
Delete/archive after 45 days if resolved
Keep if error caused open issue or data dispute
```

---

## 10.2 Alias Tables

### alias_review_queue

Policy:

```text
Do not delete pending queue
Closed/mapped queue can archive after 45 days if alias record and audit exist
```

### alias master tables

Policy:

```text
Do not delete active alias
Use inactive/superseded status instead of hard delete
```

---

## 10.3 Inquiry Snapshot Tables

### inquiry_snapshots

Policy:

```text
Keep metadata and active snapshot references
Archive old/replaced snapshot detail after 45 days if no open issue and summary exists
```

### inquiry_snapshot_lines

Policy:

```text
Archive/delete detail after 45 days if:
- daily_stock_positions generated
- no open issue links
- snapshot not needed as active anchor for later replay
```

Important:

```text
ห้ามลบ snapshot line ที่ยังเป็น anchor ของ replay range ที่ยัง active/open issue
```

---

## 10.4 System In-Out Tables

### system_inout_records

Policy:

```text
Archive/delete detail after 45 days if:
- movement_ledger/daily stock/recon results exist
- no open issue links
- no pending ERP export/correction
```

### movement_ledger

Policy:

```text
Keep active 45 days
Archive old closed ledger if daily_stock_positions and recon summaries exist
Do not delete ledger linked to open issue
```

---

## 10.5 Field Operation Tables

### field_movements

Policy:

```text
Archive/delete after 45 days if matched/closed and no open issue
```

### pending_inbounds

Policy:

```text
Do not delete unresolved pending inbound
Closed/confirmed old pending can archive after summary and audit exist
```

### physical_counts

Policy:

```text
Archive old counts if issue closed and daily/recon summary exists
Keep if count is evidence for open issue
```

### rej_notices

Policy:

```text
Do not delete REJ unresolved
Archive closed REJ after ERP post confirmed and issue closed
```

---

## 10.6 Reconciliation / Issue Tables

### recon_results

Policy:

```text
Matched/closed results can archive after 45 days
Non-matched linked to open issue must remain
Closed issue results can be compacted into issue summary
```

### issues

Policy:

```text
Open issues: never delete
Closed issues: keep summary 6–12 months, detail archive after policy
```

### issue_comments

Policy:

```text
Keep comments while issue open
Closed issue comments can archive with issue detail
```

---

## 10.7 Export / Notification / Audit Tables

### export_jobs / export_logs

Policy:

```text
Keep longer than 45 days, especially official ERP exports
Do not delete without policy approval
```

### notification_logs

Policy:

```text
Sent/skipped low priority logs can delete after 45 days
Failed/critical logs keep longer or until resolved
```

### audit_logs

Policy:

```text
Critical audit logs keep longer than 45 days
Do not delete permission/import/lock/export/retention critical logs in MVP
```

---

# 11. Archive Strategy

## 11.1 Archive Levels

### Level 1 — Summary Archive

เก็บเฉพาะ aggregated summary เช่น:

- row counts
- matched/issue counts
- open/closed status
- deleted/skipped count

เหมาะสำหรับ dashboard/report ย้อนหลัง

### Level 2 — Compact Evidence Archive

เก็บข้อมูลสำคัญที่พออธิบาย issue ได้ เช่น:

- result code
- customer/item/location/year/lot
- qty diff
- explanation
- source batch id
- source row no

เหมาะสำหรับ closed issue/report

### Level 3 — Full Raw Backup Export

เก็บ raw export นอก active database เช่น CSV/XLSX/JSON ใน private storage

เหมาะสำหรับ:

- ก่อน delete รอบใหญ่
- migration
- compliance
- manual investigation

---

## 11.2 Recommended MVP Archive

สำหรับ MVP ให้ทำอย่างน้อย:

```text
1. Daily recon summary
2. Daily issue summary
3. Import batch summary
4. Retention log per table
5. Export log
6. Critical audit log
```

ถ้า storage พอ ให้เพิ่ม compact evidence archive สำหรับ closed issues

---

# 12. Backup Strategy

## 12.1 Backup Types

| Backup Type | Purpose | Frequency |
|---|---|---|
| Config Backup | master/alias/permission/config | before major change / weekly |
| Data Summary Backup | daily summaries/issues/export logs | daily/weekly |
| Full DB Backup | recovery/migration | weekly/monthly depending platform |
| Pre-Retention Backup | safety before delete/archive | before retention execute if needed |
| Manual Export Backup | user-requested archive | on demand |

---

## 12.2 Backup Scope

Backup ควรครอบคลุม:

- master data
- alias tables
- user roles/permissions excluding password secrets if export is shared
- import batch metadata
- inquiry snapshot metadata
- movement/replay summary
- recon summary
- issue summary/detail as policy
- audit logs critical
- export logs
- retention logs

---

## 12.3 Sensitive Data in Backup

Backup ที่ export/share ต้องไม่เปิดเผย:

- password hash ถ้าไม่จำเป็น
- salt ถ้าไม่จำเป็น
- session token
- notification token
- service role key
- API secrets

ให้ใช้ masked/omitted fields

---

## 12.4 Backup File Naming

Recommended naming:

```text
backup_<type>_<date>_<warehouse_or_all>_<version>.zip
```

Examples:

```text
backup_config_2026-05-18_all_v1.zip
backup_recon_summary_2026-05-18_WH_W8_v1.xlsx
backup_pre_retention_2026-05-18_all_v1.zip
```

---

# 13. Restore / Recovery Strategy

## 13.1 Recovery Scenarios

ต้องคิด recovery สำหรับ:

- import ผิด batch
- alias map ผิด
- retention ลบผิด
- export file หาย
- daily close/lock ผิด
- database corruption/migration issue

---

## 13.2 Restore Principles

```text
Restore ต้องไม่ overwrite ข้อมูลสำคัญเงียบ ๆ
ต้องสร้าง correction/audit เสมอ
```

ตัวอย่าง:

- alias map ผิด → deactivate alias + create corrected alias + audit
- import ผิด → cancel/supersede batch + replay/reconcile ใหม่
- retention ลบผิด → restore from backup + audit RETENTION_RESTORE

---

## 13.3 Minimum Restore Tools

MVP ควรมีอย่างน้อย:

- Export backup download
- Import batch cancel/supersede
- Alias correction flow
- Re-run stock replay
- Re-run reconciliation
- View audit trail

---

# 14. Health Check Specification

## 14.1 Health Check Purpose

Health Check ทำให้ Admin รู้สถานะระบบโดยไม่ต้องเข้าไปดู database เอง

ต้องตอบว่า:

- storage ใกล้เต็มไหม
- retention ทำงานล่าสุดเมื่อไหร่
- import ล่าสุดเมื่อไหร่
- replay/reconciliation ล่าสุดสำเร็จไหม
- มี failed jobs ไหม
- มี pending alias/pending inbound เยอะไหม
- มี open critical issue ไหม

---

## 14.2 system_health_checks Table

| Field | Type | Description |
|---|---|
| health_check_id | uuid/text | primary key |
| checked_at | datetime | time |
| db_usage_mb | number | database usage |
| db_usage_percent | number | usage percentage if available |
| storage_usage_mb | number | file storage usage |
| open_critical_issues | number | critical open |
| pending_alias_count | number | pending alias |
| pending_inbound_count | number | pending inbound |
| failed_job_count | number | recent failed jobs |
| last_import_at | datetime | latest import |
| last_replay_at | datetime | latest replay |
| last_recon_at | datetime | latest reconciliation |
| last_retention_at | datetime | latest cleanup |
| last_backup_at | datetime | latest backup optional |
| status | text | OK / WARNING / CRITICAL |
| details_json | json/text | details |

---

## 14.3 Health Status Rules

### OK

```text
db_usage < 70%
no failed critical jobs
last retention within expected schedule
open critical issue within acceptable threshold
```

### WARNING

```text
db_usage 70–85%
retention delayed
pending alias high
pending inbound high
some failed non-critical jobs
```

### CRITICAL

```text
db_usage > 85%
retention failed
replay/reconciliation failed recently
open critical issue high
backup missing if required
```

---

# 15. Storage Strategy for Files

## 15.1 File Types

ระบบอาจเก็บไฟล์:

- uploaded import source file optional
- photo evidence
- export reports
- backup zip/xlsx/csv

---

## 15.2 Storage Rules

- ใช้ private storage สำหรับข้อมูล sensitive
- download link ควรมี expiry ถ้า platform support
- ทุก export/download ต้อง log
- photo evidence ต้องดูได้เฉพาะ user ที่มี permission/scope
- ห้ามเก็บ token/secret ใน file metadata แบบเปิดเผย

---

## 15.3 Storage Retention

| File Type | Retention |
|---|---|
| raw uploaded source file | delete/archive after 45 days if processed and safe |
| photo evidence linked to open issue | keep until issue closed + policy |
| export report | keep per export policy, at least log remains |
| backup file | keep according backup schedule |

---

# 16. Indexing & Performance for Retention

## 16.1 Required Indexes

Retention และ dashboard จะเร็วขึ้นถ้ามี indexes:

```text
business_date
created_at
uploaded_at
status
warehouse_id
source_type + source_ref_id
result_id
issue_id
```

---

## 16.2 Table-Specific Indexes

### import_raw_rows

```text
(batch_id)
(batch_id, row_no)
(created_at)
```

### movement_ledger

```text
(business_date, warehouse_id)
(source_type, source_ref_id)
(status)
```

### recon_results

```text
(business_date, warehouse_id)
(result_code, status)
```

### issues

```text
(status, severity)
(business_date, warehouse_id)
(result_id)
```

### audit_logs

```text
(created_at)
(action)
(entity_type, entity_id)
(severity)
```

### retention_logs

```text
(retention_job_id)
(table_name, created_at)
```

---

# 17. Retention Dry Run UI

## 17.1 Dry Run Output

UI ต้องแสดง:

- cutoff date
- table name
- candidate rows
- rows to archive
- rows to delete
- skipped rows
- skipped reason
- estimated storage reduction optional
- risk warnings

---

## 17.2 Dry Run Example

| Table | Candidate | Delete | Archive | Skip | Reason |
|---|---:|---:|---:|---:|---|
| import_raw_rows | 12,000 | 10,800 | 0 | 1,200 | OPEN_ISSUE |
| notification_logs | 2,000 | 1,950 | 0 | 50 | FAILED_CRITICAL |
| recon_results | 5,000 | 4,300 | 700 | 0 | SUMMARY_ARCHIVE |

---

## 17.3 Execute Confirmation

ก่อน EXECUTE ต้อง confirm:

```text
Run Retention Cleanup จริง?
ระบบจะลบ/archive ข้อมูลที่ปลอดภัยตาม cutoff date
ข้อมูลที่ผูกกับ open issue, pending inbound, critical audit และ export log จะถูกข้าม
```

ต้องมี reason ถ้า manual execute

---

# 18. Notification for Retention & Health

## 18.1 Events

ควรมี notification events:

```text
RETENTION_COMPLETED
RETENTION_COMPLETED_WITH_WARNING
RETENTION_FAILED
DB_USAGE_WARNING
DB_USAGE_CRITICAL
BACKUP_COMPLETED
BACKUP_FAILED
HEALTH_CHECK_CRITICAL
```

---

## 18.2 Retention Summary Message

ตัวอย่าง:

```text
Retention Cleanup Completed
Cutoff: 2026-04-02
Deleted: 14,250 rows
Skipped: 1,230 rows
Skipped open issues: 18
Status: WARNING
```

---

# 19. Backup & Retention Acceptance Criteria

Database/Retention/Backup ถือว่า complete ถ้า:

- [ ] มี retention policy 45 วันสำหรับ raw/detail active data
- [ ] มี DRY_RUN mode
- [ ] มี EXECUTE mode
- [ ] Retention job มี status/log
- [ ] Retention log ระบุ table, deleted, skipped, reason ได้
- [ ] Open issue linked data ไม่ถูกลบ
- [ ] Pending inbound unresolved ไม่ถูกลบ
- [ ] REJ_NOT_POSTED unresolved ไม่ถูกลบ
- [ ] Critical audit log ไม่ถูกลบใน MVP
- [ ] Export log ไม่ถูกลบโดยไม่มี policy
- [ ] Summary/archive ถูกสร้างก่อน delete ตาม policy
- [ ] Health check แสดง DB usage, failed jobs, last retention ได้
- [ ] Admin เห็น retention status ใน Control Center
- [ ] Manual retention execute ต้อง confirm และ audit
- [ ] Retention failed ส่ง notification/log
- [ ] Backup/export ไม่เปิดเผย password/token/secret
- [ ] มีแนวทาง restore/correction ไม่ overwrite เงียบ ๆ

---

# 20. Final Database, Retention & Backup Statement

```text
Database, Retention และ Backup ของ Warehouse Reconciliation ERP Platform ต้องออกแบบให้ active database เล็กและเร็ว โดยเก็บ raw/detail data เท่าที่จำเป็นประมาณ 45 วัน ลบ/archive เฉพาะข้อมูลที่ closed และไม่มี open issue พร้อมสร้าง summary/audit/retention log ทุกครั้ง ห้ามลบ evidence ของ issue ที่ยังเปิดอยู่หรือ critical audit/export log และต้องมี health check/backup/recovery strategy เพื่อให้ระบบใช้ได้มั่นคงแม้อยู่บน free tier หรือ storage จำกัด
```

