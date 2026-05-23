# 16_error_empty_loading_state.md

# Error, Empty, Loading & Confirmation State Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** UI State / Error Handling / Empty State / Loading State / Confirmation Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / UX Designer / QA Use  
**Primary Use:** ใช้กำหนดมาตรฐานสถานะของทุกหน้าและทุก workflow เช่น loading, empty, error, warning, success, permission denied, confirmation, long-running job, retry และ stale data เพื่อให้ Web App ใช้งานจริงได้ ไม่งง และไม่ทำให้ user ตัดสินใจผิด

---

## 1. Purpose of This Document

เอกสารนี้กำหนดมาตรฐาน UI State ของ Warehouse Reconciliation ERP Platform

ระบบนี้มี workflow ที่ซับซ้อน เช่น:

```text
Upload file
→ Preview
→ Column mapping
→ Normalize
→ Alias review
→ Stock replay
→ Reconciliation
→ Issue
→ Export
→ Notification
→ Retention
```

ถ้าไม่มี state handling ที่ดี user จะไม่รู้ว่า:

- ระบบกำลังทำงานอยู่หรือค้าง
- ไม่มีข้อมูลจริง ๆ หรือ filter ผิด
- error เกิดจากอะไร
- ต้องแก้อะไรต่อ
- action กดไปแล้วสำเร็จไหม
- export พร้อม download หรือยัง
- notification ส่งแล้วหรือ failed
- retention ลบข้อมูลอะไรไปแล้ว

หลักสำคัญ:

```text
ทุกหน้าและทุก action ต้องมี state ที่อธิบายได้ ไม่ใช่แค่ขึ้น spinner หรือ error เฉย ๆ
```

---

## 2. State Design Philosophy

## 2.1 State Must Guide Action

ทุก state ต้องตอบได้ว่า:

```text
เกิดอะไรขึ้น?
กระทบอะไร?
user ต้องทำอะไรต่อ?
กดไปหน้าไหนได้?
```

ตัวอย่างที่ดี:

```text
Import ไม่สำเร็จ
ระบบพบว่า column Qty ยังไม่ได้ map
โปรดกลับไปที่ขั้น Column Mapping แล้วเลือก field qty
[กลับไป Mapping]
```

ตัวอย่างที่ไม่ดี:

```text
Error
```

---

## 2.2 Separate Empty From Error

ไม่มีข้อมูลกับโหลดข้อมูลไม่สำเร็จไม่ใช่เรื่องเดียวกัน

| State | Meaning |
|---|---|
| Empty | โหลดสำเร็จ แต่ไม่มีข้อมูลตามเงื่อนไข |
| Error | โหลดหรือประมวลผลไม่สำเร็จ |
| Permission Denied | user ไม่มีสิทธิ์ |
| Loading | กำลังโหลด/ประมวลผล |
| Stale | ข้อมูลเก่า ไม่ใช่ล่าสุด |

---

## 2.3 Long Job Needs Progress

งานหนัก เช่น import, replay, reconciliation, export, retention ห้ามใช้ spinner เฉย ๆ นาน ๆ

ต้องแสดง:

- current step
- progress
- rows processed
- warning/error count
- job status
- retry/cancel/view log ถ้ามี

---

## 2.4 Critical Actions Need Confirmation

Action ที่กระทบข้อมูลสำคัญต้อง confirm ก่อนทำ

เช่น:

- Approve user
- Reject user and delete request
- Confirm import
- Map alias
- Close issue
- Accept difference
- Lock day
- Unlock day
- Run retention execute
- Export official ERP template

---

# 3. State Categories

ระบบต้องรองรับ state หลักดังนี้:

```text
1. Loading State
2. Long-Running Job State
3. Empty State
4. Error State
5. Validation Error State
6. Permission Denied State
7. Session Expired State
8. Success State
9. Warning State
10. Confirmation State
11. Stale Data State
12. Disabled Action State
13. Retry / Recover State
14. Offline / Poor Network State
15. Partial Success State
```

---

# 4. Global State Rules

## 4.1 Every Page Must Support These States

ทุก page ต้องมีอย่างน้อย:

- Loading
- Empty
- Error
- Permission Denied
- Success feedback for write action

## 4.2 Every Form Must Support These States

ทุก form ต้องมี:

- field validation
- submit loading
- submit success
- submit error
- unsaved changes warning if long form
- disabled submit reason

## 4.3 Every Table/List Must Support These States

