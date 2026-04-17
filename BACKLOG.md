# BACKLOG.md — Deferred work, named projects, ideas

**Updated:** 2026-04-17 (after Session 4b KenPom frontend — recruiting cards + player modal)
**Updated by:** Claude when items get added or moved
**Promotion rule:** Items move from backlog into the active 14-day plan only by explicit decision in a session. Do not silently slip them in.

---

## Named projects (multi-session efforts)

### Prospect pipeline reconciliation  ← NEW Session 4b (bug B8)
**Estimated scope:** 1–2 sessions
**Why it matters:** DATA-STATUS has claimed 1,192 portal + 657 HS prospects for weeks, but `prospect_shortlist` has only 100 rows — all with NULL `source_tag`, all with `school_id` pointing at Oregon. Either (a) a Phase 4 ingestion landed in a different table that never got joined in, (b) the 247Sports ingestion was never actually run against production and the numbers come from a staging log, or (c) the data's been there all along but gets filtered out upstream. Either way the product "feels" like an Oregon demo even when switched to Duke because the prospect universe is all Oregon-tagged.

**Investigation order:**
- Re-run `apex-intel/scripts/ingest_247*.py` (or whatever the portal ingestion script was in Phase 4) against live Supabase and see what happens
- If the script has been run, check for a `staging_prospects` or similar parallel table that never got merged
- Audit the seed script that populated the 100 existing rows — understand why they all got Oregon as `school_id`
- Once counts are fixed: re-confirm `current_school` text form is clean (not "Uncommitted" / "Committed - Oregon" compound strings)
- Decide: should `school_id` FK point at prospect's CURRENT school (where they're playing) or their COMMITTED school? Today it's neither — it's Oregon for everyone.

**Related:** unlocks Variant B "Committed: #X [school]" chip testing in Session 4b, which was code-reviewed but never fired against live data because `committed_to_school_id` is NULL on all 100 rows.

### School-scope the global queries
**Estimated scope:** 2–3 sessions
**Why it matters:** Today's brief tiles (scholarships / NIL requests / CRM follow-ups / reminders) show identical numbers on every school because 4 of 14 queries in `loadData()` (lines 1074–1087 of index.html) lack `school_id` filtering. This makes the app *look* like it has fake data even on screens where the underlying records are real. It's the single biggest "this isn't a real product" tell across the audit.
**Status:** Identified Session 1 (bug B5). Architectural fix, not cosmetic.

**Full scope:**
- Audit each table to confirm whether `school_id` column exists; ALTER TABLE + backfill where missing
- Tables to fix: `scholarships`, `crm_log`, `nil_budget_requests`, `contact_reminders`, `prospect_stats` (verify if needed), `prospect_scores`, `prospect_family`, `team_strategy`
- Update each query line in `loadData()` to include `qSchool`
- RLS policy review — ensure school-scoping is enforced at the policy level, not just the query level
- Acceptance: every Today's brief tile shows different numbers across Oregon vs Duke vs Kansas, OR shows zero with an empty state for non-Oregon
- Decide policy for empty: do we materialize defaults (like nil_budget plan in Session 8) or show "0" plus polished empty?

### NIL Contract Vault
**Estimated scope:** 4–6 sessions over 2–3 weeks
**Why it matters:** NIL compliance is a hair-on-fire problem for D1 programs. This feature alone could be the lead value-add for coaching staffs.
**Status:** Stub UI ships in Session 9 (one player, one sample contract, static AI-extracted view). Full build deferred.

**Full scope (when built):**
- Per-player contract upload (PDF/DOCX) → Supabase Storage
- AI extraction pipeline (Anthropic API or OpenAI) pulling:
  - Payment dates and amounts
  - Deliverables (appearances, social posts, brand work)
  - Performance incentive triggers (e.g., "$100K bonus if averages 14+ PPG")
  - Compliance clauses
- Live incentive tracking — pulls player stats, computes whether triggers are hit
- Notifications: coach → player, system → coach when triggers hit
- Audit log: who uploaded what, who viewed dollar amounts, who approved what
- Schema: `nil_contracts` (header), `nil_contract_terms` (extracted line items), `nil_contract_events` (notifications, status changes), `nil_contract_files` (storage refs)

