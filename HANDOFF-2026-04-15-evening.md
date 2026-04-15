# Handoff — April 15, 2026 (evening session)

## Session summary

Scout 1.1 shipped end-to-end. Refactored the apex-scout Edge Function from a regex-scanned big-prompt v0 to a full agentic Claude tool-use loop with 4 tools, structured JSON contract, and audit logging. Discovered that 0 of 7,526 players had stats populated in the DB — original Phase 4 scrape was rosters-only. Loaded Bart Torvik 2025-26 advanced stats against the D1 roster, achieving 4,122/5,677 (72.6%) coverage. Live demo now returns real player names, real PPG/RPG/APG/3P% numbers, and intelligent reasoning over the data.

Also rotated a leaked SUPABASE_SERVICE_ROLE_KEY (accidentally screenshotted), shipped a 14-page strategic brief PDF (v1.0 at /docs/Apex_Intel_Strategic_Brief.pdf), and made the strategic decision to compete with Dropback as a coach-first AI decision layer rather than a GM-first modeling tool.

## Shipped this session

### Scout 1.1 backend — agentic foundation
- Replaced supabase/functions/apex-scout/index.ts (286 lines, was 71)
- 4 tools wired: find_players, get_player_profile, diagnose_team_gaps, get_metric_distribution
- Claude tool-use loop with up to 8 iterations
- Structured response: {answer, summary, tool_calls[], iterations, schema_version}
- Backward-compatible answer field so frontend keeps working
- Backup of old version preserved at index.ts.scout-1.0.backup

### Database — advanced stats schema + data
- Added 22 columns to school_rosters: games, min_pct, ortg, drtg, usg_pct, efg_pct, ts_pct, orb_pct, drb_pct, ast_pct, to_pct, blk_pct, stl_pct, ftr, two_pct, porpag, adjoe, bpm, role, stats_source, stats_season, stats_updated_at
- Created public.scout_tool_calls audit table with RLS, service-role policy, school_id index
- Loaded Bart Torvik 2025-26 player stats: 4,006 successful updates (4,122 row coverage including duplicates from transfers)
- 80% match rate against BT universe (4,006 / 4,979 BT rows)
- 99% team match rate (363 / 365 BT teams) — only FIU and UMKC unmatched

### Scripts
- scripts/load_bt_player_stats.py — repeatable loader for any BT season
- Uses rapidfuzz for fuzzy name matching (threshold 88), team matching (threshold 85)
- Paginates all Supabase queries (handles 1000-row API cap)
- Reports detailed match metrics

### Security
- Rotated SUPABASE_SERVICE_ROLE_KEY (old: sb_secret_i6jrz... DELETED, new: sb_secret_eq2mb...)
- New key named "default_rotated_2026_04_15" in Supabase Project Settings
- Local .env updated, .env.backup preserved

### Strategic brief PDF
- Generated 14-page strategic brief at /docs/Apex_Intel_Strategic_Brief.pdf
- Charcoal/electric-blue brand identity, designed not generic
- Sections: Executive Summary, Where We Stand, What's Built, Product Family, Apex Scout Deep Dive, Apex Scout Architecture, Market Map, 6 Structural Advantages, Positioning, Why Now, Architecture, Roadmap

### Strategic positioning crystallized
Three sentences locked as positioning across all materials:
1. Other platforms are a Bloomberg terminal for college sports GMs. Apex is an AI chief of staff for the head basketball coach.
2. Other platforms make you build a model. Apex understands the conversation.
3. Other platforms give you a valuation. Apex gives you a decision.

## Competitive intel captured

### Dropback (the funded competitor)
- YC-backed, founded by ex-Microsoft + ex-Hudl engineers, parent company FanCave Inc
- Targets GM/cap-modeling persona, not head coach persona
- Drag-and-drop "Valuation Modeler" — coaches/GMs build position-specific value formulas
- Data partnerships: Synergy, KenPom, EvanMiya, SIS (Sports Info Solutions, March 2026)
- Customer wins: Kentucky "BLUEprint" deployment (April 7, 2026), unspecified SEC/Big Ten/Pac-12
- CAPS Day-1 integration (the new mandated cap compliance system from House Settlement)
- VP Football Strategy hire: Parker Fleming (March 2026)
- Marketing language to steal: "moneyball arbitrage between market value and YOUR value"

