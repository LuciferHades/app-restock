# 08_design_system.md

# Design System Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Design System / UI Component / Visual Language Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / UX Designer Use  
**Primary Use:** ใช้กำหนดมาตรฐานสี ตัวอักษร spacing component badge table form modal state และ interaction pattern เพื่อให้ระบบ Web App ดู clean, professional, consistent และใช้งานง่ายทั้งมือถือ แท็บเล็ต และเดสก์ท็อป

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Design System สำหรับ Warehouse Reconciliation ERP Platform

ระบบนี้มีข้อมูลจำนวนมาก มีหลายสถานะ และมีหลาย role ดังนั้นถ้าไม่มี design system ที่ชัดเจน UI จะกลายเป็น:

- สีใช้ไม่สม่ำเสมอ
- badge ความหมายสับสน
- status อ่านยาก
- ตารางรก
- mobile ใช้งานยาก
- action สำคัญไม่ชัด
- user ไม่รู้ว่าอะไรต้องทำก่อน

เป้าหมายของ Design System คือ:

1. ทำให้หน้าจอทุกหน้า consistent
2. ทำให้ status / severity / confidence เข้าใจได้ทันที
3. ทำให้ action สำคัญเด่นชัด
4. ลด cognitive load ของ user
5. รองรับ role-based UI
6. รองรับ mobile-first สำหรับ Checker
7. รองรับ desktop analytical UI สำหรับ Admin/Supervisor
8. ทำให้ AI Agent / Developer สร้าง component ได้เป็นระบบ

หลักสำคัญ:

```text
Design System ต้องช่วยให้ user เข้าใจสถานะและตัดสินใจเร็วขึ้น
ไม่ใช่แค่กำหนดสีให้สวย
```

---

## 2. Design Personality

ระบบควรมี visual personality แบบ:

```text
Clean
Professional
Operational
Reliable
Calm but Alert
Data-first
Action-oriented
```

ไม่ควรมีลักษณะ:

```text
สีเยอะเกิน
กราฟิกเยอะเกิน
การ์ดรก
animation เยอะจนรบกวน
ตารางอ่านยาก
ปุ่ม action ไม่ชัด
```

---

## 3. Visual Design Principles

## 3.1 Clarity First

ทุกหน้าจอต้องอ่านง่าย เข้าใจเร็ว

- ใช้หัวข้อชัด
- ใช้ status badge
- ใช้ตัวเลขพร้อม label
- ใช้คำอธิบายสั้น ๆ ใต้ตัวเลขสำคัญ
- ใช้สีเสริมความหมาย ไม่ใช่แทนความหมายทั้งหมด

---

## 3.2 Alert by Severity

ระบบต้องใช้สีแจ้งเตือนอย่างมีระดับ

```text
Red = ต้องแก้ด่วน / critical
Orange = เสี่ยงสูง / high
Yellow = warning / pending
Green = matched / success
Blue = info / in progress
Gray = neutral / inactive / closed
```

ห้ามใช้สีแดงกับสถานะทั่วไป เพราะจะทำให้ user ชินและไม่สนใจ alert จริง

---

## 3.3 Action Visibility

Primary action ของแต่ละหน้าต้องเห็นชัดที่สุด

ตัวอย่าง:

- Import Center → Confirm Import
- Alias Review → Map Alias
- Issue Detail → Suggested Action Button
- User Approval → Approve / Reject
- Checker Count → Submit Count

---

## 3.4 Consistent Status Language

คำ status ต้องเหมือนกันทั้งระบบ

เช่น ถ้าใช้ `PENDING_INBOUND` ใน Issue Center ต้องใช้ชื่อเดียวกันใน Control Center / Notification / Export

---

## 3.5 Reduce Visual Noise

ข้อมูลที่ไม่สำคัญให้ยุบไว้ใน detail drawer หรือ expandable section

เช่น:

- raw row
- audit trail
- notification logs
- source JSON

---

# 4. Color System

## 4.1 Core Color Tokens

| Token | Purpose | Meaning |
|---|---|---|
| Primary Blue | Primary action / active UI | action หลัก, link, selected state |
| Success Green | Success / matched | ข้อมูลตรง, งานสำเร็จ |
| Warning Yellow | Warning / pending | รอ action, ข้อมูลยังไม่ครบ |
| High Orange | High risk | เสี่ยงสูงแต่ไม่ critical |
| Critical Red | Critical / failed | ต้องแก้ด่วน, error, failed |
| Neutral Gray | Background / inactive | ปิดแล้ว, inactive, neutral |
| Info Cyan | Information | ข้อมูลประกอบ, system processing |
| Purple | Custody / external / special | phase หลัง เช่น custody ลูกค้านอก |

