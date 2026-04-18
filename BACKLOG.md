# BACKLOG.md — Deferred work, named projects, ideas

**Updated:** 2026-04-18 (after Session 7 — Scout LLM-default emergency fix + 10-query QA)
**Updated by:** Claude when items get added or moved
**Promotion rule:** Items move from backlog into the active 14-day plan only by explicit decision in a session. Do not silently slip them in.

---

## Named projects (multi-session efforts)

### Scout Edge Function context sync (advanced-metric payload)  ← NEW Session 7 — #1 post-demo priority
**Estimated scope:** 2–4 sessions (frontend + backend Edge Function redeploy)
**Why it matters:** Scout today reasons over **only counting stats** (ppg/rpg/apg/3pct) + `apex_model` score. The rich data layer already in Supabase — KenPom team rating / AdjO / AdjD / Rank, Torvik team advanced metrics, player-level AdjOE/AdjDE from `prospect_stats` — is **NOT piped into the LLM prompt payload**. Claude is doing impressive agentic reasoning on a thin slice of data when the full universe is sitting in the DB.

Confirmed live in Session 7 QA pass. When queried "shooters from top-50 KenPom defenses," Claude responded: *"KenPom team efficiency data is not yet integrated into the system. That feature ships in Scout 1.2."* — incorrect at the data-layer level (KenPom has been fully integrated since Session 4b) but CORRECT at the payload level (it's not being sent to Claude). Same pattern for "rim protectors" queries — Claude can't reason over Torvik defensive rebounding rate because that field isn't in the prospect object.

**Status:** New named project, promoted to #1 priority for post-demo work pending mentor feedback at 3pm 2026-04-18.

**Full scope:**

*Frontend (aiLLM function in index.html):*
- Extend prospect-enrichment step in `window.aiLLM` (around byte 146898) to include:
  - Player's current team context: `kenpom_adj_em`, `kenpom_adj_o`, `kenpom_adj_d`, `kenpom_rank`, `kenpom_tempo` — via existing `schoolKenPomMap` lookup by `current_school`
  - Player's current team Torvik: `torvik_adj_o`, `torvik_adj_d`, `torvik_tempo`, `torvik_rank`, `torvik_efg_pct` — via `schoolTorvikMap` lookup
  - Player-level Torvik: `adjoe`, `adjde`, `usg_pct`, `min_pct` — JOIN from `prospect_stats` in initial prospects load
- Update JSON payload shape sent to Edge Function — each prospect object carries `team_stats: {...}` and `player_stats: {...}` nested blocks alongside existing fields
- Backwards-compatible: Edge Function can ignore unknown fields if backend isn't updated yet (frontend-only path is lower-risk)

*Backend (Supabase Edge Function at /functions/v1/apex-scout):*
- Update system prompt to document the new fields and how Claude should use them — especially position-aware defensive metrics (SPG for guards, BPG + defensive rebounds for bigs, per-possession AdjDE for positional neutrality)
- Update tool definitions so `search_prospects` accepts filter predicates on the new fields — e.g., `team_kenpom_defense_rank_max: 50` for "top-50 KenPom defenses"
- Increase context budget if needed to fit richer prospect objects

*Validation:*
- Re-run Session 7 Q8 ("shooters from top-50 KenPom defenses") — Claude should filter correctly and NOT self-report the data as unavailable
- Re-run "rim protectors" variants — results should weight Torvik player-level AdjDE and position, not just raw BPG
- Verify Sammy-pitch-aligned queries ("high-usage scorers from weak defenses") actually work

**Risk / open questions:**
- Will larger payload slow response latency? Probably yes, modestly — worth measuring.
- Does the Edge Function need to change in lockstep, or can the frontend ship standalone first? Frontend-only should be the pilot — Claude will naturally use new JSON fields when present even without prompt changes, though precision will be higher with both.

### Position-aware defensive stat on card display  ← NEW Session 7 (from Q6 QA finding)
**Estimated scope:** ~1 session
**Why it matters:** Player cards show `ppg / rpg / apg / bpg / 3PT%`. Blocks-per-game is the wrong defensive metric for guards (always near zero) and misrepresents their defensive value. A coach sees "0.1 bpg" on a guard card and reads "can't defend" when Scout may have actually surfaced a two-way guard because of SPG or defensive rating.
**Status:** Logged Session 7. Bug B10. Related to but NOT the same as "Player-level stat chips on prospect cards" — this is a display-field swap, that's a richer chip-family.

**Full scope:**
- Detect position on card render (`p.position`)
- For guards (PG/SG/G): show `ppg / rpg / apg / **spg** / 3PT%` instead of bpg
- For forwards/centers (F/PF/C/SF/FC): keep existing `ppg / rpg / apg / bpg / 3PT%`
- Source: `spg` from `prospect_stats.spg` column (verify exists; if not, add ingestion step)
- Render via existing `statsRow` helper near line 3000s — one conditional branch on position

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
**Status:** Session 1 shipped a lightweight "Logged in as · Coach Stewart · HC" indicator but no real permissions. Commit 2b drawer label rewrite (HC → Assistant Coach) still deferred post-Session-7.

**Full scope:** `users` table: name, role (head_coach / assistant_coach / gm / dobo / ga / manager), school_id, permissions JSON. `staff_invites` table. Drawer items show/hide per role. Action buttons show/hide per role. Data masking (NIL dollar amounts hidden from non-financial roles). Activity attribution. Audit log.

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
**Why it matters:** DATA-STATUS previously claimed 657 HS + 1,192 portal = 1,849 prospects in `prospect_shortlist`, but live DB has only 100 rows — all with `school_id=Oregon` regardless of their `current_school` text.
**Status:** Identified Session 4b (bug B8).

**Full scope:** Reconcile 247Sports Composite ingestion. Decide whether `prospect_shortlist.school_id` is the interested school or the prospect's current school. Standardize `current_school` column. Populate `committed_to_school_id` for committed portal players.

### Scout engine auto-routing (LLM default)  ← RESOLVED Session 7 commit 9c0d90e
**Status:** ✅ SHIPPED. `aiMode` state init flipped from `"rules"` to `"llm"`, `aiRun` inverted so undefined/default → LLM branch. Root cause was NOT the `var aiMode` declaration (Session 6's Commit 2c theory) — it was the explicit state initializer `aiResults=null,aiMode="rules",aiLoading=false` at byte 64474 of index.html. One-byte change (`"rules"` → `"llm"`) fixed it. B9 closed.

### Player-level stat chips on prospect cards  ← NEW Session 6 (Sammy-pitch-aligned) — SUPERSEDED by Scout Edge Function context sync
**Status:** Concept preserved but effectively folded into the larger Scout Edge Function context sync project above. The card-level chip display is a separate surface from the LLM-payload work — but both depend on pulling `prospect_stats.adjoe/adjde/usg_pct`. Sequence: ship Edge Function context sync first (backend wiring), then build card chips using the same data pipeline.

### "Team ·" prefix on KenPom chip popovers  ← Session 6 1a micro-polish
**Estimated scope:** Under 15 minutes standalone
**Status:** Still open. Session 7 didn't retry. Small when attempted alone.

---

## Named projects — from Sammy Gelfand pitch (2026-04-17)

*(Unchanged from Session 6 close — all 6 projects still open.)*

### Program-fit scoring  ← Sammy pitch 2026-04-17
**Estimated scope:** 3–5 sessions (requires data science + historical modeling)
**Why it matters:** Sammy: *"everything is by the rankings, the On3, the ESPN, the NIL, and so all they're doing is paying for the number, like, the ranking number, and not actually for the player... Fit your system, fit your players, fit your coaches."*
**Status:** Concept logged. Related to Scout Edge Function context sync — once rich metrics flow to the LLM, Claude can approximate program-fit reasoning in-prompt before a proper model ships.

### Recruitment likelihood indicators (Intelligence panel)  ← Sammy pitch
**Estimated scope:** 2–3 sessions
**Why it matters:** Sammy: *"we're top 3 with this kid, or we're, like, you know, it's a long shot."*
**Status:** Concept logged.

### Shoe company affiliations  ← Sammy pitch
**Estimated scope:** 2 sessions
**Why it matters:** Nike/Adidas/Under Armour alignment creates recruiting gravitational pull.
**Status:** Concept logged. Most-achievable Sammy item.

### Transfer success models  ← Sammy pitch
**Estimated scope:** 4–6 sessions (data science heavy)
**Why it matters:** Historical transfer green-flag / red-flag modeling.
**Status:** Concept logged.

### Donor preference profiles  ← Sammy pitch
**Estimated scope:** 2–3 sessions
**Why it matters:** Sammy: *"each donor has stuff that they care about... finding the pressure points."*
**Status:** Concept logged.

### Donor-player matching  ← Sammy pitch
**Estimated scope:** 2–3 sessions (builds on Donor preference profiles)
**Why it matters:** Sammy: *"Pat Kilkenny wants Jucares. He fucking loves Jucares. He'll give you $5 million for Jucares."*
**Status:** Concept logged.

---

## Smaller items (single-session or less)

### From Session 1 audit
- **B3:** Howard `head_coach_name` is NULL → greeting reads "Good morning, Coach." Now folded into Session 8 drawer work (S7 pivoted).
- **B4:** Avg PPG = 0.0 on non-Oregon schools in Roster Pulse.
- **B6:** School picker doesn't find "Saint Mary's" when searching "st."
- **B7:** Avatar bg color isn't school-themed. **Still open** — Commit 2b deferred again.
- **Defer-from-S1:** Per-screen audit of non-Oregon schools.
- **CRM attribution row** ("added by …") — ~1 session standalone.

### From Session 2
- **S2-minor-1:** Session 2 hex colors hardcoded in `S2_ASSISTANT_COLORS`. Promote to CSS vars. ~15 min.
- **S2-minor-2:** Seasonally-aware "Next up" scheduling. ~1 session.

### From Session 3 — KenPom backend
- **S3-minor-1:** KenPom weekly cron automation. ~1 session.
- **S3-minor-2:** `scripts/kenpom_sync.py` deliberately uses `fuzz.ratio`. Documented.
- **S3-minor-3:** `KENPOM_TO_SCHOOL_OVERRIDES` startup check. ~15 min.

### From Session 4a — KenPom frontend Home surfaces
- **S4a-minor-1:** `node` not installed on Mac. ~10 min.
- **S4a-minor-2:** `CURRENT_SCHOOL_NAME` form mismatch. Not urgent.
- **S4a-minor-4:** Oregon `On roster: 17` mismatches 78 Oregon players. May share root cause with B4.

### From Session 4b — KenPom frontend recruiting/modal
- **S4b-minor-4:** Variant B chip NOT verified live — unblocks once B8 lands.

### From Session 5 — Torvik end-to-end
- **S5-minor-1:** Three Torvik columns NULL pending index-mapping. ~30 min.
- **S5-minor-2:** Prune defensive `X -> X` override no-ops. ~10 min.
- **S5-minor-3:** Startup shape-check on Torvik JSON endpoint. ~15 min.
- **S5-minor-4:** Torvik freshness automation.

### From Session 5c — Smart Filter Torvik tier
- **S5c-minor-1:** Button labels mixed case; other sections ALL CAPS. ~5 min.
- **S5c-minor-2:** Add "Player's On3 national rank" subhead. ~5 min.

### From Session 6a — Scout v2 hero
- **S6a-minor-1:** Scout v2 killed the engine toggle UI but `setAiMode` function still in source as dead code. Remove in a future Scout pass. ~5 min.
- **S6a-minor-2:** "Defensive upgrades" subtitle uses Math.random fake-ranking. Replace with real rank. ~15 min.
- **S6a-minor-3:** Dim/disable Search button when input matches last-run query. ~20 min.

### From Session 6b — Metric tooltips
- **S6b-minor-1:** Tooltip arrow/caret pointing to the `ⓘ`. ~20 min.
- **S6b-minor-2:** Refactor inline `onclick` strings into event delegation via `data-metric-key`. ~30 min.
- **S6b-minor-3:** Four of six `apxMetricInfo` definitions ready but not yet wired to UI.

### From Session 7 — Scout LLM fix + QA pass  ← NEW
- **S7-minor-1:** Card display doesn't show height or class (jr/sr). Scout reasons on these but they're invisible on cards. ~30 min for both.
- **S7-minor-2:** Edge Function system prompt references "Scout 1.2" / "Scout 1.3" roadmap labels that don't match current DATA-STATUS reality. Update prompt to reflect shipped state (KenPom is LIVE, NIL is PARTIAL). ~15 min once we have Edge Function access to edit.
- **S7-minor-3:** Scout Analysis panel sometimes suggests "query variations" with specific numeric thresholds (e.g., "try 1.5+ BPG instead of 2+"). Consider rendering these as tappable chips that re-run the query. Fun UX but non-trivial. ~1 session.
- **S7-minor-4:** No frontend display of `apex_model` score on prospect cards. Value is used in LLM payload but invisible to coach. Consider surfacing as a chip. ~20 min.

### Carryover from previous sessions
- Empty-state skeletons for seed-data screens
- CRM/Watchlist school-scoping (cross-school bleed)
- `prospect_contacts` real schema
- `prospect_visits` real schema
- Bottom nav bar (mobile UX upgrade)
- Real-time roster builder UX polish
- Logo + subtitle font sizing pass
- 9 Oregon players missing photos: Darren Harris, Dezdrick Lindsay, Ege Demir, Kwame Evans Jr., Nikolas Khamenia, Sven Djopmo, Travelle Bryson, Tyler Lundblade, Win Miller

### Carryover from 4/16 wrap
- Apex Picks preset strip in Smart Filter (note: Scout v2 absorbed the preset concept — re-evaluate)
- Default NIL budget per school (Session 8 in original plan)
- Patrons data for non-Oregon Power 5 (Session 9 in original plan)

### Repo hygiene
- **✅ DONE 2026-04-17 (Session 3):** Cleaned `~/code/apex-intel/`
- **✅ DONE 2026-04-17 (Session 4a):** Added `*-backup-*` pattern to Apex-App `.gitignore`
- **✅ DONE 2026-04-17 (Session 5):** Removed stray empty `main` file
- **✅ CONFIRMED 2026-04-18 (Session 6):** `.gitignore` for `*-backup-*` working
- **✅ DONE 2026-04-18 (Session 7, commit d1806d2):** Removed 3 backup files accidentally tracked in commit 9c0d90e; added `index.html.*-precommit-*` pattern to `.gitignore`

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
