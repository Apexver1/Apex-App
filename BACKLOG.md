# BACKLOG.md — Deferred work, named projects, ideas

**Updated:** 2026-04-17
**Updated by:** Claude when items get added or moved
**Promotion rule:** Items move from backlog into the active 14-day plan only by explicit decision in a session. Do not silently slip them in.

---

## Named projects (multi-session efforts)

### School-scope the global queries  ← NEW from Session 1 audit
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
**Status:** Pulled once in Phase 4 (1,192 entries). No live refresh wired.

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
- **B4:** Avg PPG = 0.0 on non-Oregon schools in Roster Pulse. Either `depth_chart_position` is unset post-materialization or PPG fields are NULL. Investigation + fix probably 1 session.
- **B6:** School picker doesn't find "Saint Mary's" when searching "st." — needs alias-aware search or data-side normalization. Cosmetic but a real UX issue. ~30 min.
- **B7:** Avatar bg color isn't school-themed. Map school primary color → avatar bg. Pleasant polish. ~1 session if we want to do it across all 365 schools.
- **Defer-from-S1:** Per-screen audit of Roster, Recruiting, Roster Builder, Smart Filter, Apex Scout across non-Oregon schools — Session 1 only audited Home/Visits/Drawer. Could surface more bugs.
- **CRM attribution row** ("added by …") — was on Session 1 list, deferred. Probably 1 session standalone.

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
- `.gitignore` for `.DS_Store`, `*.bak*`, `*.sb-*` (Sublime swap files), `*.rtf`
- Clean up uncommitted clutter in `~/code/Apex-App/`
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
