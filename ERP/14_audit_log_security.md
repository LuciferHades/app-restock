# 14_audit_log_security.md

# Audit Log & Security Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Audit Log / Security / Governance Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / Security Reviewer Use  
**Primary Use:** ใช้กำหนดมาตรฐาน audit log, security control, permission enforcement, session/token control, file security, notification secret handling, retention proof และ governance ของระบบ เพื่อให้ระบบตรวจย้อนหลังได้ ปลอดภัย และไม่เกิดช่องโหว่จากการใช้งานจริง

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Audit Log และ Security Rules ของ Warehouse Reconciliation ERP Platform

ระบบนี้เกี่ยวข้องกับข้อมูลสำคัญ เช่น:

- stock balance
- movement เข้า/ออก
- REJ / ของเสีย
- physical count
- ERP In-Out
- Inquiry snapshot
- user permission
- export report
- notification token
- retention/delete job
- daily close / lock

ถ้าระบบไม่มี audit และ security ที่ดี จะเกิดความเสี่ยง:

- ไม่รู้ว่าใครแก้ข้อมูล
- ไม่รู้ว่าใคร approve/reject user
- ไม่รู้ว่า import ไฟล์ไหนและใช้ version ไหน
- ไม่รู้ว่า alias ถูก map โดยใคร
- issue ถูกปิดโดยไม่มีเหตุผล
- export ข้อมูลออกไปโดยไม่มีหลักฐาน
- retention ลบข้อมูลสำคัญผิด
- user เห็น/แก้ warehouse ที่ไม่ควรเห็น
- token แจ้งเตือนรั่ว

หลักสำคัญ:

```text
ระบบนี้ต้องตรวจย้อนหลังได้ทุก critical action
และต้องไม่เชื่อ frontend เป็น security control หลัก
```

---

## 2. Security & Audit Philosophy

## 2.1 Audit Is Part of the Product, Not an Add-on

Audit log ไม่ใช่ feature เสริม แต่เป็นแกนควบคุมระบบ

ทุก action ที่เปลี่ยนข้อมูลสำคัญต้องตอบได้ว่า:

```text
ใครทำ
ทำอะไร
ทำเมื่อไหร่
ทำกับ record ไหน
ก่อนทำค่าเป็นอะไร
หลังทำค่าเป็นอะไร
เหตุผลคืออะไร
ทำผ่านช่องทางไหน
```

---

## 2.2 Backend Is the Security Boundary

Frontend ช่วยให้ UX ดีขึ้น เช่น ซ่อนเมนูหรือ disable ปุ่ม แต่ security จริงต้องอยู่ที่ backend

```text
Frontend = UX control
Backend = Security control
Database = Integrity control
Audit = Accountability control
```

---

## 2.3 Least Privilege

User ควรมีสิทธิ์เท่าที่จำเป็นต่อหน้าที่เท่านั้น

ตัวอย่าง:

- Checker ไม่ควร export official report
- ERP Admin ไม่ควรแก้ physical count
- Viewer ไม่ควรเห็นข้อมูล user permission
- Supervisor ไม่ควรจัดการ notification token
- Admin ต้องมี audit log ทุก action สำคัญ

---

## 2.4 No Silent Critical Change

ห้ามมีการเปลี่ยนข้อมูลสำคัญแบบเงียบ ๆ

Critical changes ต้องมี:

- permission check
- validation
- reason if required
- audit log
- notification if needed

---

# 3. Audit Log Core Requirements

## 3.1 Audit Log Must Answer

ทุก audit record ต้องตอบคำถามนี้ได้:

| Question | Required Field |
|---|---|
| ใครทำ | actor_id / actor_type |
| ทำอะไร | action |
| ทำกับอะไร | entity_type / entity_id |
| เมื่อไหร่ | created_at |
| ค่าเดิมคืออะไร | old_value_json |
| ค่าใหม่คืออะไร | new_value_json |
| ทำไม | reason |
| ทำจากไหน | ip_address / user_agent / device_id |
| สำคัญแค่ไหน | severity |

---

