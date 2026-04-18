# DATA-STATUS.md — Real vs Dummy data tracker

**Updated:** 2026-04-17 (after Session 5 Torvik — backend + frontend + Smart Filter surfacing)
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
- **Status:** 🟠 PARTIAL (real KenPom wired Session 4a, curated arena + opponent still generated)
- `s2nextGame(schoolId)` — curated arena + conf opponent for 25 schools, generic fallback rest
- **KenPom ranks (home + away):** ✅ REAL — Session 4a wired via `kenpomFor()` helper; all 365 D1 schools read live KenPom.

### Home — Roster Pulse
- **Status:** 🟠 PARTIAL
- On roster count: ✅ REAL
- Avg PPG (starters): 🔴 BROKEN non-Oregon (data gap — `depth_chart_position` or PPG NULL, see B4)
- Depth score "B+": 🟡 DUMMY (hardcoded)
- **KenPom AdjEM chip (4th tile, Session 4a):** ✅ REAL — shows `{+/-}X.XX AdjEM` label + `#rank` value for all 365 D1 schools.
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
- **KenPom on recruiting cards (Session 4b):** ✅ REAL on both card renderers (line 2499 + line 3470). Chips show "Wake Forest · ACC · #80 · +10.58 AdjEM"; graceful-hide for HS/international/non-D1 prospects (Variant C).

### Player cards / detail modal
- **Status:** 🟠 PARTIAL
- Photos + logos ✅
- **KenPom rank section on player modal (Session 4b):** ✅ REAL — injected into `mdl-sub` subtitle row; respects `isPr` for recruiting-vs-roster context.

### Apex CRM
- **Status:** ✅ REAL (user-scoped; school-scoping backlogged)

### Scheduled visits
- **Status:** 🟡 DUMMY — `s2visits(schoolId)` per-school generator (Session 2)

### NIL Budget
- **Status:** 🟠 PARTIAL (Oregon ✅, others default — Session 8 fix)

### Roster Builder
- **Status:** ✅ REAL

