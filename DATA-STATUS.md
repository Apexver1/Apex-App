# DATA-STATUS.md — Real vs Dummy data tracker

**Updated:** 2026-04-16 (after Phase 5f ship)
**Updated by:** Claude at end of every session

This doc tracks the **truth** of what data is real vs dummy across the app, per screen and per school. Source of truth for Claude across sessions. Not visible to end users — visual indicators (the "Sample" badges shipped in 5f) are being **removed in Session 1** in favor of invisible-but-tracked-here dummy fallbacks.

## Legend
- ✅ **REAL** — pulls from a live data source (Supabase / API)
- 🟡 **DUMMY** — hardcoded or generated placeholder, designed to look identical to real
- 🟠 **PARTIAL** — some fields real, some dummy
- ❌ **MISSING** — currently broken or empty for some/all schools

---

## By screen

### Login / Splash
- **Status:** ✅ REAL
- Auto-login wired to `apex@test.com` / `ApexTest123`
- Splash + "Build a Champion" button always renders

### Drawer / Navigation
- **Status:** ✅ REAL
- Coach name + initials per school: ✅ REAL (pulled from `schools.head_coach_name`, fallback to school initials)
- Drawer badges (Home reminders, Recruiting count, CRM overdue): ✅ REAL
- iPhone scroll: ✅ FIXED in Session 5f
- **Gap:** No "current user" / role indicator yet — Session 1 to add

### Home — Smart Suggestions
- **Status:** ✅ REAL
- Computed from prospects + CRM logs

### Home — Today's Brief tiles
- **Status:** 🟠 PARTIAL
- Open scholarships: ✅ REAL (from `scholarships` table)
- NIL requests: ✅ REAL (from `nil_budget_requests`)
- CRM follow-ups: ✅ REAL
- Visits this week: 🟡 DUMMY (from `VISITS_SEED`, Oregon-only)
- Portal churn: 🟡 DUMMY (hardcoded `12`)

### Home — Next Up game
- **Status:** 🟡 DUMMY
- `NEXT_GAME` constant, Oregon-vs-Arizona hardcoded
- Marked with "Sample" badge in 5f — **removing in Session 1**, replacing with per-school generator in Session 2

### Home — Roster Pulse
- **Status:** 🟠 PARTIAL
- On roster count, Avg PPG: ✅ REAL (from `my_roster`)
- Depth score "B+": 🟡 DUMMY (hardcoded)
- NIL committed/cap: ✅ REAL (from `nil_budget`)

### Home — Hot in the portal
- **Status:** ✅ REAL
- Pulled from `prospect_shortlist` filtered to portal source

### Home — Your shortlist
- **Status:** ✅ REAL

### Home — College basketball news
- **Status:** 🟡 DUMMY
- `NEWS_SEED` constant, generic basketball headlines
- Marked with "Sample" badge in 5f — **removing in Session 1**, replacing with per-school generator in Session 2

### Roster
- **Status:** ✅ REAL for schools we've loaded
- Photos: 5,136 of 7,580 players have ESPN photos
- Stats (PPG, RPG, etc.): ✅ REAL where available
- Photo coverage gap: 9 of 78 Oregon roster players missing photos (ESPN data gaps, not bugs)
- **Gap to address Session 1 audit:** which of the 364 schools have populated rosters vs empty?

### Recruiting (Portal / HS / Watchlist tabs)
- **Status:** ✅ REAL
- 1,192 portal players + 657 HS prospects in `school_rosters`
- Photos propagated to `prospect_shortlist`

### Player cards / detail modal
- **Status:** ✅ REAL
- Hero-size photos, school logos, KenPom rank section ❌ MISSING (placeholder text — Session 4 to wire)
- Comparison mode: ✅ REAL (shipped in 5e)

### Apex CRM
- **Status:** ✅ REAL
- `crm_added` and `watchlist` tables persist across sessions
- **Gap:** user-scoped not school-scoped (cross-school bleed) — backlogged
- **Gap:** no attribution shown ("added by ...") — Session 1 to add lightweight version

