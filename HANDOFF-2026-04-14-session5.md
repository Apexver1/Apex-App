# Handoff — April 14, 2026 (Session 5)

## Session summary
Phase 4 Edit C shipped. Hardcoded auto-login is live: app boots straight to the Oregon dashboard at the bare URL, with ?nologin=1 as an escape hatch to force the manual login screen. Phase 4 is now COMPLETE in full. One commit on main today: 6a87de4.

Process was slow — burned 3 Python re-runs on miscounted assertion expectations (AUTO_LOGIN_EMAIL count, bootAutoLogin count). Assertions correctly fired before any file write so no rollback was needed; just iteration on the validation logic. Lesson: do not hand-count token occurrences in the bootstrap. Use a "must not already be present" pre-flight guard plus a single "marker appears once" post-flight check.

## Shipped this session
- Edit C (6a87de4): hardcoded auto-login bootstrap appended as a fresh script block before </body>. Reads ?nologin=1 URL param to skip. Hides #login immediately to suppress flash. Populates #em and #pw, calls existing window.doLogin(). On auth failure (2.5s timeout check on #loginErr), un-hides login form so manual login still works.
- File now 142718 bytes / ~2267 lines (+1476 chars / +43 lines vs pre-Edit-C).
- Backup taken before edit: index.html.pre-edit-c (still in working tree, untracked).

## Acceptance tests (all PASS, verified live in Safari incognito)
- Bare URL https://apexver1.github.io/Apex-App/ -> boots to Oregon dashboard, no login form
- ?nologin=1 -> manual login form appears, fields empty (Safari Keychain may autofill, separate issue)
- Manual sign-in with apex@test.com / ApexTest123 -> succeeds, dashboard loads
- signOut -> login form reappears
- Edit B/B.1 school picker regression -> Duke switch worked (header + roster pulse updated)

## Known issues (not from Edit C, surfaced during testing)

### Safari Keychain autofill on email field
On the ?nologin=1 page, Safari iCloud Keychain autofills the email field with the user personal Apple ID address before the user types anything. Visual paper-cut for demos with real coaches. Fix: add autocomplete="off" to the #em and #pw inputs, or add name attributes that do not match common autofill patterns. Not blocking.

### "Good evening, Coach Coach." on Duke
After switching to Duke via the picker, the greeting renders as "Good evening, Coach Coach." coachFor(CURRENT_SCHOOL_ID) is returning a placeholder/null record for Duke. staff table likely has no head coach record for Duke. Affects every non-Oregon school presumably. Phase 5a: backfill staff for at least the top 30 D1 programs, or fall back to "Good evening." with no name when no coach record exists.

### "Next up" widget hardcoded to Oregon schedule
After switching to Duke, "Next up" still showed Oregon vs Arizona. Schedule widget is either hardcoded or not keyed by school_id. Phase 5a investigation needed.

## Burned-in pattern: surgical patch with bootstrap script tag (USE THIS GOING FORWARD)

For inserting a new self-contained JS block (bootstrap, feature flag, instrumentation):

1. cp index.html index.html.pre-edit-X
2. heredoc the raw JS body (no script tag) to /tmp/apex_X_bootstrap.js
3. Python regex insert that wraps in script tags and appends before </body></html>
4. Pre-flight assertion: assert "UNIQUE_MARKER" not in html — guarantees idempotency
5. Post-flight assertion: assert new_html.count("UNIQUE_MARKER") == 1 — single marker check, not per-token counts
6. Write file, print sizes

Why this beats inline insertion into the existing script block: regex target </body></html> is unambiguous, the new block is a single contiguous unit easy to find/diff/delete, and assertion logic stays simple.

## Why three Python re-runs (lesson for next time)

Iteration 1: assert new_html.count("AUTO_LOGIN_EMAIL") == 1 — wrong, bootstrap defines AND uses the constant = 2 occurrences.
Iteration 2: assert new_html.count("bootAutoLogin") == 4 — wrong, function appears as definition + addEventListener arg + direct call = 3 occurrences.
Iteration 3: dropped per-token counts entirely, used "marker not present pre / marker present once post" pattern. Worked first try.