### Smart Filter
- **Status:** 🟠 PARTIAL (Torvik wired Session 5c, KenPom + Apex Picks preset strip deferred to Session 7)
- Source / position / class / priority tier / On3 rank filters: ✅ REAL
- **Torvik tier filter (Session 5c):** ✅ REAL — 4 buckets (All / Top 50 / #51-150 / #151+) filtering prospects by `torvikFor(current_school).rank`. Gracefully excludes HS/international/non-D1 prospects when any tier filter is active.
- **KenPom "bottom-50" / "top-X" filtering:** backend data ready (Session 3), helper ready (Session 4a). Deferred to Apex Picks preset strip in Session 7 (composite KenPom+Torvik presets).

### Apex Scout
- **Status:** ✅ REAL (Scout 1.2 tool-use refactor shipped in `apex-intel` Session 3 cleanup)
- **KenPom on scout cards:** Data ready + helper ready. **Not yet surfaced** — deferred to Session 6 Scout polish pass.
- **Torvik on scout cards:** Data ready + helpers (`torvikFor`, `torvikById`) ready. **Not yet surfaced** — deferred to Session 6.
- Polish pass: pending Session 6

### Apex Patrons
- **Status:** 🟡 DUMMY (Oregon-only — Session 9 fixes for P5)

### Apex Ops
- **Status:** 🟡 DUMMY — `s2opsRecent` / `s2opsDecks` per-school generators (Session 2)

### School picker
- **Status:** ✅ REAL with B6 search bug ("st." doesn't match Saint Mary's — though "saint" does work, so bug may be narrower than originally specced; verify in future session)

### NIL Vault (NEW — does not yet exist)
- **Status:** ❌ NOT BUILT (Session 9 stub)

---

## By data source

### Supabase tables (22 total)
| Table | Coverage | School-scoped in loadData()? | Notes |
|---|---|---|---|
| schools | ✅ 1,518 rows; 365 D1 with conference; **365 with KenPom + 365 with Torvik data** | N/A (universe) | 8 kenpom_* + 9 torvik_* columns; SELECT expanded Sessions 4a + 5b |
| school_rosters | ✅ 7,580 players | ✅ via my_roster query | 5,136 with photos |
| my_roster | ✅ Oregon (78) | ✅ Yes | other schools materialized via RPC |
| prospect_shortlist | ⚠️ 100 rows actual (not 1,849 as previously documented — see B8) | ✅ Yes | school-agnostic via `school_id` |
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
| **KenPom** | ✅ **Wired Session 3 backend + Session 4a/4b frontend surfacing (5 of 6 planned surfaces live; Scout deferred to Session 6)** | 365 of 365 D1 schools |
| **Torvik** | ✅ **Wired Session 5 backend + 5b frontend helpers + 5c Smart Filter surfacing** | 365 of 365 D1 schools (6 of 9 columns populated: rank, adj_oe, adj_de, barthag, season, last_sync; eFG/eFG-D/TOV deferred per S5-minor-1) |
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
| B6 | Low | School picker | "st." doesn't match Saint Mary's ("saint" does work — may be narrower than specced) | BACKLOG |
| B7 | Cosmetic | Drawer | Avatar bg color not school-themed | BACKLOG |
| B8 | Medium | Recruiting | `prospect_shortlist` is 100 rows not 1,849 as documented — all school_id=Oregon regardless of current_school text | BACKLOG (named project — Prospect pipeline reconciliation) |

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
- Backend repo cleanup (apex-intel): 3 commits to main — .gitignore tightened, Scout 1.2 tool-use refactor committed, Bart Torvik PLAYER-level ingestion script committed
- Supabase SUPABASE_SERVICE_ROLE_KEY rotated after accidental screenshot exposure
- Schema migration: 8 new `kenpom_*` columns on schools
- `scripts/kenpom_sync.py` shipped — 3-pass resolver (62 overrides, 303 normalize-exact, 0 fuzzy, 0 errors, 365/365 matches)

### Session 4a — KenPom frontend, Home surfaces (2026-04-17)
- Commit `d010c6c` on Apex-App/main — 7 patches to index.html
- Schools SELECT expanded, `schoolKenPomMap` built, `window.kenpomFor(name)` helper shipped
- Next Up + Roster Pulse wired to live KenPom; Oregon #101/+7.08, Duke #3/+37.37, Howard #196/-2.64

### Session 4b — KenPom frontend, recruiting cards + modal (2026-04-17)
- Commit `c7f4ac7` — KenPom chips on BOTH prospect card renderers + player modal
- Helpers `window.kenpomById(uuid)` + `window.kenpomChipHTML(p, isPr)` shipped
- Scout cards deferred to Session 6 (showpiece polish session)
- Bug B8 documented (prospect_shortlist 100 rows vs 1,849 claimed)

### Session 5 — Torvik end-to-end (2026-04-17)
- **5-A backend** (commit `c4e79f1` on apex-intel): 9 new `torvik_*` columns on `schools`; `scripts/torvik_team_sync.py` shipped mirroring kenpom_sync.py 3-pass resolver; source `https://barttorvik.com/2026_team_results.json`; 365/365 D1 schools synced (54 override, 252 exact, 59 normalize, 0 fuzzy, 0 errors); 6 of 9 columns populated (rank, adj_oe, adj_de, barthag, season, last_sync); eFG/eFG-D/TOV NULL pending index-mapping verification (S5-minor-1)
- **5-B frontend data layer** (commit `5b54cb6` on Apex-App): 4 patches to index.html; schools SELECT expanded; `schoolTorvikMap` + `schoolTorvikById` globals + populate forEach extended; `window.torvikFor(name)` + `window.torvikById(uuid)` helpers shipped; Safari smoke-tested (Oregon #91 / barthag 0.7068, Duke #3 / barthag 0.9768 verified end-to-end); zero KenPom regression
- **5-C Smart Filter surfacing** (commit `6efe921` on Apex-App): 7 patches to index.html; new "Torvik tier" section with 4 buckets (All / Top 50 / #51-150 / #151+); `fTTier` state var + reset paths in `navTo`/`clearSF` + `uf()` dispatch + `portal.filter()` apply logic + UI render between Rankings and Financials + activeCount increment; Safari smoke-tested (Top 50 → 30 matches, all from verified top-50 Torvik schools)
- Mid-session course-corrections: (1) Torvik `trank.php?csv=1` endpoint returned HTML not CSV — switched to `/2026_team_results.json` JSON-array endpoint; (2) column index mapping verified against Duke + Arizona + Oregon cross-reference before committing TT_COL table; (3) initial sync run had 78 misses due to normalize_name bug (`.replace(' St.', ' State')` fix); second run had 6 remaining misses cleared via override-table additions; third run hit 365/365; (4) Python heredoc multi-line-string concatenation bug in P3 patch → single-line string fix; (5) P1 anchor colon-vs-equals bug (`fTier:"ALL"` vs `fTier="ALL"` — var-chain uses `=`) → fixed in rerun
- Session time: ~1h45min (top of 90-120min budget — detour time absorbed in buffer)
