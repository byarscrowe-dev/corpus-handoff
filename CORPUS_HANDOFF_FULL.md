# CORPUS — Source of Truth
*Single document for all Claude sessions (main chat + Claude Code). Last updated: 2026-06-04 | Commit: 99aacc6*

---

## OPERATING PARAMETERS
*These govern how Claude (main chat) operates in every CORPUS session.*

You are operating as a senior-level AI collaborator on an ongoing software project. Internalize the following before responding to anything.

Treat Byars as a capable professional learning to build. Calibrate depth accordingly — explain new concepts briefly when introducing them, skip fundamentals he already knows. Do not over-explain. No preamble, no filler affirmations, no unnecessary caveats.

Lead with the answer. Context and reasoning follow, never precede. Default to prose over bullet points unless the information is genuinely list-shaped. No hedging language unless genuinely uncertain. Avoid passive voice.

If a request is ambiguous, make a reasonable assumption, state it in one sentence, and proceed. Do not ask multiple clarifying questions — one at most.

When producing code, commands, or structured output: make it copy-paste ready. No placeholders unless a template was explicitly requested.

If a request has a better framing, say so once, then do what was asked. Never repeat the concern. If something is wrong, say so directly. Do not soften warnings to the point of uselessness.

Distinguish clearly between: (a) established fact, (b) widely held view, (c) your inference, (d) genuine uncertainty.

Response length by task type: short factual queries get 1-3 sentences; creative or strategic tasks get a full deliverable with no truncation; technical tasks lead with working output and explain after if needed; long-form documents use headers only where navigation is genuinely useful.

**Persistent context:** Read this source of truth document at the start of any session where Byars references prior work. Fetch it via:
`https://raw.githubusercontent.com/byarscrowe-dev/corpus-handoff/main/CORPUS_HANDOFF_FULL.md`
Do not ask him to paste context — fetch it. Do not speculate about prior conversations — look them up or ask once.

---

## WHO IS BYARS

**Name:** Byars Crowe
**Age:** 28, Dallas TX
**Email:** byarscrowe@gmail.com
**Career:** Pivoting into commercial real estate brokerage. CORPUS is both a personal tool and a demonstration of technical initiative.
**GitHub:** byarscrowe-dev | Token expires ~August 28, 2026 — flag when approaching
**Coding level:** Complete beginner. Never assumes terminal knowledge. Always provide full, copy-pasteable commands.
**Critical:** Laptop is missing the 'd' key. He types without it. "eploy" = "deploy", "fiel" = "field", "aress" = "address". Never correct typos — understand them and move on.

---

## WHAT IS CORPUS

A private personal dashboard — Flask/SQLite web app hosting multiple internal projects (A through K), each a separate agent or tool. The name means "the body as operating system" — each project is an organ, distinct in function, unified in system.

**Live URL:** https://corpusbc.duckdns.org *(bare IP 137.184.211.205 now 301-redirects to HTTPS — IP alone is not on the cert SAN)*
**GitHub:** https://github.com/byarscrowe-dev/Corpus.git (private)
**Local path:** `C:\Users\Byars\Clau Playgroun\First proj empty foler\`
**Server path:** `/opt/corpus/`
**Stack:** Flask + SQLite → gunicorn → nginx → Ubuntu 24.04, DigitalOcean NYC1
**Branch:** master | **Current commit:** 99aacc6

---

## PROJECTS A–K

All project identities are defined in the `PROJECTS` dict at the top of `app.py` — single source of truth. Templates loop it; nothing is hardcoded.

| ID | Name | Status | Color | Route | Description |
|----|------|--------|-------|-------|-------------|
| RE_A | RE_A | ACTIVE | `#00ff88` | `/dashboard` | CRE job search agent |
| B | Project B | PLACEHOLDER | `#1a7fff` | `/project-b` | Prediction Market Arbitrage Bot |
| C | Project C | ACTIVE | `#ff8800` | `/project-c` | Stock trading simulation lab — Phase 1 live |
| D | Project D | PLACEHOLDER | `#ff3535` | `/project-d` | Artistic / Creative Project |
| E | Project E | PLACEHOLDER | `#00c8e8` | `/project-e` | Automated Media Clipper |
| F | Project F | PLACEHOLDER | `#e8c400` | `/project-f` | HTML Mini Game |
| G | Spotify Orbit | ACTIVE | `#8844ee` | `/project-g` | Spotify agent — 5 tabs, web playback, analytics |
| H | Project H | INCOMING | `#4a84aa` | `/project-h` | TBD |
| I | Project I | INCOMING | `#cc3366` | `/project-i` | TBD |
| J | Project J | INCOMING | `#8ec410` | `/project-j` | TBD |
| K | Project K | PLACEHOLDER | `#e89a00` | `/project-k` | Property Locator System — DFW |

---

## SERVER & DEPLOYMENT

**SSH (key auth, no password):**
```powershell
ssh root@137.184.211.205
```

**Standard deploy:**
```powershell
git add <specific files>
git commit -m "message"
git push
powershell -File deploy.ps1
```

**Note:** Use `powershell -File deploy.ps1`, NOT `.\deploy.ps1` — the dot-slash form fails in Claude Code's bash context.

**Update .env only:** `.\update_env.ps1`

**Copy Spotify token to server:**
```powershell
scp "c:\Users\Byars\Clau Playgroun\First proj empty foler\.spotify_cache" root@137.184.211.205:/opt/corpus/.spotify_cache
ssh root@137.184.211.205 "chown www-data:www-data /opt/corpus/.spotify_cache"
```

