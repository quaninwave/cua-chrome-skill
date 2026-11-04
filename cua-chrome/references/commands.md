# cua.py command reference

Global flags (before or after the command): `--browser ID|NAME|ext:PREFIX`, `--tab TAB_ID`, `--task NAME` (separate state for parallel jobs), `--no-daemon`.
Locator flags (`--selector/-s --role --name --text --label --placeholder --test-id --frame --nth --has-text --exact --regex`) are accepted by commands marked *locator*.
Most action commands also take `--timeout MS`; click/type/key/check/select-option/scroll/drag/nav take `--observe` to return the fresh accessibility tree.
`open` and every playbook record the tabs they create; `cleanup` (or `stop`) closes them and deletes scratch files. Commands that need the user's go-ahead take `--confirmed`.

## Setup and session

### `doctor`
Check that the plugin runtime, proxy and browser are reachable.

```
cua.py doctor [-h]
```

### `state`
Show saved defaults (browser, tab, session id).

```
cua.py state [-h]
```

### `browsers`
List browsers (each Chrome profile with the extension is its own id) with open tabs.

```
cua.py browsers [-h]
```

### `use-browser`
Set the default browser (id, name, or ext:<instance prefix>); ids are converted to the stable ext: form.

```
cua.py use-browser [-h] browser
```

### `start`
Start the background runtime now (also happens automatically on first use).

```
cua.py start [-h]
```

### `stop`
Close the tabs this tool opened, delete its scratch files, then stop the background runtime (--keep-tabs / --keep-files to skip).

```
cua.py stop [-h] [--keep-tabs] [--keep-files]
```

### `cleanup`
Task finished: close every tab this tool opened AND delete its scratch files (screenshots, videos, frames); --logs also drops the log.

```
cua.py cleanup [-h] [--all-tasks] [--keep-files] [--logs]
```

### `artifacts`
Files this tool created: list | keep PATH (protect from cleanup) | drop PATH (delete now) | gc (delete old scratch and trim the log).

```
cua.py artifacts [-h] [--hours HOURS] [{list,keep,drop,gc}] [path]
```

### `session-name`
Name the browser automation session (short, starts with an emoji).

```
cua.py session-name [-h] name
```

### `mark`
Keep the current tab alive: deliverable (live page is the output) | handoff (work continues later).

```
cua.py mark [-h] {deliverable,handoff}
```

### `note`
Per-site memory: get | add | list | clear notes (shown automatically on open/nav to that site).

```
cua.py note [-h] {get,add,list,clear} [site] [text]
```

## Tabs and navigation

