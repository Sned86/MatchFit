# CLAUDE.md — Session Bootstrapping Guide

> **Read this file first at the start of every session.** It tells you what this project is, what's been built, what's next, and how we work together.

---

## Project Summary

**MatchFit** is a web platform for non-league football clubs to arrange pre-season fixtures, and to connect clubs with available referees for those fixtures. The core problem: pre-season fixture arrangement is currently done informally (emails, phone calls, social media DMs) with no central coordination. MatchFit fixes that.

### Product strategy & USP

MatchFit's target users are club secretaries and managers at non-league clubs (Step 1–7 and below) who are not particularly tech-savvy. The experience must be simple and fast — no jargon, no unnecessary steps. A club secretary should be able to post a fixture request or confirm a match in under two minutes.

The referee-matching side is equally important: referees at this level often struggle to find games, and clubs struggle to find available officials. MatchFit is the bridge.

**Design principle:** Every feature should pass the "would a club secretary at a Saturday afternoon club actually use this?" test. Prioritise clarity over features.

**Current version:** v0.1.0  
**Owner:** Ed — matchfitpsf@gmail.com  
**Status:** MVP in active development — core flows built, partially wired end-to-end

---

## How We Work

### About me
I am the project owner. I do not write code — Claude handles all coding. Please don't assume I remember details from previous sessions; use this document and the project state section below to re-orient at the start of each session.

### Communication style
- **Summarise changes** at a high level — what changed and why, not line-by-line walkthroughs
- **Explain new concepts simply** when they come up; avoid assuming prior technical knowledge
- **Small decisions** (naming, minor implementation choices): just make the call and note it briefly
- **Bigger decisions** (architecture, new dependencies, anything that affects scope or the roadmap): present options with a clear recommendation before building
- **Never ask more than one question at a time**

### Sessions
Sessions may involve building new features, fixing bugs, reviewing what's been built, or planning what's next. At the end of every session, update the **Current Project State** section below before closing.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Vanilla JavaScript / Static HTML (single file) |
| Styling | Custom CSS with CSS variables |
| Database | Supabase (PostgreSQL) |
| Auth | Supabase Auth (REST API, direct fetch calls) |
| Storage | Supabase Storage (club badge images) |
| Hosting | GitHub |
| Fonts | DM Sans, DM Serif Display (Google Fonts) |

---

## Core Data Model

The key entities in MatchFit:

- **Club** — a non-league football club. Fields include: name, league, location, county, lat, lng, ground, badge_url, active, nickname, home_colours, away_colours, contact_name, contact_email, contact_phone, website_url, facebook_url, twitter_url, instagram_url
- **Availability** — a club posting an open slot. Fields: club_id, date, venue (home/away/either/neutral), kickoff_pref (array), max_distance, notes, status (open/cancelled)
- **Fixture** — a potential or confirmed match. Fields: club_a_id, club_b_id, proposed_date, proposed_time, venue_type, status (available/offered/under_offer/confirmed/changed/cancelled/completed). `venue_type` uses values `home_a`, `home_b`, `neutral` (constraint: `fixtures_venue_type_check`)
- **Fixture History** — full audit trail of status changes on a fixture
- **User** — a registered user who has applied to control a club; approved or rejected by Ed
- **User_clubs** — junction table linking users to clubs. One user can control several clubs; one club can only have one active user. Has a `status` field: pending / approved / rejected, plus optional `rejection_reason`
- **User_Referee** — an available official (name, location, qualifications, contact) — not yet built
- **Referee Assignment** — links a referee to a fixture — not yet built

---

## File Structure

```
matchfit/
├── index.html                # Entire app — HTML, CSS and JS in one file
├── CLAUDE.md                 # This file
└── supabase/
    └── migrations/           # Database schema migrations (PostgreSQL)
```

> The app is currently a single HTML file. All UI, logic and Supabase calls live in index.html. This is intentional for the MVP — no build step, easy to iterate.

---

## Design System

- **Fonts:** DM Sans (UI), DM Serif Display (headings/logo)
- **Colours:** Green (`#16a34a`) and Navy (`#0f172a`) as primaries, with full semantic scale (green-light, green-xlight, green-mid, slate, amber, blue, red) defined as CSS variables
- **Radius:** `--radius-sm` (8px), `--radius` (14px), `--radius-lg` (20px)
- **Max width:** 480px — mobile-first throughout
- **Shadows:** Three levels (`--shadow-sm`, `--shadow`, `--shadow-lg`)
- **Logo/header image:** Hosted on Supabase Storage at `huokcisnlcaezqcroyon.supabase.co`

