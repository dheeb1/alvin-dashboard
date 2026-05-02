# FLIPPER Dashboard

A static HTML dashboard, deployable on **GitHub Pages**, fed by a public **GitHub Gist** that the bot updates every ~15s.

```
[FLIPPER bot, your PC] ──PATCH /gists/<id>──▶ [Gist: state.json]
                                                     │
                                                     ▼
                                       [GitHub Pages: index.html]
                                       (open from any browser, anywhere)
```

---

## One-time setup

### 1. Create a public gist

Go to https://gist.github.com and create a **public** gist with one file:

- Filename: `state.json`
- Content: paste the contents of `sample_data.json` (anything valid — the bot will overwrite it)

After saving, the URL looks like `https://gist.github.com/<USER>/<GIST_ID>`. Note the **GIST_ID** (the long hex string at the end).

### 2. Get the gist's raw URL

On the gist page, click **Raw** on `state.json`. The URL will look like:

```
https://gist.githubusercontent.com/<USER>/<GIST_ID>/raw/state.json
```

### 3. Wire the dashboard to that gist

Open `dashboard/index.html` and replace the placeholder near the top of the `<script>` block:

```js
const GIST_RAW_URL = "REPLACE_WITH_GIST_RAW_URL";
```

with your raw URL from step 2.

### 4. Create a Personal Access Token (PAT) with gist scope

- https://github.com/settings/tokens → **Generate new token (classic)**
- Scope: just `gist`
- Copy the token (looks like `ghp_...`) — you only see it once.

### 5. Deploy the dashboard via GitHub Pages

Push the `dashboard/` folder to a public GitHub repo (or commit it to one you already have). Then:

- Repo **Settings → Pages**
- Source: branch + `/dashboard` folder (or move the files to `/docs` if Pages requires it)
- Save. After ~30s, your dashboard is live at `https://<you>.github.io/<repo>/`.

### 6. Run the publisher alongside the bot

In a PowerShell window (separate from the bot):

```powershell
$env:GIST_TOKEN = "ghp_xxx"
$env:GIST_ID    = "abc123def456..."
python dashboard_publisher.py
```

The publisher reads `brain.db`, `tape_state.json`, and `config.json`, builds a snapshot every 15s, and PATCHes the gist. Output looks like:

```
[publisher] starting; interval=15s; gist=abc123...
[publisher] pushed 6 trades — HTTP 200
```

That's it. Open the Pages URL from your phone, work laptop, anywhere — it auto-refreshes every 10s.

---

## Running locally without the gist

For dev / preview:

```powershell
python dashboard_publisher.py     # without GIST_TOKEN/GIST_ID, only writes dashboard/data/state.json
python -m http.server 8080 --directory dashboard
# open http://localhost:8080/?src=local
```

Or just preview against the bundled sample:

```powershell
python -m http.server 8080 --directory dashboard
# open http://localhost:8080/?src=sample
```

---

## What the dashboard shows

- **Header** — session date, bot mode (PAPER / LIVE + window), running indicator, last update time
- **Today cards** — Day P&L (sum of pnl_pct), Win Rate, Avg Win, Avg Loss, Best, Worst
- **Open Positions** — one row per live trade: open time, contract (CALL/PUT + strike), entry, last, unrealized P&L %, peak price, hold duration. Updated every ~5s by the bot (`TradeJournal.dump_open_positions` writes `open_positions.json` on every entry, ride snapshot, and exit).
- **Trades Today table** — one row per closed trade: entry/exit time, contract, entry price, exit price, realized P&L %, MFE %, hold duration, exit reason

If the bot crashes mid-trade, `open_positions.json` is treated as stale after 90 seconds and the dashboard hides it (rather than showing ghost positions).

---

## Notes & caveats

- **Gist raw URLs are CDN-cached up to ~60s.** The dashboard appends a `?_=<timestamp>` cache-buster, but the actual update may still lag the gist push by 30-60s. Acceptable for a monitoring dashboard, not a trading terminal.
- **The dashboard is public.** No account-identifying info is published — only contract symbols (e.g. `QQQ 260502C676`), prices, P&L %, and bot mode. Don't put anything sensitive in `notes` fields if the bot ever writes them.
- **Token security:** the PAT lives only on your local machine in env vars. It is never sent to the dashboard, never committed to the repo. Revoke at https://github.com/settings/tokens if leaked.
- **Time zone:** times are formatted in CT (America/Chicago) on the publisher side; the browser displays them as-is.
