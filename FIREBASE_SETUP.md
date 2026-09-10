# Firebase setup for the shared KAVE walls

GitHub Pages can host the page, but it cannot store visitor uploads by itself. The page is wired to Firebase for shared realtime notes and photos. Photos are compressed in the browser and stored in Firestore, so a separate Storage bucket is not required.

## 1. Create the Firebase project

1. Open the Firebase Console and create a project.
2. Add a Web app and copy its config object.
3. The Firebase web config is already installed in `index.html` for the `kave-coffee-community` project.
4. In **Authentication > Sign-in method**, enable **Anonymous**.
5. Create a **Firestore Database** in the `asia-southeast1 (Singapore)` region.

The Firebase web config is intended to be public. The security rules below protect the data; do not put an admin/service-account key in `index.html`.

## 2. Firestore rules

Paste these into **Firestore Database > Rules** and publish:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /freedomWall/{noteId} {
      allow read: if true;
      allow create: if request.auth != null
        && request.resource.data.uid == request.auth.uid
        && request.resource.data.message is string
        && request.resource.data.message.size() > 0
        && request.resource.data.message.size() <= 220;
      allow delete: if request.auth != null
        && resource.data.uid == request.auth.uid;
      allow update: if false;
    }

    match /shareGallery/{photoId} {
      allow read: if true;
      allow create: if request.auth != null
        && request.resource.data.uid == request.auth.uid
        && request.resource.data.src is string
        && request.resource.data.src.size() <= 900000;
      allow delete: if request.auth != null
        && resource.data.uid == request.auth.uid;
      allow update: if false;
    }
  }
}
```

Photos are resized and compressed by the page before being written to the `shareGallery` collection. The Firestore rules limit each photo data URL to 900 KB.

After publishing the page, an uploaded photo or note is written to Firebase and `onSnapshot` immediately updates every open browser. Without the Firebase config, the existing local-only fallback remains active.
