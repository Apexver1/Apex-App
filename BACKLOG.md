# BACKLOG.md — Deferred work, named projects, ideas

**Updated:** 2026-04-18 (after Session 6 — Scout v2 + metric tooltips + new named projects)
**Updated by:** Claude when items get added or moved
**Promotion rule:** Items move from backlog into the active 14-day plan only by explicit decision in a session. Do not silently slip them in.

---

## Named projects (multi-session efforts)

### School-scope the global queries  ← from Session 1 audit
**Estimated scope:** 2–3 sessions
**Why it matters:** Today's brief tiles show identical numbers on every school because 4 of 14 queries in `loadData()` lack `school_id` filtering. Biggest "this isn't a real product" tell in the app.
**Status:** Identified Session 1 (bug B5). Architectural fix.

**Full scope:** Audit each table for `school_id` column, ALTER/backfill where missing. Tables: `scholarships`, `crm_log`, `nil_budget_requests`, `contact_reminders`, `prospect_stats`, `prospect_scores`, `prospect_family`, `team_strategy`. Update each query in `loadData()` to include `qSchool`. RLS policy review. Acceptance: every Today's brief tile shows different numbers or polished empty state per school.

### NIL Contract Vault
**Estimated scope:** 4–6 sessions over 2–3 weeks
**Status:** Stub UI ships in Session 9 (one player, one sample contract, static AI-extracted view). Full build deferred.

**Full scope:** Per-player contract upload (PDF/DOCX) → Supabase Storage. AI extraction pipeline pulling payment dates/amounts, deliverables, incentive triggers, compliance clauses. Live incentive tracking. Notifications. Audit log. Schema: `nil_contracts`, `nil_contract_terms`, `nil_contract_events`, `nil_contract_files`.

### Role-based access control (RBAC)
**Estimated scope:** 1–2 weeks
**Why it matters:** Apex Intel is for entire basketball staffs (HC, assistants, GM, DOBO, GA, managers) — different roles see different data.
**Status:** Session 1 shipped a lightweight "Logged in as · Coach Stewart · HC" indicator but no real permissions. Session 7 Commit 2b will change the label to `Coach Stewart · Assistant Coach` (Title Case, muted gray) as a visual placeholder — but that's just the label. Real RBAC is still the 1-2 week project.

**Full scope:** `users` table: name, role (head_coach / assistant_coach / gm / dobo / ga / manager), school_id, permissions JSON. `staff_invites` table. Drawer items show/hide per role. Action buttons show/hide per role. Data masking (NIL dollar amounts hidden from non-financial roles). Activity attribution. Audit log. Replace the hardcoded "Coach Stewart · Assistant Coach" with the actual logged-in user.

### Live transfer portal feed
**Estimated scope:** 3–4 sessions
**Status:** Pulled once in Phase 4 (1,192 entries). No live refresh wired.

### Push notifications
**Estimated scope:** 2 sessions
**Status:** Not built.

### Multi-user real-time collaboration
**Estimated scope:** Large — 1+ month
**Status:** Not built.

### Prospect pipeline reconciliation  ← from Session 4b recon
**Estimated scope:** 1–2 sessions
**Why it matters:** DATA-STATUS previously claimed 657 HS + 1,192 portal = 1,849 prospects in `prospect_shortlist`, but live DB has only 100 rows — all with `school_id=Oregon` regardless of their `current_school` text. UI renders fine, but product feels more Oregon-centric than it should.
**Status:** Identified Session 4b (bug B8).

**Full scope:** Reconcile 247Sports Composite ingestion. Decide whether `prospect_shortlist.school_id` is the interested school or the prospect's current school. Standardize `current_school` column. Populate `committed_to_school_id` for committed portal players.

### Scout engine auto-routing (LLM default)  ← NEW Session 6 (promoted from 2c failure)
**Estimated scope:** 1 session (fix + smart-routing) or 2 sessions (full)
**Why it matters:** Scout's Rules-mode regex parser returns semantically-bad matches. Live example from Session 6 smoke test: searching "rim protectors with 8+ rpg and 2+ bpg" returned Juke Harris (6'7" SF, 6.5 rpg, 36% from 3 — not a rim protector). The LLM mode (Scout 1.2, tool-use + agentic loop) is already shipped on the backend and returns much better results — it's just not the default. This is the single biggest "does Scout actually work" gap a beta tester will spot in 30 seconds.
**Status:** Attempted in Session 6 as Commit 2c (initialize `var aiMode="llm"` at startup). Caused JS parse error — the injected `var` conflicted with the surrounding statement context (`setAiMode=function...` was NOT prefixed with `var` or `window.`, so inserting a new `var` broke expression chaining). Reverted surgically. Commit 2a tooltip work preserved.

