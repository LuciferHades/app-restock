# 07_design.md

# UX/UI Design Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** UX/UI Design Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / UX Designer Use  
**Primary Use:** ใช้กำหนดแนวทางออกแบบหน้าจอ ประสบการณ์ใช้งาน layout interaction flow และพฤติกรรมของ UI สำหรับระบบ Warehouse Reconciliation ERP Platform ให้ใช้งานง่ายระดับ professional และรองรับงานจริงของคลัง

---

## 1. Purpose of This Document

เอกสารนี้กำหนด UX/UI ของระบบ Warehouse Reconciliation ERP Platform

ระบบนี้มีความซับซ้อนสูง เพราะมีข้อมูลหลายแหล่งและ user หลาย role:

- Admin
- Supervisor
- Checker
- ERP Admin
- Planner
- Viewer

ดังนั้น UX/UI ต้องไม่ใช่แค่ “สวย” แต่ต้องช่วยให้ผู้ใช้:

1. ทำงานเร็วขึ้น
2. กรอกข้อมูลผิดน้อยลง
3. เข้าใจทันทีว่าอะไรผิด
4. เห็นเฉพาะสิ่งที่ต้องตัดสินใจ
5. ไม่ต้องเปิดหลายไฟล์/หลายระบบเอง
6. ตรวจย้อนหลังได้
7. ไม่ต้องรู้ logic ทั้งหมดของระบบก็ใช้งานได้

หลักสำคัญ:

```text
UX ของระบบนี้ต้องซ่อนความซับซ้อนของข้อมูลไว้หลัง workflow ที่ชัดเจน
และต้องดัน exception / next action ขึ้นมาให้ user เห็นก่อน raw data
```

---

## 2. Design Philosophy

## 2.1 Operation-First Design

ระบบนี้ต้องออกแบบจากงานจริงของคลัง ไม่ใช่จาก database table

ต้องถามเสมอว่า:

```text
User เปิดหน้านี้มาเพื่อทำอะไร?
เขาต้องตัดสินใจอะไร?
เขาควรเห็นอะไรเป็นอันดับแรก?
อะไรไม่ควรให้เขาเห็นเพื่อไม่ให้สับสน?
```

ตัวอย่าง:

- Checker ไม่ควรเห็น reconciliation table ทั้งหมด
- Supervisor ไม่ควรต้องดู raw import rows
- Admin ไม่ควรต้องแก้ sheet/database ตรง
- ERP Admin ไม่ควรเห็น task นับของ ถ้าไม่เกี่ยวกับการ post/export

---

## 2.2 Exception-First Design

ระบบต้องแสดงสิ่งที่ผิด/เสี่ยงก่อน ไม่ใช่แสดงทุก record

Priority ของข้อมูลบนหน้าจอ:

```text
1. Critical issue
2. Pending action
3. Warning
4. Summary
5. Detail
6. Raw source
```

แนวคิด:

```text
Matched records ควรถูกซ่อนอยู่หลัง summary
Exception records ต้องถูกดันขึ้นมาให้เห็นชัด
```

---

## 2.3 Explainable Design

ทุก issue ต้องอธิบายได้

UI ต้องตอบให้ user เข้าใจ:

- ทำไมระบบบอกว่าไม่ตรง
- ระบบเทียบข้อมูลจากอะไร
- ข้อมูลไหนขาด
- confidence เท่าไร
- ควรทำอะไรต่อ
- ใครเป็น owner

ห้ามแสดงแค่:

```text
QTY_DIFF
```

ต้องแสดงแบบ:

```text
Field OUT = 1,000
ERP OUT = 950
Before Count = 3,000
After Count = 2,000
Expected After = 2,000
ระบบสรุปว่า ERP อาจ post ขาด 50
Suggested Action: ส่ง ERP Admin ตรวจ post
```

---

## 2.4 Mobile-First for Checker

Checker ใช้มือถือเป็นหลัก

ดังนั้นหน้าของ Checker ต้อง:

- ปุ่มใหญ่
- ฟอร์มสั้น
- กรอกน้อย
- ใช้ smart input
- รองรับกล้อง/รูปภาพ
- เห็น task ของตัวเอง
- มี last used location/customer/item
- ใช้ได้แม้รีบทำงานหน้างาน

---

## 2.5 Desktop-First for Admin/Supervisor

Admin/Supervisor ใช้ desktop/tablet เพื่อวิเคราะห์และควบคุมระบบ

ต้องมี:

- dashboard cards
- filters
- tables
- drawer/detail panel
- export
- bulk action where safe
- audit/history

---

## 2.6 Progressive Disclosure

อย่าแสดงข้อมูลทั้งหมดตั้งแต่แรก

ให้แบ่งเป็นระดับ:

```text
Summary → Issue List → Issue Detail → Source Raw Row / Audit
```

ผู้ใช้เห็นเฉพาะระดับที่จำเป็นก่อน แล้วค่อย drill down เมื่ออยากรู้เพิ่ม

---

# 3. UX Principles

## 3.1 Summary First

ทุกหน้าที่มีข้อมูลเยอะควรเริ่มจาก summary card

ตัวอย่าง Issue Center:

```text
Total Issues: 24
Critical: 5
High: 8
Pending ERP: 3
Pending Alias: 6
```

## 3.2 Action-Oriented

ทุก issue หรือ pending item ต้องมี next action ชัดเจน

เช่น:

```text
Send to ERP Admin
Assign Recount
Map Alias
Confirm Inbound
Export Correction
Accept with Reason
Close Issue
```

## 3.3 Prevent Wrong Input

ระบบต้องป้องกันข้อมูลผิดก่อนบันทึก:

- required field
- qty format
- locked date
- no active checker session
- unknown alias
- duplicate import
- out of warehouse scope

## 3.4 Keep User in Context

เมื่อ user กดจาก Control Center ไป Issue Detail ต้องยังรู้ว่า:

- มาจาก card ไหน
- filter อะไรอยู่
- issue อยู่ใน date/warehouse ไหน

ใช้ breadcrumb หรือ back context

## 3.5 Never Make User Guess

ห้ามใช้ code ล้วน ๆ โดยไม่มี explanation

เช่น:

```text
SYSTEM_MISSING
```

ต้องอธิบาย:

```text
หน้างานแจ้ง movement แล้ว แต่ยังไม่พบรายการนี้ใน System In-Out ของ ERP
```

## 3.6 Safe Destructive Actions

Action สำคัญต้อง confirm:

- reject user
- delete request
- confirm import
- map alias
- close issue
- lock day
- run retention

## 3.7 Visible System Status

ระบบต้องบอกว่ากำลังทำอะไรอยู่:

- importing
- normalizing
- running replay
- running reconciliation
- exporting
- sending notification
- cleanup running

---

# 4. Page Design Overview

ระบบมีหน้าหลักตามกลุ่มนี้:

```text
1. Authentication Pages
2. Control Center
3. Import & Backfill Pages
4. Alias Pages
5. Checker Mobile Pages
6. Reconciliation / Issue Pages
7. Export Pages
8. Notification Pages
9. User & Permission Pages
10. Maintenance Pages
```

---

# 5. Authentication Pages

## 5.1 Login Page

### Purpose

ให้ active user เข้าสู่ระบบ

### Layout

Desktop:

```text
Left: Product intro / key message
Right: Login card
```

Mobile:

```text
Logo
Product name
Login form
```

### Fields

- username
- password

### Actions

- Login
- Register
- Forgot password optional

### Error States

| Error | Message |
|---|---|
| wrong password | username หรือ password ไม่ถูกต้อง |
| pending approval | บัญชียังรอ Admin อนุมัติ |
| locked | บัญชีถูกล็อค โปรดติดต่อ Admin |
| expired | สิทธิ์ของบัญชีหมดอายุ |

### UX Rule

