# Apex Intel — Backlog

> Living document. Review at the start of every session alongside CLAUDE.md and the latest handoff.
> Last updated: 2026-04-15

---

## Shipping next

1. **Apex Scout flagship upgrade** — This is the moat. Strip rules/AI toggle (always LLM). Natural-language placeholder. Voice input working. Rich player-card responses (not plain text). Enhanced AI reasoning with stat citations. Multi-turn conversation with context persistence. Full data access: all players + team strategy + CRM state + KenPom + Torvik + Synergy (when wired).

2. **CRM redesign as "Apex CRM"** — Full rewrite. Stars (1-5) replace tier A/B/C/D. Alphabetical phonebook layout (A-Z sections). Rich contact profiles (parents, AAU, HS coach, phone, email, social). Restore vCard import/export (existed, got removed). Log contact gets date/time picker. Coach assignment per relationship (1 or multiple). Dashboard surfaces smart suggestions ("time to text [5-star kid]").

3. **Player photos on every card** — Coaches scan by face before name. Scrape ESPN headshots into school_rosters.photo_url for D1 roster players; equivalent source for portal + HS prospects. Render on: roster list, portal cards, HS cards, SCOUT results, CRM contacts, Roster Builder slots, player detail modal. Initials avatar fallback for missing photos.

4. **Universal "Add to CRM" button** — Every player card in roster, portal, HS, SCOUT results gets a tap-to-add button that creates a prospect_shortlist entry and opens contact logging. CRM stops being a silo.

5. **Login splash screen + BETA label** — After login, interstitial page with Apex Intel logo + "Build a Champion" CTA button, then dashboard. Replace "v2" with "BETA" on login page.

6. **Clickable dashboard tiles** — Every tile on home deep-links to relevant tab: Open scholarships → Roster Builder, NIL requests → NIL Budget, CRM follow-ups → CRM, Portal churn → Recruiting, Visits → Scheduled visits.

---

## Queued

- **Build out "+ New" visit flow** — Button currently stubs "coming in Phase 5b." Needs form modal, prospect_visits table migration, itinerary add-item modal, file upload to Storage bucket.
- **Player detail modal centering** — Currently anchored to bottom of screen. Should be vertically centered.
- **Remove Weighting dropdown from coach scoring modal** — Four sliders + notes is enough. Presets add complexity coaches won't use.
- **Competing schools editable** — Chips are seed-data only. Add input/chip picker so coaches can add schools they're hearing about.
- **Log contact date/time picker** — Currently assumes "now." Coaches need to log past interactions ("called yesterday").
- **NIL contract upload + AI parsing** — Upload accepted NIL agreement per commit slot (PDF). Claude parses contract → extracts payment schedule, appearance obligations, deliverables → auto-generates milestone reminders on dashboard. Platform-defining feature.
- **Roster data freshness** — ESPN scrape is Apr 13, players in portal/graduated still showing. Need either re-scrape cadence or cross-reference against 247 portal to flag departures.
- **Transfer risk automation** — Currently hardcoded notes. Should derive from portal status, HS commit fluidity, NBA draft signals, or be coach-editable.
- **Apex Model / Market / Delta calibration** — Formula is rough proxy. Needs calibration against known deal comps, or integration with real NIL market data (On3 NIL Valuation, Opendorse, INFLCR).
- **Edit D.4 — Schedule results** — ~2.5 hrs. New schedule_results table, pull 2025-26 results for 36 BT+ACC schools, wire expandable "View 2025-26 results" on Next up widget.
- **Star-cadence CRM reminders (Phase 6A.2)** — Approved Apr 10, never built. Ties into CRM redesign + dashboard smart suggestions.
- **Login debug for non-apex@test.com accounts** — stewartfam@mac.com, demo@apexintel.com, demo@test.com all fail. Root cause never captured.
- **Real rankings data** — ranking columns currently mock-seeded from nil_estimate. Need real scraper or manual load.
- **Revoke old Anthropic API key** — sk-ant-api03-oB6...c flagged Apr 10, still open.
- **.gitignore for backup files** — index.html.pre-* cluttering git status.
- **Commit SQL migrations to /sql/ folder** — Reproducibility.

---

## Design needed before building

- **Smart Filter advanced analytics integration** — Incorporate KenPom team context (AdjO/AdjD/tempo), Torvik player stats (TS%, BPM, usage), Synergy play types (PnR PPP, spot-up, transition — gated on enterprise API). Unlocks killer filter demos like "Portal PGs on bottom-50 KenPom teams." Ties into Scout flagship upgrade.
- **staff table school_id column** — 4 rows, no school affiliation. Multi-school architecture needs linkage for RLS. Blocks Phase 6 role-based access.
- **Oregon recruiting service access** — Which paid services does staff have? (On3 / 247 / Rivals). Drives portal + HS data refresh strategy.
- **Supabase secrets: Vault vs Edge Functions Secrets** — Need to standardize. Currently apex-scout uses EF secrets.
- **Decommit-history modeling** — portal_history table for tracking status over time, or overwrite on school_rosters?
- **AI Scout structured response format** — Currently regex-scans LLM text. Target: structured JSON {picks: [{prospect_id, fit_score, reasoning}]}. Enables better card rendering.
- **Photo source for HS prospects and portal** — ESPN has D1 headshots; 247Sports has HS + portal photos but scraping rules TBD. On3 also has portal headshots.
- **D2/D3 roster data source** — No source identified. Low priority unless user base expands.

