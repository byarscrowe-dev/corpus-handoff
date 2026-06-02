# CORPUS — Source of Truth
*Single document for all Claude sessions (main chat + Claude Code). Last updated: 2026-06-01 | Commit: 3666801*

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
**Stack:** Flask + SQLite → gunicorn → nginx → Ubuntu 24.04, DigitalOcean NYC1 ($4/mo)
**Branch:** master | **Current commit:** 22b4a52

---

## PROJECTS A–K

All project identities are defined in the `PROJECTS` dict at the top of `app.py` — single source of truth. Templates loop it; nothing is hardcoded.

| ID | Name | Status | Color | Route | Description |
|----|------|--------|-------|-------|-------------|
| RE_A | RE_A | ACTIVE | `#00ff88` | `/dashboard` | CRE job search agent |
| B | Project B | PLACEHOLDER | `#1a7fff` | `/project-b` | Prediction Market Arbitrage Bot |
| C | Project C | PLACEHOLDER | `#ff8800` | `/project-c` | Stock Market Modeling & Trading Agents |
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
git add .
git commit -m "message"
git push
.\deploy.ps1
```

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
- `dashboard.db` is the only database — `corpus.db` was a stray, deleted
- Never force-push to master — no recovery branch
- No swap on VPS (458MB RAM) — add if scraping becomes unstable: `fallocate -l 1G /swapfile && chmod 600 /swapfile && mkswap /swapfile && swapon /swapfile`
- Basic Auth covers ALL routes including `/api/*` — there is no bypass without editing `/etc/nginx/sites-available/corpus`. Removing `auth_basic` from that file would re-expose the Spotify token endpoint publicly.

**Server .env:**
Credentials live in `/opt/corpus/.env` on the server and in the local `.env` file. They must never be written into this document.
```
DASHBOARD_PASSWORD=<see /opt/corpus/.env — never commit>
SECRET_KEY=<see /opt/corpus/.env — never commit>
ADZUNA_APP_ID=<see /opt/corpus/.env — never commit>
ADZUNA_APP_KEY=<see /opt/corpus/.env — never commit>
SPOTIFY_CLIENT_ID=<see /opt/corpus/.env — never commit>
SPOTIFY_CLIENT_SECRET=<see /opt/corpus/.env — never commit>
SPOTIFY_REDIRECT_URI=https://corpusbc.duckdns.org/callback
```

**SECURITY / NGINX:**
- Active config: `/etc/nginx/sites-available/corpus` — NOT `default`. The `sites-enabled/corpus` symlink is active; `default` is not loaded. All future nginx edits must target the `corpus` file.
- Basic Auth: realm `CORPUS`, credentials at `/etc/nginx/.htpasswd`, username `byars`. Protects every route — all pages and all `/api/*` endpoints. Auth happens at nginx before requests reach Python.
  - To regenerate password if lost: `ssh root@137.184.211.205 "htpasswd /etc/nginx/.htpasswd byars && nginx -s reload"`
- SSL cert: `/etc/letsencrypt/live/corpusbc.duckdns.org/fullchain.pem`
- SSL key: `/etc/letsencrypt/live/corpusbc.duckdns.org/privkey.pem`
- Cert expiry: 2026-08-30. Auto-renewal via `certbot.timer` systemd timer — no manual action needed.
- Firewall (UFW): allows 22 (SSH), 80 (HTTP → HTTPS redirect), 443 (HTTPS). Port 443 was previously closed; adding it fixed initial 000 connection failures.
- gunicorn binds to `127.0.0.1:5000` (confirmed from systemd service — NOT 8000)

**Files that live only on server (not in git):**
```
/opt/corpus/.env
/opt/corpus/.spotify_cache
/opt/corpus/linkedin_connections.csv
/opt/corpus/playlists/*.csv
/opt/corpus/spotify_health_cache.json
/opt/corpus/analytics_cache.json
/opt/corpus/.playlist_id_cache.json
```

---

## DESIGN SYSTEM

**Philosophy:** CORPUS means "the body as operating system." The interface is not a window to something else — it IS the data materialized. Serious, precise, alive beneath the surface. Not corporate, not decorative. Every element earns its place.

Reference aesthetic: RECORDING.ini terminal (phosphor green on black), ISS interior (cold institutional blue), magnetic core memory (a face emerging from a grid).

**Base:** `#0f1419` (deep navy, not pure black)
**Font:** Monospace throughout — JetBrains Mono or similar. All caps for labels.
**Shape:** No rounded corners. Everything rectilinear. 1px chrome borders.
**Labels:** Bracketed notation only — `[ ACTION ]` `[ ACTIVE ]` `[ REFRESH ]` `[ PLACEHOLDER ]`
**Layout:** Strict grid. Dense information as aesthetic — instruments, not landing pages.
**Cards:** Each project card is a distinct color territory, slightly tinted, with a 3x3 dot grid at low opacity in the project color.
**Spectrum stripe:** All project colors in sequence across the top of every page.
**Breadcrumb:** `CORPUS / PROJECT NAME / PAGE` on every page top bar.
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

**Network matching:** `linkedin_connections.csv` — 429 connections. When a job's company matches a connection's employer, `connection_flag=1` and name is stored. Green badge appears on listing.

**Scheduler:** CronTrigger at 8:00 AM America/Chicago daily. Spotify sync runs every 24h starting 24h after last restart.

**Key files:** `job_agent.py`, `company_scrapers.py`

---

## SPOTIFY ORBIT (PROJECT G)

**Route:** `/project-g`
**5 tabs:** PLAYLISTS | SYNC | ANALYTICS | DISCOVER | MOOD
**6 macro playlists:** Rap Old School, Rap New Era, Rock Classic, Rock Modern, Electronic Dance, Late Night Melancholy (~2809 songs total)
**Web Playback SDK:** Persistent player widget fixed bottom-right on ALL CORPUS pages. Device name: "CORPUS". Requires Spotify Premium (confirmed). Auto-expands on the Spotify Orbit page. All SDK lifecycle events log to browser console with `[CORPUS Player]` prefix.

**Button layout:**
- PLAYLISTS tab: playlist cards with `[ PLAY ]` buttons only (no toolbar buttons)
- SYNC tab: `[ SYNC NOW ]` + `[ REFRESH PLAYLISTS ]` + `[ IMPORT MACROS ]`
- ANALYTICS tab: `[ REFRESH ]` (triggers background 2809-song analysis)

**Critical API fixes — never revert:**
- Use `sp._post('me/playlists')` not deprecated `user_playlist_create`
- Track data is under `'item'` key not `'track'` in newer API responses
- Use `auth_manager=` in Spotipy client to prevent 401 on token expiry
- `get_playlist_health` uses single `playlist_items(limit=1)` call for speed
- Audio features API returns 403 in dev/non-quota mode since Nov 2024 — energy/valence bars showing 0 is expected, not a bug
- `retries=0` on Spotipy client — prevents 14-hour thread blocks on rate limit

**Caching architecture:**
- `spotify_health_cache.json` — playlist health + URIs, 1h TTL
- `analytics_cache.json` — 2809-song analysis, 24h TTL
- `.playlist_id_cache.json` — playlist name→ID mapping, 24h TTL
- All refreshes are fire-and-poll async — never block page render on Spotify API calls

**Spotify routes:** `/spotify/login`, `/spotify/callback`, `/callback` (alias), `/api/spotify/status`, `/api/spotify/token`, `/api/spotify/playlists`, `/api/spotify/playlists/refresh`, `/api/spotify/profile`, `/api/spotify/profile/refresh`, `/api/spotify/similar/<playlist>`, `/api/spotify/mood` (POST), `/api/spotify/import`, `/api/spotify/play-playlist` (POST), `/api/spotify/transfer-playback` (POST)

---

## CLAUDE CODE MEMORY FILES

Location: `C:\Users\Byars\.claude\projects\c--Users-Byars-Clau-Playgroun-First-proj-empty-foler\memory\`

Always begin every CC session with: **"Please read all memory files first for full context."**

| File | Contents |
|------|----------|
| MEMORY.md | Index of all memory files |
| user_profile.md | No 'd' key, terminal comfort, GitHub token |
| project_corpus.md | Core project info, deployment, constraints |
| vps_setup.md | Server IP, stack, SSH, deployment flow |
| projects_overview.md | All A–K project statuses |
| project_re_a.md | Scraping layers, scoring, LinkedIn, scheduler |
| project_g_spotify.md | Spotify API fixes, auth, never-revert list |
| audit_findings.md | VPS audit 2026-05-29, fixes applied, open items |
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
This document is mirrored to a public GitHub repo. It must never contain real secret values. SPOTIFY_CLIENT_SECRET, SPOTIFY_CLIENT_ID, ADZUNA_APP_ID, ADZUNA_APP_KEY, SECRET_KEY, and DASHBOARD_PASSWORD must always appear as `<see /opt/corpus/.env — never commit>`. If any update would write an actual secret value into this document, use the placeholder instead and do not deviate from this rule.

**The public URL main chat reads:**
`https://raw.githubusercontent.com/byarscrowe-dev/corpus-handoff/main/CORPUS_HANDOFF_FULL.md`

**What to update:**
- Git restore points table — add new commit hash and description
- Known issues — mark resolved issues `[ RESOLVED — commit ]`, add new ones as `[ OPEN ]`
- Projects A–K table — if any project changed status
- Immediate next actions — reflect what should happen next session
- Decision log — record any architectural decisions made and why

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
| 2026-06-01 | Updated main-chat session start protocol — stale-doc check now uses commit age (>1 week) instead of immediate verify | docs-only |

---

## KNOWN ISSUES

| # | Issue | Status | Notes |
|---|-------|--------|-------|
| 1 | Play buttons exist but URIs empty until cache populated | `[ OPEN ]` | Click `[ REFRESH PLAYLISTS ]` on Sync tab first — populates health cache with URI field. After that `[ PLAY ]` works. |
| 2 | Analytics tab shows "NO DATA" | `[ OPEN ]` | Click `[ REFRESH ]` on Analytics tab — runs 2809-song background analysis, ~1-2 min |
| 3 | Job count low on dashboard | `[ OPEN ]` | Adzuna duplicate credentials fixed. Wait for 8 AM CT daily scheduler run. Never use browser Refresh Jobs. |
| 4 | Audio feature bars show 0 | `[ KNOWN LIMITATION ]` | Spotify audio features API returns 403 in dev mode since Nov 2024. Not fixable without Extended Quota approval |
| 5 | No swap memory on VPS | `[ OPEN — non-urgent ]` | 458MB RAM, 0 swap. Add 1GB swap if scraping causes OOM |
| 6 | Spotify SDK player shows NOT CONNECTED or CONNECT FAILED | `[ OPEN ]` | Redirect URI fix confirmed working — `https://corpusbc.duckdns.org/spotify/login` shows Spotify connected banner. Root cause: server `.env` had old `http://127.0.0.1:5000/callback` uncommented while HTTPS line was commented out. Fixed on server via sed; local `.env` synced. Code (22b4a52) now reads `SPOTIFY_REDIRECT_URI` with no fallback. Re-authenticate if token expires: visit `https://corpusbc.duckdns.org/spotify/login`. |
| 7 | No authentication on any route | `[ RESOLVED — nginx Basic Auth 2026-06-01 ]` | All routes including API endpoints now require HTTP Basic Auth at nginx level before reaching Python. |
| 8 | `/api/spotify/token` publicly exposed | `[ RESOLVED — nginx Basic Auth 2026-06-01 ]` | Covered by nginx Basic Auth — same as issue 7. |
| 9 | No HTTPS — all traffic in cleartext | `[ RESOLVED — Let's Encrypt 2026-06-01 ]` | TLS cert via certbot for `corpusbc.duckdns.org`. Auto-renews via certbot.timer. Expires 2026-08-30. |
| 10 | `/refresh-jobs` unauthenticated GET triggers scraping | `[ RESOLVED — nginx Basic Auth 2026-06-01 ]` | Covered by nginx Basic Auth — same as issue 7. |
| 11 | `source_url` XSS — `javascript:` scheme possible in job URLs | `[ RESOLVED — 6bf3e57 ]` | `jobs.html` now only renders `<a href>` when `source_url` starts with `http://` or `https://`. Any other scheme is silently dropped. |
| 12 | Credentials (Adzuna + Spotify) committed to public corpus-handoff repo | `[ RESOLVED — 2026-06-02 ]` | GitGuardian flagged the exposure. Spotify secret rotated (2026-06-01), Adzuna key rotated (2026-06-02). corpus-handoff history wiped to single clean commit (f90319e). All handoff docs scrubbed to placeholders. Three-tier credential rule added to protocol. |

---

## DECISION LOG
*Why things are built the way they are. Prevents re-litigating solved problems.*

| Decision | Rationale | Date |
|----------|-----------|------|
| JobSpy installed with `--no-deps` | numpy/tls-client conflict breaks standard install. This is the only working approach | early |
| Job refresh runs on scheduler, never via browser | Browser requests 504 after 120s — scraping takes 3-5 min. Scheduler runs detached | early |
| Spotify sync delayed 24h after restart | Prevents hammering Spotify API on every service restart | 2026-05-29 |
| Async fire-and-poll for all Spotify refresh operations | Spotify API calls too slow to block page render. Background threads + polling pattern | 2026-05-29 |
| Single gunicorn worker | Background threads (Spotify refresh, analytics) share the worker. Adding workers would break shared state | 2026-05-29 |
| `dashboard.db` only — `corpus.db` deleted | `corpus.db` was a stray empty file created accidentally. `dashboard.db` is the correct database | 2026-05-29 |
| Split playlist functionality removed entirely | Byars will never use it. Removed from UI, routes, and agent | 2026-05-30 |
| PROJECTS registry dict in app.py | Single source of truth for all project identities (full_name, system_name, acronym, color, status, endpoint, css_key). Templates loop it — no labels hardcoded anywhere. Prevents drift between sidebar, cards, and breadcrumbs. | 2026-05-30 |
| Handoff doc moved from Google Drive to public GitHub repo | Drive MCP has no update tool — every write creates a new file with a new ID, causing drift. GitHub push is atomic, versioned, and fetchable by main chat via raw URL without any MCP dependency. | 2026-05-30 |
| HTTPS + nginx Basic Auth over per-route Flask auth | Single `auth_basic` block in nginx protects every route including the previously public `/api/spotify/token`. Simpler than adding `@login_required` decorators to every Flask route. Auth happens at nginx before requests reach Python at all. | 2026-06-01 |
| DuckDNS subdomain over paid domain | Let's Encrypt cannot issue certs for bare IP addresses. DuckDNS gives a free subdomain (`corpusbc.duckdns.org`) with instant setup — just a Google sign-in. No expiry as long as DuckDNS account is accessed monthly. | 2026-06-01 |
| Active nginx config is `/etc/nginx/sites-available/corpus` not `default` | On this server, `sites-enabled/corpus` is the active symlink. The `default` file exists but is not loaded. Future nginx changes must edit `corpus` specifically — editing `default` will have no effect. | 2026-06-01 |
| Three-tier credential model: secrets in `.env` only, private context in CC memory files (never pushed), shareable context in public handoff doc — real secret values must never appear in the handoff doc | GitGuardian flagged Adzuna + Spotify credentials committed to the public corpus-handoff repo. The handoff doc is mirrored publicly for main-chat cold-start and must never hold real values. Actual credentials live only in `/opt/corpus/.env` (server) and local `.env` (gitignored). CC memory files and memory/ are private and can hold sensitive operational context but are never pushed to the public repo. | 2026-06-02 |

---

## IMMEDIATE NEXT ACTIONS

1. Verify `[ PLAY ]` buttons work on Spotify Orbit (Spotify login confirmed working)
2. Verify Analytics tab functions (click `[ REFRESH ]` if NO DATA shown)
3. Update `deploy.ps1` to print `https://corpusbc.duckdns.org` at the end instead of the old IP (cosmetic)

---

## PARKING LOT — IDEAS NOT YET STARTED

**Project B — Prediction Market Arbitrage Bot**
Monitor Polymarket and Kalshi simultaneously for arbitrage opportunities — same event priced differently across platforms. Needs real-time price feeds, arbitrage detection layer, eventually trade execution. Open questions: event name mapping across platforms, fee structure minimum spread, rate limits.

**Project C — Stock Market Modeling & Trading Agents**
Multi-agent system, multiple bots running in parallel each with a different strategy. Phase 1 is pure simulation only — no real money. Specific bots pitched:
- Political Disclosure Tracker: shadows Trump and prominent senators' disclosed trades via STOCK Act filings. Data via QuiverQuant or Capitol Trades. Model what mirroring each trade with N-day lag would have returned.
- Other bot types TBD — momentum, earnings catalyst, sector rotation, insider filing trackers, sentiment-based, macro signal.
- Long-term goal: adaptive/learning behavior — bots adjust weights based on what's working.
- UI: multiple sub-pages similar to RE_A's tab structure. Master dashboard comparing all bots side by side.

**Project E — Automated Media Clipper**
Monitor YouTube channels for new uploads, identify highlight moments, clip and process into short-form vertical video, auto-post to TikTok and Instagram. Open research needed: clip identification method, YouTube ToS on reclipping, copyright avoidance for monetization, TikTok/Instagram posting APIs.

**Project K — Property Locator System**
Multi-bot system for undervalued DFW properties. First bot: underpriced duplexes evaluated on price-to-rent ratio and cap rate. Future bots: small multifamily, commercial land, distressed, off-market. Data sources: Zillow API, Redfin, DFW MLS, ATTOM Data, CoStar.

**Network graph for RE_A**
Design brief specifically calls this out. LinkedIn connections as a living graph — companies as nodes, connections as edges, colored by industry/relationship strength. Data already exists.

**Heraldic marks**
Each project eventually gets its own emblem — chosen, not designed. Long-term identity system mentioned in design brief.

**Halftone rendering tool**
For Project D — render uploaded images in dot-matrix style matching CORPUS visual language.

---

## GIT RESTORE POINTS

| Commit | Description | Date |
|--------|-------------|------|
| 5ba1b22 | v0.1: Initial snapshot | early |
| 796694d | v0.2.1: Navigation redesign, homepage, login removed | early |
| ac28571 | v0.3: Title concat fix, salary display, CORPUS breadcrumb | early |
| ef71532 | v0.4: Adzuna expanded to 27 queries | early |
| 1e39208 | v0.5: CRE-specific search terms | early |
| 9800510 | v0.6: Column sorting, compact date format | early |
| a425590 | v0.7: LinkedIn network matching via CSV | early |
| aa78d74 | v0.8.2: LinkedIn CSV preamble fix — 429 connections loading | early |
| 9b72acf | v0.9: Project K added, Spotify deployment scripts | early |
| 0ac05bc | v1.0: Project G — full Spotify agent | early |
| 8fe430c | Fix Spotify API compat — 403s, deprecated endpoints, track key | early |
| 2660556 | Perf: get_playlist_health uses limit=1 for speed | 2026-05-29 |
| 822ecbb | Hotfix: auth_manager= fix, prevents 401 on token expiry | 2026-05-29 |
| b0ddfa3 | Hotfix: defer Spotify sync scheduler 24h after startup | 2026-05-29 |
| 113e253 | Async playlist refresh — fire-and-poll pattern | 2026-05-29 |
| 70d27bb | Eliminate Spotify network call on Project G page render | 2026-05-29 |
| 6222533 | Health cache endpoint — instant load | 2026-05-29 |
| 44b6f36 | Playlist ID cache + 45s timeout | 2026-05-29 |
| bb51f60 | Fix deploy.ps1: safe.directory before git pull | 2026-05-29 |
| 730e2fe | Rate limit hotfix: retries=0, Spotify sync 24h delay | 2026-05-29 |
| cc80bc7 | CronTrigger 8AM CT, timezone display in CT | 2026-05-29 |
| 460b36a | Play buttons, analytics cache, Sp_O sidebar, Adzuna fix | 2026-05-30 |
| b000ed1 | Remove split playlist functionality from UI, routes, and agent | 2026-05-30 |
| 37af94c | docs: add comprehensive project handoff document | 2026-05-30 |
| 3e24d09 | feat: project registry + Spotify SDK fixes (logging, connect() promise, auto-expand) | 2026-05-30 |
| a90b329 | docs: switch handoff to public GitHub repo, design_brief to memory | 2026-05-30 |
| 6bf3e57 | security: sanitize job source_url XSS; HTTPS + Basic Auth fully deployed | 2026-06-01 |
| 22b4a52 | fix: remove hardcoded Spotify redirect_uri fallback — read from env only | 2026-06-01 |

---

*This document is the single source of truth for CORPUS. Claude Code pushes it to the public corpus-handoff repo after every meaningful commit. Main chat fetches it via raw GitHub URL at session start. Byars does not manually maintain it — the system does.*