อย่าบอกละเอียดเกินไปว่า username หรือ password ผิด เพื่อความปลอดภัย

---

## 5.2 Register Page

### Purpose

ให้ user สมัครและตั้งรหัส แต่ยังไม่เปิดใช้งานจนกว่า Admin approve

### Fields

- full name
- username
- password
- confirm password
- phone
- email optional
- requested role optional
- requested warehouse optional

### Submit Result

แสดงข้อความ:

```text
ส่งคำขอสมัครเรียบร้อยแล้ว
บัญชีของคุณยังใช้งานไม่ได้จนกว่า Admin จะอนุมัติ
```

---

## 5.3 Waiting Approval Page

### Purpose

บอก user ว่ายังรอ approval

### UI

- icon pending
- message ชัดเจน
- contact admin info optional
- logout button

---

# 6. Control Center Design

## 6.1 Purpose

Control Center คือหน้าแรกของ Admin/Supervisor ใช้ดูว่าวันนี้ระบบมีอะไรต้องทำ

ไม่ใช่ dashboard สวย ๆ อย่างเดียว แต่เป็น operation command center

---

## 6.2 Main Layout

Desktop layout:

```text
Top bar:
  - Business date selector
  - Warehouse selector
  - Refresh / Run check
  - Notification bell

Row 1: Key Status Cards
Row 2: Issue Summary + Close Readiness
Row 3: Operational Queues
Row 4: Import/Backfill/Retention Status
```

Mobile/tablet:

```text
Stacked cards
Priority order: Critical → Pending Action → Summary
```

---

## 6.3 Key Cards

### Card 1: Close Status

Shows:

- OPEN
- PRELIM_CLOSED
- RECONSTRUCTED
- FINAL_CLOSED
- LOCKED

Color:

- Green = ready/final
- Yellow = preliminary/warning
- Red = not ready/critical
- Gray = locked/no activity

Action:

- View Close Readiness

---

### Card 2: Critical Issues

Shows:

```text
Critical Issues: 5
```

Subtext:

```text
ต้องแก้ก่อนปิดวัน
```

Action:

- Open Issue Center filtered severity=CRITICAL

---

### Card 3: Unknown Alias

Shows:

```text
Unknown Alias: 12
```

Subtext:

```text
รอ Admin map master
```

Action:

- Open Alias Review

---

### Card 4: Pending Inbound

Shows:

```text
Pending Inbound: 3
```

Subtext:

```text
ของเข้าแต่ข้อมูลยังไม่ครบ
```

Action:

- Open Pending Inbound Review

---

### Card 5: REJ Not Posted

Shows:

```text
REJ Not Posted: 2
```

Subtext:

```text
รอ ERP Admin ตรวจ post
```

Action:

- Open Issue Center filtered REJ_NOT_POSTED

---

### Card 6: Export Pending

Shows:

```text
Export Pending: 4
```

Action:

- Open Export Center

---

### Card 7: Backfill / Import Status

Shows:

- last import
- last in-out backfill
- failed import count

Action:

- Open Import Center / Backfill Wizard

---

### Card 8: Retention / Health

Shows:

- last cleanup status
- DB usage if applicable
- failed job count

Action:

- Open Maintenance

---

## 6.4 Control Center Rules

1. ห้ามแสดง raw rows ทั้งหมด
2. ทุก card ต้อง drilldown ได้
3. ทุกตัวเลขต้องมี date/warehouse context
4. ต้องแสดง last updated
5. ถ้าข้อมูล stale ต้องเตือน

---

# 7. Import Center Design

## 7.1 Purpose

ให้ Admin upload ไฟล์ Inquiry/In-Out โดยมี preview, column mapping, validation, alias summary ก่อน confirm

---

## 7.2 Import Flow UI

แนะนำเป็น stepper:

```text
Step 1: Select File Type
Step 2: Upload File
Step 3: Preview
Step 4: Column Mapping
Step 5: Validation & Alias Summary
Step 6: Confirm Import
Step 7: Processing Result
```

