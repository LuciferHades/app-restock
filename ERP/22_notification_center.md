# 22_notification_center.md

# Notification Center Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** Notification / Alert / Escalation / Message Rule Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / System Admin Use  
**Primary Use:** ใช้กำหนดระบบแจ้งเตือน In-app, Telegram, LINE Messaging API, Email, rule, template, cooldown, send-once, escalation, notification log และ admin control เพื่อให้ระบบแจ้งเตือนเฉพาะเรื่องที่ควรรู้ ไม่ spam และตรวจย้อนหลังได้

---

## 1. Purpose of This Document

เอกสารนี้กำหนด Notification Center ของ Warehouse Reconciliation ERP Platform

ระบบนี้ต้องแจ้งเตือนเหตุการณ์สำคัญ เช่น:

- user สมัครใหม่รออนุมัติ
- import failed
- unknown alias เกิดขึ้น
- pending inbound ต้อง review
- system missing
- qty diff
- location diff
- REJ ยังไม่ post ใน ERP
- reconciliation failed
- close day not ready
- export ready
- retention failed
- DB usage ใกล้เต็ม

แต่ notification ต้องไม่ทำให้คนรำคาญหรือ ignore ระบบ ดังนั้นต้องมี:

```text
Rule-based notification
Template-based message
Severity filter
Cooldown
Send once per issue
Digest option
Notification log
Retry / failure handling
Admin-editable config
```

หลักสำคัญ:

```text
Notification ต้องช่วยให้คนที่ถูกต้องรู้เรื่องที่ต้อง action ในเวลาที่เหมาะสม โดยไม่ spam และต้องตรวจย้อนหลังได้ว่าส่งอะไร ส่งให้ใคร สำเร็จหรือไม่
```

---

## 2. Notification Philosophy

## 2.1 Notify for Action, Not Noise

แจ้งเตือนเฉพาะสิ่งที่ต้อง action หรือเสี่ยงจริง

ตัวอย่างที่ควรแจ้ง:

```text
REJ_NOT_POSTED severity HIGH → ERP Admin ต้องตรวจ ERP
RETENTION_FAILED → Admin ต้องแก้ระบบ
USER_PENDING_APPROVAL → Admin ต้อง approve/reject
```

ตัวอย่างที่ไม่ควรแจ้งทันทีทุกครั้ง:

```text
MATCHED result
Low severity result จำนวนมาก
Import warning เล็กน้อยทุก row
```

---

## 2.2 Rule-Based, Not Hardcoded

ห้ามกระจาย logic ส่ง notification ไปตาม service หลายจุดแบบ hardcode

ต้องใช้:

```text
notification_channels
notification_rules
notification_templates
notification_logs
NotificationService.notify(event_code, payload)
```

---

## 2.3 External Message Should Be Compact

Telegram / LINE / Email ไม่ควรส่ง raw detail ใหญ่เกินไป

ควรส่ง:

- event title
- severity
- warehouse/date
- issue id / batch id
- short summary
- link/detail reference
- action owner

ไม่ควรส่ง:

- raw row ทั้งแถวจำนวนมาก
- token/secret
- password/user sensitive data
- stock dump ยาว ๆ

---

## 2.4 Every Notification Must Be Logged

ทุกครั้งที่ send / skip / fail ต้องมี log

Log ต้องตอบได้ว่า:

```text
event อะไร
ส่งช่องทางไหน
ส่งให้ใคร
ข้อความ preview คืออะไร
สำเร็จไหม
fail เพราะอะไร
ส่งเมื่อไหร่
retry กี่ครั้ง
```

---

# 3. Supported Channels

Notification Center ต้องรองรับ channel เหล่านี้:

| Channel | MVP | Purpose |
|---|---:|---|
| In-app Alert | Yes | แจ้งในระบบแบบปลอดภัยสุด |
| Telegram | Yes | แจ้ง group/admin ได้เร็ว |
| LINE Messaging API | Yes | รองรับ LINE ระยะยาว |
| Email | Basic | รายงาน/summary/backup |

---

## 3.1 Important LINE Note

```text
LINE Notify ไม่ควรใช้เป็นช่องทางหลักสำหรับระบบใหม่
ควรออกแบบให้ใช้ LINE Messaging API แทน
```

เหตุผล:

