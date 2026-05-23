# 09_responsive_spec.md

# Responsive Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Responsive UX / Device Behavior / Layout Adaptation Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / UX Designer Use  
**Primary Use:** ใช้กำหนดพฤติกรรมของระบบบน Mobile, Tablet และ Desktop เพื่อให้ Web App ใช้งานได้จริงกับทุก role โดยเฉพาะ Checker ที่ใช้มือถือ และ Admin/Supervisor ที่ใช้ desktop/tablet วิเคราะห์ข้อมูล

---

## 1. Purpose of This Document

เอกสารนี้กำหนดแนวทาง responsive design สำหรับ Warehouse Reconciliation ERP Platform

ระบบนี้มีผู้ใช้หลายกลุ่มและแต่ละกลุ่มใช้อุปกรณ์ต่างกัน:

- Checker ใช้มือถือหน้างาน
- Supervisor อาจใช้ tablet หรือ desktop
- Admin ใช้ desktop เป็นหลัก
- ERP Admin ใช้ desktop เพื่อ export/post ตรวจข้อมูล
- Viewer/Management ใช้ desktop/tablet เพื่อดู summary

ดังนั้น responsive design ต้องไม่ได้แค่ “ย่อหน้าจอให้พอดี” แต่ต้องปรับ workflow, navigation, layout, component และ action priority ให้เหมาะกับอุปกรณ์จริง

หลักสำคัญ:

```text
Mobile = ทำงานหน้างานเร็ว
Tablet = review/approve/check exception
Desktop = analyze/import/export/manage system
```

---

## 2. Responsive Design Philosophy

## 2.1 Device = Different Job

แต่ละขนาดหน้าจอไม่ได้ใช้แค่แสดงข้อมูลเดียวกัน แต่ใช้ทำงานคนละแบบ

```text
Mobile:
  ใช้กรอกข้อมูลเร็ว นับของ แจ้งเข้าออก ถ่ายรูป ทำ task

Tablet:
  ใช้เดินตรวจ ดู issue card approve/reject quick action

Desktop:
  ใช้ import, mapping, table, export, permission, audit, reconciliation analysis
```

ห้ามออกแบบแบบ:

```text
เอาหน้า desktop table ใหญ่ ๆ มาย่อบนมือถือ
```

ต้องออกแบบแบบ:

```text
Mobile ใช้ card/form/action flow
Desktop ใช้ table/filter/detail drawer
```

---

## 3. Breakpoints

## 3.1 Standard Breakpoints

| Device | Width | Layout Mode |
|---|---:|---|
| Small Mobile | < 480px | single column, bottom nav, compact cards |
| Mobile | 480–767px | single column, bottom nav, large touch actions |
| Tablet | 768–1023px | 2-column cards, collapsible sidebar, drawer detail |
| Desktop | 1024–1439px | sidebar + main content, tables, filters |
| Large Desktop | >= 1440px | expanded dashboard, wider tables, split panels |

---

## 3.2 Tailwind-Compatible Breakpoints

ถ้าใช้ Tailwind CSS ให้ยึด approximate:

```text
sm: 640px
md: 768px
lg: 1024px
xl: 1280px
2xl: 1536px
```

Mapping:

```text
Mobile default = no prefix
Tablet = md
Desktop = lg+
Large Desktop = xl/2xl
```

---

# 4. Role by Device Matrix

| Role | Primary Device | Secondary Device | UI Strategy |
|---|---|---|---|
| Checker | Mobile | Tablet | Bottom nav, task cards, simple forms |
| Supervisor | Tablet/Desktop | Mobile | Issue cards/table, quick approve, detail drawer |
| Admin | Desktop | Tablet | Sidebar, full table, import/export/config |
| ERP Admin | Desktop | Tablet | ERP queue, export table, issue detail |
| Planner | Desktop | Tablet | capacity/simulation later |
| Viewer | Desktop/Tablet | Mobile | summary dashboard, readonly report |

---

# 5. Global Responsive Layout

## 5.1 Desktop Layout

Desktop ใช้ structure:

```text
App Shell
├── Sidebar Navigation
├── Top Bar
│   ├── Business Date Selector
│   ├── Warehouse Selector
│   ├── Notification Bell
│   └── User Menu
└── Main Content
    ├── Page Header
    ├── Filters / Actions
    ├── Summary Cards
    ├── Table / Detail / Wizard
    └── Footer / Pagination
```

Desktop เหมาะกับ:

- Control Center
- Import Center
- Column Mapping
- Alias Review
- Issue Center
- Export Center
- User Permission
- Audit Log
- Retention