The first two failures were harmless because the asserts ran before the file write. Confirmed by checking byte sizes (141450 stayed unchanged). No rollback ever needed, but the rollback path exists if a future patch fails AFTER write: cp index.html.pre-edit-X index.html.

## Phase 4 status: COMPLETE
- 4a D1 rosters (ESPN, 5677 players)
- 4b HS prospects (247Sports Composite, 657)
- 4c Transfer portal (247Sports, 1192)
- 4d School mapping (334 D1 via schools.sports247_id)
- Edit A (session 3): school switcher wires to CURRENT_SCHOOL_ID
- Edit B (session 4): picker hardening, focus-loss fix, conference chips
- Edit B.1 (session 4): server-side D1 preload (sub-1000-row payload)
- Edit C (this session): hardcoded auto-login + ?nologin=1 escape hatch

## Next session: Phase 5a — MVP frontend broader work

Auto-login removes the demo friction that blocked screen-share pitches. The next true blocker on a pilot conversation is non-Oregon data quality. Recommended Phase 5a sequencing:

1. Fix the Duke "Coach Coach" bug. Audit coachFor() and staff table population. Either backfill staff for top D1 programs, or add a clean fallback. Single highest-leverage UX fix because it is the first thing a non-Oregon coach sees.
2. Fix "Next up" school-switching. Schedule widget needs to query by school_id like everything else.
3. Add autocomplete="off" to login form fields. Five-minute fix, removes Safari autofill paper-cut for demos.
4. Then move to broader Phase 5a roadmap — the "Portal PGs on bottom-50 KenPom teams" filter and other killer demo features.

## Reference constants (carry forward)
- Oregon UUID: d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7
- Duke UUID: 279abc4c-77dd-4a3a-8f46-5daa6f9d8da1
- Supabase dashboard: https://supabase.com/dashboard/project/midyxvjfoggchbugxzkd
- SQL editor: https://supabase.com/dashboard/project/midyxvjfoggchbugxzkd/sql/new
- Live app: https://apexver1.github.io/Apex-App/
- Live app (no auto-login): https://apexver1.github.io/Apex-App/?nologin=1
- Login: apex@test.com / ApexTest123
- Frontend repo: ~/code/Apex-App
- Backend repo: ~/code/apex-intel
- Backups in working tree (untracked): index.html.pre-phase4-backup, index.html.pre-edit-b, index.html.pre-edit-b1, index.html.pre-edit-c
- Current index.html: 142718 bytes, ~2267 lines
- Auto-login feature flag location: index.html line ~2231 — window.AUTO_LOGIN = true;
- Safe publishable key: sb_publishable_MizrCkmoqSNlQAF3ty_FUA_WkATU4Ai

## Housekeeping for Phase 5a session start
The four index.html.pre-* backup files are cluttering git status. Add a .gitignore entry on session 6:

    echo "index.html.pre-*" >> .gitignore
    git add .gitignore
    git commit -m "gitignore index.html backup snapshots"
    git push origin main

## Session starter for next chat

Paste this as your first message:

    Working on Apex Intel. Mac/Terminal/Safari.
    Frontend: ~/code/Apex-App
    Live: https://apexver1.github.io/Apex-App/
    Login: apex@test.com / ApexTest123 (auto-fires; ?nologin=1 to suppress)

    Before doing anything, run this in Terminal and read the output — that is the latest handoff:
    cat ~/code/Apex-App/HANDOFF-2026-04-14-session5.md
    Also read project knowledge files (CLAUDE.md and HANDOFF-2026-04-13.md).

    Phase 4 is COMPLETE. Starting Phase 5a — fix the Duke "Coach Coach" greeting bug as the highest-leverage first task. Walk me through the plan before changing any code.