### What we explicitly chose NOT to do
- Not partnering with EvanMiya (would expose product surface + roadmap to Dropback ecosystem)
- Not building drag-and-drop modeling canvas (conversational > visual modeling)
- Not building cap allocation slider UI (Scout infers from team_strategy + diagnose_team_gaps)
- Not competing on football data (Dropback owns that lane)
- Not competing with CAPS (it's the official compliance system, we feed it)

### Patterns we WILL adopt
- Arbitrage framing: "Apex Value vs Market Asking gap"
- Cohort distribution display (the "Data Summary" pattern Dropback nailed)
- Position-specific + universal evaluation criteria (black/white card model)
- Manual + automated hybrid scoring (coach prospect_scores + Bart Torvik stats in same pipeline)
- "Crawl the universe" demo with the 4,122 stat-populated D1 players

## Demo state — what works now

### Working
- Auto-login at https://apexver1.github.io/Apex-App/
- AI Scout returns real player data with reasoning
- Test queries that produce demo-quality responses:
  - "Show me the top 5 D1 scorers averaging at least 20 points per game"
  - "Tell me about Cooper Flagg" (correctly distinguishes HS Cooper Flagg from Duke Cooper Flagg, explains gap)
  - "Find me a defensive-minded power forward"
- Strategic brief PDF ready to share at /docs/Apex_Intel_Strategic_Brief.pdf

### Cosmetic gaps (Scout 1.1.1)
- Frontend regex parser shows "0 players identified" because it doesn't know the new tool_calls structure
- Markdown headers in Scout responses render as raw `## Header` instead of formatted
- Player cards from Scout don't render as rich cards yet — just plain markdown text
- Both fixable in Scout 1.1.1 (frontend rendering refresh)

## Reference constants (carry forward)

- Apex-App: https://github.com/Apexver1/Apex-App
- Live: https://apexver1.github.io/Apex-App/
- Backend: https://github.com/Apexver1/apex-intel
- Supabase: https://midyxvjfoggchbugxzkd.supabase.co
- Oregon UUID: d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7
- Duke UUID (BT diagnostics reference): 279abc4c-77dd-4a3a-8f46-5daa6f9d8da1
- Demo login: apex@test.com / ApexTest123
- School picker password: stewart2026
- Bart Torvik 2025-26 endpoint: https://barttorvik.com/getadvstats.php?year=2026&csv=1

## Next session priorities (in order)

1. **Scout 1.1.1 — Frontend rich rendering** (blocker on demo polish)
   - Update frontend to read scout-1.1 response shape (tool_calls, iterations, schema_version)
   - Render markdown answer with proper formatting (## headers as bold, **text** as bold)
   - Render players from tool_calls[].output.results[] as rich cards instead of plain markdown
   - Update "X players identified" counter to count from tool_calls properly
   - Display reasoning trace as collapsible section ("How Scout reasoned")

2. **Portal player stats backfill**
   - 1,192 portal players still have has_ppg = 0
   - They likely played D1 last year — they ARE in the BT 2024-25 dataset under their previous schools
   - Strategy: run the loader against year=2025 data, but match BT players to portal records by name only (no team filter), with high name threshold (95+)
   - Expected: 600-900 portal players gain stats from their pre-portal D1 season

3. **Backlog gap cleanup**
   - 948 unmatched D1 roster players (mostly Jr./II/III suffixes — name normalization fix)
   - 2 unmatched team names (FIU → Florida International, UMKC → University of Missouri Kansas City)
   - Add MANUAL_OVERRIDES dict to load_bt_player_stats.py for known mappings

4. **Strategic brief polish (v1.1)**
   - Remove EvanMiya from market map "We integrate as data sources" row
   - Soften "next four pilot conversations" language if needed
   - Add real contact email/phone to back cover

5. **Scout 1.2 — Decision intelligence tools**
   - value_athlete(player_id, school_id) — proper valuation engine using BT advanced stats + team_strategy + position scarcity
   - simulate_roster(school_id, additions[], removals[]) — what-if scenarios
   - find_similar_players(player_id, pool_filters) — cosine similarity over normalized advanced stat vectors

## Burned-in workflow gotchas (carry forward)

- For long Python scripts, use heredoc-to-/tmp pattern, then run with cd ~/code/apex-intel && set -a && source .env && set +a && python3 /tmp/script.py
- load_dotenv() doesn't reliably find .env across cwd changes — use `set -a && source .env && set +a` shell pattern instead
- Heredocs over ~50 lines sometimes get truncated by Terminal — verify with wc -l before running
- TextEdit defaults to Rich Text — always Cmd+Shift+T to switch to plain text before pasting code
- read -s for password input doesn't show pasted content — easy to accidentally paste extra clipboard content; always verify length after with awk
- BT season URLs: year=2025 means the 2024-25 season (ended April 2025); year=2026 means 2025-26 season (just ended April 2026)
- `open -a TextEdit /path/to/file` requires the file to exist already; use `touch /path/to/file && open -a TextEdit /path/to/file` to create+open
- `git add` will fail with "did not match any files" if the file is empty AND .gitignore'd, but more commonly it means the file doesn't exist on disk yet

## Session starter for next chat

Paste this as your first message:

Loading Apex Intel project context. Please fetch:
https://raw.githubusercontent.com/Apexver1/Apex-App/main/BACKLOG.md
https://raw.githubusercontent.com/Apexver1/Apex-App/main/HANDOFF-2026-04-15-evening.md

Scout 1.1 is shipped and working with 4,122 D1 players carrying real Bart Torvik advanced stats. Ready to ship Scout 1.1.1 — frontend rich-card rendering — to make the demo visually clean.