**Sample contracts:** Michael has real Oregon contracts to use for development. Bring into the Session 9 chat.

### Role-based access control (RBAC)
**Estimated scope:** 1–2 weeks
**Why it matters:** Apex Intel is for entire basketball staffs (HC, assistants, GM, DOBO, GA, managers) — different roles see different data.
**Status:** Session 1 shipped a lightweight "Logged in as · Coach Stewart · HC" indicator (drawer row) but no real permissions. Full RBAC deferred.

**Full scope (when built):**
- `users` table: name, role (head_coach / assistant_coach / gm / dobo / ga / manager), school_id, permissions JSON
- `staff_invites` table for inviting team members
- Drawer items show/hide per role
- Action buttons (e.g., "Approve NIL request") show/hide per role
- Data masking: NIL dollar amounts hidden from non-financial roles
- Activity attribution: every CRM log, watchlist add, NIL approval shows who did it
- Audit log: every sensitive action recorded with role + user
- Replace the hardcoded "Coach Stewart · HC" indicator with the actual logged-in user

### Live transfer portal feed
**Estimated scope:** 3–4 sessions
**Status:** Pulled once in Phase 4 (1,192 entries per historical record). See "Prospect pipeline reconciliation" above before re-opening this — the 1,192 figure is now in question.

### Push notifications
**Estimated scope:** 2 sessions
**Status:** Not built. Web Push or Firebase Cloud Messaging path TBD.

### Multi-user real-time collaboration
**Estimated scope:** Large — 1+ month
**Status:** Not built. Supabase Realtime channel-based; would also require RBAC done first.

---

## Smaller items (single-session or less)

### From Session 1 audit
- **B3:** Howard `head_coach_name` is NULL → drawer shows blank name + "Coach." greeting with no last name. Fix options: (a) backfill missing head coach data across the schools table, (b) make the fallback in `loadData()` ~line 1101 more defensive, (c) both. Probably 30 min.
- **B4:** Avg PPG = 0.0 on non-Oregon schools in Roster Pulse. Either `depth_chart_position` is unset post-materialization or PPG fields are NULL. Investigation + fix probably 1 session. Session 4a observation: Oregon shows `On roster: 17` but DATA-STATUS says 78 — likely same root cause (starter/active filter). Session 4b did NOT re-surface this; still believed to be related to B4.
- **B6:** School picker doesn't find "Saint Mary's" when searching "st." — needs alias-aware search or data-side normalization. Session 4a observation: searching "saint" does return Saint Mary's correctly, so the bug may be narrower than originally specced. Worth a 5-min probe to confirm which prefix variants fail. ~30 min.
- **B7:** Avatar bg color isn't school-themed. Map school primary color → avatar bg. Pleasant polish. ~1 session if we want to do it across all 365 schools.
- **B8:** ⚠️ NEW — prospect pipeline reconciliation. See named project above.
- **Defer-from-S1:** Per-screen audit of Roster, Recruiting, Roster Builder, Smart Filter, Apex Scout across non-Oregon schools — Session 1 only audited Home/Visits/Drawer. Could surface more bugs.
- **CRM attribution row** ("added by …") — was on Session 1 list, deferred. Probably 1 session standalone.

### From Session 2 (2026-04-18)
- **S2-minor-1:** Session 2 introduced three new hex colors hardcoded in the `S2_ASSISTANT_COLORS` array (`#146E8A`, `#7A2E8F`, `#C24A1F`) to diversify non-Oregon assistant-coach avatars. If/when the design system tightens or dark-mode lands, these should be promoted to `--coach-accent-*` CSS variables. Low priority; currently only visible on Visits coach-chip strip and Ops activity avatars. ~15 min.
- **S2-minor-2:** Seasonally-aware "Next up" scheduling. Current `s2nextGame` generator (and the original Oregon `NEXT_GAME` constant) both produce November dates regardless of the actual calendar. During a real April/May demo, a beta tester could flag that the season's over. Fix: pick dates based on current month. Affects Oregon too. ~1 session.

