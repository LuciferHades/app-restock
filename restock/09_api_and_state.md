# API and State

## API Endpoints

| Method | Endpoint | Purpose |
| :--- | :--- | :--- |
| **POST** | `/api/jobs/upload` | Upload Excel file and initialize a new reconciliation job. |
| **GET** | `/api/jobs/{id}/status` | Retrieve the current processing status of a job. |
| **POST** | `/api/jobs/{id}/validate` | Trigger data validation for the uploaded file. |
| **POST** | `/api/jobs/{id}/run` | Execute the Day T reconciliation logic. |
| **POST** | `/api/jobs/{id}/t1-upload` | Upload the Day T In-Out data for impact analysis. |
| **POST** | `/api/jobs/{id}/t1-recount` | Generate the T+1 targeted recount list. |
| **POST** | `/api/jobs/{id}/finalize` | Process recount results and generate the final action plan. |
| **GET** | `/api/jobs/{id}/results` | Fetch summary metrics and result tables for the UI. |
| **GET** | `/api/jobs/{id}/export` | Download the final Excel result workbook. |

## Job Lifecycle Statuses
A job progresses through the following states:
- `UPLOADED`: File received and stored.
- `VALIDATING`: Integrity checks in progress.
- `VALIDATION_FAILED`: Errors found in the input file.
- `READY_TO_RUN`: File passed validation.
- `RECONCILING`: Day T analysis in progress.
- `RECONCILED`: Day T analysis complete; waiting for T+1 data.
- `WAITING_T_PLUS_1`: Ready for Day T In-Out upload.
- `GENERATING_RECOUNT`: T+1 impact analysis in progress.
- `WAITING_RECOUNT`: Recount list generated; waiting for results.
- `FINALIZING`: Final reconciliation in progress.
- `COMPLETED`: All steps finished; export ready.
- `FAILED`: System error during processing.

## Frontend State Management
The frontend maintains the following key state variables:
- `currentJobId`: The ID of the active reconciliation task.
- `jobStatus`: The current state of the backend process.
- `validationErrors`: List of issues found during the validation phase.
- `resultsData`: The processed reconciliation data for display in tables.
- `activeTab`: The currently selected view (Summary, Transfer, Recount, etc.).