- ระบบใหม่ควรรองรับ Messaging API ที่จัดการ target/channel ได้ยืดหยุ่นกว่า
- ควรเก็บ token แบบ secure server-side
- ต้องมี notification log และ retry control

---

## 3.2 In-App Alert

ใช้สำหรับ:

- user pending approval
- assigned issue
- export ready
- failed job
- close readiness blocker

ข้อดี:

- ปลอดภัยกว่า external channel
- ใส่ link ไปหน้า detail ได้
- ควบคุม permission ได้

---

## 3.3 Telegram

ใช้สำหรับ:

- Admin group alert
- ERP Admin alert
- urgent issue
- retention/system health

ข้อควรระวัง:

- token ห้ามอยู่ frontend
- chat_id ต้องเก็บใน channel config
- ไม่ควรส่งข้อมูล sensitive เต็มรูปแบบ

---

## 3.4 LINE Messaging API

ใช้สำหรับ:

- กลุ่ม operation ที่ใช้ LINE เป็นหลัก
- แจ้ง issue/approval/summary
- daily digest

ข้อควรระวัง:

- channel access token ต้อง secure
- target id ต้อง config โดย Admin
- ควรมี test send

---

## 3.5 Email

ใช้สำหรับ:

- daily summary
- export ready link
- retention/backup report
- critical system failure summary

MVP ทำ basic ได้ก่อน

---

# 4. Notification Data Model

## 4.1 notification_channels

### Purpose

เก็บ channel ที่ระบบสามารถส่งได้

| Field | Type | Required | Description |
|---|---|---:|---|
| channel_id | uuid/text | Yes | primary key |
| channel_type | text | Yes | IN_APP / TELEGRAM / LINE_MESSAGING / EMAIL |
| channel_name | text | Yes | ชื่อที่ Admin เห็น |
| token_ref | text | No | reference ไป secret server-side |
| target_id | text | No | chat_id / group_id / email / user_id |
| target_role | text | No | ถ้าส่งตาม role |
| active_flag | boolean | Yes | เปิดใช้งานไหม |
| created_by | text | Yes | user/system |
| created_at | datetime | Yes | created time |
| updated_by | text | No | last updater |
| updated_at | datetime | No | updated time |

### Rules

- ห้ามเก็บ raw token ที่เปิดเผยใน frontend
- token_ref เท่านั้นที่ควรแสดงใน UI
- channel inactive ห้ามส่ง

---

## 4.2 notification_rules

### Purpose

กำหนดว่า event ไหนต้องส่งให้ใคร ผ่านช่องทางไหน

| Field | Type | Required | Description |
|---|---|---:|---|
| rule_id | uuid/text | Yes | primary key |
| event_code | text | Yes | event ที่ trigger |
| rule_name | text | Yes | display name |
| severity_filter | json/text | No | เช่น CRITICAL,HIGH |
| warehouse_filter | json/text | No | optional warehouse scope |
| product_group_filter | json/text | No | optional |
| target_role | text | No | ADMIN / ERP_ADMIN / SUPERVISOR |
| channel_id | uuid/text | Yes | FK notification_channels |
| template_id | uuid/text | Yes | FK notification_templates |
| cooldown_minutes | number | Yes | anti-spam |
| send_once_per_issue | boolean | Yes | ป้องกันซ้ำต่อ issue |
| digest_mode | text | No | NONE / DAILY / HOURLY |
| active_flag | boolean | Yes | enable/disable |
| created_by | text | Yes | creator |
| created_at | datetime | Yes | created time |
| updated_by | text | No | updater |
| updated_at | datetime | No | updated time |

### Rules

- Critical event ควรส่งทันทีถ้า rule active
- Medium/Low event อาจใช้ digest
- ต้องมี cooldown หรือ send_once_per_issue อย่างน้อยหนึ่งอย่างสำหรับ event ที่เกิดซ้ำได้

---

## 4.3 notification_templates

### Purpose

เก็บข้อความ template แยกจาก logic

| Field | Type | Required | Description |
|---|---|---:|---|
| template_id | uuid/text | Yes | primary key |
| template_name | text | Yes | display name |
| event_code | text | Yes | event ที่ใช้ |
| title_template | text | Yes | title |
| body_template | text | Yes | body with variables |
| channel_type | text | No | specific channel optional |
| active_flag | boolean | Yes | active |
| created_by | text | Yes | creator |
| created_at | datetime | Yes | created |
| updated_at | datetime | No | updated |

