# 11_api_backend_spec.md

# API & Backend Specification

**Project:** Warehouse Reconciliation ERP Platform  
**Document Type:** API Contract / Backend Service / Function Blueprint  
**Version:** v1.0  
**Status:** Ready for AI Agent / Developer / Backend Architect Use  
**Primary Use:** ใช้กำหนดมาตรฐาน API, service layer, backend validation, permission check, job processing, error format และ contract ของแต่ละ module เพื่อให้ AI Coding Agent สร้าง backend ได้แบบ clean, maintainable และไม่ hardcode logic สำคัญ

---

## 1. Purpose of This Document

เอกสารนี้กำหนด API และ Backend Architecture ของ Warehouse Reconciliation ERP Platform

ระบบนี้ไม่ใช่ CRUD App ธรรมดา เพราะมี process หลายชั้น:

```text
Import
→ Raw Staging
→ Column Mapping
→ Alias Mapping
→ Normalize
→ Snapshot / Ledger
→ Stock Replay
→ Reconciliation
→ Issue / Task
→ Notification / Export
→ Retention / Audit
```

ดังนั้น backend ต้องถูกออกแบบให้:

1. แยก service ตาม domain
2. มี validation กลาง
3. มี permission check ทุก action สำคัญ
4. มี audit log ทุก transaction สำคัญ
5. รองรับ long-running job เช่น import, replay, reconciliation, export, retention
6. ใช้ response format มาตรฐาน
7. ไม่ hardcode column mapping, alias, notification, permission
8. รองรับการทำงานแบบ batch/chunk
9. รองรับ data quality issue และ confidence
10. ขยายต่อจาก MVP ไป production ได้

หลักสำคัญ:

```text
Frontend ทำ UX
Backend ทำ trust, validation, permission, calculation, audit และ data integrity
```

---

## 2. Backend Architecture Principles

## 2.1 Clean Service Layer

Backend ต้องแยกเป็น service layer ตาม business domain

ตัวอย่าง:

```text
AuthService
UserApprovalService
PermissionService
ImportService
ColumnMappingService
AliasService
SnapshotService
MovementLedgerService
StockReplayService
ReconciliationService
IssueService
NotificationService
ExportService
RetentionService
AuditLogService
HealthCheckService
```

ห้ามให้ API endpoint หนึ่งทำทุก logic เองทั้งหมดจนกลายเป็น giant function

---

## 2.2 Backend Validation Required

ทุก action สำคัญต้อง validate ที่ backend แม้ frontend validate แล้ว

ตัวอย่าง:

```text
Frontend ซ่อนปุ่ม Export ไม่พอ
Backend ต้องเช็ค CAN_EXPORT อีกครั้ง
```

```text
Frontend บอกว่า qty เป็นตัวเลขแล้วไม่พอ
Backend ต้อง parse/validate qty อีกครั้ง
```

---

## 2.3 Permission First

API ทุกตัวที่เปลี่ยนข้อมูล หรือดูข้อมูลสำคัญ ต้องเช็ค:

```text
1. session valid
2. user active
3. permission
4. warehouse scope
5. operational session if needed
```

---

## 2.4 Audit Every Critical Write

ทุก action ที่มีผลต่อข้อมูลสำคัญต้องเขียน audit log

ตัวอย่าง:

- approve user
- reject user request
- confirm import
- map alias
- create movement
- run replay
- run reconciliation
- close issue
- export
- lock day
- retention delete

---

## 2.5 Job-Based Heavy Processing

งานหนักต้องทำเป็น job ไม่ใช่ทำ synchronous ใน request เดียวเสมอ

งานที่ควรเป็น job:

```text
Import large file
Normalize batch
Stock replay
Reconciliation
Export large report
Retention cleanup
Health check
```

MVP อาจทำ synchronous สำหรับข้อมูลเล็กได้ แต่ API contract ต้องรองรับ job status ตั้งแต่แรก

---

## 2.6 Idempotency Where Needed

บาง API ต้องป้องกันการกดซ้ำ เช่น:

- confirm import
- approve user
- export file
- send notification
- run retention

ควรใช้:

```text
idempotency_key
batch status check
duplicate checksum
unique constraint
```

---

## 2.7 No Hardcoded Business Mapping

Backend ห้าม hardcode:

- column index
- alias เช่น MKMK = customer แบบฝัง code
- notification token
- permission เฉพาะใน UI
- export column โดยไม่มี template/spec
- status transition โดยไม่มี rule กลาง

---

# 3. API Response Standard

## 3.1 Success Response

ทุก API success ควรตอบ format เดียวกัน:

```json
{
  "success": true,
  "data": {},
  "message": "",
  "code": "OK",
  "meta": {}
}
```

### Field Meaning

| Field | Description |
|---|---|
| success | true/false |
| data | payload หลัก |
| message | ข้อความสำหรับ user/dev |
| code | machine-readable code |
| meta | pagination, counts, timing, job status |

---

## 3.2 Error Response

```json
{
  "success": false,
  "code": "VALIDATION_ERROR",
  "message": "ข้อมูลไม่ถูกต้อง",
  "details": {}
}
```

### Error Details Example

```json
{
  "success": false,
  "code": "MISSING_REQUIRED_MAPPING",
  "message": "ยังไม่ได้ map column ที่จำเป็น",
  "details": {
    "missing_fields": ["business_date", "qty", "location_raw"]
  }
}
```

---

## 3.3 Pagination Response Meta

สำหรับ list API:

```json
{
  "success": true,
  "data": [],
  "message": "",
  "code": "OK",
  "meta": {
    "page": 1,
    "page_size": 50,
    "total_rows": 245,
    "total_pages": 5
  }
}
```

---

## 3.4 Job Response Meta

สำหรับ job-based API:

```json
{
  "success": true,
  "data": {
    "job_id": "JOB-001"
  },
  "message": "เริ่มประมวลผลแล้ว",
  "code": "JOB_QUEUED",
  "meta": {
    "status": "QUEUED",
    "estimated_seconds": 30
  }
}
```

---

# 4. Standard Error Codes

## 4.1 Auth / Permission