---

## 4.2 Suggested Hex Values

| Token | Hex | Use |
|---|---|---|
| primary-50 | #EFF6FF | light blue background |
| primary-500 | #2563EB | primary button |
| primary-700 | #1D4ED8 | hover primary |
| success-50 | #ECFDF5 | success background |
| success-500 | #10B981 | success badge/button |
| success-700 | #047857 | success dark |
| warning-50 | #FFFBEB | warning background |
| warning-500 | #F59E0B | warning badge |
| orange-500 | #F97316 | high severity |
| critical-50 | #FEF2F2 | critical background |
| critical-500 | #EF4444 | critical badge/button |
| critical-700 | #B91C1C | critical dark |
| gray-50 | #F9FAFB | app background |
| gray-100 | #F3F4F6 | card/subtle bg |
| gray-300 | #D1D5DB | border |
| gray-500 | #6B7280 | secondary text |
| gray-900 | #111827 | primary text |
| purple-500 | #8B5CF6 | custody/external |
| cyan-500 | #06B6D4 | info/process |

---

## 4.3 Usage Rules

### Red / Critical

ใช้เฉพาะ:

- CRITICAL issue
- failed job
- permission denied
- retention failed
- import failed
- system cannot proceed

ห้ามใช้กับ warning ทั่วไป

### Yellow / Warning

ใช้กับ:

- pending inbound
- pending alias
- waiting ERP post
- ready with warning
- low confidence but not critical

### Green / Success

ใช้กับ:

- matched
- completed
- confirmed
- final closed
- cleanup success

### Gray / Neutral

ใช้กับ:

- closed
- inactive
- no activity
- locked
- disabled

---

# 5. Typography System

## 5.1 Font Family

ควรใช้ font ที่อ่านไทยและอังกฤษชัด

Recommended:

```text
Inter + Noto Sans Thai
```

Fallback:

```text
system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif
```

---

## 5.2 Font Scale

| Token | Size | Use |
|---|---:|---|
| text-xs | 12px | badge, metadata, helper text |
| text-sm | 14px | table cell, form helper |
| text-base | 16px | body text, input text |
| text-lg | 18px | card title, section title |
| text-xl | 20px | page subheading |
| text-2xl | 24px | page title |
| text-3xl | 30px | dashboard hero number optional |

---

## 5.3 Font Weight

| Weight | Use |
|---|---|
| 400 Regular | body text |
| 500 Medium | labels, table header |
| 600 Semibold | card title, section title |
| 700 Bold | KPI number, critical emphasis |

---

## 5.4 Typography Rules

1. หัวหน้า page ใช้ text-2xl semibold
2. KPI number ใช้ text-2xl ถึง text-3xl bold
3. Table cell ใช้ text-sm หรือ text-base ตาม density
4. Helper text ใช้ text-xs หรือ text-sm gray
5. ห้ามใช้ font size เล็กกว่า 12px
6. Mobile form input ควรใช้ 16px เพื่ออ่านง่ายและลด zoom บน browser

---

# 6. Spacing System

## 6.1 Spacing Tokens

| Token | Size | Use |
|---|---:|---|
| space-1 | 4px | tight gap |
| space-2 | 8px | small gap |
| space-3 | 12px | compact padding |
| space-4 | 16px | default padding |
| space-5 | 20px | section gap |
| space-6 | 24px | card/page spacing |
| space-8 | 32px | large section gap |
| space-10 | 40px | page block gap |

---

## 6.2 Layout Spacing Rules

- Card padding desktop: 20–24px
- Card padding mobile: 16px
- Form field gap: 12–16px
- Section gap: 24–32px
- Table cell padding: 8–12px vertical, 12–16px horizontal
- Button padding: 10–16px minimum

---

# 7. Radius & Shadow

## 7.1 Border Radius

| Token | Size | Use |
|---|---:|---|
| radius-sm | 6px | badge, small tag |
| radius-md | 8px | input, button |
| radius-lg | 12px | card |
| radius-xl | 16px | modal, drawer |
| radius-2xl | 24px | large panel / mobile card optional |

Recommended default:

```text
Card = radius-lg or radius-xl
Button/Input = radius-md
Modal/Drawer = radius-xl
```

---

## 7.2 Shadow

| Token | Use |
|---|---|
| shadow-sm | small cards |
| shadow-md | popover/dropdown |
| shadow-lg | modal/drawer |
| no shadow | dense table area |

UX Rule:

