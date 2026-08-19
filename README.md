# Brainrot

**Measure your Brainrot Score (0–100) from your real browsing. Then reverse it.**
100% local. No accounts. No servers. No page contents. Ever.

Brainrot is a Manifest V3 Chrome extension that tracks how much of your active
attention goes to infinite-feed / short-form "brainrot" sites versus deep-work
sites, turns it into a single blunt number, and hands you a small daily plan to
claw your focus back. Everything is computed and stored on your machine.

---

## Install (unpacked, dev)

1. `git clone` this repo (or download the folder).
2. Go to `chrome://extensions`.
3. Toggle **Developer mode** (top right).
4. Click **Load unpacked** and select the `brainrot/` folder.
5. Pin the extension and click the icon.

No build step. No `npm install`. The code you load is the code that runs —
which is the point for a privacy tool: it's trivially auditable.

> Requires Chrome 116+ (for `chrome.storage.session` in MV3).

---

## What it does

- **Tracks active-tab time** per domain (only the tab you're actually looking
  at, only while the browser is focused).
- **Classifies** each domain as `high` brainrot, `low` (deep work), or
  `neutral` (ignored) using hardcoded lists plus your own overrides.
- **Scores** your day 0–100 (higher = more brainrot) and shows a 7-day trend.
- **Estimates "Attention Residual"** — a heuristic for whether you're building
  or eroding deep-focus capacity.
- **Generates a daily recovery plan** personalized to your worst time sinks.
- **Exports a shareable report card** (PNG, drawn on a canvas).

---

## Permissions — and why each one is the minimum

| Permission | Why it's needed | What it is **not** used for |
|---|---|---|
| `storage`  | Persist time totals, settings, and daily recovery state via `chrome.storage.local`, `chrome.storage.session`, and IndexedDB. | Nothing leaves the device. No sync backend. |
| `tabs`     | Read the **active tab's URL** to classify the current domain and attribute active time. We use hostname + (for YouTube) whether the path is `/shorts/`. | We never read page contents, history, titles for storage, or other tabs' data. |
| `alarms`   | MV3 service workers sleep. A 1-minute alarm wakes the worker to checkpoint active-tab time so tracking stays accurate without a persistent background page. | Not used for notifications or scheduling anything network-related. |

**Deliberately not requested:** no host permissions, no `scripting`, no content
scripts, no `webNavigation`, no `history`, no `idle`. We never inject code into
pages. `tabs` alone gives us the URL, which is all we need.

---

## Architecture

```
brainrot/
├── manifest.json                 # MV3, 3 permissions, module service worker
├── src/
│   ├── background/
│   │   └── service-worker.js      # active-tab time tracking (capped ticks)
│   ├── lib/
│   │   ├── constants.js           # domain lists + all tunable weights
│   │   ├── classifier.js          # URL -> {domain, category}
│   │   ├── db.js                  # IndexedDB (per-day per-domain seconds)
│   │   ├── storage.js             # chrome.storage wrappers + date utils
│   │   ├── scoring.js             # score, residual, trend, top sinks, diagnosis
│   │   └── recovery.js            # personalized daily plan
│   ├── popup/                     # main UI (score, trend, plan, share)
│   ├── options/                   # custom lists, reset, privacy
│   └── report/
│       └── report-card.js         # canvas report card + copy/download
├── assets/icons/                  # placeholder icons (regenerate via tools/)
├── tools/
│   ├── make_icons.py              # regenerate icons
│   └── test.mjs                   # smoke tests for the pure logic
├── PRIVACY.md
├── LICENSE                        # MIT
└── jsconfig.json                  # editor type-checking via JSDoc
```

### How tracking survives a sleeping service worker

MV3 kills the background worker when idle, so no state is kept in memory. The
"what am I looking at right now" pointer lives in `chrome.storage.session`
(in-memory, cleared on browser close, survives worker restarts). Every tracking
decision is a read-modify-write:

- On tab switch / URL change / window focus change → flush the old session's
  elapsed time, then point at the new domain.
- A 1-minute alarm flushes time **and self-heals**: if the focused active tab
  drifted from our pointer (an event we slept through), we re-align.
- Each flush ("tick") adds `min(now − lastTick, 90s)`. The cap means a long
  sleep or a closed laptop lid can't inflate your time — worst case we slightly
  **under**count, which is the honest direction.

---

## The Brainrot Score

Higher = more brainrot. `0` = pristine focus, `100` = terminal. **Neutral time
(email, news, random browsing) is intentionally ignored** — the score is a ratio
of your *intentional* attention, not total screen time.

Per day, with `H` = high-brainrot seconds and `L` = deep-work seconds:

```
ratio  = H / (H + L + 1)                          // how dominant feeds are
volume = 1 - exp(-Hmin / 120)                      // absolute dose, saturating
raw    = 100 * (0.6*ratio + 0.4*volume)
rebate = 25 * (1 - exp(-Lmin / 90))                // deep work earns a discount
score  = clamp(raw - rebate, 0, 100)
```

- `ratio` punishes letting feeds dominate even when total time is low.
- `volume` makes 4h of TikTok strictly worse than 20m, with diminishing returns.
- `rebate` lets a real deep-work session pull a bad day back toward green
  (capped, so you can't grind off unlimited scroll).

Every constant lives in `src/lib/constants.js` (`WEIGHTS`) so it's easy to tune.

**Attention Residual** (7-day, signed "focus-minutes") is an interpretable
heuristic — `deepWork * 0.5 − brainrot * 0.4` per day, capped and summed. It is
**not** a clinical measurement and is labeled as such in the UI.

---

## Customize

Open **Settings** (gear icon in the popup) to:

- Add domains to your **high**, **deep-work**, or **ignore** lists (overrides beat
  defaults; changes apply to your current tab immediately).
- See the built-in default lists.
- Read the privacy summary.
- **Erase all data** (irreversible).

---

## Develop

```bash
node tools/test.mjs        # smoke-test the pure logic (no browser needed)
python3 tools/make_icons.py # regenerate placeholder icons
```

The code is modern ES-module JavaScript with thorough JSDoc types and a
`jsconfig.json` so editors type-check it without a build. It's structured to
convert to TypeScript cleanly if you'd prefer that — see the note in
`jsconfig.json`.

## Roadmap ideas (not in v0.1)

- Optional `idle` detection to pause tracking when you're AFK.
- Per-day snapshots for a longer historical view.
- Focus-session timer, streaks, and gentle interventions on high-brainrot sites.

## License

MIT — see [LICENSE](./LICENSE).