## 3.2 Audit Log Table

### Table: audit_logs

| Field | Type | Required | Description |
|---|---|---:|---|
| audit_id | uuid/text | Yes | primary key |
| actor_id | uuid/text | Yes | user_id หรือ SYSTEM |
| actor_type | text | Yes | USER / SYSTEM / SERVICE |
| action | text | Yes | event/action code |
| entity_type | text | Yes | table/module/entity |
| entity_id | text | No | id ของ record ที่ถูกกระทำ |
| old_value_json | json/text | No | ค่าก่อนเปลี่ยน |
| new_value_json | json/text | No | ค่าหลังเปลี่ยน |
| reason | text | No | เหตุผล ถ้ามี |
| ip_address | text | No | IP |
| user_agent | text | No | browser/device |
| device_id | text | No | device fingerprint optional |
| request_id | text | No | request trace id |
| severity | text | Yes | LOW / MEDIUM / HIGH / CRITICAL |
| created_at | datetime | Yes | timestamp |

---

## 3.3 Audit Severity

| Severity | Meaning | Examples |
|---|---|---|
| LOW | action ทั่วไป | login success, view log optional |
| MEDIUM | เปลี่ยนข้อมูลทั่วไป | update profile, update note |
| HIGH | กระทบ operation/report | confirm import, map alias, export |
| CRITICAL | กระทบ stock/security/delete/lock | retention delete, unlock day, permission change |

---

## 3.4 Audit Write Rule

Critical action ต้องเขียน audit log ใน transaction เดียวกัน หรือใน flow ที่รับประกันว่า audit ไม่หาย

ตัวอย่าง:

```text
Approve user
→ create user
→ assign role/scope
→ write audit
→ commit
```

ถ้า audit fail:

```text
ต้อง rollback หรือ retry ตาม policy
```

---

# 4. Actions Requiring Audit

## 4.1 User & Permission Audit

| Action | Severity | Required Reason |
|---|---|---:|
| USER_REGISTER | LOW | No |
| USER_APPROVE | HIGH | Optional |
| USER_REJECT_DELETE_REQUEST | HIGH | Recommended/Config |
| USER_LOCK | HIGH | Yes |
| USER_UNLOCK | HIGH | Yes |
| USER_SUSPEND | HIGH | Yes |
| USER_RESET_PASSWORD | HIGH | Optional |
| ROLE_ASSIGN | HIGH | Optional |
| ROLE_REMOVE | HIGH | Optional |
| PERMISSION_CHANGE | CRITICAL | Yes |
| WAREHOUSE_SCOPE_CHANGE | CRITICAL | Yes |

---

## 4.2 Auth / Session Audit

| Action | Severity | Notes |
|---|---|---|
| LOGIN_SUCCESS | LOW | include device/ip |
| LOGIN_FAILED | MEDIUM | include username/ip |
| LOGOUT | LOW |  |
| SESSION_EXPIRED | LOW | optional |
| SESSION_REVOKED | HIGH | if admin/system revoked |
| CHECKER_CHECKIN | MEDIUM | include GPS |
| CHECKER_SESSION_SUSPENDED | MEDIUM | when check-in other warehouse |
| CHECKER_CHECKOUT | LOW |  |

---

## 4.3 Import Audit

| Action | Severity | Required Details |
|---|---|---|
| IMPORT_BATCH_CREATED | HIGH | file_type, file_name, checksum |
| IMPORT_RAW_UPLOADED | MEDIUM | batch_id, chunk info |
| COLUMN_MAPPING_SAVED | HIGH | mapping old/new |
| IMPORT_NORMALIZE_RUN | HIGH | rows processed |
| IMPORT_CONFIRM | CRITICAL | batch_id, row_count, policy |
| IMPORT_CANCEL | HIGH | reason |
| IMPORT_FAILED | HIGH | error summary |
| DUPLICATE_IMPORT_OVERRIDE | CRITICAL | reason required |

---

## 4.4 Alias / Master Audit

