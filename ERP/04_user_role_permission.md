# 04_user_role_permission.md

# User Role & Permission Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** User Role / Permission / Access Control Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer Use  
**Primary Use:** ใช้กำหนดผู้ใช้งาน บทบาท สิทธิ์ การอนุมัติบัญชี ขอบเขตคลัง การ check-in ของ Checker และกฎความปลอดภัยในการเข้าถึงระบบ

---

## 1. Purpose of This Document

เอกสารนี้กำหนดโครงสร้างผู้ใช้และสิทธิ์ของระบบ Warehouse Reconciliation ERP Platform

ระบบนี้มีข้อมูลสำคัญเกี่ยวกับ stock, ERP movement, issue, export, audit, user approval และ retention ดังนั้นการคุมสิทธิ์ต้องละเอียดกว่าการมี role แบบง่าย ๆ

เป้าหมายของเอกสารนี้คือ:

1. กำหนด roles หลักของระบบ
2. กำหนด permissions ที่ใช้จริง
3. กำหนด warehouse scope
4. กำหนด user approval flow
5. กำหนด checker GPS/session rule
6. กำหนด action ที่ต้องตรวจ backend permission
7. กำหนด audit event สำหรับการเปลี่ยนสิทธิ์
8. ป้องกันการเข้าถึงข้อมูลผิด warehouse หรือผิดหน้าที่
9. ให้ AI Agent / Developer สร้างระบบ access control ได้โดยไม่ hardcode

หลักสำคัญ:

```text
Role เป็นเพียง template
Permission คือสิทธิ์จริง
Warehouse Scope คือขอบเขตข้อมูล
Session Rule คือเงื่อนไขการใช้งานจริงในวันนั้น
```

---

## 2. Access Control Philosophy

ระบบนี้ต้องใช้แนวคิด:

```text
RBAC + Permission Override + Warehouse Scope + Operational Session
```

แปลว่า:

- **RBAC** = ใช้ role เป็นกลุ่มสิทธิ์เริ่มต้น
- **Permission Override** = เปิด/ปิดสิทธิ์เฉพาะคนได้
- **Warehouse Scope** = จำกัดว่าคนนี้ทำงานได้ที่ warehouse/location ไหน
- **Operational Session** = บาง action ต้องมี session พิเศษ เช่น Checker ต้อง check-in ก่อนนับ

ห้ามออกแบบแบบนี้:

```text
User มี role เดียว
Role ดูเมนูได้/ไม่ได้เท่านั้น
Frontend hide menu แล้วถือว่าปลอดภัย
```

ต้องออกแบบแบบนี้:

```text
User มีหลาย role ได้
User มี permission หลายตัวได้
User มี warehouse scope ได้
Backend ตรวจ permission ทุก action สำคัญ
ทุกการเปลี่ยนสิทธิ์มี audit log
```

---

## 3. User Lifecycle

## 3.1 User Status

| Status | Meaning | Can Login | Notes |
|---|---|---:|---|
| PENDING_APPROVAL | สมัครแล้ว รอ Admin อนุมัติ | No | อยู่ใน user_access_requests |
| ACTIVE | ใช้งานได้ | Yes | ต้องมี permission/scope |
| LOCKED | ถูกล็อค | No | อาจเกิดจาก Admin หรือ login fail |
| SUSPENDED | ถูกพักชั่วคราว | No/Partial | เช่น outsource หมดรอบงาน |
| EXPIRED | สิทธิ์หมดอายุ | No | ใช้กับ user ชั่วคราวได้ |
| DELETED | ลบออกจากระบบ | No | ควร soft delete ใน users |

---

## 3.2 Registration Flow

```text
User เปิดหน้าสมัคร
→ กรอกชื่อ / username / email / phone / password
→ ระบบ hash password + salt
→ สร้าง USER_ACCESS_REQUESTS
→ status = PENDING_APPROVAL
→ แจ้ง Admin ว่ามี user รออนุมัติ
→ User ยัง login ใช้งานจริงไม่ได้
```

### Required Fields

- username
- password
- full_name
- phone or email อย่างน้อยหนึ่งอย่าง
- requested_role optional
- requested_warehouse optional

