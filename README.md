# opt-gemini-web

Local optimization layer for `gemini.google.com` (and related Google web
properties) on this machine. Goal: reduce load time, main-thread jank, and
resource usage (CPU/GPU/RAM) without modifying Google's servers.

## How it works

All optimization is done client-side via **Chrome DevTools Local Overrides**.
The browser is pointed at a local folder; matching network responses are
served from files in that folder instead of the network.

- `gemini-overrides/` — the folder to select in DevTools
  (Sources → Overrides → Select folder). It mirrors Google's URL layout:
  `<host>/longurls/<hashed-name>` for very long boq chunk URLs.

### Setup (one-time, in the browser)

1. Open `gemini.google.com/app` in Chromium.
2. DevTools (F12) → Network tab → right-click any request →
   **Override content** → select `gemini-overrides/` → Allow.
3. Reload. Overridden requests show a purple badge in the Network panel.

## Current optimizations

- **Duplicate chunk elision**: Google's boq loader fetches ~25 JS modules
  twice under different `exm=` URLs; 24 pairs return byte-identical content.
  The second URL of each pair is overridden with a stub comment file,
  saving ~1.6 MB of transfer (~25% of all JS) per load.

## Safety rules

- Overrides are **fail-safe by design**: Google rotates `rs=` tokens on every
  deploy, so stubs stop matching automatically and the page loads normally.
- Never override a URL whose content differs between duplicate fetches
  (only byte-identical pairs are stubbed).
- Stub files must remain valid empty JS (comment-only is fine).
- Do not commit secrets, cookies, or session data into this folder.
- Verify the page still works after any change; keep a way to disable
  overrides instantly (untick "Enable Local Overrides" or empty the folder).

## Analysis artifacts

- `gemini-load-trace.json` — Chrome DevTools performance trace of a page
  load (Perfetto format, ~160 MB). Use `chrome://tracing` or
  https://ui.perfetto.dev to inspect.
