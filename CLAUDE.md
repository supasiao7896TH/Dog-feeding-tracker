# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page PWA ("Dog Feeding Tracker" / ตารางติดตามการให้อาหารสุนัข) that shows a Thai-language monthly calendar marking alternate-day dog feeding ("ให้" 🟢) vs. rest days ("หยุด" 🔴), computed from a user-chosen base date. No backend — all state lives in `localStorage` on the device.

## Structure

There is no build step, no package manager, and no framework. The entire app is three static files served as-is:

- `index.html` — everything: markup, inline `<style>`, and the inline `<script>` app logic (state, calendar rendering, event listeners). This is the only file you'll usually need to edit.
- `sw.js` — service worker; cache-first for same-origin GET requests, cache-busts via the `CACHE` constant name (currently `dog-feeding-v3`).
- `manifest.json` — PWA manifest (Thai `lang`, standalone display, icons).
- `icons/` — PWA icon assets (svg/png).

## Running locally

No build/install/test commands exist. Serve the directory statically and open it in a browser, e.g.:

```
npx serve .
# or
python -m http.server
```

A service worker is registered, so during development use a hard refresh / unregister the SW (or an incognito window) to avoid stale cached assets.

## Key things to know when editing

- **Bump the SW cache name on every asset change.** `sw.js`'s `CACHE` constant (e.g. `dog-feeding-v3`) must be incremented whenever `index.html`, `manifest.json`, or icon files change, or returning users will keep getting the stale cached version. If you add/rename a cached asset, update the `ASSETS` array too.
- **CSP is locked down** via the `<meta http-equiv="Content-Security-Policy">` tag in `index.html`. Only `https://cdn.jsdelivr.net` (Tailwind CSS) and Google Fonts domains are allowlisted for scripts/styles/fonts. Any new external dependency requires updating this CSP.
- **Core scheduling algorithm** is `getDayStatus()` in `index.html`: it diffs a target date against `state.baseDate` in whole UTC days and uses even/odd parity to decide feed vs. skip. Date math intentionally zeroes out time via `Date.UTC(...)` to avoid DST/timezone drift — preserve that pattern if you touch date logic.
- **Calendar dates are displayed as Thai Buddhist Era** (`year + 543`) via the `thaiMonths` array — don't swap in Gregorian year display.
- **Calendar cells have known overflow footguns**: prior commits fixed the "หยุด" label and cell content overflowing the grid column/row. Keep cell text short and test narrow viewports (e.g. `min-w-0`, `overflow-hidden`) when touching calendar cell markup/classes.
- **No native `alert`/`confirm`/`prompt`** — user-facing messages go through the custom `showToast(message, icon)` toast, not browser dialogs.
- **Data portability**: export/import is a hand-rolled JSON blob (`{version, baseDate}`) via `localStorage` key `dog_feeding_base_date`, written as a `data:` URI download and read back with `FileReader`. Keep the export/import shape in sync if you add more persisted state.
- **UI text and comments are in Thai**; match that convention for user-facing strings and keep existing comment style (Thai description of *why*/*what section*) when editing nearby code.
