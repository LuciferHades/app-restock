# 06_information_architecture.md

# Information Architecture Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Information Architecture / Navigation / Page Hierarchy Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer Use  
**Primary Use:** ใช้กำหนดโครงสร้างเมนู หน้า ระบบ navigation สิทธิ์การเข้าถึงแต่ละหน้า และการจัดกลุ่มข้อมูลให้ผู้ใช้แต่ละ role ใช้งานง่าย ไม่สับสน และไม่เห็นข้อมูลเกินหน้าที่

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Information Architecture หรือโครงสร้างการจัดวางข้อมูลและหน้าจอของ Warehouse Reconciliation ERP Platform

เป้าหมายคือให้ AI Agent / Developer / UX Designer เข้าใจว่า:

1. ระบบควรมีกี่กลุ่มเมนู
2. แต่ละ role ควรเห็นหน้าอะไร
3. หน้าไหนเป็น desktop-first
4. หน้าไหนเป็น mobile-first
5. หน้าไหนอยู่ใน MVP
6. หน้าไหนอยู่ phase หลัง
7. user ควรเริ่มจากหน้าไหน
8. หน้าไหน drill down ไปหน้าไหน
9. ข้อมูลระดับ summary/detail ควรถูกแยกอย่างไร
10. ห้ามยัดทุกอย่างไว้หน้าเดียวจน user งง

หลักสำคัญ:

```text
Information Architecture ต้องช่วยให้คนตัดสินใจเร็วขึ้น
ไม่ใช่แค่รวมเมนูให้ครบ
```

---

## 2. Core IA Principle

ระบบนี้มี user หลายกลุ่ม แต่ละกลุ่มมีเป้าหมายต่างกัน

ดังนั้นเมนูต้องออกแบบแบบ role-based และ task-based

ห้ามออกแบบแบบ:

```text
ทุกคนเห็นเมนูเหมือนกันหมด
ทุกข้อมูลอยู่ในตารางเดียว
ทุกหน้าเต็มไปด้วย raw data
```

ต้องออกแบบแบบ:

```text
Admin เห็น control/config
Supervisor เห็น issue/action
Checker เห็น task/mobile input
ERP Admin เห็น export/post queue
Viewer เห็น summary/report
```

---

## 3. Primary User Journeys

## 3.1 Admin Journey

Admin เปิดระบบเพื่อ:

- ดูสถานะวันนี้
- upload Inquiry/In-Out
- จัดการ unknown alias
- อนุมัติ user
- ตั้ง permission
- export report
- ตั้ง notification
- ตรวจ retention/health

Admin entry point:

```text
Control Center
```

Main navigation path:

```text
Control Center
→ Import Center / Backfill Wizard
→ Alias Review
→ Issue Center
→ Export Center
→ User Approval
→ Notification Center
→ Maintenance
```

---

## 3.2 Supervisor Journey

Supervisor เปิดระบบเพื่อ:

- เห็น issue ที่ต้องแก้
- ดู critical issue
- assign task
- สั่ง recount
- approve/close issue
- ดู close readiness

Supervisor entry point:

```text
Control Center หรือ Issue Center
```

Main navigation path:

```text
Control Center
→ Issue Queue
→ Issue Detail
→ Assign Recount / Send to ERP / Close Issue
→ Daily Close Readiness
```

---

## 3.3 Checker Journey

Checker เปิดระบบบนมือถือเพื่อ:

- check-in warehouse
- ดูงานของฉัน
- แจ้งเข้า
- แจ้งออก
- นับจริง
- แจ้ง REJ
- ถ่ายรูปป้าย/goods evidence

Checker entry point:

```text
Check-in / งานของฉัน
```

Main navigation path:

```text
Login
→ Check-in
→ งานของฉัน
→ ทำ task / แจ้ง movement / นับจริง
→ Submit
```

---

## 3.4 ERP Admin Journey

ERP Admin เปิดระบบเพื่อ:

- ดูรายการที่ ERP ยังไม่ post
- ดู REJ_NOT_POSTED
- export ERP template
- mark posted
- confirm ERP post

ERP Admin entry point:

```text
ERP Queue / Export Center
```

Main navigation path:

```text
ERP Queue
→ Issue Detail
→ Export ERP Template
→ Mark Posted
→ Close/Verify
```

---

## 3.5 Viewer / Management Journey