| Code | Meaning |
|---|---|
| AUTH_REQUIRED | ยังไม่ได้ login |
| SESSION_EXPIRED | session หมดอายุ |
| USER_NOT_ACTIVE | user ไม่ active |
| PENDING_APPROVAL | user ยังรออนุมัติ |
| PERMISSION_DENIED | ไม่มี permission |
| WAREHOUSE_SCOPE_DENIED | ไม่มีสิทธิ์ warehouse นี้ |
| CHECKER_SESSION_REQUIRED | checker ยังไม่ได้ check-in |
| CHECKER_SESSION_SUSPENDED | checker session ถูกพัก |

---

## 4.2 Validation

| Code | Meaning |
|---|---|
| VALIDATION_ERROR | validation ทั่วไปไม่ผ่าน |
| REQUIRED_FIELD_MISSING | field จำเป็นหาย |
| INVALID_DATE | date format ไม่ถูกต้อง |
| INVALID_QTY | qty ไม่ถูกต้อง |
| INVALID_STATUS_TRANSITION | เปลี่ยน status ไม่ได้ |
| LOCKED_DAY | วันถูก lock แล้ว |
| DUPLICATE_RECORD | record ซ้ำ |

---

## 4.3 Import

| Code | Meaning |
|---|---|
| INVALID_FILE_TYPE | file type ไม่รองรับ |
| EMPTY_FILE | ไม่มี row |
| HEADER_NOT_FOUND | ไม่พบ header |
| MISSING_REQUIRED_MAPPING | mapping ไม่ครบ |
| DUPLICATE_FILE_CHECKSUM | file อาจซ้ำ |
| IMPORT_BATCH_NOT_FOUND | ไม่พบ batch |
| IMPORT_ALREADY_CONFIRMED | batch confirm ไปแล้ว |
| NORMALIZE_FAILED | normalize ไม่สำเร็จ |

---

## 4.4 Alias

| Code | Meaning |
|---|---|
| UNKNOWN_ALIAS | ไม่รู้จัก alias |
| ALIAS_QUEUE_NOT_FOUND | ไม่พบรายการ review |
| ALIAS_TYPE_INVALID | alias type ไม่ถูกต้อง |
| ALIAS_DUPLICATE | alias ซ้ำ |
| LOW_CONFIDENCE_REQUIRES_REVIEW | confidence ต่ำ ต้อง review |

---

## 4.5 Reconciliation / Issue

| Code | Meaning |
|---|---|
| MISSING_ANCHOR | ไม่มี anchor snapshot |
| MISSING_SYSTEM_INOUT | In-Out ขาด |
| COVERAGE_NOT_READY | coverage ยังไม่พร้อม |
| RECON_JOB_NOT_FOUND | ไม่พบ recon job |
| ISSUE_NOT_FOUND | ไม่พบ issue |
| ISSUE_ALREADY_CLOSED | issue ปิดแล้ว |
| OPEN_CRITICAL_ISSUE_EXISTS | ยังมี critical issue เปิดอยู่ |

---

## 4.6 Export / Notification / Retention

| Code | Meaning |
|---|---|
| EXPORT_FAILED | export ไม่สำเร็จ |
| EXPORT_NOT_READY | file ยังไม่พร้อม |
| NOTIFICATION_FAILED | ส่งแจ้งเตือนไม่สำเร็จ |
| NOTIFICATION_COOLDOWN_SKIPPED | skip เพราะ cooldown |
| RETENTION_FAILED | retention ล้มเหลว |
| RETENTION_BLOCKED_OPEN_ISSUE | ลบไม่ได้เพราะมี open issue |

---

# 5. Common Request Context

ทุก backend request ควรมี context ภายในหลัง verify session:

```json
{
  "request_id": "REQ-001",
  "actor_id": "USR-001",
  "actor_roles": ["ADMIN"],
  "permissions": ["CAN_IMPORT_INOUT"],
  "warehouse_scope": ["WH_W8"],
  "business_date": "2026-05-17",
  "ip_address": "",
  "user_agent": ""
}
```

Context นี้ใช้สำหรับ:

- permission check
- warehouse scope check
- audit log
- notification owner resolution
- filter data visibility

---

# 6. Auth APIs

## 6.1 registerUser

### Purpose

สมัคร user ใหม่และสร้าง request รออนุมัติ

### Permission

Public / unauthenticated allowed

### Request

```json
{
  "username": "checker01",
  "password": "secret",
  "full_name": "ชื่อผู้ใช้",
  "phone": "0800000000",
  "email": "user@example.com",
  "requested_role": "CHECKER",
  "requested_warehouse": "WH_W8"
}
```

### Backend Flow

```text
Validate input
→ Check username/email duplicate
→ Hash password + salt
→ Create user_access_requests status=PENDING_APPROVAL
→ Write audit USER_REGISTER
→ Notify Admin USER_PENDING_APPROVAL
```

### Response

```json
{
  "success": true,
  "data": {
    "request_id": "REQ-USER-001",
    "status": "PENDING_APPROVAL"
  },
  "message": "ส่งคำขอสมัครเรียบร้อยแล้ว รอ Admin อนุมัติ",
  "code": "REGISTERED_PENDING_APPROVAL",
  "meta": {}
}
```

---

## 6.2 login

### Purpose

Login user ที่ active แล้ว

### Request

```json
{
  "username": "admin01",
  "password": "secret",
  "device_id": "optional-device-id"
}
```

### Backend Flow

```text
Find user
→ Check status ACTIVE
→ Verify password hash
→ Create login session
→ Load roles/permissions/scope
→ Write audit login success/fail
```

### Response

```json
{
  "success": true,
  "data": {
    "session_token": "token",
    "user": {
      "user_id": "USR-001",
      "username": "admin01",
      "full_name": "Admin"
    },
    "roles": ["ADMIN"],
    "permissions": ["CAN_IMPORT_INQUIRY", "CAN_MANAGE_ALIAS"],
    "warehouse_scope": ["ALL_WAREHOUSES"]
  },
  "message": "เข้าสู่ระบบสำเร็จ",
  "code": "LOGIN_SUCCESS",
  "meta": {}
}
```

---

## 6.3 logout

### Purpose

Logout และ revoke session

### Permission

Authenticated

### Request

```json
{
  "session_token": "token"
}
```

### Response