| Action | Severity | Required Details |
|---|---|---|
| ALIAS_DETECTED | LOW/MEDIUM | source row/batch |
| ALIAS_MAP | HIGH | alias → canonical |
| ALIAS_CREATE_MASTER | HIGH | created master data |
| ALIAS_REJECT | MEDIUM | reason |
| ALIAS_UPDATE | HIGH | old/new |
| MASTER_CREATE | HIGH | master type/data |
| MASTER_UPDATE | HIGH | old/new |
| MASTER_INACTIVATE | HIGH | reason |
| MASTER_MERGE | CRITICAL | old/new target |

---

## 4.5 Operation Audit

| Action | Severity | Required Details |
|---|---|---|
| FIELD_MOVEMENT_CREATED | HIGH | movement data |
| FIELD_MOVEMENT_CANCELLED | HIGH | reason |
| PENDING_INBOUND_CREATED | HIGH | label/location/qty |
| PENDING_INBOUND_CONFIRMED | HIGH | canonical mapping |
| PENDING_INBOUND_REJECTED | HIGH | reason |
| PHYSICAL_COUNT_SUBMITTED | HIGH | count scope/qty |
| PHYSICAL_COUNT_VERIFIED | HIGH | verifier |
| REJ_NOTICE_CREATED | HIGH | qty/reason/location |
| REJ_MARKED_POSTED | HIGH | ERP reference |
| TRANSFER_CREATED | HIGH | from/to/qty |

---

## 4.6 Data Engine Audit

| Action | Severity | Required Details |
|---|---|---|
| LEDGER_ENTRY_CREATED | MEDIUM | source/link |
| LEDGER_ENTRY_CANCELLED | HIGH | reason |
| LEDGER_CORRECTION_CREATED | CRITICAL | reason |
| RUN_STOCK_REPLAY | HIGH | date range, rows |
| STOCK_REPLAY_FAILED | HIGH | error |
| RUN_RECONCILIATION | HIGH | date range, result counts |
| RECONCILIATION_FAILED | HIGH | error |
| COVERAGE_CHECK_RUN | MEDIUM | status |

---

## 4.7 Issue / Task Audit

| Action | Severity | Required Details |
|---|---|---|
| ISSUE_CREATED | HIGH | result_id, issue_type |
| ISSUE_ASSIGN | HIGH | assigned_to/owner |
| ISSUE_STATUS_CHANGE | HIGH | old/new status |
| ISSUE_COMMENT_ADD | LOW/MEDIUM | comment type |
| ISSUE_CLOSE | HIGH | reason required |
| ISSUE_REOPEN | HIGH | reason required |
| ISSUE_ACCEPT_DIFF | CRITICAL | reason + approver |
| CHECKER_TASK_CREATED | MEDIUM/HIGH | task type/scope |
| CHECKER_TASK_DONE | HIGH | result/count link |

---

## 4.8 Export Audit

| Action | Severity | Required Details |
|---|---|---|
| EXPORT_CREATE | HIGH | export type/filters/row count |
| EXPORT_DOWNLOAD | MEDIUM/HIGH | export_id/user |
| EXPORT_SEND | HIGH | recipient/channel |
| EXPORT_FAILED | MEDIUM/HIGH | error |
| ERP_TEMPLATE_EXPORT | CRITICAL | selected ids/filters |

---

## 4.9 Notification Audit

| Action | Severity | Required Details |
|---|---|---|
| NOTIFICATION_CHANNEL_CREATE | HIGH | channel type/name |
| NOTIFICATION_CHANNEL_UPDATE | HIGH | old/new without raw secret |
| NOTIFICATION_RULE_CHANGE | HIGH | old/new rule |
| NOTIFICATION_TEMPLATE_CHANGE | MEDIUM/HIGH | template change |
| NOTIFICATION_TEST_SEND | MEDIUM | channel/result |
| NOTIFICATION_SEND_FAILED | MEDIUM | error |

---

## 4.10 Close / Lock / Retention Audit