Viewer เปิดระบบเพื่อ:

- ดูภาพรวม
- ดู report
- ไม่แก้ข้อมูล

Viewer entry point:

```text
Dashboard / Report View
```

Main navigation path:

```text
Summary Dashboard
→ Report Detail
→ Export readonly if allowed
```

---

# 4. Top-Level Navigation Structure

ระบบควรมี navigation หลัก 8 กลุ่ม

```text
1. Control
2. Operations
3. Data & Import
4. Reconciliation
5. Master & Alias
6. Export & Notification
7. Admin & Security
8. Maintenance
```

---

## 4.1 Control

ใช้ดูภาพรวมและตัดสินใจรายวัน

Pages:

```text
- Control Center
- Reconciliation Calendar
- Daily Close Readiness
```

Main users:

- Admin
- Supervisor
- Viewer optional

MVP:

- Control Center = Yes
- Daily Close Readiness = Yes
- Reconciliation Calendar = Basic / Optional in MVP

---

## 4.2 Operations

ใช้สำหรับงานหน้างานและ movement

Pages:

```text
- Checker Check-in
- My Tasks
- Smart Inbound Input
- Field Movement Notice
- Physical Count
- REJ Notice
- Count Required Locations
```

Main users:

- Checker
- Supervisor

MVP:

- Checker Check-in = Yes
- My Tasks = Yes
- Smart Inbound Input = Yes
- Physical Count = Yes
- REJ Notice = Yes if REJ critical
- Count Required Locations = Yes

---

## 4.3 Data & Import

ใช้จัดการไฟล์ข้อมูลและ backfill

Pages:

```text
- Import Center
- Import Preview
- Column Mapping
- Import Batch Detail
- Backfill Wizard
- Data Coverage Check
```

Main users:

- Admin
- ERP Admin optional

MVP:

All required

---

## 4.4 Reconciliation

ใช้ดูผลตรวจสอบความตรง/ไม่ตรง

Pages:

```text
- Issue Center
- Issue Detail
- Reconciliation Result List
- Reconciliation Run Log
- Investigation Detail
```

Main users:

- Supervisor
- Admin
- ERP Admin

MVP:

- Issue Center = Yes
- Issue Detail = Yes
- Reconciliation Result List = Yes
- Run Log = Basic
- Investigation Detail = Basic / can be issue detail first

---

## 4.5 Master & Alias

ใช้จัดการ master และชื่อไม่ตรงกัน

Pages:

```text
- Alias Review
- Customer Alias
- Item Alias
- Location Alias
- DC/Warehouse Alias
- UOM Alias
- Master Data Management
```

Main users:

- Admin

MVP:

- Alias Review = Yes
- Alias lists = Yes
- Master Data basic = Yes

---

## 4.6 Export & Notification

ใช้ส่งข้อมูลออกและแจ้งเตือน

Pages:

```text
- Export Center
- Export Job Detail
- Export Log
- Notification Center
- Notification Rules
- Notification Templates
- Notification Logs
```

Main users:

- Admin
- ERP Admin

MVP:

- Export Center basic = Yes
- Export Log = Yes
- Notification Center basic = Yes

---

## 4.7 Admin & Security

ใช้จัดการ user และสิทธิ์

Pages:

```text
- User Approval Center
- User Management
- Role Management
- Permission Management
- Warehouse Scope Management
- Checker Session Monitor
```

Main users:

- Admin

MVP:

All required except Role/Permission UI อาจเริ่มแบบ basic config ได้

---

## 4.8 Maintenance

ใช้ดูสุขภาพระบบ retention backup audit

Pages:

```text
- System Health
- Retention Center
- Retention Log
- Audit Log
- Backup Export
```

Main users:

- Admin

MVP:

- Retention Log = Yes
- Audit Log = Yes
- System Health basic = Recommended

---

# 5. Role-Based Navigation

## 5.1 Admin Navigation

Admin Desktop Sidebar:

```text
Control
  - Control Center
  - Daily Close Readiness
  - Reconciliation Calendar

Data & Import
  - Import Center
  - Backfill Wizard
  - Import Batches
  - Column Mapping

Master & Alias
  - Alias Review
  - Customer Alias
  - Item Alias
  - Location Alias
  - DC Alias
  - UOM Alias
  - Master Data

Reconciliation
  - Issue Center
  - Reconciliation Results
  - Investigation

Export & Notification
  - Export Center
  - Export Log
  - Notification Center

Admin & Security
  - User Approval
  - Users
  - Roles & Permissions
  - Warehouse Scope
  - Checker Sessions

Maintenance
  - Audit Log
  - Retention
  - System Health
```