```json
{
  "success": true,
  "data": {},
  "message": "ออกจากระบบแล้ว",
  "code": "LOGOUT_SUCCESS",
  "meta": {}
}
```

---

## 6.4 verifySession

### Purpose

ตรวจว่า session ยังใช้ได้ และคืน user context

### Response Data

- user
- roles
- permissions
- warehouse_scope
- active_checker_session optional

---

# 7. User Approval APIs

## 7.1 listPendingUserRequests

### Permission

CAN_MANAGE_USER

### Request

```json
{
  "page": 1,
  "page_size": 50,
  "filters": {
    "requested_role": "CHECKER",
    "requested_warehouse": "WH_W8"
  }
}
```

### Response

Returns pending user access requests

---

## 7.2 approveUserRequest

### Permission

CAN_MANAGE_USER

### Request

```json
{
  "request_id": "REQ-USER-001",
  "role_ids": ["ROLE-CHECKER"],
  "permission_overrides": [],
  "warehouse_scopes": [
    {
      "scope_type": "WAREHOUSE",
      "warehouse_id": "WH_W8"
    }
  ]
}
```

### Backend Flow

```text
Assert CAN_MANAGE_USER
→ Load request
→ Validate status=PENDING_APPROVAL
→ Validate roles/permissions/scopes
→ Create user
→ Assign roles
→ Assign scopes
→ Mark request APPROVED or remove from pending according policy
→ Write audit USER_APPROVED
→ Notify user/admin optional
```

### Response

```json
{
  "success": true,
  "data": {
    "user_id": "USR-NEW-001",
    "status": "ACTIVE"
  },
  "message": "อนุมัติผู้ใช้เรียบร้อยแล้ว",
  "code": "USER_APPROVED",
  "meta": {}
}
```

---

## 7.3 rejectUserRequest

### Permission

CAN_MANAGE_USER

### Request

```json
{
  "request_id": "REQ-USER-001",
  "reason": "ข้อมูลไม่ครบ"
}
```

### Backend Flow

```text
Assert CAN_MANAGE_USER
→ Load request
→ Validate status=PENDING_APPROVAL
→ Write audit USER_REJECT_DELETE_REQUEST with reason
→ Delete/remove request from pending queue
→ Return success
```

### Important Rule

```text
Reject แล้วลบ request ได้ แต่ต้องเขียน audit ก่อนหรือใน transaction เดียวกัน
```

---

# 8. Permission APIs

## 8.1 getUserPermissions

### Permission

CAN_MANAGE_USER or self with authenticated session

### Response Data

- user_id
- roles
- permissions effective
- permission overrides
- warehouse scopes

---

## 8.2 updateUserRoles

### Permission

CAN_MANAGE_PERMISSION

### Request

```json
{
  "user_id": "USR-001",
  "role_ids": ["ROLE-SUPERVISOR"],
  "reason": "เปลี่ยนหน้าที่"
}
```

### Audit

ROLE_ASSIGNED / ROLE_REMOVED

---

## 8.3 updateUserPermissionOverrides

### Permission

CAN_MANAGE_PERMISSION

### Request

```json
{
  "user_id": "USR-001",
  "overrides": [
    {
      "permission_code": "CAN_EXPORT",
      "override_type": "ALLOW",
      "reason": "ให้ export ชั่วคราว"
    }
  ]
}
```

---

## 8.4 updateWarehouseScope

### Permission

CAN_MANAGE_PERMISSION

### Request

```json
{
  "user_id": "USR-001",
  "scopes": [
    {
      "scope_type": "WAREHOUSE",
      "warehouse_id": "WH_W8"
    }
  ],
  "reason": "ย้ายพื้นที่ทำงาน"
}
```

---

# 9. Checker Session APIs

## 9.1 checkInWarehouse

### Permission

CAN_CHECKIN_WAREHOUSE

### Request

```json
{
  "warehouse_id": "WH_W8",
  "business_date": "2026-05-17",
  "gps": {
    "lat": 13.7563,
    "lng": 100.5018,
    "accuracy": 25
  },
  "device_id": "device-001"
}
```

### Backend Flow

```text
Assert CAN_CHECKIN_WAREHOUSE
→ Assert warehouse scope
→ Validate GPS if required
→ Find existing active checker session same day
→ If another WH active, suspend old session
→ Create new ACTIVE session
→ Audit CHECKER_CHECKIN and CHECKER_SESSION_SUSPENDED if needed
```

### Response

```json
{
  "success": true,
  "data": {
    "checker_session_id": "CHK-SESSION-001",
    "warehouse_id": "WH_W8",
    "status": "ACTIVE"
  },
  "message": "Check-in สำเร็จ",
  "code": "CHECKIN_SUCCESS",
  "meta": {}
}
```

---

## 9.2 getActiveCheckerSession

### Permission

Authenticated

### Response Data

- active session if exists
- suspended reason if suspended
- warehouse
- business_date

---

## 9.3 checkoutWarehouse

### Permission

CAN_CHECKIN_WAREHOUSE

### Request

```json
{
  "checker_session_id": "CHK-SESSION-001"
}
```

---

# 10. Import APIs

## 10.1 createImportBatch

### Permission

Depends on file_type:

| file_type | Permission |
|---|---|
| INQUIRY | CAN_IMPORT_INQUIRY |
| SYSTEM_INOUT | CAN_IMPORT_INOUT |
| MASTER_DATA | CAN_IMPORT_MASTER |
| FIELD_COUNT | CAN_IMPORT_COUNT optional |

### Request

```json
{
  "file_type": "SYSTEM_INOUT",
  "file_name": "inout_2026_05_17.xlsx",
  "file_size": 123456,
  "file_checksum": "checksum-value",
  "business_date_from": "2026-05-12",
  "business_date_to": "2026-05-17",
  "warehouse_id": "WH_W8"
}
```

### Backend Flow

```text
Assert import permission
→ Assert warehouse scope
→ Check duplicate checksum
→ Create import_batches status=UPLOADED
→ Audit IMPORT_UPLOAD
```

### Response

```json
{
  "success": true,
  "data": {
    "batch_id": "IMP-001",
    "status": "UPLOADED",
    "duplicate_warning": false
  },
  "message": "สร้าง import batch แล้ว",
  "code": "IMPORT_BATCH_CREATED",
  "meta": {}
}
```

