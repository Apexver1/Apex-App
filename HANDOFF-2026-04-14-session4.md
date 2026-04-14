# Handoff — April 14, 2026 (Session 4)

## Session summary
Phase 4 Edit B shipped and verified live. School picker is now production-quality: D1 filter, conference chips, name/conference sort, live search, dedupe, Oregon findable. Two commits on main today: d6dd870 (Edit B) and b6894c5 (Edit B.1 server-side filter to dodge Supabase 1000-row API cap).

Burned roughly an hour chasing a phantom 230-row ceiling that turned out to be Safari cache holding a pre-Edit-B.1 snapshot. Proven by hitting the API directly with curl+JWT which returned content-range 0-364/365 while the browser capped at 230. Clearing Safari history for the last hour fixed it instantly.

## Shipped this session
- Edit B (d6dd870): split renderPicker into scaffold + renderPickerResults. Fixes search focus-loss bug (oninput used to rebuild entire modal mid-keystroke, destroying the input element). Adds conference chips, name/conf sort, dedupe via getD1Schools cache.
- Edit B.1 (b6894c5): added conference=not.is.null to schools preload URL. Server-side filter keeps payload to ~365 rows, under Supabase 1000-row cap.

## Diagnostics captured
- schools total: 1518
- conference non-null: 365 (= D1)
- conference null: 1153 (D2/D3/NAIA)
- schools.division exists but is blanket 'D1' on every row (useless for filtering)
- RLS on schools: single policy anyone_read_schools, qual=true, role authenticated (unconditional SELECT)
- Role caps: anon + authenticated both at pgrst.db_max_rows=5000 (not bottleneck)
- curl w/ anon apikey only: content-range */0 (RLS blocks anon as expected)
- curl w/ JWT + apikey: content-range 0-364/365 (backend healthy, returns all 365)

## NEW GOTCHA (burn into CLAUDE.md)
Safari caches Supabase REST responses aggressively across Cmd+Shift+R. If the backend proves correct via curl but frontend shows stale data:
1. Cmd+Option+E (Empty caches)
2. Safari menu > History > Clear History > last hour
3. Hard-reload twice

## Phase 4 status: COMPLETE
- 4a D1 rosters (ESPN, 5677 players)
- 4b HS prospects (247Sports Composite, 657)
- 4c Transfer portal (247Sports, 1192)
- 4d School mapping (334 D1 via schools.sports247_id)
- Edit A (prior session): school switcher wires to CURRENT_SCHOOL_ID
- Edit B (this session): picker hardening
- Edit B.1 (this session): server-side D1 preload

## Next session
Phase 4 Edit C: hardcoded auto-login, then Phase 4 closes. After that, Phase 5a (MVP frontend broader work) is the gating milestone for pitch/pilot conversations.

## Reference constants
- Oregon UUID: d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7
- Duke UUID: 279abc4c-77dd-4a3a-8f46-5daa6f9d8da1
- Supabase dashboard: https://supabase.com/dashboard/project/midyxvjfoggchbugxzkd
- SQL editor: https://supabase.com/dashboard/project/midyxvjfoggchbugxzkd/sql/new
- Live app: https://apexver1.github.io/Apex-App/
- Login: apex@test.com / ApexTest123
- Frontend repo: ~/code/Apex-App
- Backend repo: ~/code/apex-intel
- Backup before Edit B: ~/code/Apex-App/index.html.pre-edit-b
- Backup before Edit B.1: ~/code/Apex-App/index.html.pre-edit-b1
- Current index.html: ~141219 bytes, ~2226 lines
- Safe publishable key: sb_publishable_MizrCkmoqSNlQAF3ty_FUA_WkATU4Ai

## Surgical patch pattern (burned in, do not deviate)
1. cp index.html index.html.pre-edit-X
2. Write new JS to /tmp/apex_X_new.js via heredoc (cat > /tmp/... <<'JS_EOF' ... JS_EOF)
3. Python heredoc swap with strict regex + match count assertion (FAIL if matches != 1)
4. Sanity grep for key tokens
5. diff before/after, verify contiguous hunk with no stray hits
6. git add + commit + push
7. Wait 45-60s for GitHub Pages
8. Hard-reload twice in Safari, then clear history if stale

## Edit C spec (next session)
Goal: Skip the manual login screen for demo/dev convenience by auto-submitting apex@test.com / ApexTest123 on app load.

Implementation notes:
- Current login flow: doLogin() in index.html performs password grant against /auth/v1/token, stores JWT in T global, calls schools preload + initial render
- Auto-login should: on DOMContentLoaded, if no T set, call doLogin() with hardcoded credentials, suppress the login UI flash
- Must NOT break the manual login form (in case the auto-login fails or credentials change)
- Must be trivially removable for production (single feature flag at top of script: AUTO_LOGIN = true)
- Security: this is fine for the demo phase — single shared test account, no real user data, anyone with the URL can already see everything via that account anyway. Disable before any pilot with real coaching staff data.

Acceptance criteria:
- Open https://apexver1.github.io/Apex-App/ in a fresh incognito window
- App loads directly to home dashboard for Oregon, no login screen visible
- School picker still works
- Logout button (if exists) still works and re-shows login form