### Example Variables

```text
{{business_date}}
{{warehouse}}
{{issue_id}}
{{issue_type}}
{{severity}}
{{location}}
{{customer}}
{{item}}
{{qty}}
{{diff_qty}}
{{suggested_action}}
{{owner_role}}
{{link}}
```

---

## 4.4 notification_logs

### Purpose

เก็บประวัติการส่ง/skip/fail ทุก notification

| Field | Type | Required | Description |
|---|---|---:|---|
| notification_log_id | uuid/text | Yes | primary key |
| event_code | text | Yes | event |
| rule_id | uuid/text | No | applied rule |
| channel_id | uuid/text | No | channel |
| channel_type | text | Yes | IN_APP / TELEGRAM / LINE / EMAIL |
| target | text | No | target id/email/user/role |
| issue_id | uuid/text | No | linked issue |
| batch_id | uuid/text | No | linked import batch |
| job_id | uuid/text | No | linked job |
| severity | text | No | severity |
| message_title | text | No | rendered title |
| message_preview | text | No | preview body |
| status | text | Yes | SENT / FAILED / SKIPPED / RETRYING |
| skip_reason | text | No | COOLDOWN / SEND_ONCE / RULE_INACTIVE |
| error_message | text | No | failure reason |
| retry_count | number | Yes | retry count |
| sent_at | datetime | No | sent time |
| created_at | datetime | Yes | created time |

---

## 4.5 in_app_notifications

Optional but recommended for MVP

| Field | Type | Required | Description |
|---|---|---:|---|
| in_app_id | uuid/text | Yes | primary key |
| user_id | uuid/text | Yes | receiver |
| event_code | text | Yes | event |
| title | text | Yes | title |
| body | text | Yes | body |
| link_url | text | No | internal link |
| severity | text | No | severity |
| read_flag | boolean | Yes | read/unread |
| read_at | datetime | No | read time |
| created_at | datetime | Yes | created time |

---

# 5. Event Codes

## 5.1 User / Permission Events

```text
USER_PENDING_APPROVAL
USER_APPROVED
USER_REJECTED
USER_LOCKED
PERMISSION_CHANGED
WAREHOUSE_SCOPE_CHANGED
```

---

## 5.2 Import / Alias Events

```text
IMPORT_FAILED
IMPORT_CONFIRMED
IMPORT_WARNING
UNKNOWN_ALIAS_CREATED
ALIAS_REVIEW_REQUIRED
ALIAS_MAPPED
DUPLICATE_IMPORT_WARNING
```

---

## 5.3 Operation Events

```text
PENDING_INBOUND_CREATED
PENDING_INBOUND_CANDIDATE_FOUND
PENDING_INBOUND_CONFIRMED
FIELD_MOVEMENT_CREATED
PHYSICAL_COUNT_SUBMITTED
REJ_NOTICE_CREATED
REJ_NOT_POSTED
```

---

## 5.4 Reconciliation / Issue Events

```text
SYSTEM_MISSING
FIELD_MISSING
QTY_DIFF
LOCATION_DIFF
ITEM_DIFF
LOT_DIFF
NEED_RECOUNT
NEED_INVESTIGATION
ISSUE_CREATED
ISSUE_ASSIGNED
ISSUE_OVER_SLA
ISSUE_CLOSED
```

---

## 5.5 Job / Maintenance Events

```text
BACKFILL_COMPLETED
BACKFILL_FAILED
STOCK_REPLAY_FAILED
RECONCILIATION_FAILED
DAILY_CLOSE_NOT_READY
DAILY_CLOSED
DAILY_LOCKED
EXPORT_READY
EXPORT_FAILED
RETENTION_COMPLETED
RETENTION_COMPLETED_WITH_WARNING
RETENTION_FAILED
DB_USAGE_WARNING
DB_USAGE_CRITICAL
HEALTH_CHECK_CRITICAL
```

---

# 6. Event Priority Matrix

