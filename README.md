# Expense Tracker

A free, ad-free expense tracker you own. Snap photos of receipts (or upload PDFs), tag them by trip, employer, date, and expense type, track what's been reimbursed, and export everything as a single document ready for Google Drive.

Your data lives on your own devices and (optionally) in your own Google Drive. No accounts, no subscriptions, no scan limits.

---

## What's in this folder

| File | Purpose |
|---|---|
| `index.html` | The entire app |
| `manifest.json` | Makes the app installable to your home screen |
| `sw.js` | Service worker — makes the app work offline |
| `icon-192.png`, `icon-512.png` | App icons |

Keep all five files together in the same folder.

---

## 1. Setup

### Option A — GitHub Pages (recommended)

This gives you a permanent URL that works on your phone and computer, and enables install + sync.

1. Go to github.com and create a new repository (e.g. `expense-tracker`). Public is fine.
2. Upload all five files from this folder to the root of the repository.
3. In the repo, go to **Settings → Pages**. Under "Build and deployment", set Source to **Deploy from a branch**, pick the `main` branch and `/ (root)` folder, and save.
4. After a minute, your app is live at:
   `https://YOUR-USERNAME.github.io/expense-tracker/`

Open that URL on any device. Done.

### Option B — Run it locally on your computer

From a terminal in this folder:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000` in your browser. Everything works, including offline caching and Drive sync (add `http://localhost:8000` as an authorized origin — see the sync section).

> Note: double-clicking `index.html` (a `file://` URL) mostly works, but installing, offline mode, and Drive sync require a real URL (GitHub Pages or localhost).

---

## 2. Install it like an app

**Android (Chrome):** Open your app URL → tap the **⋮** menu → **Add to Home screen** (or accept the install banner). It gets its own icon and opens full-screen.

**iPhone/iPad (Safari):** Open your app URL → tap the **Share** button → **Add to Home Screen**.

**Computer (Chrome/Edge):** Open your app URL → click the small **install icon** at the right end of the address bar (or ⋮ menu → "Install Expense Tracker"). It becomes a standalone windowed app.

Once installed, the app opens instantly and works with no signal — receipts you add offline are saved locally and sync later.

---

## 3. Daily use

### Adding a receipt
1. Open the **＋ Add** tab.
2. Tap **Snap receipt photo** (camera) or **…or choose a photo / PDF** (gallery, email attachments, e-tickets). PDFs are converted to images automatically (first 3 pages).
3. Optional: tap **✨ Auto-fill from receipt** to have AI read the photo and fill in the vendor, date, amount, currency, and expense type. Always double-check the amount.
4. Fill in **Trip** and **Employer / client** — these fields remember and suggest everything you've typed before.
5. Pick an expense type chip and hit **Save receipt**.

### Browsing and grouping — Receipts tab
Filter by trip, employer, expense type, reimbursement status, or date range. Group the list by trip, employer, month, or type. Every group shows its own running total. Tap any receipt to view it full size, edit it, or delete it.

### Tracking reimbursements
- Every receipt starts as **Pending** (orange badge).
- Tap a receipt → pick the date the money landed → **Mark reimbursed**. Green badge with the date appears everywhere.
- The header always shows your total outstanding — what you're still owed.

### History tab
A ledger of your reimbursements over time:
- Lifetime totals (reimbursed vs. still awaiting)
- Everything still pending, at the top
- Reimbursed receipts grouped by the **month the money came in**, with monthly totals — handy for spotting an employer that's slow to pay.

### Reports → Google Doc — Report tab
1. Set a title, your name, and filters (trip, employer, dates). It defaults to **not-yet-reimbursed only**, which is usually what you want to submit.
2. Check the on-screen summary, then tap **Download Word doc (.doc)**. The file contains a summary by expense type, an itemized table (with reimbursement status), and every receipt image with a caption.
3. Upload the .doc to **Google Drive** and open it — Drive converts it to a Google Doc automatically. Share or submit from there.
4. When you get paid, come back with the same filters and tap **✓ Mark these receipts reimbursed today** to flip the whole batch at once.

