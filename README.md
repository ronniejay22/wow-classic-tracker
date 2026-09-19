# Classic Roll Call

A one-page roll call for a friend group heading into WoW Classic. Everyone adds themselves — faction, PvP or PvE, classes they're eyeing — and the whole group sees the same live board. Signing up means you're in; each person can only edit their own row (from the browser they added it in). Nobody's the decider; the page just makes the spread visible.

- `index.html` — the whole app (no build step)
- `firebase-rules.json` — database rules (deployable with the CLI, or paste into the console)
- `firebase.json` / `.firebaserc` — Firebase Hosting config
- Backend: Firebase Realtime Database (free Spark plan). Hosting: Firebase Hosting at <https://wow-forever-tracker.web.app>.

## One-time setup (about 5 minutes)

You need a Google account. No credit card.

### 1. Create the Firebase project

1. Go to <https://console.firebase.google.com> and click **Create a project** (or **Add project**).
2. Name it something like `classic-roll-call`. Click **Continue**.
3. **Turn off** "Enable Google Analytics for this project". Click **Create project**, then **Continue**.

### 2. Create the Realtime Database

1. In the left sidebar: **Build → Realtime Database** → **Create Database**.
2. Location: **United States (us-central1)**. Click **Next**.
3. Security rules: pick **Start in locked mode**. Click **Enable**.
   (Don't pick test mode — those rules silently expire after 30 days and the page stops saving.)

### 3. Paste the rules

1. Still in Realtime Database, open the **Rules** tab.
2. Delete everything in the editor and paste the entire contents of [`firebase-rules.json`](firebase-rules.json).
3. Click **Publish**.

These rules let anyone with the link read and write the roster, but only the roster — every field is validated (allowed values, length caps), and nothing outside `players/` and `meta/title` can be written.

### 4. Register a web app and grab the config

1. Click the **gear icon** next to "Project Overview" → **Project settings**.
2. Scroll to **Your apps** → click the **`</>`** (Web) icon.
3. Nickname: `roll-call`. Leave "Firebase Hosting" **unchecked**. Click **Register app**.
4. You'll see a `firebaseConfig = { ... }` block. Copy the whole object.

### 5. Paste it into `index.html`

Open `index.html`, find the block marked `PASTE YOUR FIREBASE CONFIG HERE`, and replace the placeholder object with what you copied. It should look like:

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "classic-roll-call-xxxxx.firebaseapp.com",
  databaseURL: "https://classic-roll-call-xxxxx-default-rtdb.firebaseio.com",
  projectId: "classic-roll-call-xxxxx",
  storageBucket: "classic-roll-call-xxxxx.firebasestorage.app",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef"
};
```

**Check that `databaseURL` is there.** If you registered the web app before creating the database it may be missing — add it by hand using the URL shown at the top of the Realtime Database **Data** tab.

(The config is safe to publish. It identifies the project; the rules are what control access.)

### 6. Deploy

One-time: `npx firebase-tools login` (opens a Google sign-in). Then, whenever `index.html` changes:

```sh
npx firebase-tools deploy --only hosting
```

It's live at <https://wow-forever-tracker.web.app> within seconds. Commit and push to GitHub too so the repo stays the source of truth:

```sh
git add -A
git commit -m "Describe the change"
git push
```

To push a rules change from `firebase-rules.json` instead of pasting in the console: `npx firebase-tools deploy --only database`.

## Running it locally

Module scripts need to be served over HTTP (opening the file directly won't work in every browser):

```sh
python3 -m http.server 8000
# → http://localhost:8000
```

Before the config is pasted in, the page shows a "Not connected to Firebase yet" banner and keeps changes in the tab only — handy for poking at the UI.

## Troubleshooting

- **"Can't reach the roster. Check the Firebase rules."** — Step 3 was skipped or the rules didn't publish. Re-paste and hit Publish.
- **"That didn't save. Firebase rejected it."** — Same cause, or a field failed validation (name over 40 chars, note over 70). The form already caps these, so it's almost always the rules.
- **Banner says "Not connected"** — the config in `index.html` still contains `PASTE_ME`.
- **Nothing changes after deploying** — hard-refresh (Cmd+Shift+R); Firebase Hosting caches aggressively for a few seconds.