| Event | Default Severity | Default Target | Send Mode | MVP |
|---|---|---|---|---:|
| USER_PENDING_APPROVAL | MEDIUM | ADMIN | immediate | Yes |
| IMPORT_FAILED | HIGH | ADMIN | immediate | Yes |
| UNKNOWN_ALIAS_CREATED | MEDIUM | ADMIN | digest/immediate if blocking | Yes |
| PENDING_INBOUND_CREATED | MEDIUM | SUPERVISOR/ADMIN | immediate/digest | Yes |
| SYSTEM_MISSING | HIGH | ERP_ADMIN | immediate if high qty | Yes |
| QTY_DIFF | HIGH/MEDIUM | SUPERVISOR | immediate/digest | Yes |
| LOCATION_DIFF | MEDIUM | SUPERVISOR/CHECKER | digest/task | Yes |
| REJ_NOT_POSTED | HIGH/CRITICAL | ERP_ADMIN | immediate | Yes |
| ISSUE_OVER_SLA | HIGH | OWNER/SUPERVISOR | immediate | Basic |
| EXPORT_READY | LOW/MEDIUM | requester | immediate | Yes |
| RETENTION_FAILED | CRITICAL | ADMIN | immediate | Yes |
| DB_USAGE_CRITICAL | CRITICAL | ADMIN | immediate | Yes |
| DAILY_CLOSE_NOT_READY | HIGH | ADMIN/SUPERVISOR | immediate | Yes |

---

# 7. Notification Rule Logic

## 7.1 notify(event_code, payload)

Internal function:

```text
NotificationService.notify(event_code, payload)
```

Flow:

```text
Receive event_code + payload
→ Load active notification_rules by event_code
→ Filter by severity / warehouse / product_group
→ Resolve target users or target channel
→ Check cooldown
→ Check send_once_per_issue
→ Render template
→ Send channel
→ Write notification_log
→ If failed, mark FAILED and schedule retry if allowed
```

---

## 7.2 Cooldown Logic

ถ้า rule มี cooldown_minutes:

```text
Find last SENT/RETRYING log for same rule + event + target + issue_id/source_id
If within cooldown → SKIPPED with skip_reason = COOLDOWN
```

---

## 7.3 Send Once Per Issue Logic

ถ้า send_once_per_issue = true:

```text
If notification_log exists for same rule_id + issue_id + status SENT
→ SKIPPED with skip_reason = SEND_ONCE_PER_ISSUE
```

---

## 7.4 Digest Logic

Digest mode ใช้กับ event ที่เกิดบ่อย เช่น:

- unknown alias
- medium QTY_DIFF
- location diff
- low priority issue

Digest should group by:

- event_code
- business_date
- warehouse
- severity
- owner_role

Digest message example:

```text
Daily Issue Digest
Warehouse: WH_W8
Date: 2026-05-18
Unknown Alias: 8
QTY_DIFF: 5
LOCATION_DIFF: 3
Pending Inbound: 2
```

---

# 8. Message Template Examples

## 8.1 USER_PENDING_APPROVAL

Title:

```text
มีผู้ใช้รออนุมัติ
```

Body:

```text
{{full_name}} สมัครใช้งานระบบและรอ Admin อนุมัติ
Role ที่ขอ: {{requested_role}}
Warehouse: {{requested_warehouse}}
```

Action:

```text
เปิด User Approval Center
```

---

## 8.2 UNKNOWN_ALIAS_CREATED

Title:

```text
พบ Alias ที่ยังไม่รู้จัก
```

Body:

```text
ระบบพบข้อความ "{{raw_text}}" ประเภท {{alias_type}}
Batch: {{batch_id}}
จำนวนครั้งที่พบ: {{occurrence_count}}
โปรด review เพื่อให้ระบบ reconcile ได้ถูกต้อง
```

Action:

```text
เปิด Alias Review
```

---

## 8.3 PENDING_INBOUND_CREATED

Title:

```text
มี Pending Inbound รอ confirm
```

Body:

```text
วันที่ {{business_date}} คลัง {{warehouse}}
Location: {{location}}
ข้อความป้าย: {{text_on_label}}
Qty: {{qty}} {{uom}}
โปรดรอ/เทียบ In-Out หรือ confirm ข้อมูล
```

Action:

```text
เปิด Pending Inbound Detail
```

---

## 8.4 SYSTEM_MISSING

Title:

```text
พบรายการที่หน้างานแจ้ง แต่ ERP ยังไม่พบ
```

Body:

```text
Issue: {{issue_id}}
วันที่ {{business_date}} คลัง {{warehouse}}
Location: {{location}}
Item: {{item}}
Qty: {{qty}} {{uom}}
Suggested action: {{suggested_action}}
```

Action:

```text
เปิด Issue Detail
```

---