---

## Conventions & Workflow

### Git workflow
- Branch from `main`: `feature/description`, `fix/description`
- Keep commits focused — one logical change per commit
- Write clear commit messages: what changed and why

### Supabase
- All schema changes must be done via migration files in `supabase/migrations/` — never edit the database directly in the Supabase dashboard
- Use Row Level Security (RLS) on all tables — no exceptions
- Never put Supabase keys or secrets in committed code; use `.env.local` only
- **Note:** The anon key is currently embedded in index.html for the MVP. This is acceptable for now but should be moved before any public launch.

### Single-file approach
- All app code lives in `index.html` — HTML structure, `<style>` block, and `<script>` block
- This keeps the MVP simple and removes build complexity
- When the codebase grows to the point where this becomes painful, we'll discuss splitting it out

### Hard rules
- **Never commit secrets or API keys** to public repos
- **No direct database edits** outside of migrations — keeps schema history clean
- **Always test the happy path and at least one error case** before marking a feature done

---

## Roadmap

### Phase 1 — MVP (current focus)
- [x] Club database loaded from Supabase
- [x] Auth wall — login, register, pending/rejected states
- [x] Club claim flow — user registers and requests a club; Ed approves/rejects
- [x] Post availability (date, venue, kick-off, distance, notes)
- [x] Edit and cancel availability
- [x] Find a fixture — search with date/venue/kick-off/distance filters
- [x] Results sheet with club cards, distance, travel time
- [x] Shortlist clubs
- [x] Offer a match modal — with required field logic and counter-proposal
- [x] Browse all clubs with league filter
- [x] Team profile sheet
- [x] My Club screen — stats, editable details, pre-season calendar
- [x] My Fixtures screen — Open / Offers / Confirmed sub-tabs
- [x] Offer a match writes to Supabase fixtures table
- [x] Offers tab shows received offers (with Accept/Decline) and sent offers (awaiting state)
- [ ] Have a proper check of how 'Home' and 'Away' shows from the point of view of each team
- [ ] Confirmed tab wired to real data — accepted offers should move here
- [ ] Fixture negotiation fully tested end-to-end (see known issues)
- [ ] Notifications screen wired to real data — UI exists, empty state only
- [ ] Referee registration and availability
- [ ] Assign a referee to a confirmed fixture

### Phase 2 — Post-MVP
- [ ] Email notifications when a match is confirmed or a referee assigned
- [ ] Club dashboard — upcoming fixtures, pending requests
- [ ] Referee dashboard — upcoming assignments, available fixtures nearby
- [ ] Search/filter fixture requests by region, date, level
- [ ] Admin dashboard (visual mockup previously built as separate file)

### Phase 3 — Future
- [ ] Ratings / reviews for referees
- [ ] Integration with league management systems
- [ ] Mobile app / PWA

---

## Current Project State

> **Update this section at the end of every session.**

**Last updated:** 27 March 2026  
**Version:** v0.1.0

### What's built

**Auth**
- Login and register flows using Supabase Auth REST API
- Register form includes club search — user selects their club from the database
- On registration, a `user_clubs` row is inserted with `status: pending`
- On login, the app checks `user_clubs` status and routes to: pending screen / rejected screen / full app
- Sign out clears session from sessionStorage and resets app state
- Session persisted in `sessionStorage` so users don't need to re-login on refresh
- Registration flow fully tested and working end-to-end (register → pending → approved → full app; rejected flow also tested)
- `notify_registration_email` trigger function fixed — was calling `net.http_post` with wrong argument types (text instead of jsonb)

**Club data**
- Clubs table in Supabase, loaded on app init via REST fetch
- `active` boolean filters out inactive clubs
- Club badges loaded from `badge_url` (Supabase Storage), falls back to initials avatar
- Haversine distance calculation between MY club and all others
- RLS policy: anon role can SELECT from clubs (required for register form club search)

**Availability**
- Post availability modal: date picker, venue seg control, kick-off pills (contextual weekday/weekend times), distance slider, notes
- Edit existing availability (pre-fills modal, PATCHes existing row)
- Cancel availability (PATCHes status to cancelled)
- My Fixtures > Open tab renders live availability cards from Supabase
- Open availability cards show an amber "X offers received" badge when offers exist against that date
- RLS policies added: authenticated users can INSERT and UPDATE their own availability; authenticated users can SELECT all open availability

