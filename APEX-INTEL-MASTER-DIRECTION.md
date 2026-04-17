# Apex Intel — Master Direction Document
> Living document. Every fix, feature, suggestion, and future plan in one place.
> Last updated: April 15, 2026
> Source: compiled from all Apex Intel chat sessions (April 9–15, 2026)

---

## TABLE OF CONTENTS
1. Bugs & fixes needed now
2. Apex Scout (flagship — the moat)
3. Apex CRM (recruit relationship management)
4. Apex Patrons (donor / development / fundraising)
5. Apex Ops (recruiting materials hub)
6. Dashboard & home screen
7. Roster tab
8. Recruiting tab (portal + HS)
9. NIL Budget & Roster Builder
10. Smart Filter
11. Scheduled visits
12. Login & onboarding UX
13. Player cards (universal)
14. Data quality & freshness
15. Advanced analytics integration
16. AI & automation features
17. Auth, roles & permissions
18. Notifications & communications
19. Architecture & infrastructure
20. Business & GTM
21. Someday / long-term vision

---

## 1. BUGS & FIXES NEEDED NOW

### Fixed (for reference)
- [x] "Coach Coach" greeting when school has no head_coach_name (D.1 — shipped Apr 15)
- [x] Password gate on school picker removed during v22 redesign (restored Apr 15)
- [x] School picker showing all 1,518 schools instead of D1 only (Edit B — shipped Apr 14)
- [x] Search input losing focus on re-render in school picker (Edit B — shipped Apr 14)
- [x] Supabase 1000-row API cap on schools preload (Edit B.1 — shipped Apr 14)
- [x] apex-scout Edge Function 401 Invalid JWT (JWT verification toggle turned off — shipped Apr 11)
- [x] apex-scout model name wrong: claude-sonnet-4-5 → claude-sonnet-4-5-20250929 (shipped Apr 11)
- [x] Safari aggressive caching of Supabase REST responses (documented workaround: clear last hour history)

### Still open
- [ ] Non-apex@test.com login accounts fail — stewartfam@mac.com, demo@apexintel.com, demo@test.com all reject. Root cause never captured (need Safari Web Inspector → Network → actual API response body).
- [ ] Safari iCloud Keychain autofills personal email on ?nologin=1 page — add autocomplete="off" to email input (5 min fix).
- [ ] Token refresh handling — T global has no refresh token logic. Sessions can expire silently during long use.
- [ ] Old Anthropic API key sk-ant-api03-oB6...c ("Apex Intel", Never Used) flagged Apr 10 — still not confirmed revoked. Console UI wouldn't allow deletion. Try from different browser/device.
- [ ] Old CBBD API key briefly visible in screenshot Apr 11 — new key works, old key revocation status unknown at collegebasketballdata.com.
- [ ] Four index.html.pre-* backup files in working tree — .gitignore entry added Apr 15 but files themselves still exist on disk.

---

## 2. APEX SCOUT (flagship — the moat)

This is the product's defining feature. Nobody else has this. Justifies $3,500/mo pricing.

### Current state
- Rules-based mode + LLM mode toggle exists
- LLM mode calls apex-scout Edge Function → Claude → returns text response
- Frontend regex-scans LLM text for prospect names and renders as tappable cards
- Quick query chips exist (e.g. "Find me a PG with 15+ ppg")
- Voice input stubbed via webkitSpeechRecognition (built in v9, untested post-redesign)

