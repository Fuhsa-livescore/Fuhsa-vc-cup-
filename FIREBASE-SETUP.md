# FUHSA Live Score — Firebase connection setup

The Firebase web app configuration for project `fuhsa-live-score` is already inserted in index.html. Cross-phone sync will only work after enabling Authentication, creating Firestore, adding an admin account, and publishing secure Firestore rules below.

## A. Enter your Firebase web config
1. Firebase Console → your existing project → Project settings → Your apps → select/add the Web app.
2. Copy the `firebaseConfig` object.
3. In `index.html`, replace the six `PASTE_...` values in `FIREBASE_CONFIG` with the exact values from Firebase. Keep the property names and quotation marks.
4. The Firebase web config is intended for client apps; never paste a service-account private key into the website.

## B. Enable admin email/password sign-in
Firebase Console → Build → Authentication → Get started → Sign-in method → enable Email/Password → Save.
Then Authentication → Users → Add user. Create the administrator's email/password. Do not place the password in source code.

## C. Create Firestore
Firebase Console → Build → Firestore Database → Create database → Production mode.

## D. Add secure Firestore rules
Before using the app, replace `ADMIN_UID_FROM_AUTH_USERS` below with the UID shown for your admin account in Authentication → Users. In Firestore → Rules, publish these rules:

```rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAdmin() {
      return request.auth != null && request.auth.uid == 'ADMIN_UID_FROM_AUTH_USERS';
    }
    match /appData/liveState {
      allow read: if true;
      allow create, update, delete: if isAdmin();
    }
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

Only the specific admin UID can change the shared match data. Do not publish rules that allow every user to write. Add a separate UID for each additional trusted admin if needed.

## E. Publish
Upload/replace `index.html`, `sw.js`, `manifest.json`, `icon-192.png`, and `icon-512.png` in the existing GitHub Pages repository. Commit changes, wait for Pages to publish, then open the website online. Refresh/close and reopen any installed PWA to update its cached version.

## F. Test on two phones
1. Open the public site on both phones; the same fixtures and scores should appear.
2. On the admin phone, tap Admin and sign in with the Firebase email/password.
3. Create a test match or change a score. The other phone should receive the update automatically.
4. If it fails, read the status message shown below the header and check Firebase Console → Firestore → Rules and Authentication → Users.

## Data note
The app uses one Firestore document, `appData/liveState`, for the teams and match array. This is suitable for a small tournament app but has Firestore's document size limit (1 MiB). For a much larger tournament, separate documents per match would be preferable. Local storage is retained as an offline display cache; only successful Firestore writes are shared between phones.
