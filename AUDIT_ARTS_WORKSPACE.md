# A440 Audit + Arts Workspace Refactor Plan

## 1) Current data models and Firestore access points

### Project/WorkItem/Artifact models
- `a440/Models/Project.swift`
  - `Project` is local-first, `UUID`-keyed, with discipline/city/collaboration/signal metadata.
- `a440/Models/WorkItem.swift`
  - `WorkItem` is local-first, `UUID`-keyed, with artifact array, version history, discipline-specific metadata blocks (`music`, `realEstate`, `health`, `engineering`, `lesson`).
  - `WorkItemVersion` stores local version notes/snapshot.
- `a440/Models/Artifact.swift`
  - Attachment model (`photo/image/video/audio/pdf/document/musicXML/cad/other`) with local metadata/import support.
- `a440/ProjectViewModel.swift`
  - Persists `projects` + `workItems` in `UserDefaults`.
  - Drives most in-app project/work-item behaviors.

### Current Firestore access
- `a440/Services/BeamHubService.swift`
  - Active Firestore service (`units`, `events`, `events/{id}/submissions`, `events/{id}/signups`, `roles`, `users`, `memberships`).
- `a440/Services/FirestoreService.swift`
  - Legacy lesson/version Firestore service currently stubbed (disabled behavior before this refactor path).
- `a440/Services/StorageService.swift`
  - Legacy upload service currently stubbed (disabled behavior before this refactor path).
- `firestore.rules`
  - Existing rules for BeamHub collections + memberships.

## 2) Current non-arts modules + smallest refactor proposal

### Non-arts modules currently in app
- Real estate views
  - `a440/Views/RealEstate/InspectionsListView.swift`
  - `a440/Views/RealEstate/PropertiesListView.swift`
  - `a440/Views/RealEstate/RealEstateToolsView.swift`
- Non-arts metadata and templates
  - `a440/Models/InspectionChecklist.swift`
  - Real-estate/health/engineering metadata blocks in `a440/Models/WorkItem.swift`
  - Discipline templates in `a440/Models/Project.swift` and `a440/Models/ScheduleItem.swift`
- Root routing still includes `.realEstate`, `.health`, `.engineering`
  - `a440/Views/RootView.swift`

### Smallest safe refactor
- Keep legacy files compiling (no destructive delete).
- Move primary app entry to a new arts-first workspace shell.
- Add a dedicated `WorkspaceDiscipline` enum for arts routing (`orchestra`, `film`, `dance`, `theatre`, `visualArts`) without forcing a risky full migration of legacy `Discipline`/`WorkItem` now.
- Keep legacy modules dormant/unexposed in Phase 1 UI; migrate or remove in a later cleanup phase.

## 3) New Workspace architecture (implemented)

### Discipline enum + module routing
- New enum: `a440/Models/WorkspaceDiscipline.swift`
  - `orchestra`, `film`, `dance`, `theatre`, `visualArts`
- New routed shell: `a440/Views/Workspace/WorkspaceRootView.swift`
  - Picker-based discipline routing.
  - Orchestra implemented in Phase 1.
  - Film/Dance/Theatre/Visual Arts show routed placeholders (architecture in place).

### Firestore path adapter for beam-orchestra-platform
- New models: `a440/Models/OrchestraWorkspaceModels.swift`
  - `OrchestraProject`, `OrchestraVersion`, `OrchestraAudioTrack`, `OrchestraMembership`, `OrchestraVersionDraft`.
- New adapter: `a440/Services/OrchestraFirestoreAdapter.swift`
  - Read projects from `projects`.
  - Read versions from `projects/{projectId}/versions`.
  - Write versions to `projects/{projectId}/versions/{versionId}` with `masterVideoUrl` and `audioTracks[]`.
  - Uploads master video + stems to Storage under `projects/{projectId}/versions/{versionId}/...`.

### Membership gating
- `OrchestraFirestoreAdapter.hasActiveMembership(projectId:userId:)`
  - Checks top-level `memberships` with `projectId + userId + status=active`.
- `a440/ViewModels/WorkspaceViewModel.swift`
  - Calculates `canEditSelectedProject`.
- `a440/Views/Workspace/WorkspaceRootView.swift`
  - Shows `Add Version` only when membership is active.

## 4) Phase 1 implementation status

### Read-only: Milwaukee published chamber projects + details
- Implemented in:
  - `a440/Services/OrchestraFirestoreAdapter.swift`
    - `fetchPublishedChamberProjects(city: "Milwaukee")`
    - `fetchVersions(projectId:)`
  - `a440/Views/Workspace/WorkspaceRootView.swift`
    - Project list + detail with versions and audio tracks.

### Logged-in membership flow: Add Version
- Implemented in:
  - `a440/Views/Workspace/AddOrchestraVersionView.swift`
    - Pick master video (PhotosPicker) + pick audio stems (FileImporter).
  - `a440/ViewModels/WorkspaceViewModel.swift`
    - Membership-gated create flow.
  - `a440/Services/OrchestraFirestoreAdapter.swift`
    - Upload files to Storage, write version document with `masterVideoUrl` + `audioTracks[]`.

### App integration
- `a440/a440App.swift`
  - Main app now opens `WorkspaceRootView()` after launchpad.
- `a440/App/FirebaseAppBootstrap.swift`
  - Firebase initialize restored via `FirebaseApp.configure()`.

### Firestore rules updates
- `firestore.rules`
  - Added `projects/{projectId}` and `projects/{projectId}/versions/{versionId}` rules.
  - Added member check helper using deterministic membership ID pattern: `membership_<uid>_<projectId>`.

## Info.plist permissions
- No new permission keys were required beyond existing generated keys already present in build settings:
  - `NSCameraUsageDescription`
  - `NSMicrophoneUsageDescription`
  - `NSPhotoLibraryUsageDescription`
- Existing keys are sufficient for Phase 1 media selection/upload flows.
