# Limits and troubleshooting

## Contents
- Error messages and what to do
- Things that cannot be done through this skill
- How the background runtime behaves

## Error messages and what to do

| Message (or symptom) | Cause | What to do |
|---|---|---|
| `No tab selected` | No current tab saved | `open URL` or `use TAB_ID` (list with `tabs`), or pass `--tab` |
| `Browser is not available: N` | Browser ids are reassigned each runtime start; the saved id is stale | `browsers`, then `use-browser ext:XXXXXXXX` (stable id from the `use` field) |
| `No connected browser with extension instance ...` | That Chrome profile is closed or the ChatGPT extension is off/disconnected | Ask the user to open that profile with the extension enabled; `browsers` shows what is connected |
| `Tab not found in browser N` | Tab was closed, or the id belongs to another browser | `tabs` to find it again; check `--browser` |
| `Tab ... is already part of browser session ...` | The tab is controlled by another runtime (Codex app, an old process, another client) | Open a new tab, or use a tab nobody has claimed |
| `Timed out ... Page.navigate`, `Debugger unattached` | Page slow/blocked, tab in background, or a single-threaded local dev server holding the connection | Retry once; check the page with `read`; serve local test pages with a threaded server |
| `Timed out ... Page.captureScreenshot` | Background tab | `shot` retries 3 times; if it still fails, `wait --load load` and retry |
| `trusted Node process exited ... kernel reset` / `the browser runtime crashed and was restarted` | The runtime crashed; the daemon restarts it, but tabs opened before are gone | Re-run `open` (or `use` a tab that still exists) |
| `CDP approval gate did not grant access` | Raw DevTools Protocol needs the Codex app's approval | Do not work around it; use higher-level commands, or ask the user to run it from Codex |
| `To enable file upload ... Allow access to file URLs` | Chrome extension setting is off for this profile | Tell the user to open `chrome://extensions`, Details on the ChatGPT extension, enable "Allow access to file URLs" |
| `EPERM ... mkdir ...\Temp\browser-use` (assets bundle) | Runtime file-system sandbox | Use `assets list` and download URLs yourself, or `download --media` |
| `eval` returns `null` / cannot see a variable | `eval` runs in an isolated read-only DOM scope without `window.*` of the page | Read state from the DOM (text, attributes, values), not from JS variables |
| Output ends with `[truncated ...]` | Output cap (60,000 chars) | Narrow the query, or set `CUA_MAX_OUTPUT_CHARS` and redirect to a file |
| `Required documentation has not been read` | A capability needs its docs read in the session | The CLI does this for `cdp`; for other capabilities call `mgmt`/`js` after `js` reading `await (await tab.capabilities.get("<id>")).documentation()` |

## Things that cannot be done through this skill

- Raw CDP without Codex approval; reading `window.*` page variables; arbitrary page-world JavaScript.
- Freehand drawing: `drag` moves in a straight line between two points (draw curves as many short segments).
- Hover (no API), file uploads on profiles without the "file URLs" permission, bundling page assets under the sandbox.
- Mobile viewports narrower than ~416px (Chrome's window minimum): `viewport --preset mobile` produces ~416px.
- Captchas, paywalls, browser safety interstitials, permission prompts, password/OTP entry: hand these back to the user.
- Meta Ad Library has no spend/impression numbers for commercial ads.
- Google Ads Transparency Center: search matches advertiser names and domains only (not ad text); `google-ads` types into the box and opens the suggestions. Video creatives play inside Google ad frames, so only preview images, format, last shown date, variants and countries are collected.
- TikTok Ad Library (library.tiktok.com) covers ads shown in the EU/EEA (plus some others); results for a keyword can be dominated by one advertiser. Detail pages give targeting, payer, landing page; videos are not downloaded.
- TikTok Creative Center (trends, Top Ads): when not logged in to TikTok for Business only a few rows show (`login_wall: true`). The keyword filter of Top Ads may be ignored when logged out.
- tiktok.com search needs the profile to be logged in to TikTok; results are personalised. Video pages show likes/comments/shares/saves but not views.
- Google Trends Explore usually returns an empty chart to an automated session (bot check / rate limit). `google-trends` therefore relies on Google/YouTube autocomplete and the daily trending RSS; `--explore` reports `chart_has_line: false` when it got nothing.
- Facebook video search shows posts and captions but no view or reaction counts.

## How the background runtime behaves

- The first command starts one background `cua.py _daemon` process (it launches the runtime). Later commands connect to it over `127.0.0.1` with a random token stored in `~/.cua-chrome/daemon.json`; they take ~0.2s instead of seconds.
- It exits after 20 idle minutes (`CUA_IDLE_MINUTES` changes this) or on `cua.py stop`. Tabs it opened are released then.
- `--no-daemon` runs a one-shot runtime: simpler, but tabs it opened are closed when the command exits.
- The daemon runs without a console window (a black `python.exe` window means an old version: `cua.py stop` and update).
- Commands are serialised (one at a time) because the runtime is stateful.
- Tabs opened with `open` are tracked in `state.json` (`opened`). `cleanup` closes them; `stop` cleans first (`--keep-tabs` to skip). When the daemon exits on its own (idle timeout or crash) the runtime releases and closes the tabs it created.
- State lives in `~/.cua-chrome/` (`state.json`: default browser, current tab, session id; `shots/`: screenshots without `--out`). `CUA_STATE_DIR` relocates it.
- After the Codex plugin updates, run `cua.py stop` so the next command picks up the newest runtime.