| Action | Severity | Required Details |
|---|---|---|
| DAILY_CLOSE | CRITICAL | date/warehouse/status/reason |
| DAILY_LOCK | CRITICAL | reason |
| DAILY_UNLOCK | CRITICAL | Admin + reason |
| CLOSE_WITH_WARNING_OVERRIDE | CRITICAL | warning list + reason |
| RETENTION_DRY_RUN | MEDIUM | cutoff/preview counts |
| RETENTION_DELETE | CRITICAL | deleted/skipped counts |
| RETENTION_FAILED | CRITICAL | error |
| BACKUP_EXPORT_CREATED | HIGH | file/scope |

---

# 5. Security Model

## 5.1 Security Layers

ระบบต้องมี security หลายชั้น:

```text
Authentication
→ Session Validation
→ User Status Check
→ Permission Check
→ Warehouse Scope Check
→ Operational Session Check
→ Input Validation
→ Audit Log
→ Database Constraint
```

---

## 5.2 Authentication Rules

### SEC-AUTH-001 — Password Hash + Salt

ห้ามเก็บ plain password

ต้องใช้:

```text
hash + salt
```

Recommended:

- bcrypt
- argon2
- PBKDF2 if environment limited

---

### SEC-AUTH-002 — Pending User Cannot Login

User ที่ยัง PENDING_APPROVAL ห้าม login ใช้งานจริง

---

### SEC-AUTH-003 — User Status Check

Block login ถ้า status เป็น:

```text
LOCKED
SUSPENDED
EXPIRED
DELETED
```

---

### SEC-AUTH-004 — Login Failure Control

ควรเก็บ login failed log และใน production ควรมี:

- rate limit
- temporary lock after repeated failures
- admin alert optional

---

# 6. Session & Token Security

## 6.1 Session Token Rules

- token ต้องหมดอายุ
- token ไม่ควรเก็บแบบ plain ใน database ถ้าเป็นไปได้ ให้เก็บ hash
- logout ต้อง revoke session
- password reset ควร revoke old sessions
- locked user ต้อง revoke active sessions

---

## 6.2 Session Expiry

Recommended MVP:

```text
Session expires after 8–24 hours
```

Checker daily session แยกจาก login session:

```text
Login session = เข้าระบบ
Checker daily session = สิทธิ์นับที่ warehouse ต่อวัน
```

---

## 6.3 Request Context

ทุก request หลัง auth ควรมี context:

```json
{
  "request_id": "REQ-001",
  "actor_id": "USR-001",
  "roles": ["ADMIN"],
  "permissions": ["CAN_IMPORT_INOUT"],
  "warehouse_scope": ["WH_W8"],
  "ip_address": "",
  "user_agent": ""
}
```

ใช้สำหรับ:

- permission check
- audit log
- warehouse scope
- data filtering
- error tracing

---

# 7. Permission Security

## 7.1 Backend Permission Enforcement

ทุก write/action API ต้องเรียก:

```text
PermissionService.assertPermission(actor_id, permission_code)
```

และถ้าเกี่ยวกับ warehouse:

```text
PermissionService.assertWarehouseScope(actor_id, warehouse_id)
```

---

## 7.2 Frontend Permission Is Not Enough

ห้ามถือว่า hide menu = secure

ถ้า user ยิง API ตรง backend ต้อง block

---

## 7.3 Permission Override Priority

Recommended:

```text
DENY override wins over ALLOW
```

---

## 7.4 Sensitive Permissions

Permission ต่อไปนี้ถือว่า sensitive:

- CAN_MANAGE_USER
- CAN_MANAGE_PERMISSION
- CAN_IMPORT_INQUIRY
- CAN_IMPORT_INOUT
- CAN_CONFIRM_IMPORT
- CAN_MANAGE_ALIAS
- CAN_RUN_RECONCILIATION
- CAN_ACCEPT_DIFF
- CAN_EXPORT
- CAN_EXPORT_ERP_TEMPLATE
- CAN_CLOSE_DAY
- CAN_LOCK_DAY
- CAN_UNLOCK_DAY
- CAN_RUN_RETENTION
- CAN_VIEW_AUDIT_LOG
- CAN_MANAGE_NOTIFICATION