**Full scope:**
- Investigate the correct JS-context location for `aiMode` initialization. Likely needs to be inside the same IIFE / closure / object literal where other Scout state is declared. Use DevTools "Sources" panel to step through the Scout init path and find the right host variable.
- Option A (fast ship): set `aiMode="llm"` in the right place, all queries go through LLM mode.
- Option B (smart): auto-route per query — LLM for semantic queries ("rim protectors", "two-way wing", "diamonds in the rough"), rules for structured queries ("PG under $500K", "6'8+ 3PT>35%"). Preserves cost-savings intent while defaulting to LLM for anything ambiguous.
- Acceptance: searching "rim protectors with 8+ rpg and 2+ bpg" returns actual rim protectors (6'10+ players with blocks/rebounds profile), not scoring wings.

### Player-level stat chips on prospect cards  ← NEW Session 6 (Sammy-pitch-aligned)
**Estimated scope:** 1–2 sessions
**Why it matters:** Current KenPom chips on prospect cards show the **school's team rating** (e.g., Juke Harris's chip reads "Wake Forest #80 · +10.58 AdjEM"). A coach looking at that card wants to know *is Juke good*, not how good Wake Forest is. This is literally Sammy's "stop paying for the ranking number, pay for the player" critique in app form. We already have player-level Bart Torvik stats (AdjOE, AdjDE, usage%, min%) in `prospect_stats` — they just don't surface on the cards, only in the player modal's BART TORVIK ADVANCED section.
**Status:** Concept logged Session 6. Aligned with Sammy-pitch "Program-fit scoring" but narrower — this is the card-level signal, program-fit is the bigger modeling effort.

**Full scope:**
- Extend `loadData()` prospects SELECT to join `prospect_stats.adjoe` + `prospect_stats.adjde` + `prospect_stats.usg_pct`
- Build `playerStatsChipHTML(p)` helper mirroring `kenpomChipHTML` pattern
- Render both chips on cards when available: team-rating chip (existing) + new player-rating chip — e.g., "Offense 118 · Defense 104"
- Handle missing data gracefully — HS prospects and new portal entries won't have player-level Torvik. Show only the chips where we have real data.
- Popover support via existing `window.apxShowInfo()` helper — reuse `player_offense` key in `window.apxMetricInfo`
- Decide visual hierarchy: which chip reads first? Leaning player-first for Scout results (coaches want "is this kid good") and team-first for Home Next Up (coaches want "how good is Arizona").

### "Team ·" prefix on KenPom chip popovers  ← NEW Session 6 (1a micro-polish)
**Estimated scope:** Under 15 minutes (trivial copy fix — but needs working JS context)
**Why it matters:** Popovers on KenPom chips currently show values as `Wake Forest: +10.58 AdjEM · #80`. The colon framing makes it slightly ambiguous whether the rating belongs to the player or the team. Changing to `Team · Wake Forest · +10.58 AdjEM · #80` makes it explicit. Small but real — reduces coach confusion.
**Status:** Attempted in Session 6 as part of Commit 2c/1a combo. Reverted because the 2c change broke the app; 1a was rolled back at the same time. Needs a standalone patch without the 2c JS-context issue.

**Full scope:** Single string rewrite in the `kenpomChipHTML` `_valLine` assignment. Patch anchor: the `_valLine=(_schoolTxt?_schoolTxt+": ":"")+...` line. Change to `_valLine=(_schoolTxt?"Team \u00b7 "+_schoolTxt+" \u00b7 ":"")+...`. Trivial when attempted standalone. Five minutes.

---

## Named projects — from Sammy Gelfand pitch (2026-04-17)

*(Unchanged from Session 5 close — all 6 projects still open, details preserved below.)*

### Program-fit scoring  ← Sammy pitch 2026-04-17
**Estimated scope:** 3–5 sessions (requires data science + historical modeling)
**Why it matters:** Sammy: *"everything is by the rankings, the On3, the ESPN, the NIL, and so all they're doing is paying for the number, like, the ranking number, and not actually for the player... Fit your system, fit your players, fit your coaches."*
**Status:** Concept logged.