Admin bottom priority on mobile if needed:

```text
Control / Issues / Import / Alias / More
```

---

## 5.2 Supervisor Navigation

Supervisor Sidebar:

```text
Control
  - Control Center
  - Daily Close Readiness

Reconciliation
  - Issue Center
  - Critical Issues
  - Pending Approval
  - Count Required Locations

Operations
  - Checker Tasks
  - REJ Aging
  - Pending Inbound Review

Reports
  - Daily Reconciliation Report
```

Supervisor should not see:

- User Permission Management unless granted
- System retention setting
- Notification channel token

---

## 5.3 Checker Mobile Navigation

Checker Bottom Navigation:

```text
1. Home / งานของฉัน
2. แจ้งเข้า
3. แจ้งออก
4. นับจริง
5. More
```

More:

```text
- Check-in Status
- แจ้ง REJ
- ประวัติล่าสุด
- Profile / Logout
```

Checker Home:

```text
- Active Warehouse Session
- My Open Tasks
- Quick Actions
- Last Used Location
```

Checker should not see:

- Import
- Alias official mapping
- User management
- Export official
- System health
- Full raw data

---

## 5.4 ERP Admin Navigation

ERP Admin Sidebar:

```text
ERP Queue
  - SYSTEM_MISSING
  - REJ_NOT_POSTED
  - Export Pending

Export
  - Export Center
  - Export Log

Reconciliation
  - Issue Detail
  - Posted Confirmation

Import optional
  - Import System In-Out
```

ERP Admin should not see:

- Physical Count editing
- User approval
- Master alias mapping unless granted

---

## 5.5 Viewer Navigation

Viewer Navigation:

```text
Dashboard
Reports
Issue Summary readonly
Export readonly optional
```

Viewer should not see action buttons

---

# 6. Page Hierarchy

## 6.1 Control Center

```text
Control Center
  ├── Today Status Cards
  ├── Issue Summary
  ├── Pending Alias
  ├── Pending Inbound
  ├── REJ Status
  ├── Import / Backfill Status
  ├── Export Pending
  ├── Close Readiness
  └── System Health Mini Card
```

Drilldown:

```text
Issue Count → Issue Center
Unknown Alias → Alias Review
Pending Inbound → Pending Inbound Review
REJ Not Posted → Issue Center filtered REJ_NOT_POSTED
Export Pending → Export Center
Close Status → Daily Close Readiness
Retention Status → Retention Log
```

---

## 6.2 Import Center

```text
Import Center
  ├── Select Import Type
  ├── Upload File
  ├── Preview Headers
  ├── Preview Sample Rows
  ├── Column Mapping
  ├── Validation Summary
  ├── Unknown Alias Summary
  ├── Confirm Import
  └── Import Log
```

Child pages:

```text
Import Batch Detail
Column Mapping Template
Import Error Detail
```

---

## 6.3 Backfill Wizard

```text
Backfill Wizard
  ├── Step 1: Select Date Range
  ├── Step 2: Upload In-Out File
  ├── Step 3: Coverage Check
  ├── Step 4: Run Stock Replay
  ├── Step 5: Run Reconciliation
  ├── Step 6: Count Required Locations
  └── Step 7: Summary
```

Wizard output links:

```text
Coverage Detail
Replay Log
Issue Center filtered by date range
Count Task List
```

---

## 6.4 Alias Review

```text
Alias Review
  ├── Queue Summary
  ├── Filters by alias type
  ├── Unknown Text List
  ├── Suggested Match
  ├── Confidence Score
  ├── Source Batch/Row
  ├── Actions
  │   ├── Map to Existing
  │   ├── Create New Master
  │   └── Reject
  └── Bulk Actions
```

Child pages:

```text
Alias Detail
Master Detail
Source Row Preview
```

---

## 6.5 Smart Inbound Input

```text
Smart Inbound Input
  ├── Active Check-in Status
  ├── Location Input
  ├── Label Text Input
  ├── Qty / UOM
  ├── Photo Evidence
  ├── Parse Preview
  ├── Confidence Badge
  ├── Confirm as Inbound
  └── Save as Pending Inbound
```