## 8.5 QTY_DIFF

Title:

```text
พบยอดไม่ตรง
```

Body:

```text
Issue: {{issue_id}}
วันที่ {{business_date}} คลัง {{warehouse}}
Item: {{item}}
Field Qty: {{field_qty}}
System Qty: {{system_qty}}
Diff: {{diff_qty}} {{uom}}
Suggested action: {{suggested_action}}
```

---

## 8.6 REJ_NOT_POSTED

Title:

```text
REJ ยังไม่ถูก post ใน ERP
```

Body:

```text
Issue: {{issue_id}}
วันที่ {{business_date}} คลัง {{warehouse}}
From: {{from_location}}
REJ Location: {{rej_location}}
Item: {{item}}
Qty: {{qty}} {{uom}}
Reason: {{reason}}
ERP Admin โปรดตรวจสอบการ post
```

---

## 8.7 DAILY_CLOSE_NOT_READY

Title:

```text
ยังไม่พร้อมปิดวัน
```

Body:

```text
วันที่ {{business_date}} คลัง {{warehouse}}
สถานะ: {{readiness_status}}
Blockers:
{{blocker_summary}}
```

---

## 8.8 EXPORT_READY

Title:

```text
Export พร้อมแล้ว
```

Body:

```text
ไฟล์ {{export_type}} พร้อมใช้งาน
Rows: {{row_count}}
สร้างโดย: {{exported_by}}
```

---

## 8.9 RETENTION_FAILED

Title:

```text
Retention Cleanup ล้มเหลว
```

Body:

```text
Cutoff: {{cutoff_date}}
Job: {{retention_job_id}}
Error: {{error_message}}
โปรดตรวจ Retention Log
```

---

## 8.10 DB_USAGE_CRITICAL

Title:

```text
พื้นที่ฐานข้อมูลใกล้เต็ม
```

Body:

```text
DB Usage: {{db_usage_percent}}%
สถานะ: {{health_status}}
โปรดตรวจ retention, archive หรือ backup
```

---

# 9. Notification Center UI

## 9.1 Main Tabs

Notification Center ควรมี tabs:

```text
1. Channels
2. Rules
3. Templates
4. Logs
5. Test Send
```

---

## 9.2 Channels Tab

แสดง:

- channel name
- channel type
- target id/role
- active flag
- last test status
- updated at

Actions:

- add channel
- edit channel
- deactivate channel
- test send

Security:

- token_ref แสดงได้
- raw token ห้ามแสดง

---

## 9.3 Rules Tab

แสดง:

- rule name
- event code
- severity filter
- target role/channel
- cooldown
- send once per issue
- digest mode
- active flag

Actions:

- create rule
- edit rule
- duplicate rule
- deactivate rule

---

## 9.4 Templates Tab

แสดง:

- template name
- event code
- channel type
- title preview
- active flag

Actions:

- create/edit template
- preview render with sample payload
- validate variables

---

## 9.5 Logs Tab

Filters:

- event_code
- channel_type
- status
- severity
- date range
- issue_id
- target

Columns:

- time
- event
- channel
- target
- status
- message preview
- retry count
- error

Actions:

- view detail
- retry failed
- open linked issue/job/batch

---

## 9.6 Test Send Tab

Fields:

- channel
- template/event
- target
- sample payload

Actions:

- render preview
- send test
- view test result log

---

# 10. Security Rules

## NOTI-SEC-001 — No Raw Token in Frontend

ห้ามส่ง raw token ไป frontend

Frontend เห็นได้แค่:

```text
token_ref
masked target
channel status
```

---

## NOTI-SEC-002 — No Secret in Logs

notification_logs และ audit_logs ห้ามเก็บ:

- raw token
- password
- secret key
- service role key

---

## NOTI-SEC-003 — Permission Required

จัดการ Notification Center ต้องมี permission:

```text
CAN_MANAGE_NOTIFICATION
CAN_TEST_NOTIFICATION
CAN_VIEW_NOTIFICATION_LOG
```

---

## NOTI-SEC-004 — External Payload Minimization

External channel ต้องส่งข้อมูลเท่าที่จำเป็น

ถ้าต้องดู detail ให้เปิดในระบบที่ permission ตรวจแล้ว

---

# 11. Validation Rules

## NOTI-VAL-001 — Channel Validation

- channel_type required
- channel_name required
- target_id required for external channel
- token_ref required for Telegram/LINE/Email if needed
- active_flag required

