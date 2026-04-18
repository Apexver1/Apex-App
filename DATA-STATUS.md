# DATA-STATUS.md — Real vs Dummy data tracker

**Updated:** 2026-04-18 (after Session 7 — Scout LLM-default emergency fix)
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
- **Status:** 🟠 PARTIAL (Commit 2b still deferred — Session 7 scope pivoted to Scout emergency)
- Coach name per school: ✅ REAL (pulled from `schools.head_coach_name`, fallback to school initials)
- Drawer badges (Home reminders, Recruiting count, CRM overdue): 🔴 BROKEN — globally counted, not school-scoped (see B5)
- iPhone scroll: ✅ FIXED in Session 5f
- "Logged in as · Coach Stewart · HC" indicator: ✅ in drawer (Session 1 — Option C row); **role label rewrite (HC → Assistant Coach, Title Case, muted gray) STILL deferred post-Session-7**
- **Drawer logo swap to 72px centered school logo: STILL DEFERRED** — Session 7 scope pivoted to Scout LLM-default emergency. Session 8 pickup.
- **Gap (B3):** Howard `head_coach_name` is NULL — greeting reads "Good morning, Coach." STILL open.

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
- **Metric tooltip:** ✅ Session 6 — KenPom chips on Next Up tappable, opens dark popover with plain-English "Team strength" + AdjEM explanation.

### Home — Roster Pulse
- **Status:** 🟠 PARTIAL (metric-translation shipped Session 6)
- On roster count: ✅ REAL
- Avg PPG (starters): 🔴 BROKEN non-Oregon (data gap — `depth_chart_position` or PPG NULL, see B4)
- Depth score "B+": 🟡 DUMMY (hardcoded)
- **4th tile (Session 6 Commit 2a):** ✅ label rewritten from `+7.08 AdjEM` to plain-English `Team strength ⓘ`.
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
- **KenPom on recruiting cards (Session 4b):** ✅ REAL on both card renderers.
- **Metric tooltip (Session 6 Commit 2a):** ✅ every KenPom chip across BOTH card renderers has tappable `ⓘ` + popover.

### Player cards / detail modal
- **Status:** 🟠 PARTIAL
- Photos + logos ✅
- **KenPom rank section (Session 4b):** ✅ REAL — injected into `mdl-sub` subtitle row.
- **Metric tooltip on subtitle chip (Session 6 Commit 2a):** ✅ tappable popover.
- **BART TORVIK ADVANCED stat-box AdjOE label (Session 6 Commit 2a):** ✅ label rewritten to plain-English `Offense ⓘ`.
- **🔴 NEW GAP Session 7:** player cards show `ppg / rpg / apg / bpg / 3PT%` — blocks per game is near-zero for guards and misrepresents their defensive value. Should show SPG (steals) on guards instead of or alongside BPG. See BACKLOG → "Position-aware defensive stat on card display".
- **🔴 NEW GAP Session 7:** cards don't show height or class (jr/sr) — Scout queries can filter on these, but cards can't display them. Not a Scout bug, a card-display bug.

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
- **Torvik tier filter (Session 5c):** ✅ REAL — 4 buckets.
- **KenPom "bottom-50" / "top-X" filtering:** backend + helper ready. Deferred to Apex Picks preset strip.

