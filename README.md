# CREED dashboard (formerly ALVIN)

Rebooted 2026-08-14. One static index.html (GitHub Pages) showing the whole
CREED-operated trading stack at a glance:

- component health (risk-engine server, tunnel) with red ACTION NEEDED banner
- paper account equity + day P/L
- AUGUST 0DTE bot: mode + today's session verdict
- signal tournament: today's signal count by strategy + recent signal table

Data source: state.json, written by ..\CREED\creed_publish.py every 5 min
on market weekdays (task CREED-Publish).

## Two ways to view

1. Local (works now): python -m http.server 8090 in this folder ->
   http://localhost:8090 - reads the sibling state.json the publisher writes.
2. Anywhere (needs 2-min setup): the FLIPPER-era gist pattern -
   - create a public gist with one file state.json (any content)
   - put the gist id in ..\CREED\creed_config.json -> dashboard.gist_id
   - put a GitHub PAT with gist scope in ..\CREED\state\gist_token.txt
   - paste the gist RAW url into GIST_RAW_URL at the top of index.html
   - push this repo; enable GitHub Pages -> open the Pages URL from any device

Publishes PAPER data only - no keys, no tokens, no order ids.

FLIPPER's original dashboard lives in git history (git log before 2026-08-14).
