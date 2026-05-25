Xcode Firebase Package Reset

1) Quit Xcode.
2) Delete DerivedData:
   - ~/Library/Developer/Xcode/DerivedData
3) Delete SwiftPM SourcePackages:
   - ~/Library/Developer/Xcode/SourcePackages
4) Delete the project SwiftPM cache:
   - a440.xcodeproj/project.xcworkspace/xcshareddata/swiftpm
5) Reopen the project in Xcode.
6) In the Project navigator, remove `firebase-ios-sdk` from Package Dependencies.
7) Xcode menu: File → Add Packages... → `https://github.com/firebase/firebase-ios-sdk`
   - Add to target: `a440`
   - Products: FirebaseCore, FirebaseFirestore, FirebaseFirestoreSwift, FirebaseStorage
8) Xcode menu: File → Packages → Reset Package Caches.
9) Xcode menu: File → Packages → Resolve Package Versions.