### Rule

```text
ผู้สมัครใหม่ยังไม่มีสิทธิ์เข้าใช้ระบบจนกว่า Admin จะ approve
```

---

## 3.3 Approval Flow

```text
Admin เปิด User Approval Center
→ เลือก request
→ ตรวจข้อมูล
→ เลือก role
→ เลือก permission
→ เลือก warehouse scope
→ กด Approve
→ สร้าง user active
→ เขียน audit log
→ แจ้ง user/admin
```

### Approve Requirements

ก่อน approve ต้องมีอย่างน้อย:

- role หรือ permission อย่างน้อย 1 ตัว
- warehouse scope ถ้า role นั้นเกี่ยวกับ operation
- status ของ request = PENDING_APPROVAL
- approver มี CAN_MANAGE_USER

---

## 3.4 Reject Flow

```text
Admin เปิด request
→ กด Reject
→ ใส่เหตุผล optional/required ตาม config
→ ลบ request ออกจาก pending queue
→ เก็บ audit log ว่า reject ใคร โดยใคร เวลาไหน เหตุผลอะไร
```

### Important Rule

```text
Reject แล้วลบออกจาก queue ได้
แต่ห้ามหายแบบไม่มี audit
```

---

## 3.5 User Lock / Unlock

Admin ต้องสามารถ:

- lock user
- unlock user
- suspend user
- expire user
- reset password
- change role/permission
- change warehouse scope

ทุก action ต้องมี audit log

---

# 4. Core Roles

ระบบมี roles หลักดังนี้

```text
1. Admin
2. Supervisor
3. Planner
4. ERP Admin
5. Checker
6. Viewer
7. External Warehouse later
8. System Service Account optional
```

---

## 4.1 Admin

### Purpose

ผู้ดูแลระบบหลังบ้าน คุม user, permission, import, alias, export, notification, retention, audit

### Main Responsibilities

- อนุมัติ user
- จัดการ role/permission
- upload Inquiry/In-Out
- map alias
- จัดการ master
- ดู Control Center
- ตั้งค่า Notification
- export report
- run/monitor retention
- ดู audit log

### Typical Permissions

- CAN_LOGIN
- CAN_MANAGE_USER
- CAN_MANAGE_PERMISSION
- CAN_MANAGE_MASTER
- CAN_MANAGE_ALIAS
- CAN_IMPORT_INQUIRY
- CAN_IMPORT_INOUT
- CAN_RUN_RECONCILIATION
- CAN_REVIEW_ISSUE
- CAN_EXPORT
- CAN_MANAGE_NOTIFICATION
- CAN_VIEW_AUDIT_LOG
- CAN_RUN_RETENTION
- CAN_VIEW_CONTROL_CENTER

### UI Access

- Control Center
- User Approval
- User & Permission
- Import Center
- Alias Review
- Master Data
- Issue Center
- Export Center
- Notification Center
- Audit Log
- Maintenance / Retention

### Restrictions

Admin ไม่ควรถูกใช้เป็น default สำหรับทุกคน เพราะสิทธิ์กว้างมาก

---

## 4.2 Supervisor

### Purpose

ผู้ควบคุมงาน operation และตัดสินใจเรื่อง exception

### Main Responsibilities

- ดู issue/exception
- assign task
- approve exception
- สั่ง recount
- review REJ
- review pending inbound
- ดู daily close readiness
- close issue

### Typical Permissions

- CAN_LOGIN
- CAN_VIEW_CONTROL_CENTER
- CAN_REVIEW_ISSUE
- CAN_ASSIGN_TASK
- CAN_APPROVE
- CAN_CREATE_RECOUNT_TASK
- CAN_VIEW_RECON_DETAIL
- CAN_CLOSE_ISSUE
- CAN_VIEW_EXPORT

### UI Access

- Control Center
- Issue Center
- Issue Detail
- Count Required Locations
- Pending Approval
- REJ Aging
- Daily Close Readiness

### Restrictions

Supervisor ไม่จำเป็นต้องมีสิทธิ์จัดการ user/master ทั้งหมด

---

## 4.3 Planner

### Purpose