### From Session 3 (2026-04-17, overnight) — KenPom backend
- **S3-minor-1:** KenPom data freshness. `scripts/kenpom_sync.py` in the `apex-intel` repo is idempotent and safe to re-run. Should be re-run weekly during the season (Oct–Apr) to pull updated ranks. Not yet wired to a cron job. If/when we want auto-refresh, candidate paths: Supabase Edge Function on a schedule, GitHub Action cron, local launchd plist. ~1 session to automate, or manual monthly re-run is fine for MVP.
- **S3-minor-2:** `scripts/kenpom_sync.py` uses `fuzz.ratio` (character-level) for Pass 3 fuzzy fallback, deliberately chosen over `fuzz.token_set_ratio` because the latter caused 6 wrong-row writes where short KenPom names like "Miami OH" scored 100 against longer DB names like "Miami". Pass 3 is currently unused (override + normalize-exact cover 365/365), but if KenPom adds new names we don't have overrides for, the script should still be safe. Documented here so future me doesn't "helpfully" switch back to token_set_ratio.
- **S3-minor-3:** `KENPOM_TO_SCHOOL_OVERRIDES` in `kenpom_sync.py` has 62 entries, hand-coded against the current `schools.name` set. If `schools` table is ever renamed/reloaded (e.g., a future data-vendor swap), these overrides may silently break. Defensive option: add a startup check that every override target EXISTS in `schools.name` and fail fast if not. ~15 min.
- **S3-minor-4:** 8 more `kenpom_*` columns added to `schools`. Session 4's frontend work will want a `schoolKenPomMap` / `kenpomFor(schoolName)` helper. Pattern mirrors existing `schoolEspnMap`. **RESOLVED Session 4a** — helper shipped, Home surfaces wired. **Also RESOLVED Session 4b** — recruiting cards + player modal wired.

### From Session 4a (2026-04-17) — KenPom frontend Home surfaces
- **S4a-minor-1:** `node` not installed on Mac. `node --check` syntax validation fails (`-bash: node: command not found`) during patch workflow. Session 4a/4b both fell back to Python regex brace/paren balance check against extracted `<script>` bodies, which worked — but a proper JS syntax check is a better safety net. Options: (a) `brew install node`, (b) switch to `esbuild --check` or similar Python-based JS parser, (c) keep the Python balance check and accept lighter guarantees. ~10 min for option (a).
- **S4a-minor-2:** `CURRENT_SCHOOL_NAME` global holds the full "Oregon Ducks" form. `schoolKenPomMap` + `schoolEspnMap` are keyed by the shorter `schools.name` form ("Oregon"). Session 4a worked around this by deriving the canonical name via `schools.filter(s=>s.id===CURRENT_SCHOOL_ID)[0].name` inside the Roster Pulse patch. Session 4b repeated the same pattern in `kenpomChipHTML`. Cleaner would be: store `CURRENT_SCHOOL_CANONICAL` alongside `CURRENT_SCHOOL_NAME`, updated in the same setters. Or key both maps off `id` not `name`. Session 4b also shipped a parallel `schoolKenPomById` map keyed by UUID, which partially addresses this. Not urgent but will repay itself across future sessions.
- **S4a-minor-3:** Session 4a left `index.html.s4a-backup-20260417-043201` in `~/code/Apex-App/` as untracked. Session 4b also left `index.html.s4b-backup-20260417-112411`. The `*.s4b-backup-*` gitignore pattern added in Session 4a's close DOES appear to catch both (Session 4b's `git status` at commit time showed neither backup as untracked). Consider broadening the pattern to `*-backup-*` or `index.html.*-backup-*` for future-proofing. ~5 min.
- **S4a-minor-4:** Oregon `On roster: 17` vs DATA-STATUS's 78 Oregon players in `my_roster`. Likely a filter upstream (probably active/starter-only). Related to B4; fix together.

