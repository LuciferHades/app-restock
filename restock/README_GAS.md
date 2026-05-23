# Restock Reconciliation Tool - Google Apps Script

Mobile-first warehouse stock reconciliation using Google Apps Script.

## Architecture

- **Backend**: Google Apps Script (.gs services)
- **Database**: Google Sheets
- **Storage**: Google Drive
- **Frontend**: HTML/CSS/Vanilla JS (responsive mobile UI)
- **Deployment**: GAS Web App

## Project Structure (21 files)

**Backend (.gs files - 13 total):**
- appsscript.json - GAS project manifest
- Code.gs - Main entry point & doGet()
- Config.gs - Configuration constants
- Utils.gs - Helper functions
- SheetService.gs - Google Sheets operations (batch read/write)
- UploadService.gs - Excel upload handling
- ValidationService.gs - Data validation
- MappingService.gs - Item/location mapping
- ReconciliationEngine.gs - Day T reconciliation
- TPlusOneEngine.gs - Day T+1 movement analysis
- CountSessionService.gs - Mobile count session management
- ExportService.gs - Excel export
- LogService.gs - Centralized logging

**Frontend (.html files - 7 total):**
- `index.html` - Main app shell
- `css.html` - Mobile-first responsive CSS (vanilla)
- `js-app.html` - App routing & state
- `js-upload.html` - Upload page
- `js-count-mobile.html` - Mobile count UI
- `js-results.html` - Results display

**Documentation:**
- README.md - Project overview
- README_GAS.md - Deployment guide

## One-Click Deployment

We have introduced a fully automated setup wizard to handle the complex configurations.

1. **Create Apps Script project**: Go to [script.google.com](https://script.google.com) and create a new project.
2. **Paste files**: Copy all `.gs` and `.html` files from this repo into the editor. Ensure you include `SetupService.gs` and update `appsscript.json`.
3. **Run setupAll()**: Open `SetupService.gs`, select `runSetupAll` from the execution dropdown, and click Run.
4. **Authorize permissions**: Accept the OAuth prompt to allow the script to create the Database and Drive folders.
5. **Deploy as Web App**: Click **Deploy** -> **New Deployment** -> **Web App**. Execute as "Me", Access to "Anyone".
6. **Open Web App URL**: Use the generated link on your mobile phone or desktop to start.

*For detailed instructions or troubleshooting, see [SETUP_GUIDE.md](./SETUP_GUIDE.md).*

## Mobile Usage

### Day T (Baseline)

1. **Upload**: Select ข้อมูล.xlsx file
2. **Validate**: System checks sheets and columns
3. **Count**: Mobile user counts physical stock by location
4. **Reconcile**: System runs baseline reconciliation
5. **Results**: View exceptions and transfer plan
6. **Export**: Download draft result as Excel

### Day T+1 (Recount)

1. **Upload**: Upload Day T In-Out movements
2. **Generate**: System creates targeted recount list
3. **Recount**: Mobile user counts only impacted items/locations
4. **Finalize**: System generates final action plan
5. **Export**: Download final result as Excel

## Key Features

- ✅ Mobile-first responsive design
- ✅ Large touch buttons (48px minimum)
- ✅ Location-first counting workflow
- ✅ localStorage for offline draft saving
- ✅ Real-time sync status indicator
- ✅ Exception-first result view (MATCHED hidden by default)
- ✅ Batch Sheets operations (no row-by-row loops)
- ✅ Chunk upload for large Excel files
- ✅ Multi-sheet export workbooks

## Performance Rules

- Batch read with `getValues()`
- Batch write with `setValues()`
- No row-by-row SpreadsheetApp calls in loops
- Build maps/indexes before reconciliation
- No nested loops against 50k In-Out rows
- Chunk upload large data (500 rows per chunk)

## Business Rules

- Inquiry is snapshot at Day T
- Day T result is draft (not final)
- No final loss/over before T+1
- Transfer only exact ItemKey
- Do not auto substitute across customer
- RE T1 cross-customer must be blocked
- Mapping error rows excluded from reconciliation
- Qty only, not TonQty

## Development Phases

- **Phase 1**: ✅ Skeleton + mobile UI + service stubs
- **Phase 2**: ✅ Upload + SheetJS chunk import
- **Phase 3**: ✅ Validation + Mapping
- **Phase 4**: ✅ Count session mobile UI
- **Phase 5**: ✅ Baseline reconciliation engine
- **Phase 6**: ✅ T+1 movement impact recount
- **Phase 7**: ✅ Final Action Export + Multi-sheet workbook

## Troubleshooting

### Web App not loading
- Check GAS deployment is active
- Verify appsscript.json has correct timeZone
- Check browser console for errors

### Sheets not found
- Verify SPREADSHEET_ID is set correctly
- Check sheet names match CONFIG.SHEETS
- Run setupDatabase() to create sheets

### Upload failing
- Check file is .xls or .xlsx
- Verify file size < 25MB
- Check browser console for errors

### Performance issues
- Check for row-by-row loops in services
- Use batch operations with getValues()/setValues()
- Reduce chunk size if timeout occurs

## Support

For issues or questions, check the GAS execution logs:
- Click "Executions" tab in GAS editor
- Review logs for errors and stack traces