ผู้วางแผนพื้นที่และการรับเข้าใน phase หลัง

### Main Responsibilities

- ดู capacity
- run available location simulation
- plan inbound/outbound
- create reservation
- propose relocation

### Typical Permissions

- CAN_LOGIN
- CAN_VIEW_CONTROL_CENTER
- CAN_RUN_SIMULATION
- CAN_CREATE_PLAN
- CAN_VIEW_CAPACITY
- CAN_CREATE_CHECKER_TASK
- CAN_REVIEW_PLAN_ACTUAL

### MVP Status

Planner role มีไว้ใน permission model ได้ แต่ feature simulation/capacity ขั้นสูงยังเป็น future phase

---

## 4.4 ERP Admin

### Purpose

ผู้ดูแลรายการที่ต้อง post/confirm ใน ERP และจัดการ export template

### Main Responsibilities

- ดู SYSTEM_MISSING
- ดู REJ_NOT_POSTED
- export ERP correction/adjustment/template
- mark posted
- confirm ERP post
- upload System In-Out ได้ในบางองค์กร

### Typical Permissions

- CAN_LOGIN
- CAN_VIEW_ERP_QUEUE
- CAN_EXPORT
- CAN_MARK_ERP_POSTED
- CAN_IMPORT_INOUT optional
- CAN_REVIEW_ISSUE
- CAN_VIEW_RECON_DETAIL

### UI Access

- ERP Admin Queue
- Export Center
- Issue Detail
- REJ Not Posted
- System Missing

### Restrictions

ERP Admin ไม่ควรแก้ Physical Count โดยตรง

---

## 4.5 Checker

### Purpose

ผู้ใช้งานหน้างาน ใช้มือถือ นับของ แจ้ง movement แจ้ง REJ และทำ task

### Main Responsibilities

- check-in warehouse
- ดูงานของฉัน
- นับจริง
- แจ้งเข้า
- แจ้งออก
- แจ้ง REJ
- แนบรูปป้าย/กอง
- ทำ recount/verify task

### Typical Permissions

- CAN_LOGIN
- CAN_CHECKIN_WAREHOUSE
- CAN_COUNT
- CAN_CREATE_FIELD_MOVEMENT
- CAN_CREATE_REJ
- CAN_VIEW_MY_TASK
- CAN_UPLOAD_PHOTO

### UI Access

- Check-in
- งานของฉัน
- แจ้งเข้า
- แจ้งออก
- นับจริง
- แจ้ง REJ
- ประวัติล่าสุด

### Important Rule

```text
Checker ต้องมี active checker_daily_session ก่อนนับหรือทำ task ใน warehouse นั้น
```

### Restrictions

Checker ไม่ควร:

- manage alias official
- approve own issue
- export official ERP template
- close day
- edit locked records

---

## 4.6 Viewer

### Purpose

ดูข้อมูลอย่างเดียว เช่น ผู้บริหารหรือผู้เกี่ยวข้องที่ไม่ต้องแก้ไขข้อมูล

### Typical Permissions

- CAN_LOGIN
- CAN_VIEW_CONTROL_CENTER
- CAN_VIEW_REPORT
- CAN_VIEW_ISSUE_READONLY

### Restrictions

- ห้าม edit/import/export unless assigned
- ห้าม close/approve

---

## 4.7 External Warehouse Later

### Purpose

ใช้ใน phase หลังสำหรับคลังภายนอกหรือ cross-warehouse custody

### MVP Status

Not required in MVP

---

## 4.8 System Service Account

### Purpose

ใช้สำหรับ system job เช่น retention, nightly replay, scheduled notification

### Typical Permissions

- CAN_RUN_SYSTEM_JOB
- CAN_RUN_RETENTION
- CAN_RUN_HEALTH_CHECK
- CAN_SEND_NOTIFICATION

### Rule

ต้องแยกจาก human user และ audit log ต้องระบุว่าเป็น system job

---

# 5. Permission Catalog

## 5.1 Authentication & User Management