ทุก table/list ต้องมี:

- loading skeleton
- empty result
- filtered empty result
- error loading data
- pagination state
- row action disabled reason

## 4.4 Every Job Must Support These States

ทุก job ต้องมี:

```text
QUEUED
RUNNING
SUCCESS
SUCCESS_WITH_WARNING
FAILED
CANCELLED
```

---

# 5. Loading State Specification

## 5.1 Loading Types

| Type | Use Case | UI Pattern |
|---|---|---|
| Page Loading | เปิดหน้าใหม่ | skeleton layout |
| Card Loading | dashboard card | skeleton card |
| Table Loading | list/table | table skeleton rows |
| Form Submit Loading | กด save/submit | button spinner + disabled form |
| Job Loading | import/replay/export | progress panel |
| Lazy Detail Loading | drawer/detail | section skeleton |

---

## 5.2 Loading Copy

ควรใช้ข้อความที่บอก context

Examples:

```text
กำลังโหลดข้อมูล Control Center...
กำลังโหลด Issue Detail...
กำลังตรวจสอบสิทธิ์ผู้ใช้...
กำลังดึงรายการ Alias ที่รอ review...
กำลังเตรียมข้อมูล Export...
```

ห้ามใช้แต่:

```text
Loading...
```

ทุกที่โดยไม่มี context

---

## 5.3 Page Loading Pattern

ใช้ skeleton แทน blank screen

Desktop:

```text
Page Header Skeleton
Filter Bar Skeleton
KPI Cards Skeleton
Table/List Skeleton
```

Mobile:

```text
Header Skeleton
Card Skeleton List
Bottom Action Disabled
```

---

## 5.4 Submit Loading Pattern

เมื่อ user submit form:

- disable primary button
- show spinner/loading text
- prevent double submit
- keep form values visible
- do not clear form until success confirmed

Example button text:

```text
กำลังบันทึก...
กำลังส่งข้อมูล...
กำลังยืนยัน...
กำลังสร้าง Export...
```

---

# 6. Long-Running Job State

## 6.1 Applies To

งานที่อาจใช้เวลานาน:

- Import large file
- Upload raw chunk
- Normalize batch
- Stock replay
- Reconciliation
- Backfill
- Export large report
- Retention cleanup
- Health check
- Notification retry batch

---

## 6.2 Job Progress Panel

Progress panel ต้องแสดง:

| Field | Description |
|---|---|
| job_type | ประเภทงาน |
| status | QUEUED/RUNNING/SUCCESS/FAILED |
| current_step | ขั้นตอนปัจจุบัน |
| progress_percent | ถ้าคำนวณได้ |
| rows_processed | จำนวน row ที่ประมวลผลแล้ว |
| warning_count | warning |
| error_count | error |
| started_at | เวลาเริ่ม |
| elapsed_time | ใช้เวลามาแล้ว |
| message | ข้อความอธิบาย |

---

## 6.3 Job Steps by Workflow

### Import Job Steps

```text
Reading file
Uploading chunks
Saving raw rows
Validating rows
Normalizing data
Matching alias
Creating unknown alias queue
Preparing confirm summary
```

### Stock Replay Steps

```text
Checking data coverage
Finding anchor snapshot
Loading movement ledger
Calculating daily stock position
Saving result
Updating summary
```

### Reconciliation Steps

```text
Loading field movement
Loading system in-out
Loading physical count
Loading expected stock
Matching records
Generating result code
Creating issues
Updating dashboard summary
```

### Export Steps

```text
Preparing dataset
Applying filters
Generating file
Saving export job
Creating download link
Writing export log
```

### Retention Steps

```text
Running health check
Calculating cutoff date
Creating summary/archive
Finding deletable records
Skipping open issues
Deleting safe records
Writing retention log
Sending summary notification
```

---

## 6.4 Job Failure UI

ถ้า job failed ต้องแสดง:

- job name
- failed step
- error code
- user-friendly message
- what to do next
- retry button if safe
- view log link

Example:

```text
Normalize Batch ไม่สำเร็จ
ระบบพบว่า column Qty มีค่าที่อ่านเป็นตัวเลขไม่ได้ 12 แถว
โปรดเปิด Import Error Detail เพื่อตรวจ row ที่ผิด
[ดู Error Detail] [Retry Normalize]
```

---

# 7. Empty State Specification

## 7.1 Empty State Structure

ทุก empty state ควรมี:

```text
Icon / subtle illustration
Title
Description
Next action optional
Clear filter optional
```

---

## 7.2 Empty Types

| Empty Type | Meaning | Required Action |
|---|---|---|
| True Empty | ยังไม่มีข้อมูลเลย | Create/Import action |
| Filtered Empty | มีข้อมูลแต่ filter แล้วไม่เจอ | Clear filter |
| Completed Empty | ไม่มีงานค้าง | No action / view history |
| Permission Empty | ไม่มีข้อมูลใน scope | Contact Admin / change scope |
| Not Yet Generated | ยังไม่ได้ run job | Run job action |

---

## 7.3 Empty Copy by Page

### Control Center — No Issues

```text
ยังไม่มี issue ที่ต้องแก้
ระบบไม่พบรายการผิดปกติในช่วงวันที่และคลังที่เลือก
```

Action:

```text
ดู Reconciliation Results
```

---

### Alias Review — No Pending Alias

```text
ไม่มี alias ที่รอ review
ชื่อทั้งหมดถูก map แล้วในตอนนี้
```

Action:

```text
ดู Alias ทั้งหมด
```

---

### Pending Inbound — No Pending

```text
ไม่มี Pending Inbound
ยังไม่มีรายการของเข้าที่รอ confirm ในช่วงวันที่นี้
```

---

### User Approval — No Pending Users

```text
ยังไม่มี user รออนุมัติ
เมื่อมีคนสมัครใหม่ รายการจะปรากฏที่นี่
```

---

### Import Center — No Import Batch

```text
ยังไม่มีไฟล์ที่ upload
เริ่มต้นโดยเลือกประเภทไฟล์และ upload Inquiry หรือ System In-Out
```

Action:

```text
Upload File
```

---

### Issue Center — Filtered Empty

```text
ไม่พบ issue ตาม filter ที่เลือก
ลองเปลี่ยนช่วงวันที่ คลัง หรือ status
```

Action:

```text
ล้าง Filter
```

---

### Export Center — No Export Jobs

```text
ยังไม่มี export job
เมื่อคุณสร้างรายงานหรือไฟล์ ERP template ประวัติจะอยู่ที่นี่
```

---

### Notification Logs — No Logs

```text
ยังไม่มีประวัติการแจ้งเตือน
ระบบจะแสดง log เมื่อมี event ที่เข้าเงื่อนไข notification rule
```

---

### Retention Log — No Retention Run

```text
ยังไม่มีประวัติ Retention Job
ระบบจะบันทึก log หลังจาก job cleanup ทำงานครั้งแรก
```

---

### Checker My Tasks — No Tasks

```text
ยังไม่มีงานที่ต้องทำ
ถ้ามีงานนับหรือ verify ระบบจะแสดงที่นี่
```

Action:

```text
แจ้งเข้า / แจ้งออก / นับจริง
```

---

# 8. Error State Specification

## 8.1 Error Message Structure

ทุก error ควรมี:

```text
Title: เกิดอะไรขึ้น
Description: สาเหตุที่เข้าใจง่าย
Impact: กระทบอะไรถ้ามี
Next Action: user ควรทำอะไร
Error Code: สำหรับแจ้ง Admin/Developer
```

---

## 8.2 Error Severity

| Severity | UI Pattern | Example |
|---|---|---|
| Field Error | inline field message | qty invalid |
| Section Error | section alert | alias suggestion failed |
| Page Error | full page error | issue detail load failed |
| Job Error | progress panel failed | reconciliation failed |
| Critical Error | modal/banner + notify | retention failed |

---

## 8.3 Common Error Copy

### Permission Denied

```text
คุณไม่มีสิทธิ์ใช้งานส่วนนี้
โปรดติดต่อ Admin หากต้องการสิทธิ์เพิ่มเติม
```

Code:

```text
PERMISSION_DENIED
```

---

### Warehouse Scope Denied

```text
คุณไม่มีสิทธิ์เข้าถึงข้อมูลของคลังนี้
โปรดเลือกคลังที่คุณมีสิทธิ์ หรือขอสิทธิ์จาก Admin
```

Code:

```text
WAREHOUSE_SCOPE_DENIED
```

---

### Session Expired

```text
Session หมดอายุ
โปรดเข้าสู่ระบบใหม่เพื่อใช้งานต่อ
```

Action:

```text
กลับไป Login
```

Code:

```text
SESSION_EXPIRED
```

