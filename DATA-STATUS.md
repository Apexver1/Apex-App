# DATA-STATUS.md — Real vs Dummy data tracker

**Updated:** 2026-04-17 (after Session 3 KenPom backend ingestion)
**Updated by:** Claude at end of every session

This doc tracks the **truth** of what data is real vs dummy across the app, per screen and per school. Source of truth for Claude across sessions. Not visible to end users — visual indicators (the "Sample" badges shipped in 5f) were removed in Session 1; gaps are now tracked here only.

## Legend
- ✅ **REAL** — pulls from a live data source (Supabase / API)
- 🟡 **DUMMY** — hardcoded or generated placeholder, designed to look identical to real
- 🟠 **PARTIAL** — some fields real, some dummy
- ❌ **MISSING** — currently broken or empty for some/all schools
- 🔴 **BROKEN** — looks like real data but isn't (worse than dummy because it lies)

---

## By screen

### Login / Splash
- **Status:** ✅ REAL
- Auto-login wired to `apex@test.com` / `ApexTest123` (skips login form entirely — works as designed)
- Splash + "Build a Champion" button always renders

### Drawer / Navigation
- **Status:** ✅ REAL
- Coach name + initials per school: ✅ REAL (pulled from `schools.head_coach_name`, fallback to school initials)
- Drawer badges (Home reminders, Recruiting count, CRM overdue): 🔴 BROKEN — globally counted, not school-scoped (see B5)
- iPhone scroll: ✅ FIXED in Session 5f
- "Logged in as · Coach Stewart · HC" indicator: ✅ NEW in Session 1 (Option C — separate row below school-coach card)
- **Gap (B3):** Howard `head_coach_name` is NULL — drawer shows blank name. Fix: data backfill OR more defensive fallback.

### Home — Smart Suggestions
- **Status:** ✅ REAL
- Computed from prospects + CRM logs

### Home — Today's Brief tiles
- **Status:** 🔴 BROKEN (see BACKLOG → "School-scope the global queries")
- 4 of 14 `loadData()` queries lack `school_id` filtering
- Scholarships / NIL requests / CRM follow-ups / Reminders all show globally-identical numbers across schools

### Home — Next Up game
- **Status:** 🟡 DUMMY (per-school generator shipped Session 2)
- `s2nextGame(schoolId)` — curated arena + conf opponent for 25 schools, generic fallback rest

### Home — Roster Pulse
- **Status:** 🟠 PARTIAL → 🔴 BROKEN for non-Oregon (B4)
- On roster count: ✅ REAL
- Avg PPG (starters): 🔴 BROKEN non-Oregon (data gap — `depth_chart_position` or PPG NULL)
- Depth score "B+": 🟡 DUMMY (hardcoded)
- NIL committed/cap: ✅ REAL Oregon, default for others until Session 8

### Home — Hot in the portal
- **Status:** ✅ REAL

### Home — Your shortlist
- **Status:** ✅ REAL

### Home — College basketball news
- **Status:** 🟡 DUMMY — `s2news(schoolId)` per-school generator (Session 2)

### Roster
- **Status:** ✅ REAL (5,136 of 7,580 players have ESPN photos)

### Recruiting (Portal / HS / Watchlist tabs)
- **Status:** ✅ REAL (1,192 portal + 657 HS)

### Player cards / detail modal
- **Status:** 🟠 PARTIAL
- Photos + logos ✅
- **KenPom rank section on player card:** Data now exists in backend (Session 3), **not yet surfaced in UI** — Session 4's job

### Apex CRM
- **Status:** ✅ REAL (user-scoped; school-scoping backlogged)

### Scheduled visits
- **Status:** 🟡 DUMMY — `s2visits(schoolId)` per-school generator (Session 2)

### NIL Budget
- **Status:** 🟠 PARTIAL (Oregon ✅, others default — Session 8 fix)

### Roster Builder
- **Status:** ✅ REAL

### Smart Filter
- **Status:** 🟠 PARTIAL
- Source / position / class filters ✅
- **KenPom "bottom-50" / "top-X" filtering:** backend data ready (Session 3), surfacing deferred to Session 4 + Apex Picks strip in Session 7

### Apex Scout
- **Status:** ✅ REAL (Scout 1.2 tool-use refactor shipped in `apex-intel` Session 3 cleanup)
- Polish pass: pending Session 6

### Apex Patrons
- **Status:** 🟡 DUMMY (Oregon-only — Session 9 fixes for P5)

### Apex Ops
- **Status:** 🟡 DUMMY — `s2opsRecent` / `s2opsDecks` per-school generators (Session 2)

### School picker
- **Status:** ✅ REAL with B6 search bug ("st." doesn't match Saint Mary's)

### NIL Vault (NEW — does not yet exist)
- **Status:** ❌ NOT BUILT (Session 9 stub)

---

## By data source