| Permission | Description | Typical Role |
|---|---|---|
| CAN_LOGIN | เข้าใช้งานระบบ | All active users |
| CAN_MANAGE_USER | approve/reject/lock/unlock user | Admin |
| CAN_MANAGE_PERMISSION | จัดการ role/permission | Admin |
| CAN_RESET_USER_PASSWORD | reset password user | Admin |
| CAN_VIEW_USER_AUDIT | ดูประวัติ user | Admin |

---

## 5.2 Import & Data Management

| Permission | Description | Typical Role |
|---|---|---|
| CAN_IMPORT_INQUIRY | upload Inquiry | Admin |
| CAN_IMPORT_INOUT | upload System In-Out | Admin / ERP Admin |
| CAN_IMPORT_MASTER | upload master data | Admin |
| CAN_CONFIRM_IMPORT | confirm import หลัง preview | Admin |
| CAN_MANAGE_COLUMN_MAPPING | จัดการ column mapping | Admin |
| CAN_VIEW_IMPORT_LOG | ดู import log | Admin |

---

## 5.3 Alias & Master Data

| Permission | Description | Typical Role |
|---|---|---|
| CAN_MANAGE_ALIAS | map/create/reject alias | Admin |
| CAN_VIEW_ALIAS_QUEUE | ดู alias queue | Admin/Supervisor optional |
| CAN_MANAGE_MASTER | จัดการ master data | Admin |
| CAN_CREATE_TEMP_MASTER | สร้าง temporary master | Admin |

---

## 5.4 Operation

| Permission | Description | Typical Role |
|---|---|---|
| CAN_CHECKIN_WAREHOUSE | check-in warehouse | Checker |
| CAN_VIEW_MY_TASK | ดู task ของตัวเอง | Checker |
| CAN_COUNT | นับจริง | Checker |
| CAN_CREATE_FIELD_MOVEMENT | แจ้ง movement | Checker/Staff |
| CAN_CREATE_PENDING_INBOUND | แจ้ง inbound ไม่ครบข้อมูล | Checker/Staff |
| CAN_CREATE_REJ | แจ้ง REJ | Checker |
| CAN_UPLOAD_PHOTO | แนบรูป | Checker |
| CAN_CREATE_RECOUNT_TASK | สร้างงาน recount | Supervisor |
| CAN_ASSIGN_TASK | assign task | Supervisor/Admin |

---

## 5.5 Reconciliation & Issue

| Permission | Description | Typical Role |
|---|---|---|
| CAN_RUN_STOCK_REPLAY | run stock replay | Admin/System |
| CAN_RUN_RECONCILIATION | run reconciliation | Admin/System |
| CAN_VIEW_RECON_RESULT | ดู recon result | Admin/Supervisor/ERP Admin |
| CAN_VIEW_RECON_DETAIL | ดู detail | Admin/Supervisor/ERP Admin |
| CAN_REVIEW_ISSUE | review issue | Supervisor/Admin |
| CAN_ASSIGN_ISSUE | assign issue | Supervisor/Admin |
| CAN_CLOSE_ISSUE | close issue | Supervisor/Admin |
| CAN_ACCEPT_DIFF | accept difference with reason | Supervisor/Admin |

---

## 5.6 ERP & Export

| Permission | Description | Typical Role |
|---|---|---|
| CAN_VIEW_ERP_QUEUE | ดูรายการรอ ERP | ERP Admin |
| CAN_EXPORT | export file/report | Admin/ERP Admin |
| CAN_EXPORT_ERP_TEMPLATE | export ERP template | ERP Admin/Admin |
| CAN_MARK_ERP_POSTED | mark posted in ERP | ERP Admin |
| CAN_VIEW_EXPORT_LOG | ดู export log | Admin/ERP Admin |

---

## 5.7 Notification

| Permission | Description | Typical Role |
|---|---|---|
| CAN_MANAGE_NOTIFICATION | ตั้งค่า channel/rule/template | Admin |
| CAN_TEST_NOTIFICATION | test send | Admin |
| CAN_VIEW_NOTIFICATION_LOG | ดู log แจ้งเตือน | Admin |

---

## 5.8 Close / Lock / Maintenance

