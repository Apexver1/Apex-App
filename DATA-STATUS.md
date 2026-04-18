# DATA-STATUS.md — Real vs Dummy data tracker

**Updated:** 2026-04-18 (after Session 6 — Scout v2 hero + metric tooltips)
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
- **Status:** 🟠 PARTIAL (deferred to Session 7 Commit 2b)
- Coach name per school: ✅ REAL (pulled from `schools.head_coach_name`, fallback to school initials)
- Drawer badges (Home reminders, Recruiting count, CRM overdue): 🔴 BROKEN — globally counted, not school-scoped (see B5)
- iPhone scroll: ✅ FIXED in Session 5f
- "Logged in as · Coach Stewart · HC" indicator: ✅ in drawer (Session 1 — Option C row); **role label rewrite (HC → Assistant Coach, Title Case, muted gray) deferred to Session 7 Commit 2b**
- **Drawer logo swap to 72px centered school logo: DEFERRED to Session 7 Commit 2b** — blocked in Session 6 by grep recon returning zero hits on dynamic text ("Dana Altman", "Coach Stewart", "Head coach", "Logged in as" are all built via JS string concatenation from variables). Next session starts with Safari DevTools inspection of the drawer header block to find the actual render function before patching.
- **Gap (B3):** Howard `head_coach_name` is NULL — greeting reads "Good morning, Coach." Fix: data backfill OR more defensive fallback copy. Probably fold into Session 7 drawer work.

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
- **Metric tooltip:** ✅ NEW Session 6 — KenPom chips on Next Up now tappable, opens dark popover with plain-English "Team strength" + AdjEM explanation + values inline. Uses `window.apxShowInfo()` helper.

### Home — Roster Pulse
- **Status:** 🟠 PARTIAL (metric-translation shipped Session 6)
- On roster count: ✅ REAL
- Avg PPG (starters): 🔴 BROKEN non-Oregon (data gap — `depth_chart_position` or PPG NULL, see B4)
- Depth score "B+": 🟡 DUMMY (hardcoded)
- **4th tile (Session 6 Commit 2a):** ✅ NEW — label rewritten from `+7.08 AdjEM` to plain-English `Team strength ⓘ`. Value column still shows `#101` (KenPom rank). Tapping ⓘ opens popover with full metric explanation + Oregon's live values.
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
- **KenPom on recruiting cards (Session 4b):** ✅ REAL on both card renderers. Chips show "Wake Forest · ACC · #80 · +10.58 AdjEM"; graceful-hide for HS/international/non-D1 prospects.
- **Metric tooltip (Session 6 Commit 2a):** ✅ NEW — every KenPom chip across BOTH card renderers now has a tappable `ⓘ` icon + popover. Same treatment as Home chips.

### Player cards / detail modal
- **Status:** 🟠 PARTIAL
- Photos + logos ✅
- **KenPom rank section (Session 4b):** ✅ REAL — injected into `mdl-sub` subtitle row.
- **Metric tooltip on subtitle chip (Session 6 Commit 2a):** ✅ NEW — the `#80 · +10.58 AdjEM` subtitle chip is now tappable; popover shows "Team strength" explanation.
- **BART TORVIK ADVANCED stat-box AdjOE label (Session 6 Commit 2a):** ✅ NEW — label rewritten from `AdjOE` to plain-English `Offense ⓘ`. Tapping ⓘ opens popover with "Offense (AdjOE player-level)" explanation specific to Bart Torvik's player-level offensive rating.

### Apex CRM
- **Status:** ✅ REAL (user-scoped; school-scoping backlogged)

### Scheduled visits
- **Status:** 🟡 DUMMY — `s2visits(schoolId)` per-school generator (Session 2)

### NIL Budget
- **Status:** 🟠 PARTIAL (Oregon ✅, others default — Session 8 fix)

### Roster Builder
- **Status:** ✅ REAL

