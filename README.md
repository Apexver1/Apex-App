# Apex Intel

Basketball operations and NIL management platform for Division I men's basketball programs.

**Live dashboard:** https://apexver1.github.io/Apex-App/
**Developed by Miles Stewart**

## Stack
- Frontend: Vanilla HTML/JS single-file deployed via GitHub Pages
- Backend: Supabase (Postgres + Auth + Edge Functions + RLS)
- AI: Anthropic Claude Sonnet 4.5 via Supabase Edge Function
- Data sources: BallDontLie, CBBD, KenPom, ESPN, NCAA-API

## Features (as of v18 — April 10 2026)
- Multi-factor Apex Model NIL valuation (PPG, position, years remaining, demand, conference)
- 9-tab dashboard: Home, Roster, Recruit, CRM, Filter, Budget, Build, Scout
- Shared CRM with vCard import and export
- Star-based reminder system with tier cadence (A=3d, B=7d, C=14d, D=30d)
- AI Scout with voice input via Web Speech API
- War room dashboard with pipeline funnel, top targets, position coverage
- Budget approval workflow with audit trail
- Coach Scorecard with 4 weighting modes
- 15-slot roster builder with auto valuation

## Repos
- **Private main (this repo):** github.com/Apexver1/apex-intel
- **Public dashboard:** github.com/Apexver1/Apex-App

## Phase status
- Phases 0-4: Schema and data seed complete
- Phase 5A-5E: Budget, Roster Builder, Approval, Scorecard, AI Scout complete
- Phase 6A.1-6A.2: CRM, vCard, Reminders complete
- Phase 6B pending: Load all 364 D1 schools
- Phase 6C pending: Portal scraping with daily refresh
- Phase 6D pending: HS top 250 per class (2026-2029)

## Directory layout
- dashboard/index.html — single-file frontend
- sql/migrations.sql — cumulative schema migrations
- supabase/functions/apex-scout/index.ts — LLM scouting Edge Function
- docs/ — phase planning docs
- schemas/ — initial schema scaffolding
- tools/ — data loading utilities