### Recruitment likelihood indicators (Intelligence panel)  ← Sammy pitch 2026-04-17
**Estimated scope:** 2–3 sessions
**Why it matters:** Sammy: *"we're top 3 with this kid, or we're, like, you know, it's a long shot."* Traffic-light status on every prospect.
**Status:** Concept logged.

### Shoe company affiliations  ← Sammy pitch 2026-04-17
**Estimated scope:** 2 sessions
**Why it matters:** Nike/Adidas/Under Armour alignment creates recruiting gravitational pull. Practical first-order win.
**Status:** Concept logged. Most-achievable Sammy item.

### Transfer success models  ← Sammy pitch 2026-04-17
**Estimated scope:** 4–6 sessions (data science heavy)
**Why it matters:** Historical transfer green-flag / red-flag modeling. Sammy: *"AJ Storr, great player, Wisconsin, goes to Kansas, and it's just a complete disaster."*
**Status:** Concept logged.

### Donor preference profiles  ← Sammy pitch 2026-04-17
**Estimated scope:** 2–3 sessions (integrates into Apex Patrons)
**Why it matters:** Sammy: *"each donor has stuff that they care about... finding the pressure points."*
**Status:** Concept logged.

### Donor-player matching  ← Sammy pitch 2026-04-17
**Estimated scope:** 2–3 sessions (builds on Donor preference profiles)
**Why it matters:** Sammy: *"Pat Kilkenny wants Jucares. He fucking loves Jucares. He'll give you $5 million for Jucares."*
**Status:** Concept logged.

---

## Smaller items (single-session or less)

### From Session 1 audit
- **B3:** Howard `head_coach_name` is NULL → greeting reads "Good morning, Coach." Fold into Session 7 drawer work.
- **B4:** Avg PPG = 0.0 on non-Oregon schools in Roster Pulse.
- **B6:** School picker doesn't find "Saint Mary's" when searching "st."
- **B7:** Avatar bg color isn't school-themed. **Becomes moot in Session 7** — the 72px school-logo replacement eliminates the avatar color surface entirely.
- **Defer-from-S1:** Per-screen audit of Roster, Recruiting, Roster Builder, Smart Filter, Apex Scout across non-Oregon schools.
- **CRM attribution row** ("added by …") — ~1 session standalone.

### From Session 2
- **S2-minor-1:** Session 2 hex colors hardcoded in `S2_ASSISTANT_COLORS`. Promote to `--coach-accent-*` CSS variables. ~15 min.
- **S2-minor-2:** Seasonally-aware "Next up" scheduling. ~1 session.

### From Session 3 — KenPom backend
- **S3-minor-1:** KenPom data freshness. Weekly cron automation. ~1 session.
- **S3-minor-2:** `scripts/kenpom_sync.py` deliberately uses `fuzz.ratio` (not `token_set_ratio`). Documented so future me doesn't "helpfully" switch back.
- **S3-minor-3:** `KENPOM_TO_SCHOOL_OVERRIDES` startup check. ~15 min.

### From Session 4a — KenPom frontend Home surfaces
- **S4a-minor-1:** `node` not installed on Mac. ~10 min.
- **S4a-minor-2:** `CURRENT_SCHOOL_NAME` holds "Oregon Ducks" form; `schoolKenPomMap` keyed by shorter "Oregon" form. Not urgent.
- **S4a-minor-4:** Oregon `On roster: 17` mismatches DATA-STATUS's documented 78 Oregon players. May share root cause with B4.

### From Session 4b — KenPom frontend recruiting/modal
- **S4b-minor-4:** Variant B chip ("Committed: #X [school]") NOT verified live — unblocks once B8 (Prospect pipeline reconciliation) lands.

### From Session 5 — Torvik end-to-end
- **S5-minor-1:** Three Torvik columns (`torvik_efg_pct`, `torvik_efg_d_pct`, `torvik_tov_pct`) NULL pending index-mapping verification. ~30 min.
- **S5-minor-2:** Prune defensive `X -> X` override no-ops in `torvik_team_sync.py`. ~10 min.
- **S5-minor-3:** Startup shape-check on `torvik_team_sync.py` JSON endpoint. ~15 min.
- **S5-minor-4:** Torvik freshness automation — unify with S3-minor-1. ~1 session for both.