---

## 5.2 Tablet Layout

Tablet ใช้ structure:

```text
App Shell
├── Collapsible Sidebar or Top Nav
├── Top Bar Compact
└── Main Content
    ├── Page Header
    ├── 2-column Cards
    ├── List/Table Compact
    └── Detail Drawer / Fullscreen Drawer
```

Tablet เหมาะกับ:

- Supervisor issue review
- Count required locations
- Pending inbound review
- Quick approval
- Control Center summary

---

## 5.3 Mobile Layout

Mobile ใช้ structure:

```text
Mobile App Shell
├── Top Compact Header
│   ├── Page Title
│   ├── Active Warehouse / Date
│   └── Notification icon optional
├── Main Content Single Column
└── Bottom Navigation
```

Mobile เหมาะกับ:

- Checker check-in
- My Tasks
- Smart Inbound
- Field Out
- Physical Count
- REJ Notice
- Recent Activity

---

# 6. Navigation Responsiveness

## 6.1 Desktop Sidebar

ใช้กับ desktop/tablet large

Sidebar groups:

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

Rules:

- แสดงเฉพาะเมนูที่ user มี permission
- active menu ต้องชัด
- sidebar collapse ได้ใน tablet/desktop narrow
- group label ใช้ text-sm/medium

---

## 6.2 Tablet Navigation

Options:

```text
1. Collapsible sidebar
2. Top tabs for limited role เช่น Supervisor
3. Drawer menu from hamburger
```

Supervisor tablet recommended:

```text
Top tabs:
- Control
- Issues
- Tasks
- Close
```

---

## 6.3 Mobile Bottom Navigation

ใช้กับ Checker เป็นหลัก

Bottom nav:

```text
Home
แจ้งเข้า
แจ้งออก
นับจริง
More
```

Rules:

1. ไม่เกิน 5 items
2. item สูงอย่างน้อย 56px
3. icon + label
4. active state ชัดเจน
5. ถ้าไม่มี permission ให้ไม่แสดงเมนู
6. ถ้าไม่มี active checker session ให้ block action และพาไป check-in

---

## 6.4 Mobile More Menu

More menu ประกอบด้วย:

```text
- Check-in Status
- แจ้ง REJ
- ประวัติล่าสุด
- Profile
- Logout
```

ถ้า user เป็น Admin เปิดมือถือ:

```text
- Control Center
- Issue Center
- Alias Review
- Import Status
- More admin tools
```

แต่ Admin mobile ไม่ใช่ full admin replacement

---

# 7. Page-Level Responsive Behavior

---

# 7.1 Login Page

## Desktop

Layout:

```text
2 columns
Left: product intro / key concept
Right: login card
```

Login card width:

```text
360–420px
```

## Mobile

Layout:

```text
Single column
Logo / title
Login form
Register link
```

Rules:

- input height >= 44px
- password field readable
- no large decorative panel

---

# 7.2 Control Center

## Desktop

Layout:

```text
Header row
KPI cards 4 columns
Issue summary + close readiness 2 columns
Operational queues 2–3 columns
System health row
```

Example grid:

```text
lg: grid-cols-4
xl: grid-cols-4 or 5
```

## Tablet

```text
KPI cards 2 columns
Issue summary full width
Close readiness full width
```

## Mobile

```text
Stack cards single column
Priority order:
1. Critical Issue
2. Close Status
3. Pending Action
4. Import/Backfill
5. Retention/Health
```

Mobile card should be compact:

```text
Title + number/status + one-line explanation + action link
```

---

# 7.3 Import Center

## Desktop

Best layout:

```text
Stepper top
Main panel left/center
Validation summary right optional
Preview table full width
```

Column Mapping:

```text
Two-column mapping table
Source Column | Canonical Field
```

## Tablet

```text
Stepper compact horizontal or vertical
Preview table scrollable
Mapping cards or compact table
```

## Mobile

Import on mobile is not primary

Mobile behavior:

- allow view status only
- upload may be hidden or warning
- if upload allowed, use simple stepper
- preview only sample rows as cards
- recommend desktop for large file

UI message:

```text
การ Import ไฟล์ขนาดใหญ่แนะนำให้ทำบน Desktop เพื่อความเสถียร
```

---

# 7.4 Backfill Wizard

## Desktop

Layout:

```text
Stepper left or top
Main content center
Coverage summary table
Progress panel
```

Coverage table can show many dates/metrics

## Tablet

```text
Vertical stepper
Coverage cards by date
Action buttons sticky bottom
```

## Mobile

Backfill is Admin tool; not primary mobile