### Scheduled visits
- **Status:** 🟡 DUMMY (Oregon-only)
- `VISITS_SEED` constant
- Marked with "Sample" badge in 5f — **removing Session 1**, per-school generator in Session 2

### NIL Budget
- **Status:** 🟠 PARTIAL
- ✅ REAL for Oregon (loaded in Phase 4)
- ❌ MISSING for other 363 schools (no `nil_budget` row)
- **Fix:** Session 8 — default budget materialization on first school switch

### Roster Builder
- **Status:** ✅ REAL
- Reads from `my_roster` + `nil_budget` for current school

### Smart Filter
- **Status:** 🟠 PARTIAL
- Filter engine: ✅ REAL
- Filter results: ✅ REAL (from `prospect_shortlist`)
- KenPom-based filtering: ❌ MISSING (no KenPom data in DB) — Sessions 3+4 unblock
- Torvik-based filtering: ❌ MISSING — Session 5 unblocks
- Apex Picks preset strip: ❌ MISSING — Session 7 ships

### Apex Scout
- **Status:** 🟠 PARTIAL
- LLM-mode and rules-mode both work
- Result cards have photos
- KenPom + Torvik integration: ❌ MISSING (Sessions 4 + 5 unblock)
- Polish pass: pending Session 6

### Apex Patrons
- **Status:** 🟡 DUMMY (Oregon-only — `DONORS_SEED`, `CAMPAIGNS_SEED`)
- Marked with "Sample" badge in 5f — **removing Session 1**
- **Fix:** Session 9 — Power 5 schools get unique seeded data; non-P5 = "Enable Patrons" empty state

### Apex Ops
- **Status:** 🟡 DUMMY (Oregon-only — `OPS_CATS`, `OPS_RECENT`, `OPS_DECKS`)
- Marked with "Sample" badge in 5f — **removing Session 1**, per-school generator in Session 2

### NIL Vault (NEW — does not yet exist)
- **Status:** ❌ NOT BUILT
- **Session 9** ships stub: drawer entry + one Oregon player with sample contract + static AI-extracted view
- Real spec: BACKLOG → "NIL Contract Vault" project

---

## By data source

### Supabase tables (22 total)
| Table | Coverage | Notes |
|---|---|---|
| schools | ✅ 1,518 rows, 365 with `espn_id` + conference | KenPom cols pending Session 3 |
| school_rosters | ✅ 7,580 players | 5,136 with photos |
| my_roster | ✅ Oregon (78) | other schools materialized via RPC on switch |
| prospect_shortlist | ✅ 50 Oregon prospects | school-agnostic via `school_id` |
| prospect_stats | ✅ | |
| prospect_scores | ✅ | |
| prospect_film | partial | |
| prospect_family | partial | |
| crm_contacts | partial | |
| crm_log | ✅ | |
| crm_added | ✅ | user-scoped — needs school-scoping |
| watchlist | ✅ | user-scoped — needs school-scoping |
| contact_reminders | ✅ | |
| team_strategy | partial | |
| nil_budget | 🟠 | Oregon only — fix Session 8 |
| nil_budget_requests | ✅ | |
| scholarships | ✅ | |
| staff | partial | |
| audit_log | ✅ | |
| search_history | ✅ | |
| source_tags | ✅ | |
| agent_memory | ✅ | |

### External data sources
| Source | Status | Coverage |
|---|---|---|
| ESPN (photos + roster + logos) | ✅ Active | 5,136 photos, 365 logos |
| BallDontLie | ✅ Active (NCAA) | not currently surfaced |
| CBBD | ✅ Active | not currently surfaced |
| KenPom | ❌ Not yet wired | Sessions 3+4 |
| Torvik | ❌ Not yet wired | Session 5 |
| 247Sports (HS + portal) | ✅ Phase 4 ingested | 657 HS + 1,192 portal |
| NCAA-API | ✅ Verified | not currently surfaced |

---

## Session log (append at end of each session)

### Session 0 — Phase 5f shipped (2026-04-16)
- iPhone drawer scrolls
- Sample badges added to seed-data screens (will be removed in Session 1)
- DATA-STATUS.md created
- 14-day plan locked
