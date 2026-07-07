# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file PWA expense tracker. The entire app — CSS, HTML, and one IIFE `<script>` — lives in `index.html`. There is **no build step, no dependencies, no test suite, no lint config**. Deployment is pushing to `main`; GitHub Pages serves the repo root as-is.

- Repo: `andyanderso/expense-tracker` (public — required for free GitHub Pages)
- Live: `https://andyanderso.github.io/expense-tracker/`
- Other files: `sw.js` (service worker), `manifest.json`, `icon-192.png`/`icon-512.png`, `README.md`
- User data (receipts, images, keys) lives in browser IndexedDB and the user's Google Drive — **never in this repo**

## Commands

Run locally:

```
python3 -m http.server 8000
```

Syntax-check the inline script — **required before every commit** (a syntax error takes down the whole app):

```
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/app-check.js && node --check /tmp/app-check.js && node --check sw.js
```

## Deploy rule (critical)

Every deploy **must bump the `CACHE` version constant in `sw.js`** (e.g. `expense-tracker-v7` → `v8`). The service worker caches the app shell aggressively; without the bump, users keep the old version indefinitely.

## Architecture

All app code is one strict-mode IIFE in `index.html`, organized in commented sections (`/* ============ name ============ */`): constants & state, storage, image handling, add-form/AI extraction, list rendering, bulk select, history, report generation, folder picker, backup/restore, settings, Google Drive sync, boot.

**Data model & merge.** `receipts` is an in-memory array mirroring the IndexedDB `expense-tracker` DB, version 2 (object stores: `receipts` keyPath `id`, `meta` keyPath `k`). Receipt fields:

```js
{ id, date, amount, currency,            // receipt face value; currencies incl. USD/CAD/EUR/CLP/ARS/NZD/INR
  billedAmount, billedImage,             // optional USD card charge + statement screenshot (reconciliation)
  vendor, trip, employer, type, notes,   // type ∈ TYPES; trip/employer autocomplete via <datalist>
  image,                                 // dataURL JPEG, ≤1600px, EXIF-rotated, auto-cropped
  reimbursed, reimbursedDate,
  deleted,                               // tombstone
  createdAt, updatedAt }                 // updatedAt drives sync merge
```

Deletes are **soft** — a `deleted` tombstone flag (images stripped to save space); `live()` filters them out. Never hard-delete: tombstones are how deletions propagate through sync. `mergeIn()` is the single merge primitive (newest `updatedAt`/`createdAt` wins) used by both Drive sync and backup restore — **any new receipt mutation must set `updatedAt`** or sync will silently discard it.