If unknown:

```text
Unknown alias warning
→ Save Pending Inbound
→ Create Alias Queue
```

---

## 6.6 Issue Center

```text
Issue Center
  ├── Summary Filters
  ├── Severity Tabs
  ├── Result Code Filters
  ├── Issue Table / Cards
  ├── Bulk Assign optional
  └── Export filtered rows
```

Required filters:

```text
Date Range
Warehouse
Result Code
Severity
Confidence
Owner Role
Status
Customer
Item
Location
```

---

## 6.7 Issue Detail

```text
Issue Detail
  ├── Header
  │   ├── Result Code
  │   ├── Severity
  │   ├── Confidence
  │   └── Status
  ├── Stock Identity
  │   ├── Customer
  │   ├── Item
  │   ├── Year
  │   ├── Lot
  │   └── Location
  ├── Comparison Panel
  │   ├── Field Movement
  │   ├── System In-Out
  │   ├── Physical Count
  │   └── Expected Stock
  ├── Explanation
  ├── Likely Cause
  ├── Suggested Action
  ├── Action Buttons
  ├── Related Tasks
  ├── Notification Log
  └── Audit Trail
```

Action buttons by issue type:

```text
Assign Recount
Send to ERP Admin
Map Alias
Confirm Pending Inbound
Export Correction
Accept with Reason
Close Issue
```

---

## 6.8 Export Center

```text
Export Center
  ├── Select Export Type
  ├── Select Date / Warehouse / Filters
  ├── Select Rows or Filtered Rows
  ├── Preview Export Data
  ├── Generate File
  ├── Download Link
  └── Export Log
```

Export types:

```text
Issue List
Daily Reconciliation
REJ Report
ERP Correction Template
Count Task
Audit Export
Retention Report
```

---

## 6.9 Notification Center

```text
Notification Center
  ├── Channels
  ├── Rules
  ├── Templates
  ├── Test Send
  └── Logs
```

Channels:

```text
In-app
Telegram
LINE Messaging API
Email
```

Rules:

```text
Event Code
Severity Filter
Target Role
Channel
Template
Cooldown
Send Once Per Issue
Active
```

---

## 6.10 User Approval Center

```text
User Approval Center
  ├── Pending Users Table
  ├── User Detail Drawer
  ├── Approve Modal
  │   ├── Role Selector
  │   ├── Permission Selector
  │   └── Warehouse Scope Selector
  └── Reject Modal
      └── Reason
```

---

## 6.11 Maintenance / Retention

```text
Maintenance
  ├── System Health
  ├── Retention Status
  ├── Last Cleanup Log
  ├── DB Usage
  ├── Open Critical Issues
  ├── Pending Alias
  ├── Backup Export
  └── Manual Run Retention optional
```

---

# 7. MVP Pages

## 7.1 Must Build Pages

| Page | MVP | Primary Role |
|---|---:|---|
| Login | Yes | All |
| Register | Yes | New User |
| Waiting Approval | Yes | New User |
| User Approval Center | Yes | Admin |
| User Management Basic | Yes | Admin |
| Checker Check-in | Yes | Checker |
| My Tasks | Yes | Checker |
| Smart Inbound Input | Yes | Checker |
| Field Movement Notice | Yes | Checker |
| Physical Count | Yes | Checker |
| Import Center | Yes | Admin |
| Column Mapping | Yes | Admin |
| Backfill Wizard | Yes | Admin |
| Alias Review | Yes | Admin |
| Issue Center | Yes | Supervisor/Admin |
| Issue Detail | Yes | Supervisor/Admin/ERP Admin |
| Control Center | Yes | Admin/Supervisor |
| Export Center Basic | Yes | Admin/ERP Admin |
| Notification Center Basic | Yes | Admin |
| Daily Close Readiness | Yes | Admin/Supervisor |
| Audit Log Basic | Yes | Admin |
| Retention Log | Yes | Admin |

---

## 7.2 Can Be Basic in MVP

| Page | MVP Level |
|---|---|
| Reconciliation Calendar | Basic status list instead of full calendar |
| Role Permission Management | Basic UI or config seed |
| System Health | Basic metrics only |
| REJ Notice | Basic form if REJ flow not fully built |
| Export Templates | Start with CSV/Excel basic |

---