Sensitive permission change ต้อง audit severity CRITICAL

---

# 8. Warehouse Scope Security

## 8.1 Scope Required for Warehouse Data

ทุก request ที่เกี่ยวข้องกับ warehouse ต้องตรวจ warehouse scope

Applies to:

- import data
- view issue
- count
- movement
- export
- close day
- stock replay
- reconciliation

---

## 8.2 Checker Own Scope

Checker เห็นเฉพาะ:

- active warehouse session
- task ของตัวเอง หรือ task ใน active warehouse ตาม config
- submission history ของตัวเอง

---

## 8.3 Viewer Scope

Viewer ควรเห็น summary ตาม scope เท่านั้น ไม่เห็น raw detail ถ้าไม่ได้รับ permission

---

# 9. Checker GPS Session Security

## 9.1 GPS Is Control, Not Absolute Proof

GPS ช่วยลดความเสี่ยง แต่ไม่ควรถือเป็น proof absolute เพราะ spoof ได้

ระบบต้องเก็บ:

- lat
- lng
- accuracy
- timestamp
- device_id optional

---

## 9.2 One Active Warehouse Rule

Checker 1 คน active ได้ 1 warehouse ต่อ business_date

ถ้า check-in ใหม่:

```text
Suspend old session
Create new active session
Audit both
```

---

## 9.3 Count Requires Active Session

ก่อน submit count ต้องตรวจ:

- login session active
- user active
- CAN_COUNT
- active checker_daily_session
- session warehouse ตรงกับ count warehouse

---

# 10. File & Upload Security

## 10.1 File Type Validation

Allowed:

```text
.xlsx
.csv
```

Optional later:

```text
.jpg/.png for photo evidence
```

ต้อง reject file type ที่ไม่รองรับ

---

## 10.2 File Size Limit

ต้องมี limit เพื่อกัน timeout/storage abuse

Recommended MVP:

- warning > 5MB
- chunk upload for large file
- hard limit configurable

---

## 10.3 Raw File Handling

Raw row ต้องเก็บใน import_raw_rows โดยมี batch_id

ห้ามเอา raw file ไปใช้คำนวณโดยตรงแบบไม่มี mapping/normalize

---

## 10.4 Photo Evidence Security

ถ้ามีรูปภาพ:

- store in private bucket/storage if possible
- file URL ไม่ควร public ถ้าไม่จำเป็น
- link access ต้องเช็ค permission
- avoid exposing raw storage token

---

## 10.5 Export File Security

Export file ควร:

- มี expiration URL ถ้าใช้ cloud storage
- ไม่ public ถ้าเป็น official report
- log ทุก download
- filter data ตาม permission/scope

---

# 11. Secret & Token Security

## 11.1 Notification Token Handling

Token สำหรับ Telegram / LINE / Email ต้องไม่ถูกส่งกลับ frontend แบบ raw

เก็บเป็น:

```text
token_ref
```

และ backend ใช้ token_ref ไปอ่าน secret จาก secure environment/config

---

## 11.2 Secrets Never in Audit Log

Audit log ห้ามเก็บ raw token/password/secret

ถ้าจำเป็นให้เก็บ:

```text
masked value
last 4 chars
secret reference
```

---

## 11.3 Environment Variables

Secrets ควรอยู่ใน environment variables หรือ secret manager

Examples:

- TELEGRAM_BOT_TOKEN
- LINE_CHANNEL_ACCESS_TOKEN
- EMAIL_SMTP_PASSWORD
- DATABASE_URL
- JWT_SECRET

---

# 12. Data Security

## 12.1 Data Classification

| Data Type | Sensitivity | Notes |
|---|---|---|
| Password hash | High | never expose |
| Notification token | High | never expose |
| User permission | High | Admin only |
| Audit log | High | Admin only |
| Stock movement | Medium/High | scope by warehouse |
| Issue detail | Medium/High | role/scope-based |
| Summary dashboard | Medium | scope-based |
| Export file | High | log + permission |
| Photo evidence | Medium/High | private access |

---

## 12.2 PII Handling