---

## 10.2 uploadRawChunk

### Purpose

Upload raw rows แบบ chunk เพื่อกัน timeout / payload ใหญ่

### Permission

Same as batch file_type

### Request

```json
{
  "batch_id": "IMP-001",
  "chunk_index": 0,
  "total_chunks": 10,
  "rows": [
    {
      "row_no": 1,
      "raw_json": {
        "Posting Date": "2026-05-17",
        "DC": "DC08",
        "Material Desc": "MKMK W150 69",
        "Qty": "1000"
      }
    }
  ]
}
```

### Response Meta

- rows_received
- total_received
- status

---

## 10.3 previewImportBatch

### Permission

Import permission or CAN_VIEW_IMPORT_LOG

### Request

```json
{
  "batch_id": "IMP-001",
  "sample_size": 20
}
```

### Response Data

- headers
- sample rows
- row count
- duplicate warning
- detected date range
- suggested column mapping

---

## 10.4 saveColumnMapping

### Permission

CAN_MANAGE_COLUMN_MAPPING or import permission

### Request

```json
{
  "batch_id": "IMP-001",
  "file_type": "SYSTEM_INOUT",
  "mapping_name": "ERP InOut v1",
  "mappings": [
    {
      "source_column_name": "Posting Date",
      "canonical_field": "posting_date"
    },
    {
      "source_column_name": "Qty",
      "canonical_field": "qty"
    }
  ]
}
```

### Backend Flow

```text
Validate batch
→ Validate required canonical fields
→ Save mapping template
→ Mark batch MAPPED
→ Audit COLUMN_MAPPING_UPDATED
```

---

## 10.5 normalizeBatch

### Purpose

Normalize raw rows เป็น canonical records หรือ queue unknown alias

### Permission

Import permission

### Request

```json
{
  "batch_id": "IMP-001",
  "run_async": true
}
```

### Backend Flow

```text
Load batch
→ Check status UPLOADED/MAPPED
→ Load column mapping
→ Parse rows
→ Validate date/qty/movement type
→ Match alias
→ Create normalized records or alias queue
→ Write import_error_logs
→ Update batch status NORMALIZED/WARNING/FAILED
```

### Response

Returns job_id if async

---

## 10.6 confirmImportBatch

### Purpose

ยืนยันว่า normalized data จะถูกนำไปใช้สร้าง snapshot/ledger/replay/reconciliation

### Permission

CAN_CONFIRM_IMPORT

### Request

```json
{
  "batch_id": "IMP-001",
  "confirm_policy": {
    "snapshot_conflict_action": "ARCHIVE_OLD_USE_NEW",
    "run_stock_replay": true,
    "run_reconciliation": true
  }
}
```

### Backend Flow by File Type

#### INQUIRY

```text
Create inquiry_snapshot
→ Create snapshot lines
→ Handle active snapshot conflict
→ Update coverage
```

#### SYSTEM_INOUT

```text
Create system_inout_records
→ Create movement_ledger entries
→ Update coverage
→ Queue stock replay/reconciliation if requested
```

### Audit

IMPORT_CONFIRMED

---

## 10.7 getImportBatchDetail

### Permission

CAN_VIEW_IMPORT_LOG or import permission

### Response Data

- batch metadata
- status
- row counts
- error counts
- warning counts
- unknown alias counts
- processing log

---

# 11. Alias APIs

## 11.1 listAliasQueue

### Permission

CAN_VIEW_ALIAS_QUEUE or CAN_MANAGE_ALIAS

### Request

```json
{
  "filters": {
    "alias_type": "ITEM",
    "status": "PENDING",
    "min_confidence": 0,
    "max_confidence": 80,
    "source_batch_id": "IMP-001"
  },
  "page": 1,
  "page_size": 50
}
```

---

## 11.2 mapAlias

### Permission

CAN_MANAGE_ALIAS

### Request

```json
{
  "queue_id": "ALIAS-Q-001",
  "alias_type": "ITEM",
  "canonical_id": "ITEM-W150",
  "note": "Map W150 to item master"
}
```

### Backend Flow

```text
Assert CAN_MANAGE_ALIAS
→ Load queue item
→ Validate canonical exists
→ Create alias record in correct alias table
→ Mark queue MAPPED
→ Reprocess affected rows optional
→ Audit ALIAS_MAPPED
```

---

## 11.3 createMasterFromAlias

### Permission

CAN_MANAGE_ALIAS and CAN_MANAGE_MASTER or CAN_CREATE_TEMP_MASTER

### Request

```json
{
  "queue_id": "ALIAS-Q-001",
  "alias_type": "CUSTOMER",
  "master_data": {
    "customer_code": "CUST-MKMK",
    "customer_name": "MKMK",
    "customer_type": "EXTERNAL"
  }
}
```

---

## 11.4 rejectAlias

### Permission

CAN_MANAGE_ALIAS

### Request

```json
{
  "queue_id": "ALIAS-Q-001",
  "reason": "ไม่ใช่ alias ใช้งานจริง"
}
```

---

## 11.5 searchMasterForAlias

### Purpose

Search master candidate ตอน review alias

### Request

```json
{
  "alias_type": "ITEM",
  "query": "W150",
  "product_group_id": "SUGAR"
}
```

---

# 12. Master Data APIs

## 12.1 listMasters

### Permission

Depends on master type; usually CAN_MANAGE_MASTER or view permission

### Request

```json
{
  "master_type": "ITEM",
  "filters": {
    "product_group_id": "SUGAR",
    "active_flag": true
  },
  "page": 1,
  "page_size": 50
}
```

---

## 12.2 createMaster

### Permission

CAN_MANAGE_MASTER

### Request Example Item

```json
{
  "master_type": "ITEM",
  "data": {
    "item_code": "ITEM-W150",
    "item_name": "น้ำตาลทรายมิตรผล W150",
    "product_group_id": "SUGAR",
    "grade": "W150",
    "pack_size": "50KG",
    "default_uom": "BAG"
  }
}
```

---

## 12.3 updateMaster

### Permission

CAN_MANAGE_MASTER

### Rule

