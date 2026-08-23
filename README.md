# Coach

Personal training + accountability app. Single self-contained HTML file, no build step
and no dependencies. Works entirely offline with local storage; optionally syncs to
Firestore behind Google sign-in.

## Deploy (Firebase Hosting — recommended)

Firebase Hosting is preferred over GitHub Pages because it serves the app from the
same origin as the Firebase auth handler. `signInWithRedirect` silently fails in
Safari and iOS home-screen apps when the auth domain is cross-origin, so a
cross-origin host breaks Google sign-in on exactly the device this is built for.

```sh
npm i -g firebase-tools
firebase login
firebase init hosting      # public dir: .   single-page app: no
firebase deploy
```

Live at `https://<project>.web.app`.

## Firebase setup

1. **Authentication** → Sign-in method → enable **Google**.
2. **Authentication** → Settings → Authorized domains → add your hosting domain.
3. **Firestore Database** → create (production mode).
4. Publish `firestore.rules` (`firebase deploy --only firestore:rules`).
5. Project settings → Web app → copy the `firebaseConfig` object.
6. Open the app → **Sync** tab → paste the config → **Connect** → sign in.

The `firebaseConfig` values are public by design; access is controlled by the
security rules in `firestore.rules`, which restrict every user to `users/{their-uid}`.

## Installing on iPhone

Open the hosting URL in **Safari** → **Share** → **Add to Home Screen**.
Use the Home Screen icon rather than a Safari tab: iOS clears script-writable
storage after 7 days of inactivity for sites used in a tab, and standalone
home-screen apps are exempt.

## Data model

Single Firestore document per user at `users/{uid}`:

| Field | Contents |
|---|---|
| `log` | `{ 'YYYY-MM-DD': { type, done, sets } }` |
| `weights` | `{ 'YYYY-MM-DD': kg }` |
| `away` | `[{ from, to, reason }]` |
| `stake` | amount, currency, cause, paid, escalate |
| `settings` | startWeight, goalWeight |
| `updatedAt` | epoch ms, used for merge resolution |

Signing in merges local and cloud data by key rather than overwriting; the most
recently saved version wins per entry. Without a config the app runs local-only.

## Programme

Built around fixed Monday/Thursday basketball. Day A (lower-body led) on Tuesday or
Wednesday, Day B (upper-body led, legs kept fresh) on Sunday, easy 5km Saturday.

Four sessions are scored each week: Monday basketball, Day A, Thursday basketball,
Day B. 3 or 4 of 4 is on track — one miss a week is free by design. 2 or fewer
triggers the stake. Consecutive off-track weeks escalate 1x, 1.5x, 2x. Days flagged
as away in advance are excused and reset the escalation.

## Continuous deployment from GitHub

The code lives in GitHub; Firebase Hosting serves it. Push to `main` deploys.

The quickest way to wire this up is to let the CLI do it, which creates the
service-account secret in the repo for you:

```sh
firebase init hosting:github
```

That overwrites `.github/workflows/` with a generated equivalent of the workflow
included here. If setting it up by hand instead, replace `YOUR_PROJECT_ID` in
`.github/workflows/deploy.yml` and add a `FIREBASE_SERVICE_ACCOUNT` repository
secret containing a service-account JSON key with the Firebase Hosting Admin role.

Note that GitHub *Pages* is deliberately not used: it is cross-origin to the
Firebase auth handler, which breaks `signInWithRedirect` in Safari and iOS
home-screen apps with no error message.