**Find a Fixture**
- Multi-select date picker with upcoming Sat/Sun/Tue/Wed pre-populated + custom date picker
- Venue filter (any/home/away/neutral), kick-off multi-select, distance slider with "any distance" toggle
- Specific club search with dropdown
- Search filters against CLUBS array (availability-matched clubs only shown in results)
- Results slide-up sheet with club cards showing distance, travel time, date, kick-off, venue
- Shortlist toggle on each result card; shortlist modal with offer/remove actions

**Offer a Match**
- Opens from result card, browse card, team profile, or shortlist
- Banner shows the target club's details and fixture summary (date/kick-off/venue)
- Required fields section: if a club left date/kick-off/venue open, the offering club must specify them
- Counter-proposal section: propose a different date/kick-off/venue from what the club specified
- Banner summary updates live as fields are filled
- **Wired to Supabase**: submitting an offer inserts a row into `fixtures` with `status: offered`
- `venue_type` mapped correctly: home→`home_a`, away→`home_b`, neutral→`neutral` (per DB check constraint)
- RLS policies on fixtures: authenticated users can INSERT; involved clubs can SELECT and UPDATE their own fixtures

**Browse**
- All clubs listed, sorted by distance
- League filter tabs built dynamically from loaded club data
- Clicking a club opens the team profile sheet

**Team Profile**
- Shows club stats, availability slots, club details
- "Propose a Match" button opens offer modal

**My Club**
- Profile header with badge, name, location, league, stats (confirmed/unconfirmed/open slots)
- Pre-season calendar — month view, colour-coded dots for open/offer/confirmed dates, day detail panel on tap
- Club Details section — view and edit mode for contact info, colours, social links
- Edits saved back to Supabase via PATCH

**My Fixtures**
- Three sub-tabs: Open, Offers, Confirmed
- Open tab: renders live availability from Supabase with edit/cancel actions; shows offer count badge when offers exist
- Offers tab: **now wired to real data** — received offers show Accept/Decline buttons; sent offers show "Awaiting response" with pulse dot
- Confirmed tab: empty state only — not yet wired
- Fixtures reloaded with authenticated token after login (fixes RLS issue where club_b couldn't see their incoming offers)

### What's next (start here next session)
1. **Fix: accepted offer should move to Confirmed tab** — when Team A accepts an offer, the fixture updates to `confirmed` in Supabase but the UI keeps it in Open rather than moving it to the Confirmed tab. The `renderMyFixtures` Confirmed tab section needs to be wired up to show fixtures with `status: confirmed`.
2. **Test: multiple offers against one availability slot** — a club's open availability should be able to receive offers from multiple clubs simultaneously. Need to verify the UI handles this correctly (each offer shown separately in the Offers tab, offer count badge on the Open tab card increments correctly).
3. Wire notifications to real events
4. Begin referee registration flow

### Known issues / decisions pending
- **Accepted offer doesn't move to Confirmed tab** — `acceptOffer()` correctly PATCHes the fixture to `confirmed` in Supabase, but `renderMyFixtures` doesn't yet render the Confirmed tab with real data. The fixture also remains visible in Team A's Open tab rather than being removed.
- A multiple-offers scenario has not yet been tested — behaviour is designed but unverified
- Email confirmation is **off** in Supabase — intentional; Ed manually approves/rejects registrations
- Supabase anon key is hardcoded in index.html — acceptable for MVP, needs moving before public launch
- RLS policies not fully audited — fixtures, availability and clubs have policies; other tables may need review
- `postcodes.io` integration not yet built; lat/lng currently entered directly into the database
- No email notifications yet
- The `level` field on clubs is mapped as an empty string in `mapClub()` — not currently displayed in UI; can be derived from `league` name if needed later

---

## Key Constraints & Context

- **Target users are not tech-savvy** — UI must be extremely simple and forgiving
- **Mobile is important** — club secretaries will often be on their phones
- **Trust matters** — clubs need to trust that the other party is a real, legitimate club. The pending/approved/rejected registration flow is the first layer of this
- **Geography is core** — fixture requests and referee matching should always be aware of location/travel distance
- **Single file for now** — all code in index.html; don't split into separate files without discussing first