---

## 7.3 Step 1: Select File Type

Options:

- Inquiry
- System In-Out
- Field Count
- Master Data

User must select before upload

---

## 7.4 Step 2: Upload File

UI:

- drag/drop area
- file picker
- accepted file types
- file size warning

Show after upload:

- file name
- file size
- row count after parse
- detected headers

---

## 7.5 Step 3: Preview

Show:

- first 10–20 rows
- headers
- row count
- duplicate file warning if checksum matched

UX Rule:

```text
Preview is mandatory before confirm
```

---

## 7.6 Step 4: Column Mapping

Layout:

```text
Left: Source Column
Right: Canonical Field dropdown
```

Example:

| Source Column | Canonical Field |
|---|---|
| Posting Date | posting_date |
| DC | warehouse_raw |
| Material Desc | item_raw_text |
| Qty | qty |

UI requirements:

- required fields marked red if missing
- saved mapping templates
- auto-suggest previous mapping
- warning if header changed

---

## 7.7 Step 5: Validation & Alias Summary

Show sections:

### Data Validation

- valid rows
- error rows
- missing required values
- invalid dates
- invalid qty

### Alias Summary

- known customer aliases
- unknown customer aliases
- known item aliases
- unknown item aliases
- unknown location/DC

### Coverage Summary for In-Out

- date range detected
- missing dates
- movement count by day

---

## 7.8 Step 6: Confirm Import

Before confirm show:

```text
คุณกำลังจะ import file นี้เข้าสู่ระบบ
ข้อมูลจะถูกนำไปใช้ในการ replay/reconciliation หลัง confirm
```

Actions:

- Confirm Import
- Save Draft
- Cancel

---

## 7.9 Step 7: Processing Result

Show progress:

- parsing
- upload chunk
- normalize
- alias matching
- snapshot/ledger build
- replay/reconciliation queued

Show result:

- success
- warning
- failed
- next action

---

# 8. Backfill Wizard Design

## 8.1 Purpose

รองรับ admin ไม่ upload ทุกวัน และต้อง upload In-Out ย้อนหลัง 1–5 วันหรือมากกว่า

---

## 8.2 Layout

Wizard stepper:

```text
1. Date Range
2. Upload In-Out
3. Coverage Check
4. Stock Replay
5. Reconciliation
6. Count Required Locations
7. Summary
```

---

## 8.3 Step 1: Date Range

Fields:

- date_from
- date_to
- warehouse

Show warning:

```text
ระบบจะใช้ Anchor Snapshot ล่าสุดก่อนช่วงวันที่เลือกเพื่อ replay stock
```

---

## 8.4 Step 3: Coverage Check UI

Show calendar/list:

| Date | In-Out | Field Movement | Pending Inbound | REJ | Status |
|---|---|---|---|---|---|
| 2026-05-01 | OK | OK | 0 | 0 | OK |
| 2026-05-02 | Missing | OK | 1 | 0 | Warning |

Status color:

- Green = OK
- Yellow = Warning
- Red = Critical
- Gray = No Activity

---

## 8.5 Step 6: Count Required Locations

Show list:

| Location | Reason | Movement Count | Suggested Task |
|---|---|---:|---|
| W8-03 | QTY_DIFF | 3 | Recount |
| W8-REJ | REJ_NOT_POSTED | 1 | Verify REJ |

Actions:

- Create all tasks
- Create selected tasks
- Skip with reason

---

# 9. Alias Review Design

## 9.1 Purpose

ให้ Admin map unknown alias ได้เร็วและแม่นยำ

---

## 9.2 Main Layout

Desktop:

```text
Left: Queue list
Right: Detail / suggested match panel
```

Mobile/tablet:

```text
Card list → detail drawer
```

---

## 9.3 Queue List Columns

- alias type
- raw text
- suggested match
- confidence
- source
- count occurrences
- status

---

## 9.4 Detail Panel