Mobile should support:

- view backfill status
- view warnings
- not recommended for upload/run heavy job

---

# 7.5 Alias Review

## Desktop

Layout:

```text
Left: Alias queue table
Right: Detail panel / suggested match
```

or:

```text
Full table + detail drawer
```

Columns visible:

- alias type
- raw text
- suggested match
- confidence
- occurrences
- source
- action

## Tablet

```text
Alias cards 2 columns or list
Detail drawer fullscreen/side
```

## Mobile

```text
Card list
Tap card → Fullscreen detail
Action buttons sticky bottom
```

Mobile card:

```text
Alias type badge
Raw text
Suggested match
Confidence badge
Occurrences
```

---

# 7.6 Checker Check-in

## Mobile First

Layout:

```text
Active session card
Warehouse selector
GPS status
Check-in button
Warnings
```

Rules:

- one primary button
- GPS permission instruction clear
- if already active, show current warehouse prominently
- if changing warehouse, show warning modal

## Tablet/Desktop

Use same content centered card

---

# 7.7 My Tasks

## Mobile

Task cards single column

Task card includes:

```text
Task type
Location
Reason
Priority
Due optional
Start button
```

Sort order:

1. Critical/High priority
2. Due soon
3. Assigned first
4. Location grouping optional

## Tablet

Task cards 2 columns

## Desktop

Table or cards depending role

---

# 7.8 Smart Inbound Input

## Mobile First

Layout order:

```text
1. Active warehouse/session
2. Location
3. Text from label
4. Qty / UOM
5. Photo
6. Parse Preview
7. Submit buttons
```

Sticky bottom action:

```text
Confirm Inbound | Save Pending
```

Rules:

- input height >= 44px
- camera/photo button large
- parse preview must be visible before submit
- unknown tokens shown in warning area

## Tablet/Desktop

Can use two-column layout:

```text
Left: input form
Right: parse preview / recent history
```

---

# 7.9 Field Movement Notice

## Mobile

Single column form

For OUT:

```text
Location
Customer/Item smart search
Qty/UOM
Photo optional
Submit
```

Use last used suggestions

## Desktop/Tablet

Form + recent movement list side by side

---

# 7.10 Physical Count

## Mobile

Task-based layout:

```text
Task header
Location confirmation
Qty input large
Optional item/lot verify
Photo optional
Submit count
```

Qty input should be prominent

## Tablet/Desktop

Count task list + detail drawer

---

# 7.11 Issue Center

## Desktop

Use full data table

Layout:

```text
Summary cards
Filter bar
Issue table
Pagination
Detail drawer optional
```

Columns:

- severity
- result code
- customer
- item
- location
- diff
- confidence
- owner
- status
- age
- action

## Tablet

Use hybrid:

```text
Summary cards 2 columns
Filter chips
Issue card list
Detail drawer fullscreen/side
```

## Mobile

Use card list only

Card includes:

```text
Severity
Result code
Item/customer
Location
Diff
Owner/status
Primary action
```

Avoid wide table

---

# 7.12 Issue Detail

## Desktop

Layout:

```text
Header
2-column identity + status
4-column comparison panel
Explanation + Suggested Action
Related evidence + Audit timeline
```

Comparison panel can be table

## Tablet

Comparison panel becomes stacked cards

```text
Field card
System card
Count card
Expected card
```

## Mobile

Use accordion sections:

```text
Summary
Why it happened
What to do next
Field data
ERP data
Count data
Expected data
Audit
```

Primary action sticky bottom if action needed

---

# 7.13 Export Center

## Desktop

Full workflow:

```text
Export type selector
Filters
Preview table
Generate button
Export log
```

## Tablet

Preview condensed
Use columns selection drawer

## Mobile

Export not primary
Allow:

- view export log
- download existing file
- not recommended to generate large export

---

# 7.14 Notification Center

## Desktop

Tabs:

```text
Channels | Rules | Templates | Test Send | Logs
```

Tables/forms full

## Tablet

Tabs horizontal scroll
Drawer for edit

## Mobile

Admin-only compact view:

- view logs
- toggle rule active
- avoid editing complex templates on mobile

---

# 7.15 User Approval Center

## Desktop

```text
Pending user table
Detail drawer
Approve modal
Reject modal
```

## Tablet

Card list + fullscreen modal

## Mobile

Card list
Approve flow step-by-step:

```text
Review user
→ Select role
→ Select warehouse scope
→ Confirm
```

Reject button danger with confirmation

---

# 7.16 Maintenance / Retention

## Desktop

Dashboard cards + logs table

## Tablet