## 7.3 Not in MVP

| Page | Reason |
|---|---|
| Visual Warehouse Map | Future UX enhancement |
| Full Capacity Planning | Needs stable stock data first |
| Relocation Scenario Builder | Complex optimization, later phase |
| Custody Contract Center | Later phase |
| Advanced FIFO Risk Board | Needs reliable lot data first |
| Native Mobile App | Web mobile first is enough |

---

# 8. Menu Visibility by Permission

## 8.1 Menu Access Rules

| Menu | Permission Required |
|---|---|
| Control Center | CAN_VIEW_CONTROL_CENTER |
| Import Center | CAN_IMPORT_INQUIRY or CAN_IMPORT_INOUT |
| Backfill Wizard | CAN_IMPORT_INOUT and CAN_RUN_STOCK_REPLAY |
| Alias Review | CAN_MANAGE_ALIAS or CAN_VIEW_ALIAS_QUEUE |
| Issue Center | CAN_REVIEW_ISSUE or CAN_VIEW_RECON_RESULT |
| Export Center | CAN_EXPORT |
| Notification Center | CAN_MANAGE_NOTIFICATION |
| User Approval | CAN_MANAGE_USER |
| Permission Management | CAN_MANAGE_PERMISSION |
| Maintenance | CAN_RUN_RETENTION or CAN_VIEW_HEALTH_CHECK |
| Audit Log | CAN_VIEW_AUDIT_LOG |
| Checker Pages | CAN_COUNT / CAN_CREATE_FIELD_MOVEMENT / active session |

---

## 8.2 Backend Rule

Menu visibility is not security

```text
Frontend hides menu for UX
Backend blocks action for security
```

---

# 9. Data Visibility Levels

## 9.1 Summary Level

ใช้ใน dashboard/report

Example:

```text
Issue count
Matched count
Pending alias count
Close status
```

Allowed for:

- Admin
- Supervisor
- Viewer if granted

---

## 9.2 Operational Detail Level

ใช้ใน issue detail / movement / count

Example:

```text
Field qty
System qty
Count qty
Item/customer/location
```

Allowed for:

- Admin
- Supervisor
- ERP Admin where related

---

## 9.3 Sensitive Admin Level

Example:

```text
User permission
Audit detail
Notification token reference
Retention delete logs
```

Allowed for:

- Admin only

---

## 9.4 Own Task Level

Checker เห็นเฉพาะ:

- task ของตัวเอง
- warehouse ที่ check-in
- history ของตัวเอง

---

# 10. Drilldown Structure

## 10.1 Control Center Drilldown

```text
Critical Issues → Issue Center filtered severity=CRITICAL
Unknown Alias → Alias Review status=PENDING
Pending Inbound → Pending Inbound Review
REJ Not Posted → Issue Center result_code=REJ_NOT_POSTED
Backfill Warning → Backfill Detail / Coverage Check
Export Pending → Export Center status=PENDING
Retention Failed → Retention Log
```

---

## 10.2 Issue Detail Drilldown

```text
Field Movement → Movement Detail
System In-Out → Source Row / Import Batch Detail
Physical Count → Count Detail
Expected Stock → Daily Stock Position
Alias → Alias Detail
Export → Export Job Detail
Audit → Audit Log filtered entity
```

---

# 11. Page State Requirements

ทุกหน้าต้องรองรับ state เหล่านี้

## 11.1 Loading State

- Loading cards
- Skeleton table
- Progress bar for long job

## 11.2 Empty State

ตัวอย่าง:

```text
ยังไม่มี issue ที่ต้องแก้
ยังไม่มี alias ที่รอ review
ยังไม่มี user รออนุมัติ
ยังไม่มี export pending
```

## 11.3 Error State

ตัวอย่าง:

```text
โหลดข้อมูลไม่สำเร็จ
คุณไม่มีสิทธิ์เข้าหน้านี้
session หมดอายุ
import batch failed
```

## 11.4 Permission Denied State

ต้องบอก user ว่า:

```text
คุณไม่มีสิทธิ์ใช้งานหน้านี้
โปรดติดต่อ Admin
```

## 11.5 Confirm State

ใช้กับ action สำคัญ:

- approve user
- reject user
- confirm import
- map alias
- close issue
- lock day
- run retention

---

# 12. Search & Filter Architecture

## 12.1 Global Search Optional