### From Session 4b (2026-04-17) — KenPom frontend cards + modal
- **S4b-minor-1:** Chip size initially shipped at 10px, bumped to 11px after user feedback that it read too recessive next to 11px subtitle text. Final spec lives in `.kp-chip` CSS. Worth noting as a reference for future chip styling decisions — 11px is the "present but subordinate" size in this typography scale.
- **S4b-minor-2:** Patch-script anchor brittleness. Three rebuilds needed during 4b because (a) PATCH C expected 2 matches but only 1 exists in source (I conflated declaration and populate loops), (b) PATCH E expected 1 but 2 prospect-card renderers share the anchor string, (c) PATCH F used `\u00b7` escape form but source file stores literal U+00B7 middot character. All caught by idempotency guard before any write. Lesson: grep anchor counts BEFORE authoring `expect=` values in the patch script, not after. Not actionable — just a process note for future patch-heavy sessions.
- **S4b-minor-3:** `kenpomChipHTML(p, isPr)` takes an `isPr` boolean. When `isPr === false` (Oregon roster player viewing their own modal), the function derives the user's school via `schools.filter(s=>s.id===CURRENT_SCHOOL_ID)`. Uncovered edge case: if `CURRENT_SCHOOL_ID` ever becomes null or stale between renders, the chip silently hides. Defensive — not a bug today, but flag if we ever see roster modals missing the chip unexpectedly.
- **S4b-minor-4:** Variant B ("Committed: #X [school]") is code-reviewed but UNTESTED against live data. Zero `prospect_shortlist` rows have `committed_to_school_id` populated. When the named project "Prospect pipeline reconciliation" (B8 above) lands real data, re-check Oregon HS tab for the committed chip rendering correctly. No code changes expected — just live verification.
- **S4b-minor-5:** Prospect-school name normalization deferred (user chose "proceed with graceful hide" over "add normalizer in kenpomFor"). Edge cases observed in live data: `"Cal"` (should match "California"), `"USC"` (should match "Southern California"), `"Committed - Oregon"` (compound string, no KenPom match). Fix would be a ~20-line normalizer table in `kenpomFor` OR a data-side pass updating `current_school` to canonical names. Filed here for future data-cleaning session.

### Carryover from previous sessions
- Empty-state skeletons for seed-data screens (lower priority once dummy fallback ships in Session 2)
- CRM/Watchlist school-scoping (cross-school bleed) — currently user-scoped, should be school+user-scoped
- `prospect_contacts` real schema (replace `dummyContactsFor()` in `index.html`)
- `prospect_visits` real schema (replace `VISITS_SEED`)
- Bottom nav bar (mobile UX upgrade — drawer-only currently)
- Real-time roster builder UX polish
- Logo + subtitle font sizing pass (carryover from 5e)
- 9 Oregon players missing photos (ESPN data gaps): Darren Harris, Dezdrick Lindsay, Ege Demir, Kwame Evans Jr., Nikolas Khamenia, Sven Djopmo, Travelle Bryson, Tyler Lundblade, Win Miller. Manual photo_url from alt sources.

### Carryover from 4/16 wrap
- Apex Picks preset strip (now slated for Session 7)
- Default NIL budget per school (now slated for Session 8)
- Patrons data for non-Oregon Power 5 (now slated for Session 9)

### Repo hygiene
- **✅ DONE 2026-04-17 (Session 3):** Cleaned up uncommitted clutter in `~/code/apex-intel/` — `.env.backup`, `*.bak`, `*.backup` files gitignored + 2 real commits extracted (Scout 1.2 refactor, Bart Torvik ingestion script).
- **✅ DONE 2026-04-17 (Session 4a):** Added `*.s4a-backup-*` gitignore pattern in `~/code/Apex-App/`. Session 4b confirms it also catches `*.s4b-backup-*` via matching wildcard.
- Remaining: consider broadening the backup pattern to `index.html.*-backup-*` for future-proofing. ~5 min.
- Standard commit message format

---

## Ideas / explorations (not yet specced)

### Competitor research (incoming)
- Michael will send screenshots from a competitor in next session
- Action: review, extract integrate-able patterns, add to backlog with `competitor-inspired` tag

### Other API integrations not yet wired
- Synergy / Sportradar (commercial, paid) — would cost real money, eval before committing
- InStat / Hudl — film integration
- On3 NIL valuation feed — would replace Apex's internal NIL model with a third-party comp; or run alongside

### Mobile-app-style features
- "Add to Home Screen" PWA configuration (iOS web-app meta tags)
- Offline mode for travel / weak-signal
- Native iOS app eventually (out of scope for current architecture)

---

## Out of scope (not pursuing)

- Recruiting *services* (we're a tool, not a service)
- Player social media management (related but separate product surface)
- Coach education / playbook content (different audience)