**Trigger job refresh manually (never via browser):**
```bash
ssh root@137.184.211.205 "nohup /opt/corpus/venv/bin/python3 /tmp/run_job_refresh.py > /tmp/corpus_job_refresh.log 2>&1 &"
ssh root@137.184.211.205 "tail -20 /tmp/corpus_job_refresh.log"
```

**Non-fatal warnings to always ignore:**
- `ERROR: Cannot install -r requirements.txt` — JobSpy numpy conflict, harmless
- `[Errno 13] Permission denied: '/var/www/.gunicorn'` — gunicorn socket, harmless

**Hard constraints:**
- JobSpy must stay installed with `--no-deps` — never reinstall normally
- Never trigger job refresh from the browser — 504 timeout. SSH or scheduler only
- `dashboard.db` is the only database — WAL mode enabled; never raw-`cp` it (use `sqlite3 .backup`)
- Never force-push to master — no recovery branch
- Git hygiene: stage only the specific files being changed — never accidentally commit `linkedin_connections.csv`, `.claude/settings.local.json`, or `playlists/*.csv`
- Basic Auth covers ALL routes including `/api/*` — there is no bypass without editing `/etc/nginx/sites-available/corpus`

**Server .env:**
Credentials live in `/opt/corpus/.env` on the server and in the local `.env` file. They must never be written into this document. Authentication is nginx Basic Auth — `DASHBOARD_PASSWORD` is a dead variable and not needed.
```
SECRET_KEY=<see /opt/corpus/.env — never commit>
ADZUNA_APP_ID=<see /opt/corpus/.env — never commit>
ADZUNA_APP_KEY=<see /opt/corpus/.env — never commit>
SPOTIFY_CLIENT_ID=<see /opt/corpus/.env — never commit>
SPOTIFY_CLIENT_SECRET=<see /opt/corpus/.env — never commit>
SPOTIFY_REDIRECT_URI=https://corpusbc.duckdns.org/callback
```

**SECURITY / NGINX:**
- Active config: `/etc/nginx/sites-available/corpus` — NOT `default`.
- Basic Auth: realm `CORPUS`, credentials at `/etc/nginx/.htpasswd`, username `byars`. Protects every route and every `/api/*` endpoint at nginx level before requests reach Python.
  - To regenerate password if lost: `ssh root@137.184.211.205 "htpasswd /etc/nginx/.htpasswd byars && nginx -s reload"`
- SSL cert: `/etc/letsencrypt/live/corpusbc.duckdns.org/fullchain.pem`
- Cert expiry: 2026-08-30. Auto-renewal via `certbot.timer` — no manual action needed.
- Firewall (UFW): allows 22, 80, 443.
- gunicorn binds to `127.0.0.1:5000`

**Files that live only on server (not in git):**
```
/opt/corpus/.env
/opt/corpus/.spotify_cache
/opt/corpus/linkedin_connections.csv
/opt/corpus/playlists/*.csv
/opt/corpus/price_cache/              ← Project C yfinance JSON cache
/opt/corpus/spotify_health_cache.json
/opt/corpus/analytics_cache.json
/opt/corpus/.playlist_id_cache.json
```

---

## DESIGN SYSTEM

**Philosophy:** CORPUS means "the body as operating system." The interface is not a window to something else — it IS the data materialized. Serious, precise, alive beneath the surface. Not corporate, not decorative. Every element earns its place.

**Base:** `#0f1419` (deep navy, not pure black)
**Font:** Monospace throughout. All caps for labels.
**Shape:** No rounded corners. Everything rectilinear. 1px chrome borders.
**Labels:** Bracketed notation only — `[ ACTION ]` `[ ACTIVE ]` `[ REFRESH ]`
**Layout:** Strict grid. Dense information as aesthetic — instruments, not landing pages.
**Cards:** Each project card is a distinct color territory with a 3x3 dot grid in the project color.
**Breadcrumb:** `CORPUS / PROJECT NAME / PAGE` on every page.
**Sidebar:** Two layers — outer (project nav) + inner (tab nav for current project).

**Always read `docs/design_brief.md` before any visual work.**

---

## RE_A — CRE JOB SEARCH AGENT

**Route:** `/dashboard`
**Pipeline:**
- Layer 1: Adzuna API — 29 queries across 14 CRE role types × Dallas TX variations
- Layer 2: JobSpy/LinkedIn scraping
- Layer 3: 37 company career pages via `company_scrapers.py`

**Scoring:** HIGH ≥ 20 | MED 12–19 (must also have CRE keyword in description) | LOW < 12 (hidden)

**Network matching:** `linkedin_connections.csv` — 429 connections. When a job's company matches a connection's employer, `connection_flag=1` and name is stored.

**Scheduler:** CronTrigger at 8:00 AM America/Chicago daily. Never trigger via browser — 504 timeout.

**Key files:** `job_agent.py`, `company_scrapers.py`

---

## PROJECT C — STOCK TRADING SIMULATION LAB

**Route:** `/project-c` | **Status:** ACTIVE — Phase 1 complete 2026-06-03

Multi-bot stock trading simulation. Each bot starts with $100k fake cash and runs a different strategy. Equity curves accumulate over time on a comparison chart with a leaderboard vs. SPY. Phase 1 is pure simulation — no real money, no live broker.

### Bots (10 total, 9 active)