ถ้า master ถูกใช้แล้ว ห้ามเปลี่ยน key identity โดยไม่มี audit/impact warning

---

## 12.4 deactivateMaster

### Permission

CAN_MANAGE_MASTER

### Rule

ใช้ inactive แทน delete ถ้า master ถูกใช้งานแล้ว

---

# 13. Field Operation APIs

## 13.1 createFieldMovement

### Permission

CAN_CREATE_FIELD_MOVEMENT

### Request

```json
{
  "business_date": "2026-05-17",
  "movement_type": "OUT",
  "warehouse_id": "WH_W8",
  "location_id": "LOC-W8-03",
  "customer_id": "CUST-MKMK",
  "item_id": "ITEM-W150",
  "year": "69",
  "lot": "",
  "qty": 1000,
  "uom": "BAG",
  "source_text": "MKMK W150 69",
  "photo_url": ""
}
```

### Backend Flow

```text
Assert CAN_CREATE_FIELD_MOVEMENT
→ Assert warehouse scope
→ If checker, assert active checker session
→ Validate locked day
→ Validate qty/location/movement_type
→ Create field_movements
→ Create movement_ledger source_type=FIELD_NOTICE
→ Audit FIELD_MOVEMENT_CREATED
```

---

## 13.2 createPendingInbound

### Permission

CAN_CREATE_PENDING_INBOUND or CAN_CREATE_FIELD_MOVEMENT

### Request

```json
{
  "business_date": "2026-05-17",
  "warehouse_id": "WH_W8",
  "location_id": "LOC-W8-03",
  "qty": 1000,
  "uom": "BAG",
  "text_on_label": "MKMK W150 69",
  "photo_url": ""
}
```

### Backend Flow

```text
Assert permission/session
→ Parse smart input
→ If high confidence and complete, allow create inbound movement
→ Else create pending_inbounds
→ Create alias queue for unknown tokens
→ Audit PENDING_INBOUND_CREATED
→ Notify if rule active
```

---

## 13.3 confirmPendingInbound

### Permission

CAN_REVIEW_ISSUE or CAN_MANAGE_ALIAS depending case

### Request

```json
{
  "pending_inbound_id": "PIN-001",
  "candidate_id": "CAND-001",
  "confirmed_customer_id": "CUST-MKMK",
  "confirmed_item_id": "ITEM-W150",
  "year": "69",
  "lot": "",
  "note": "Matched with ERP In-Out"
}
```

### Backend Flow

```text
Load pending inbound
→ Validate status
→ Validate candidate or manual master
→ Create canonical field movement/ledger
→ Update pending status CONFIRMED
→ Update related issue
→ Audit PENDING_INBOUND_CONFIRMED
```

---

## 13.4 createPhysicalCount

### Permission

CAN_COUNT

### Request

```json
{
  "business_date": "2026-05-17",
  "checker_session_id": "CHK-SESSION-001",
  "task_id": "TASK-001",
  "warehouse_id": "WH_W8",
  "location_id": "LOC-W8-03",
  "customer_id": "CUST-MKMK",
  "item_id": "ITEM-W150",
  "year": "69",
  "lot": "",
  "qty_counted": 2000,
  "uom": "BAG",
  "count_type": "RECOUNT",
  "photo_url": "",
  "remark": ""
}
```

### Backend Flow

```text
Assert CAN_COUNT
→ Assert active checker session for warehouse
→ Validate task if provided
→ Validate locked day
→ Save physical_counts
→ Update task status if linked
→ Trigger reconciliation update optional
→ Audit PHYSICAL_COUNT_SUBMITTED
```

---

## 13.5 createRejNotice

### Permission

CAN_CREATE_REJ

### Request

```json
{
  "business_date": "2026-05-17",
  "warehouse_id": "WH_W8",
  "from_location_id": "LOC-W8-03",
  "rej_location_id": "LOC-W8-REJ",
  "customer_id": "CUST-MKMK",
  "item_id": "ITEM-W150",
  "year": "69",
  "lot": "",
  "qty": 10,
  "uom": "BAG",
  "reason_id": "REJ-DAMAGE",
  "photo_url": ""
}
```

### Backend Flow

```text
Assert CAN_CREATE_REJ
→ Assert checker session if checker
→ Validate REJ location/reason/qty
→ Create rej_notice
→ Create movement_ledger source_type=REJ_NOTICE
→ Status WAIT_SYSTEM_POST
→ Audit REJ_NOTICE_CREATED
→ Notify if configured
```

---

# 14. Movement Ledger APIs

## 14.1 getMovementLedger

### Permission

CAN_VIEW_RECON_RESULT or appropriate view permission

### Request

```json
{
  "filters": {
    "business_date_from": "2026-05-17",
    "business_date_to": "2026-05-17",
    "warehouse_id": "WH_W8",
    "location_id": "LOC-W8-03",
    "item_id": "ITEM-W150"
  },
  "page": 1,
  "page_size": 100
}
```

---

## 14.2 rebuildLedgerForBatch

### Permission

Admin/System only

### Purpose

ใช้ตอน normalize/reprocess batch หรือ correction

---

# 15. Stock Replay APIs

## 15.1 runStockReplay

### Permission

CAN_RUN_STOCK_REPLAY

### Request

```json
{
  "date_from": "2026-05-12",
  "date_to": "2026-05-17",
  "warehouse_id": "WH_W8",
  "run_async": true,
  "reason": "Backfill In-Out"
}
```

### Backend Flow

```text
Assert CAN_RUN_STOCK_REPLAY
→ Assert warehouse scope
→ Run data coverage check
→ Find anchor snapshot
→ Load movement ledger
→ Calculate daily stock position
→ Save daily_stock_positions
→ Write replay job log
→ Audit RUN_STOCK_REPLAY
```

### Response

Returns replay_job_id

---

## 15.2 getStockReplayJob

### Permission

CAN_RUN_STOCK_REPLAY or CAN_VIEW_RECON_RESULT

### Response Data

- job status
- rows processed
- warning/error count
- logs

---

## 15.3 getDailyStockPosition

### Permission

CAN_VIEW_RECON_RESULT / role-based view

### Request