There's also **Print / save as PDF** if PDF is preferred.

---

## 4. Free sync across devices (Google Drive)

Sync keeps one file (`expense-tracker-sync.json`) in your own Google Drive and merges it with each device — newest edit wins, deletions carry over. It's free forever because it uses your own Drive storage. One-time setup, about 5 minutes:

1. Go to **console.cloud.google.com** and sign in with your Google account.
2. Create a **new project** (name it anything, e.g. "Expense Tracker").
3. In the search bar, find **Google Drive API** and click **Enable**.
4. Go to **APIs & Services → OAuth consent screen**:
   - User type: **External**, then fill in the app name and your email.
   - You don't need to submit for verification. Either add your own Google account under **Test users**, or use **Publish app** — both work for personal use. (Unverified apps show a warning screen on first sign-in; click "Advanced → Go to app" — it's your own app.)
5. Go to **APIs & Services → Credentials → Create credentials → OAuth client ID**:
   - Application type: **Web application**
   - Under **Authorized JavaScript origins**, add your app's origin(s):
     - `https://YOUR-USERNAME.github.io`
     - `http://localhost:8000` (if you run it locally)
   - Create, then copy the **Client ID** (ends in `.apps.googleusercontent.com`). No client secret is needed.
6. In the app: **Report tab → Sync & AI settings** → paste the Client ID → **Save settings**.
7. Tap **⇅ Sync with Google Drive** and approve access on each device once.

**Auto-sync:** with the "Auto-sync when the app opens" box checked (it is by default), the app quietly syncs every time you open it. If the browser blocks the silent sign-in (usually only the very first time on a device), you'll get a toast asking you to tap the sync button once — after that, auto-sync runs cleanly.

The app requests only the `drive.file` permission, meaning it can see and touch **only the one file it creates** — nothing else in your Drive.

---

## 5. AI auto-fill (optional)

The **✨ Auto-fill** button sends the receipt image to Claude, which extracts the vendor, date, amount, currency, and expense type.

- Get an API key at **console.anthropic.com** (Settings → API keys).
- Paste it in **Report tab → Sync & AI settings** → Save.
- Cost is roughly **half a cent per receipt** — a full season of expenses costs less than a coffee. This is the only non-free feature; everything else works without it.
- The key is stored only on your device and is sent only to Anthropic's API.

---

## 6. Backups

**Backup data** (Report tab) downloads a single JSON file containing every receipt, photo included. **Restore** merges a backup back in — it's smart about timestamps, so restoring an old backup won't clobber newer edits. Even with Drive sync on, it's worth downloading a backup once in a while (especially on iPhone, where Safari can evict site data from apps you haven't opened in months).

---

## 7. Data & privacy

- Receipts are stored in your browser's IndexedDB on each device.
- Sync data goes only to a file in **your** Google Drive.
- Auto-fill images go only to Anthropic's API, only when you tap the button.
- Nothing else leaves your device. There is no server, no analytics, no ads.

---

## 8. Troubleshooting

**"Persistent storage isn't available" banner** — you're viewing the file in a sandboxed preview. Open the app from its real URL (GitHub Pages or localhost).

**Sync fails with an origin/client error** — the URL you're using isn't listed under Authorized JavaScript origins in Google Cloud. Add it exactly (scheme + domain, no path, no trailing slash) and wait a few minutes.

**Google shows "unverified app" on first sign-in** — expected for a personal OAuth app. Advanced → continue, or add yourself as a test user.

**Auto-fill says it needs an API key** — the keyless mode only works when the app runs inside claude.ai; on your own hosting, add an Anthropic API key in settings.

**PDF conversion fails** — pdf.js loads from a CDN, so the first PDF needs an internet connection.

**Updated the code but the app looks old** — the service worker caches aggressively. Bump the `CACHE` name in `sw.js` (e.g. `expense-tracker-v2`) when you deploy changes, or in the browser: DevTools → Application → Service workers → Update.