| Permission | Description | Typical Role |
|---|---|---|
| CAN_VIEW_CLOSE_READINESS | ดู readiness | Admin/Supervisor |
| CAN_CLOSE_DAY | close day | Admin/Supervisor |
| CAN_LOCK_DAY | lock day | Admin |
| CAN_UNLOCK_DAY | unlock day | Admin only |
| CAN_RUN_RETENTION | run retention manually | Admin/System |
| CAN_VIEW_HEALTH_CHECK | ดู health check | Admin |
| CAN_VIEW_AUDIT_LOG | ดู audit log | Admin |

---

# 6. Role-Permission Matrix

## 6.1 Default Matrix

| Permission | Admin | Supervisor | ERP Admin | Checker | Planner | Viewer |
|---|---:|---:|---:|---:|---:|---:|
| CAN_LOGIN | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| CAN_MANAGE_USER | ✓ |  |  |  |  |  |
| CAN_MANAGE_PERMISSION | ✓ |  |  |  |  |  |
| CAN_IMPORT_INQUIRY | ✓ |  |  |  |  |  |
| CAN_IMPORT_INOUT | ✓ |  | ✓ |  |  |  |
| CAN_CONFIRM_IMPORT | ✓ |  |  |  |  |  |
| CAN_MANAGE_ALIAS | ✓ | Optional |  |  |  |  |
| CAN_MANAGE_MASTER | ✓ |  |  |  |  |  |
| CAN_CHECKIN_WAREHOUSE |  |  |  | ✓ |  |  |
| CAN_COUNT |  |  |  | ✓ |  |  |
| CAN_CREATE_FIELD_MOVEMENT | ✓ | Optional |  | ✓ |  |  |
| CAN_CREATE_PENDING_INBOUND | ✓ | Optional |  | ✓ |  |  |
| CAN_CREATE_REJ | ✓ | Optional |  | ✓ |  |  |
| CAN_ASSIGN_TASK | ✓ | ✓ |  |  | Optional |  |
| CAN_RUN_STOCK_REPLAY | ✓ | Optional |  |  |  |  |
| CAN_RUN_RECONCILIATION | ✓ | Optional |  |  |  |  |
| CAN_REVIEW_ISSUE | ✓ | ✓ | ✓ |  | Optional | Readonly |
| CAN_CLOSE_ISSUE | ✓ | ✓ |  |  |  |  |
| CAN_VIEW_ERP_QUEUE | ✓ |  | ✓ |  |  |  |
| CAN_EXPORT | ✓ | Optional | ✓ |  | Optional |  |
| CAN_MARK_ERP_POSTED | ✓ |  | ✓ |  |  |  |
| CAN_MANAGE_NOTIFICATION | ✓ |  |  |  |  |  |
| CAN_CLOSE_DAY | ✓ | ✓ |  |  |  |  |
| CAN_LOCK_DAY | ✓ |  |  |  |  |  |
| CAN_UNLOCK_DAY | ✓ |  |  |  |  |  |
| CAN_RUN_RETENTION | ✓ |  |  |  |  |  |
| CAN_VIEW_AUDIT_LOG | ✓ |  |  |  |  |  |

## 6.2 Important Note

ตารางนี้เป็น default เท่านั้น ระบบต้องให้ Admin ปรับ permission ต่อ user ได้

---

# 7. Warehouse Scope

## 7.1 Purpose

Warehouse Scope จำกัดว่าผู้ใช้ทำงานกับ warehouse/location ไหนได้

ตัวอย่าง:

```text
User A ใช้ได้เฉพาะ WH_W8
User B ใช้ได้ WH_W8 และ WH_W9
Checker C ใช้ได้เฉพาะ warehouse ที่ check-in วันนี้
```

---

## 7.2 Scope Levels

| Scope Level | Description |
|---|---|
| ALL_WAREHOUSES | เห็น/ทำได้ทุกคลัง |
| WAREHOUSE | จำกัดเป็น warehouse |
| ZONE | จำกัดเป็น zone |
| LOCATION_GROUP | จำกัดเป็นกลุ่ม location |
| OWN_TASK_ONLY | เห็นเฉพาะ task ตัวเอง |

---

## 7.3 Warehouse Scope Rules