```json
{
  "business_date": "2026-05-17",
  "warehouse_id": "WH_W8",
  "filters": {
    "location_id": "LOC-W8-03",
    "customer_id": "CUST-MKMK",
    "item_id": "ITEM-W150"
  }
}
```

---

## 15.4 getCoverageCheck

### Permission

CAN_VIEW_CONTROL_CENTER or CAN_RUN_STOCK_REPLAY

### Request

```json
{
  "date_from": "2026-05-12",
  "date_to": "2026-05-17",
  "warehouse_id": "WH_W8"
}
```

### Response Data

- anchor_status
- inout_status
- alias_status
- pending_inbound_status
- rej_status
- count_status
- overall_status
- detail list

---

# 16. Reconciliation APIs

## 16.1 runReconciliation

### Permission

CAN_RUN_RECONCILIATION

### Request

```json
{
  "date_from": "2026-05-17",
  "date_to": "2026-05-17",
  "warehouse_id": "WH_W8",
  "run_async": true,
  "reason": "After In-Out import"
}
```

### Backend Flow

```text
Assert permission/scope
→ Load coverage check
→ Load field movement/system inout/count/daily stock position/REJ
→ Build matching candidates
→ Compare records
→ Generate result_code/severity/confidence
→ Generate explanation/suggested_action
→ Create/update issues
→ Update summary tables
→ Audit RUN_RECONCILIATION
→ Notify based on rules
```

### Response

Returns recon_job_id

---

## 16.2 getReconciliationJob

### Permission

CAN_VIEW_RECON_RESULT

### Response Data

- status
- matched count
- issue count
- warning/error count
- time

---

## 16.3 listReconResults

### Permission

CAN_VIEW_RECON_RESULT

### Request

```json
{
  "filters": {
    "date_from": "2026-05-17",
    "date_to": "2026-05-17",
    "warehouse_id": "WH_W8",
    "result_code": "QTY_DIFF",
    "severity": "HIGH",
    "confidence": "HIGH"
  },
  "page": 1,
  "page_size": 50
}
```

---

## 16.4 getReconResultDetail

### Permission

CAN_VIEW_RECON_DETAIL

### Response Data

- result header
- field movement data
- system in-out data
- physical count data
- daily stock position data
- explanation
- likely cause
- suggested action
- linked issue
- audit/source links

---

# 17. Issue APIs

## 17.1 listIssues

### Permission

CAN_REVIEW_ISSUE or CAN_VIEW_RECON_RESULT

### Request

```json
{
  "filters": {
    "date_from": "2026-05-17",
    "date_to": "2026-05-17",
    "warehouse_id": "WH_W8",
    "issue_type": "REJ_NOT_POSTED",
    "severity": "CRITICAL",
    "status": "OPEN",
    "owner_role": "ERP_ADMIN"
  },
  "page": 1,
  "page_size": 50
}
```

---

## 17.2 getIssueDetail

### Permission

CAN_REVIEW_ISSUE or role owner view

### Response Data

- issue header
- result detail
- comparison panel
- explanation
- suggested action
- related source records
- tasks
- comments
- audit timeline
- notification logs

---

## 17.3 assignIssue

### Permission

CAN_ASSIGN_ISSUE or CAN_REVIEW_ISSUE

### Request

```json
{
  "issue_id": "ISS-001",
  "assigned_to": "USR-ERP-001",
  "owner_role": "ERP_ADMIN",
  "due_at": "2026-05-18T12:00:00",
  "note": "ตรวจ ERP post"
}
```

### Audit

ISSUE_ASSIGN

---

## 17.4 createCheckerTaskFromIssue

### Permission

CAN_CREATE_RECOUNT_TASK or CAN_ASSIGN_TASK

### Request

```json
{
  "issue_id": "ISS-001",
  "task_type": "RECOUNT",
  "assigned_to": "USR-CHECKER-001",
  "priority": "HIGH",
  "due_at": "2026-05-17T17:00:00"
}
```

---

## 17.5 closeIssue

### Permission

CAN_CLOSE_ISSUE

### Request

```json
{
  "issue_id": "ISS-001",
  "close_reason": "ERP post แก้ไขแล้วและ reconcile ตรง",
  "resolution_code": "ERP_POSTED_CORRECTED"
}
```

### Backend Flow

```text
Assert permission
→ Load issue
→ Validate not already closed
→ Validate required resolution
→ Close issue
→ Update recon result status
→ Audit ISSUE_CLOSE
→ Notify if configured
```

---

## 17.6 acceptDifferenceWithReason

### Permission

CAN_ACCEPT_DIFF

### Request

```json
{
  "issue_id": "ISS-001",
  "reason": "ยอดต่างเล็กน้อยและได้รับ approval แล้ว",
  "approved_by": "USR-SUP-001"
}
```

### Important Rule

ต้องมี reason และ audit เสมอ

---

# 18. Control Center APIs

## 18.1 getControlCenterSummary

### Permission

CAN_VIEW_CONTROL_CENTER

### Request

```json
{
  "business_date": "2026-05-17",
  "warehouse_id": "WH_W8"
}
```

### Response Data

```json
{
  "close_status": "OPEN",
  "matched_count": 120,
  "issue_count": 14,
  "critical_issue_count": 3,
  "unknown_alias_count": 5,
  "pending_inbound_count": 2,
  "rej_not_posted_count": 1,
  "export_pending_count": 4,
  "last_import_at": "2026-05-17T10:00:00",
  "last_recon_at": "2026-05-17T10:20:00",
  "retention_status": "SUCCESS",
  "health_status": "OK"
}
```

### Rule

อ่านจาก summary/result tables ไม่ใช่ raw full history

---

## 18.2 getDailyCloseReadiness

### Permission

CAN_VIEW_CLOSE_READINESS

### Request

```json
{
  "business_date": "2026-05-17",
  "warehouse_id": "WH_W8"
}
```

### Response Data

- readiness_status: READY / READY_WITH_WARNING / NOT_READY
- blockers
- warnings
- suggested actions

---

## 18.3 closeDay

### Permission

CAN_CLOSE_DAY

### Request

```json
{
  "business_date": "2026-05-17",
  "warehouse_id": "WH_W8",
  "close_type": "PRELIM_CLOSED",
  "reason": "ตรวจแล้วไม่มี critical open"
}
```

