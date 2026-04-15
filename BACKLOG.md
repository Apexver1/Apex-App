# Apex Intel — Backlog

> Living document. Review at the start of every session alongside CLAUDE.md and the latest handoff.
> Last updated: 2026-04-15

---

## Shipping next

1. **Edit D.1 — "Coach Coach" greeting fix** — 5 min code patch. When head_coach_name is NULL, greeting shows "Coach Coach." Fix fallback to school name or bare "Coach." Lines ~569 + ~1370 in index.html.

2. **Edit D.3 — "Next up" widget empty state** — 30 min. Hardcoded Oregon vs Arizona shows for every school. Replace with "Season complete · 2026-27 schedule TBD" banner. Option 2 approved Apr 14 session 5.

3. **Verify SCOUT + password gate post-redesign** — 15 min. Grep index.html for apex-scout and APEX_PW/stewart2026 to confirm both survive v22 redesign. Test live at the deployed URL.

4. **Phase 5a stack decision + MVP build** — THE BLOCKER on any pitch or pilot. Demo target: school dropdown, roster view, portal/scouting grid, killer filter (e.g. "Portal PGs on bottom-50 KenPom teams"). Options: (a) Streamlit (fast Python, laptop demos), (b) keep iterating Apex-App/index.html (already wired to Supabase). See "Design needed" section.

---

## Queued

- **Edit D.4 — Schedule results table + frontend** — ~2.5 hrs. New schedule_results table, pull 2025-26 results for 36 BT+ACC schools, wire expandable "View 2025-26 results" on Next up widget. Spec written Apr 14.
- **Edit E — Dashboard personalization** — 3-5 hrs. Coaches toggle widgets on/off from hamburger menu. Needs widget registry + preferences storage + settings UI + render rewrite.
- **Star-cadence CRM reminders (Phase 6A.2)** — Approved Apr 10, never built. Star = auto-cadence (A=3d/B=7d/C=14d/D=30d), unstarred = manual only. contact_reminders table already exists.
- **prospect_visits + visit_itinerary SQL migration** — DDL exists (phase-5a-schema.sql + phase-5a-itinerary.sql), never run against live Supabase. Blocks visit form and itinerary UI.
- **New-visit form + add-itinerary-item modal** — Stubs in v22 toast "coming in Phase 5b." Depends on visit tables above.
- **Visit file uploads to Storage bucket** — Needs visit-materials bucket + signed URLs for View button. Depends on visit tables.
- **Real rankings data** — espn_rank, sports247_rank, etc. currently seeded from nil_estimate ordering (mock). Need real scraper or manual load.
- **Login debug: non-apex@test.com accounts** — stewartfam@mac.com, demo@apexintel.com, demo@test.com all fail. Never captured actual API error body. Procedure in HANDOFF-2026-04-14.
- **ESPN headshot URLs** — No player photos anywhere. Scrape from ESPN roster pages into school_rosters.photo_url + avatar fallback component.
- **Commit SQL migrations to /sql/ folder** — All DDL should be version-controlled for reproducibility.
- **Phone/iPad testing** — Live URL never tested on non-Mac device since v22 deploy.
- **Revoke old Anthropic API key** — sk-ant-api03-oB6...c (label "Apex Intel", Never Used) flagged Apr 10, still unconfirmed revoked.
- **Confirm CBBD key revocation** — Old key briefly visible in screenshot Apr 11. New key works. Old key status unknown.
- **Safari autocomplete="off" fix** — iCloud Keychain autofills personal email on ?nologin=1 page. 5 min fix.
- **.gitignore for backup files** — 4 index.html.pre-* files cluttering git status.
- **Token refresh handling** — T global has no refresh token logic. Sessions can expire silently.

---

## Design needed before building

- **Phase 5a stack: Streamlit vs HTML+supabase-js** — Streamlit is fastest to demo but limited customization. Existing Apex-App/index.html (142KB, ~2,267 lines) is already wired to Supabase and deployed but fragile as a single file. Decision blocks all frontend work.
- **staff table school_id** — 4 rows, no school affiliation column. Multi-school architecture needs staff-to-school linkage for RLS. Blocks Phase 6 role-based access.
- **Oregon recruiting service access** — Which paid services does Oregon staff have? (On3 / 247Sports / Rivals). Drives portal + HS data refresh strategy. Asked Apr 13, never answered.
- **Supabase secrets: Vault vs Edge Functions Secrets** — Need to pick one and standardize. Currently apex-scout uses EF secrets. Deferred since Phase 3.
- **D2/D3 roster data source** — No source identified. Likely school athletic sites or NCAA-API stats. Low priority unless user base expands beyond D1.
- **AI Scout structured response format** — Currently regex-scans LLM text for prospect names. Target: structured JSON with prospect_id, fit_score, reasoning. Blocks add-to-CRM from SCOUT results.
- **Decommit-history modeling** — Separate portal_history table for tracking status changes over time? Or keep overwriting portal_status on school_rosters?

---

## Pre-commercial deployment gaps

Original framework from Apr 9 audit. Status as of Apr 15:

