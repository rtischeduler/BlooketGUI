# BlooketGUI

`https://play.blooket.com` proxied through Singuloxy (libcurl + wisp), the same way `rtischeduler/DeepAI` does it, with a script injected into the page after it loads.

## index.html

A single page, no build step. Open it and it loads the real Blooket full screen.

- Singuloxy (`lotsacookie/Singuloxy`, `dist/sandstone.mjs`) is loaded from jsDelivr with a GitHub raw fallback.
- Wisp discovery, probing, bad-server tracking and "Switch proxy" are the same code as DeepAI (lists from `TongSherbet/storage` and `lotsacookie/kstuff`). Storage keys are prefixed `BK_`.
- Instead of DeepAI's hand-written fetch shim, the whole site is rendered by Singuloxy's own `ProxyFrame`, so every page, script, request and WebSocket under `play.blooket.com` goes through the proxy.

## Script injection

Near the top of the page script:

```js
const INJECT_URL = "https://gitlab.com/blooket/blooket-cheats/-/raw/c322469ebbd192b6731480af76b1ac0a3d4a7b22/cheats/gui.js";
```

It currently points at `cheats/gui.js` from `gitlab.com/blooket/blooket-cheats`, pinned to commit `c322469`. Replace it with any other script URL. After every page load the script is downloaded **through the proxy** (so CORS is not an issue) and run in the proxied page with `main_frame.eval_js(code)`. That is the same call behind the console run bar in Singuloxy's example frontend: it rewrites the code and does an indirect `eval` in the page's global scope.

- If `INJECT_URL` is set back to `https://example.com/PLACEHOLDER.js`, nothing is injected.
- The downloaded code is cached and re-run on each full page load. In-app (SPA) route changes keep the page alive, so it is not re-run for those.
- Failures show a toast; details are in the console.

Handy from the top-level console:

```js
__inject()        // refetch the script and run it again right now
main_frame.eval_js("document.title")   // run anything in the proxied page
__rotateProxy()   // switch wisp server
```

## Notes

- The script runs after the page has loaded, as if typed into a console, so it cannot hook anything that happens during page start-up.
- Proxied pages run in a sandboxed iframe with no access to the parent page.
- Not yet tested against live Blooket and live wisp servers.