### Supabase tables (22 total)
| Table | Coverage | School-scoped in loadData()? | Notes |
|---|---|---|---|
| schools | ✅ 1,518 rows; 365 D1 with conference; **365 with KenPom data (Session 3)** | N/A (universe) | 8 new kenpom_* columns |
| school_rosters | ✅ 7,580 players | ✅ via my_roster query | 5,136 with photos |
| my_roster | ✅ Oregon (78) | ✅ Yes | other schools materialized via RPC |
| prospect_shortlist | ✅ 50 Oregon prospects | ✅ Yes | school-agnostic via `school_id` |
| prospect_stats | ✅ | ❌ No (`is_seed_data` filter only) | |
| prospect_scores | ✅ | ❌ No | |
| prospect_film | partial | — | |
| prospect_family | partial | ❌ No | |
| crm_contacts | partial | — | |
| crm_log | ✅ | 🔴 NO — bug B5 | |
| crm_added | ✅ | user-scoped not school-scoped | already in BACKLOG |
| watchlist | ✅ | user-scoped not school-scoped | already in BACKLOG |
| contact_reminders | ✅ | 🔴 NO — bug B5 | |
| team_strategy | partial | ❌ No | |
| nil_budget | 🟠 | ✅ Yes | Oregon only — fix Session 8 |
| nil_budget_requests | ✅ | 🔴 NO — bug B5 | |
| scholarships | ✅ | 🔴 NO — bug B5 | |
| staff | partial | — | |
| audit_log | ✅ | — | |
| search_history | ✅ | — | |
| source_tags | ✅ | — | |
| agent_memory | ✅ | — | |

### External data sources
| Source | Status | Coverage |
|---|---|---|
| ESPN (photos + roster + logos) | ✅ Active | 5,136 photos, 365 logos |
| BallDontLie | ✅ Active (NCAA) | not currently surfaced |
| CBBD | ✅ Active | not currently surfaced |
| **KenPom** | ✅ **Active — wired Session 3 (2026-04-17)** | **365 of 365 D1 schools synced** |
| Torvik | ❌ Not yet wired | Session 5 |
| 247Sports (HS + portal) | ✅ Phase 4 ingested | 657 HS + 1,192 portal |
| NCAA-API | ✅ Verified | not currently surfaced |

---

## Audit-discovered bug list (Session 1)

| ID | Severity | Surface | Description | Fix path |
|---|---|---|---|---|
| B1 | High | Home → Next up | "Oregon vs Arizona" hardcoded on every school | ✅ RESOLVED Session 2 |
| B2 | High | Visits | Jalen Washington + Oregon facilities show on every school | ✅ RESOLVED Session 2 |
| B3 | Medium | Drawer | Howard shows blank head coach name (NULL in DB) | BACKLOG |
| B4 | Medium | Home → Roster pulse | Avg PPG = 0.0 on Duke/Kansas/Howard | BACKLOG |
| B5 | High (architectural) | Home → Today's brief | 4 of 14 loadData queries lack school_id filter | BACKLOG (named project) |
| B6 | Low | School picker | "st." doesn't match Saint Mary's | BACKLOG |
| B7 | Cosmetic | Drawer | Avatar bg color not school-themed | BACKLOG |

---

## Session log (append at end of each session)

### Session 0 — Phase 5f shipped (2026-04-16)
- iPhone drawer scrolls; Sample badges added; DATA-STATUS.md created; 14-day plan locked

### Session 1 — Audit + badge removal + current-user indicator (2026-04-17)
- 5 Sample badges + 4 demo-note paragraphs removed
- Current-user indicator added to drawer (Option C — "Logged in as · Coach Stewart · HC")
- Audit across Oregon, Duke, Kansas, Howard; 7 bugs catalogued (B1–B7)
- Session shipped as `index-s1.html` → `index.html` on main

### Session 2 — Universal dummy-data fallback pattern (2026-04-18)
- Per-school generators: `s2nextGame`, `s2news`, `s2staff`, `s2visits`, `s2opsRecent`, `s2opsDecks`
- 25 curated schools + generic fallback for remaining ~340
- B1 + B2 resolved
- Oregon pilot demo byte-identical (verified diff)

### Session 3 — KenPom backend ingestion (2026-04-17, overnight)
- **Backend repo cleanup (apex-intel):** 3 commits to main — .gitignore tightened (`.env.backup` security risk neutralized), Scout 1.2 tool-use refactor committed, Bart Torvik ingestion script committed
- **Supabase SUPABASE_SERVICE_ROLE_KEY rotated** after accidental screenshot exposure (old key disabled, new `apex_backend` key issued)
- **Schema migration:** 8 new columns on `schools` — `kenpom_rank`, `kenpom_adj_em`, `kenpom_adj_o`, `kenpom_adj_d`, `kenpom_tempo`, `kenpom_season`, `kenpom_team_id`, `kenpom_last_sync`
- **`scripts/kenpom_sync.py` shipped** in `apex-intel` — 3-pass resolver: hand-coded KenPom→Supabase overrides (62 entries) → normalize-exact (St. → State) → fuzzy-fallback (`fuzz.ratio`, 94/4 threshold+margin; 0 fuzzy hits in final run)
- **Sync result:** 365 of 365 D1 schools matched. 62 override, 303 normalize-exact, 0 fuzzy, 0 update errors. DataThrough 2026-04-06.
- **Spot-check:** Oregon rank 101 / AdjEM 7.08 (matches API probe baseline); all 10 edge-case schools (Miami vs Miami OH, USC vs South Carolina Upstate, Tennessee vs UT Martin, etc.) landed on correct rows; distinct ranks = 365 (no wrong-row duplicates)
- **Frontend NOT yet surfacing KenPom** — that's Session 4's job