---

### Import Failed

```text
Import ไม่สำเร็จ
ระบบพบข้อผิดพลาดระหว่างอ่านหรือประมวลผลไฟล์
โปรดตรวจสอบรายละเอียด error แล้วลองใหม่
```

Action:

```text
ดู Error Detail
```

---

### Unknown Schema

```text
ระบบไม่รู้จักโครงสร้างไฟล์นี้
โปรดตรวจ header และทำ Column Mapping ใหม่
```

Code:

```text
HEADER_NOT_FOUND / MISSING_REQUIRED_MAPPING
```

---

### Reconciliation Failed

```text
Reconciliation ไม่สำเร็จ
ระบบไม่สามารถสร้างผลตรวจได้จากข้อมูลที่มีอยู่
โปรดตรวจ Data Coverage และลอง run ใหม่
```

Action:

```text
ดู Coverage Check
```

---

### Retention Failed

```text
Retention Job ไม่สำเร็จ
ระบบยังไม่ได้ลบข้อมูลตามรอบ cleanup โปรดตรวจ log เพื่อดูสาเหตุ
```

Action:

```text
ดู Retention Log
```

---

### Notification Failed

```text
ส่งแจ้งเตือนไม่สำเร็จ
โปรดตรวจ channel, token reference หรือ target id
```

Action:

```text
ดู Notification Log
```

---

# 9. Validation Error State

## 9.1 Inline Field Error

ใช้กับ form field เช่น:

- qty invalid
- date invalid
- required field missing
- password confirm mismatch
- UOM unknown

Pattern:

```text
Field label
[Input with red border]
Error message below input
```

Example:

```text
Qty
[abc]
Qty ต้องเป็นตัวเลขมากกว่าหรือเท่ากับ 0
```

---

## 9.2 Form-Level Validation Summary

ถ้ามีหลาย error ให้แสดง summary ด้านบน form:

```text
พบข้อมูลที่ต้องแก้ 3 จุด
- Qty ต้องเป็นตัวเลข
- Location จำเป็นต้องระบุ
- UOM ยังไม่รู้จัก
```

---

## 9.3 Import Validation Summary

Import validation ต้องแสดง:

- valid rows
- error rows
- warning rows
- missing required mappings
- unknown alias count
- invalid qty/date count

Example:

```text
ตรวจสอบไฟล์แล้วพบ:
Valid rows: 1,240
Warning rows: 18
Error rows: 4
Unknown Alias: 7
```

Action:

```text
ดูแถวที่ผิด
ไป Alias Review
กลับไป Column Mapping
```

---

# 10. Permission Denied State

## 10.1 Direct URL Access

ถ้า user เปิด URL ที่ไม่มีสิทธิ์:

```text
คุณไม่มีสิทธิ์เข้าหน้านี้
โปรดติดต่อ Admin หากคิดว่าควรเข้าถึงได้
```

Actions:

```text
กลับหน้าแรก
```

---

## 10.2 Disabled Action Due to Permission

ถ้าปุ่มแสดงแต่ disabled เพราะไม่มีสิทธิ์ ต้องมี tooltip/message:

```text
คุณไม่มีสิทธิ์ Export ข้อมูลนี้
```

ห้าม disable ปุ่มแบบไม่มีเหตุผล

---

## 10.3 Checker No Active Session

ถ้า Checker กดนับ/แจ้ง movement โดยยังไม่ check-in:

```text
คุณยังไม่ได้ check-in ที่คลัง
โปรด check-in ก่อนเริ่มนับหรือแจ้ง movement
```

Action:

```text
ไปหน้า Check-in
```

---

# 11. Success State Specification

## 11.1 Success Toast

ใช้กับ action สำเร็จทั่วไป:

```text
บันทึกเรียบร้อยแล้ว
Import confirmed แล้ว
Alias mapped แล้ว
Export พร้อม download แล้ว
ส่งแจ้งเตือนทดสอบสำเร็จ
```

---

## 11.2 Success With Next Action

Action สำคัญควรบอก next action

### Import Confirm Success

```text
Confirm Import สำเร็จ
ระบบกำลังเตรียม Stock Replay และ Reconciliation
```

Action:

```text
ดู Job Status
```

### Alias Map Success

```text
Map Alias สำเร็จ
ระบบจะใช้ mapping นี้กับข้อมูลใหม่ทันที และสามารถ reprocess row ที่เกี่ยวข้องได้
```