| Rule | Description |
|---|---|
| WS-001 | ทุก action ที่เกี่ยวกับ warehouse ต้องเช็ค scope |
| WS-002 | Checker เห็นเฉพาะ task ใน active warehouse session |
| WS-003 | Viewer อาจเห็น summary แต่ไม่เห็น detail ตาม permission |
| WS-004 | ERP Admin อาจเห็น ERP queue ทุก warehouse ที่ได้รับ scope |

---

# 8. Checker GPS Daily Session

## 8.1 Purpose

รองรับ checker outsource ที่มี 1–2 คน มาไม่ซ้ำกัน มีทั้งคนเก่า/ใหม่ และต้องจำสิทธิ์นับต่อ warehouse เป็นรายวัน

---

## 8.2 Session Rule

```text
1 user มี active warehouse ได้ 1 ที่ต่อ business date
ถ้า check-in warehouse ใหม่ session เดิมถูก suspend
```

---

## 8.3 Check-in Flow

```text
Checker login
→ กด Check-in
→ ระบบขอ GPS permission
→ บันทึก lat/lng/accuracy
→ ตรวจ warehouse radius optional
→ สร้าง CHECKER_DAILY_SESSION
→ user สามารถ count/task ใน warehouse นี้ได้จนสิ้น business date
```

---

## 8.4 Check-in Fields

| Field | Description |
|---|---|
| session_id | id |
| checker_user_id | user |
| business_date | วันที่ทำงาน |
| warehouse_id | warehouse ที่ check-in |
| checkin_lat | latitude |
| checkin_lng | longitude |
| checkin_accuracy | accuracy จาก device |
| checkin_at | เวลา check-in |
| device_id | device fingerprint optional |
| status | ACTIVE/SUSPENDED/EXPIRED |
| active_flag | true/false |
| suspended_reason | เหตุผลถ้าถูกพัก |
| last_seen_at | ล่าสุดที่ใช้งาน |

---

## 8.5 Session Status

| Status | Meaning |
|---|---|
| ACTIVE | ใช้งานได้ใน warehouse นี้ |
| SUSPENDED_BY_OTHER_WH_LOGIN | ถูกพักเพราะไป check-in warehouse อื่น |
| EXPIRED | หมดอายุสิ้น business date |
| CHECKED_OUT | user กดออกเอง |
| MANUAL_SUSPENDED | Admin พักสิทธิ์ |

---

## 8.6 GPS Rules

| Rule | Description |
|---|---|
| GPS-001 | ถ้าระบบเปิด GPS required ต้องมี lat/lng |
| GPS-002 | ต้องบันทึก accuracy ทุกครั้ง |
| GPS-003 | ถ้าอยู่นอก radius ให้ warning/block ตาม config |
| GPS-004 | GPS ไม่ใช่ proof absolute เพราะ spoof ได้ ต้องใช้เป็น control เบื้องต้น |

---

# 9. Segregation of Duties

## 9.1 Purpose

ลดความเสี่ยงจากคนคนเดียวทำทุกขั้นตอนเอง

---

## 9.2 Recommended Rules

| Rule | Description |
|---|---|
| SOD-001 | คนสร้าง user request ไม่ approve ตัวเอง |
| SOD-002 | Checker ไม่ approve issue ที่ตัวเองแจ้ง |
| SOD-003 | ERP Admin ไม่แก้ physical count โดยตรง |
| SOD-004 | คน close day ควรเห็น open critical issue ก่อน |
| SOD-005 | Unlock day ต้องเป็น Admin และต้องใส่ reason |
| SOD-006 | Export official file ต้องมี export log |

### MVP Note

SOD บางข้ออาจเป็น warning ใน MVP และค่อย strict ใน phase หลัง

---

# 10. Backend Permission Check Pattern

ทุก API สำคัญต้องเรียก permission check ก่อนทำงาน

Pseudo flow:

```text
Receive request
→ verify session
→ load user profile
→ load permissions
→ load warehouse scope
→ validate action permission
→ validate record scope
→ perform action
→ write audit log
→ return response
```

ตัวอย่าง action:

```text
import Inquiry
requires: CAN_IMPORT_INQUIRY + warehouse scope
```