Show:

- raw text
- alias type
- source file/batch/row
- sample rows
- suggested match
- confidence
- existing similar aliases

Actions:

- Map to existing
- Create new master
- Reject
- Defer

---

## 9.5 Confidence UI

| Confidence | UI |
|---|---|
| 90–100 | Green badge |
| 70–89 | Yellow badge |
| <70 | Red/Manual review |

Rule:

```text
Low confidence cannot auto-map
```

---

# 10. Checker Mobile Design

## 10.1 Purpose

ให้ Checker ทำงานหน้างานเร็วและง่ายที่สุด

---

## 10.2 Bottom Navigation

```text
Home
แจ้งเข้า
แจ้งออก
นับจริง
More
```

---

## 10.3 Home / My Tasks

Show:

- active warehouse session
- check-in status
- open tasks
- quick actions
- recent submissions

Task card example:

```text
RECOUNT
Location: W8-03
Reason: QTY_DIFF
Priority: High
[Start]
```

---

## 10.4 Check-in Page

Show:

- warehouse selector
- current GPS status
- accuracy
- check-in button
- active session card

If already checked-in:

```text
คุณกำลัง active ที่ WH_W8
สิทธิ์นี้ใช้ได้ถึงสิ้นวันทำงาน
```

If check-in another warehouse:

```text
การ check-in ที่คลังใหม่จะพักสิทธิ์ของคลังเดิม
```

---

## 10.5 Smart Inbound Page

Fields:

1. Location
2. Text from label
3. Qty
4. UOM
5. Photo optional

Parse preview:

```text
Customer: MKMK
Item: W150
Year: 69
Confidence: 95%
```

Buttons:

- Confirm Inbound
- Save as Pending
- Clear

If unknown:

```text
ระบบยังไม่รู้จักบางค่า
สามารถบันทึกเป็น Pending Inbound เพื่อรอ In-Out ยืนยันได้
```

---

## 10.6 Field Out Page

Fields:

- location
- customer/item smart search
- qty
- uom
- reason optional
- photo optional

UX rule:

แจ้งออกควร default จาก existing stock/location ถ้ามีข้อมูล

---

## 10.7 Physical Count Page

Fields:

- task/location
- qty counted
- item/lot optional depending task
- photo optional

Buttons:

- Submit Count
- Save Draft optional

---

## 10.8 REJ Page

Fields:

- from location
- item/customer
- qty
- reason
- photo

After submit:

```text
REJ ถูกบันทึกแล้ว และรอ ERP post
```

---

# 11. Issue Center Design

## 11.1 Purpose

ให้ Supervisor/Admin ดูและจัดการ exception

---

## 11.2 Layout

Top:

- summary cards
- filters

Main:

- table on desktop
- card list on tablet/mobile

Right:

- detail drawer optional

---

## 11.3 Summary Cards

- Critical
- High
- Waiting ERP
- Waiting Checker
- Unknown Alias
- Pending Inbound

---

## 11.4 Filters

- date range
- warehouse
- result_code
- severity
- confidence
- owner role
- status
- customer
- item
- location

---

## 11.5 Issue Table Columns

| Column | Description |
|---|---|
| Issue ID | id |
| Severity | badge |
| Result Code | issue type |
| Customer | canonical/alias |
| Item | item alias/name |
| Location | location |
| Diff | qty difference |
| Confidence | badge |
| Owner | owner role/user |
| Status | status badge |
| Age | time open |
| Action | view/action |

---

# 12. Issue Detail Design

## 12.1 Purpose

หน้า detail ต้องทำให้ user เข้าใจปัญหาโดยไม่ต้องเปิด Excel เอง

---

## 12.2 Layout

```text
Header
→ Stock Identity
→ Comparison Panel
→ Explanation Panel
→ Suggested Action
→ Related Evidence
→ Audit Trail
```

---

## 12.3 Header

Show:

- issue id
- result code
- severity
- confidence
- status
- owner role
- business date