User data เช่น full_name, phone, email ต้องใช้เท่าที่จำเป็น และไม่ export โดยไม่จำเป็น

---

## 12.3 Data Minimization

Dashboard/Checker mobile ไม่ควรโหลดข้อมูลเกินจำเป็น

ตัวอย่าง:

```text
Checker ไม่ต้องโหลด raw import rows
Viewer ไม่ต้องโหลด audit log
```

---

# 13. Database Security

## 13.1 Database Access

แนะนำให้ backend/service layer เป็นตัวเข้าถึง database โดยตรง

ถ้าใช้ Supabase:

- เปิด RLS สำหรับ tables สำคัญ
- service role key ใช้เฉพาะ backend/server-side
- client key ต้องไม่ให้สิทธิ์เขียนตารางสำคัญโดยตรง

---

## 13.2 Row Level Security Concept

RLS policy ควรยึด:

- user_id
- role
- permission
- warehouse scope
- owner/task assignment

---

## 13.3 Database Constraints

ควรมี constraints:

- unique username
- unique role_code
- unique permission_code
- foreign key where practical
- not null for required fields
- status enums/check constraints if supported
- indexes for date/warehouse/status

---

# 14. Locked Day Security

## 14.1 Locked Day Protection

ถ้า daily_close_status = LOCKED:

ห้ามแก้โดยตรง:

- import confirmed data
- field movement
- physical count
- ledger
- daily stock position
- recon result

ต้องใช้:

```text
Correction / Investigation Flow
```

---

## 14.2 Unlock Requires Reason

Unlock ต้องมี:

- CAN_UNLOCK_DAY
- Admin role or explicit permission
- reason
- audit CRITICAL
- optional notification

---

# 15. Retention Security

## 15.1 Retention Must Not Destroy Evidence

Retention 45 วันต้องไม่ลบหลักฐานที่ยังจำเป็นต่อ open issue / audit / export

ห้ามลบ:

- open issue
- issue-linked source data if needed
- critical audit log
- export log
- daily close summary
- retention log
- pending inbound unresolved
- REJ_NOT_POSTED unresolved

---

## 15.2 Dry Run Recommended

Manual retention ควรเริ่มจาก DRY_RUN เพื่อ preview:

- table
- delete count
- skip count
- skip reason

---

## 15.3 Retention Audit

Retention execute ต้องมี:

- retention_job
- retention_log per table
- audit RETENTION_DELETE
- notification summary

---

# 16. Export Security

## 16.1 Export Permission

ทุก export ต้องตรวจ:

- login session
- user active
- CAN_EXPORT or specific export permission
- warehouse/data scope

---

## 16.2 Export Scope Required

Export ต้องมี filter หรือ selected rows เพื่อกัน export ข้อมูลเกินจำเป็น

---

## 16.3 Official ERP Template

ERP template export ถือว่า sensitive เพราะอาจนำไปใช้ post/adjust ERP

ต้องมี:

- CAN_EXPORT_ERP_TEMPLATE
- selected rows or clear filters
- export log
- audit severity CRITICAL

---

# 17. Notification Security

## 17.1 Notification Payload Minimization

Notification ไม่ควรใส่ข้อมูลละเอียดเกินจำเป็น โดยเฉพาะช่องทาง external เช่น Telegram/LINE

ควรใส่:

- event type
- issue id
- warehouse/date
- summary
- link to system detail

ไม่ควรใส่:

- full raw row
- user sensitive data
- secret/token
- large detailed stock dump

---

## 17.2 Anti-Spam Control

ต้องมี:

- cooldown_minutes
- send_once_per_issue
- daily digest optional
- retry limit

---

## 17.3 Notification Log

ทุก send/skipped/failed ต้องมี notification_log

---

# 18. Security Error Handling

## 18.1 Do Not Leak Sensitive Detail

Login error ไม่ควรบอกว่า username มีจริงหรือ password ผิดอย่างละเอียดเกินไป

ใช้:

```text
username หรือ password ไม่ถูกต้อง
```

---

## 18.2 Permission Error Message