| bot_id | Family | Strategy | Notes |
|---|---|---|---|
| spy_benchmark | benchmark | Buy & Hold SPY | Benchmark reference line — `#888888` grey |
| momentum | momentum | MA Crossover fast=20/slow=50 | Flagship — `#ff8800` orange |
| momentum_v2 | momentum | MA Crossover fast=20/slow=80 | Wider window, fewer trades |
| momentum_v3 | momentum | MA Crossover fast=20/slow=35 | Eager end of slow-window spectrum |
| momentum_v4 | momentum | MA Crossover fast=20/slow=40 | Fills gap between v3 and v1 |
| momentum_v5 | momentum | MA Crossover fast=20/slow=25 | Whipsaw-territory probe |
| chaos_random | chaos | RandomStrategy uniform/sell=10% | Scientific control — `#bb44ff` violet |
| chaos_lazy | chaos | RandomStrategy uniform/sell=0% | Isolates turnover vs chaos_random |
| chaos_hot | chaos | RandomStrategy recent_winners/sell=10% | Tests if momentum edge = just "buy winners" |
| twitter_x | (none) | — | Shelved — X API restrictions |

### Baseline backtest results (2022-01-03 → 2024-12-31) — all 9 bots, run 2026-06-04

| Bot | Return | Notes |
|---|---|---|
| momentum (slow=50) | +39.00% | |
| chaos_random | +38.52% | 0.48 pts behind v1 — coin-flip nearly beat structure |
| momentum_v4 (slow=40) | +34.74% | |
| chaos_hot | +31.96% | |
| spy_benchmark | +27.76% | benchmark |
| momentum_v5 (slow=25) | +25.18% | |
| momentum_v3 (slow=35) | +18.12% | below both neighbors — ladder is jagged |
| momentum_v2 (slow=80) | +13.55% | |
| chaos_lazy | +8.60% | sell=0%, lowest turnover |

Results are **survivorship-biased** (current SP500_100 universe). Key finding: coin-flip random beat 4 of 5 momentum variants — in this window, universe+stop did the work, crossover added ~nothing vs dice. Caveat: each chaos bot = one draw; forward and Mode 2 populations are the rigorous tests.

### Architecture highlights

- 6 new DB tables: c_bots, c_trades, c_equity_snapshots, c_positions, c_portfolio_state, c_pending_orders
- Strict mode isolation: backtest never touches c_positions, c_portfolio_state, or c_pending_orders
- T+1 execution: signals queue orders; orders fill at next day's open — no same-day fills
- Backtest idempotency: re-run replaces old backtest rows for that bot — safe to re-run from UI
- Forward idempotency guard: checks for existing today's snapshot; calling run_forward_step multiple times is safe
- SPY-derived trading calendar — no external calendar dependency
- yfinance price source with JSON file cache in `price_cache/` (gitignored)
- **Variant rule:** serious strategy change = new bot_id; never edit a bot with live forward history

### Scheduler

`run_all_forward_bots` fires at **6:00 PM CT weekdays** (3h after market close — yfinance data has propagated). First run 2026-06-03. Forward rows are append-only.

### UI structure (2026-06-04)

Tab strip: `[ OVERVIEW ] [ MOMENTUM BOTS ] [ CHAOS BOTS ] [ SPY BUY & HOLD ] [ ENGINE ]`

**OVERVIEW:** Family-champion leaderboard (one row per family; champion = highest return in current mode; benchmark exempt from competition). Chart draws 3 curves — one champion curve per family in family base color; benchmark dashed grey. Mode toggle (backtest/forward) drives both leaderboard and chart. Default mode uses diverged-from-$100k rule.

**Family tabs:** Family header (colored left-border + description) → family leaderboard (all members, vs. SPY) → family chart (all members in family-color shades + SPY dashed grey overlay, own mode toggle) → per-bot detail sections (metrics, mode toggle, positions, trades, backtest/baseline buttons) always visible nested inside the panel.

**FAMILY_REGISTRY** defined in `stock_agent.py`: 3 families — momentum `#ff8800`, chaos `#bb44ff`, benchmark `#888888`. Every new bot must declare `family` in BOT_REGISTRY. `subfamily` optional (null OK now). `initialize_bots` writes `_family`/`_subfamily` to config_json; propagates to all existing rows on each restart.

### Project C routes

| Route | Method | Purpose |
|---|---|---|
| `/project-c` | GET | Main page — family tabs, champion overview, ENGINE tab |
| `/api/project-c/equity-data` | GET | Snapshots by bot_id + mode + `family`/`subfamily`; `_families` top-level key |
| `/api/project-c/bot/<bot_id>` | GET | Per-bot metrics (bt_metrics/fwd_metrics/default_mode), positions, last 50 trades |
| `/api/project-c/backtest/run` | POST | `{bot_id, start_date, end_date}` or `{bot_id, baseline:true}` — fire backtest |
| `/api/project-c/backtest/status` | GET | `{running, bot_id, pct}` — poll progress |

### Phase 2+ roadmap (each needs a design session)

Capitol Trades political disclosure bot → Claude-static bots → GDELT news bot → 13F follower → Claude Live API bot → AI-ETF mirror bot. `stock_ingest.py` stub is already in place for Phase 2.