---

## 12.4 Stock Identity

Show:

- customer
- item
- year
- lot
- warehouse
- location
- uom

---

## 12.5 Comparison Panel

4-column comparison:

| Source | Qty | Location | Note |
|---|---:|---|---|
| Field Movement | 1,000 | W8-03 | OUT |
| System In-Out | 950 | W8-03 | ERP posted |
| Physical Count | 2,000 | W8-03 | After count |
| Expected Stock | 2,000 | W8-03 | Replay result |

---

## 12.6 Explanation Panel

Example:

```text
ระบบพบว่า Field แจ้ง OUT 1,000 แต่ ERP ตัด OUT 950
และ Physical Count หลัง movement สนับสนุนว่าออกจริง 1,000
จึงมีแนวโน้มว่า ERP post ขาด 50
```

---

## 12.7 Suggested Action Panel

Buttons:

- Send to ERP Admin
- Assign Recount
- Map Alias
- Confirm Pending Inbound
- Export Correction
- Accept with Reason
- Close Issue

Button visibility depends on permission and result code

---

# 13. Export Center Design

## 13.1 Purpose

ให้ user export ข้อมูลจาก issue/result/summary อย่างมี log

---

## 13.2 Layout

```text
Export Type
→ Filters
→ Row Selection
→ Preview
→ Generate
→ Download / Send
→ Export Log
```

---

## 13.3 Export Types

- Issue List
- Daily Reconciliation
- REJ Report
- ERP Correction Template
- Count Task List
- Audit Export
- Retention Report

---

## 13.4 Export Preview

ต้องแสดง:

- row count
- selected filters
- columns to export
- warning if official ERP template

---

# 14. Notification Center Design

## 14.1 Purpose

ให้ Admin ตั้งค่าแจ้งเตือนได้เอง ไม่ต้องแก้โค้ด

---

## 14.2 Tabs

```text
Channels
Rules
Templates
Test Send
Logs
```

---

## 14.3 Channel Page

Fields:

- channel type
- channel name
- target id
- token reference
- active flag

Channel types:

- In-app
- Telegram
- LINE Messaging API
- Email

---

## 14.4 Rule Page

Fields:

- event_code
- severity filter
- target role
- channel
- template
- cooldown minutes
- send once per issue
- active

---

## 14.5 Logs Page

Columns:

- event code
- channel
- target
- status
- error
- sent at
- retry count

---

# 15. User Approval & Permission Design

## 15.1 User Approval Center

Layout:

```text
Pending Users Table
→ Detail Drawer
→ Approve Modal
→ Reject Modal
```

Pending table columns:

- name
- username
- phone/email
- requested role
- requested warehouse
- requested at
- action

Approve modal:

- role selector
- permission selector
- warehouse scope selector

Reject modal:

- reason
- confirm delete request

---

## 15.2 User Management

Features:

- status badge
- role tags
- permission summary
- warehouse scope
- lock/unlock
- reset password
- edit permission

---

# 16. Maintenance Design

## 16.1 System Health Page

Show:

- DB usage
- row count by table
- last import
- last replay
- last reconciliation
- last retention
- failed jobs
- open critical issues

---

## 16.2 Retention Page

Show:

- cutoff date
- last job status
- deleted rows
- skipped open issues
- archive status
- run manually button if permission

Warning:

```text
Retention จะไม่ลบรายการที่ยังมี open issue หรือ pending action
```

---

# 17. Visual Priority Rules

## 17.1 Severity Colors

| Severity | Color | Meaning |
|---|---|---|
| CRITICAL | Red | ต้องแก้ด่วน |
| HIGH | Orange/Red | สำคัญ |
| MEDIUM | Yellow | ควรตรวจ |
| LOW | Gray/Blue | ข้อมูลประกอบ |

---

## 17.2 Confidence Colors

| Confidence | Color |
|---|---|
| HIGH | Green |
| MEDIUM | Blue/Yellow |
| LOW | Yellow/Orange |
| CRITICAL | Red |