### Backend Flow

```text
Assert CAN_CLOSE_DAY
→ Run readiness gate
→ If NOT_READY block unless override policy
→ Update daily_close_status
→ Audit DAILY_CLOSE
```

---

## 18.4 lockDay

### Permission

CAN_LOCK_DAY

### Request

```json
{
  "business_date": "2026-05-17",
  "warehouse_id": "WH_W8",
  "reason": "ปิดรอบเรียบร้อย"
}
```

### Rule

LOCKED day ห้ามแก้ตรง ต้องผ่าน correction/investigation

---

# 19. Export APIs

## 19.1 createExportJob

### Permission

CAN_EXPORT

### Request

```json
{
  "export_type": "ISSUE_LIST",
  "file_format": "XLSX",
  "filters": {
    "date_from": "2026-05-17",
    "date_to": "2026-05-17",
    "warehouse_id": "WH_W8",
    "status": "OPEN"
  },
  "selected_ids": []
}
```

### Backend Flow

```text
Assert CAN_EXPORT
→ Assert data scope
→ Create export job
→ Generate file from summary/result/issue tables
→ Save file_url
→ Write export log
→ Audit EXPORT_CREATE
→ Notify if configured
```

---

## 19.2 exportSelectedRows

### Permission

CAN_EXPORT

### Request

```json
{
  "export_type": "ERP_TEMPLATE",
  "file_format": "CSV",
  "selected_ids": ["ISS-001", "ISS-002"]
}
```

---

## 19.3 getExportJob

### Permission

CAN_EXPORT or CAN_VIEW_EXPORT_LOG

---

## 19.4 listExportLogs

### Permission

CAN_VIEW_EXPORT_LOG

---

# 20. Notification APIs

## 20.1 saveNotificationChannel

### Permission

CAN_MANAGE_NOTIFICATION

### Request

```json
{
  "channel_type": "TELEGRAM",
  "channel_name": "ERP Admin Group",
  "token_ref": "TELEGRAM_BOT_MAIN",
  "target_id": "chat-id",
  "active_flag": true
}
```

### Important Rule

ห้ามส่ง raw token ไป frontend ถ้าไม่จำเป็น ให้ใช้ token_ref

---

## 20.2 saveNotificationRule

### Permission

CAN_MANAGE_NOTIFICATION

### Request

```json
{
  "event_code": "REJ_NOT_POSTED",
  "rule_name": "แจ้ง ERP Admin เมื่อ REJ ยังไม่ post",
  "severity_filter": ["HIGH", "CRITICAL"],
  "target_role": "ERP_ADMIN",
  "channel_id": "CH-TELEGRAM-ERP",
  "template_id": "TPL-REJ-NOT-POSTED",
  "cooldown_minutes": 60,
  "send_once_per_issue": true,
  "active_flag": true
}
```

---

## 20.3 saveNotificationTemplate

### Permission

CAN_MANAGE_NOTIFICATION

### Request

```json
{
  "template_name": "REJ Not Posted Template",
  "event_code": "REJ_NOT_POSTED",
  "title_template": "REJ ยังไม่ถูก post",
  "body_template": "วันที่ {{business_date}} Location {{location}} Qty {{qty}} Issue {{issue_id}}"
}
```

---

## 20.4 testNotification

### Permission

CAN_TEST_NOTIFICATION

---

## 20.5 listNotificationLogs

### Permission

CAN_VIEW_NOTIFICATION_LOG

---

## 20.6 Internal notify(event_code, payload)

### Purpose

service ภายในสำหรับส่งแจ้งเตือนจากระบบ

### Internal Flow

```text
Load active rules by event_code
→ Filter severity/role/channel
→ Check cooldown/send once
→ Render template
→ Send channel
→ Write notification log
```

---

# 21. Retention & Maintenance APIs

## 21.1 runRetentionJob

### Permission

CAN_RUN_RETENTION or System scheduled job

### Request

```json
{
  "cutoff_date": "2026-04-02",
  "run_mode": "DRY_RUN",
  "reason": "manual check"
}
```

### run_mode

```text
DRY_RUN = preview delete/skip only
EXECUTE = actually delete/archive
```

### Backend Flow

```text
Assert permission/system
→ Health check
→ Create/archive summary
→ Find records older than cutoff
→ Skip open issues/pending export/pending approval
→ If DRY_RUN return preview
→ If EXECUTE delete/archive safe records
→ Write retention_logs
→ Audit RETENTION_DELETE
→ Notify Admin summary
```

---

## 21.2 getRetentionLogs

### Permission

CAN_RUN_RETENTION or CAN_VIEW_HEALTH_CHECK

---

## 21.3 getHealthStatus

### Permission

CAN_VIEW_HEALTH_CHECK

### Response Data

- db usage
- row counts
- last import
- last replay
- last reconciliation
- last retention
- open critical issues
- pending alias
- pending inbound
- failed jobs

---

## 21.4 runHealthCheck

### Permission

System scheduled job or Admin

---

# 22. Backfill APIs

## 22.1 createBackfillJob

### Permission

CAN_IMPORT_INOUT and CAN_RUN_STOCK_REPLAY

### Request

```json
{
  "date_from": "2026-05-12",
  "date_to": "2026-05-17",
  "warehouse_id": "WH_W8",
  "inout_batch_ids": ["IMP-001"],
  "run_replay": true,
  "run_reconciliation": true,
  "create_count_tasks": true
}
```

### Backend Flow

```text
Assert permissions
→ Validate date range
→ Run coverage check
→ Ensure In-Out batches normalized/confirmed
→ Run stock replay
→ Run reconciliation
→ Create count required tasks
→ Save backfill summary
→ Notify BACKFILL_COMPLETED
```

---

## 22.2 getBackfillJobStatus

### Permission

CAN_IMPORT_INOUT / CAN_VIEW_CONTROL_CENTER

---

# 23. Audit APIs

## 23.1 listAuditLogs

### Permission

CAN_VIEW_AUDIT_LOG

### Request

```json
{
  "filters": {
    "actor_id": "USR-001",
    "action": "IMPORT_CONFIRMED",
    "entity_type": "import_batch",
    "date_from": "2026-05-01",
    "date_to": "2026-05-17"
  },
  "page": 1,
  "page_size": 50
}
```

