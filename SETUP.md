# Coach — Setup Guide

**For: Mac, using only your web browser.** Nothing to install. No Terminal. No commands.

**Time:** about 35 minutes
**Cost:** free — you will never be asked for a card
**You need:** your Mac, your Google account, your iPhone

You will use two websites: **Firebase** (Google's, where your data lives) and **GitHub** (where the app's files live). GitHub automatically sends the app to Firebase whenever you change something.

Follow the parts in order. Each step tells you exactly what to click.

> **Nothing here can break your Mac.** You're only clicking around two websites. If a step goes wrong you can always redo it.

---

# Before you start

1. Find **`coach-repo.zip`** in your **Downloads** folder.
2. **Double-click it.** A folder called **`coach-repo`** appears next to it.
3. Open that folder. You should see **5 files**:
   - `index.html`
   - `firebase.json`
   - `firestore.rules`
   - `README.md`
   - `SETUP.md`

Leave this folder window open — you'll drag from it in Part 3.

---

# Part 1 — Set up Firebase

*This creates your account system and database. About 12 minutes.*

## 1.1 — Create the project

1. Go to **https://console.firebase.google.com**
2. Sign in with your Google account if asked.
3. Click **Create a project**.
4. For the name, type: **`coach`**
   *If it says the name is taken, Firebase adds numbers like `coach-4821`. That's fine — just remember the full name, you'll need it.*
5. Click **Continue**.
6. You'll see a screen about **Google Analytics**. **Turn the toggle OFF.** You don't need it.
7. Click **Create project**.
8. Wait about 30 seconds, then click **Continue**.

You're now on your project's dashboard.

## 1.2 — Turn on Google sign-in

1. On the left sidebar, click **Security** (under *Product categories*).
2. Click **Authentication**.
   *Older guides say "Build → Authentication". Firebase removed the Build menu — it's under Security now.*
3. Click the **Get started** button.
4. You'll see a list of sign-in providers. Click **Google**.
5. Click the **Enable** toggle (top right of that panel) so it turns blue.
6. A box appears: **Project support email**. Click it and pick your own email address.
7. Click **Save**.

Google should now show as **Enabled**.

## 1.3 — Create the database

1. Left sidebar → **Databases & Storage** → **Firestore Database**.
2. Click **Create database**.
3. It asks for a **location**. Choose **`europe-west2 (London)`**.
   ⚠️ **This cannot be changed later.** Pick London.
4. Click **Next**.
5. Choose **Start in production mode**.
   *This locks the database completely. You'll open it up correctly in the next step.*
6. Click **Create**. Wait about 30 seconds.

## 1.4 — Set the security rules

This is what stops anyone else reading your data. Do not skip it.

1. You should now be looking at your empty database. Click the **Rules** tab at the top.
2. You'll see a small code box with some text in it.
3. Click inside the box, select everything (**Cmd + A**), and delete it.
4. Copy the text below **exactly** and paste it in:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

5. Click **Publish**.

> **What this does:** it says each signed-in person can read and write only their own record, and nothing else is accessible to anyone. Without it, your data would be either locked to you *or* open to the world — this is the bit that makes it genuinely private.

## 1.5 — Get your app's config

1. Click the **⚙️ gear icon** in the left sidebar → **General**.
2. Scroll down to the section called **Your apps**.
   *Your **Project ID** is also on this page, near the top — you need it in Part 4.*