### Apex Scout — hero (Session 6 Commit 1, refined Session 7)
- **Status:** ✅ REAL
- Hero redesign: serif "Build your roster." headline, dynamic subhead, pill-shaped input with inline circular navy mic button, 2×2 preset grids, below-fold instructions.
- Engine toggle UI killed.
- **Session 7 Commit `9c0d90e` — aiMode default flipped from `rules` to `llm`.** Default engine is now LLM (Scout 1.1.1). The real root cause was NOT `aiMode` being undefined — it was the state initializer explicitly setting `aiMode="rules"` at page load. Patch: changed `aiResults=null,aiMode="rules",aiLoading=false` to `aiResults=null,aiMode="llm",aiLoading=false`.
- **Session 7 — `aiRun` inverted:** `aiRun=function(){if(aiMode==="rules")aiSearch();else aiLLM();}` — default branch now LLM.
- **Session 7 — mic button fixed:** no longer destroys the SVG icon by setting `textContent="Listening…"`, no longer turns ugly red via `--neg`. Now uses `.listening` class with subtle navy pulse animation + tap-to-cancel via `window._apxRec`.
- Smart preset copy interpolates live values: Tempo match shows school's Torvik tempo, Rebuild targets shows school's Torvik rank, Defensive upgrades shows school's AdjDE.