### `tabs`
List tabs (scope all = every user tab; agent = only this session's).

```
cua.py tabs [-h] [--scope {all,agent}] [--query QUERY] [--limit LIMIT]
```

### `open`
Open a new tab (becomes the current tab).

```
cua.py open [-h] [--session SESSION] [--keep] [url]
```

### `use`
Claim an existing tab and make it current.

```
cua.py use [-h] tab_id
```

### `close`
Close the current tab (or --tab); tabs you did not open need --confirmed.

```
cua.py close [-h] [--confirmed]
```

### `nav`
Navigate: goto URL | back | forward | reload.

```
cua.py nav [-h] [--timeout TIMEOUT] [--observe] {goto,back,forward,reload} [url]
```

## Reading and extracting

### `look`
One-call page overview: url/title, headings, main text, inputs and buttons with ready-to-paste locator flags, login/captcha hints.

```
cua.py look [-h] [--chars CHARS] [--links LINKS] [--shot SHOT] [--timeout TIMEOUT]
```

### `ax`
Accessibility tree with element indexes (diff vs previous unless --full).

```
cua.py ax [-h] [--full]
```

### `dom`
DOM snapshot (includes iframe bodies).

```
cua.py dom [-h]
```

### `read` *(locator)*
Read text/attribute/html/value (whole body if no locator).

```
cua.py read [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--what {inner_text,text,attribute,html,value}] [--attr ATTR] [--all] [--limit LIMIT] [--timeout TIMEOUT]
```

### `find` *(locator)*
Describe matching elements (tag, text, href, visibility, bounding box).

```
cua.py find [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--limit LIMIT]
```

### `eval` *(locator)*
Evaluate READ-ONLY JS in the page's isolated DOM scope (cannot see window.* of the page).

```
cua.py eval [-h] [--arg ARG] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] expression
```

### `links`
List links (text + absolute href); filter by regex, same-site, container.

```
cua.py links [-h] [--filter FILTER] [--same-site] [--within WITHIN] [--limit LIMIT] [--timeout TIMEOUT]
```

### `table`
Extract an HTML table as rows (or --csv file).

```
cua.py table [-h] [--selector SELECTOR] [--nth NTH] [--csv CSV] [--limit LIMIT] [--timeout TIMEOUT]
```

### `items`
Extract repeated cards/rows into JSON/CSV with named fields; optional pagination.

```
cua.py items [-h] --container CONTAINER --field FIELD [--limit LIMIT] [--next NEXT] [--pages PAGES] [--csv CSV] [--timeout TIMEOUT]
```

### `scroll-load`
Scroll to the bottom repeatedly until no more items load (infinite scroll).

```
cua.py scroll-load [-h] [--item ITEM] [--rounds ROUNDS] [--wait WAIT]
```

### `snapshot`
Save page text under a name, later `diff` it to see what changed (prices, stock, new items).

```
cua.py snapshot [-h] [--selector SELECTOR] [--update] {save,diff,list} [name]
```

## Acting on pages

### `click` *(locator)*
Click / double-click by locator, ax index, or x,y (risky-looking controls need --confirmed).

```
cua.py click [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--index INDEX] [--xy XY] [--button {left,right,middle}] [--double] [--mod {Alt,Control,ControlOrMeta,Meta,Shift}] [--force] [--confirmed] [--timeout TIMEOUT] [--observe]
```

### `type` *(locator)*
Enter text (fill/type/sequential/native/set-value/paste).

```
cua.py type [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--index INDEX] [--mode {fill,type,sequential,native,set-value,paste}] [--enter] [--timeout TIMEOUT] [--observe] TEXT
```

### `fill-form`
Fill several fields at once by label/placeholder/name/id (never submits).

```
cua.py fill-form [-h] --data DATA [--frame FRAME] [--confirmed] [--timeout TIMEOUT]
```

### `key` *(locator)*
Press a key / shortcut.

```
cua.py key [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--index INDEX] [--timeout TIMEOUT] [--observe] key
```

### `check` *(locator)*
Set a checkbox/switch/radio (checked by default).

```
cua.py check [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--off] [--force] [--timeout TIMEOUT] [--observe]
```

### `select-option` *(locator)*
Pick option(s) in a native <select>.

```
cua.py select-option [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] --option OPTION [--timeout TIMEOUT] [--observe]
```

### `scroll`
Scroll by pages.

```
cua.py scroll [-h] [--pages PAGES] [--index INDEX] [--xy XY] [--timeout TIMEOUT] [--observe] {up,down,left,right}
```

### `drag`
Drag from (x1,y1) to (x2,y2) in viewport coordinates (straight line).

```
cua.py drag [-h] [--timeout TIMEOUT] [--observe] x1 y1 x2 y2
```

### `wait` *(locator)*
Wait for --ms, a locator state, --url, or --load (use sparingly).

```
cua.py wait [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--ms MS] [--state {attached,detached,visible,hidden}] [--url URL] [--load {load,domcontentloaded,networkidle}] [--timeout TIMEOUT]
```

## Screenshots, files, clipboard, dialogs

### `shot` *(locator)*
Screenshot (JPEG) of viewport, --full page, --clip, or one element; prints the saved path.

```
cua.py shot [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--out OUT] [--full] [--clip CLIP] [--timeout TIMEOUT]
```

### `upload` *(locator)*
Upload local files through the page's file chooser (sends file contents to the site).

```
cua.py upload [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--wait WAIT] [--confirmed] [--timeout TIMEOUT] files [files ...]
```

### `download` *(locator)*
Download a file (click a download control, or --media) and print its local path.

```
cua.py download [-h] [--selector SELECTOR] [--role ROLE] [--name NAME] [--text TEXT] [--label LABEL] [--placeholder PLACEHOLDER] [--test-id TEST_ID] [--frame FRAME] [--nth NTH] [--has-text HAS_TEXT] [--exact] [--regex] [--media] [--wait WAIT] [--copy-to COPY_TO] [--scratch] [--timeout TIMEOUT]
```

### `clipboard`
Read or write the browser clipboard (write overwrites what the user copied).

```
cua.py clipboard [-h] {read,write} [text]
```

### `dialog`
Inspect or answer a JS alert/confirm/prompt dialog.

```
cua.py dialog [-h] [--text TEXT] {get,accept,dismiss}
```

## Tab groups, windows, bookmarks (management)

### `tabs-query`
Chrome.tabs.query across windows (id, windowId, index, groupId, pinned, title, url).

```
cua.py tabs-query [-h] [filter]
```

### `tabs-organize`
Plan (dry-run) or apply: close duplicate tabs, or group same-domain tabs; --apply needs --confirmed.

```
cua.py tabs-organize [-h] [--min MIN] [--window WINDOW] [--apply] [--confirmed] {dedupe,group-by-domain}
```

### `group-list`
List tab groups with their tab ids.

```
cua.py group-list [-h]
```

### `group-create`
Group tabs (new group, or add to --group-id), optionally title/color.

```
cua.py group-create [-h] --tabs TABS [--title TITLE] [--color {grey,blue,red,yellow,green,pink,purple,cyan,orange}] [--group-id GROUP_ID] [--collapsed {true,false}]
```

### `group-update`
Rename / recolor / collapse a tab group.

```
cua.py group-update [-h] [--title TITLE] [--color {grey,blue,red,yellow,green,pink,purple,cyan,orange}] [--collapsed {true,false}] group_id
```

### `ungroup`
Remove tabs from their groups.

```
cua.py ungroup [-h] tabs
```

### `pin`
Pin (or --off unpin) tabs.

```
cua.py pin [-h] [--off] tabs
```

### `tab-move`
Move tabs to an index (and optionally another window).

```
cua.py tab-move [-h] --index INDEX [--window WINDOW] tabs
```

### `sort-tabs`
Sort a window's unpinned tabs, or the given tab ids, by title/url/host.

```
cua.py sort-tabs [-h] [--by {title,url,host}] [--window WINDOW] [--tabs TABS] [--desc]
```

### `window`
Windows: list | create | update (focus/resize/move).

```
cua.py window [-h] [--id ID] [--move-tab WTAB] [--populate] [--focused {true,false}] [--state {normal,minimized,maximized,fullscreen}] [--left LEFT] [--top TOP] [--width WIDTH] [--height HEIGHT] {list,create,update}
```

### `bookmark`
Bookmarks: search | list | create | move | update | remove.

```
cua.py bookmark [-h] [--query QUERY] [--id ID] [--parent PARENT] [--title TITLE] [--url URL] [--index INDEX] [--confirmed] {search,list,create,move,update,remove}
```

### `mgmt`
Any chrome.tabs/windows/tabGroups/bookmarks method exposed by the management capability.

```
cua.py mgmt [-h] [--confirmed] {windows,tabs,tabGroups,bookmarks} method [args ...]
```

## Research playbooks (own tab, closed after)

### `market-scan`
Playbook: one keyword across Google (keywords, trending), YouTube (views/day), TikTok (engagement, hashtags, ads), Meta (ads, organic), Google Ads -> folder + summary.md.

```
cua.py market-scan [-h] --query QUERY [--advertiser ADVERTISER] [--geo GEO] --out OUT [--skip {keywords,youtube,tiktok,meta-ads,tiktok-ads,google-ads,meta-trends}]
```

### `meta-ads`
Playbook: scan Meta Ad Library for a keyword -> cards.json + ads.csv (+ videos, frame tiles); opens and closes its own tab.

```
cua.py meta-ads [-h] --query QUERY [--country COUNTRY] [--media {video,image,all}] [--all-status] [--scrolls SCROLLS] [--out OUT] [--download DOWNLOAD] [--frames] [--keep-media] [--keep-tab]
```

### `tiktok-ads`
Playbook: TikTok Ad Library search (advertiser, first/last shown, users seen, targeting, paid by, landing page) + optional Creative Center Top Ads.

```
cua.py tiktok-ads [-h] --query QUERY [--days DAYS] [--region REGION] [--more MORE] [--details DETAILS] [--top-ads] [--out OUT] [--keep-tab]
```

### `google-ads`
Playbook: Google Ads Transparency Center - advertisers matching a name/domain, their creatives, last shown, format, variants, countries (own tab, closed after).

```
cua.py google-ads [-h] --query QUERY [--region REGION] [--advertisers ADVERTISERS] [--details DETAILS] [--scrolls SCROLLS] [--out OUT] [--keep-tab]
```

### `tiktok-trends`
Playbook: TikTok organic heat for a keyword - videos with likes/comments/shares/saves, engagement ranking, top hashtags, + Creative Center trending hashtags.

```
cua.py tiktok-trends [-h] --query QUERY [--scrolls SCROLLS] [--details DETAILS] [--region REGION] [--no-hashtag-trends] [--out OUT] [--keep-tab]
```

### `yt-search`
Playbook: YouTube search with date/sort/kind filters -> title, url, channel line, views text (own tab, closed after).

```
cua.py yt-search [-h] --query QUERY [--when {any,hour,today,week,month,year}] [--sort {relevance,date,views}] [--kind {any,video,short}] [--limit LIMIT] [--keep-tab]
```

### `google-trends`
Playbook: keyword ideas from Google + YouTube autocomplete (+ a-z long tail), daily trending searches by country, optional Trends Explore.

```
cua.py google-trends [-h] [--query QUERY] [--geo GEO] [--lang LANG] [--expand] [--trending TRENDING] [--explore] [--out OUT]
```

### `meta-trends`
Playbook: Facebook video search for a keyword (uses this profile's Facebook login) - captions, pages, hashtags; view counts only when Facebook shows them.

```
cua.py meta-trends [-h] --query QUERY [--scrolls SCROLLS] [--out OUT] [--keep-tab]
```

### `play-profile`
Playbook: Google Play listing facts (title, developer, rating, reviews, downloads, ads/IAP) for packages or search phrases.

```
cua.py play-profile [-h] [--lang LANG] [--out OUT] [--keep-tab] targets [targets ...]
```

### `appstore-profile`
IOS App Store facts via Apple's public lookup API: rating, rating count, developer, genre, price, version dates, size (no download counts on iOS).

```
cua.py appstore-profile [-h] [--country COUNTRY] [--search SEARCH] [--out OUT] targets [targets ...]
```

## Tracking, creative analysis, reporting

### `ads-watch`
Competitor-ad history: run (scan Meta/TikTok/Google ad libraries and merge) | show | list | forget; records first/last observed, times seen, new and no-longer-seen creatives.

```
cua.py ads-watch [-h] [--query QUERY] [--advertiser ADVERTISER] [--sources SOURCES] [--scrolls SCROLLS] [--out OUT] [--limit LIMIT] [--confirmed] {run,show,list,forget} [name]
```

### `creative-tag`
AI tagging of ad creatives: prepare (download + 8-frame tiles + tag sheet) | set (Claude writes tags after reading a tile) | show | summary | taxonomy.

```
cua.py creative-tag [-h] [--dir DIR] [--limit LIMIT] [--per-advertiser PER_ADVERTISER] [--keep-media] [--id ID] [--hook HOOK] [--mechanic MECHANIC] [--visual VISUAL] [--structure STRUCTURE] [--cta CTA] [--emotion EMOTION] [--insight INSIGHT] {prepare,set,show,summary,taxonomy}
```

### `report`
Build one self-contained HTML report from a market-scan folder (+ creative tags, frames, ads-watch history, store listings).

```
cua.py report [-h] --from SRC [--watch WATCH] [--title TITLE] [--out OUT] [--max-frames MAX_FRAMES]
```

## History, logs, viewport, assets, export, CDP

### `history`
Search browsing history (sensitive, may need user approval: use narrowly).

```
cua.py history [-h] [--query QUERY] [--from FROM_DATE] [--to TO_DATE] [--limit LIMIT] [--confirmed]
```

### `logs`
Console messages captured for the current tab.

```
cua.py logs [-h] [--filter FILTER] [--level {debug,info,log,warn,error,warning}] [--limit LIMIT]
```

### `audit`
Recent model-initiated window/tab/bookmark changes (for undoing organisation changes).

```
cua.py audit [-h] [--limit LIMIT]
```

### `viewport`
Responsive testing: set (width/height or preset) | reset (Chrome floors the width near 416px).

```
cua.py viewport [-h] [--width WIDTH] [--height HEIGHT] [--preset {mobile,tablet,laptop,desktop}] {set,reset}
```

### `assets`
Page assets: list | bundle (bundle needs the runtime's file-system grant; may fail with EPERM).

```
cua.py assets [-h] [--out OUT] [--kind {image,font,stylesheet,video,script,other}] [--ids IDS] [--inventory INVENTORY] [--limit LIMIT] {list,bundle,save}
```

### `export`
Export Google Docs/Sheets/Slides (--type) or a YouTube transcript from the current tab.

```
cua.py export [-h] [--type {pdf,md,xlsx,csv,docx,pptx}] {gsuite,youtube}
```

### `cdp`
Raw DevTools Protocol send/events (behind Browser Use's approval gate: may be refused).

```
cua.py cdp [-h] [--params PARAMS] [--after AFTER] [--methods METHODS] [--limit LIMIT] [--wait WAIT] {send,events} [method]
```

## Batch and escape hatch

### `batch`
Run many commands in ONE runtime (faster, shares the tab); lines are normal cua.py commands without the program name.

```
cua.py batch [-h] [file]
```

### `js`
Run your own JS in the cua runtime (escape hatch); call out(value) to return a result.

```
cua.py js [-h] [--file FILE] [--args ARGS] [code]
```