---

## 23.2 getEntityAuditTrail

### Permission

CAN_VIEW_AUDIT_LOG or related detail permission

### Request

```json
{
  "entity_type": "issue",
  "entity_id": "ISS-001"
}
```

---

# 24. Backend Services Summary

## 24.1 AuthService

Functions:

- registerUser
- login
- logout
- verifySession
- hashPassword
- verifyPassword
- createSession
- revokeSession

---

## 24.2 PermissionService

Functions:

- getEffectivePermissions
- assertPermission
- assertWarehouseScope
- assertCheckerSession
- updateUserRoles
- updatePermissionOverrides
- updateWarehouseScope

---

## 24.3 ImportService

Functions:

- createBatch
- uploadChunk
- previewBatch
- normalizeBatch
- confirmBatch
- getBatchDetail

---

## 24.4 AliasService

Functions:

- detectAlias
- createQueueItem
- suggestMatch
- mapAlias
- createMasterFromAlias
- rejectAlias
- reprocessAffectedRows

---

## 24.5 SnapshotService

Functions:

- createInquirySnapshot
- createSnapshotLines
- handleSnapshotConflict
- getActiveSnapshot

---

## 24.6 MovementLedgerService

Functions:

- createLedgerFromSystemInOut
- createLedgerFromFieldMovement
- createLedgerFromRej
- rebuildLedgerForBatch
- cancelLedgerEntry

---

## 24.7 StockReplayService

Functions:

- runReplay
- findAnchorSnapshot
- loadLedgerRange
- calculateDailyPositions
- saveDailyPositions
- calculateConfidence

---

## 24.8 ReconciliationService

Functions:

- runReconciliation
- buildMatchingKey
- matchFieldSystem
- compareExpectedCount
- compareRejSystem
- generateResult
- generateExplanation
- createOrUpdateIssue

---

## 24.9 IssueService

Functions:

- listIssues
- getIssueDetail
- assignIssue
- createTaskFromIssue
- closeIssue
- acceptDifference
- reopenIssue

---

## 24.10 NotificationService

Functions:

- notify
- renderTemplate
- sendInApp
- sendTelegram
- sendLineMessaging
- sendEmail
- writeNotificationLog
- checkCooldown

---

## 24.11 ExportService

Functions:

- createExportJob
- buildExportDataset
- generateCsv
- generateXlsx
- generatePdfOrPrintView
- writeExportLog

---

## 24.12 RetentionService

Functions:

- runRetention
- dryRunRetention
- archiveSummary
- findDeletableRecords
- skipOpenIssueRecords
- deleteSafeRecords
- writeRetentionLog

---

## 24.13 AuditLogService

Functions:

- writeAudit
- listAuditLogs
- getEntityTrail

---

# 25. Transaction Boundaries

Critical operations should be atomic where possible:

## 25.1 Approve User Transaction

```text
Create user
→ assign roles
→ assign scope
→ update/delete request
→ audit log
```

If audit fails, transaction should fail or retry safely

---

## 25.2 Confirm Import Transaction

```text
Update batch status
→ create snapshot/records/ledger
→ write audit
→ queue jobs
```

---

## 25.3 Map Alias Transaction

```text
Create alias
→ update queue status
→ mark affected rows for reprocess
→ audit log
```

---

## 25.4 Close Issue Transaction

```text
Update issue
→ update recon result
→ close related task if needed
→ audit log
→ notify
```

---

## 25.5 Retention Execute Transaction

Retention อาจทำเป็น batch transaction แยก table เพื่อลด lock

ต้องเขียน log per table

---

# 26. Job Status Model

Use common job status:

```text
QUEUED
RUNNING
SUCCESS
WARNING
FAILED
CANCELLED
```

Job fields:

- job_id
- job_type
- triggered_by
- started_at
- finished_at
- status
- rows_processed
- warning_count
- error_count
- message

Applies to:

- import normalize
- stock replay
- reconciliation
- export
- retention
- health check

---

# 27. API Security Rules

1. Do not expose secret/token in response
2. Do not trust client role/permission claims
3. Verify session from backend/session store
4. Validate warehouse scope on every warehouse-specific request
5. Use audit log for security-sensitive changes
6. Use rate limiting or lockout for login failures if possible
7. Use server-side generated IDs
8. Validate file type and size
9. Sanitize exported values if needed
10. Do not allow direct edit on locked day

---

# 28. API Acceptance Criteria

API/Backend ถือว่า complete ถ้า:

- [ ] ทุก API ใช้ response format มาตรฐาน
- [ ] Error code เป็น machine-readable
- [ ] Auth/session ตรวจได้
- [ ] Permission ตรวจ backend ทุก action สำคัญ
- [ ] Warehouse scope ตรวจทุก request ที่เกี่ยวกับคลัง
- [ ] Checker count ต้องมี active session
- [ ] Import ใช้ batch/chunk/staging
- [ ] Column mapping ไม่ hardcode
- [ ] Alias mapping แยก service และมี review queue
- [ ] Inquiry confirm สร้าง Anchor Snapshot
- [ ] In-Out confirm สร้าง System Movement + Ledger
- [ ] Stock replay เป็น job และมี log
- [ ] Reconciliation สร้าง result + explanation
- [ ] Issue มี owner/status/action
- [ ] Export มี export job/log
- [ ] Notification มี rule/template/log/cooldown
- [ ] Retention มี dry run, execute, skip open issue, log
- [ ] ทุก critical write มี audit log

---

# 29. Final API & Backend Statement

```text
Backend ของ Warehouse Reconciliation ERP Platform ต้องเป็นระบบ service-based ที่ตรวจ session, permission, warehouse scope, validation และ audit ทุกครั้ง ก่อนประมวลผลข้อมูลผ่าน import staging, alias mapping, anchor snapshot, movement ledger, stock replay, reconciliation, issue, export, notification และ retention โดยทุก API ต้องตอบรูปแบบมาตรฐานและรองรับ job processing สำหรับงานหนัก เพื่อให้ระบบมั่นคง ขยายต่อได้ และตรวจย้อนหลังได้จริง
```

