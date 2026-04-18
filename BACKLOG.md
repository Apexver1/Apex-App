# BACKLOG.md — Deferred work, named projects, ideas

**Updated:** 2026-04-17 (after Session 5 Torvik end-to-end + Sammy pitch feedback integration)
**Updated by:** Claude when items get added or moved
**Promotion rule:** Items move from backlog into the active 14-day plan only by explicit decision in a session. Do not silently slip them in.

---

## Named projects (multi-session efforts)

### School-scope the global queries  ← from Session 1 audit
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

### Prospect pipeline reconciliation  ← from Session 4b recon
**Estimated scope:** 1–2 sessions
**Why it matters:** DATA-STATUS previously claimed 657 HS + 1,192 portal = 1,849 prospects in `prospect_shortlist`, but live DB has only 100 rows — all with `school_id=Oregon` regardless of their `current_school` text. UI renders fine (chip graceful-hide handles the junk values), but the product "feels" more Oregon-centric than it should for any pitch where "we have 1,800+ prospects" would be said out loud. Also blocks Variant B chip testing (Committed: #X [school]) since no rows have `committed_to_school_id` populated.
**Status:** Identified Session 4b (bug B8). Data-backfill problem, not a code bug.

**Full scope:**
- Reconcile 247Sports Composite ingestion — verify where the 657 HS + 1,192 portal rows went
- Decide whether `prospect_shortlist.school_id` should be the interested school (Oregon) or the prospect's current school
- Standardize `current_school` column — never raw strings like "Committed - Oregon", always clean school-name form
- Populate `committed_to_school_id` for committed portal players (unlocks Variant B chip testing)

---

## Named projects — NEW from Sammy Gelfand pitch (2026-04-17 transcript)

Six items surfaced from the Sammy Gelfand pitch conversation on 2026-04-17. All new named projects, filed for future scoping. Do not promote into the 14-day plan without explicit decision.

### Program-fit scoring  ← Sammy pitch 2026-04-17
**Estimated scope:** 3–5 sessions (requires data science + historical modeling)
**Why it matters:** Sammy's biggest strategic critique of the college recruiting landscape is that schools pay for the **ranking number** (On3 / 247 / ESPN) instead of the **player**. Every other coach is optimizing the same ranking-based board, so paying top-dollar for a top-ranked kid is a commodity play. The edge is identifying players whose *skillset + style + system fit* the program better than the ranking implies. Quoting Sammy: *"everything is by the rankings, the On3, the ESPN, the NIL, and so all they're doing is paying for the number, like, the ranking number, and not actually for the player"* — and separately: *"Fit your system, fit your players, fit your coaches — and I wonder if there's a way of like... diamonds in the rough. These are guys that, you know, from smaller schools have shown some attributes to be better at moving up. Or guys who have struggled at maybe bigger schools that just need to change a system."*
**Status:** Concept logged. Requires a real modeling effort — not a single-session build.

**Full scope (when built):**
- Define program profile per school: pace (Torvik tempo), shooting-vs-paint-attack mix (eFG% / 3PA% / rim rate from player stats), defensive intensity (Torvik AdjDE + TOV%), coaching system tags (motion / ball-screen / zone)
- Model player → program fit as a vector distance between the player's statistical profile (usage, shot distribution, defensive role) and the program's historical profile of players who succeeded there
- Training signal: historical transfers where the player's destination-school production **exceeded** their source-school production vs. where it regressed
- Surface as a score on every prospect card — e.g., "Fit: A- (87)" with a 1-line explanation ("similar usage profile to Ajay Mitchell, 2023 OBB transfer success")
- Sammy's framing: pair this with NIL valuation so the pitch becomes "we think this player will produce above his ranking — that's why we're paying his market"

### Recruitment likelihood indicators (Intelligence panel)  ← Sammy pitch 2026-04-17
**Estimated scope:** 2–3 sessions
**Why it matters:** Sammy's direct ask during the pitch: a front-and-center read on "who's actually coming" vs. "who's a long shot". Quoting: *"we're top 3 with this kid, or we're, like, you know, it's a long shot... if we give this guy 600K, he's coming. He loves Nike, he wants to meet Tinker Hatfield, he's a shoe head."* And later: *"hey, look, we love Padinga, but, like, he's an Adidas kid. We're competing against some, like, the Blue Bloods, probably a long shot... we should probably start working on, like, backup plans, versus, like, counting on him."* The current app has a "competing schools" field but no actual *likelihood* output.
**Status:** Concept logged. Michael's on-the-call response: "an intelligence feature on the player card is not a bad idea... maybe called Intelligence."

**Full scope (when built):**
- New "Intelligence" section on every player card — not a single field, a living panel
- Inputs: coach-entered notes ("he wants to stay home", "his mom doesn't like the cold"), competing-schools count + quality, shoe-company alignment (see separate project), priority-tier vs. our competing-schools ranks, AAU connections, recent CRM touchpoints
- Output: traffic-light style signal — **Green** (top 3, high-probability close), **Yellow** (in the mix, needs push), **Red** (long shot, start backup plans)
- Each signal has a 1-line rationale pulled from the inputs
- On the home dashboard, an "Intelligence meter" summary: "5 green, 8 yellow, 3 red across your active watchlist"
- Allows the coach to assign NIL resources rationally — don't overpay a Green, don't underpay a Yellow we can still flip

### Shoe company affiliations  ← Sammy pitch 2026-04-17
**Estimated scope:** 2 sessions (data + UI)
**Why it matters:** Sammy's direct lift from his NBA scouting background applied to college recruiting: shoe-company alignment creates gravitational pull in recruiting. Quoting: *"this guy was a Nike guy through, like, Bidunga, right? Adidas all the way through, went to Louisville, like, kind of stayed in the Adidas family... guys like, hey, they've been kind of like Under Armour or Adidas, like, you know, we gotta see if we can get them to flip, but maybe they're a little bit lower on the priority, knowing it's a little bit longer of a shot, or we may need to up the money to convince them."* Sammy attributed the insight to D. West: *"that would be, like, the first thing to, like, look at is: get connections with all those shoe companies, and that'll help you with the players."*
**Status:** Concept logged. Practical, achievable, high-value. Easy first-order win.

**Full scope (when built):**
- New column on players: `shoe_affiliation` (Nike / Jordan / Adidas / Under Armour / Puma / New Balance / Unsigned)
- Derived from: AAU program (Nike EYBL, Adidas 3SSB, Under Armour UA Rise, etc.), explicit NIL deals, HS apparel contract
- New column on schools: `shoe_affiliation` (Oregon = Nike, Louisville = Adidas, etc.)
- Match signal on player cards: **Match** (player shoe = school shoe), **Mismatch** (flip required), **Unsigned** (green field)
- Feeds into Program-fit scoring AND Recruitment likelihood as an input
- Smart Filter: add a "Shoe fit" filter (Match / Mismatch / All)
- Sammy's example: Oregon is Nike, so a Nike-aligned player like Bidunga has a gravitational pull advantage; an Adidas-aligned player requires "flip the family" plus higher NIL to overcome the gravity

### Transfer success models  ← Sammy pitch 2026-04-17
**Estimated scope:** 4–6 sessions (data science heavy)
**Why it matters:** Sammy's most technically-ambitious suggestion, straight from his NBA scouting work. Quoting: *"I do think the other probably last thing I think is important is just, like, the historical study of these transfers. Right? Like, why are these transfers failing? Who are the guys that are red flag transfers versus who are the guys that are green flag transfers, historically."* And later, mid-explanation: *"like AJ Storr, great player, Wisconsin, goes to Kansas, and it's just a complete disaster. You know, guys who switch conferences, right? Like, if you go from the Big 12 to the Big Ten, are you just not a good fit for that?... Is A-10 guys better in the Big East, but not good in the Big 12? Because I think there's a lot of stylistic differences in the conferences, at least when I watch it as a scout."*
**Status:** Concept logged. Genuine product differentiator if built well.

**Full scope (when built):**
- Dataset: historical D1 transfers (last 5 seasons minimum) — source → destination, pre-transfer stats, post-transfer stats, conference pair, style-pair, coach-change indicator
- Labels: **Green flag** (post-transfer production ≥ pre-transfer, ≥ 24 mpg sustained), **Red flag** (post-transfer production regressed by >20% or transferred again within 1 season)
- Features to model: conference-pair (Big 12 → Big Ten, A-10 → Big East, etc.), usage-rate delta, shot-profile delta, team-pace delta, coaching-system pair, player physical (height / wingspan), age at transfer
- Output: per-transfer-prospect card — "Green flag: 73% confidence — 18 similar historical transfers, 13 succeeded" OR "Red flag: conference-switch + style-mismatch, 22% confidence"
- Feeds Program-fit scoring + NIL valuation — a Red-flag transfer should be priced lower regardless of raw ranking
- Requires actual data science work — this is not a single-session build and may need a dedicated modeling effort + Supabase `transfer_outcomes` table

### Donor preference profiles  ← Sammy pitch 2026-04-17
**Estimated scope:** 2–3 sessions (integrates into Apex Patrons)
**Why it matters:** Sammy's critique that donor development is currently just "get their money" when it should be "understand what they want to give money to". Quoting: *"each donor has stuff that they care about. So let's say Phil Knight... doesn't care about what players he uses his money for, but where his passion is, is he wants to have the nicest arena ever. Versus, like, the second biggest donor, he wants a top 10 recruit. He'll give you as much money as you need, but he wants a top 10 recruit."* And the frame: *"the more I talk to teams of finding the pressure points for each of the patrons and the donors of what they really care about... finding the pressure points of the donors of what they want to give their money to sometimes is just as important as just getting their money."* Contrasting Notre Dame example: *"Pat Garrity in Notre Dame, their donors don't care about getting players as much. When they donate, it's more to the arena, it's more to just the organization."*
**Status:** Concept logged. Extends existing Apex Patrons data model.

**Full scope (when built):**
- New fields on patrons: `preferred_allocation` (multi-select: NIL pool / facilities / arena / annual fund / specific player NIL), `giving_trigger` (free text: "wants top-10 recruit", "wants nicest arena", "cares about academic outcomes"), `preferred_recognition` (naming rights / courtside seats / food on road / etc.)
- Per-donor view: "Pat Kilkenny · wants a star · $5M available" alongside current-year giving and lifetime total
- Donor-player matching is a separate project (see below) that uses these fields as inputs

### Donor-player matching  ← Sammy pitch 2026-04-17
**Estimated scope:** 2–3 sessions (builds on Donor preference profiles)
**Why it matters:** Sammy's concrete operational example: *"Pat Kilkenny wants Jucares. He fucking loves Jucares. He'll give you $5 million for Jucares."* The idea is that if we know donor preferences and player personalities/backgrounds, we can pitch specific players to specific donors — maximize dollar-per-player by matching the right donor to the right recruiting target. This allows redistributing funds: *"That allows you to then redistribute your funds from other guys to maybe other things, you know, when you know kind of what these donors really want to give money to."*
**Status:** Concept logged. Requires Donor preference profiles + an enriched player-background data model.

**Full scope (when built):**
- Enrich player cards with: hometown, academic interests, personal story hooks (first-gen college, community work, specific religious or cultural affiliation, HS connections)
- Enrich donor profiles with: alma mater preference, position preference, style preference (rim-protector-lover vs. pure-scorer-lover), hometown connections
- Match engine: for each unsigned prospect on the watchlist, rank current donors by match score
- Output UI: on a player card, a "Pitch this player to:" list — top 3 donors likely to fund, each with a 1-line rationale ("Pat Kilkenny — loves local Oregon kids, has funded 3 in-state recruits in last 2 years")
- Closes the loop on donor development — gives the coach a direct fundraising playbook, not just a donor list

---

## Smaller items (single-session or less)

### From Session 1 audit
- **B3:** Howard `head_coach_name` is NULL → drawer shows blank name + "Coach." greeting with no last name. ~30 min.
- **B4:** Avg PPG = 0.0 on non-Oregon schools in Roster Pulse. Either `depth_chart_position` is unset post-materialization or PPG fields are NULL. Investigation + fix probably 1 session.
- **B6:** School picker doesn't find "Saint Mary's" when searching "st." — needs alias-aware search or data-side normalization. ~30 min.
- **B7:** Avatar bg color isn't school-themed. ~1 session if we want to do it across all 365 schools.
- **Defer-from-S1:** Per-screen audit of Roster, Recruiting, Roster Builder, Smart Filter, Apex Scout across non-Oregon schools.
- **CRM attribution row** ("added by …") — was on Session 1 list, deferred. ~1 session standalone.

### From Session 2
- **S2-minor-1:** Session 2 hex colors hardcoded in `S2_ASSISTANT_COLORS`. Promote to `--coach-accent-*` CSS variables. ~15 min.
- **S2-minor-2:** Seasonally-aware "Next up" scheduling. ~1 session.

### From Session 3 — KenPom backend
- **S3-minor-1:** KenPom data freshness. `scripts/kenpom_sync.py` should be re-run weekly during the season. Candidate paths: Supabase Edge Function cron, GitHub Action cron, local launchd plist. ~1 session to automate.
- **S3-minor-2:** `scripts/kenpom_sync.py` deliberately uses `fuzz.ratio` (not `token_set_ratio`). Documented here so future me doesn't "helpfully" switch back.
- **S3-minor-3:** `KENPOM_TO_SCHOOL_OVERRIDES` has 62 entries, hand-coded against current `schools.name` set. Defensive option: add a startup check that every override target EXISTS in `schools.name`. ~15 min.

### From Session 4a — KenPom frontend Home surfaces
- **S4a-minor-1:** `node` not installed on Mac. Options: (a) `brew install node`, (b) switch to Python-based JS parser, (c) keep Python balance check. ~10 min for (a).
- **S4a-minor-2:** `CURRENT_SCHOOL_NAME` holds "Oregon Ducks" form; `schoolKenPomMap` keyed by shorter "Oregon" form. Cleaner: store `CURRENT_SCHOOL_CANONICAL` or key both maps off `id`. Not urgent.
- **S4a-minor-3:** `*-backup-*` gitignore pattern working ✓ (Session 5 confirmed).
- **S4a-minor-4:** Oregon `On roster: 17` mismatches DATA-STATUS's documented 78 Oregon players. Likely starter/active filter upstream of Roster Pulse. May share root cause with B4.

### From Session 4b — KenPom frontend recruiting/modal
- **S4b-minor-1:** Chip size tweaked 10px → 11px mid-session. Baked in.
- **S4b-minor-2:** PATCH anchor discipline — grep `-c` the anchor count BEFORE building every patch. Documented lesson.
- **S4b-minor-3:** Literal U+00B7 middot vs. `\u00b7` escape form on disk. Always probe byte form before regex anchors on HTML/JS source.
- **S4b-minor-4:** Variant B chip ("Committed: #X [school]") NOT verified live — zero `prospect_shortlist` rows have `committed_to_school_id` populated. Unblocks once B8 (Prospect pipeline reconciliation) lands.

### From Session 5 — Torvik end-to-end
- **S5-minor-1:** Three Torvik columns (`torvik_efg_pct`, `torvik_efg_d_pct`, `torvik_tov_pct`) exist in the `schools` schema but are NULL. The Torvik JSON column indexes for these three were not verifiable with three team cross-references (Duke / Arizona / Oregon) — directions didn't clean-match the "lower = better D for opp eFG%" intuition. To avoid shipping wrong stats, we left them NULL. Fix: cross-reference against 5+ teams using known values from [barttorvik.com](https://barttorvik.com) to pin down correct indices, then update `TT_COL` and re-run `scripts/torvik_team_sync.py`. ~30 min.
- **S5-minor-2:** `scripts/torvik_team_sync.py` override table has 60+ entries, many "defensive" no-ops pointing `X -> X`. Could be pruned. Harmless but noisy. ~10 min.
- **S5-minor-3:** `trank.php?csv=1` endpoint returns HTML not CSV (breaks the obvious Torvik team-data path). Session 5 solved this by switching to `/2026_team_results.json` which is an undocumented but stable endpoint. If Torvik ever removes this endpoint or changes its shape, `torvik_team_sync.py` breaks silently. Defensive: add a startup shape-check (column count, first-team name = rank-1 team, etc.) that fails loudly. ~15 min.
- **S5-minor-4:** Torvik freshness — like KenPom, should re-run weekly during season. Candidate: unify with S3-minor-1 into a single "data-refresh cron" project. ~1 session to automate both at once.

### From Session 5c — Smart Filter Torvik tier
- **S5c-minor-1:** Button labels for Torvik tier use mixed case (`All`, `Top 50`, `#51-150`, `#151+`) while other filter sections (Position, Priority tier) use ALL CAPS (`ALL`, `PG`, `A`, `UNTIERED`). Small visual inconsistency. Fix: update labels in the `[["ALL","All"]...]` array in the P6 UI section to all-caps. ~1-char fix per button label, ~5 min.
- **S5c-minor-2:** Smart Filter now has TWO rank-based filters that can both be active simultaneously: Rankings (On3 player rank) + Torvik tier (school rank). Visual grouping is correct (adjacent sections) but there's no explicit UX hint that they filter on DIFFERENT things. A 1-line subhead under "RANKINGS" saying "Player's On3 national rank" (mirroring the Torvik subhead "School's current Torvik rank") would close the loop. ~5 min.

### Carryover from previous sessions
- Empty-state skeletons for seed-data screens
- CRM/Watchlist school-scoping (cross-school bleed)
- `prospect_contacts` real schema (replace `dummyContactsFor()`)
- `prospect_visits` real schema (replace `VISITS_SEED`)
- Bottom nav bar (mobile UX upgrade)
- Real-time roster builder UX polish
- Logo + subtitle font sizing pass (carryover from 5e)
- 9 Oregon players missing photos: Darren Harris, Dezdrick Lindsay, Ege Demir, Kwame Evans Jr., Nikolas Khamenia, Sven Djopmo, Travelle Bryson, Tyler Lundblade, Win Miller

### Carryover from 4/16 wrap
- Apex Picks preset strip (Session 7)
- Default NIL budget per school (Session 8)
- Patrons data for non-Oregon Power 5 (Session 9)

### Repo hygiene
- **✅ DONE 2026-04-17 (Session 3):** Cleaned `~/code/apex-intel/` — gitignored `.env.backup`, `*.bak`, `*.backup`; 2 commits extracted.
- **✅ DONE 2026-04-17 (Session 4a):** Added `*-backup-*` pattern to Apex-App `.gitignore`. Confirmed Session 5 that `index.html.s5b-backup-*` files don't appear in `git status`. Working.
- **✅ DONE 2026-04-17 (Session 5):** Removed stray empty `main` file from `~/code/Apex-App/` root (0-byte file from accidental redirect earlier in day).
- Standard commit message format — loose convention adopted (`feat(scope): Session Nx — summary`). Works well enough; no enforcement.

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

### Sammy pitch — Apex Finder (donor-scraping)
- Sammy's demo-era interest confirmed: the "Apex Finder at the very bottom" (LinkedIn / internet scraping for Pacific-NW UO-alum business owners) is interesting but currently a stub. Revisit after Donor preference profiles lands — scraping makes more sense when there's somewhere specific to put the findings.

---

## Out of scope (not pursuing)

- Recruiting *services* (we're a tool, not a service)
- Player social media management (related but separate product surface)
- Coach education / playbook content (different audience)
