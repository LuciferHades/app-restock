# Deployment Checklist

Follow these steps strictly to deploy the Restock Reconciliation application to Google Apps Script.

## 1. Prepare Google Sheet Database
- [ ] Create a new Google Sheet.
- [ ] Name it "Restock_Reconciliation_DB" (or similar).
- [ ] Copy the Spreadsheet ID from the URL (`https://docs.google.com/spreadsheets/d/[SPREADSHEET_ID]/edit`).

## 2. Configure Apps Script Project
- [ ] Open the Apps Script editor from the Google Sheet (`Extensions > Apps Script`).
- [ ] Rename the project to "Restock Reconciliation App".
- [ ] Open `Project Settings` (gear icon) -> Check the box for "Show 'appsscript.json' manifest file in editor".
- [ ] Copy the contents of the local `appsscript.json` into the editor's `appsscript.json`. Ensure the `oauthScopes` list is present.

## 3. Upload Codebase
- [ ] Create corresponding `.gs` and `.html` files in the Apps Script editor for every file in the `restock_reconciliation_gas` directory.
- [ ] Copy the code exactly as it appears in your local files.
- [ ] Open `Config.gs` and update `CONFIG.SPREADSHEET_ID` with your Google Sheet ID.

## 4. Initialize Database
- [ ] Open `Code.gs` in the editor.
- [ ] Select `runSetupDatabase` (or execute `SheetService.setupDatabase()` if you mapped a wrapper) from the function dropdown.
- [ ] Click **Run**.
- [ ] Accept the OAuth authorization prompt.
- [ ] Verify that 14+ sheets (CONFIG, JOBS, UPLOAD_RAW, etc.) are automatically created in the Google Sheet with bold headers.

## 5. Deploy Web App
- [ ] Click **Deploy** -> **New Deployment**.
- [ ] Select type **Web app**.
- [ ] Set Description (e.g., "v1.0.0 Initial Release").
- [ ] Execute as: **Me**.
- [ ] Who has access: **Anyone** (or restrict to your domain).
- [ ] Click **Deploy** and copy the Web App URL.

## 6. Initial Smoke Test
- [ ] Open the Web App URL on your desktop browser.
- [ ] Open the Web App URL on your mobile phone to verify responsive layout.
- [ ] Upload a small test workbook (`.xlsx`) to ensure the SheetJS engine and staging logic function correctly.
- [ ] Complete the full workflow (Validation -> Count -> Recon -> T+1 -> Export).