**Multi-currency (core feature — don't break).** Each receipt has `amount` + `currency`, plus an optional `billedAmount` (the real USD card charge for foreign receipts). `totalsByCurrency()` uses `billedAmount` as USD when present, falling back to the receipt currency otherwise — this precedence must hold everywhere totals appear (header, groups, reports). Totals are kept per-currency and never converted.

**Google Drive sync (core feature — don't break).** One file (`expense-tracker-sync.json`) in the user's Drive, merged both ways via `mergeIn()`; syncs on open, after every receipt save/edit/delete/reimbursement change (debounced via `scheduleAutoSync()`), and on a ~60s interval while the tab is visible and online (`startSyncInterval()`/`stopSyncInterval()`, driven by `visibilitychange`/`online`/`offline` listeners), with a "tap sync" toast fallback when silent auth fails. Scope escalates: minimal `drive.file` normally, full `drive` scope when a sync folder is configured or the report folder browser is used (`driveScope()`); tokens are cached per scope (~1h) in memory *and* persisted to the `meta` store (key `'gtoken'`) so a page reload doesn't force re-authorization — rehydrated only if unexpired and the scope still matches. A `401` from Drive clears the cached/persisted token so the next sync gets a fresh one. `syncInFlight` guards against overlapping sync attempts; `lastSyncOk`/`lastSyncAt`/`lastSyncSuccessAt` drive the `#syncBanner` staleness warning (shown when a sync fails or none has succeeded in 5 minutes while online). Stray sync files in Drive root get moved into the configured folder. The `DEFAULT_GCLIENT` and `DEFAULT_GFOLDER` constants near the top of the script are **safe to commit** (OAuth client IDs and folder IDs are public by design); Gemini/Anthropic **API keys are real secrets** and live only in per-device settings — never hardcode them.

**Settings** (`settings` object in the `meta` store) are per-device and deliberately **not synced** — that's why the `DEFAULT_*` constants exist.

**AI auto-fill.** `extractWithGemini()` (model `gemini-2.5-flash`, free-tier key) / `extractWithClaude()` (`claude-sonnet-4-6`, browser-direct header); Gemini is preferred when both keys are set. Both share `EXTRACT_PROMPT` demanding raw JSON; parse defensively (strip fences, slice `{…}`).

**Images.** `loadBitmap()` uses `createImageBitmap({imageOrientation:'from-image'})` so EXIF rotation is always applied; `downscale(file, doCrop)`; `autoCrop()` is deliberately conservative (bails unless the content box is ≥25% per dimension) — keep it that way, over-cropping a receipt is worse than not cropping. Card-statement screenshots always use `doCrop=false`. PDFs render via pdf.js (up to 3 pages stitched into one JPEG).

**Reports.** `reportHTML()` builds Word-flavored HTML (summary by type, itemized table — `<colgroup>`-driven column widths, "Card charge (USD)" column omitted when no receipt in the set has a `billedAmount` — then each receipt image page-broken with its statement screenshot beneath, images capped to `max-width:300px;max-height:480pt` with `page-break-inside:avoid` so one receipt doesn't spill across pages). All three export paths share it: `docBtn`/`printBtn` (local download / print) wrap it as legacy Word-flavored HTML (`application/msword` blob); "Save to Drive" (`pickSaveBtn`) uploads it as genuine `text/html` with metadata `mimeType: 'application/vnd.google-apps.document'` so Drive converts it to a native Google Doc. **Known unresolved bug:** Drive's conversion silently drops the inline `data:` URI receipt images regardless of upload encoding tried so far (`application/msword`-labeled HTML, genuine `text/html`, and a genuine client-side-generated `.docx` via the `docx` library all failed — the `.docx` attempt fixed images but broke table column widths in Google's docx importer in a way that survived several encoding fixes, DXA widths included, so it was reverted). Local download/print always show images fine; only the Drive-Doc conversion loses them. Before trying another fix, re-read the conversation history in git log for this file — several approaches have already been tried and ruled out.

**External libraries** (pdf.js 3.11.174, Google Identity Services) load lazily from CDNs via `loadScript()` — nothing is vendored. The service worker only caches same-origin GETs and must never intercept cross-origin API calls.

## UI rules

- Five tabs in a **sticky top bar**: ＋ Add (canary CTA pill), Receipts, History, Report, ⚙ Settings. Underline = active.
- **Nothing may exceed viewport width — this was a shipped bug.** Text lines inside receipt cards must be block-level elements with `overflow:hidden;text-overflow:ellipsis` (inline spans can't truncate and once pushed the page wider than the phone screen). `html,body` keep `overflow-x:hidden`; global `img{max-width:100%}`.
- **Never use localStorage/sessionStorage** — IndexedDB only.
- Design system ("carbon-copy receipt slip"): ink `#22303A`, paper `#EFF1EE`, canary `#F2C744`, pine `#3E6B4F`, rust `#B4552D`; Barlow Condensed (display/labels), IBM Plex Mono (numbers/dates), Inter (body); cards have a zigzag clip-path tear edge. Reuse the CSS variables; don't introduce new colors casually.

The README's user-facing behavior descriptions (reconciliation display, report format, sync semantics, Google Cloud setup steps) are the spec — keep them accurate when changing behavior, in the same commit.

## Owner context

Andy: PHP/JS/TS developer; Pop!_OS; Zed with `claude` CLI in the terminal; git over SSH. Tracks expenses across multiple employers (Sierra Avalanche Center, WMS/DiMM courses, Broken Arrow Skyrace, Siesta Solutions clients) and international trips (BC/Canada, Chile/Argentina, NZ, India). Primary device: Pixel 9 Pro, Chrome.

## Backlog (discussed, not built)

Mileage entries at IRS rate; CSV export; report auto-upload without the folder picker.
