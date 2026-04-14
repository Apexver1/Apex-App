# HANDOFF — April 14, 2026 (Session 3, mid-session break)

**Session owner:** Michael Stewart
**Break reason:** Screenshot limit hit on chat; resuming in fresh chat
**Last confirmed working state:** Phase 4 Edit A deployed + tested in browser

---

## 🔑 Session starter — paste as first message in next chat
cat > ~/code/Apex-App/HANDOFF-2026-04-14-session3.md <<'HANDOFF_EOF'
# HANDOFF — April 14, 2026 (Session 3, mid-session break)

Resume: Phase 4 Edit B — Fix school picker.

## What shipped this session
- Database: schema additions to prospect_shortlist, school_rosters, nil_budget, my_roster (school_id + indexes)
- Database: materialize_school_defaults() function installed and tested against Duke (UUID 279abc4c-77dd-4a3a-8f46-5daa6f9d8da1), seeds $10M budget + copies roster from school_rosters
- Frontend Edit A: selectSchool() and loadData() rewritten to filter by CURRENT_SCHOOL_ID (commit 596b1e0)
- Verified working in Safari: switching from Oregon to Duke shows Duke's 14 real players + fresh $10M budget

## Known issues in school picker (to fix in Edit B)
1. Picker shows ~1000 schools (mixed D1/D2/D3) — should be D1 only (~334)
2. Search input broken (typing does nothing)
3. No conference filter/sort
4. Duplicate rows (Denison appears twice)
5. Many schools show "?" for conference (NULLs)
6. Oregon not visible in picker (likely cut off by 200-item slice)

## STEP 1 — Diagnostic SQLs to run first in new chat

Supabase SQL editor: https://supabase.com/dashboard/project/midyxvjfoggchbugxzkd/sql/new

SQL 1 — Conference distribution:
  SELECT conference, COUNT(*) AS schools FROM public.schools GROUP BY conference ORDER BY schools DESC LIMIT 50;

SQL 2 — Totals + null audit:
  SELECT COUNT(*) AS total, COUNT(DISTINCT name) AS unique_names, COUNT(*) FILTER (WHERE conference IS NULL) AS null_conf FROM public.schools;

SQL 3 — Confirm Oregon:
  SELECT id, name, conference, city, state FROM public.schools WHERE name ILIKE '%oregon%' ORDER BY name;

SQL 4 — Check if division column exists:
  SELECT column_name FROM information_schema.columns WHERE table_schema='public' AND table_name='schools' AND column_name IN ('division','level','classification');

## Reference constants
- Oregon UUID: d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7
- Duke UUID: 279abc4c-77dd-4a3a-8f46-5daa6f9d8da1
- Supabase project: https://supabase.com/dashboard/project/midyxvjfoggchbugxzkd
- Live app: https://apexver1.github.io/Apex-App/
- Login: apex@test.com / ApexTest123
- Repo: ~/code/Apex-App/
- Backup: ~/code/Apex-App/index.html.pre-phase4-backup
- Current index.html: 2201 lines, ~138KB

## Phase 4 plan status
- Edit A (backend wiring) — SHIPPED
- Edit B (fix picker) — NEXT
- Edit C (remove login, auto-login) — AFTER EDIT B
- Phase 5 (ESPN roster photos) — AFTER EDIT C

## Gotchas burned this session
1. Supabase SQL editor runs only first statement — paste one at a time
2. nil_budget.remaining_amount is GENERATED — don't INSERT into it
3. Terminal paste wraps at ~119 cols — keep Python heredocs under 60 lines
4. GitHub Pages cache — hard reload with Cmd+Shift+R, sometimes twice

## Surgical patch pattern
Python heredoc in Terminal (keep under 60 lines per heredoc):
  python3 << 'PYEOF'
  from pathlib import Path
  FILE = Path.home() / "code" / "Apex-App" / "index.html"
  src = FILE.read_text()
  OLD = 'exact text to find'
  NEW = 'replacement text'
  if OLD in src:
      FILE.write_text(src.replace(OLD, NEW))
      print("OK")
  else:
      print("FAIL")
  PYEOF

Deploy:
  cd ~/code/Apex-App && git add index.html && git commit -m "msg" && git push origin main
