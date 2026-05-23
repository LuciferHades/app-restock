# Setup & Installation Guide

This document outlines how to install, configure, and reset the Restock Reconciliation Google Apps Script application.

## 1. First Install (Zero to Production)

1. Create a new Google Apps Script project at [script.google.com](https://script.google.com).
2. Copy all `.gs` and `.html` files from this repository into the Apps Script editor.
3. Update `appsscript.json` (Project Settings > Show manifest file) with the provided manifest.
4. Open `SetupService.gs` in the editor.
5. Select the function **`runSetupAll`** from the top dropdown menu and click **Run**.
6. Review and accept the requested OAuth permissions (Drive and Sheets access).
7. The installer will automatically:
   - Create the `Restock_Reconciliation_DB` Spreadsheet.
   - Set up all 18 formatted schema sheets.
   - Create organized Drive folders (uploads, exports, logs).
   - Bind environment IDs to your Google account's PropertiesService.
8. Go to **Deploy** > **New Deployment** > **Web App** to generate your application URL.

## 2. Re-run Setup / Validation

If you accidentally delete a sheet, a header row, or suspect a configuration issue:
- Select **`checkInstallation`** from the GAS dropdown and run it to view any missing dependencies in the execution log.
- Select **`runSetupAll`** and execute it again.
- **Safety:** The installer is idempotent. It will strictly repair missing sheets, headers, or folders without overwriting your existing operational data.

## 3. Resetting the Database (Destructive)

If you wish to completely wipe all imported data, count logs, and reconciliations to start fresh (while preserving the structural headers):

1. Open `SetupService.gs`.
2. Locate the line or write a temporary script calling: `SetupService.resetDatabaseConfirm("RESET_RESTOCK_APP")`
3. Execute it. All user data rows will be truncated instantly. **There is no undo.**

## 4. Deploy Web App

Whenever you modify `.html` or `.gs` code:
1. Click **Deploy** > **Manage Deployments**.
2. Edit your existing deployment.
3. Select **New version**.
4. Click Deploy to push code changes to the live URL.

## 5. Mobile Access
Send the final Web App URL to the warehouse floor team. The UI is 100% responsive and requires no installation on mobile devices. Ensure users open the link using Safari (iOS) or Chrome (Android) for optimal layout.

## 6. Common Errors

- **"Cannot access Spreadsheet"**: The root spreadsheet was deleted or ownership was transferred out of domain. Re-run `runSetupAll()`.
- **"Execution Timeout (6 minutes)"**: Google limit hit. If importing >10,000 movements, split into smaller batches before uploading.
- **"Script function not found: X"**: Check if a `.gs` file failed to save or was omitted during the copy-paste phase. Run `checkInstallation` to see exactly which global function is missing.
