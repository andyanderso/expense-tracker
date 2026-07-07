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

Keep these files together in the same folder (plus `README.md` and `CLAUDE.md`).

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

### Foreign-currency receipts & card reconciliation
When a receipt is in a local currency (CAD, CLP, etc.) but your card was charged in USD, use the **Card charges — reconcile USD** section on the Add form: for each separate card charge tied to that receipt, tap **+ Add card charge** and enter the USD amount plus (optionally) a screenshot of that line on your statement. This supports invoices billed as multiple separate charges — e.g. a hotel invoice where lodging and parking hit the card separately — so the sum of the card charges can be reconciled against the one receipt total. The app then:
- shows the combined USD total on the receipt (local amount on top, "$xx.xx card" underneath)
- uses the **sum of the USD card charges in all totals and report summaries** (that's the real reimbursable number), falling back to the receipt currency when no card charge is entered
- pairs each statement screenshot with the receipt photo in the exported report, labeled with its own amount, so the reviewer sees the receipt and the proof of every USD charge side by side

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
- **One receipt:** tap it → pick the date the money landed → **Mark reimbursed**. (The Edit form also has Reimbursed + date fields.) Green badge with the date appears everywhere.
- **Several at once:** on the Receipts tab, tap **Select**, tap the receipts (or **Select all** for everything currently filtered), set the date, and hit **✓ Reimbursed** — or **Pending** to un-mark. Tip: filter to a trip or employer first, then Select all.
- The header always shows your total outstanding — what you're still owed.

### History tab
A ledger of your reimbursements over time:
- Lifetime totals (reimbursed vs. still awaiting)
- Everything still pending, at the top
- Reimbursed receipts grouped by the **month the money came in**, with monthly totals — handy for spotting an employer that's slow to pay.

### Reports → Google Doc — Report tab
1. Set a title, your name, and filters (trip, employer, dates). It defaults to **not-yet-reimbursed only**, which is usually what you want to submit.
2. Check the on-screen summary, then tap **Download Word doc (.doc)**. The file contains a summary by expense type, an itemized table (with reimbursement status), and every receipt image with a caption.
3. Get it into Drive either way:
   - **📁 Save to Drive as Google Doc…** — the app opens a folder browser (starting at your configured sync folder), you drill to any folder and tap "Save in this folder". The report is uploaded and converted to a **native Google Doc** on the spot — no download/upload round-trip. Requires the Drive sync setup (client ID) below.
   - **Download Word doc (.doc)** — downloads locally; upload it to Drive yourself and open it, and Drive converts it to a Google Doc.
4. When you get paid, come back with the same filters and tap **✓ Mark these receipts reimbursed today** to flip the whole batch at once.

There's also **Print / save as PDF** if PDF is preferred.

---

## 4. Free sync across devices (Google Drive)

Sync keeps one file (`expense-tracker-sync.json`) in your own Google Drive and merges it with each device — newest edit wins, deletions carry over. It's free forever because it uses your own Drive storage. One-time setup, about 5 minutes:

1. Go to **console.cloud.google.com** and sign in with your Google account.
2. Create a **new project** (name it anything, e.g. "Expense Tracker").
3. In the search bar, find **Google Drive API** and click **Enable**.
4. Go to **APIs & Services → OAuth consent screen** (in newer console versions this appears as **Google Auth Platform**). Choose User type **External**, give the app a name (e.g. "Expense Tracker"), and enter your email where required. Then pick ONE of these two paths:

   **Path A — add yourself as a test user (fastest):**
   - Go to the **Audience** page (or the "Test users" section of the consent screen).
   - Under **Test users**, click **+ Add users**, type your own Gmail address, and **Save**.
   - Done. The app stays in "Testing" status forever, which is fine — you're the only user. You may need to re-approve access occasionally.

   **Path B — publish the app:**
   - On the same page, find **Publishing status** and click **Publish app**, then confirm.
   - **Caveat:** this works review-free only while the app uses the minimal `drive.file` scope (no sync folder configured). If you configure a Drive folder (see step 6), the app requests the full `drive` scope, which Google classifies as **restricted** — a published app with that scope will keep showing the "unverified" warning and Google may nag about verification. **If you use the folder feature, prefer Path A.**

   Either way, the first time you sign in you may see a "Google hasn't verified this app" screen — click **Advanced → Go to [app name]**. It's your own app; the warning is just Google's default for personal OAuth apps.

5. Go to **APIs & Services → Credentials → Create credentials → OAuth client ID**:
   - Application type: **Web application**
   - Under **Authorized JavaScript origins**, add your app's origin(s):
     - `https://YOUR-USERNAME.github.io`
     - `http://localhost:8000` (if you run it locally)
   - Create, then copy the **Client ID** (ends in `.apps.googleusercontent.com`). No client secret is needed.
6. **Optional — pick a Drive folder for the sync file.** By default the sync file lives in your Drive root using the minimal `drive.file` permission (the app can only see files it created). If you want the file in a specific folder, paste the folder's link or ID into **Settings tab → Drive folder for sync file** (or set the `DEFAULT_GFOLDER` constant near the top of `index.html` so all devices target it — folder IDs are safe to commit). **Tradeoff:** targeting an existing folder requires the app to request full Drive access instead of the minimal scope, because the minimal scope can't see folders it didn't create. For a personal app on your own account that's fine — but it's why the folder is optional. If a sync file already exists elsewhere from earlier syncs, the app automatically moves it into the folder on the next sync.
7. Give the app the client ID — two ways:
   - **Per device:** Settings tab → paste → Save. You'd repeat this once on each device/browser, because settings are stored locally, not synced.
   - **Once for all devices (recommended):** open `index.html`, find the line near the top of the script that reads `const DEFAULT_GCLIENT = '';`, paste your client ID between the quotes, and push the change. Every device then has it automatically. This is safe to commit publicly — see the privacy section below.
8. Tap **⇅ Sync with Google Drive** and approve access on each device once. (If you add or change the folder later, the next sync will ask for consent again — that's the scope change, approve it once.)

**Auto-sync:** with the "Auto-sync when the app opens" box checked (it is by default), the app syncs quietly and continuously: when you open it, on every receipt save/edit/delete/reimbursement change, and about once a minute while the app stays open and visible (paused while the tab is hidden or offline, and it catches up immediately when you switch back or reconnect). Your sign-in is remembered across page reloads for as long as it stays valid (roughly an hour, refreshed automatically in the background), so you shouldn't need to re-authorize every time you open or refresh the app — only after it's been closed long enough for that session to expire. If sync can't succeed (offline, or the browser blocks the silent sign-in), a banner appears asking you to tap the sync button once; it clears itself as soon as sync succeeds again.

**What the app can access:** with no sync folder configured, it requests only the minimal `drive.file` permission — it can see and touch **only files it creates**. With a sync folder configured (or when using the report folder browser), it requests full Drive access so it can write into folders it didn't create; even then, the app itself only ever reads/writes its one sync file and the reports you explicitly save.

---

## 5. AI auto-fill (optional)

The **✨ Auto-fill** button sends the receipt image to an AI model, which extracts the vendor, date, amount, currency, and expense type. Two supported providers — you only need one:

**Option A — Gemini (free, recommended):**
1. Go to **aistudio.google.com**, sign in with your Google account, and click **Get API key** → Create API key. No credit card, no billing setup.
2. Paste it in **Settings tab** → Save.
3. The free tier covers Gemini Flash models with generous daily limits — far more than any realistic receipt volume. Genuinely $0.

**Option B — Claude (paid, pay-as-you-go):**
- Get an API key at **console.anthropic.com** (Settings → API keys). Costs roughly half a cent per receipt.
- If both keys are saved, the app uses Gemini.

**Note on subscriptions:** a Google AI Pro subscription or a Claude Pro subscription does **not** include API access — those cover the consumer chat apps only. API keys are separate products for both companies, which is why the free Gemini AI Studio key is the right fit here.

Either key is stored only on your device and sent only to that provider's API when you tap the button. **Never commit an API key to the repo** — both Gemini and Anthropic keys are real secrets (unlike the OAuth client ID).

---

## 6. Backups

**Backup data** (Settings tab) downloads a single JSON file containing every receipt, photo included. **Restore** merges a backup back in — it's smart about timestamps, so restoring an old backup won't clobber newer edits. Even with Drive sync on, it's worth downloading a backup once in a while (especially on iPhone, where Safari can evict site data from apps you haven't opened in months).

---

## 7. Data, privacy & the public repo

**Your repo can (and on a free GitHub plan, must) be public.** GitHub Pages only works on public repositories unless you have a paid GitHub plan. That's fine, because nothing private ever lives in the repo:

- **Your receipts and financial data are never in the repo.** They exist only in your browser's local database on each device and in the sync file in your own Google Drive. Pushing code to GitHub never touches them.
- **The OAuth client ID is safe to commit publicly.** Google designed web client IDs to be public — they're visible in the URL bar during every sign-in anyway. Security comes from the **Authorized JavaScript origins** list: Google will only issue tokens for your client ID to pages served from the origins you listed. Someone who copies your client ID onto their own site just gets an error. (For the same reason, don't add origins you don't control.)
- **API keys (Gemini or Anthropic) are real secrets — never commit them.** Only enter them in the app's settings, where they're stored locally on your device. Don't hardcode them into `index.html` or anything you push. If you ever leak one, revoke it (aistudio.google.com or console.anthropic.com) and make a new one.
- **Anyone can visit your public app URL** — but they'd see an empty app connected to *their* browser storage and *their* Google account, never your data.

Other privacy notes:
- Receipts are stored in your browser's IndexedDB on each device.
- Sync data goes only to a file in **your** Google Drive. (Scope depends on setup — see "What the app can access" in the sync section.)
- Auto-fill images go only to the AI provider you configured — Gemini or Anthropic — and only when you tap the button.
- There is no server, no analytics, no ads.

---

## 8. Troubleshooting

**"Persistent storage isn't available" banner** — you're viewing the file in a sandboxed preview. Open the app from its real URL (GitHub Pages or localhost).

**Sync fails with an origin/client error** — the URL you're using isn't listed under Authorized JavaScript origins in Google Cloud. Add it exactly (scheme + domain, no path, no trailing slash) and wait a few minutes.

**Google shows "unverified app" on first sign-in** — expected for a personal OAuth app. Advanced → continue, or add yourself as a test user.

**Auto-fill says it needs an API key** — add a free Gemini key from aistudio.google.com (or a paid Anthropic key) in the Settings tab.

**PDF conversion fails** — pdf.js loads from a CDN, so the first PDF needs an internet connection.

**Updated the code but the app looks old** — the service worker caches aggressively. Bump the `CACHE` name in `sw.js` (e.g. `expense-tracker-v2`) when you deploy changes, or in the browser: DevTools → Application → Service workers → Update.