Action:

```text
Reprocess Affected Rows
```

### Issue Close Success

```text
ปิด Issue สำเร็จ
รายการนี้จะไม่แสดงใน open issue แล้ว
```

Action:

```text
กลับไป Issue Center
```

---

# 12. Warning State Specification

## 12.1 Warning Use Cases

ใช้เมื่อระบบไปต่อได้ แต่มี risk:

- duplicate file checksum
- missing In-Out บางวัน
- low/medium confidence
- unknown alias ที่ไม่ block
- pending inbound exists
- dashboard stale
- close ready with warning
- export row count high

---

## 12.2 Warning Pattern

```text
Warning title
Short explanation
Impact
Recommended action
Continue button only if safe
```

Example:

```text
พบไฟล์ที่อาจซ้ำ
Checksum ของไฟล์นี้ตรงกับ batch ที่เคย upload แล้ว
ถ้าดำเนินการต่ออาจทำให้ข้อมูลซ้ำ
[ยกเลิก] [ดำเนินการต่อพร้อมระบุเหตุผล]
```

---

# 13. Confirmation State Specification

## 13.1 Confirmation Modal Structure

ทุก confirmation modal ต้องมี:

```text
Title
Description
Impact / Risk
Affected data summary
Reason input if required
Cancel button
Confirm button
```

---

## 13.2 Confirmation Actions

| Action | Confirmation Required | Reason Required | Severity |
|---|---:|---:|---|
| Approve User | Yes | No | High |
| Reject User | Yes | Recommended/Config | High |
| Confirm Import | Yes | If warning | Critical |
| Map Alias | Yes for bulk/low confidence | Optional | High |
| Create Master from Alias | Yes | Optional | High |
| Close Issue | Yes | Yes | High |
| Accept Difference | Yes | Yes | Critical |
| Close Day | Yes | Yes/Optional | Critical |
| Lock Day | Yes | Yes | Critical |
| Unlock Day | Yes | Yes | Critical |
| Run Retention Execute | Yes | Yes | Critical |
| Export ERP Template | Yes | Optional | Critical |

---

## 13.3 Confirmation Copy Examples

### Reject User

```text
Reject และลบคำขอสมัครนี้?
คำขอนี้จะหายจากคิวอนุมัติ แต่ระบบจะเก็บ audit log ไว้
```

Buttons:

```text
ยกเลิก
Reject และลบคำขอ
```

---

### Confirm Import

```text
ยืนยันการ Import ไฟล์นี้?
หลังยืนยัน ข้อมูลจะถูกนำไปใช้สร้าง Snapshot หรือ Movement Ledger และอาจ trigger Stock Replay/Reconciliation
```

Buttons:

```text
ยกเลิก
Confirm Import
```

---

### Lock Day

```text
Lock วันทำงานนี้?
หลัง Lock แล้วจะไม่สามารถแก้ข้อมูลโดยตรงได้ ต้องผ่าน Correction/Investigation เท่านั้น
```

Reason required:

```text
เหตุผลในการ lock
```

---

### Run Retention Execute

```text
Run Retention Cleanup จริง?
ระบบจะลบ/archive ข้อมูลที่เกิน cutoff ตาม policy และจะไม่ลบ open issue หรือ critical audit
โปรดตรวจ DRY_RUN ก่อนดำเนินการ
```

Buttons:

```text
ยกเลิก
Run Cleanup
```

---

# 14. Stale Data State

## 14.1 Definition

Stale data คือข้อมูล summary/dashboard ที่เก่ากว่าข้อมูล source หรือ job ล่าสุด

Example:

```text
Import confirmed แล้ว แต่ dashboard summary ยังไม่ได้ refresh
Reconciliation job ล่าสุด failed
Stock replay ล่าสุดเก่ากว่า In-Out import ล่าสุด
```

---

## 14.2 Stale Warning UI

```text
ข้อมูลอาจไม่ใช่ล่าสุด
มี Import/Reconciliation ที่เกิดขึ้นหลังจาก summary ล่าสุด
```

Actions:

```text
Refresh Summary
Run Reconciliation
View Job Log
```

---

# 15. Disabled Action State

## 15.1 Disabled Button Must Explain Why

ถ้าปุ่ม disabled ต้องมี reason:

```text
ปิดวันไม่ได้ เพราะยังมี Critical Issue 3 รายการ
```