```text
count location
requires: CAN_COUNT + active checker session + warehouse scope
```

```text
export issue report
requires: CAN_EXPORT + issue visibility scope
```

---

# 11. Frontend Access Rules

Frontend ต้องใช้ permission เพื่อ:

- ซ่อนเมนูที่ไม่เกี่ยวข้อง
- disable action button
- แสดงข้อความ permission denied
- redirect ถ้า session หมดอายุ

แต่ frontend ไม่ใช่แหล่งความปลอดภัยหลัก

```text
Frontend controls UX
Backend controls security
```

---

# 12. Audit Events for User & Permission

| Event | Trigger |
|---|---|
| USER_REGISTER | user สมัคร |
| USER_APPROVED | Admin approve |
| USER_REJECT_DELETE_REQUEST | Admin reject request |
| USER_LOCKED | user ถูก lock |
| USER_UNLOCKED | unlock user |
| USER_SUSPENDED | suspend user |
| PASSWORD_RESET | reset password |
| ROLE_ASSIGNED | assign role |
| ROLE_REMOVED | remove role |
| PERMISSION_OVERRIDE_ADDED | add override |
| PERMISSION_OVERRIDE_REMOVED | remove override |
| WAREHOUSE_SCOPE_CHANGED | เปลี่ยน scope |
| CHECKER_CHECKIN | checker check-in |
| CHECKER_SESSION_SUSPENDED | session ถูกพัก |

Audit log ต้องเก็บ:

- actor_id
- action
- target_user_id
- old_value
- new_value
- reason
- timestamp

---

# 13. Database Tables

## 13.1 users

| Field | Type | Notes |
|---|---|---|
| user_id | uuid/string | primary key |
| username | text | unique |
| email | text | optional/unique if used |
| phone | text | optional |
| full_name | text | required |
| hashed_password | text | required |
| salt | text | required |
| status | text | ACTIVE/LOCKED/etc |
| created_at | datetime |  |
| updated_at | datetime |  |
| last_login_at | datetime | optional |

---

## 13.2 user_access_requests

| Field | Type | Notes |
|---|---|---|
| request_id | uuid/string | primary key |
| username | text | requested username |
| email | text | optional |
| phone | text | optional |
| full_name | text | required |
| hashed_password | text | temporary hash until approved |
| salt | text |  |
| requested_role | text | optional |
| requested_warehouse | text | optional |
| status | text | PENDING_APPROVAL |
| requested_at | datetime |  |
| approved_by | user_id | nullable |
| approved_at | datetime | nullable |
| rejected_by | user_id | nullable |
| rejected_at | datetime | nullable |
| rejection_reason | text | nullable |

---

## 13.3 roles

| Field | Type |
|---|---|
| role_id | uuid/string |
| role_code | text |
| role_name | text |
| description | text |
| active | boolean |

---

## 13.4 permissions

| Field | Type |
|---|---|
| permission_id | uuid/string |
| permission_code | text |
| permission_name | text |
| module | text |
| description | text |
| active | boolean |

---

## 13.5 role_permissions

| Field | Type |
|---|---|
| role_id | uuid/string |
| permission_id | uuid/string |
| active | boolean |

---

## 13.6 user_roles

| Field | Type |
|---|---|
| user_id | uuid/string |
| role_id | uuid/string |
| active | boolean |
| assigned_by | user_id |
| assigned_at | datetime |

---

## 13.7 user_permission_overrides

| Field | Type | Notes |
|---|---|---|
| user_id | uuid/string |  |
| permission_id | uuid/string |  |
| override_type | text | ALLOW/DENY |
| reason | text |  |
| active | boolean |  |
| assigned_by | user_id |  |
| assigned_at | datetime |  |

---

## 13.8 user_warehouse_scope

| Field | Type |
|---|---|
| user_id | uuid/string |
| scope_type | text |
| warehouse_id | uuid/string |
| zone_id | uuid/string nullable |
| location_group_id | uuid/string nullable |
| active | boolean |
| assigned_by | user_id |
| assigned_at | datetime |

---

## 13.9 checker_daily_sessions

