# 21 Forth Cres — Household Ledger

A private financial dashboard for tracking household net worth, spending,
contributions, savings goals, and monthly forecasts. Single static page
(`index.html`), gated behind Google sign-in, backed by a private Firestore
database.

## Why there's no data in this repo

This repository is **public**, and the page is served as plain static
files (GitHub Pages) — anyone who can load the URL can also view-source it
or `git clone` this repo. So no real balances, transactions, or targets
are ever committed here. `index.html` ships as an empty shell; every
widget loads its data live from Firestore, only after a Google sign-in
that Firestore's own security rules approve.

If you're setting this up for the first time, you'll have a separate,
**local-only** JSON file with your real household data (sent to you
outside of this repo). Use it once, in step 5 below, then keep it
somewhere private — never commit it here.

## One-time setup

### 1. Create a Firebase project
Go to <https://console.firebase.google.com>, click **Add project**, and
follow the prompts (Google Analytics is optional, skip it if you like).

### 2. Register a Web App
In the project's dashboard, click the `</>` (Web) icon to add a web app.
Give it any nickname. Firebase will show you a `firebaseConfig` object
that looks like:

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

Copy this whole object. In `index.html`, find the `firebaseConfig` object
near the bottom of the file (inside the last `<script type="module">`
block) and replace the placeholder values with your real ones. These
values are not secret — they're meant to be public in client-side code —
so it's fine to commit this change.

### 3. Enable Google sign-in
In the Firebase Console: **Authentication** > **Sign-in method** > enable
**Google**. Add your own and your partner's Google account as the support
email if asked.

### 4. Add your GitHub Pages domain as authorized
Still under **Authentication** > **Settings** > **Authorized domains**,
add `<your-github-username>.github.io` (Firebase adds `localhost` by
default, which is handy for testing locally too).

### 5. Create the Firestore database
**Firestore Database** > **Create database** > pick a region close to you
> start in **production mode** (the rules below take over immediately).

### 6. Lock it down with security rules
Open `firestore.rules` in this repo, replace the two placeholder emails
with your and your partner's real Google account emails, then paste the
whole file into **Firestore Database** > **Rules** in the console and
click **Publish**. This is what actually restricts who can read or write
your data — the sign-in screen in the app is just UI.

Also fill in the same two emails in the `ALLOWED_EMAILS` array near the
bottom of `index.html`, so the app can show a friendly "not authorized"
message instead of a confusing permission error if the wrong account
signs in. This part is cosmetic; the Firestore rules are what matters.

### 7. Enable GitHub Pages
This repo has a GitHub Actions workflow (`.github/workflows/deploy-pages.yml`)
that deploys on every push to `main`. In this repo on GitHub: **Settings**
> **Pages** > under **Build and deployment**, set **Source** to
**"GitHub Actions"**. Your site will be live shortly after at:

```
https://<your-github-username>.github.io/21ForthCresFinance/
```

### 8. Seed your real data
Once the site is live and Firebase is wired up:

1. Open the site and sign in with Google.
2. You'll see an empty dashboard with a note that there's no cloud data
   yet.
3. Scroll to **Widget 09 · Monthly Data Update**, paste the contents of
   your local seed data file into the box, and click **Apply Locally** to
   preview it.
4. If it looks right, click **Save to Cloud**. From then on, both of you
   will see the same live data whenever you sign in.
5. Delete or securely store the local seed file — it's no longer needed
   day-to-day; the JSON textarea + "Copy Current Data as JSON" button in
   Widget 09 is how you'll read/write full snapshots going forward, and
   the data itself now lives in Firestore.

## Updating data each month

Widget 09 (**Monthly Data Update**) accepts a JSON object with any of
these top-level keys — only the keys you include get replaced, everything
else is left as-is:

```
household, principles, netWorth, moneyApproaches, spend, contrib,
savingsGoals, nextMonthConfig, story, actions, meta
```

Workflow:
1. Click **Copy Current Data as JSON** to grab what's currently loaded.
2. Edit it (add the new month's figures, update net worth, write this
   month's Story of the Month, append new Rolling Actions, etc.) in a
   text editor.
3. Paste the edited JSON back into the box and click **Apply Locally** to
   preview the change.
4. If it looks right, click **Save to Cloud** to make it live for both of
   you.

## Local testing

You can open `index.html` directly in a browser (`file://`) to check
layout and styling, but sign-in won't work over `file://` — Firebase Auth
needs `http://` or `https://`. Firebase's authorized domains list
includes `localhost` by default, so running any static file server
locally (e.g. `python3 -m http.server`) and visiting
`http://localhost:8000` will let you test the full sign-in flow before
it's live on GitHub Pages.

## Architecture notes

- **Hosting:** GitHub Pages, static files only, no build step.
- **Auth:** Firebase Authentication, Google provider only.
- **Storage:** One Firestore document (`household/snapshot`) holding the
  entire app state as JSON. Simple, and plenty for two people editing a
  few times a month.
- **Security:** Enforced by `firestore.rules`, not by the app's UI. The
  sign-in gate in `index.html` only controls what's *displayed* — the
  rules control what Firestore actually *allows*.
- **No secrets in git:** The Firebase config values committed in
  `index.html` are safe to be public (that's how Firebase web apps are
  designed to work). The household's real financial figures are not —
  they only ever exist in Firestore and in whatever local file you use to
  seed/update it.