**Bots as of 2026-06-04 (10 total, 9 active):** Momentum ladder v1(50)/v2(80)/v3(35)/v4(40)/v5(25); chaos_random (uniform/sell=10%), chaos_lazy (uniform/sell=0%), chaos_hot (recent_winners/sell=10%); spy_benchmark; twitter_x (shelved). Baseline backtests complete for all 9 — see results table above. Forward: momentum/momentum_v2/spy_benchmark have 1 row from 2026-06-03; full 9-bot runs begin tonight (6 PM CT). Verify: journalctl ran=9 skipped=1 failed=0; 9 rows dated 2026-06-04; SPY value diverges from $100k (KI#12 fix production test).

**Phase 2 — Capitol Trades design questions:** data source (capitoltrades.com scrape vs Quiver Quant API vs raw House/Senate disclosures), signal rules (buy on first disclosure? accumulate? threshold?), universe (trades may be outside SP500), disclosure lag ~45 days (edge likely decayed — bot tests whether signal survives).

**Baseline backtest button:** `BASELINE_BACKTEST_START = '2022-01-03'`, `BASELINE_BACKTEST_END = '2024-12-31'` defined once in app.py — single source of truth. `POST /api/project-c/backtest/run` accepts `{"bot_id": ..., "baseline": true}` to use these constants server-side; start/end in payload ignored. [ RUN BASELINE 2022–2024 ] button appears next to [ RUN BACKTEST ] in both the per-bot tab and engine tab. All new bots should be baseline-tested so comparisons share one window.

**Future — Multi-horizon robustness panel:** Four horizons (1/5/10/20-year) replacing the single baseline; four equity graphs on overview. Needs design session — survivorship bias worsens at long horizons; yfinance data gets patchy past ~20 years; rolling calendar questions. Baseline-constant pattern is the architectural seed.

**Variant discipline rule:** A variant changes EXACTLY ONE thing vs its parent. Non-varied params stay at genre house defaults. Multi-change variants are uninterpretable.

**Claude-static caveat:** Any backtest of Claude-written bots is contaminated by Claude's hindsight knowledge of 2022-24 market events. Forward performance is the only honest grade.

**Mode 2 — Population Search (long-term):** Large populations of randomized bots judged on forward results only. Requires fluke-control protocol before building: (a) winners must keep winning on new data for months; (b) larger populations need higher significance bar; (c) clone winners into near-variants to distinguish real edge from noise. Roadmap: genres → variants (Mode 1) → population search (Mode 2).

---

## SPOTIFY ORBIT (PROJECT G)

**Route:** `/project-g`
**5 tabs:** PLAYLISTS | SYNC | ANALYTICS | DISCOVER | MOOD
**6 macro playlists:** Rap Old School, Rap New Era, Rock Classic, Rock Modern, Electronic Dance, Late Night Melancholy (~2809 songs total)
**Web Playback SDK:** Persistent player widget fixed bottom-right on all CORPUS pages. Requires Spotify Premium (confirmed).

**Critical API fixes — never revert:**
- Use `sp._post('me/playlists')` not deprecated `user_playlist_create`
- Track data is under `'item'` key not `'track'` in newer API responses
- Use `auth_manager=` in Spotipy client to prevent 401 on token expiry
- `get_playlist_health` uses single `playlist_items(limit=1)` call for speed
- Audio features API returns 403 in dev/non-quota mode since Nov 2024 — energy/valence bars showing 0 is expected, not a bug
- `retries=0` on Spotipy client — prevents 14-hour thread blocks on rate limit

**Caching:** `spotify_health_cache.json` (1h), `analytics_cache.json` (24h), `.playlist_id_cache.json` (24h). All refreshes are fire-and-poll async.

---

## CLAUDE CODE MEMORY FILES

Location: `C:\Users\Byars\.claude\projects\c--Users-Byars-Clau-Playgroun-First-proj-empty-foler\memory\`

Always begin every CC session with: **"Read all memory files first for full context, including handoff_protocol.md."**

| File | Contents |
|------|----------|
| MEMORY.md | Index of all memory files |
| user_profile.md | No 'd' key, terminal comfort, GitHub token |
| project_corpus.md | Core project info, deployment, constraints |
| vps_setup.md | Server IP, stack, SSH, deployment flow |
| projects_overview.md | All A–K project statuses |
| project_re_a.md | Scraping layers, scoring, LinkedIn, scheduler |
| project_c.md | Stock trading lab — bots, architecture, conventions, roadmap |
| project_g_spotify.md | Spotify API fixes, auth, never-revert list |
| audit_findings.md | VPS audit 2026-05-29 + 2026-06-03 housekeeping |
| handoff_protocol.md | GitHub push protocol — read at session start, follow after every commit |
| design_brief.md | CORPUS visual design system — read before any visual or UI work |

---

## SESSION PROTOCOLS

### New Claude Code Session
**Start:** User opens with: *"Read all memory files first for full context, including handoff_protocol.md."*
CC reads all memory files, then fetches SESSION LOG and IMMEDIATE NEXT ACTIONS from the public URL to orient on what changed last session.
**End:** CC auto-updates the source of truth and pushes both repos per the handoff protocol below. Confirm both pushes succeeded. Nothing required from the user.

### New Main Chat Session (always inside the CORPUS project)
**Start:** User opens with: *"Fetch the CORPUS handoff doc at https://raw.githubusercontent.com/byarscrowe-dev/corpus-handoff/main/CORPUS_HANDOFF_FULL.md. If it shows a commit older than a week, verify against the browser view at https://github.com/byarscrowe-dev/corpus-handoff/blob/main/CORPUS_HANDOFF_FULL.md (the raw URL sometimes lags due to GitHub cache). Once you have the current version, you're calibrated. Then ask what I want to work on."*
**End:** Nothing required. Decisions/plans that must persist get logged by CC via a user prompt. Main chat reads the source of truth but cannot write to it — only CC writes.

### Standing Rules (always apply)
1. **Three-tier model:** (a) credentials → `.env` only, never in any doc/memory/chat; (b) private context → CC memory files only, never pushed; (c) shareable context → this public handoff doc only. Sort every new piece of information into one tier at creation.
2. **Work only from inside the CORPUS project.** Outside conversations cannot see memory or route through CC.

---

## CLAUDE CODE UPDATE PROTOCOL
*CC must follow this at the end of any session that makes a meaningful change.*

After any commit that changes functionality, routes, known issues, or project status, Claude Code must:

1. Update `docs/CORPUS_HANDOFF_FULL.md` in the main private repo — only the changed sections
2. Log a one-line session entry in the Session Log section (date, what changed, commit hash)
3. Update Known Issues, Git Restore Points, Immediate Next Actions, Decision Log as needed
4. Commit the local copy: `git commit -m "docs: update handoff document"`
5. Copy the updated content to `C:\Users\Byars\Clau Playgroun\corpus-handoff\CORPUS_HANDOFF_FULL.md`
6. In the corpus-handoff repo: `git add CORPUS_HANDOFF_FULL.md && git commit -m "update handoff doc" && git push origin main`
7. Push the main private repo: `git push`

**CREDENTIAL RULE — never negotiate this:**
This document is mirrored to a public GitHub repo. It must never contain real secret values. All credentials must appear as `<see /opt/corpus/.env — never commit>`.

**The public URL main chat reads:**
`https://raw.githubusercontent.com/byarscrowe-dev/corpus-handoff/main/CORPUS_HANDOFF_FULL.md`

---

## SESSION LOG
*One line per session. Claude Code writes to this after every meaningful commit.*

| Date | Summary | Commit |
|------|---------|--------|
| 2026-05-29 | VPS deployment, Spotify Orbit build, LinkedIn matching live | 460b36a |
| 2026-05-30 | Remove split playlist functionality, play buttons, analytics cache | b000ed1 |
| 2026-05-30 | Project registry (PROJECTS dict), Spotify SDK full logging + connect() fix, player auto-expands on Orbit | 3e24d09 |
| 2026-05-30 | Switch handoff system from Google Drive to public GitHub repo; design_brief copied to memory | a90b329 |
| 2026-06-01 | HTTPS + Basic Auth via nginx, DuckDNS domain (corpusbc.duckdns.org), source_url XSS fix, all audit findings resolved | 6bf3e57 |
| 2026-06-01 | Fix Spotify redirect_uri — remove hardcoded localhost fallback, read SPOTIFY_REDIRECT_URI from env only | 22b4a52 |
| 2026-06-01 | Spotify login confirmed working — root cause was commented-out HTTPS line in server .env; local .env synced | env-only |
| 2026-06-02 | Credential leak remediation — Spotify secret + Adzuna key rotated, corpus-handoff history wiped to single clean commit, all handoff docs scrubbed to placeholders, three-tier credential rule added | docs-only |
| 2026-06-02 | Session protocols + standing rules added to handoff doc and CC memory; MEMORY.md index updated | docs-only |
| 2026-06-03 | Project C Phase 0b — WAL mode, 6 DB tables, stubs, /project-c UI skeleton deployed | 58430f7 |
| 2026-06-03 | Project C Phase 1 Steps 1–2 — stock_price.py (yfinance cache, SPY calendar) + stock_broker.py | 1230462 |
| 2026-06-03 | Project C Phase 1 Steps 3–4 — engine, strategies, app wiring, API routes, full UI live (96 tests total) | 3d23f9d |
| 2026-06-03 | VPS resized 512MB → 2GB RAM + 2GB swap; sqlite3 installed; .env cleaned; DB backups at /root/corpus_backups/ | server-only |
| 2026-06-03 | First backtests run via UI: momentum +39.00%, SPY +27.76%, momentum_v2 +13.55% (2022-01-03 → 2024-12-31) | UI-only |
| 2026-06-03 | First automated forward run verified (ran=3 skipped=1 failed=0); per-bot metrics display bug logged | server-only |
| 2026-06-04 | Fix Known Issue #12 — per-bot metrics now show backtest results when forward is day-one only; [BACKTEST]/[FORWARD] toggle added to metric cards; 10 new C24–C33 test assertions | ce2b5e5 |
| 2026-06-04 | Fix Known Issue #12 (leaderboard instance) — OVERVIEW leaderboard now obeys chart mode toggle; statusBadge and populateLeaderboard accept mode param; default mode uses diverged-from-$100k check | b5630dd |
| 2026-06-04 | Planning session — momentum_v3 greenlit; chaos bot greenlit; variant discipline rule; Mode 2 population search; bot genres; universe expansion; deploy.ps1 URL fix | docs-only |
| 2026-06-04 | Six new bots live: momentum eagerness ladder v3(35)/v4(40)/v5(25) + chaos_random/lazy/hot; RandomStrategy with deterministic seeding + recent_winners mode; 54 new tests (159 total) | 6421560 |
| 2026-06-04 | Baseline backtest button — BASELINE_BACKTEST_START/END constants in app.py; baseline:true API field; [ RUN BASELINE ] buttons in per-bot and engine tabs; 5 new test assertions (164 total) | c95f3a4 |
| 2026-06-04 | Family-based UI — FAMILY_REGISTRY (3 families: momentum/#ff8800, chaos/#bb44ff, benchmark/#888888); family/subfamily in BOT_REGISTRY + config_json; champion-per-family OVERVIEW leaderboard + chart; family tabs with shaded member charts + SPY overlay; per-bot sections nested inside family panels; equity-data adds family/_families; 5 new test assertions (63 in step4, 164 total) | f146d0a |
| 2026-06-04 | Click-to-expand bot details — family leaderboard rows clickable; one-at-a-time detail slot below family chart; lazy /api/project-c/bot fetch on first click; [ × ] close; active row family-color highlight; responsive width fixes (metrics grid minmax 160px, overflow-x on panels + table wrapper) | 79ab52b |
| 2026-06-04 | Fix panel scroll — flex-shrink:0 on .pc-panel.active children forces overflow-y:auto to scroll instead of compressing content; remove auto-scrollIntoView on row click; stronger active-row highlight (0.13 opacity) | 5e91889 |
| 2026-06-04 | Display integrity — no silent cross-mode substitution: seriesForMode removes forward→backtest fallback; statusBadge shows PENDING (not BACKTEST) in forward view; family leaderboard always renders rows; sparse-series point dots; honest bt_metrics positions_count from trade replay; mode-aware positions empty state; static mode label; 2 new test assertions (65 total) | c7cad32 |
| 2026-06-04 | Merged SNAPSHOT health column — date badge replaces separate LAST SNAPSHOT + STATUS columns; green/red/grey freshness coloring in forward mode; neutral grey date in backtest mode; fix stale vs.SPY cell persisting on mode switch | 2ba4bbe |
| 2026-06-04 | Mega memory backup — full baseline results (all 9 bots), key findings, workflow migration, fixture-collision rule, Phase 2 design questions, position-sizing ladder parking lot, UI architecture accuracy pass | docs-only |
| 2026-06-04 | gitignore playlist CSV exports — stop recurring git-status clutter | f04ac47 |
| 2026-06-04 | CLAUDE.md auto-boot pointer — routes every session to memory + handoff, inlines hard rules | 99aacc6 |

---

## KNOWN ISSUES

| # | Issue | Status | Notes |
|---|-------|--------|-------|
| 1 | Play buttons exist but URIs empty until cache populated | `[ RESOLVED ]` | Click `[ REFRESH PLAYLISTS ]` on Sync tab confirmed working |
| 2 | Analytics tab shows "NO DATA" | `[ RESOLVED ]` | Click `[ REFRESH ]` on Analytics tab — confirmed working |
| 3 | Job count low on dashboard | `[ OPEN ]` | Adzuna duplicate credentials fixed. Wait for 8 AM CT daily scheduler run. Never use browser Refresh Jobs. |
| 4 | Audio feature bars show 0 | `[ KNOWN LIMITATION ]` | Spotify audio features API returns 403 in dev mode since Nov 2024. Not fixable without Extended Quota approval |
| 5 | No swap memory on VPS | `[ RESOLVED — 2026-06-03 ]` | VPS resized to 2GB RAM + 2GB swap |
| 6 | Spotify SDK player shows NOT CONNECTED | `[ RESOLVED — 22b4a52 ]` | Redirect URI fix confirmed working. Re-authenticate if token expires: visit `https://corpusbc.duckdns.org/spotify/login` |
| 7 | No authentication on any route | `[ RESOLVED — nginx Basic Auth 2026-06-01 ]` | All routes protected at nginx level |
| 8 | No HTTPS — all traffic in cleartext | `[ RESOLVED — Let's Encrypt 2026-06-01 ]` | TLS cert via certbot for `corpusbc.duckdns.org`. Auto-renews. Expires 2026-08-30. |
| 9 | `source_url` XSS — `javascript:` scheme possible in job URLs | `[ RESOLVED — 6bf3e57 ]` | `jobs.html` only renders `<a href>` when scheme is http/https |
| 10 | Credentials committed to public corpus-handoff repo | `[ RESOLVED — 2026-06-02 ]` | Keys rotated, history wiped, three-tier credential rule enforced |
| 11 | `deploy.ps1` prints bare IP at end | `[ OPEN — cosmetic ]` | Change to print `https://corpusbc.duckdns.org` |
| 12 | Per-bot metrics cards show forward day-one values ($100k) instead of backtest results once forward data exists | `[ RESOLVED — ce2b5e5, b5630dd ]` | Endpoint returns bt_metrics + fwd_metrics separately (ce2b5e5). [BACKTEST]/[FORWARD] toggle on metric cards. OVERVIEW leaderboard also fixed to obey chart mode toggle (b5630dd). Both fixes use diverged-from-$100k default logic. |

---

## DECISION LOG
*Why things are built the way they are. Prevents re-litigating solved problems.*

| Decision | Rationale | Date |
|----------|-----------|------|
| JobSpy installed with `--no-deps` | numpy/tls-client conflict breaks standard install. This is the only working approach | early |
| Job refresh runs on scheduler, never via browser | Browser requests 504 after 120s — scraping takes 3-5 min | early |
| Spotify sync delayed 24h after restart | Prevents hammering Spotify API on every service restart | 2026-05-29 |
| Async fire-and-poll for all Spotify refresh operations | Spotify API calls too slow to block page render | 2026-05-29 |
| Single gunicorn worker | Background threads share the worker. Adding workers would break shared state | 2026-05-29 |
| `dashboard.db` only | `corpus.db` was a stray empty file, deleted | 2026-05-29 |
| PROJECTS registry dict in app.py | Single source of truth for all project identities. Templates loop it — no labels hardcoded | 2026-05-30 |
| Handoff doc moved from Google Drive to public GitHub repo | Drive MCP has no update tool — creates new file on every write. GitHub push is atomic, versioned, and fetchable by main chat via raw URL | 2026-05-30 |
| HTTPS + nginx Basic Auth over per-route Flask auth | Single `auth_basic` block in nginx protects every route. Simpler than adding `@login_required` decorators everywhere | 2026-06-01 |
| DuckDNS subdomain over paid domain | Let's Encrypt cannot issue certs for bare IPs. DuckDNS gives a free subdomain with instant setup | 2026-06-01 |
| Three-tier credential model | GitGuardian flagged Adzuna + Spotify credentials in the public corpus-handoff repo. Real values live only in `.env` files, never in any doc | 2026-06-02 |
| SPY-derived trading calendar (no external dependency) | `pandas_market_calendars` adds complexity and a dependency; SPY's own price history is sufficient for all calendar needs | 2026-06-03 |
| T+1 execution model | Real equity orders fill at next day's open, not same-day close. Using signal-day execution would overstate returns | 2026-06-03 |
| Backtest idempotency: wipe + re-insert; forward rows append-only | Backtests need to be rerun safely; forward history is ground truth that must never be deleted | 2026-06-03 |
| Bot variant rule: serious changes get a new bot_id | Editing a live bot's config stitches two strategies into one uninterpretable equity curve. New bot_id preserves clean, honest history for both variants | 2026-06-03 |
| Stock tests run against throwaway DATABASE_PATH temp DB | Tests must never touch live financial data in dashboard.db — even with idempotency guards in place | 2026-06-03 |
| Per-bot endpoint returns both bt_metrics and fwd_metrics; default_mode fallback prefers forward only when diverged | Day-one forward snapshot at $100k was masking backtest results. Coexistence with explicit default_mode allows the UI toggle without losing either dataset; forward-only bots (Phases 4/6) always default to forward | 2026-06-04 |
| stock_engine.py freeze interpretation: strategy class registration is the sanctioned exception | The freeze protects execution mechanics (T+1, no-lookahead, broker idempotency). Adding a new name to `_STRATEGY_MAP` is purely additive registration — no logic change. Future new strategy classes must be registered here; everything else in engine is frozen | 2026-06-04 |
| Scope default: reported issues apply to all bots/families unless explicitly narrowed | CORPUS is one system — partial fixes create inconsistent views that erode trust in the data. Standardization across all families is the default posture; deviations require a design discussion | 2026-06-04 |
| No silent cross-mode substitution: explicit mode shows only that mode's data | Silently swapping backtest data into a forward view or vice versa made the chart x-axis lie (2022 dates appearing in a "forward" view), hid missing data behind misleading BACKTEST badges, and created a false sense that forward data existed. Honest empty/PENDING states are less visually impressive but always accurate. The defaultModeForBots fallback chain is separate — it determines which mode is selected by default, not what renders | 2026-06-04 |
| Test fixture collision: always wipe relevant snapshot rows before seeding | Two incidents — C29: a pre-existing backtest snapshot at today's date masked the seeded 2024-12-31 test row (bt_snap ORDER BY DESC LIMIT 1 returned wrong row). C27/C28: Section B's run_forward_step wrote a forward snapshot for momentum with cash=75k from Section A's idempotency test; that row persisted into C24's mode comparison. Fix in both cases: DELETE FROM c_equity_snapshots WHERE bot_id=? (full bot, not just the specific date/mode) before seeding. Rule: scope the delete broadly | 2026-06-04 |

---

## IMMEDIATE NEXT ACTIONS

1. Verify tonight's forward run (6 PM CT) — journalctl `ran=9 skipped=1 failed=0`; 9 forward rows dated 2026-06-04; SPY value diverges from $100k (first production test of KI#12 default_mode logic)
2. Plan Phase 2 — Capitol Trades political disclosure bot (design session needed; `stock_ingest.py` Phase 2 stub ready; see design questions above)
3. Multi-horizon robustness panel design session — once meaningful forward data accumulates (weeks/months)
4. Future chaos exploration: churn-rate ladder (sell_prob never/10%/30%) — pure-randomness variable isolation

---

## PARKING LOT — IDEAS NOT YET STARTED

**Project B — Prediction Market Arbitrage Bot**
Monitor Polymarket and Kalshi simultaneously for arbitrage opportunities — same event priced differently across platforms. Needs real-time price feeds, arbitrage detection layer, eventually trade execution. Open questions: event name mapping across platforms, fee structure minimum spread, rate limits.

**Project E — Automated Media Clipper**
Monitor YouTube channels for new uploads, identify highlight moments, clip and process into short-form vertical video, auto-post to TikTok and Instagram. Open research needed: clip identification method, YouTube ToS on reclipping, copyright avoidance for monetization, TikTok/Instagram posting APIs.

**Project K — Property Locator System**
Multi-bot system for undervalued DFW properties. First bot: underpriced duplexes evaluated on price-to-rent ratio and cap rate. Future bots: small multifamily, commercial land, distressed, off-market.

**Project C — Position-sizing variant ladder**
Vary position_size_pct (5%/10%/20%) keeping all else constant — isolates sizing from signal quality. All current bots use 10%. Single-variable discipline: one bot per sizing level, compared within same strategy family. No design block — low effort.

**Project C — Genre leaderboard display question**
When multiple genres exist, should the main leaderboard show each genre's average return or its best performer? Settle at build time.

**Project C — Full backtest holdings list**
Reconstruct end-of-backtest holdings (ticker / shares / avg-cost) by replaying `c_trades WHERE mode='backtest'` for each bot. `bt_metrics.positions_count` already does net-share replay to COUNT open positions; the holdings TABLE needs the same replay plus current-price enrichment from YFinancePriceSource. The positions panel currently shows `[ POSITION DETAIL IS RECORDED FOR FORWARD ONLY ]` in backtest view. No design block — scope only.

**Project C — Bot genres for future ideation (design session before building each)**
Calendar/seasonality effects; volatility-regime switching (different behavior in calm vs panicked markets); mean-reversion / anti-momentum (buy what crashed); insider-buying via SEC Form 4 (~2-day filing lag, less than Capitol Trades); sector-rotation; dividend-aristocrat tilt. Universe expansion to full S&P 500 (~5x nightly fetch, acceptable; does NOT fix backtest survivorship bias unless historical membership data added). Genre leaderboards UI once multiple genres exist.

**Network graph for RE_A**
Design brief specifically calls this out. LinkedIn connections as a living graph — companies as nodes, connections as edges, colored by industry/relationship strength. Data already exists.

**Heraldic marks**
Each project eventually gets its own emblem — chosen, not designed. Long-term identity system mentioned in design brief.

**Halftone rendering tool**
For Project D — render uploaded images in dot-matrix style matching CORPUS visual language.

---

## WORKFLOW — DISPATCH & EXECUTION (migration complete 2026-06-04)

**Status: COMPLETE as of 2026-06-04.** The dispatch-agent transition is done; the structure below is live, not planned.

- **Execution is native in the Claude Code tab.** Flow: plan-gate → human approves the plan → Accept-edits execution for file edits, with terminal and git commands still individually gated (explicit approval each time).
- **CLAUDE.md auto-boot is live** (commit `99aacc6`). Every session loads `CLAUDE.md` at the repo root, which routes to the two mandatory-read files (`memory/project_c.md`, `docs/CORPUS_HANDOFF_FULL.md`) and inlines the hard rules.
- **Visual verification uses Claude Code's built-in preview** — this replaced the earlier Playwright-MCP plan.
- **MCP-dispatch architecture is abandoned** as the active plan (a separate orchestrator brain registering CC as an MCP server via `claude mcp serve`). It remains a KNOWN FALLBACK if a dedicated orchestrator is ever wanted.
- **The claude.ai web thread is retired** as the predecessor planning seat. Ideation/strategy now lives in the Chat tab with full cross-project context; builds run in task-scoped Code sessions; written dispatch prompts are the handoff.
- **Workflow vision, the ideation/execution bifurcation, and the Auto-mode trust plan live in `memory/workflow_structure.md`** (private memory) — evaluate future workflow decisions against it.

---

## GIT RESTORE POINTS

| Commit | Description | Date |
|--------|-------------|------|
| a2851e1 | Mega memory backup — full baseline results, workflow migration, fixture-collision rule | 2026-06-04 |
| 2ba4bbe | Merged SNAPSHOT health column — date badge with freshness coloring; fix stale vs.SPY on mode switch | 2026-06-04 |
| c7cad32 | Display integrity — no cross-mode substitution, honest positions_count, sparse-series dots, PENDING states | 2026-06-04 |
| 79ab52b | Click-to-expand bot details on family pages — lazy detail slot, responsive width fixes | 2026-06-04 |
| f146d0a | Family-based UI — FAMILY_REGISTRY, champion overview, family tabs with shaded charts; 5 new test assertions | 2026-06-04 |
| c95f3a4 | Baseline backtest button — BASELINE_BACKTEST_START/END constants, baseline:true API, 5 new tests | 2026-06-04 |
| 6421560 | Six new bots: momentum v3/v4/v5 + chaos_random/lazy/hot; RandomStrategy; 54 tests | 2026-06-04 |
| b5630dd | Fix Known Issue #12 leaderboard — OVERVIEW rows now obey chart mode toggle | 2026-06-04 |
| ce2b5e5 | Fix Known Issue #12 — dual-mode metrics, [BACKTEST]/[FORWARD] toggle, 10 new tests (53 total) | 2026-06-04 |
| 3d23f9d | Project C Step 4 — app wiring, API routes, full UI live (43 tests) | 2026-06-03 |
| a1cc9a7 | Project C Step 3 — strategies, engine, test suite (53 tests) | 2026-06-03 |
| 1230462 | Project C Step 2 — BacktestBroker + SimulatedBroker | 2026-06-03 |
| 265e846 | Add 'tickers' explicit-list to resolve_tickers (test fixture support) | 2026-06-03 |
| 340d97f | Project C Step 1 — stock_price.py + sp500_universe.csv | 2026-06-03 |
| 58430f7 | Project C Phase 0b — WAL, 6 DB tables, stubs, /project-c UI skeleton | 2026-06-03 |
| 22b4a52 | Fix Spotify redirect_uri — read from env only | 2026-06-01 |
| 6bf3e57 | HTTPS + Basic Auth + source_url XSS fix | 2026-06-01 |
| b000ed1 | Remove split playlist functionality from UI, routes, and agent | 2026-05-30 |
| 3e24d09 | Project registry + Spotify SDK fixes (logging, connect() promise, auto-expand) | 2026-05-30 |
| a90b329 | Switch handoff to public GitHub repo, design_brief to memory | 2026-05-30 |
| 460b36a | Play buttons, analytics cache, Sp_O sidebar, Adzuna fix | 2026-05-30 |
| cc80bc7 | CronTrigger 8AM CT, timezone display in CT | 2026-05-29 |
| 730e2fe | Rate limit hotfix: retries=0, Spotify sync 24h delay | 2026-05-29 |
| bb51f60 | Fix deploy.ps1: safe.directory before git pull | 2026-05-29 |
| 0ac05bc | v1.0: Project G — full Spotify agent | early |
| a425590 | v0.7: LinkedIn network matching via CSV | early |
| 5ba1b22 | v0.1: Initial snapshot | early |

---

*This document is the single source of truth for CORPUS. Claude Code pushes it to the public corpus-handoff repo after every meaningful commit. Main chat fetches it via raw GitHub URL at session start. Byars does not manually maintain it — the system does.*
