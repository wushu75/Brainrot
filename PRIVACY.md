# Privacy

Brainrot is built to be a tool you can trust without trusting us, because there
is no "us" in the loop. Here is exactly what it does and doesn't do.

## The short version

- **Everything stays on your device.** All data is stored locally in
  `chrome.storage.local`, `chrome.storage.session`, and IndexedDB.
- **Zero network requests.** Brainrot has no backend. It never phones home,
  because there is no home to phone. No analytics, no telemetry, no crash
  reporting, no ads.
- **No accounts.** You never sign in. There is nothing to sign into.

## What data Brainrot reads

- The **domain of your active tab** (e.g. `tiktok.com`, `github.com`).
- For YouTube only, whether the URL path is `/shorts/` (to tell Shorts apart
  from regular videos).

That's it. Brainrot does **not** read:

- Page contents, text, images, or the DOM (it injects no code into pages).
- Full URLs or query strings (only the registrable domain is stored; the
  `/shorts/` check is evaluated in memory and not saved).
- Your browsing history API, bookmarks, downloads, cookies, or form data.
- Anything from tabs that aren't the focused, active one.

## What data Brainrot stores (locally)

- **Per day, per domain: active seconds** — e.g. "2026-08-19, tiktok.com, 1840s".
  This is what powers the score, trend, and "top time sinks".
- **Your settings** — custom high / deep-work / ignore domain lists.
- **Recovery-plan checkboxes** for each day.
- **Install metadata** — a timestamp and a schema version.

Because per-domain time is stored, this local database effectively records how
long you spend on each site each day. It never leaves your machine, you can wipe
it anytime, and you can add any site to the **Ignore** list to stop tracking it.

## Permissions

Brainrot requests exactly three, and no host permissions:

- `storage` — to save the local data above.
- `tabs` — to read the active tab's URL for classification.
- `alarms` — to wake the service worker on a timer so time tracking is accurate.

It does **not** request `scripting`, host permissions, `history`, `webNavigation`,
`idle`, `cookies`, or anything else.

## Your controls

- **Ignore a site:** add it to the Ignore list in Settings — it won't be tracked.
- **Erase everything:** Settings → *Erase all data*. This permanently deletes all
  tracked time, your custom lists, and recovery progress. It cannot be undone.
- **Uninstall:** removing the extension deletes its local storage per Chrome's
  normal behavior.

## No remote code

All logic runs from the files bundled in the extension. There is no `eval`, no
remotely hosted script, and no dynamic code loading. The source is MIT-licensed
and auditable end to end.

## Questions

Because Brainrot collects nothing centrally, we can't look anything up for you —
which is the design. Read the source; it's short.
