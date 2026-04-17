# DATA-STATUS.md — Real vs Dummy data tracker

**Updated:** 2026-04-17 (after Session 4b KenPom frontend — recruiting cards + player modal)
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
- **KenPom ranks (home + away):** ✅ REAL — Session 4a wired via `kenpomFor()` helper; both Oregon path (via lazy NEXT_GAME patch) and non-Oregon path (via generator rewire) now read live KenPom.

### Home — Roster Pulse
- **Status:** 🟠 PARTIAL
- On roster count: ✅ REAL
- Avg PPG (starters): 🔴 BROKEN non-Oregon (data gap — `depth_chart_position` or PPG NULL, see B4)
- Depth score "B+": 🟡 DUMMY (hardcoded)
- **KenPom AdjEM chip (4th tile, Session 4a):** ✅ REAL — shows `{+/-}X.XX AdjEM` label + `#rank` value for all 365 D1 schools. Negative AdjEM verified rendering correctly (Howard -2.64).
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
- **Status:** ✅ REAL (see data-reality note below re: row counts)
- **KenPom chip on prospect cards (both renderers — line 2499 and line 3470):** ✅ REAL — Session 4b wired via `kenpomChipHTML(p, true)`. Variant A (current-school KenPom) verified live on Oregon Portal (Juke Harris #80 Wake Forest, Flory Bidunga #21 Kansas, Massamba Diop #66 Arizona State, John Blackwell #23 Wisconsin). Variant C (graceful hide) verified on HS prospects with non-D1 `current_school` ("Uncommitted", "AZ Compass Prep", "Eagles Landing (GA)").

### Player cards / detail modal
- **Status:** ✅ REAL (for KenPom chip)
- Photos + logos ✅
- **KenPom rank chip in mdl-sub subtitle:** ✅ REAL — Session 4b wired via `kenpomChipHTML(p, isPr)`. Verified live (Juke Harris modal showing Wake Forest #80 · +10.58 AdjEM). Honors `isPr` semantics: prospects read `p.current_school`, Oregon roster players would read CURRENT_SCHOOL_ID→schools→name path.

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
- **KenPom "bottom-50" / "top-X" filtering:** backend data ready (Session 3), helper ready (Session 4a), surfacing deferred to Apex Picks preset strip in Session 7

### Apex Scout
- **Status:** ✅ REAL (Scout 1.2 tool-use refactor shipped in `apex-intel` Session 3 cleanup)
- **KenPom on scout cards:** Data ready + helpers ready. **Deferred to Session 6** (Apex Scout polish — "the showpiece" session) per Session 4b opener decision. Tradeoff rationale: Scout has two render branches (LLM + rules-mode); Session 6 already budgeted for polish can absorb the wire cleanly.
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

## ⚠️ Data-reality note (Session 4b discovery)

This doc's table below previously claimed **657 HS + 1,192 portal = 1,849 prospects**. Session 4b recon hit live DB and found:

- **`prospect_shortlist` table has only 100 rows.**
- **All 100 rows have `source_tag = NULL`.** The frontend discriminates on `p.source === 'portal'` / `p.source === 'high_school'` (text column), NOT `source_tag`.
- **All 100 rows have `school_id` pointing at Oregon's UUID**, regardless of what `current_school` text says (data-seed artifact — `school_id` was populated generically during seed, not against the prospect's actual school).
- **`current_school` text column is a grab-bag:** real D1 names for portal ("Kansas", "Arizona"), HS names for recruits ("AZ Compass Prep", "Eagles Landing (GA)"), literals like "Uncommitted", and compound strings like "Committed - Oregon".

**Where did 1,849 come from?** Probably from a Phase 4 ingestion log that landed into a staging table that never made it into `prospect_shortlist`, OR the numbers represent aspirational counts from 247Sports / ESPN that haven't been loaded yet. **Backlog item filed** to reconcile: either ingest the missing rows, update DATA-STATUS to reflect 100, or rewrite the ingestion script.

**Operational impact:** zero — the UI renders `p.source==='portal'` filters fine against whatever's there, and the chip's graceful-hide covers junk `current_school` values. But the "it looks like a real product" bar is lower than the stated counts suggest.

---

## By data source

### Supabase tables (23 total; +1 new since last update: `scout_tool_calls`)
| Table | Coverage | School-scoped in loadData()? | Notes |
|---|---|---|---|
| schools | ✅ 1,518 rows; 365 D1 with conference; **365 with KenPom data (Session 3, surfaced Session 4a Home, Session 4b cards+modal)** | N/A (universe) | 8 kenpom_* columns |
| school_rosters | ✅ 7,580 players | ✅ via my_roster query | 5,136 with photos |
| my_roster | ✅ Oregon (78) | ✅ Yes | other schools materialized via RPC |
| prospect_shortlist | **⚠️ 100 rows (not 1,849 as previously documented)** | ✅ Yes via `school_id` + `source` | See reality note above |
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
| scout_tool_calls | ✅ (new) | — | Session 3 Scout 1.2 refactor artifact |
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
| **KenPom** | ✅ **Active — wired Session 3 backend + Session 4a Home + Session 4b prospect cards + player modal** | **365 of 365 D1 schools; 5 of 5 surfaces live (Home Next Up, Home Roster Pulse, recruiting cards both renderers, player modal). Scout cards deferred to Session 6.** |
| Torvik | ❌ Not yet wired | Session 5 |
| 247Sports (HS + portal) | ⚠️ Ingestion status in question — see data-reality note | DATA-STATUS previously stated 657 HS + 1,192 portal; live DB shows only 100 rows in prospect_shortlist |
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
| B8 | ⚠️ **NEW** Session 4b | Data pipeline | `prospect_shortlist` has 100 rows, all with NULL `source_tag` and Oregon-pointing `school_id`. DATA-STATUS claimed 1,849. Reconcile. | BACKLOG |

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
- Backend repo cleanup: 3 commits to main (.gitignore tightened, Scout 1.2 refactor, Bart Torvik ingestion script)
- SUPABASE_SERVICE_ROLE_KEY rotated after accidental screenshot exposure
- Schema migration: 8 kenpom_* columns on `schools`
- `scripts/kenpom_sync.py` shipped — 3-pass resolver (override → normalize-exact → fuzzy-fallback)
- Sync result: 365/365 D1 schools matched, 0 update errors, 0 wrong-row bugs
- Frontend NOT yet surfacing KenPom — Session 4's job

### Session 4a — KenPom frontend, Home surfaces (2026-04-17)
- Commit `d010c6c`: 7 patches, +909 bytes
- Schools SELECT expanded to include 8 kenpom_* columns at line 1339
- `schoolKenPomMap` built in `loadData()` mirroring `schoolEspnMap` pattern
- `window.kenpomFor(name)` helper added
- Oregon NEXT_GAME lazy-patched; `s2nextGame` rewired; Roster Pulse 4th AdjEM tile added
- Live verification: Oregon #101/+7.08, Duke #3/+37.37, Kansas #21/+24.15, Howard #196/-2.64 (negative renders)
- Remaining Session 4 surfaces deferred to Session 4b

### Session 4b — KenPom frontend, recruiting cards + player modal (2026-04-17)
- **Commit `c7f4ac7`** to `Apex-App/main` — 6 patches + 1 chip-size tweak, +1349 bytes (234068 → 235417)
- **`schoolKenPomById` map** added (UUID-keyed, parallel to `schoolKenPomMap`) for clean FK-based lookup
- **`window.kenpomById(uuid)` + `window.kenpomChipHTML(p, isPr)`** helpers shipped; `kenpomChipHTML` centralizes 3 variants (A current-school, B "Committed: #X [school]", C graceful hide)
- **`.kp-chip` CSS class** added — 11px muted inline chip (bumped from initial 10px after user feedback that it read too recessive next to subtitle text)
- **Chip wired into 3 surfaces:** prospect card renderer #1 at line 2499, prospect card renderer #2 at line 3470, player modal mdl-sub at line 1512
- **Live verification on apexver1.github.io:** Juke Harris modal (Wake Forest #80 · +10.58 AdjEM), Oregon Portal cards showing real KenPom chips on Wake Forest/Kansas/Arizona State/Wisconsin prospects; Duke/Kansas/Howard Home tiles regression-free; HS prospects with "Uncommitted"/"AZ Compass Prep" correctly hide chip (Variant C)
- **Variant B ("Committed: #X") unverified in live** — seed data has zero `committed_to_school_id` records populated; code-reviewed and expected to work once real data lands
- **Scout cards deferred to Session 6** per opener decision (tradeoff: 4b scope tighter without Scout's two-branch render; Session 6 owns Scout polish anyway)
- **Data-reality discovery:** DATA-STATUS's "1,849 prospects" claim is inaccurate — actual is 100 rows in `prospect_shortlist`, all with NULL source_tag, all with Oregon-pointing school_id FK. New bug B8 filed.
- **Mid-session failures:** 3 patch-script rebuilds needed (PATCH C expect=2 → expect=1; PATCH E expect=1 → expect=2 because 2 card renderers share anchor; PATCH F anchor used `\u00b7` escape form when source file has literal U+00B7 middot char). All caught by idempotency guard before any write hit disk.