บอก user ชัดเจนแต่ไม่เปิดข้อมูลเกินจำเป็น:

```text
คุณไม่มีสิทธิ์ใช้งานส่วนนี้ โปรดติดต่อ Admin
```

---

## 18.3 System Error

ไม่ควรแสดง stack trace ให้ user

ให้แสดง:

```text
เกิดข้อผิดพลาดในการประมวลผล โปรดลองใหม่ หรือแจ้ง Admin พร้อม error code
```

---

# 19. Audit Log Retention

## 19.1 Critical Audit Retention

Critical audit log ควรเก็บนานกว่า raw data 45 วัน

อย่างน้อยควรเก็บ:

- permission change
- import confirm
- alias map
- issue accept diff
- daily close/lock/unlock
- export ERP template
- retention delete

---

## 19.2 Audit Log Query Access

เฉพาะผู้มีสิทธิ์:

```text
CAN_VIEW_AUDIT_LOG
```

ควรสามารถ filter:

- actor
- action
- entity_type
- entity_id
- date range
- severity

---

# 20. Security Test Cases

## 20.1 Auth Tests

- pending user login ต้องถูก block
- locked user login ต้องถูก block
- wrong password log failed login
- password ไม่ถูกเก็บ plain text
- logout revoke session

## 20.2 Permission Tests

- user ไม่มี CAN_EXPORT ยิง API export ต้อง block
- checker ไม่มี active session submit count ต้อง block
- user warehouse scope WH_W8 ดู WH_W9 ต้อง block
- viewer ยิง API import ต้อง block

## 20.3 Audit Tests

- approve user มี audit
- reject user request มี audit แม้ request ถูกลบ
- confirm import มี audit
- map alias มี audit old/new
- close issue ไม่มี reason ต้อง block
- retention execute มี retention log + audit

## 20.4 File Security Tests

- upload invalid file type ต้อง block
- duplicate checksum warning
- export official ต้องมี permission + log
- photo URL access ต้องตรวจ permission

## 20.5 Retention Security Tests

- open issue source ไม่ถูกลบ
- closed old raw ถูกลบหลัง summary
- retention dry run ไม่ลบจริง
- retention failed ส่ง notification/admin log

---

# 21. Security Acceptance Criteria

Audit & Security ถือว่า complete ถ้า:

- [ ] Password ไม่เก็บ plain text
- [ ] Pending/locked/suspended user login ไม่ได้
- [ ] Session/token หมดอายุและ revoke ได้
- [ ] Backend ตรวจ permission ทุก write/action API
- [ ] Warehouse scope ถูกตรวจทุก request ที่เกี่ยวข้อง
- [ ] Checker ต้องมี active session ก่อน count
- [ ] Secret/token ไม่ถูก expose ไป frontend/audit log
- [ ] Export ทุกครั้งมี export log
- [ ] Official ERP export ต้องมี permission เฉพาะ
- [ ] ทุก critical write มี audit log
- [ ] Audit log มี actor/action/entity/old/new/reason/timestamp
- [ ] Reject user request ลบ queue แต่ audit ยังอยู่
- [ ] Locked day แก้ตรงไม่ได้
- [ ] Unlock day ต้องมี Admin + reason + audit
- [ ] Retention ไม่ลบ open issue/critical audit/export log
- [ ] Retention execute มี retention log + notification summary
- [ ] Error message ไม่ leak sensitive detail
- [ ] Security test cases ผ่าน

---

# 22. Final Audit & Security Statement

```text
Audit Log & Security ของ Warehouse Reconciliation ERP Platform ต้องออกแบบให้ทุก critical action ตรวจ permission, warehouse scope, validation และเขียน audit log เสมอ โดยป้องกัน password/token/secret รั่ว ป้องกัน user เข้าถึงข้อมูลนอก scope ป้องกันการแก้ locked day และป้องกัน retention ลบหลักฐานสำคัญ เพื่อให้ระบบใช้งานจริงได้อย่างมั่นคง ตรวจย้อนหลังได้ และปลอดภัยในระดับ operation/ERP control
```