3. Click the **`</>`** icon (it's the web option, next to the Apple and Android icons).
4. **App nickname:** type `coach`
5. **Do NOT tick** "Also set up Firebase Hosting for this app."
6. Click **Register app**.
7. You'll now see a block of code. Find the part that starts `const firebaseConfig = {` and ends with `};`

**Select that whole block from `const` to `};` and copy it (Cmd + C).**

It looks like this:

```js
const firebaseConfig = {
  apiKey: "AIzaSyD-EXAMPLE-KEY-1234567890",
  authDomain: "coach-4821.firebaseapp.com",
  projectId: "coach-4821",
  storageBucket: "coach-4821.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abc123def456"
};
```

8. **Paste it somewhere safe for now** — open the **Notes** app on your Mac, paste it there, and leave it open. You'll need it in Part 4.
9. Back in Firebase, click **Continue to console**.

## 1.6 — Get your deploy key

This is the key that lets GitHub send updates to Firebase on your behalf.

1. Click the **⚙️ gear icon** in the left sidebar → **Service accounts**.
2. Click the button **Generate new private key**.
3. A warning box appears — click **Generate key**.
4. A **`.json` file downloads** to your Downloads folder. It'll have a long name like `coach-4821-firebase-adminsdk-xxxxx.json`.

**Now open that file so you can copy what's inside:**

5. Go to your **Downloads** folder and find that `.json` file.
6. **Right-click it** → **Open With** → **TextEdit**.
7. Press **Cmd + A** (select all), then **Cmd + C** (copy).
8. Leave it copied — you'll paste it in Part 3.

> ⚠️ **Treat this key like a password.** Don't email it or post it anywhere. You'll paste it into GitHub's secrets box, which is built to store exactly this. Delete the downloaded file afterwards.

---

# Part 2 — Create your GitHub repository

*About 5 minutes.*

1. Go to **https://github.com** and sign in.
2. Click the **+** in the top-right corner → **New repository**.
3. **Repository name:** type `coach`
4. Leave it set to **Public**.
   *This is fine — your personal data is never in these files. It's stored in Firebase, behind your Google login.*
5. **Don't tick** any of the "Initialize this repository" boxes.
6. Click **Create repository**.

You'll land on a mostly empty page with setup instructions. Ignore all of it — just keep this tab open.

---

# Part 3 — Add the deploy key to GitHub

*Do this before uploading files. About 3 minutes.*

1. In your new `coach` repository, click **Settings** (the tab along the top, on the right).
2. In the left sidebar, click **Secrets and variables** → then **Actions**.
3. Click the green **New repository secret** button.
4. **Name:** type exactly `FIREBASE_SERVICE_ACCOUNT`
   *(all capitals, with underscores — it must match exactly)*
5. **Secret:** click in the big box and press **Cmd + V** to paste the key you copied in step 1.6.
   *It should be a long block of text starting with `{` and containing `"type": "service_account"`.*
6. Click **Add secret**.

You should now see `FIREBASE_SERVICE_ACCOUNT` listed. You can't view it again — that's normal and intended.

7. **Now delete the downloaded `.json` file** from your Downloads folder (drag it to Trash, then empty Trash).

---

# Part 4 — Upload the app files

*About 8 minutes.*

## 4.1 — Upload the five files

1. Go back to your repository's main page (click **`coach`** at the top-left, next to your username).
2. Click the link **uploading an existing file** (in the middle of the page).
   *If you can't see it: click **Add file** → **Upload files**.*
3. Open your **`coach-repo`** folder from earlier.
4. **Select all 5 files** inside it (click one, then **Cmd + A**).
5. **Drag them** into the dashed box on the GitHub page.
   *Drag the files themselves, not the folder.*
6. Wait for all 5 to finish uploading.
7. Scroll down and click **Commit changes**.

You should now see your 5 files listed.

## 4.2 — Create config.js with your Firebase keys

Your keys live in their **own file**, separate from the app. That way future app
updates overwrite `index.html` only, and your sign-in keeps working.

1. On your repo's main page, click **Add file** → **Create new file**.
2. Filename: `config.js`
3. Paste this in, replacing each value with the ones from your own `firebaseConfig`:

```js
window.FIREBASE_CONFIG = {
  apiKey: "PASTE_YOUR_API_KEY",
  authDomain: "PASTE_YOUR_PROJECT.firebaseapp.com",
  projectId: "PASTE_YOUR_PROJECT_ID",
  storageBucket: "PASTE_YOUR_PROJECT.appspot.com",
  messagingSenderId: "PASTE_YOUR_SENDER_ID",
  appId: "PASTE_YOUR_APP_ID"
};
```

4. **Commit changes**.

> ⚠️ **Never put your keys in `index.html`.** Updating the app replaces that file
> and would wipe them, dropping the app back to local-only with no sign-in.

## 4.3 — Create the deploy instructions

This tells GitHub to send the app to Firebase automatically.

1. Go back to your repository's main page.
2. Click **Add file** → **Create new file**.
3. In the filename box at the top, type this **exactly**, including the slashes:

```
.github/workflows/deploy.yml
```

*As you type each `/`, the box will split into folders. That's correct.*

4. In the large text box below, paste this:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: ${{ secrets.GITHUB_TOKEN }}
          firebaseServiceAccount: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
          channelId: live
          projectId: PUT_YOUR_PROJECT_ID_HERE
```

5. **Replace `PUT_YOUR_PROJECT_ID_HERE`** with your actual project ID.
   *That's the `projectId` value from your config — e.g. `coach-4821`. No quotes needed.*
6. Click **Commit changes...** → **Commit changes**.

---

# Part 5 — Check it deployed

1. Click the **Actions** tab at the top of your repository.
2. You'll see a job running with a **yellow dot** 🟡. Wait about a minute.
3. It should turn into a **green tick** ✅.

**If you got a green tick**, your app is live. Use the **`.firebaseapp.com`** address:

```
https://YOUR-PROJECT-ID.firebaseapp.com
```

⚠️ **Use `.firebaseapp.com`, not `.web.app`.** Firebase serves the identical site at both, but only
`.firebaseapp.com` is pre-authorised for Google sign-in. The `.web.app` address gives
`Error 400: redirect_uri_mismatch` until you manually authorise it in Google Cloud Console.

4. Open that address in your Mac's browser. The app should load.
5. Click the **Sync** tab at the bottom, then **Sign in with Google**.
6. Sign in. You should come back to a green **Synced** badge.

**If it worked here, it will work on your phone.** If you got a red ✗ instead, jump to *If something goes wrong* at the bottom.

---

# Part 6 — Install it on your iPhone

*About 2 minutes.*

1. On your iPhone, open **Safari**.
   ⚠️ It must be Safari. Chrome on iPhone can't install home-screen apps.
2. Go to your address: `https://YOUR-PROJECT-ID.firebaseapp.com`
3. Tap the **Share** button at the bottom (a square with an arrow pointing up).
4. Scroll down the list and tap **Add to Home Screen**.
5. Tap **Add** (top right).
6. **Close Safari completely.**
7. Find the new **Coach** icon on your home screen and tap it.
8. Go to the **Sync** tab → **Sign in with Google**.

Done.

> ### One rule: always open it from the home screen icon
> Not from a Safari tab. iPhones delete website data after 7 days of not visiting, but apps added to the home screen are exempt. Your data is safely in the cloud either way, but using the icon keeps you signed in.

---

# Using the app

| Tab | What it's for |
|---|---|
| **Today** | Today's session, log your sets, log your morning weight |
| **Week** | This week's schedule and what's counting |
| **Progress** | Weight trend, strength trend, consistency |
| **Stake** | What you owe, and flagging holidays |
| **Plan** | The full programme |

### The one habit that matters

**Flag holidays before the week ends.** Stake tab → *Going away?* → set the dates → *Flag these days as away*.

The app can't tell a holiday from quietly giving up unless you tell it beforehand. An unflagged holiday gets charged as missed weeks.

### How the stake works

Four sessions count each week: **Monday basketball, Day A, Thursday basketball, Day B.**

- **3 or 4 of 4** → no charge. One miss a week is free, deliberately.
- **2 or fewer** → charged.
- **Flagged away in advance** → excused.

Miss weeks back-to-back and the charge escalates: 1× → 1.5× → 2×.

**The app can't actually take your money** — it does the accounting and holds you to the number. If you want it properly enforced, Beeminder or StickK run these same rules with real payments attached.

---

# Updating the app later

Upload the new `index.html` (Add file → Upload files → drag → Commit). It goes live
in about a minute.

**Never overwrite `config.js`** — that holds your keys and stays put forever.

Check it worked: **Sync tab → App version**. If it does not match what you deployed,
tap **Force refresh**.

---

# If something goes wrong

| Problem | What to do |
|---|---|
| **Red ✗ in the Actions tab** | Click the failed run, then the red step, to see the error. Usually the secret name is misspelled (must be exactly `FIREBASE_SERVICE_ACCOUNT`) or `projectId` still says `PUT_YOUR_PROJECT_ID_HERE`. |
| **"HTTP Error: 403"** in Actions | The key was pasted incompletely. Redo step 1.6, making sure you copied everything including the first `{` and last `}`. |
| **Site loads but the Sync tab has no sign-in button** | The config paste in step 4.2 didn't take. Reopen `index.html` and check the line starts `const FIREBASE_CONFIG = {` and ends `};` |
| **"This domain is not authorised"** | Firebase console → **Authentication** → **Settings** → **Authorized domains** → **Add domain** → enter your `.web.app` address. |
| **Sign-in does nothing on the iPhone** | You're in a Safari tab rather than the home-screen app, or on the wrong address. Use the icon and your `.web.app` address. |
| **"Missing or insufficient permissions"** | The rules in step 1.4 weren't published. Redo it and click **Publish**. |
| **Page is blank after an update** | Force-quit the app (swipe up) and reopen. Your data is in the cloud, so nothing is lost. |
| **`Error 400: redirect_uri_mismatch`** on sign-in | You're on the `.web.app` address. Switch to `.firebaseapp.com` — same site, and it's pre-authorised. |
| **`error in your yaml syntax on line 2`** | The word `yaml` ended up on line 1 of `deploy.yml`. The file must begin with `name: Deploy`. |
| **`npx failed with exit code 2`** | Usually `projectId` in `deploy.yml` doesn't match your real project ID. Check it against Settings → General. |
| **Something else** | Copy the error message and send it to me — I'll tell you what it means. |

### Undoing a bad change
Firebase console → **Hosting** → find the previous version in the list → **⋮** → **Rollback**.

---

# What it costs

Free, permanently, at your level of use. Firebase never asks for a card on the free plan.

| | Free each day | What you'll actually use |
|---|---|---|
| Database writes | 20,000 | about 15 **per week** |
| Database reads | 50,000 | about 20 **per week** |
| Storage | 1 GB | under 1 MB after a year |

You'd need to use it roughly a thousand times more heavily before cost became a question.