### From Session 5c — Smart Filter Torvik tier
- **S5c-minor-1:** Button labels mixed case; other sections ALL CAPS. ~5 min.
- **S5c-minor-2:** Add "Player's On3 national rank" subhead to Rankings section. ~5 min.

### From Session 6a — Scout v2 hero  ← NEW
- **S6a-minor-1:** Scout v2 killed the engine toggle UI but `setAiMode` function is still in the source as dead code. Harmless but noisy. Remove during a future Scout pass. ~5 min.
- **S6a-minor-2:** Smart preset copy interpolates live values but "Defensive upgrades" subtitle shows rounded `AdjD` value with some Math.random fake-ranking logic in the initial ship. Real rank display (e.g., "Ranked #X on defense") would be more accurate. ~15 min.
- **S6a-minor-3:** UX polish — when user clicks a preset card, the input auto-fills and the search auto-fires. The Search button is then a no-op (clicking it re-runs the same query on the same data, looks broken). Dim/disable the Search button when input matches last-run query. ~20 min.

### From Session 6b — Metric tooltips  ← NEW
- **S6b-minor-1:** Tooltip arrow/caret pointing to the `ⓘ` — popover currently just floats below/above the element with no visual connection. Small design polish. ~20 min.
- **S6b-minor-2:** Tooltip on Home Roster Pulse label uses inline `onclick="window.apxShowInfo(...)"` with escaped-quote string. Works but is brittle. Prefer event delegation pattern via a `data-metric-key` attribute + one global click handler. Refactor if we do more tooltip sites. ~30 min.
- **S6b-minor-3:** Six metric definitions exist in `window.apxMetricInfo` but only `team_strength` and `player_offense` are wired to live UI. The other four (`team_offense`, `team_defense`, `team_pace`, `team_winprob`) are ready to use when we surface those metrics. No urgent work — they'll be used as needed.

### Carryover from previous sessions
- Empty-state skeletons for seed-data screens
- CRM/Watchlist school-scoping (cross-school bleed)
- `prospect_contacts` real schema (replace `dummyContactsFor()`)
- `prospect_visits` real schema (replace `VISITS_SEED`)
- Bottom nav bar (mobile UX upgrade)
- Real-time roster builder UX polish
- Logo + subtitle font sizing pass
- 9 Oregon players missing photos: Darren Harris, Dezdrick Lindsay, Ege Demir, Kwame Evans Jr., Nikolas Khamenia, Sven Djopmo, Travelle Bryson, Tyler Lundblade, Win Miller

### Carryover from 4/16 wrap
- Apex Picks preset strip in Smart Filter (note: Scout v2 absorbed the preset concept into the Scout landing; the Smart-Filter version may now be redundant — re-evaluate before building)
- Default NIL budget per school (Session 8 in original plan)
- Patrons data for non-Oregon Power 5 (Session 9 in original plan)

### Repo hygiene
- **✅ DONE 2026-04-17 (Session 3):** Cleaned `~/code/apex-intel/`
- **✅ DONE 2026-04-17 (Session 4a):** Added `*-backup-*` pattern to Apex-App `.gitignore`
- **✅ DONE 2026-04-17 (Session 5):** Removed stray empty `main` file
- **✅ CONFIRMED 2026-04-18 (Session 6):** `.gitignore` for `*-backup-*` working (Session 6's `.s6-backup-*` and `.s6b-precommit-*` files not in `git status`)
- Standard commit message format — loose convention

---

## Ideas / explorations (not yet specced)

### Competitor research (incoming)
- Michael will send screenshots from a competitor in next session
- Action: review, extract integrate-able patterns, add to backlog with `competitor-inspired` tag

### Other API integrations not yet wired
- Synergy / Sportradar (commercial, paid)
- InStat / Hudl — film integration
- On3 NIL valuation feed

### Mobile-app-style features
- "Add to Home Screen" PWA configuration
- Offline mode for travel / weak-signal
- Native iOS app

### Sammy pitch — Apex Finder (donor-scraping)
- Revisit after Donor preference profiles lands.

---

## Out of scope (not pursuing)

- Recruiting *services* (we're a tool, not a service)
- Player social media management
- Coach education / playbook content