```text
Confirm Import ไม่ได้ เพราะยังไม่ได้ map column Qty
```

```text
นับไม่ได้ เพราะยังไม่ได้ check-in
```

---

## 15.2 Disabled State Reasons

Common reasons:

- missing permission
- missing required field
- invalid validation
- job running
- day locked
- issue already closed
- no active checker session
- unknown alias unresolved
- coverage not ready

---

# 16. Retry / Recover State

## 16.1 Retryable Errors

Retry ได้ในกรณี:

- network timeout
- upload chunk failed
- notification send failed
- export generation failed due temporary issue
- job failed before writing final result

---

## 16.2 Non-Retryable Errors Until Fixed

Retry ไม่ช่วยถ้า:

- missing required column mapping
- permission denied
- locked day
- unknown alias required
- invalid file schema
- missing anchor without override

UI ต้องบอกว่าต้องแก้อะไรก่อน

---

## 16.3 Retry Copy

```text
ลองใหม่
Retry Upload Chunk
Retry Normalize
Retry Export
Retry Notification
```

---

# 17. Offline / Poor Network State

## 17.1 MVP Behavior

Full offline mode ไม่อยู่ใน MVP แต่ต้องรองรับ poor network ขั้นต่ำ:

- show network error
- do not clear form after failed submit
- prevent duplicate submit
- allow retry
- keep local state while page still open

---

## 17.2 Mobile Form Network Error

Example:

```text
ส่งข้อมูลไม่สำเร็จ
เครือข่ายอาจไม่เสถียร ข้อมูลที่กรอกยังอยู่ในหน้านี้ โปรดลองส่งอีกครั้ง
```

Action:

```text
ลองส่งอีกครั้ง
```

---

# 18. Partial Success State

## 18.1 Definition

บางงานสำเร็จบางส่วน เช่น:

- Import valid 1,000 rows แต่ error 20 rows
- Notification ส่งได้ 3 channel fail 1 channel
- Retention ลบได้บาง table แต่ skip บาง table
- Export created แต่ send email failed

---

## 18.2 Partial Success UI

```text
ดำเนินการสำเร็จบางส่วน
สำเร็จ: 1,000 rows
Warning/Error: 20 rows
โปรดตรวจรายละเอียดก่อนใช้ข้อมูลต่อ
```

Actions:

```text
ดูรายละเอียด
Export Error Rows
Retry Failed Items
```

---

# 19. Page-Specific State Requirements

## 19.1 Login / Register

### Loading

```text
กำลังเข้าสู่ระบบ...
กำลังส่งคำขอสมัคร...
```

### Error

- username/password invalid
- pending approval
- locked user
- session expired

### Success

- login success redirect
- register success waiting approval

---

## 19.2 User Approval Center

### Empty

```text
ยังไม่มี user รออนุมัติ
```

### Confirmation

- approve user
- reject and delete request
- lock/unlock user

### Error

- request no longer exists
- permission denied
- role/scope missing

---

## 19.3 Import Center

### Loading

- parsing file
- uploading chunks
- validating rows
- normalizing

### Empty

- no batch uploaded

### Error

- invalid file type
- missing header
- missing mapping
- invalid qty/date
- duplicate file warning

### Confirmation

- confirm import
- cancel import
- overwrite/archive snapshot

---

## 19.4 Alias Review

### Empty

```text
ไม่มี alias ที่รอ review
```

### Loading

```text
กำลังโหลดรายการ alias...
```

### Error

- alias queue not found
- canonical target missing
- alias conflict
- low confidence bulk action blocked

### Confirmation

- map alias
- create new master
- reject alias
- bulk map

---

## 19.5 Backfill Wizard

### Loading / Job

- coverage check
- stock replay
- reconciliation
- task creation

### Error

- missing anchor
- missing In-Out
- replay failed
- recon failed

### Warning

- partial coverage
- low confidence

---

## 19.6 Checker Mobile Pages

### Empty

- no tasks
- no recent submissions

### Error

- no active check-in
- GPS denied
- location outside scope
- network submit failed

### Loading

- submitting count
- uploading photo
- parsing smart input

### Success

- count submitted
- inbound saved
- pending inbound created
- REJ notice created

---

## 19.7 Issue Center / Issue Detail

### Empty

- no open issue
- filtered no result

### Loading

- loading issue list/detail
- loading comparison panel
- loading audit trail

### Error

- issue not found
- permission denied
- source data archived/unavailable

### Confirmation