| Field | Type |
|---|---|
| session_id | uuid/string |
| checker_user_id | uuid/string |
| business_date | date |
| warehouse_id | uuid/string |
| checkin_lat | number |
| checkin_lng | number |
| checkin_accuracy | number |
| checkin_at | datetime |
| device_id | text |
| status | text |
| active_flag | boolean |
| suspended_reason | text |
| last_seen_at | datetime |

---

# 14. UI Pages

## 14.1 User Approval Center

### Purpose

จัดการ user สมัครใหม่

### Components

- Pending request table
- User detail drawer
- Approve modal
- Reject modal
- Role selector
- Permission selector
- Warehouse scope selector

### Actions

- Approve
- Reject & Delete Request
- View Detail

---

## 14.2 User & Permission Management

### Purpose

จัดการ user active

### Components

- User list
- Status badge
- Role tags
- Permission summary
- Warehouse scope summary
- User detail

### Actions

- Lock
- Unlock
- Suspend
- Reset Password
- Change Role
- Change Permission
- Change Scope

---

## 14.3 Checker Check-in Page

### Purpose

ให้ checker check-in warehouse ก่อนทำงาน

### Components

- Warehouse selector
- GPS permission button
- Check-in status
- Active session card
- Last check-in
- Warning if session suspended

---

# 15. API / Service Functions

## AuthService

- registerUser(input)
- login(username, password)
- logout(sessionToken)
- verifySession(sessionToken)
- resetPassword(userId)
- lockUser(userId, reason)
- unlockUser(userId, reason)

## UserApprovalService

- listPendingRequests()
- approveRequest(requestId, roles, permissions, scopes)
- rejectRequest(requestId, reason)

## PermissionService

- getUserPermissions(userId)
- hasPermission(userId, permissionCode)
- assertPermission(userId, permissionCode)
- assertWarehouseScope(userId, warehouseId)
- updateUserRoles(userId, roles)
- updateUserPermissionOverride(userId, permission, overrideType)

## CheckerSessionService

- checkIn(userId, warehouseId, gps)
- getActiveSession(userId, businessDate)
- suspendSession(sessionId, reason)
- expireDailySessions(businessDate)
- assertCanCount(userId, warehouseId)

---

# 16. Permission Error Codes

| Code | Meaning |
|---|---|
| AUTH_REQUIRED | ยังไม่ได้ login |
| SESSION_EXPIRED | session หมดอายุ |
| USER_NOT_ACTIVE | user ไม่ active |
| PENDING_APPROVAL | user ยังไม่อนุมัติ |
| PERMISSION_DENIED | ไม่มี permission |
| WAREHOUSE_SCOPE_DENIED | ไม่มีสิทธิ์ warehouse นี้ |
| CHECKER_SESSION_REQUIRED | checker ยังไม่ check-in |
| CHECKER_SESSION_SUSPENDED | session ถูกพัก |
| ADMIN_REQUIRED | ต้องเป็น Admin |

---

# 17. Acceptance Criteria

ระบบ user/permission ถือว่าเสร็จเมื่อ:

- [ ] User สมัครแล้วเข้า PENDING_APPROVAL
- [ ] Pending user login ใช้งานจริงไม่ได้
- [ ] Admin approve พร้อมเลือก role/permission/scope ได้
- [ ] Admin reject แล้ว request หายจาก queue แต่มี audit log
- [ ] Password ไม่เก็บ plain text
- [ ] User มีหลาย role ได้
- [ ] User มี permission override ได้
- [ ] Warehouse scope ถูกตรวจ backend
- [ ] Checker ต้อง check-in ก่อน count
- [ ] Check-in warehouse ใหม่ suspend session เดิม
- [ ] ทุก permission change มี audit log
- [ ] Frontend ซ่อนเมนูตาม permission
- [ ] Backend block action เมื่อไม่มี permission

---

# 18. Final Rule

```text
สิทธิ์ของระบบนี้ต้องออกแบบเพื่อป้องกันข้อมูลผิดตั้งแต่ต้น ไม่ใช่แค่ป้องกันคนเห็นเมนู
```

User, permission, warehouse scope และ checker session คือ control layer สำคัญของระ