### What needs to change
- **Strip the rules/AI toggle** — it's always LLM. Remove "Rules · free" vs "Apex AI · $0.01/q" entirely.
- **Conversational natural language input** — placeholder should say "Ask Apex anything..." not "e.g., Find me a PG shooter under $1M"
- **Coaches talk like coaches** — the system must handle: "I need a wing who can guard 1 through 4, play in transition, and won't cost me more than $1.5M. Oregon just lost Shelstad so I need somebody who can run pick-and-roll too."
- **Voice input fully working** — Mic button triggers continuous-mode speech recognition. Long-form dictation. Works while walking, driving, in the gym.
- **Rich player-card responses** — when Scout recommends players, render full player cards with Apex Model/Market/Delta, portal rankings, scouting notes, competing schools, "Score this player" button, "Add to CRM" button. NOT plain text with names.
- **AI reasoning panel** — expands to show why Apex picked them, what it considered, what it dismissed, its confidence. Cite specific stats ("Rob Wright was #1 because he's shooting 41% from 3 on 6.2 attempts per game at BYU, which addresses your 248th-ranked 3PT offense"). Already exists in basic form — make it richer and more honest about tradeoffs.
- **Multi-turn conversation** — not one-shot Q&A. Coach can refine: "Show me the same list but under $1M" or "What about ones who've played in the tournament" or "Who does Kentucky also want?" Context persists across the session.
- **Full data access in context** — every query should have access to:
  - Full player universe (D1 rosters + portal + HS + international)
  - KenPom team context (AdjO/AdjD/tempo for every school)
  - Torvik advanced player stats (TS%, usage, BPM, eFG%, ORtg/DRtg, minutes%)
  - Synergy play types (PnR PPP, isolation, spot-up, transition — when API wired)
  - Your team's current roster, strategy profile, depth chart, NIL cap
  - CRM state (who's been contacted, competing schools, status)
  - Recent portal activity (who entered in last 24h/7d)
- **Proactive suggestions** — Scout watches the database and surfaces alerts: "Rob Wright just entered the portal 2 hours ago, you should know — he matches 4 of your 5 priority filters." Ties into dashboard smart notifications.
- **Per-school strategy context** — pulls from the team's editable analytics profile so every recommendation is auto-contextualized by YOUR team's weaknesses.
- **Structured response format** — backend returns JSON {picks: [{prospect_id, fit_score, reasoning}]} instead of relying on regex text parsing. Enables better card rendering and Add to CRM action.
- **Cache LLM responses per query** — avoid re-billing for identical searches.

---

## 3. APEX CRM (recruit relationship management)

Rename from "CRM" to "Apex CRM." Position as a recruiting Salesforce.

### Current state
- 100 prospects listed with tier A/B/C/D filter
- Player card opens with stats, rankings, film links, scoring, status, scouting notes, competing schools, log contact, history
- Contact logging: type dropdown + notes + Log Contact button
- History shows chronological entries per prospect

### What needs to change
- **Stars (1-5) replace tier letters (A/B/C/D)** — 5 stars is coaching vernacular. Coaches assign star value to a kid and it tracks everywhere across the platform (roster cards, portal cards, SCOUT results, dashboard).
- **Alphabetical phonebook layout** — section dividers A through Z, like iOS Contacts. Not infinite scroll. These are your contacts, organize them like a phone book.
- **Rich contact profiles** — full contact details: parents (names, phones, emails), AAU coach (name, phone, program), HS coach (name, phone), player's own phone/email/social handles. Must support multiple family contacts per prospect.
- **Restore vCard import/export** — this existed in an earlier build (v13) and disappeared during a later update. Needs to come back. Coaches live in their phone contacts.
- **Log contact date/time picker** — currently assumes every log is "now." Coaches need to record past interactions: "I called him yesterday," "Met at AAU tournament last Saturday." Date and time stamp required on every logged contact.
- **Coach assignment per relationship** — one or multiple staff members assigned. Visible on the card. Prevents two assistants calling the same kid saying contradictory things.
- **Competing schools editable** — current chips are seed data only. Add an input field or school-picker chip selector so coaches can type in schools they're hearing about from recruits, AAU coaches, or the rumor mill. This is high-value intel that only lives in coaches' heads today.
- **Smart suggestions on dashboard** — "Time to check in with [5-star kid]" or "Text [player] — last contact 12 days ago" based on star rating + cadence rules. Ties into the star-cadence system (Phase 6A.2, approved Apr 10, never built). Cadence defaults: 5-star = every 3 days, 4-star = every 7 days, 3-star = every 14 days, 2-star = every 30 days, 1-star = manual only.
- **Universal "Add to CRM" button** — every player card in roster, portal, HS recruits, SCOUT results, anywhere a player appears gets a one-tap button to add them to CRM and start tracking. CRM stops being a silo with its own separate seed data.

---

## 4. APEX PATRONS (donor / development / fundraising / communications)

Brand new section. White space — nobody builds donor CRM for college athletics in the NIL era. Raiser's Edge costs $50K+/yr and nobody uses it well.

### Ship as demoable shell first
Build the drawer navigation entry, screens with realistic dummy data, interactive but not wired to backend. Full build is Phase 6-scale (estimated 3-4 weeks dedicated).

### Full feature spec
1. **Donor database** — who, how much, when, to what (collective / program / specific sport / specific purpose). Lifetime giving totals, YTD, largest gift, first gift date, last gift date.
2. **Prospect research** — find future donors by scraping LinkedIn, public web, social media for high-net-worth alumni, local business owners, corporate sponsors. Flag lapsed donors. Tier prospects by giving capacity.
3. **Relationship management** — every touchpoint logged (call, email, in-person, game attendance, dinner). Like recruit CRM but for donors. Assigned staff member per relationship. Last-contact aging, auto-reminders when relationships go cold.
4. **Event and game attendance tracking** — which donors attended which home/away games, dinners, golf outings. Auto-trigger outreach: "John hasn't been to a game in 6 months — offer free tickets" or "Sarah will be in Tucson for work during the Arizona game — invite to the team dinner."
5. **Gifting and stewardship** — track what's been sent (signed ball, jersey, personalized note from HC, birthday card). Suggest stewardship actions based on giving tier and lapse time.
6. **Campaigns** — capital campaigns, year-end appeals, NIL fundraisers. Track who was solicited, who gave, who's still pending. Segment by tier for targeted asks.
7. **Communications manager** — bulk email/text to donor segments. "All Platinum tier donors, we're playing at UCLA next month, here's a pre-game reception invite." Integrates with email (Resend/SendGrid) and SMS (Twilio). Templates with personalization.
8. **Travel and road game intelligence** — when the team travels to a city, surface donors located in that city with their last-contact date and last-gift info. One tap to send a personalized invite.
9. **AI prospecting** — Apex Scout pattern applied to donors. "Find me alumni in Portland with a net worth over $5M who haven't been approached." "Draft a stewardship email to Mike Roberts — he gave $50K last year and hasn't heard from us in 4 months."
10. **Compliance dimension** — NCAA has rules about booster contact, benefits to recruits, etc. Audit trail of donor-recruit interactions matters for compliance.

---

## 5. APEX OPS (recruiting materials hub)

Stays in recruiting lane. Does NOT compete with Teamworks for general team operations (travel, scheduling, practice planning). Gives managers and grad assistants a daily-use surface.

### Ship as demoable shell first
Same as Patrons — drawer entry, screens with dummy data, interactive but not backend-wired.

### Full feature spec
1. **Recruiting decks** — upload PDF/Keynote/PPT per recruit, version history, who created it, when last updated. Real scenario: HC says "recruit Jalen Washington," GA builds a personalized deck (facilities, NIL package, alumni, family-specific touches), uploads it, it's on the HC's iPad for the meeting.
2. **Visit itineraries** — already partially in Scheduled visits. Also lives here as printable/shareable deliverable.
3. **Scouting reports** — opponent breakdowns, pre-game preps, post-game film notes.
4. **Family touchpoint materials** — letters from HC, photo collages from campus visits, personalized highlight reels.
5. **NIL pitch packets** — what we offer, comparable deals, payment structure. Can pull from NIL Budget data.
6. **Transfer portal reports** — staff-prepared deep dives on portal targets beyond what's in the database.
7. **Team materials** — playbooks, scout cards, practice plans (light storage only — don't compete with Teamworks here).

### Differentiators
- **Templates with auto-fill** — recruit name, position, school context auto-populates from the database.
- **AI-assisted draft generation** — "Apex, draft a recruiting deck for Jalen Washington using our standard template, his stats, and his family's interest in academics."
- **Tracked sharing** — send a deck to a recruit's family via a unique link. See when they opened it, what pages they spent time on.
- **Permission-aware** — ops staff can edit decks, head coach can comment/approve, recruits/families get view-only.
- **Searchable archive** — "show me every recruiting deck we sent to wings in the last two years."

---

## 6. DASHBOARD & HOME SCREEN

### Clickable tiles
Every KPI tile on the dashboard should deep-link to its relevant tab:
- Open scholarships → Roster Builder
- NIL requests → NIL Budget (filtered to pending)
- CRM follow-ups → CRM (filtered to overdue)
- Reminders today → CRM or reminders view
- Portal churn · 24h → Recruiting (portal tab)
- Visits this week → Scheduled visits

### Smart suggestions surface
Dashboard should show time-sensitive suggestions from across the platform:
- CRM: "Time to text [5-star kid] — last contact 12 days ago"
- NIL: "Flory Bidunga $125K payment due April 30"
- Patrons: "John Morrison hasn't been to a game in 6 months — offer free tickets"
- SCOUT proactive: "Rob Wright just entered the portal — matches 4 of 5 priority filters"
- Ops: "Jalen Washington's recruiting deck hasn't been updated in 3 weeks"

### "Next up" widget
Currently shows "Season complete / 2026-27 schedule TBD" (correct for off-season). When schedule data is loaded (Edit D.4), show upcoming game with venue, opponent, KenPom rankings, win probability. Expandable "View 2025-26 results" section.

### Dashboard personalization (Edit E — deferred)
Coaches toggle widgets on/off from hamburger menu. Requires widget registry + preferences storage + settings UI + render rewrite. Estimated 3-5 hours.

---

## 7. ROSTER TAB

### Player detail modal
- **Center vertically** — currently anchored to bottom of screen. Should be centered on desktop.
- **Player photos** — headshot at top of card (see section 13 below).

### Roster data freshness
- ESPN scrape from April 13 is stale. Players who've entered the portal or graduated still showing.
- Need either periodic re-scrape or cross-reference against 247 portal list to auto-flag/remove players who left.
- Consider changing primary roster source if ESPN proves unreliable for real-time accuracy.

### Transfer risk
- Currently hardcoded seed data notes (e.g. Sean Stewart: "Duke transfer. Could test portal again.").
- Should derive dynamically from: portal status in school_rosters, HS commit fluidity, NBA draft signals, coach departure signals.
- Or at minimum be coach-editable (coaches have intel the database doesn't).

---

## 8. RECRUITING TAB (portal + HS)

### Portal players
- Data is one-time 247Sports scrape (1,192 players). Needs daily refresh cron or GitHub Action, especially during portal window (through May 1).
- Committed/withdrawn status should update automatically.

### HS prospects
- 657 loaded from 247Sports Composite. ~14 missing due to ad-interrupted rows in parser (2026 ranks 251, 293-308; 2028 page 3). Fix parser and re-run.
- Rankings data (espn_rank, sports247_rank, on3_rank, rivals_rank) currently mock-seeded from nil_estimate ordering — doesn't match reality. Need real data load.

---

## 9. NIL BUDGET & ROSTER BUILDER

### NIL contract upload + AI parsing
Platform-defining feature. When a player commits via Roster Builder:
- Upload accepted NIL agreement (PDF) to Supabase Storage bucket.
- Claude reads the contract → extracts structured data: payment schedule, appearance obligations, social media deliverables, exclusivity clauses, termination conditions.
- Auto-generate milestone reminders on dashboard: "Flory Bidunga $125K payment due April 30," "Social post for Nike due Friday," "Autograph signing Saturday 2pm."
- Compliance benefit: structured contract data = instant audit trail for NCAA NIL reporting.
- Nobody else is doing AI contract parsing for college NIL.

### Apex Model calibration
- Current formula: PPG×$80K + position bonus + years×$150K + demand×$100K × conference multiplier.
- $1.9M for Bidunga and $850K "market" are both likely off.
- Need to calibrate against known deal comps or integrate real NIL market data (On3 NIL Valuation, Opendorse, INFLCR).

### Roster Builder
- 18 slots (13 NIL + 5 depth) with Planned/Offered/Accepted/Declined status.
- "Make offer →" and "Remove" buttons on filled slots.
- NIL contract upload should attach here per committed slot.

---

## 10. SMART FILTER

### Advanced analytics integration (deferred design)
When revisited, incorporate:
- **KenPom team context** — AdjO/AdjD/tempo, conference rank, efficiency margin. Filter by current school's metrics.
- **Torvik player stats** (via cbbdata) — true shooting %, usage rate, box plus-minus, eFG%, offensive/defensive rating, minutes %.
- **Synergy play types** — PnR ball handler PPP, isolation PPP, spot-up 3PT%, transition efficiency (gated behind enterprise API).
- **Shot profile** — rim %, mid-range %, 3PT attempt rate, FT rate.
- **Defensive metrics** — steal %, block %, defensive rebound rate, defensive rating.
- **Health/availability** — games played, minutes trend, injury history flags.
- **Portal-specific** — days in portal, previous transfer count, coach change trigger (did their HC leave?).

Killer demo unlocked: **"Portal PGs on bottom-50 KenPom teams."**

---

## 11. SCHEDULED VISITS

### Build out "+ New" visit flow
Currently button stubs "coming in Phase 5b." Needs:
- Form modal for new visit (select prospect, set date, visit type official/unofficial).
- prospect_visits table migration (SQL DDL exists as phase-5a-schema.sql, never run against live Supabase).
- visit_itinerary table migration (phase-5a-itinerary.sql, never run).
- Add-itinerary-item modal (time picker, duration, title, location, notes).
- File upload to visit-materials Storage bucket + signed URLs for View button.
- "Add to Calendar" button generates .ics download (already exists for seed visits).
- Two-way Google Calendar OAuth sync (someday/Phase 5c).

---

## 12. LOGIN & ONBOARDING UX

### Login splash screen
After login, before dashboard, show a branded interstitial:
- Apex Intel logo centered.
- Tagline or CTA button: "Build a Champion" (or similar coach-facing phrase).
- Click the button → lands on dashboard.
- Creates a branded moment. Makes the app feel like a destination.

### Login page label
- Replace "v2" with **"BETA"** — signals active development without implying incomplete.

### Auto-login
- Currently works: bare URL auto-fires login with apex@test.com / ApexTest123.
- ?nologin=1 URL param shows manual login form.
- Must be disabled before any pilot with real coaching staff data (hardcoded credentials in client JS).

---

## 13. PLAYER CARDS (universal — applies everywhere)

### Player photos
Coaches scan by face before name. Photos needed on EVERY card surface:
- Roster list cards
- Portal prospect cards
- HS prospect cards
- SCOUT result cards
- CRM contact cards
- Roster Builder slots
- Player detail modal (top of card)

**Source:** ESPN headshots for D1 roster players (scrape into school_rosters.photo_url). For portal + HS prospects, 247Sports and On3 have photos (scraping rules TBD).
**Fallback:** Initials avatar component for missing photos (e.g., "NB" for Nate Bittle in a circle).

### Coach scoring modal
- **Remove Weighting dropdown** — the four sliders (Shooting, Defense, IQ, Fit) + notes + save is clean enough. The weighting presets (Equal, Production, Fit-heavy, Custom) add complexity coaches won't use.

### Universal actions on every player card
- "Add to CRM" button (creates prospect_shortlist entry, opens contact logging)
- "Score this player" button (opens coach scoring modal)
- Status chips (Watching / Contacted / Visited / Offered)
- Film links (Watch on Synergy, YouTube)

---

## 14. DATA QUALITY & FRESHNESS

### Schema fixes needed
- schools.division: all 1,518 rows labeled D1, only 365 are D1 MBB. Update based on espn_id presence (~10 min SQL).
- source_247_player_id column: ncaa_player_id currently overloaded for 247 player keys. Add dedicated column + migrate (~15 min).
- ESPN '--' class_year: parser stores literal string instead of NULL. Patch scrape_espn_roster.py.
- St. Francis PA: add "Saint Francis Red Flash": "St. Francis (PA)" to MANUAL_OVERRIDES in backfill_espn_ids.py.
- 54 player_type=NULL residue rows from early phases. Clean up or backfill.
- 76 duplicate school names: deduped client-side only. DB-level merge needed (Kentucky Wesleyan x2, Ohio Wesleyan x2, others).
- schools.city and schools.state: NULL for all 1,518 rows. No backfill source identified.
- schools.ncaa_id / balldontlie_id: NULL for 364/365 D1 schools (only Oregon populated). Low priority unless those APIs become primary sources.
- 33 portal players with NULL source school: non-D1 source schools not in schools table. Cosmetic.

### Coach name coverage
- 79 of 1,518 schools have non-NULL head_coach_name (78 Power 5 + Big East from Edit D.2 + Oregon prior).
- Mid-major and non-Power schools show generic "Coach." greeting (D.1 handles cleanly, but real names better for demos).
- Mid-season coaching changes may be stale: UNC (Hubert Davis → reportedly Michael Malone), Arizona State (Bobby Hurley out), Cincinnati (Wes Miller out), Kansas State (Jerome Tang fired Feb 15, Matthew Driscoll interim), Boston College (Earl Grant fired March 8).

---

## 15. ADVANCED ANALYTICS INTEGRATION

### Data sources available
| Source | Auth | Status | Use |
|---|---|---|---|
| KenPom | Bearer token ($95/yr) | Active, key purchased | Team efficiency metrics (AdjO, AdjD, tempo, SOS) |
| cbbdata/Torvik | Free API key | Not yet registered | Player-level advanced stats (TS%, BPM, usage, eFG%, ORtg/DRtg) |
| ESPN | Keyless | Active | Rosters, schedules, basic stats |
| 247Sports | Scrape (no API) | Active | HS recruits, transfer portal |
| BallDontLie | API key (GOAT tier) | Active | Historical player/team/game data (not current rosters) |
| CBBD | API key | Active | College-specific stats |
| Sportradar/Synergy | Enterprise (contact sales) | Not purchased | Possession-level: play types, lineups, defensive matchups, shot coords |
| On3 | No API (scrape or paid tier) | Not explored | NIL valuations, portal rankings, recruiting rankings |
| NCAA-API | Keyless | Verified | Scoreboard, stats, standings, rankings (no rosters endpoint) |

### Integration priority
1. Wire KenPom team data to Smart Filter + Scout (team efficiency context per school)
2. Register for cbbdata and wire Torvik player stats
3. Evaluate Synergy/Sportradar pricing for play-type data
4. On3 NIL Valuation for Apex Model calibration

---

## 16. AI & AUTOMATION FEATURES

### Across the platform
- **Apex Scout** — see section 2 above (flagship)
- **AI contract parsing** — see section 9 (NIL contracts)
- **AI donor prospecting** — see section 4 (Apex Patrons)
- **AI deck generation** — see section 5 (Apex Ops)
- **AI stewardship drafts** — "Draft a thank-you email to Mike Roberts referencing his $50K gift and the new locker room renovation it funded"
- **Proactive alerts everywhere** — portal entries, cold contacts, donor lapse, NIL milestones, coaching changes affecting recruits

### Rules-based automation
- Star-cadence CRM reminders: 5-star=3d, 4-star=7d, 3-star=14d, 2-star=30d, 1-star=manual (approved Apr 10, never built)
- Auto-create CRM log entry when roster builder status changes
- Flag roster players who appear in portal feed → auto-update transfer risk

---

## 17. AUTH, ROLES & PERMISSIONS (Phase 6)

Currently hardcoded auto-login with apex@test.com / ApexTest123. No per-coach roles.

### Needed for commercial deployment
- **Head Coach** — full access including NIL numbers, donor data, AI Scout
- **Assistant Coach** — player evaluation, CRM, contact logging, no NIL financials, no donor data
- **Director of Operations** — roster management, budget, compliance export, Apex Ops materials
- **NIL Collective** — own login with compliance firewall between their view and internal scouting notes
- **Patron/Development staff** — Apex Patrons access, donor data, campaign management

### Technical requirements
- staff table needs school_id column for per-school RLS
- Multi-school architecture already designed (school_id on every table) but role layer not built
- Hardcoded credentials in client JS must be disabled before any pilot with real coaching staff data

---

## 18. NOTIFICATIONS & COMMUNICATIONS

### Push notifications (not started)
- Portal entry alerts ("Jalen Washington just entered the portal")
- Cold contact warnings ("You haven't contacted [5-star kid] in 14 days")
- Approaching deadline reminders ("NLI deadline in 3 days for [player]")
- NIL payment due reminders
- Donor stewardship triggers
- Scout proactive recommendations

### Communications manager (for Apex Patrons)
- Bulk email to donor segments via Resend/SendGrid
- Bulk SMS via Twilio
- Templates with personalization (donor name, last gift, team schedule)
- Track opens/clicks

### Email/SMS for CRM (someday)
- Automated cadence-based texts to recruits (with compliance guardrails)
- Email templates for initial outreach, follow-up, offer communication

---

## 19. ARCHITECTURE & INFRASTRUCTURE

### Current stack
- Frontend: single-file index.html (~142KB, ~2,268 lines) at github.com/Apexver1/Apex-App, deployed via GitHub Pages
- Backend: Supabase (Postgres + RLS + Auth + Edge Functions + Storage)
- AI: Claude via apex-scout Edge Function
- Data scripts: Python 3.12.13 at ~/code/apex-intel, venv-based
- Dev workflow: Mac Terminal, surgical Python patches to index.html

### Known architectural concerns
- Single-file HTML is fragile at 142KB — will need to break apart into components (React/Next.js refactor) by Phase B for maintainability
- Supabase default 1000-row API cap — must paginate or filter at preload URL level
- Safari aggressively caches Supabase REST responses — clearing last-hour history is the workaround
- Edge Function model: claude-sonnet-4-5-20250929
- JWT verification OFF on apex-scout (fine for demo, must secure for production)

### Patch workflow (burned in)
- Write Python patch script to /tmp via heredoc, then execute (not inline python3 -c — quote escaping breaks)
- Pre-flight assert: pattern count == 1
- Post-flight verification: grep or sanity check
- When a marker appears in multiple contexts (CSS + JS), use c.index(marker, start_from) to target the right one
- Test each patch independently before stacking
- Revert-and-reapply-one-at-a-time when stacking breaks deploy
- Don't drag screenshots into Terminal during a heredoc

---

## 20. BUSINESS & GTM

### Pricing (from pitch deck)
- Power tier: $3,500/mo
- Pro tier: $1,500/mo
- Starter tier: $400/mo

### Anchor client
- Oregon Basketball as pilot / case study generator

### Contractor path (explored but deferred)
- Upwork overseas: Eastern Europe $35-65/hr, LatAm $30-55/hr, India $15-40/hr
- Recommended: 15-20 hrs/week senior contractor at ~$50/hr = ~$750-1,000/week
- Phase A spend estimate: $10K-15K over 2-3 months
- User chose to continue solo for now

### Timeline estimate (8 hrs/day solo)
- Phase A (demoable pilot product): 6-8 weeks
- Phase B (production-ready for paid pilot): 3-4 months from start
- Phase C (multi-program scale): 9-12 months from start

---

## 21. SOMEDAY / LONG-TERM VISION

- Daily 247 portal cron / GitHub Action
- Two-way Google Calendar OAuth sync for visits
- Real On3/247/ESPN/Rivals nightly scrapers
- Compute On3 IRI in-house using published weighting
- Auto-balance NIL budget to cap (one-tap)
- AD approval notification flow (budget request → push/email → approve)
- Historical budget snapshots (season over season)
- Real-time CBB news feed (apex-news Edge Function, CBS Sports RSS)
- Predictive transfer risk model (trained on historical transfer data)
- NBA draft probability on player cards
- Video annotation and clip tagging in-platform
- Multi-sport architecture (football / W-basketball under one AD login)
- Parent/family read-only portal for recruits
- Conference group licensing as GTM channel
- NIL Collective partner login with compliance firewall
- Offline mode / PWA with Service Worker for field use
- Synergy API deep integration (enterprise pricing, play-type clips in player card)
- cbbdata (Andrew Weatherman) registration (needs R environment for signup)
- KenPom renewal reminder before April 1, 2027
- audit_log triggers wired to all tables (function exists, never connected)
- docs/fork-guide.md (placeholder since Phase 2)
- Replace emoji icons with Lucide icon set
- Initials avatar fallback for roster/prospect cards
- Browser Contact Picker API for CRM import
- Per-school team color theming (CSS custom property hooks wired, not populated)
- Techstars Sports / Stadia Ventures applications
- Native iOS app for full Contacts framework access
- Social media / marketability scoring integrated into NIL valuation
- Competitive intelligence module — track rival roster holes and portal targets
- Player retention strategy module (early warning: playing time, NIL dissatisfaction, development stagnation)
- Full cost-of-attendance scholarship modeling (NIL + tuition + room/board)
- Recruiting calendar sync to Google / Apple Calendar
- Offer letter and compliance tracking with timestamps
- Development tracking for current roster (season-over-season trends)
- NIL market intelligence database — anonymized deal comps across clients

---

## PRODUCT FAMILY

| Product | Purpose | Status |
|---------|---------|--------|
| Apex Scout | AI scouting / recruiting recommendations | Live (needs flagship upgrade) |
| Apex CRM | Recruit relationship management | Live (needs ground-up redesign) |
| Apex Patrons | Donor / development / fundraising | Design phase (demoable shell next) |
| Apex Ops | Recruiting materials hub | Design phase (demoable shell next) |

---

## HOW TO MAINTAIN THIS DOCUMENT

- Add new ideas under the appropriate section heading
- When something ships, move it to section 1 under "Fixed" with the date
- When starting a new chat, paste this document or reference it from project knowledge
- This is the single source of truth for product direction — BACKLOG.md in the repo tracks execution priority, this document tracks the full vision