MVP อาจยังไม่ต้องมี global search

Future global search ควรค้นหา:

- issue id
- customer
- item
- location
- lot
- import batch
- export job

---

## 12.2 Required Filters by Page

### Issue Center

- date range
- warehouse
- result code
- severity
- confidence
- owner role
- status
- customer
- item
- location

### Alias Review

- alias type
- status
- confidence range
- source batch
- raw text

### Import Batches

- file type
- date range
- status
- uploaded by

### Export Center

- export type
- date range
- status
- exported by

### Audit Log

- actor
- action
- entity type
- date range

---

# 13. Breadcrumb / Context Rules

Desktop detail pages should show breadcrumb:

```text
Control Center > Issue Center > Issue Detail ISS-0001
```

or:

```text
Import Center > Batch IMP-0001 > Source Row 25
```

Mobile can use simple back button instead

---

# 14. Mobile Information Architecture

## 14.1 Checker Mobile IA

```text
Home
  ├── Check-in Status
  ├── My Tasks
  ├── Quick Actions
  └── Recent Activity

แจ้งเข้า
  ├── Smart Input
  ├── Qty/UOM
  ├── Location
  ├── Photo
  └── Submit/Pending

แจ้งออก
  ├── Location
  ├── Item/Customer
  ├── Qty
  └── Submit

นับจริง
  ├── Task Selector
  ├── Location
  ├── Qty
  ├── Photo optional
  └── Submit

More
  ├── แจ้ง REJ
  ├── ประวัติล่าสุด
  ├── Check-in Detail
  └── Logout
```

## 14.2 Mobile Constraints

- ไม่แสดงตารางใหญ่
- ใช้ card/list
- action ใหญ่และชัด
- form สั้น
- auto-fill last used
- avoid multi-level navigation

---

# 15. Desktop Information Architecture

Desktop ใช้ sidebar + top bar

## 15.1 Sidebar Groups

```text
Control
Operations
Data & Import
Reconciliation
Master & Alias
Export & Notification
Admin & Security
Maintenance
```

## 15.2 Top Bar

- Current warehouse/date selector
- User profile
- Notification bell
- Quick search optional
- System health badge optional

---

# 16. Information Architecture Risks

| Risk | Impact | Mitigation |
|---|---|---|
| เมนูเยอะเกิน user งง | High | Role-based navigation |
| Checker เห็นข้อมูลเยอะเกิน | Medium | Mobile simplified IA |
| Admin ต้องเข้าหลายหน้ามาก | Medium | Control Center drilldown |
| Issue detail ไม่เชื่อม source | High | Drilldown to source row/batch |
| Dashboard แสดง raw data เยอะ | High | Summary-first IA |
| Permission hide menu แต่ backend ไม่ block | High | Backend permission required |
| Alias/Import/Issue แยกกันเกินจน flow ขาด | Medium | Cross-link from detail pages |

---

# 17. IA Acceptance Criteria

Information Architecture ถือว่า complete ถ้า:

- [ ] ทุก role มี entry point ชัดเจน
- [ ] Admin มี Control Center เป็นศูนย์กลาง
- [ ] Checker ใช้ mobile flow ได้โดยไม่เห็นเมนูซับซ้อน
- [ ] Supervisor เห็น issue/action ไม่ใช่ raw data ทั้งหมด
- [ ] ERP Admin เห็น ERP queue/export ที่เกี่ยวข้อง
- [ ] Import → Alias → Replay → Reconciliation → Issue มี flow เชื่อมกัน
- [ ] Control Center drilldown ไป detail ได้
- [ ] Issue Detail drilldown ไป source data ได้
- [ ] ทุกเมนูมี permission guard
- [ ] MVP page list ชัดเจน
- [ ] Future page list ไม่ปนกับ MVP

---

# 18. Final IA Statement

```text
Information Architecture ของระบบนี้ต้องพาคนไปสู่ action ที่ถูกต้องเร็วที่สุด โดยแยกหน้าตาม role และ workflow: Admin คุมระบบ, Supervisor คุม exception, Checker ทำงานหน้างาน, ERP Admin จัดการ post/export และ Viewer ดู summary เท่านั้น ระบบต้องเริ่มจาก Control Center, รองรับ drilldown ถึง source และต้องซ่อนความซับซ้อนของ raw data ไว้หลัง summary/result/issue layer
```

