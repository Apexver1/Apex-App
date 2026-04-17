# DATA-STATUS.md — Real vs Dummy data tracker

**Updated:** 2026-04-17 (after Session 1 audit + badge removal)
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
- **Gap (B3):** Howard `head_coach_name` is NULL — drawer shows blank name + bare "Head coach · Howard"; greeting reads "Good morning, Coach." with no last name. The fallback to "Head Coach" + initials in `loadData()` line ~1101 is not actually rendering. Fix: data backfill OR make fallback more defensive.

### Home — Smart Suggestions
- **Status:** ✅ REAL
- Computed from prospects + CRM logs

### Home — Today's Brief tiles
- **Status:** 🔴 BROKEN (downgraded from 🟠 PARTIAL — see B5)
- Open scholarships: 🔴 BROKEN (Supabase query has `is_seed_data=eq.true` filter, no `school_id` filter — every school shows the same "7 of 13")
- NIL requests: 🔴 BROKEN (no school filter — every school shows same global count)
- CRM follow-ups: 🔴 BROKEN (`crm_log` has no school filter)
- Reminders today: 🔴 BROKEN (`contact_reminders` has no school filter)
- Visits this week: 🟡 DUMMY (from `VISITS_SEED`, Oregon-only — see B2)
- Portal churn: 🟡 DUMMY (hardcoded `12` in renderHome)
- **Root cause:** `loadData()` lines 1074–1087 — 4 of 14 queries lack `school_id` filtering. Architectural fix, not cosmetic. See BACKLOG named project "School-scope the global queries."

### Home — Next Up game
- **Status:** 🟡 DUMMY (Sample badge removed Session 1)
- `NEXT_GAME` constant, Oregon-vs-Arizona hardcoded — shows on EVERY school (B1 confirmed)
- **Fix:** Session 2 — per-school generator

### Home — Roster Pulse
- **Status:** 🟠 PARTIAL → 🔴 BROKEN for non-Oregon (B4)
- On roster count: ✅ REAL (varies per school: Oregon 17 / Duke 14 / Kansas 16 / Howard 16)
- Avg PPG (starters): 🔴 BROKEN — Oregon "12.9" but Duke/Kansas/Howard all show "0.0". Either depth_chart_position is unset for non-Oregon or PPG fields are NULL post-materialization.
- Depth score "B+": 🟡 DUMMY (hardcoded — every school shows B+)
- NIL committed/cap: ✅ REAL for Oregon, default `$0 / $10.0M` for others (acceptable until Session 8)

### Home — Hot in the portal
- **Status:** ✅ REAL
- Pulled from `prospect_shortlist` filtered to portal source (`qSchool` filter present)

### Home — Your shortlist
- **Status:** ✅ REAL

### Home — College basketball news
- **Status:** 🟡 DUMMY (Sample badge removed Session 1)
- `NEWS_SEED` constant, generic basketball headlines
- **Fix:** Session 2 — per-school generator

### Roster
- **Status:** ✅ REAL for schools we've loaded
- Audit: not screenshot-tested across schools in Session 1 (deferred — known good per Phase 4 work)
- Photos: 5,136 of 7,580 players have ESPN photos
- Photo coverage gap: 9 of 78 Oregon roster players missing photos (ESPN data gaps, not bugs)

### Recruiting (Portal / HS / Watchlist tabs)
- **Status:** ✅ REAL
- 1,192 portal players + 657 HS prospects in `school_rosters`
- Photos propagated to `prospect_shortlist`
- Drawer badge "Recruiting 42" shown across all schools — likely also affected by lack of school-scoping in `prospects` query result (verify in Session 2)

### Player cards / detail modal
- **Status:** ✅ REAL
- Hero-size photos, school logos, KenPom rank section ❌ MISSING (placeholder text — Session 4 to wire)
- Comparison mode: ✅ REAL (shipped in 5e)

### Apex CRM
- **Status:** ✅ REAL
- `crm_added` and `watchlist` tables persist across sessions
- **Gap:** user-scoped not school-scoped (cross-school bleed) — backlogged
- **Gap:** no attribution shown ("added by ...") — was on Session 1 list, deferred to dedicated session

### Scheduled visits
- **Status:** 🟡 DUMMY (Oregon-only, Sample badge removed Session 1)
- `VISITS_SEED` constant, hardcoded Jalen Washington / Casanova / Knight Arena (Oregon facilities)
- Calendar header subtitle correctly reflects current school name + date — but visit cards below are Oregon data on every school (B2 confirmed)
- Calendar dots on Apr 25 + Apr 29 same root cause
- **Fix:** Session 2 — per-school generator