ใช้ shadow เบา ๆ เพื่อแยก layer ไม่ควรใช้เงาหนักแบบรก

---

# 8. Status Badge System

## 8.1 Severity Badge

| Severity | Color | Label TH | Use |
|---|---|---|---|
| CRITICAL | Red | ด่วนมาก | ต้อง action ก่อนปิดวัน |
| HIGH | Orange | สูง | ควรแก้เร็ว |
| MEDIUM | Yellow | กลาง | ต้องตรวจ |
| LOW | Gray/Blue | ต่ำ | ข้อมูลประกอบ |

---

## 8.2 Confidence Badge

| Confidence | Color | Label TH | Meaning |
|---|---|---|---|
| HIGH | Green | มั่นใจสูง | ข้อมูลครบหลายแหล่ง |
| MEDIUM | Blue/Yellow | มั่นใจกลาง | ข้อมูลพอใช้ แต่ยังขาดบางส่วน |
| LOW | Yellow/Orange | มั่นใจต่ำ | ข้อมูลขาดหลายจุด |
| CRITICAL | Red | เสี่ยงสูง | ข้อมูลไม่พอหรือเสี่ยงผิดสูง |

---

## 8.3 Issue Status Badge

| Status | Color | Meaning |
|---|---|---|
| OPEN | Blue | เปิดใหม่ |
| ASSIGNED | Cyan | มีผู้รับผิดชอบแล้ว |
| IN_REVIEW | Yellow | กำลังตรวจ |
| WAIT_ERP_ADMIN | Orange | รอ ERP Admin |
| WAIT_CHECKER_COUNT | Yellow | รอ Checker นับ |
| RESOLVED | Green | แก้แล้ว |
| CLOSED | Gray/Green | ปิดแล้ว |
| REOPENED | Red/Orange | เปิดกลับมาใหม่ |

---

## 8.4 Daily Status Badge

| Status | Color | Meaning |
|---|---|---|
| OPEN | Blue | เปิดทำงาน |
| PRELIM_CLOSED | Yellow | ปิดเบื้องต้น |
| RECONSTRUCTED | Cyan | คำนวณย้อนหลังแล้ว |
| FINAL_CLOSED | Green | ปิดสมบูรณ์ |
| LOCKED | Dark Gray | ล็อคแล้ว |
| NOT_READY | Red | ยังปิดไม่ได้ |

---

## 8.5 Import Status Badge

| Status | Color |
|---|---|
| UPLOADED | Blue |
| MAPPED | Cyan |
| NORMALIZING | Blue/Info |
| NORMALIZED | Green |
| CONFIRMED | Green |
| FAILED | Red |
| CANCELLED | Gray |

---

## 8.6 Badge Rules

1. Badge ต้องมี text ไม่ใช้สีอย่างเดียว
2. Badge ต้องใช้คำเดียวกันทุกหน้า
3. Critical badge ต้องเด่นแต่ไม่รก
4. ในตาราง ควรใช้ badge size small
5. ใน detail header ใช้ badge size medium

---

# 9. Button System

## 9.1 Button Types

| Type | Use | Style |
|---|---|---|
| Primary | action หลัก | blue filled |
| Secondary | action รอง | white/gray border |
| Success | confirm positive | green filled |
| Warning | action ต้องระวัง | yellow/orange |
| Danger | destructive/critical | red filled |
| Ghost | low emphasis | transparent |
| Link | navigation/drilldown | text blue |

---

## 9.2 Button Examples by Page

### User Approval

- Approve = Success
- Reject = Danger
- View Detail = Secondary

### Import Center

- Confirm Import = Primary
- Cancel Import = Secondary/Danger depending state
- Save Mapping = Primary

### Alias Review

- Map to Existing = Primary
- Create New Master = Secondary
- Reject = Danger

### Issue Detail

- Suggested primary action = Primary
- Close Issue = Success
- Accept with Reason = Warning
- Reopen = Danger/Warning

### Retention

- Run Cleanup = Danger with confirmation
- View Log = Secondary

---

## 9.3 Button Rules

1. หนึ่ง section ควรมี primary action เดียว
2. Destructive action ต้องมี confirm modal
3. Disabled button ต้องมี tooltip/message ว่าทำไม disabled
4. Mobile button ต้องสูงอย่างน้อย 44px
5. Loading button ต้องแสดง spinner หรือ loading text

---

# 10. Form System

## 10.1 Input Types

- Text input
- Number input
- Date picker
- Date range picker
- Select
- Multi-select
- Searchable select
- Textarea
- File upload
- Photo upload
- Checkbox
- Radio group
- Toggle

