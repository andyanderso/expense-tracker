# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file PWA expense tracker. The entire app — CSS, HTML, and one IIFE `<script>` — lives in `index.html`. There is **no build step, no dependencies, no test suite, no lint config**. Deployment is pushing to `main`; GitHub Pages serves the repo root as-is.

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

All app code is one strict-mode IIFE in `index.html`, organized in commented sections (`/* ============ name ============ */`): constants & state, storage, image handling, add-form/AI extraction, list rendering, history, report generation, backup/restore, settings, Google Drive sync, boot.

**Data model & merge.** `receipts` is an in-memory array mirroring the IndexedDB `expense-tracker` DB (object stores: `receipts`, `meta`). Deletes are **soft** — a `deleted` tombstone flag; `live()` filters them out. Never hard-delete: tombstones are how deletions propagate through sync. `mergeIn()` is the single merge primitive (newest `updatedAt`/`createdAt` wins) used by both Drive sync and backup restore — any new receipt mutation must set `updatedAt` or sync will silently discard it.

**Multi-currency (core feature — don't break).** Each receipt has `amount` + `currency`, plus an optional `billedAmount` (the real USD card charge for foreign receipts). `totalsByCurrency()` uses `billedAmount` as USD when present, falling back to the receipt currency otherwise — this precedence must hold everywhere totals appear (header, groups, reports). Totals are kept per-currency and never converted.

**Google Drive sync (core feature — don't break).** One file (`expense-tracker-sync.json`) in the user's Drive, merged both ways via `mergeIn()`. Scope escalates: minimal `drive.file` normally, full `drive` scope only when a sync folder is configured (`driveScope()`). The `DEFAULT_GCLIENT` and `DEFAULT_GFOLDER` constants near the top of the script are **safe to commit** (OAuth client IDs and folder IDs are public by design); Gemini/Anthropic **API keys are real secrets** and live only in per-device settings — never hardcode them.

**Settings** (`settings` object in the `meta` store) are per-device and deliberately **not synced** — that's why the `DEFAULT_*` constants exist.

**AI auto-fill.** `extractWithGemini()` / `extractWithClaude()`; Gemini is preferred when both keys are set.

**External libraries** (pdf.js, Google Identity Services) load lazily from CDNs via `loadScript()` — nothing is vendored. The service worker only caches same-origin GETs and must never intercept cross-origin API calls.

The README's user-facing behavior descriptions (reconciliation display, report format, sync semantics) are the spec — keep them accurate when changing behavior.