| # | Gap | Status | Notes |
|---|-----|--------|-------|
| 1 | Data persistence | SHIPPED | Supabase backend live, 7,526 players loaded |
| 2 | Auth / user roles | PARTIAL | Hardcoded auto-login (apex@test.com). No per-coach roles. Phase 6. |
| 3 | Live transfer portal feed | PARTIAL | One-time 247 scrape (1,192 players). No daily refresh cron. |
| 4 | Push notifications | NOT STARTED | Portal alerts, cold-contact warnings, deadline reminders. |
| 5 | Editable team analytics profiles | NOT STARTED | Team weaknesses/strengths hardcoded in JS. Need per-school editable profile via team_strategy table. |

---

## Data quality / schema polish

- **schools.division cleanup** — All 1,518 rows labeled D1; only 365 are D1 MBB. Update based on espn_id presence. ~10 min SQL.
- **source_247_player_id column** — ncaa_player_id currently overloaded for 247 player keys. Add dedicated column + migrate. ~15 min.
- **~14 missing HS prospects** — 247 parser skips ad-interrupted rows (2026 ranks 251, 293-308; 2028 page 3). Fix parser, re-run.
- **ESPN '--' class_year** — Parser stores literal string instead of NULL. Patch scrape_espn_roster.py at parse time.
- **St. Francis PA override** — Add to MANUAL_OVERRIDES in backfill_espn_ids.py for full idempotency.
- **54 player_type=NULL residue rows** — From early phases. Clean up or backfill.
- **76 duplicate school names** — Deduped client-side only. DB-level merge needed (Kentucky Wesleyan x2, Ohio Wesleyan x2, others).
- **schools.city / schools.state** — NULL for all 1,518 rows. No backfill source identified.
- **schools.ncaa_id / balldontlie_id** — NULL for 364/365 D1 schools (only Oregon populated). Low priority unless those APIs become primary sources.
- **33 portal players with NULL source school** — Non-D1 source schools not in schools table. Cosmetic.
- **Mid-season coaching changes** — head_coach_name stale for: UNC, Arizona State, Cincinnati, Kansas State, Boston College. Verify before next demo.

---

## Ideas / someday

- Daily 247 portal cron or GitHub Action (until May 1 portal close)
- Push notifications / SMS (Twilio + Resend/SendGrid via Edge Functions)
- Two-way Google Calendar OAuth sync for visits
- Real On3/247/ESPN/Rivals nightly scrapers
- Compute On3 IRI in-house using published weighting
- Auto-balance NIL budget to cap (one-tap)
- AD approval notification flow (budget request push/email then approve)
- Historical budget snapshots (season over season)
- Auto-create CRM log entry on roster builder status change
- Cache LLM responses per query to reduce billing
- Real-time CBB news feed (apex-news Edge Function, CBS Sports RSS)
- Predictive transfer risk model (trained on historical transfer data)
- NBA draft probability on player cards
- Video annotation and clip tagging in-platform
- Multi-sport architecture (football / W-basketball under one AD login)
- Parent/family read-only portal
- Conference group licensing as GTM channel
- NIL Collective partner login with compliance firewall
- Offline mode / PWA with Service Worker
- Synergy API deep integration (play-type clips in player card, enterprise pricing)
- cbbdata (Andrew Weatherman) registration (needs R environment)
- KenPom renewal reminder before April 1, 2027
- audit_log triggers on all tables (function exists, not wired)
- docs/fork-guide.md (placeholder since Phase 2)
- Replace emoji icons with Lucide icon set
- Initials avatar fallback for roster/prospect cards
- Browser Contact Picker API for CRM import
- Per-school team color theming (CSS hooks wired, not populated)
- Techstars Sports / Stadia Ventures applications

---

## Shipped recently

| Date | Item |
|------|------|
| Apr 14 | Edit C: auto-login bootstrap (6a87de4) |
| Apr 14 | Edit D.2: coach names for 79 Power 5 + Big East schools |
| Apr 14 | Edit B: school picker D1 filter + conference chips + search (d6dd870) |
| Apr 14 | Edit B.1: server-side preload filter for 1000-row cap (b6894c5) |
| Apr 14 | Edit A: school-switching + materialize RPC (596b1e0) |
| Apr 14 | v22: rankings, visit calendar, NIL sliders, roster builder 18-slot |
| Apr 13 | Phase 4d: 1,192 portal players (247Sports) |
| Apr 13 | Phase 4c: 657 HS prospects (247Sports Composite) |
| Apr 13 | Phase 4b: 5,677 D1 players / 365 schools (ESPN) |
| Apr 13 | Phase 4a: Oregon roster + ESPN ID backfill for 364 schools |
| Apr 13 | CLAUDE.md + handoff system established |
| Apr 11 | Password gate stewart2026 on school picker |
| Apr 11 | apex-scout Edge Function (Claude LLM + rules-based) |
| Apr 10 | v3-v21: war room, CRM, vCard, budget, coach scorecard, school picker |
| Apr 9 | Phase 2 schema (14 tables) + Phase 3 RLS + API keys |

---

## Conventions

- **Location:** ~/code/Apex-App/BACKLOG.md (frontend repo root)
- **Review:** Start of every session alongside CLAUDE.md and latest handoff.
- **Updates:** When an item ships, move to "Shipped recently" with date. New ideas go to appropriate section. 1-3 lines per item.
- **Shipped log:** Rolling ~15 items. Archive older entries if list grows.
- **Priority changes:** "Shipping next" always has 3-5 items. Promote from "Queued" when one ships.
- **Cross-reference:** Note handoff filename when detailed spec exists elsewhere.