---

## 10.2 Form Field Structure

ทุก field ควรมี:

```text
Label
Input
Helper text optional
Error message optional
```

ตัวอย่าง:

```text
Qty
[ 1000 ]
หน่วย: BAG
กรอกจำนวนมากกว่า 0
```

---

## 10.3 Validation Style

Invalid field:

- red border
- error text ใต้ field
- icon optional

Warning field:

- yellow border/background
- warning text

---

## 10.4 Required Field Indicator

ใช้ `*` และคำอธิบาย

ตัวอย่าง:

```text
Location *
```

---

## 10.5 Smart Input Field

Smart input เป็น component สำคัญ

ใช้กับ:

- inbound label text
- customer/item search
- location alias

Component behavior:

```text
User types raw text
→ system tokenizes
→ show parse preview
→ show confidence
→ allow confirm/pending
```

Smart input must display:

- parsed customer
- parsed item
- parsed year
- parsed lot optional
- unknown tokens
- confidence badge

---

# 11. Card System

## 11.1 KPI Card

Use in Control Center

Structure:

```text
Title
Main Number / Status
Short explanation
Trend/last updated optional
Action link
```

Example:

```text
Critical Issues
5
ต้องแก้ก่อนปิดวัน
View issues →
```

---

## 11.2 Issue Card

Use on mobile/tablet

Structure:

```text
Severity badge + result code
Customer / Item
Location / Date
Diff summary
Confidence badge
Primary action
```

---

## 11.3 Task Card

Use for Checker

Structure:

```text
Task Type
Location
Reason
Priority
Due time optional
Start button
```

---

## 11.4 Health Card

Use in Maintenance

Structure:

```text
Metric name
Current value
Limit / threshold
Status badge
Last checked
```

---

# 12. Table System

## 12.1 Table Use Cases

- Issue Center desktop
- Alias Review
- Import Batch list
- Export Log
- Audit Log
- User Management

---

## 12.2 Table Requirements

Every data table should support:

- filters
- sorting
- pagination
- row action
- status badges
- empty state
- loading state
- error state

---

## 12.3 Table Density

Desktop:

- compact or normal density
- row height 44–56px

Mobile:

- avoid full table
- use cards instead

---

## 12.4 Sticky Columns

For wide tables:

- first column sticky optional
- action column sticky right optional

Use carefully; avoid clutter

---

# 13. Drawer, Modal, Dialog System

## 13.1 Drawer

Use for quick detail without leaving page

Examples:

- issue quick view
- alias detail
- user request detail
- import row error detail

Drawer width:

- desktop: 420–720px
- mobile: full screen

---

## 13.2 Modal

Use for confirmation or focused action

Examples:

- approve user
- reject user
- map alias
- confirm import
- close issue
- lock day
- run retention

---

## 13.3 Confirmation Modal Pattern

Structure:

```text
Title
Description
Impact warning
Optional reason input
Cancel button
Confirm button
```

Danger confirmation should use red confirm button

---

# 14. Wizard System

Use for multi-step workflows:

- Import
- Backfill
- Export official template optional

## Wizard Requirements

- visible step indicator
- previous/next buttons
- save draft optional
- validation per step
- summary before final confirm
- progress state for processing

---

# 15. Navigation System

## 15.1 Desktop Sidebar

Groups:

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

Sidebar rules:

- show only permitted menus
- active state visible
- support collapsed mode optional

---

## 15.2 Mobile Bottom Navigation

For Checker:

```text
Home
แจ้งเข้า
แจ้งออก
นับจริง
More
```

Bottom nav rules:

- maximum 5 items
- large touch target
- active state clear

---

## 15.3 Breadcrumb

Use on desktop detail pages:

```text
Control Center > Issue Center > Issue Detail
```

Mobile uses back button instead

---

# 16. Toast & Notification UI

## 16.1 Toast Types

| Type | Use |
|---|---|
| Success | save/export/import success |
| Error | failed action |
| Warning | action completed with warning |
| Info | background job started |

---

## 16.2 Toast Rules

- auto dismiss for normal success/info
- error should stay longer
- include action link if useful
- do not spam many toasts at once

---

## 16.3 In-App Notification Bell

Should show:

- unread count
- critical badge if any
- list grouped by severity/date
- link to issue/detail

---

# 17. Empty State System

Every empty state should include:

1. Icon or subtle visual
2. Clear message
3. Explanation
4. Next action if relevant

Examples:

## No Issues

```text
ยังไม่มี issue ที่ต้องแก้
ระบบไม่พบรายการผิดปกติในช่วงวันที่เลือก
```

