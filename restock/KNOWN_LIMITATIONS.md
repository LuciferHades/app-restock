# Known Limitations

The following limitations are intrinsic to the current architecture and execution environment (Google Apps Script).

## 1. Concurrency Limits
- Google Apps Script does not inherently support true database transactions or row-level locking.
- **Impact:** If multiple users attempt to upload workbooks or run the `runBaselineReconciliation` engine simultaneously for the *same job*, data duplication or race conditions may occur.
- **Mitigation:** The application is designed to be used sequentially by a site manager.

## 2. File Size and Execution Time
- GAS scripts are strictly limited to 6 minutes of execution time per invocation.
- **Impact:** Processing massive datasets (>50,000 rows) synchronously will cause timeouts.
- **Mitigation:** Excel parsing is offloaded to the client (SheetJS). Data is uploaded in chunks of 500 rows. Engines (`ValidationService`, `ReconciliationEngine`) utilize batching (`setValues`/`getValues`), however extremely large single-job processing might require breaking down files by warehouse or region.

## 3. UI Authentication & Role Management
- **Impact:** There is no distinct application-level login, role-based access control (RBAC), or view segregation.
- **Mitigation:** Access is strictly controlled via Google Workspace sharing settings on the Web App deployment. Anyone with access to the Web App has full access to all tabs.

## 4. Final Loss/Over Actions
- Automated engine processing strictly assigns `PENDING_T1_CONFIRM` for unaccounted variances.
- **Impact:** The system will not finalize absolute financial loss/over figures automatically. It relies on human review and the final T+1 verification step.

## 5. Storage Quotas
- Google Sheets enforces a hard limit of 10,000,000 cells per workbook.
- **Mitigation:** Staging and processing tables (`UPLOAD_RAW`, `INOUT_T_PLUS_1`) are cleared automatically between jobs. However, canonical logs (`COUNT_RESULT`, `VALIDATION_LOG`, `EXPORT_LOG`) append indefinitely. Manual log rotation or archiving will be required eventually.