### Apex Scout — results
- **Status:** ✅ REAL (LLM default, shipped Session 7 Commit 9c0d90e)
- LLM mode (Scout 1.2 tool-use) is the default. Returns real agentic reasoning + Scout Analysis panel + iteration counts ("9 TOOL CALLS · 6 ITERATIONS").
- **🟠 NEW GAP Session 7 (QA finding):** Scout's Edge Function prompt payload does NOT currently include KenPom team data, Torvik team data, or player-level AdjOE/AdjDE. Claude reasons over counting stats (ppg/rpg/apg/3pct) + apex_model score only. When asked "shooters from top-50 KenPom defenses" Claude responds "KenPom team efficiency data is not yet integrated" — which is WRONG at the data-layer level (it's in Supabase since Session 4) but CORRECT at the LLM-payload level (not being sent to Claude). See BACKLOG → "Scout Edge Function context sync" (new #1 named project post-Session-7).
- **🟠 NEW GAP Session 7 (QA finding):** NIL data in `nil_estimate` column is dummy/partial. Claude correctly self-reports "NIL valuation data is not available yet" when asked "portal transfers under $500K". Great anti-hallucination behavior, but user-visible dollar amounts on cards (`$2.6M`, `$475K`) are rendered from `valueNil(p,ps).mid` — partially derived, not real NIL data.
- Rules mode (legacy regex parser) is now non-default fallback; only fires when `aiMode==="rules"` is explicitly set.

### Apex Patrons
- **Status:** 🟡 DUMMY (Oregon-only — Session 9 fixes for P5)
- **Session 7 copy fix (commit `f68fb4f`):** header rewritten from "Donor & development" → "Donor Development". Small cosmetic polish.

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
| schools | ✅ 1,518 rows; 365 D1 with conference; **365 with KenPom + 365 with Torvik data** | N/A (universe) | 8 kenpom_* + 9 torvik_* columns; **NOT piped into Scout Edge Function prompt** — see new BACKLOG project |
| school_rosters | ✅ 7,580 players | ✅ via my_roster query | 5,136 with photos |
| my_roster | ✅ Oregon (78) | ✅ Yes | other schools materialized via RPC |
| prospect_shortlist | ⚠️ 100 rows actual (B8) | ✅ Yes | school-agnostic via `school_id` |
| prospect_stats | ✅ | ❌ No (`is_seed_data` filter only) | Player-level Torvik AdjOE lives here — **NOT piped into Scout Edge Function prompt** |
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
| **KenPom** | ✅ Wired end-to-end through Session 4b + tooltips Session 6. **NOT yet in Scout LLM payload (Session 7 finding).** | 365 of 365 D1 schools |
| **Torvik** | ✅ Wired Session 5c (team) + Session 6 tooltips. **Team data NOT yet in Scout LLM payload. Player AdjOE/AdjDE NOT yet in Scout LLM payload.** | 365 of 365 D1 schools (team) |
| 247Sports (HS + portal) | ✅ Phase 4 ingested | 657 HS + 1,192 portal |
| NCAA-API | ✅ Verified | not currently surfaced |

---

## Audit-discovered bug list

| ID | Severity | Surface | Description | Fix path |
|---|---|---|---|---|
| B1 | High | Home → Next up | "Oregon vs Arizona" hardcoded on every school | ✅ RESOLVED Session 2 |
| B2 | High | Visits | Jalen Washington + Oregon facilities show on every school | ✅ RESOLVED Session 2 |
| B3 | Medium | Drawer / greeting | Howard shows blank head coach name (NULL in DB) | STILL OPEN — Session 8 drawer work (deferred from S7 pivot) |
| B4 | Medium | Home → Roster pulse | Avg PPG = 0.0 on Duke/Kansas/Howard | BACKLOG |
| B5 | High (architectural) | Home → Today's brief | 4 of 14 loadData queries lack school_id filter | BACKLOG (named project) |
| B6 | Low | School picker | "st." doesn't match Saint Mary's | BACKLOG |
| B7 | Cosmetic | Drawer | Avatar bg color not school-themed | Moot — being replaced by 72px logo in Commit 2b |
| B8 | Medium | Recruiting | `prospect_shortlist` is 100 rows not 1,849 as documented | BACKLOG (named project) |
| B9 | Medium (Sammy feedback) | Scout results | Rules mode returns semantically-bad matches | ✅ **RESOLVED Session 7 commit `9c0d90e`** (aiMode default flipped to LLM) |
| B10 | 🟠 NEW Session 7 | Player cards | Guard cards show BPG (near-zero, misleading) instead of SPG | BACKLOG — Position-aware defensive stat on card display |
| B11 | 🟠 NEW Session 7 | Scout Edge Function | LLM payload missing KenPom team / Torvik team / player AdjOE advanced metrics | BACKLOG — Scout Edge Function context sync (#1 priority) |

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
- **Commit 1 (Scout v2, commit d30a3ca):** Direction B premium redesign, engine toggle killed, 2×2 preset grids shipped.
- **Commit 2a (metric tooltips, commit 7ee3241):** plain-English translation across KenPom chips and player-level AdjOE stat-box.
- **Attempted + reverted:** Commit 2c (aiMode LLM default) — broke app due to JS statement-context issue.
- **Deferred to Session 7:** Commit 2b drawer work.

### Session 7 — EMERGENCY PIVOT: Scout LLM-default + mic fix (2026-04-18)
- **Planned target:** Commit 2b (drawer logo + role label + Howard fallback).
- **Emergency pivot at ~11:30 AM:** Demo at 3:00 PM, Scout returning broken results. User reported Scout "no longer works" and "mic turns into weird red listening button."
- **Shipped (Commit `9c0d90e`, 4 patches in one atomic commit):**
  - P1 `aiRun` inverted: default branch is now LLM (`if(aiMode==="rules")aiSearch();else aiLLM()`)
  - P5 `aiMode` state initializer flipped from `"rules"` to `"llm"` — this was the REAL root cause Session 6 Commit 2c missed
  - P2 `aiVoice` rewritten: uses `.listening` class toggle instead of destroying `textContent`, adds tap-to-cancel via `window._apxRec`
  - P3 mic button gets `apx-mic` class; P4 adds `.apx-mic.listening` pulse CSS
- **Shipped (Commit `d1806d2`):** repo cleanup — removed 3 backup files accidentally tracked in `9c0d90e`, added `index.html.*-precommit-*` to `.gitignore`.
- **Shipped (Commit `f68fb4f`):** cosmetic — "Donor & development" → "Donor Development" in Apex Patrons header.
- **Resolved: B9** (Scout rules-mode bad matches).
- **QA pass on deployed version:** 10 queries tested. 7 green, 2 yellow (revealed UI/Edge-Function gaps), 1 red (skip in demo). Full matrix in HANDOFF-2026-04-18-session7.md.
- **New bugs logged:** B10 (guard cards show BPG not SPG), B11 (Scout Edge Function missing advanced-metric payload).
- **Deferred to Session 8:** Commit 2b drawer work (original S7 target, now bumped).
