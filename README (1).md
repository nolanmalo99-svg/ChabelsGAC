# Chabels — Fantasy Football League Site

A static site for your ESPN fantasy league. Every page (standings, teams, history, awards,
matchups) is rendered in the browser from JSON pulled live out of the ESPN Fantasy API —
nothing is hand-typed. A GitHub Action re-syncs that data on a schedule and commits it, so
GitHub Pages always shows a recent snapshot.

Your League ID (`1720416456`) is already baked into `tools/lib.py` — it's not sensitive,
it's just the public ID in your league's URL. What *is* sensitive is your ESPN login
cookies, which are required because your league is private.

## 1. Get your ESPN cookies

1. Log into your league at https://fantasy.espn.com in a desktop browser.
2. Open DevTools (F12) → **Application** (Chrome) or **Storage** (Firefox) → **Cookies** →
   `https://fantasy.espn.com`.
3. Copy the values of two cookies:
   - `espn_s2` (a long string)
   - `SWID` (looks like `{XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX}`, including the curly braces)

## 2. Add them as GitHub Secrets

In your GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**

| Secret name | Value |
|---|---|
| `ESPN_S2` | the `espn_s2` cookie value |
| `ESPN_SWID` | the `SWID` cookie value (with braces) |

## 3. Turn on GitHub Pages

**Settings → Pages → Source: Deploy from a branch → `main` / `/(root)`**. Your site will be
live at `https://<you>.github.io/<repo>/`.

## 4. Run the first sync

Go to **Actions → Sync ESPN data → Run workflow**. This fetches every season since 2022,
computes standings/history/awards/head-to-head, and commits `js/site-data.js` +
`js/matchups-data.js`. Refresh the site after it finishes.

After that, it re-runs automatically every 4 hours during the season (Sep–Jan). You can
always trigger it manually from the Actions tab too.

## Running locally (optional)

```bash
cd tools
echo "ESPN_S2=..."   >> .env
echo "ESPN_SWID=..." >> .env
python3 build_data.py
```

This writes `js/site-data.js` and `js/matchups-data.js` in the project root — open
`home.html` directly in a browser (or `python3 -m http.server` from the project root) to
preview.

## Project layout

```
home.html, teams.html, history.html, awards.html, matchups.html   -- static page shells
js/site-data.js, js/matchups-data.js                               -- GENERATED, don't hand-edit
js/gate.js, js/polish.js                                            -- site chrome, generic
css/style.css                                                       -- theme (dark slate/green)
tools/lib.py       -- ESPN API client + league ID + position/team constants
tools/league.py    -- pulls this season's matchups/rosters/scores (one API call)
tools/history.py   -- pulls every season, computes career stats/h2h/records
tools/build_data.py -- orchestrates the above, writes the js/*.js data files
.github/workflows/sync.yml -- scheduled + manual sync job
```

## Adding pages later

The original template this was adapted from also had an **Analytics** page and an
AI-written weekly recap pipeline — both were left out here to keep things simple and
free of an Anthropic API key requirement. Both can be added back later if you want them.
