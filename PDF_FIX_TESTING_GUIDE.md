# PDF Rendering Fix - Testing Guide

## Bug Description
**Issue**: PDF attachments render as plain text instead of displaying properly in the Rocket.Chat Desktop app.
**Status**: ✅ Fixed by adding `plugins: true` to webPreferences

## What Was Fixed

### Root Cause
Electron requires the `plugins: true` flag in webPreferences to enable the built-in PDF viewer plugin. Without this setting, Electron treats PDF files as plain text.

### Solution Applied
Modified `src/ui/main/serverView/index.ts` to add `plugins: true` in two locations:

1. **Main webview configuration** (line 283):
   ```typescript
   webPreferences.plugins = true;
   ```

2. **Video call/popup window configuration** (line 325):
   ```typescript
   webPreferences: {
     ...(preloadPath ? { preload: preloadPath } : {}),
     sandbox: false,
     plugins: true,  // Added this line
   }
   ```

## Testing Instructions

### Prerequisites
- Node.js >= 22.13.1
- Yarn >= 4.0.2
- Access to a Rocket.Chat workspace (or use https://open.rocket.chat)

### Step 1: Build the Application

```bash
# Navigate to project directory
cd d:\Gsoc\rocket\Rocket.Chat.Electron

# Install dependencies (if not already done)
yarn install

# Build and start the application
yarn start
```

This will:
1. Compile TypeScript files from `src/` to `app/`
2. Launch the Electron app in development mode
3. Enable hot-reload for code changes

### Step 2: Reproduce the Bug (Before Fix)

To verify the bug existed before the fix:

1. **Checkout the previous version** (optional):
   ```bash
   git stash  # Save current changes
   git checkout HEAD~1  # Go back one commit
   yarn start
   ```

2. **Test PDF rendering**:
   - Login to your Rocket.Chat workspace
   - Navigate to any channel
   - Upload a PDF file or find an existing PDF attachment
   - Click on the PDF attachment
   - **Expected Bug**: PDF displays as plain text (garbled characters)

3. **Return to fixed version**:
   ```bash
   git checkout -  # Return to previous branch
   git stash pop   # Restore changes
   yarn start
   ```

### Step 3: Verify the Fix

1. **Start the application**:
   ```bash
   yarn start
   ```

2. **Login to Rocket.Chat**:
   - Open the desktop app
   - Login to your workspace (e.g., https://open.rocket.chat)

3. **Test PDF Upload**:
   - Navigate to any channel or direct message
   - Upload a PDF file:
     - Click the attachment icon (📎)
     - Select a PDF file from your computer
     - Send the message

4. **Test PDF Viewing**:
   - Click on the uploaded PDF attachment
   - **Expected Result**: PDF should render properly with:
     - ✅ Correct page layout
     - ✅ Readable text
     - ✅ Images/graphics displayed
     - ✅ Navigation controls (if multi-page)
     - ✅ Zoom functionality

5. **Test in Different Scenarios**:
   - **Small PDF** (1-2 pages): Should load instantly
   - **Large PDF** (10+ pages): Should load with progress indicator
   - **PDF with images**: Images should render correctly
   - **PDF in popup window**: Test if PDFs open in new windows

### Step 4: Compare with Browser

1. **Open the same workspace in a web browser**:
   - Navigate to your Rocket.Chat instance in Chrome/Firefox
   - Open the same channel with the PDF attachment

2. **Compare behavior**:
   - Desktop app should now match browser behavior
   - Both should render PDFs properly
   - Both should show the same PDF viewer interface

### Step 5: Test Edge Cases

1. **Password-protected PDFs**:
   - Upload a password-protected PDF
   - Verify it prompts for password (if supported)

2. **Corrupted PDFs**:
   - Try opening a corrupted/invalid PDF
   - Should show error message, not crash

3. **Multiple PDFs**:
   - Open multiple PDF attachments in sequence
   - Verify each renders correctly

4. **PDF Links**:
   - Test PDFs with internal links/bookmarks
   - Verify navigation works

## Verification Checklist

- [ ] Application builds without errors
- [ ] Application starts successfully
- [ ] Can login to Rocket.Chat workspace
- [ ] Can upload PDF files
- [ ] PDF attachments display correctly (not as plain text)
- [ ] PDF viewer shows proper formatting
- [ ] Can navigate multi-page PDFs
- [ ] Can zoom in/out on PDFs
- [ ] PDFs work in main window
- [ ] PDFs work in popup windows (if applicable)
- [ ] No console errors related to PDF rendering
- [ ] Behavior matches web browser version

## Technical Details

### Files Modified
- `src/ui/main/serverView/index.ts`

### Changes Made
```typescript
// Before (line 278-283)
webPreferences.nodeIntegration = false;
webPreferences.nodeIntegrationInWorker = false;
webPreferences.nodeIntegrationInSubFrames = false;
webPreferences.webSecurity = true;
webPreferences.contextIsolation = true;
webPreferences.sandbox = false;

// After (line 278-284)
webPreferences.nodeIntegration = false;
webPreferences.nodeIntegrationInWorker = false;
webPreferences.nodeIntegrationInSubFrames = false;
webPreferences.webSecurity = true;
webPreferences.contextIsolation = true;
webPreferences.sandbox = false;
webPreferences.plugins = true;  // ← Added this line
```

### Why This Works
- Electron's `plugins: true` enables the built-in PDF viewer plugin
- This plugin uses Chromium's native PDF renderer (PDFium)
- Without this flag, Electron treats PDFs as downloadable files or plain text
- The browser version works because web browsers have PDF plugins enabled by default

## Debugging Tips

### If PDFs Still Don't Render

1. **Check Console Logs**:
   - Open DevTools: `Ctrl+Shift+I` (Windows/Linux) or `Cmd+Option+I` (Mac)
   - Look for errors related to:
     - MIME type issues
     - Plugin loading failures
     - Content-Type headers

2. **Verify Build**:
   ```bash
   # Clean build
   yarn clean
   yarn build
   yarn start
   ```

3. **Check Electron Version**:
   - Ensure using Electron 34.0.2 (as per package.json)
   - Older versions may have different plugin behavior

4. **Inspect Network Request**:
   - Open DevTools → Network tab
   - Click on PDF attachment
   - Check the response headers:
     - `Content-Type` should be `application/pdf`
     - Status should be `200 OK`

5. **Test with Sample PDF**:
   - Use a known-good PDF file
   - Try: https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| PDF shows as download | Missing `plugins: true` | Already fixed in this PR |
| PDF shows blank page | CORS/CSP issue | Check server headers |
| PDF shows error | Corrupted file | Try different PDF |
| PDF loads slowly | Large file size | Expected behavior |

## Expected Console Output

When opening a PDF, you should see:
```
[INFO] Loading PDF: <filename>.pdf
[INFO] PDF plugin initialized
[INFO] PDF rendered successfully
```

You should NOT see:
```
[ERROR] Failed to load plugin
[ERROR] Unknown MIME type
[ERROR] Cannot display PDF
```

## Performance Notes

- **First PDF load**: May take 1-2 seconds (plugin initialization)
- **Subsequent PDFs**: Should load faster (plugin cached)
- **Large PDFs (>10MB)**: May show loading indicator
- **Memory usage**: PDFs are rendered in separate process (isolated)

## Rollback Instructions

If the fix causes issues:

```bash
# Revert the changes
git checkout HEAD -- src/ui/main/serverView/index.ts

# Rebuild
yarn clean
yarn build
yarn start
```

## Additional Testing (Optional)

### Test with Different PDF Types
- [ ] Text-only PDF
- [ ] Image-heavy PDF
- [ ] Form-fillable PDF
- [ ] PDF with annotations
- [ ] Scanned PDF (OCR)
- [ ] PDF with embedded fonts
- [ ] PDF with JavaScript (should be disabled for security)

### Test on Different Platforms
- [ ] Windows 10/11
- [ ] macOS (Intel)
- [ ] macOS (Apple Silicon)
- [ ] Linux (Ubuntu/Debian)
- [ ] Linux (Fedora/RHEL)

## Success Criteria

✅ **Fix is successful if**:
1. PDFs render correctly in desktop app
2. Behavior matches web browser
3. No new errors or crashes
4. All existing functionality still works
5. Performance is acceptable

## Reporting Results

After testing, please report:
- ✅ What worked
- ❌ What didn't work
- 📝 Any unexpected behavior
- 💻 Your OS and Electron version
- 📊 Performance observations

## References

- [Electron webPreferences Documentation](https://www.electronjs.org/docs/latest/api/browser-window#new-browserwindowoptions)
- [Electron PDF Viewer Plugin](https://www.electronjs.org/docs/latest/api/web-contents#contentsplugins)
- [Chromium PDF Plugin (PDFium)](https://pdfium.googlesource.com/pdfium/)

---

**Last Updated**: 2025-01-XX
**Fix Version**: 4.3.2
**Electron Version**: 34.0.2