Cards + compact log list

## Mobile

Status view only:

- last cleanup
- failed jobs
- DB usage
- notify admin

Manual retention run should require desktop or strong confirmation

---

# 8. Responsive Component Behavior

## 8.1 Cards

Desktop:

- 3–4 cards per row
- show more metadata

Tablet:

- 2 cards per row

Mobile:

- 1 card per row
- show only essential metadata
- actions large

---

## 8.2 Tables

Desktop:

- full table
- filters
- pagination
- sticky action column optional

Tablet:

- compact table or card list
- horizontal scroll only if necessary

Mobile:

- replace table with cards
- never require horizontal scrolling for main workflow

---

## 8.3 Filters

Desktop:

- full filter bar
- advanced filter button

Tablet:

- filter chips + drawer

Mobile:

- filter button opens full-screen filter drawer
- show active filter chips

---

## 8.4 Drawers

Desktop:

- side drawer 420–720px

Tablet:

- side drawer or fullscreen depending width

Mobile:

- fullscreen drawer/page

---

## 8.5 Modals

Desktop:

- centered modal

Mobile:

- bottom sheet or fullscreen modal
- action buttons sticky bottom

---

## 8.6 Wizards

Desktop:

- horizontal stepper

Tablet:

- vertical stepper or compact horizontal

Mobile:

- one step per screen
- sticky next/back buttons

---

## 8.7 Forms

Desktop:

- 2-column fields where appropriate

Mobile:

- single column
- large inputs
- avoid dense layout
- sticky action bar

---

# 9. Touch Target & Mobile Usability

## 9.1 Minimum Touch Size

```text
44px x 44px minimum
```

Recommended for primary action:

```text
48–56px height
```

---

## 9.2 Mobile Input Rules

- input font size at least 16px
- number input uses numeric keyboard
- camera/photo button large
- dropdown should be searchable if list > 10
- avoid multi-select complex controls for Checker

---

## 9.3 Sticky Action Bar

Use on mobile forms:

```text
Bottom sticky action bar
Primary button full width or two buttons
```

Examples:

- Submit Count
- Confirm Inbound / Save Pending
- Submit REJ
- Approve/Reject user

---

# 10. Responsive Data Density

## 10.1 Desktop Data Density

Desktop can show:

- more columns
- filters
- metadata
- audit preview
- export actions

## 10.2 Tablet Data Density

Tablet should show:

- summary + key fields
- expandable cards
- quick actions

## 10.3 Mobile Data Density

Mobile should show:

- only essential fields
- progressive detail
- no raw table

Example Issue mobile card:

```text
[CRITICAL] QTY_DIFF
MKMK / W150 / 69
Location: W8-03
Diff: -50 BAG
Owner: ERP Admin
[View Detail]
```

---

# 11. Offline / Poor Network Considerations

Full offline sync is out of MVP, but UI should handle poor network:

## 11.1 MVP Behavior

- show loading clearly
- prevent duplicate submit
- retry upload if chunk fails
- save draft locally for critical mobile forms optional
- show submitted/pending status clearly

## 11.2 Not MVP

- full offline mode
- conflict resolution
- background sync

## 11.3 Recommended Minimal Draft

For Checker mobile forms:

```text
If network fails after user fills form:
- keep form data in local state
- show retry button
- do not clear form
```

---

# 12. Performance Responsive Rules

## 12.1 Desktop

- large tables must use pagination/virtualization if needed
- dashboard uses summary data
- import preview limited rows

## 12.2 Mobile

- avoid loading big data
- lazy load details
- compress photo before upload if possible
- do not show huge dropdowns

## 12.3 Tablet

- load summary first
- detail drawer lazy loads heavy comparison/audit

---

# 13. Responsive Permission Behavior

If user lacks permission:

## Desktop

- hide menu
- if direct URL, show permission denied page

## Mobile

- hide bottom nav item
- if action blocked, show clear reason

Example:

```text
คุณยังไม่ได้ check-in ที่คลังนี้ จึงยังนับไม่ได้
[ไปหน้า Check-in]
```

---

# 14. Responsive Status Priority

On small screens, show status in this order:

1. Severity / Status badge
2. Entity identity เช่น customer/item/location
3. Quantity/diff
4. Owner/next action
5. Secondary metadata

Do not show raw ids first unless user needs them

---

# 15. Responsive Examples by Role

## 15.1 Checker Mobile Home

```text
[Active WH: W8] [Session Active]

งานของฉัน
- RECOUNT W8-03 QTY_DIFF [Start]
- VERIFY REJ W8-REJ [Start]

Quick Actions
[แจ้งเข้า] [แจ้งออก] [นับจริง]

Recent
- Count W8-03 submitted 10:25
```