## No Pending Alias

```text
ไม่มี alias ที่รอ review
ชื่อทั้งหมดถูก map แล้วในตอนนี้
```

## No User Approval

```text
ยังไม่มี user รออนุมัติ
```

---

# 18. Loading & Progress System

## 18.1 Simple Loading

Use skeleton for:

- cards
- tables
- detail panels

## 18.2 Long Job Progress

Use progress panel for:

- import large file
- stock replay
- reconciliation
- export
- retention

Progress panel should show:

- current step
- rows processed
- percentage
- elapsed time
- estimated time optional
- warnings/errors count

---

# 19. Error State System

## 19.1 Error Message Structure

Every error should include:

```text
เกิดอะไรขึ้น
ทำไมอาจเกิดขึ้น
user ทำอะไรต่อได้
error code optional
```

Example:

```text
Import ไม่สำเร็จ
ระบบพบว่า column Qty ยังไม่ได้ map
โปรดกลับไปที่ขั้น Column Mapping แล้วเลือก field ให้ครบ
```

---

## 19.2 Error Severity

| Severity | UI |
|---|---|
| Field error | inline error |
| Page error | error panel |
| Job error | job result panel |
| Critical system error | modal/banner + notification |

---

# 20. Icon System

Suggested icon categories:

| Category | Icon Meaning |
|---|---|
| Upload | import/upload |
| Alert | warning/critical |
| Check | matched/success |
| User | user/permission |
| Map | alias/location |
| Box | stock/item |
| Clipboard | count/task |
| Bell | notification |
| Lock | locked/permission |
| Download | export |
| Database | retention/health |

Icons must be simple line icons, not decorative-heavy

---

# 21. Layout Grid

## 21.1 Desktop

- max content width optional: 1440px
- sidebar fixed
- main content scroll
- 12-column grid optional
- dashboard cards 3–4 columns depending screen

## 21.2 Tablet

- sidebar collapsible
- cards 2 columns
- tables may become compact
- detail drawer full height

## 21.3 Mobile

- single column
- bottom nav
- cards instead of tables
- sticky bottom action bar for forms

---

# 22. Component Naming Convention

Recommended component naming:

```text
StatusBadge
SeverityBadge
ConfidenceBadge
KpiCard
IssueCard
TaskCard
DataTable
FilterBar
DetailDrawer
ConfirmModal
ImportStepper
BackfillWizard
SmartInputParser
PermissionGate
AuditTimeline
ProgressPanel
EmptyState
ErrorState
```

---

# 23. Interaction Rules

## 23.1 Permission Gate

If no permission:

- hide menu
- disable action if visible due to shared view
- show permission denied message on direct URL

## 23.2 Unsaved Changes

Forms with unsaved changes should warn before leaving

## 23.3 Bulk Actions

Bulk actions allowed only for safe operations:

- bulk map aliases if confidence high
- bulk assign issue
- bulk export

Destructive bulk actions require confirmation

## 23.4 Disabled State Explanation

If button disabled, show reason:

```text
ปิดวันไม่ได้ เพราะยังมี Critical Issue 3 รายการ
```

---

# 24. Accessibility Rules

Minimum requirements:

- Contrast sufficient
- Focus visible
- Keyboard navigation for desktop forms/tables
- Labels for inputs
- Status text in addition to color
- Touch target >= 44px on mobile
- Error messages attached to fields
- Avoid tiny text

---

# 25. Design System Acceptance Criteria

Design System ถือว่า complete ถ้า:

- [ ] มี color tokens ชัดเจน
- [ ] มี typography scale
- [ ] มี spacing/radius/shadow rules
- [ ] มี badge system สำหรับ severity/confidence/status
- [ ] มี button hierarchy
- [ ] มี form validation style
- [ ] มี card/table/drawer/modal/wizard pattern
- [ ] มี loading/empty/error state
- [ ] มี mobile bottom navigation rule
- [ ] มี desktop sidebar rule
- [ ] มี permission gate behavior
- [ ] มี accessibility minimum rules
- [ ] component naming พร้อมให้ dev ใช้

---

# 26. Final Design System Statement

```text
Design System ของ Warehouse Reconciliation ERP Platform ต้องทำให้ข้อมูลซับซ้อนอ่านง่าย สถานะสำคัญเห็นชัด action ที่ต้องทำเด่น และ user แต่ละ role ใช้งานได้โดยไม่สับสน โดยใช้สี badge card table form modal และ wizard อย่างสม่ำเสมอทั่วทั้งระบบ
```