---

## 17.3 Status Colors

| Status | Color |
|---|---|
| MATCHED | Green |
| PENDING | Yellow |
| OPEN | Blue |
| WARNING | Orange |
| FAILED | Red |
| LOCKED | Dark Gray |
| CLOSED | Gray/Green |

---

# 18. Component Guidelines

## 18.1 Cards

Use for:

- KPI
- status
- task
- issue mobile list

Card must include:

- title
- number/status
- short explanation
- action/drilldown

---

## 18.2 Tables

Use for desktop lists:

- issue center
- alias queue
- import batch
- export log
- audit log

Must have:

- filters
- sorting
- pagination
- status badges
- action buttons

---

## 18.3 Drawers

Use for quick detail:

- issue quick view
- user approval detail
- alias detail
- import error row

---

## 18.4 Modals

Use for actions needing confirmation:

- approve user
- reject user
- map alias
- confirm import
- close issue
- lock day
- run retention

---

## 18.5 Wizards

Use for multi-step tasks:

- import
- backfill
- export official template optional

---

# 19. Required UI States

Every page must support:

## Loading

- skeleton cards
- table skeleton
- progress indicator for jobs

## Empty

Friendly message + next action

Example:

```text
ยังไม่มี issue ที่ต้องแก้
```

## Error

Clear explanation + retry button

## Permission Denied

```text
คุณไม่มีสิทธิ์ใช้งานหน้านี้ โปรดติดต่อ Admin
```

## Success

Show success toast + link to next action

---

# 20. Accessibility & Usability

Minimum requirements:

- readable font size
- sufficient contrast
- large touch target on mobile
- clear labels
- avoid color-only meaning
- status text must accompany color
- confirmation for destructive actions

---

# 21. UX Risks

| Risk | Impact | Mitigation |
|---|---|---|
| หน้าจอซับซ้อนเกิน | User ไม่ใช้ | Role-based simplified UI |
| Checker กรอกยาก | ข้อมูลไม่เข้า | Mobile-first smart input |
| Issue code เข้าใจยาก | ต้องถามซ้ำ | Explanation panel |
| Admin ต้องแก้หลายหน้า | เสียเวลา | Control Center drilldown |
| Notification เยอะ | user mute | Cooldown/digest |
| Alias review ช้า | data ค้าง | Suggested match + bulk action |
| Import ไม่เห็น error ก่อน confirm | data พัง | Preview + validation mandatory |

---

# 22. UX Acceptance Criteria

UX/UI ถือว่า complete ถ้า:

- [ ] Checker ใช้มือถือแจ้งเข้า/ออก/นับได้โดยไม่สับสน
- [ ] Admin เห็นสถานะรวมจาก Control Center
- [ ] Supervisor เห็นเฉพาะ issue/action
- [ ] Issue Detail อธิบายปัญหาได้โดยไม่ต้องเปิด Excel
- [ ] Import มี preview/mapping/validation ก่อน confirm
- [ ] Alias Review map unknown ได้เร็ว
- [ ] Backfill Wizard แสดง coverage ชัด
- [ ] Export มี preview และ log
- [ ] Notification ตั้งค่าเองได้
- [ ] Retention แสดงว่าจะลบ/ข้ามอะไร
- [ ] ทุกหน้า loading/error/empty state ครบ
- [ ] ทุก action สำคัญมี confirmation

---

# 23. Final Design Statement

```text
UX/UI ของ Warehouse Reconciliation ERP Platform ต้องทำให้ระบบซับซ้อนดูใช้งานง่าย โดยให้ Checker ทำงานเร็วบนมือถือ Admin คุมระบบจาก Control Center Supervisor เห็นเฉพาะ exception ERP Admin เห็นงาน post/export และทุก issue ต้องอธิบายได้ว่าไม่ตรงอย่างไร ต้องทำอะไรต่อ และใครต้องรับผิดชอบ
```