---

## 15.2 Supervisor Tablet Issue Review

```text
Critical Issues: 5
Pending ERP: 2
Pending Count: 3

[Issue Card]
QTY_DIFF / High
MKMK W150 69
W8-03 Diff -50
[Assign Recount] [View]
```

---

## 15.3 Admin Desktop Control Center

```text
Sidebar
Top date/warehouse filter
KPI card grid
Issue summary table
Backfill status
Retention status
```

---

# 16. Responsive QA Checklist

ต้องทดสอบทุกหน้าสำคัญที่ขนาด:

```text
375px mobile
414px mobile
768px tablet
1024px desktop
1440px large desktop
```

## 16.1 General Checklist

- [ ] ไม่มี horizontal scroll บน mobile สำหรับ workflow หลัก
- [ ] ปุ่มกดได้ง่ายบนมือถือ
- [ ] input ไม่เล็กเกิน
- [ ] bottom nav ไม่ชน browser UI
- [ ] table กลายเป็น card บน mobile
- [ ] drawer กลายเป็น fullscreen บน mobile
- [ ] modal ใช้ได้บน mobile
- [ ] filter ใช้ drawer บน mobile
- [ ] sticky action ไม่บัง content
- [ ] loading/error/empty state responsive

## 16.2 Checker Checklist

- [ ] check-in ใช้มือถือได้
- [ ] smart inbound ใช้มือถือได้
- [ ] submit count ใช้มือถือได้
- [ ] photo upload ใช้มือถือได้
- [ ] task card อ่านง่าย

## 16.3 Admin Checklist

- [ ] import center ใช้ desktop ได้ดี
- [ ] column mapping อ่านชัด
- [ ] alias review table/detail ใช้ได้
- [ ] user approval modal ไม่ล้นจอ
- [ ] export preview ใช้ได้

## 16.4 Supervisor Checklist

- [ ] issue card ใช้ tablet ได้ดี
- [ ] issue detail comparison อ่านเข้าใจบน tablet/mobile
- [ ] action buttons ไม่ซ่อนเกินไป

---

# 17. Responsive Risks

| Risk | Impact | Mitigation |
|---|---|---|
| เอา desktop table ไปแสดงมือถือ | Checker ใช้ไม่ได้ | Card layout on mobile |
| Mobile form ยาวเกิน | กรอกผิด/เลิกใช้ | Single task per screen + sticky action |
| Import บนมือถือ fail | ข้อมูลเข้าไม่ครบ | Recommend desktop for heavy import |
| Issue detail ซับซ้อนบน mobile | ไม่เข้าใจสาเหตุ | Accordion + summary first |
| Filter เยอะบน mobile | ใช้งานยาก | Filter drawer + active chips |
| Action button เล็ก | กดผิด | 44px minimum touch target |
| Admin mobile คาดหวังทำได้ทุกอย่าง | ความสามารถไม่พอ | Status/review on mobile, heavy config desktop-first |

---

# 18. Responsive Acceptance Criteria

Responsive design ถือว่า complete ถ้า:

- [ ] Checker ใช้ mobile เป็นหลักได้จริง
- [ ] Mobile ไม่มี table ใหญ่ใน workflow หลัก
- [ ] Bottom navigation ใช้กับ Checker ได้ง่าย
- [ ] Smart Inbound และ Count มี sticky action bar
- [ ] Admin desktop มี sidebar/table/filter ครบ
- [ ] Supervisor tablet ใช้ issue card/review ได้
- [ ] Issue detail อ่านได้ทั้ง desktop/tablet/mobile
- [ ] Import/Backfill desktop-first แต่ mobile view status ได้
- [ ] Permission denied state ชัดทุก device
- [ ] Loading/error/empty state responsive ทุกหน้า
- [ ] Touch target >= 44px บน mobile
- [ ] หน้าจอทดสอบที่ 375/414/768/1024/1440px ผ่าน

---

# 19. Final Responsive Statement

```text
Responsive design ของ Warehouse Reconciliation ERP Platform ต้องไม่ใช่แค่การย่อขนาดหน้าจอ แต่ต้องปรับ workflow ตามอุปกรณ์: มือถือสำหรับ Checker ทำงานเร็ว, tablet สำหรับ Supervisor review exception, desktop สำหรับ Admin/ERP Admin import, analyze, export และควบคุมระบบ โดยทุกหน้าต้องรักษา clarity, action priority, permission control และ usability ไว้ครบ
```

