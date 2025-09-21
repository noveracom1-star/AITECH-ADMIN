# AITECH Admin — Live Links feature

This tiny README explains how to enable the "Live Links" input in `index.html` and have the links stored in Firestore collection `live_links`.

What I changed
- Added an inline input and "Add Live Link" button in `index.html` (top of the Meeting Links card).
- When clicked the button writes a document to Firestore collection `live_links` with fields: `url`, `title`, `createdAt`.
- Added a realtime listener that displays `live_links` in the dashboard and allows deletion.

Firebase setup
1. Create a Firebase project at https://console.firebase.google.com/ and enable Firestore (in native mode).
2. In the Firebase console go to Project Settings -> General, get your web app config and replace the `firebaseConfig` object in `index.html` if needed. The file already contains a config placeholder for the `aitech-1e85c` project.
3. Configure Firestore rules for simple testing (development only). In the Firestore rules tab, you can temporarily allow reads/writes like:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

IMPORTANT: The above rule is completely open and should only be used for local development. For production restrict access to authenticated admin users.

Recommended production rule example (requires authentication and admin flag in user document):
```
service cloud.firestore {
  match /databases/{database}/documents {
    match /live_links/{docId} {
      allow read: if true; // public read
      allow write: if request.auth != null && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    // other collection rules...
  }
}
```

How to test locally
1. Serve the `index.html` file. You can use a simple HTTP server. In PowerShell (Windows):

```powershell
# If you have Python 3 installed
python -m http.server 8000; # then open http://localhost:8000/index.html

# Or using Node (if http-server is installed globally)
# npx http-server -p 8000
```

2. Open `http://localhost:8000/index.html` and find the Meeting Links card.
3. Enter a live URL (e.g. `https://example.com/live/abc`) and optional title, click "Add Live Link".
4. Check the Firestore console -> `live_links` collection to see the saved document.
5. The Live Links list updates in realtime; you can delete items using the Delete button.

Edge cases considered
- Empty URL validation: the UI shows a toast and does not write an empty URL.
- Realtime listener errors: UI shows a message and logs the error.
- Deletion confirmation to avoid accidental deletes.

Next steps / improvements
- Add URL validation (regex) and optional domain whitelist.
- Show createdAt formatted time per link in UI.
- Add editing/updating of live links.
- Harden Firestore security rules before deploying.

If you'd like, I can also add a small unit test harness or an admin-only UI layer for the live links feature.