### Smart Filter
- **Status:** 🟠 PARTIAL (Torvik tier wired Session 5c)
- Source / position / class / priority tier / On3 rank filters: ✅ REAL
- **Torvik tier filter (Session 5c):** ✅ REAL — 4 buckets (All / Top 50 / #51-150 / #151+).
- **KenPom "bottom-50" / "top-X" filtering:** backend + helper ready. Deferred to Apex Picks preset strip (Session 7-ish).
- **No tooltips added in Session 6** — Torvik tier section is plain-English already ("Schools current Torvik rank" subhead).

### Apex Scout — hero (Session 6 Commit 1)
- **Status:** ✅ REAL (redesigned Session 6 Commit 1)
- **Hero redesign shipped (Direction B — premium):** serif "Build your roster." headline, dynamic "fit [School]" subhead, pill-shaped input with inline circular navy mic button, 2×2 preset grid per section (4 smart Torvik/KenPom-driven + 4 static evergreen), below-fold "How to use Scout" instructions card.
- Engine toggle (`Rules · free / Apex AI · $0.01/q`) killed entirely — UI removed.
- **aiMode JS state:** still defaults to rules internally (2c LLM-default attempted in Session 6 but reverted — see BACKLOG). Means current default engine is still rules regex-parser which returns mediocre matches on semantic queries.
- Smart preset copy interpolates live values: Tempo match shows school's Torvik tempo, Rebuild targets shows school's Torvik rank, Defensive upgrades shows school's AdjDE.

### Apex Scout — results
- **Status:** ✅ REAL (Scout 1.2 tool-use refactor shipped in `apex-intel` Session 3 cleanup)
- LLM mode (Scout 1.2) with tool_calls + narrative answer renders correctly when engaged.
- Rules mode (legacy regex parser) is the current default; returns loosely-matched results on semantic queries (e.g., "rim protectors" returns non-rim-protectors).
- **KenPom chips on scout result cards (Session 6 Commit 2a):** ✅ Now tappable with plain-English popover, via `kenpomChipHTML` rewrite propagating to both LLM mode (~line 3271) and rules mode (~line 3460).
- **Scout card polish pass from original Session 6 plan:** effectively shipped via the Commit 1 hero redesign — cards unchanged but the overall screen feels like the "showpiece" Sammy called out.

### Apex Patrons
- **Status:** 🟡 DUMMY (Oregon-only — Session 9 fixes for P5)

### Apex Ops
- **Status:** 🟡 DUMMY — `s2opsRecent` / `s2opsDecks` per-school generators (Session 2)

### School picker
- **Status:** ✅ REAL with B6 search bug ("st." doesn't match Saint Mary's — though "saint" does work)

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
| prospect_shortlist | ⚠️ 100 rows actual (B8) | ✅ Yes | school-agnostic via `school_id` |
| prospect_stats | ✅ | ❌ No (`is_seed_data` filter only) | Player-level Torvik AdjOE lives here |
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
| **KenPom** | ✅ Wired end-to-end through Session 4b + now with tooltips (Session 6) | 365 of 365 D1 schools |
| **Torvik** | ✅ Wired end-to-end through Session 5c (team-level); player-level lives in `prospect_stats` | 365 of 365 D1 schools (team) |
| 247Sports (HS + portal) | ✅ Phase 4 ingested | 657 HS + 1,192 portal |
| NCAA-API | ✅ Verified | not currently surfaced |

---

## Audit-discovered bug list

| ID | Severity | Surface | Description | Fix path |
|---|---|---|---|---|
| B1 | High | Home → Next up | "Oregon vs Arizona" hardcoded on every school | ✅ RESOLVED Session 2 |
| B2 | High | Visits | Jalen Washington + Oregon facilities show on every school | ✅ RESOLVED Session 2 |
| B3 | Medium | Drawer / greeting | Howard shows blank head coach name (NULL in DB) — "Good morning, Coach." | BACKLOG → Session 7 drawer work |
| B4 | Medium | Home → Roster pulse | Avg PPG = 0.0 on Duke/Kansas/Howard | BACKLOG |
| B5 | High (architectural) | Home → Today's brief | 4 of 14 loadData queries lack school_id filter | BACKLOG (named project) |
| B6 | Low | School picker | "st." doesn't match Saint Mary's | BACKLOG |
| B7 | Cosmetic | Drawer | Avatar bg color not school-themed | ✅ Being replaced Session 7 by 72px logo (decision: scrap the avatar colors entirely) |
| B8 | Medium | Recruiting | `prospect_shortlist` is 100 rows not 1,849 as documented | BACKLOG (named project) |
| B9 | Medium (Sammy feedback) | Scout results | Rules mode returns semantically-bad matches (e.g., Juke Harris returned for "rim protectors") | BACKLOG (LLM-default engine — Commit 2c attempted Session 6, reverted due to JS parse break) |

---

## Session log (append at end of each session)

### Session 0 — Phase 5f shipped (2026-04-16)
- iPhone drawer scrolls; Sample badges added; DATA-STATUS.md created; 14-day plan locked

### Session 1 — Audit + badge removal + current-user indicator (2026-04-17)
- 5 Sample badges + 4 demo-note paragraphs removed
- Current-user indicator added to drawer (Option C — "Logged in as · Coach Stewart · HC")
- Audit across Oregon, Duke, Kansas, Howard; 7 bugs catalogued (B1–B7)

### Session 2 — Universal dummy-data fallback pattern (2026-04-18)
- Per-school generators: `s2nextGame`, `s2news`, `s2staff`, `s2visits`, `s2opsRecent`, `s2opsDecks`
- 25 curated schools + generic fallback for remaining ~340
- B1 + B2 resolved

### Session 3 — KenPom backend ingestion (2026-04-17, overnight)
- Backend repo cleanup (apex-intel): 3 commits to main
- Schema migration: 8 new `kenpom_*` columns on schools
- `scripts/kenpom_sync.py` shipped — 3-pass resolver, 365/365 matches

### Session 4a — KenPom frontend, Home surfaces (2026-04-17)
- Commit `d010c6c` — schoolKenPomMap, window.kenpomFor(name) helper
- Next Up + Roster Pulse wired to live KenPom

### Session 4b — KenPom frontend, recruiting cards + modal (2026-04-17)
- Commit `c7f4ac7` — KenPom chips on BOTH prospect card renderers + player modal
- Helpers `window.kenpomById(uuid)` + `window.kenpomChipHTML(p, isPr)` shipped

### Session 5 — Torvik end-to-end (2026-04-17)
- 5-A backend: 9 torvik_* cols, `scripts/torvik_team_sync.py`, 365/365 synced
- 5-B frontend: schoolTorvikMap, window.torvikFor / window.torvikById
- 5-C Smart Filter: Torvik tier section with 4 buckets

### Session 6 — Scout v2 hero redesign + metric tooltips (2026-04-18)
- **Commit 1 (Scout v2, commit d30a3ca):** Direction B premium redesign. Killed AI Scout / Pro badge, v1.1.1 meta, Rules/Apex AI engine toggle, flat quick-queries pill strip. Shipped serif "Build your roster." headline + dynamic "fit [School]" subhead, pill input with inline circular navy mic button, 2×2 preset grids (4 smart KenPom/Torvik-driven + 4 static evergreen), below-fold "How to use Scout" instructions card. Smart presets interpolate live values: Tempo match reads `torvikFor(school).tempo`, Rebuild targets reads `torvikFor(school).rank`, Defensive upgrades reads `kenpomFor(school).adjD`. Engine toggle killed entirely — aiMode defaults to rules internally.
- **Commit 2a (metric tooltips, commit 7ee3241):** Plain-English translation across every KenPom chip site plus the player-level AdjOE stat-box. New CSS (`.apx-info`, `.apx-pop`, `.apx-chip-tap`). New JS: `window.apxMetricInfo` (6 metric definitions: team_strength, team_offense, team_defense, team_pace, team_winprob, player_offense) + `window.apxShowInfo(ev, key, extraVal)` (popover render + positioning + tap-outside-close + × button). `kenpomChipHTML` rewritten to emit tappable spans with trailing ⓘ — one rewrite propagates to Home Next Up, Recruiting (both renderers), Player modal subtitle, Scout results (LLM + rules modes). Home Roster Pulse 4th tile: label changed from `+7.08 AdjEM` to plain-English `Team strength` + inline ⓘ. Player modal BART TORVIK stat-box: `AdjOE` label changed to `Offense` + inline ⓘ.
- **Committed then reverted:** Fix patch for 1a (popover value-line copy adding "Team · " prefix) + 2c (initializing `var aiMode="llm"` to default Scout to the better engine) broke auto-login and the app wouldn't load. Surgically reverted both changes — Commit 2a tooltips preserved intact, 1a + 2c filed to BACKLOG for fresh debugging. Root cause of 2c failure: injected `var aiMode="llm";` before a `setAiMode=function...` declaration that wasn't prefixed with `var` or `window.` — the injected `var` broke the outer statement context and JS failed to parse.
- **Deferred to Session 7 Commit 2b:** drawer logo swap, role label rewrite, Howard blank-greeting fallback.
- **Key lesson:** byte-extraction-for-anchors pattern added to CLAUDE.md — for any patch anchor containing nested quotes/escapes/unicode, extract bytes from live file via `python3 .find()` + save to `/tmp/` + read back in patch script. Burned Session 5 once (fTier colon/equals) and Session 6 four times. Now a working agreement.
- Session time: ~3 hours across multiple back-and-forth cycles for recon; actual patch application once recon was solid was tight. Two clean commits shipped.