- assign issue
- close issue
- accept difference
- create recount task

---

## 19.8 Export Center

### Empty

- no export jobs

### Loading / Job

- preparing dataset
- generating file
- saving file

### Error

- export permission denied
- row limit exceeded
- file generation failed

### Success

- export ready
- download link shown

### Confirmation

- official ERP template export

---

## 19.9 Notification Center

### Empty

- no logs
- no channels
- no rules

### Error

- token_ref missing
- target invalid
- send failed
- template variable invalid

### Success

- test send success
- rule saved

---

## 19.10 Retention / Maintenance

### Empty

- no retention jobs yet

### Loading / Job

- dry run
- execute cleanup
- health check

### Error

- retention failed
- DB usage fetch failed
- open issue skip error

### Confirmation

- run retention execute

---

# 20. Toast, Banner, Modal Usage Rules

## 20.1 Toast

Use for short feedback:

- saved success
- action completed
- minor warning
- retry started

Do not use toast only for critical errors

---

## 20.2 Banner

Use for persistent page-level status:

- stale data
- coverage warning
- system health warning
- permission limitation
- close not ready

---

## 20.3 Modal

Use for confirmation or blocking critical state:

- destructive action
- lock/unlock
- retention execute
- accept diff
- duplicate import override

---

# 21. State Copy Guidelines

## 21.1 Tone

ข้อความต้อง:

- ชัดเจน
- ไม่โทษ user
- บอกวิธีแก้
- ใช้ภาษาหน้างานเข้าใจได้
- มี error code สำหรับ Admin/Dev

---

## 21.2 Avoid

ห้ามใช้ข้อความแบบ:

```text
Something went wrong
Failed
Invalid
Error 500
Cannot process
```

โดยไม่มีคำอธิบาย

---

## 21.3 Preferred Pattern

```text
[เกิดอะไรขึ้น]
[ทำไมอาจเกิดขึ้น]
[ต้องทำอะไรต่อ]
[Error code]
```

---

# 22. QA Checklist for States

## 22.1 Page State Checklist

- [ ] loading state exists
- [ ] empty state exists
- [ ] error state exists
- [ ] permission denied state exists
- [ ] mobile state works
- [ ] retry or next action exists when relevant

## 22.2 Form State Checklist

- [ ] required field validation
- [ ] submit loading
- [ ] submit success
- [ ] submit error
- [ ] disabled reason
- [ ] confirmation for critical action

## 22.3 Job State Checklist

- [ ] job queued state
- [ ] job running progress
- [ ] success state
- [ ] success with warning state
- [ ] failed state
- [ ] view log link
- [ ] retry policy

## 22.4 Mobile State Checklist

- [ ] no blank screen
- [ ] loading is visible
- [ ] network error keeps form data
- [ ] sticky action disabled reason shown
- [ ] session/check-in error guides to check-in

---

# 23. Acceptance Criteria

Error/Empty/Loading State ถือว่า complete ถ้า:

- [ ] ทุก page มี loading/empty/error/permission denied state
- [ ] ทุก form มี field validation และ submit state
- [ ] ทุก long-running job มี progress/status/log
- [ ] Error message มี code/message/details/next action
- [ ] Empty state แยก true empty กับ filtered empty
- [ ] Critical action มี confirmation modal
- [ ] Disabled button มีเหตุผล
- [ ] Session expired พาไป login ได้
- [ ] Checker ไม่มี session แล้วระบบพาไป check-in
- [ ] Import error ชี้ row/column/field ได้
- [ ] Reconciliation failed พาไป coverage/job log ได้
- [ ] Retention failed มี log และแจ้ง Admin
- [ ] Notification failed มี retry/log
- [ ] Stale dashboard แสดง warning
- [ ] Mobile network error ไม่ล้างข้อมูลที่กรอก

---

# 24. Final State Handling Statement

```text
Error, Empty, Loading และ Confirmation State ของ Warehouse Reconciliation ERP Platform ต้องทำให้ user รู้เสมอว่าระบบกำลังทำอะไร ไม่มีข้อมูลเพราะอะไร ผิดตรงไหน ต้องแก้อย่างไร และ action ใดมีความเสี่ยง โดยเฉพาะ workflow สำคัญอย่าง import, alias, stock replay, reconciliation, issue, export, notification, close day และ retention ต้องมี progress, validation, recovery path และ audit-aware confirmation ที่ชัดเจน
```

