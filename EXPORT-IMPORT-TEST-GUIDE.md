# Export/Import System Test Guide

## Overview
This guide explains how to test the complete export/import system for the Prompt Library application.

## Test Environment
- **Main App:** http://localhost:8000
- **Test Suite:** http://localhost:8000/test-export-import.html
- **Server:** Python HTTP Server (port 8000)

## Test Files Created
1. **test-import-valid.json** - Modern schema format with 2 prompts
2. **test-import-legacy-array.json** - Legacy array format (backward compatibility)
3. **test-import-legacy-object.json** - Legacy object format (backward compatibility)
4. **test-import-duplicates.json** - File with duplicate IDs to test conflict resolution
5. **test-import-invalid.json** - Invalid format to test error handling
6. **test-import-corrupt.json** - Malformed JSON to test error recovery

## Automated Test Suite

### Access
Open http://localhost:8000/test-export-import.html in your browser

### Test Categories

#### 1. Setup Test Data
- **Create Test Prompts**: Generates 3 sample prompts with ratings and notes
- **Clear All Data**: Removes all data from localStorage
- **Show Current Data**: Displays current localStorage contents

#### 2. Export Tests
- **Test Export Structure**: Validates JSON schema structure
- **Test Statistics Calculation**: Verifies average rating, total prompts, most used model
- **Test Export Validation**: Ensures all prompts have required fields
- **Perform Actual Export**: Triggers download (use main app for this)

#### 3. Import Tests
- **Test Valid Import**: Validates modern schema format parsing
- **Test Legacy Array Format**: Tests backward compatibility with array format
- **Test Legacy Object Format**: Tests backward compatibility with object format
- **Test Invalid Format**: Verifies error detection for malformed files

#### 4. Conflict Resolution Tests
- **Test Replace Strategy**: Verifies incoming versions overwrite existing
- **Test Skip Strategy**: Verifies existing versions are kept
- **Test Keep Both Strategy**: Verifies both versions preserved with new IDs

#### 5. Error Recovery Tests
- **Test Backup Creation**: Verifies automatic backup before import
- **Test Rollback on Error**: Verifies data restoration on failure
- **Test Corrupt File Import**: Tests handling of malformed JSON

### Running the Tests
1. Open the test suite page
2. Click individual test buttons or run sections sequentially
3. Watch the output panels for results
4. Check the summary section for pass/fail counts

## Manual Testing in Main App

### Test Export Functionality

1. **Navigate to main app**: http://localhost:8000
2. **Add sample data**:
   - Create 3-5 prompts with different models
   - Add ratings to some prompts
   - Add notes to some prompts
3. **Click "Export" button** in the header
4. **Verify**:
   - File downloads automatically
   - Filename format: `prompt-library-export-YYYY-MM-DDTHH-MM-SS-sssZ.json`
   - Success message appears
5. **Inspect exported file**:
   - Open in text editor
   - Verify JSON structure matches schema
   - Check statistics are calculated correctly
   - Verify all prompts and notes are included

### Test Import Functionality

#### Test 1: Import Valid File
1. Click "Import" button
2. Select `test-import-valid.json`
3. **Expected**:
   - 2 new prompts added
   - Success message: "Import successful. Added 2 prompt(s)."
   - Prompts appear in the list

#### Test 2: Import Legacy Array Format
1. Click "Import" button
2. Select `test-import-legacy-array.json`
3. **Expected**:
   - Legacy format recognized and converted
   - Metadata auto-generated for missing fields
   - Success message displayed

#### Test 3: Import Legacy Object Format
1. Click "Import" button
2. Select `test-import-legacy-object.json`
3. **Expected**:
   - Legacy object format parsed correctly
   - Notes imported along with prompts
   - Success message displayed

#### Test 4: Import with Duplicates (Conflict Resolution)
1. First, import `test-import-duplicates.json` once (clean state)
2. Import `test-import-duplicates.json` again
3. **Dialog 1**: "2 duplicate ID(s) found. Proceed to choose resolution?"
   - Click OK to continue