---

## NOTI-VAL-002 — Rule Validation

- event_code required
- channel_id required
- template_id required
- cooldown_minutes required or send_once_per_issue true for repeatable events
- target role/channel must be valid

---

## NOTI-VAL-003 — Template Validation

- title_template required
- body_template required
- variables must exist in event payload schema
- template cannot expose forbidden variables

---

## NOTI-VAL-004 — Test Send Validation

- user must have CAN_TEST_NOTIFICATION
- channel active
- sample payload valid
- token_ref available server-side

---

# 12. Retry & Failure Handling

## 12.1 Retryable Failures

Retry ได้:

- network timeout
- temporary API rate limit
- transient channel error
- email service temporary fail

---

## 12.2 Non-Retryable Failures

Retry ไม่ช่วยจนกว่า config จะถูกแก้:

- invalid token_ref
- invalid target_id
- template variable missing
- channel inactive
- permission/config missing

---

## 12.3 Retry Policy

Recommended MVP:

```text
max_retry = 3
retry intervals = 1 min, 5 min, 15 min
```

ถ้ายัง fail:

```text
status = FAILED
notify Admin via in-app if possible
```

---

# 13. Default Notification Rules for MVP

## Rule Set v0

| Rule | Event | Target | Channel | Severity | Cooldown | Send Once |
|---|---|---|---|---|---:|---:|
| New User Approval | USER_PENDING_APPROVAL | ADMIN | In-app/Telegram | MEDIUM | 30 | No |
| Blocking Alias | UNKNOWN_ALIAS_CREATED | ADMIN | In-app | HIGH/MEDIUM | 60 | No |
| Pending Inbound | PENDING_INBOUND_CREATED | SUPERVISOR | In-app/LINE | MEDIUM | 60 | No |
| System Missing | SYSTEM_MISSING | ERP_ADMIN | In-app/Telegram | HIGH | 60 | Yes |
| REJ Not Posted | REJ_NOT_POSTED | ERP_ADMIN | In-app/Telegram/LINE | HIGH/CRITICAL | 60 | Yes |
| Close Not Ready | DAILY_CLOSE_NOT_READY | ADMIN/SUPERVISOR | In-app/Telegram | HIGH | 120 | Yes per date/warehouse |
| Export Ready | EXPORT_READY | requester | In-app/Email | LOW | 0 | Yes |
| Retention Failed | RETENTION_FAILED | ADMIN | In-app/Telegram/Email | CRITICAL | 60 | No |
| DB Usage Critical | DB_USAGE_CRITICAL | ADMIN | In-app/Telegram/Email | CRITICAL | 240 | No |

---

# 14. Notification Acceptance Criteria

Notification Center ถือว่า complete ถ้า:

- [ ] มี notification_channels
- [ ] มี notification_rules
- [ ] มี notification_templates
- [ ] มี notification_logs
- [ ] รองรับ In-app
- [ ] รองรับ Telegram abstraction
- [ ] รองรับ LINE Messaging API abstraction
- [ ] รองรับ Email basic
- [ ] Admin สร้าง/แก้/ปิด channel ได้
- [ ] Admin สร้าง/แก้/ปิด rule ได้
- [ ] Admin สร้าง/แก้ template ได้
- [ ] Test send ได้
- [ ] Event trigger ผ่าน NotificationService.notify ได้
- [ ] Cooldown ทำงาน
- [ ] Send once per issue ทำงาน
- [ ] Failed send ถูก log
- [ ] Retry failed notification ได้ตาม policy
- [ ] Raw token/secret ไม่ถูก expose ไป frontend/log
- [ ] Notification Center มี loading/error/empty states
- [ ] Retention failed / DB usage critical แจ้ง Admin ได้

---

# 15. Final Notification Center Statement

```text
Notification Center ของ Warehouse Reconciliation ERP Platform ต้องเป็นระบบ rule-based ที่แยก channel, rule, template และ log ออกจาก business logic โดยรองรับ In-app, Telegram, LINE Messaging API และ Email พร้อม severity filter, cooldown, send-once-per-issue, digest, retry และ security control เพื่อให้ระบบแจ้งเตือนเฉพาะเรื่องที่ต้อง action จริง ตรวจย้อนหลังได้ และไม่ทำให้ผู้ใช้ ignore เพราะ spam
```