---

## Pre-commercial deployment gaps

Original framework from Apr 9. Status as of Apr 15:

| # | Gap | Status | Notes |
|---|-----|--------|-------|
| 1 | Data persistence | SHIPPED | Supabase backend, 7,526 players loaded |
| 2 | Auth / user roles | PARTIAL | Hardcoded auto-login (apex@test.com). Phase 6 for per-coach roles. |
| 3 | Live transfer portal feed | PARTIAL | One-time 247 scrape. Daily refresh cron needed. |
| 4 | Push notifications | NOT STARTED | Portal alerts, cold contacts, deadlines. Dashboard smart suggestions is a prerequisite. |
| 5 | Editable team analytics profiles | NOT STARTED | Strengths/weaknesses hardcoded in JS. Need per-school editable profile via team_strategy table. Blocks Scout flagship upgrade. |

---

## Data quality / schema polish

- **schools.division cleanup** — All 1,518 rows labeled D1; only 365 are D1 MBB. Update based on espn_id presence.
- **source_247_player_id column** — ncaa_player_id currently overloaded. Add dedicated column.
- **~14 missing HS prospects** — 247 parser skips ad-interrupted rows.
- **ESPN '--' class_year** — Parser stores literal instead of NULL.
- **St. Francis PA override** — Add to MANUAL_OVERRIDES in backfill_espn_ids.py.
- **54 player_type=NULL residue rows** — From early phases.
- **76 duplicate school names** — Deduped client-side only.
- **schools.city/state** — NULL for all rows.
- **schools.ncaa_id / balldontlie_id** — NULL for 364/365 D1 schools.
- **33 portal players with NULL source school** — Non-D1 source schools not in table.
- **Mid-season coaching changes stale** — UNC, Arizona State, Cincinnati, Kansas State, Boston College.
- **Populate coach names beyond 79 Power 5 + Big East schools** — Mid-major schools show generic "Coach." greeting.

---

## Ideas / someday

- Daily 247 portal cron / GitHub Action (until May 1 close)
- Push notifications / SMS (Twilio + Resend/SendGrid via Edge Functions)
- Two-way Google Calendar OAuth sync for visits
- Real On3/247/ESPN/Rivals nightly scrapers
- Compute On3 IRI in-house
- Auto-balance NIL budget to cap
- AD approval notification flow
- Historical budget snapshots (season over season)
- Auto-create CRM log entry on roster builder status change
- Cache LLM responses per query
- Real-time CBB news feed (apex-news Edge Function, CBS Sports RSS)
- Predictive transfer risk model (trained on historical data)
- NBA draft probability on player cards
- Video annotation and clip tagging in-platform
- Multi-sport architecture (football / W-basketball under one AD login)
- Parent/family read-only portal
- Conference group licensing as GTM channel
- NIL Collective partner login with compliance firewall
- Offline mode / PWA with Service Worker
- Synergy API deep integration (enterprise pricing)
- cbbdata (Andrew Weatherman) registration (needs R environment)
- KenPom renewal reminder before April 1, 2027
- audit_log triggers on all tables
- docs/fork-guide.md placeholder fill
- Replace emoji icons with Lucide icon set
- Browser Contact Picker API for CRM import
- Per-school team color theming (CSS hooks wired, not populated)
- Techstars Sports / Stadia Ventures applications

---

## Shipped recently

| Date | Item |
|------|------|
| Apr 15 | D.1: Coach greeting fix for non-Power 5 schools |
| Apr 15 | Password gate on school picker restored (stewart2026) |
| Apr 15 | BACKLOG.md + CLAUDE.md reference update |
| Apr 14 | Edit C: auto-login bootstrap (6a87de4) |
| Apr 14 | Edit D.2: coach names for 79 Power 5 + Big East schools |
| Apr 14 | Edit B: school picker D1 filter + conference chips + search (d6dd870) |
| Apr 14 | Edit B.1: server-side preload filter (b6894c5) |
| Apr 14 | Edit A: school-switching + materialize RPC (596b1e0) |
| Apr 14 | v22: rankings, visit calendar, NIL sliders, roster builder 18-slot |
| Apr 13 | Phase 4d: 1,192 portal players (247Sports) |
| Apr 13 | Phase 4c: 657 HS prospects (247Sports Composite) |
| Apr 13 | Phase 4b: 5,677 D1 players / 365 schools (ESPN) |
| Apr 13 | Phase 4a: Oregon roster + ESPN ID backfill |
| Apr 13 | CLAUDE.md + handoff system |
| Apr 11 | apex-scout Edge Function (Claude LLM + rules) |
| Apr 10 | v3-v21: war room, CRM, vCard, budget, coach scorecard, school picker |
| Apr 9 | Phase 2 schema + Phase 3 RLS + API keys |

---

## Conventions

- **Location:** ~/code/Apex-App/BACKLOG.md (frontend repo root)
- **Review:** Start of every session alongside CLAUDE.md and latest handoff.
- **Updates:** When an item ships, move to "Shipped recently" with date. New ideas go to appropriate section. 1-3 lines per item.
- **Shipped log:** Rolling ~15 items. Archive older entries if list grows.
- **Priority:** "Shipping next" always has 3-5 items. Promote from "Queued" when one ships.
- **Cross-reference:** Note handoff filename when detailed spec exists elsewhere.
