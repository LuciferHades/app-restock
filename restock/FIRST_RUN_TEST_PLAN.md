# First Run Test Plan

This test plan validates the end-to-end functionality of the freshly deployed Restock Reconciliation application.

## Prerequisites
- Web app successfully deployed.
- `setupDatabase` executed (all required sheets exist).
- Test Excel file prepared (containing headers mapping to INQUIRY, ITEM_MASTER, etc.).

## Execution Steps

### 1. Upload & Ingestion
- [ ] Open the Web App.
- [ ] Navigate to **Upload** tab.
- [ ] Upload the test Excel workbook.
- **Expected:** Progress bar indicates chunked uploading. UI shows "Staging Complete".

### 2. Validation & Mapping
- [ ] Click **Validate & Import**.
- **Expected:** The validation engine processes the data. Valid rows hit `INQUIRY` and `ITEM_MASTER`. UI shows success and redirects to the Count screen. Check Google Sheets `VALIDATION_LOG` for any intentional errors included in your test file.

### 3. Mobile Count Session
- [ ] Open the web app on a mobile device (or simulate in browser DevTools).
- [ ] Navigate to **Count**.
- [ ] Verify the location list is populated.
- [ ] Search and select a location.
- [ ] Enter a quantity (Physical Qty) for one or more items. (Notice `Expected Qty` is hidden).
- [ ] Click **Save Draft** -> Verify `localStorage` persistence by refreshing the page.
- [ ] Click **Submit** -> Verify rows are added to `COUNT_RESULT` and `PHYSICAL_COUNT` in Google Sheets.

### 4. Baseline Reconciliation (Day T)
- [ ] Navigate to **Results**.
- [ ] Click **Run Baseline Reconciliation**.
- **Expected:** Summary cards update. Exceptions are shown first. Ensure exact ItemKey discrepancies generate a Transfer Plan, while cross-customer items (e.g. RE T1) trigger `BLOCKED_NO_ACTION` or `SUBSTITUTION_REVIEW`.

### 5. T+1 Recount Expansion
- [ ] Navigate to **T+1**.
- [ ] Upload a mock Day T+1 `INOUT` Excel file.
- **Expected:** Movement rules apply. (e.g., An `OUT` movement forces the `FromLocation` into the recount scope).
- [ ] Click **Start Recount Session**.
- **Expected:** The Count screen loads in `RECOUNT` mode, restricting the location list and items strictly to the impacted scope.

### 6. Final Export
- [ ] Navigate to **Export**.
- [ ] Click **Generate & Download Excel**.
- **Expected:** Loading spinner appears. Shortly after, a direct download link is provided. The exported Excel workbook must contain exactly 8 formatted sheets (`01_SUMMARY`, `02_TRANSFER`, etc.) with `05_FINAL_ACTION` omitting matched rows.
