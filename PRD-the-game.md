# PRD — The Game (AI Location Game Master)

> Hackathon PRD. Locked before code starts.

## Hackathon Context

**Event:** [Event name + date]
**Track / theme:** [Pull from hackathon page]
**Team:** Jo (AI/prompts), Joffre (multiplayer/lobby), Christian (game flow/state machine), Lebert (player UI/phase screens)
**Time budget:** 2 hours build + demo prep

## Problem Statement

**Who is affected:** Groups of friends or family stuck doing the same party games (charades, Heads Up, drinking games) that don't adapt to who's in the room or where they are.

**What's broken:** Existing party games are static. They don't react to the actual location, the actual people, or the vibe of the night. Family game night and a chaotic 21+ hangout get the exact same prompts, which kills the energy of both.

**How we know it's real:** Personal experience. Every party game peaks once and then gets repetitive because the AI behind it (or the deck of cards) can't read the room.

## Target User

**Primary user:** A host setting up a hangout — family game night, friends pre-game, holiday gathering — who wants something fresh that fits the actual group, not a generic deck of cards.

**How they solve this today:** Charades, Heads Up, Cards Against Humanity, drinking games. All static, all the same every time, none of them adapt to who's playing or where you are.

**User needs:**
- As a host, I need to set the tone (family vs 21+) so the game fits my actual group.
- As a player, I need a secret role and mission so I have something to play toward beyond just answering prompts.
- As a group, we need shared challenges that feel specific to where we are right now, not generic.

## Solution

**One-liner:** An AI game master turns your real location into a mission zone, gives each player a secret role and shared challenges, and crowns the group at the end. Family Mode for wholesome chaos, 21+ Mode for the real kind.

**Core demo flow (the 2-minute story):**
1. Host creates a room, picks location + mode (Family or 21+) → gets a 4-letter code
2. Players scan QR code, join lobby, see each other live
3. Host hits Start → AI generates the scene, assigns secret roles + missions to each player privately
4. Group plays through challenges, then a mid-game twist bends the rules
5. Players vote on who they think is the Traitor → AI delivers final results and group title

## Feature Scope

**P0 (demo-critical):**
- [ ] Room creation with 4-letter code + Family/21+ mode toggle
- [ ] Join room flow, lobby with live player list
- [ ] Phase state machine: lobby → scene → round1 → twist → round2 → vote → reveal
- [ ] AI scene + role generation (private role reveal per player)
- [ ] AI challenge generation (shared, mode-aware)
- [ ] AI twist generation
- [ ] Voting screen + AI final result
- [ ] Mobile-first UI (this is played on phones)
- [ ] Deployed to Vercel with QR code for judges to join

**P1 (only if P0 is locked):**
- [ ] Shareable result card on reveal screen
- [ ] Per-phase timer visible to all players
- [ ] Multiple location options beyond the demo default

**Explicitly out of scope:**
- User accounts, login, persistence across games
- Game history or replay
- More than one active room per session
- Sound effects or music
- Spectator mode

## Data Sources

| Data | Source | Format | Auth needed? |
|------|--------|--------|--------------|
| Player + room state | Firebase Realtime Database | JSON | Anon auth, set up at hour 0 |
| Game content (scene, roles, challenges, twist, result) | Claude API (claude-sonnet-4-20250514) | JSON via prompt | API key in Vercel env vars |
| Fallback game content | Local JSON cache (3 pre-generated sessions) | JSON | None |

## Tech Stack

**Frontend:** Next.js (App Router) + Tailwind, mobile-first
**Backend / data:** Firebase Realtime Database for room sync
**AI / APIs:** Claude API via Next.js API route (`/api/generate`) so the key stays server-side
**Deployment:** Vercel auto-domain, deploy at hour 1:30 not hour 1:55

## Demo Plan

**Hook (15 sec):** "We built an AI-powered party game that turns your real location into a mission zone."

**Problem (10 sec):** "Party games are static. They don't read the room. Family night and a 21+ hangout get the same prompts."

**Solution (60 sec live):** "Pull out your phones, scan this QR code." Team plays a live round in front of judges. Judges become spectators of a real game.

**Close (15 sec):** "Family Mode for wholesome, 21+ Mode for chaos. Same engine, completely different vibe. The AI is the game master."

**Backup plan if live demo fails:** "Use Demo Mode" toggle loads a pre-cached game from local JSON instead of hitting the API. Pre-cache 3 sessions: Park/Family, Park/21+, Home/Family.

## Judging Criteria Alignment

> Pull actual criteria from the hackathon page and update this table.

| Criterion | How we hit it |
|-----------|---------------|
| Creativity | Audience mode toggle (Family / 21+) — same engine, two completely different products |
| Technical execution | Realtime multiplayer + AI generation + private per-player state in 2 hours with 4 builders |
| User experience | Mobile-first, QR code entry, live demo with judges as spectators |
| AI use | Claude is the actual game master, not a chatbot bolted on. Mode controls tone end-to-end. |

## Build Timeline

| Time | Goal | Owner |
|------|------|-------|
| 0:00–0:15 | All four agree on data shape + API contract. Joffre creates Firebase + Vercel projects. Jonel adds API key. | All |
| 0:15–1:00 | Parallel solo build with mock data. No talking unless blocked. | All |
| 1:00–1:30 | Integration. Christian wires room data + phase components + AI calls. Push to Vercel. | Christian leads |
| 1:30–1:50 | Playtest a full round on real phones. Fix in priority order: lobby > state sync > AI > UI polish. | All |
| 1:50–2:00 | Demo prep. Pre-load cached game. QR code ready. Jo practices 45-sec pitch. Pick which mode to demo (Family is safer for judges, 21+ is more memorable). | All |

## Risks

- [ ] **Claude API slow or fails mid-demo.** Mitigation: pre-cache 3 full sessions, "Use Demo Mode" toggle.
- [ ] **One builder finishes late and blocks integration.** Mitigation: at hour 1:00, whoever isn't done ships with mock data. Integration happens with whatever exists.
- [ ] **Phones can't reach deployed app.** Mitigation: deploy at 1:30, test on real phones immediately.
- [ ] **Judges don't want to pull out their phones.** Mitigation: team plays the live round on our own phones, judges spectate.
- [ ] **Two builders touch the same file.** Mitigation: PLAN.md ownership boundaries are absolute.

## Open Questions

- [ ] Which location options ship for the demo? (Default: Park, Home. Confirm at hour 0.)
- [ ] Which mode does Jo pitch with? (Decide at hour 1:50 based on which is more polished and the judge audience.)
