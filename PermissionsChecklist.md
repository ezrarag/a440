# Permissions QA Checklist

Use this checklist to verify that the app no longer crashes on TCC (privacy) violations and that denied permissions show a blocking screen instead of a crash.

## Info.plist keys (app target)

Confirm these keys are present in the **app target** (Xcode → Target → Info or Build Settings → INFOPLIST_KEY_*):

| Key | Purpose |
|-----|--------|
| `NSCameraUsageDescription` | Camera for scanning sheet music and capturing media |
| `NSMicrophoneUsageDescription` | Tuner and practice recording |
| `NSPhotoLibraryUsageDescription` | Import photos/videos (scores, practice media) |
| `NSPhotoLibraryAddUsageDescription` | Save exported recordings/media to library |

If any key is missing, the app can **crash** when that capability is used (e.g. opening camera without `NSCameraUsageDescription`).

---

## Steps to reproduce the original crash

1. **Fresh install** (or delete app and reinstall).
2. Do **not** grant any permissions when prompted by the system (or revoke in Settings → Privacy if already granted).
3. Open the app and go to the flow that uses the **camera** (e.g. Add Piece → tap **Camera**).
4. **Before fix**: App crashes (TCC violation; missing or insufficient usage description).
5. **After fix**: No crash; user sees a blocking screen with explanation and “Open Settings” (or system permission prompt if not yet determined).

---

## Validation on device (fresh install)

1. Delete the app if installed.
2. Install from Xcode (Run on device) or install a TestFlight build.
3. **Camera**
   - Navigate to Add Piece (or any screen that opens the camera).
   - Tap **Camera**.
   - **Expected**: Either system permission dialog (first time) or in-app blocking screen (if denied). **No crash.**
   - If you tap “Open Settings”, the app’s Settings page opens.
4. **Photo Library**
   - On the same screen, tap **Library** (if shown).
   - **Expected**: Either system permission dialog or in-app blocking screen if denied. **No crash.**
5. **Microphone** (tuner / recording)
   - Open Tuner or start a practice recording.
   - **Expected**: System microphone permission dialog on first use. No crash. (If you add a microphone blocking screen, verify it appears when denied.)

---

## Validation on TestFlight

1. Install the **new build** from TestFlight (with plist keys and permission flow fixes).
2. On a device that has **never** had the app, or after deleting and reinstalling:
   - Repeat the same flows as in “Validation on device” (camera, library, microphone).
3. Confirm:
   - No crash when tapping Camera or Library.
   - If permission is denied, the blocking screen appears with “Open Settings”.
   - After granting permission in Settings, returning to the app and retrying the action works.

---

## Smoke test (permissions)

- [ ] Fresh install runs without crash.
- [ ] Add Piece → Camera: no crash; prompt or blocking screen.
- [ ] Add Piece → Library: no crash; prompt or blocking screen.
- [ ] Blocking screen “Open Settings” opens app settings.
- [ ] After granting camera in Settings, Camera opens successfully.
- [ ] Tuner / recording: microphone prompt or behavior correct; no crash.

---

## Code audit reference (permission-sensitive features)

| Feature | Permission | Entry point | Usage description |
|--------|------------|-------------|--------------------|
| Score scan / add piece (camera) | Camera | `AddPieceView` → Camera button | `NSCameraUsageDescription` |
| Score scan / add piece (library) | Photo Library | `AddPieceView` → Library, `PhotosPicker` | `NSPhotoLibraryUsageDescription` |
| New version (video/photo) | Photo Library | `NewVersionView` → `PhotosPicker` | `NSPhotoLibraryUsageDescription` |
| Tuner | Microphone | `TunerView` → Start | `NSMicrophoneUsageDescription` |
| Practice recording | Microphone | `AudioRecorder` / session UI | `NSMicrophoneUsageDescription` |
| Save to Photos (future) | Photo Library Add | — | `NSPhotoLibraryAddUsageDescription` |

All of the above should have a corresponding key in the app target Info.plist. Camera and photo library entry points are gated by `PermissionService` and show `PermissionBlockingView` when denied.
