# Connect login and automatic saving to Firebase

The login screen, account creation, password reset, sign-out and cloud synchronization are implemented in plain JavaScript. Add your own Firebase project configuration to activate them. Guest mode remains available without Firebase.

1. In Firebase Console, create/select a project and register a **Web app**.
2. Copy its web configuration into `public/firebase-config.js` (apiKey, authDomain, projectId and appId). This public configuration identifies your app; never put a service-account private key or an AI key there.
3. Under **Authentication > Sign-in method**, enable **Email/Password**. Configure password policy and reset-email templates as needed.
4. Under Authentication settings, add your actual Netlify hostname and any custom domain to **Authorized domains**. Add `localhost` / `127.0.0.1` if using those for development.
5. Create a **Cloud Firestore Standard** database using the default database. In its Rules tab, replace the rules with `firestore.rules` from this project and publish them. Do not leave the database in open test mode.
6. Redeploy to Netlify. Create an account through the login screen, edit your profile or finish a study session, and wait for **Saved to your account**. Sign in on a second device to check the same progress appears.

## What saves

The account workspace lives at `users/{Firebase UID}/workspaces/main`. It includes your profile, subjects, chapters, homework, exams, timetable, lectures, saved questions, marks, completed study sessions, revisions, mistakes, notes and preferences. The same validated backup format is used for cloud data. Passwords are handled by Firebase Authentication; AI keys are never included in study data.

The focus timer itself stays in this tab, scoped to the account. Finished sessions sync automatically. Guest data is separate and is not automatically uploaded into an account. Export your guest workspace, sign in, and import the backup if you want to move it.

Changes save locally first, then upload after a short delay. Firestore listeners deliver changes from other devices. Failed writes keep a pending local copy and retry when the browser comes online or you choose **Profile > Sync now**. First sign-in/workspace loading needs an internet connection. Existing signed-in pages can keep local changes while temporarily offline.

Concurrent edits use revision checks. If two devices edit the same workspace, automatic saving pauses instead of silently discarding one version. Export local changes first, then use **Profile > Load cloud copy** to resolve the conflict. This version does not automatically merge concurrent edits. Sign-out waits for successful cloud saving; export or resolve a sync error before signing out.

Workspace resets/imports update the signed-in cloud workspace too. Resetting a profile does not delete the Firebase login account. Account deletion can be managed in Firebase Console. Each workspace is limited to 850 KB for cloud saving; export backups and trim long notes if it reaches that limit. Monitor Firebase usage and quotas for your deployment.

## Verification checklist for your Firebase project

- Create an account, sign out, sign back in, and send a password reset email.
- Verify a saved task and finished timer session appear on another device.
- Verify a different account starts with a separate clean workspace.
- In Firebase Rules Playground, verify an unauthenticated request and another UID cannot read/write a user's workspace; the owner can.
- Test a temporary network outage and a concurrent edit from two devices.

Automated local tests cover synchronization, retry, reload recovery and conflict handling using a mock remote store. Live authentication, email delivery, Firestore rules and cross-device access must be verified after connecting your project.

References: [Firebase web setup](https://firebase.google.com/docs/web/alt-setup), [email/password authentication](https://firebase.google.com/docs/auth/web/password-auth), [Firestore access rules](https://firebase.google.com/docs/firestore/security/rules-conditions).

Optional browser smoke tests: with Playwright and its Edge browser available, run `node tests/auth-browser.mjs`. Set `NORTHSTAR_RUNTIME` to a node_modules directory when using a bundled Playwright installation. The test intercepts Firebase requests with mocks and never contacts a real project. Screenshots are written to ignored `tests/artifacts/`.
