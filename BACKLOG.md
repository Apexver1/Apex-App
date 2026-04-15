# Apex Intel — Backlog

> Living document. Review at the start of every session alongside CLAUDE.md and the latest handoff.
> Last updated: 2026-04-15 (late session — Apex Patrons + Apex Ops added)

---

## Shipping next

1. **Scout 1.1.1 — Frontend rich rendering** — Read scout-1.1 structured response (tool_calls, iterations, schema_version). Render markdown answer with proper formatting (## headers as bold, **text** as bold). Render players from tool_calls[].output.results[] as rich cards instead of plain markdown. Fix "X players identified" counter. Add collapsible reasoning trace ("How Scout reasoned").

2. **CRM redesign as "Apex CRM"** — Full rewrite. Stars (1-5) replace tier A/B/C/D. Alphabetical phonebook layout (A-Z sections). Rich contact profiles (parents, AAU, HS coach, phone, email, social). Restore vCard import/export (existed, got removed). Log contact gets date/time picker. Coach assignment per relationship (1 or multiple). Dashboard surfaces smart suggestions ("time to text [5-star kid]").

3. **Apex Patrons + Apex Ops — demoable shells with dummy data** — Build both as navigable sections in the drawer with realistic seed data so they can be demoed immediately. Full feature specs in "Design needed" below. Goal: vision is visible in the product, full builds happen later. Estimate: 2-4 hrs each for shells.

4. **Player photos on every card** — Coaches scan by face before name. Scrape ESPN headshots into school_rosters.photo_url for D1; equivalent source for portal + HS. Render on: roster, portal, HS, SCOUT, CRM, Roster Builder, player modal. Initials fallback for missing.

5. **Universal "Add to CRM" button** — Every player card across the app gets a tap-to-add button that creates a prospect_shortlist entry and opens contact logging. CRM stops being a silo.

6. **Login splash + BETA label + clickable dashboard tiles** — Three small UX wins grouped: (a) post-login interstitial with logo + "Build a Champion" CTA, (b) replace "v2" with "BETA" on login page, (c) every home dashboard tile deep-links to its relevant tab.

---

## Queued

- **Portal player stats backfill** — 1,192 portal players have has_ppg=0. Run loader against BT 2024-25 (year=2025) and match by name only with high threshold (95+); portal players pre-portal D1 season has full stats. Expected 600-900 stats gained.

- **Build out "+ New" visit flow** — Button currently stubs "coming in Phase 5b." Needs form modal, prospect_visits table migration, itinerary add-item modal, file upload to Storage bucket.
- **Player detail modal centering** — Currently anchored to bottom of screen. Should be vertically centered.
- **Remove Weighting dropdown from coach scoring modal** — Four sliders + notes is enough.
- **Competing schools editable** — Chips are seed-data only. Add input/chip picker so coaches can add schools they're hearing about.
- **Log contact date/time picker** — Currently assumes "now." Coaches need to log past interactions.
- **NIL contract upload + AI parsing** — Upload accepted NIL agreement per commit slot (PDF). Claude parses contract → extracts payment schedule, appearance obligations, deliverables → auto-generates milestone reminders on dashboard. Platform-defining feature.
- **Roster data freshness** — ESPN scrape is Apr 13, players in portal/graduated still showing.
- **Transfer risk automation** — Currently hardcoded notes. Should derive from portal status, HS commit fluidity, NBA draft signals, or be coach-editable.
- **Apex Model / Market / Delta calibration** — Formula is rough proxy. Calibrate against deal comps or integrate with On3 NIL Valuation, Opendorse, INFLCR.
- **Edit D.4 — Schedule results** — ~2.5 hrs. New schedule_results table, pull 2025-26 results for 36 BT+ACC schools.
- **Star-cadence CRM reminders (Phase 6A.2)** — Approved Apr 10, never built. Ties into CRM redesign + dashboard smart suggestions.
- **Login debug for non-apex@test.com accounts** — stewartfam@mac.com, demo@apexintel.com, demo@test.com all fail.
- **Real rankings data** — Ranking columns currently mock-seeded from nil_estimate.
- **Revoke old Anthropic API key** — sk-ant-api03-oB6...c flagged Apr 10, still open.
- **Commit SQL migrations to /sql/ folder** — Reproducibility.

---

## Design needed before building

### Apex Patrons — full feature spec (donor / development / fundraising / communications)

Demoable shell ships in next session (see Shipping next #3). Full build is Phase 6-scale. Captured scope:

1. **Donor database** — who, how much, when, to what (collective / program / specific sport / specific purpose). Lifetime giving, YTD, largest gift, first/last gift dates.
2. **Prospect research** — find future donors by scraping LinkedIn, public web, social media for high-net-worth alumni, local business owners, corporate sponsors. Flag lapsed donors. Tier prospects by giving capacity.
3. **Relationship management** — every touchpoint logged (call, email, in-person, game attendance, dinner). Like recruit CRM but for donors. Assigned staff per relationship. Aging + auto-reminders for cold relationships.
4. **Event and game attendance tracking** — which donors attended which home/away games, dinners, golf outings. Auto-trigger outreach: "John hasn't been to a game in 6 months, offer free tickets" or "Sarah is in Tucson during Arizona game, invite to team dinner."
5. **Gifting and stewardship** — track what's been sent (signed ball, jersey, HC note, birthday card). Suggest stewardship actions by tier and lapse time.
6. **Campaigns** — capital campaigns, year-end appeals, NIL fundraisers. Track who was solicited, who gave, who's pending. Segment by tier for targeted asks.
7. **Communications manager** — bulk email/SMS to donor segments. Resend/SendGrid + Twilio integration. Templates with personalization.
8. **Travel and road game intelligence** — when team travels to a city, surface donors located there with last-contact + last-gift info. One-tap personalized invites.
9. **AI prospecting** — Apex Scout pattern for donors. "Find alumni in Portland with NW > $5M not yet approached." "Draft stewardship email to Mike Roberts — gave $50K last year, no contact in 4 months."
10. **Compliance dimension** — NCAA booster contact rules, recruit-donor interaction audit trail.

White space: nobody builds donor CRM specifically for college athletics in the NIL era. Raiser's Edge costs $50K+/yr and nobody uses it well. This is a flagship-tier feature.

### Apex Ops — recruiting materials hub

Demoable shell ships in next session (see Shipping next #3). Full spec:

1. **Recruiting decks** — upload PDF/Keynote/PPT per recruit, version history, creator + last-updated metadata
2. **Visit itineraries** — already partially in Scheduled visits, lives here as deliverable to print/share
3. **Scouting reports** — opponent breakdowns, pre-game preps, post-game film notes
4. **Family touchpoint materials** — letters from HC, photo collages from visits, personalized highlight reels
5. **NIL pitch packets** — what we offer, comparable deals, payment structure (pulls from NIL Budget data)
6. **Transfer portal reports** — staff-prepared deep dives beyond database
7. **Team materials** — light playbook/scout card storage (don't compete with Teamworks here)

Differentiators:
- Templates with auto-fill (recruit name, position, school context populates from database)
- AI-assisted draft generation ("Apex, draft a recruiting deck for Jalen Washington using our standard template, his stats, his family's interest in academics")
- Tracked sharing — unique link to recruit's family, see when opened + which pages viewed
- Permission-aware — ops staff edit, HC comments/approves, families view-only
- Searchable archive — "show every deck sent to wings in the last 2 years"

Scope: stays in recruiting lane (where Apex's center of gravity is). Avoids Teamworks competition. Gives ops staff (DOBO, GAs, video coordinator) a daily-use surface.

### Other design questions

- **Smart Filter advanced analytics integration** — Incorporate KenPom team context, Torvik player stats, Synergy play types. Unlocks "Portal PGs on bottom-50 KenPom teams" killer demo. Ties into Scout flagship upgrade.
- **staff table school_id column** — Multi-school architecture needs linkage for RLS. Blocks Phase 6 role-based access (which Apex Patrons + Apex Ops both depend on for permissions).
- **Oregon recruiting service access** — Which paid services does staff have? (On3 / 247 / Rivals).
- **Supabase secrets: Vault vs Edge Functions Secrets** — Standardize.
- **Decommit-history modeling** — portal_history table or overwrite on school_rosters?
- **AI Scout structured response format** — Currently regex-scans LLM text. Target: structured JSON for better card rendering.
- **Photo source for HS prospects and portal** — ESPN has D1 headshots; 247Sports has HS + portal photos but scraping rules TBD.
- **D2/D3 roster data source** — No source identified.

---

## Pre-commercial deployment gaps

Original framework from Apr 9. Status as of Apr 15:

| # | Gap | Status | Notes |
|---|-----|--------|-------|
| 1 | Data persistence | SHIPPED | Supabase backend, 7,526 players loaded |
| 2 | Auth / user roles | PARTIAL | Hardcoded auto-login. Phase 6 for per-coach roles. Blocks Apex Patrons + Apex Ops permissions. |
| 3 | Live transfer portal feed | PARTIAL | One-time 247 scrape. Daily refresh cron needed. |
| 4 | Push notifications | NOT STARTED | Portal alerts, cold contacts, deadlines. Critical for Apex Patrons (donor follow-up cadence). |
| 5 | Editable team analytics profiles | NOT STARTED | Strengths/weaknesses hardcoded. Blocks Scout flagship upgrade. |

---

## Data quality / schema polish

- schools.division cleanup (1,518 labeled D1; only 365 are D1 MBB)
- source_247_player_id column (ncaa_player_id overloaded)
- ~14 missing HS prospects (247 parser skips ad rows)
- ESPN '--' class_year stored as literal instead of NULL
- St. Francis PA override missing from MANUAL_OVERRIDES
- 54 player_type=NULL residue rows
- 76 duplicate school names
- schools.city/state NULL everywhere
- schools.ncaa_id / balldontlie_id NULL for 364/365 D1
- 33 portal players with NULL source school
- Mid-season coaching changes stale (UNC, Arizona State, Cincinnati, Kansas State, Boston College)
- Coach names beyond 79 Power 5 + Big East schools
- 948 D1 roster players unmatched by Bart Torvik loader (mostly Jr./II/III suffix mismatches; need name normalization in fuzzy matcher)
- 2 unmatched BT team names: FIU = Florida International, UMKC = University of Missouri-Kansas City; add MANUAL_OVERRIDES dict to loader
- Strategic brief v1.1 polish: remove EvanMiya from market map, soften pilot count language, add contact email

---

## Ideas / someday

- Daily 247 portal cron / GitHub Action
- Push notifications / SMS (Twilio + Resend/SendGrid)
- Two-way Google Calendar OAuth sync for visits
- Real On3/247/ESPN/Rivals nightly scrapers
- Compute On3 IRI in-house
- Auto-balance NIL budget to cap
- AD approval notification flow
- Historical budget snapshots
- Auto-create CRM log entry on roster builder status change
- Cache LLM responses per query
- Real-time CBB news feed (apex-news Edge Function)
- Predictive transfer risk model
- NBA draft probability on player cards
- Video annotation and clip tagging
- Multi-sport architecture (football / W-basketball)
- Parent/family read-only portal
- Conference group licensing as GTM channel
- NIL Collective partner login with compliance firewall
- Offline mode / PWA with Service Worker
- Synergy API deep integration
- cbbdata registration (needs R env)
- KenPom renewal reminder before April 1, 2027
- audit_log triggers on all tables
- docs/fork-guide.md placeholder fill
- Replace emoji icons with Lucide icon set
- Per-school team color theming
- Techstars Sports / Stadia Ventures applications

---

## Shipped recently

| Date | Item |
| Apr 15 (PM) | Scout 1.1: agentic Edge Function + 4 tools + 4,122 D1 players with Bart Torvik advanced stats |
|------|------|
| Apr 15 | D.1: Coach greeting fix for non-Power 5 schools (d301b99) |
| Apr 15 | Password gate on school picker restored (38956ec) |
| Apr 15 | BACKLOG.md + CLAUDE.md reference + Apex Patrons/Ops captured |
| Apr 14 | Edit C: auto-login bootstrap (6a87de4) |
| Apr 14 | Edit D.2: coach names for 79 Power 5 + Big East schools |
| Apr 14 | Edit B: school picker D1 filter + chips + search (d6dd870) |
| Apr 14 | Edit B.1: server-side preload filter (b6894c5) |
| Apr 14 | Edit A: school-switching + materialize RPC (596b1e0) |
| Apr 14 | v22: rankings, visit calendar, NIL sliders, roster builder 18-slot |
| Apr 13 | Phase 4d: 1,192 portal players (247Sports) |
| Apr 13 | Phase 4c: 657 HS prospects (247Sports Composite) |
| Apr 13 | Phase 4b: 5,677 D1 players / 365 schools (ESPN) |
| Apr 13 | Phase 4a: Oregon roster + ESPN ID backfill |
| Apr 13 | CLAUDE.md + handoff system |
| Apr 11 | apex-scout Edge Function (Claude LLM + rules) |

---

## Conventions

- **Location:** ~/code/Apex-App/BACKLOG.md (frontend repo root)
- **Review:** Start of every session alongside CLAUDE.md and latest handoff.
- **Updates:** When an item ships, move to "Shipped recently" with date. New ideas go to appropriate section. 1-3 lines per item (full specs go in "Design needed" if longer).
- **Shipped log:** Rolling ~15 items.
- **Priority:** "Shipping next" should hold 5-7 strategic priorities. Promote from "Queued" when one ships.
- **Cross-reference:** Note handoff filename when detailed spec exists elsewhere.