4. **Dialog 2**: "Click OK to REPLACE duplicates with incoming versions"
   - **Option A**: Click OK → Tests REPLACE strategy
     - Verify: Existing prompts updated with new content
     - Check titles say "UPDATED VERSION" and "ENHANCED"
   - **Option B**: Click Cancel → Go to Dialog 3
5. **Dialog 3**: "Click OK to SKIP incoming duplicates"
   - **Option A**: Click OK → Tests SKIP strategy
     - Verify: Existing prompts unchanged
     - Only new unique prompt added
   - **Option B**: Click Cancel → Tests KEEP BOTH strategy
     - Verify: Both versions exist with different IDs
     - Total prompt count increases

#### Test 5: Import Invalid File
1. Click "Import" button
2. Select `test-import-invalid.json`
3. **Expected**:
   - Error message: "Unsupported file type" or similar
   - No data changed
   - Original data intact

#### Test 6: Import Corrupt File
1. Click "Import" button
2. Select `test-import-corrupt.json`
3. **Expected**:
   - Error message: "File is not valid JSON"
   - Automatic rollback performed
   - Original data restored
   - Error displayed in red

### Test Error Recovery

#### Test Backup and Rollback
1. **Setup**: Have some prompts in the app
2. **Action**: Import a corrupt file
3. **Verify**:
   - Check browser DevTools → Application → Local Storage
   - Look for `.backup` keys:
     - `promptLibrary.items.v1.backup`
     - `promptNotes.v1.backup`
   - These should contain your data before import attempt
4. **After Error**:
   - Main data should be restored from backup
   - App should function normally
   - No data loss

## Testing Dark Mode Compatibility

1. Click "Dark" button in header to enable dark mode
2. Verify export/import buttons are visible and styled correctly
3. Perform export/import operations
4. Check message container is readable in dark mode

## Expected Results Summary

### Export
✅ File downloads with timestamp
✅ JSON contains all required fields
✅ Statistics calculated correctly
✅ Metadata included for all prompts
✅ Notes exported with prompts

### Import
✅ Modern schema format parsed
✅ Legacy formats supported (array & object)
✅ Invalid formats rejected with errors
✅ Corrupt JSON detected and handled
✅ Automatic backup created before import
✅ Automatic rollback on error

### Conflict Resolution
✅ Replace: Incoming overwrites existing
✅ Skip: Existing retained, incoming ignored
✅ Keep Both: Both versions preserved with unique IDs

### Error Handling
✅ Detailed error messages shown
✅ Data backed up automatically
✅ Rollback on failure
✅ No data loss on errors

## Browser Console Testing

For advanced testing, open Browser DevTools Console and run:

```javascript
// View current data
console.log('Prompts:', JSON.parse(localStorage.getItem('promptLibrary.items.v1')));
console.log('Notes:', JSON.parse(localStorage.getItem('promptNotes.v1')));

// View backups
console.log('Backup:', localStorage.getItem('promptLibrary.items.v1.backup'));

// Clear all data
localStorage.clear();

// Test export function (from main app page)
window.__promptMeta.estimateTokens('test content', false);
```

## Success Criteria

All tests should pass with:
- ✅ 0 data loss incidents
- ✅ All format types supported
- ✅ Error recovery working
- ✅ UI feedback clear and accurate
- ✅ File structure valid JSON
- ✅ Statistics calculated correctly

## Troubleshooting

### Export button not working
- Check browser console for errors
- Verify localStorage has data
- Try refreshing the page

### Import not accepting file
- Verify file is valid JSON
- Check file extension is .json
- Try opening file in text editor to validate structure

### Duplicate dialog not appearing
- Ensure you're importing a file with IDs that already exist
- Clear localStorage and try `test-import-duplicates.json` twice

### Rollback not working
- Check browser console for errors
- Verify backup was created (check localStorage)
- Try the automated test suite for detailed logging

## Notes

- All data is stored in localStorage (browser-specific)
- Clearing browser data will remove all prompts
- Backups are temporary and cleared on successful import
- Export files are JSON and can be edited manually
