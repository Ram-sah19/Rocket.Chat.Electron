# Quick Start - Test PDF Fix

## ✅ Fix Applied
Added `plugins: true` to webPreferences in `src/ui/main/serverView/index.ts` (lines 213 and 254)

## 🚀 Test Now

### 1. Build & Run
```bash
cd d:\Gsoc\rocket\Rocket.Chat.Electron
yarn start
```

### 2. Test PDF Rendering
1. Login to Rocket.Chat workspace (or use https://open.rocket.chat)
2. Upload a PDF file to any channel
3. Click the PDF attachment
4. **Expected**: PDF renders correctly (not plain text)

### 3. Verify Fix
- ✅ PDF displays with proper formatting
- ✅ Can navigate pages (if multi-page)
- ✅ Can zoom in/out
- ✅ Matches browser behavior

## 📝 What Changed

**Before**: PDFs showed as plain text (garbled characters)
**After**: PDFs render properly using Electron's built-in PDF viewer

**Technical**: Electron requires `plugins: true` in webPreferences to enable the Chromium PDF plugin (PDFium)

## 🐛 If Issues Occur

1. **Clean build**:
   ```bash
   yarn clean
   yarn build
   yarn start
   ```

2. **Check console**: Press `Ctrl+Shift+I` for DevTools

3. **Test with sample PDF**: https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf

## 📚 Full Documentation
See `PDF_FIX_TESTING_GUIDE.md` for comprehensive testing instructions
