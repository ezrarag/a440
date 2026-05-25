# Share Extension Setup: A440 Import

This guide explains how to configure the Share Extension for importing files from email apps (Outlook, Mail, Gmail) into A440.

## Overview

The Share Extension allows users to:
1. Open an email attachment in their mail app
2. Tap "Share" and select "A440 Import"
3. Files are copied to a shared container
4. Main app opens to complete the import flow

## Xcode Configuration Steps

### Step 1: Add App Group Capability (Main App)

1. Select the `a440` target in Xcode
2. Go to "Signing & Capabilities"
3. Click "+ Capability"
4. Add "App Groups"
5. Add group: `group.com.beamthinktank.a440`

### Step 2: Create Share Extension Target

1. File → New → Target
2. Choose "Share Extension" (iOS)
3. Product Name: `A440Import`
4. Language: Swift
5. Click "Finish"
6. When prompted to activate the scheme, click "Activate"

### Step 3: Configure Share Extension

1. Delete the auto-generated `ShareViewController.swift` and storyboard
2. Copy `A440Import/ShareViewController.swift` from the project
3. Copy `A440Import/Info.plist` (replace the generated one)
4. Copy `A440Import/A440Import.entitlements`

### Step 4: Add App Group to Extension

1. Select the `A440Import` target
2. Go to "Signing & Capabilities"
3. Click "+ Capability"
4. Add "App Groups"
5. Add the SAME group: `group.com.beamthinktank.a440`

### Step 5: Add URL Scheme (Main App)

1. Select the `a440` target
2. Go to "Info" tab
3. Expand "URL Types"
4. Add a new URL Type:
   - Identifier: `com.beamthinktank.a440`
   - URL Schemes: `a440`
   - Role: Editor

### Step 6: Configure Extension Activation Rules

The Info.plist already includes activation rules for:
- Files (up to 20)
- Images (up to 20)
- Movies (up to 10)

To add support for more specific types, modify the `NSExtensionActivationRule` in `A440Import/Info.plist`.

### Step 7: Embed Extension in App

1. Select the `a440` target
2. Go to "General" → "Frameworks, Libraries, and Embedded Content"
3. Verify `A440Import.appex` is listed (Xcode usually does this automatically)

## Testing

### Test with Outlook iOS:
1. Open Outlook
2. Open an email with a PDF attachment
3. Tap the attachment to preview
4. Tap the Share button
5. Select "A440 Import"
6. Complete the import flow

### Test with Apple Mail:
1. Open Mail
2. Open an email with an attachment
3. Long-press the attachment
4. Tap "Share"
5. Select "A440 Import"

### Test with Files app:
1. Open Files
2. Navigate to a PDF/audio/video file
3. Long-press and tap "Share"
4. Select "A440 Import"

## Debugging

### Extension doesn't appear in Share Sheet:
- Ensure the extension is embedded in the app bundle
- Clean build folder and rebuild
- Check that activation rules match the file types

### Files not appearing in main app:
- Verify both targets have the same App Group
- Check that UserDefaults suite name matches
- Ensure deep link URL scheme is configured

### Deep link not working:
- Verify URL scheme is registered in main app Info.plist
- Check `onOpenURL` handler in `a440App.swift`
- Try manually opening `a440://import?batch=test` in Safari

## File Type Support

Currently supported:
- PDF (`.pdf`)
- Images (`.jpg`, `.png`, `.heic`, etc.)
- Video (`.mp4`, `.mov`, `.m4v`, etc.)
- Audio (`.mp3`, `.m4a`, `.wav`, etc.)
- MusicXML (`.musicxml`, `.mxl`)
- Generic files

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Mail App      │────▶│  A440Import Ext  │────▶│   A440 App      │
│  (Outlook/Mail) │     │  (Share Sheet)   │     │  (Import Flow)  │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                               │                        ▲
                               │                        │
                               ▼                        │
                        ┌──────────────────┐            │
                        │  Shared App      │────────────┘
                        │  Group Container │
                        │  (UserDefaults   │
                        │   + Files)       │
                        └──────────────────┘
```

## Files Created

- `A440Import/ShareViewController.swift` - Share extension view controller
- `A440Import/Info.plist` - Extension configuration
- `A440Import/A440Import.entitlements` - App group entitlement
- `a440/a440.entitlements` - Main app entitlement
- `a440/Models/ImportSource.swift` - Import metadata model
- `a440/Services/ImportService.swift` - Import handling service
- `a440/Views/ImportInboxView.swift` - Import flow UI

## Known Limitations

1. **Email metadata**: Share extensions don't receive sender/subject from email apps. Users can manually enter "From" info.

2. **Large files**: Very large files may timeout. Consider implementing background upload.

3. **App not installed**: If A440 isn't installed, the deep link will fail silently.