### NIL Budget
- **Status:** 🟠 PARTIAL
- ✅ REAL for Oregon (loaded in Phase 4)
- ❌ MISSING for other 363 schools (no `nil_budget` row → defaults to `$0 / $10.0M`)
- **Fix:** Session 8 — default budget materialization on first school switch

### Roster Builder
- **Status:** ✅ REAL
- Reads from `my_roster` + `nil_budget` for current school
- Audit: not screenshot-tested across schools in Session 1

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
- Sample badge removed Session 1
- **Fix:** Session 9 — Power 5 schools get unique seeded data; non-P5 = "Enable Patrons" empty state

### Apex Ops
- **Status:** 🟡 DUMMY (Oregon-only — `OPS_CATS`, `OPS_RECENT`, `OPS_DECKS`)
- Sample badge removed Session 1
- **Fix:** Session 2 — per-school generator (originally planned for later, but lumped with Visits/News)

### School picker
- **Status:** ✅ REAL with one search bug
- 365 D1 programs filterable by name + conference
- **Bug B6:** Searching "st." doesn't match Saint Mary's (Gaels) because their DB row uses full word "Saint" not "St." Cosmetic search-UX issue. Fix: alias-aware search OR data-side normalization.

### NIL Vault (NEW — does not yet exist)
- **Status:** ❌ NOT BUILT
- **Session 9** ships stub: drawer entry + one Oregon player with sample contract + static AI-extracted view
- Real spec: BACKLOG → "NIL Contract Vault" project

---

## By data source

### Supabase tables (22 total)
| Table | Coverage | School-scoped in loadData()? | Notes |
|---|---|---|---|
| schools | ✅ 1,518 rows, 365 with `espn_id` + conference | N/A (universe) | KenPom cols pending Session 3 |
| school_rosters | ✅ 7,580 players | ✅ via my_roster query | 5,136 with photos |
| my_roster | ✅ Oregon (78) | ✅ Yes | other schools materialized via RPC on switch |
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
| team_strategy | partial | ❌ No (`is_seed_data` filter only) | |
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
| KenPom | ❌ Not yet wired | Sessions 3+4 |
| Torvik | ❌ Not yet wired | Session 5 |
| 247Sports (HS + portal) | ✅ Phase 4 ingested | 657 HS + 1,192 portal |
| NCAA-API | ✅ Verified | not currently surfaced |

---

## Audit-discovered bug list (Session 1)

Tested across Oregon, Duke, Kansas, Howard. Saint Mary's not tested (search bug B6 prevented).

| ID | Severity | Surface | Description | Fix path |
|---|---|---|---|---|
| B1 | High | Home → Next up | "Oregon vs Arizona" hardcoded on every school | Session 2 |
| B2 | High | Visits | Jalen Washington + Oregon facilities show on every school | Session 2 |
| B3 | Medium | Drawer | Howard shows blank head coach name (NULL in DB); fallback not rendering | BACKLOG |
| B4 | Medium | Home → Roster pulse | Avg PPG = 0.0 on Duke/Kansas/Howard (Oregon shows real 12.9) | BACKLOG |
| B5 | High (architectural) | Home → Today's brief | 4 of 14 loadData queries lack school_id filter; 4 tiles show identical numbers across all schools | BACKLOG (named project) |
| B6 | Low | School picker | Searching "st." doesn't match Saint Mary's (uses "Saint" full-word) | BACKLOG |
| B7 | Cosmetic | Drawer | Avatar bg color isn't school-themed | BACKLOG |

---

## Session log (append at end of each session)

### Session 0 — Phase 5f shipped (2026-04-16)
- iPhone drawer scrolls
- Sample badges added to seed-data screens (will be removed in Session 1)
- DATA-STATUS.md created
- 14-day plan locked

### Session 1 — Audit + badge removal + current-user indicator (2026-04-17)
- All 5 Sample badges removed (Home Next Up, Home News, Visits, Patrons, Ops eyebrows) + 4 demo-note paragraphs
- `.chip-demo` / `.demo-note` CSS rules retained for potential Session 2 reuse
- Current-user indicator added to drawer (Option C — separate row below school-coach card, "Logged in as · Coach Stewart · HC")
- Color palette confirmed byte-identical to Phase 5f
- Audit completed across Oregon, Duke, Kansas, Howard (Saint Mary's blocked by B6)
- 7 bugs catalogued (B1–B7); B1+B2 already in Session 2 plan, B5 escalated to named BACKLOG project
- Session shipped as `index-s1.html` → `index.html` on main
- Out of scope (deferred): per-screen audit of Roster, Recruiting, Roster Builder, Smart Filter, Apex Scout — Session 1 focused on Home/Visits/Drawer where the badges lived
