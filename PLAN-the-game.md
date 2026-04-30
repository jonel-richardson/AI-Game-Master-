# PLAN — The Game (AI Location Game Master)

> Team coordination plan. Ownership boundaries are absolute.
> Goal: 4 builders, 2 hours, zero merge conflicts.

## Team

| Builder | Role | Branch |
|---------|------|--------|
| Joffre | Multiplayer & lobby | `feat/multiplayer` |
| Christian | Game flow & state machine | `feat/game-flow` |
| Lebert | Player UI & phase screens | `feat/phase-ui` |
| Jonel (Jo) | AI game master | `feat/ai-gm` |

**Merge owner:** Jo. Reviews PRs, merges to main, owns the deploy pipeline after Joffre sets it up.

## Repo Structure (locked at hour 0, do not rename)

```
/the-game
├── /app
│   ├── page.jsx                      # Joffre — landing page
│   ├── /lobby/[code]/page.jsx        # Joffre — waiting room
│   ├── /game/[code]/page.jsx         # Christian — main game screen
│   ├── /api/generate/route.js        # Jo — Claude API route
│   └── globals.css                   # Lebert — styling
├── /components
│   ├── PhaseRouter.jsx               # Christian — routes to phase components
│   └── /phases
│       ├── SceneScreen.jsx           # Lebert
│       ├── RoleRevealScreen.jsx      # Lebert
│       ├── ChallengeScreen.jsx       # Lebert
│       ├── TwistScreen.jsx           # Lebert
│       ├── VoteScreen.jsx            # Lebert
│       └── RevealScreen.jsx          # Lebert
├── /lib
│   ├── firebase.js                   # Joffre — Firebase config
│   ├── room.js                       # Joffre — create/join/sync rooms
│   ├── gameFlow.js                   # Christian — state machine, transitions
│   ├── prompts.js                    # Jo — prompt templates
│   ├── claude.js                     # Jo — API client + parsing
│   └── /cache                        # Jo — pre-generated fallback sessions
│       ├── park-family.json
│       ├── park-21plus.json
│       └── home-family.json
├── PRD.md
├── PLAN.md
└── README.md
```

## Ownership Boundaries

### Joffre owns
- `/lib/firebase.js`
- `/lib/room.js`
- `/app/page.jsx`
- `/app/lobby/[code]/page.jsx`

### Christian owns
- `/lib/gameFlow.js`
- `/app/game/[code]/page.jsx`
- `/components/PhaseRouter.jsx`

### Lebert owns
- `/components/phases/*` (all 6 phase screens)
- `/app/globals.css`

### Jo owns
- `/lib/prompts.js`
- `/lib/claude.js`
- `/app/api/generate/route.js`
- `/lib/cache/*` (the 3 fallback JSON files)

### Shared files (only the merge owner edits during the build)
- `package.json` — Jo adds dependencies. Others request via team chat.
- `.env.example` — append-only. Never delete or rename existing keys.
- `README.md` — Jo edits during the build.
- `next.config.js`, `tsconfig.json`, `tailwind.config.js` — Jo only.
- `.env.local` — each builder maintains their own. The Vercel project has the canonical set.

### No-touch zones
- `main` branch — no direct commits. PRs only.
- Other people's feature branches — read-only.

## Interface Contracts

### Contract 1: Room Data Shape (Joffre owns, everyone reads)

This is the source of truth. Locked at hour 0. Do not change without team agreement.

```javascript
// /rooms/{ROOM_CODE} in Firebase
{
  code: "PLAY",
  hostId: "player_abc",
  mode: "family" | "21plus",
  location: "park",
  status: "lobby" | "playing" | "voting" | "reveal",
  currentPhase: "scene" | "round1" | "twist" | "round2" | "vote" | "reveal",
  players: {
    "player_abc": {
      name: "Jo",
      role: null,           // populated by AI in scene phase
      secretMission: null,  // populated by AI in scene phase
      votes: 0
    }
  },
  gameData: {
    scene: "",
    challenges: [],
    twist: "",
    finalResult: null
  }
}
```

**Functions Joffre exposes from `/lib/room.js`:**
```javascript
createRoom(hostName, mode) → Promise<{ code, hostId }>
joinRoom(code, playerName) → Promise<{ playerId }>
subscribeToRoom(code, callback) → unsubscribe function
updateRoom(code, partialUpdate) → Promise<void>
```

### Contract 2: AI API Route (Jo owns, Christian consumes)

```javascript
POST /api/generate
Body: {
  type: "sceneAndRoles" | "challenge" | "twist" | "finalResult",
  payload: { ... }
}
```

**Per-type payloads and responses:**

```javascript
// type: "sceneAndRoles"
payload: { location, mode, players: [{ name }] }
returns: {
  scene: string,
  assignments: [{ playerName, role, secretMission }]
  // exactly one player has role === "The Traitor"
}

// type: "challenge"
payload: { location, mode, roundNumber }
returns: { challenge: string }

// type: "twist"
payload: { location, mode }
returns: { twist: string }

// type: "finalResult"
payload: { location, mode, players, votes }
returns: {
  groupTitle: string,
  mvp: string,        // player name
  individualTitles: [{ playerName, title }]
}
```

**Fallback behavior:** If the API call fails or takes longer than 8 seconds, Jo's route returns from the local cache (`/lib/cache/*.json`). Christian doesn't need to handle the fallback — it's transparent.

### Contract 3: Phase Component Props (Lebert owns, Christian renders)

Every phase component takes the same prop shape:

```javascript
{
  player: { id, name, role, secretMission },  // current player only
  room: { ...full room object... },
  onAction: (actionType, payload) => void
}
```

**`onAction` types per phase:**
- SceneScreen: `("ready")` — player marks ready
- RoleRevealScreen: `("acknowledged")` — player has seen their role
- ChallengeScreen: `("complete")` — player completed the challenge
- TwistScreen: `("acknowledged")`
- VoteScreen: `("vote", { targetPlayerId })`
- RevealScreen: `("share")` or `("playAgain")`

## Branch Strategy

- Every builder works on `feat/[name]` off main.
- Commit at least once per hour. Push so Jo can see progress.
- No rebasing or force-pushing on a branch anyone else might be looking at.
- PRs target `main`. Jo reviews and merges.
- After every merge to main, every builder pulls main into their feature branch immediately.

## Merge Sequence

1. **First to merge:** `feat/multiplayer` (Joffre). Sets the foundation — Firebase, room data shape, lobby flow. Everything else depends on this.
2. **Second:** `feat/ai-gm` (Jo). The API route is independent and unblocks Christian's integration work.
3. **Third:** `feat/phase-ui` (Lebert). Phase components are leaf nodes — they only depend on the prop contract.
4. **Last:** `feat/game-flow` (Christian). Wires everything together. Merges last because it imports from all three other branches.

## Communication

**Channel:** [Pick one — Slack, Discord, iMessage thread — and stick to it]

**Mandatory check-ins:**
- **0:00 — Kickoff.** Walk through this plan together. Confirm everyone understands their files and the contracts. Lock the room data shape and the API contract.
- **1:00 — P0 status check.** Whoever isn't done ships mock data. No one waits.
- **1:30 — Integration freeze.** Stop adding features, start merging. Deploy to Vercel.
- **1:50 — Demo run-through.** Full round on real phones.

**Blocker protocol:** Stuck for more than 15 minutes? Post in the channel. No silent struggling.

## Conflict Resolution

When two builders need to edit the same file:

1. Stop. Don't both push.
2. Post in the channel: "I need to edit [file] for [reason]. Anyone else in there?"
3. Smaller change waits. Bigger change merges first, smaller rebases on top.
4. Jo has final say.

## Definition of Done (per builder)

Before opening a PR, your branch must:
- [ ] Run locally without errors
- [ ] No console errors in the browser
- [ ] Match every interface contract above
- [ ] PR description: what changed, what it depends on, what to test
- [ ] Tested on a real phone (not just desktop)

## Risks Specific to Parallel Builds

- [ ] **Christian's integration depends on all 3 other branches.** Mitigation: he builds against mocks until hour 1:00, then integrates whatever exists.
- [ ] **Jo's API route depends on the prompt templates working end-to-end.** Mitigation: pre-cache the 3 fallback sessions FIRST so the demo is safe even if the live API has issues.
- [ ] **Lebert's phase components depend on the prop contract.** Mitigation: contract is locked at hour 0. Lebert builds against mock props.
- [ ] **Joffre's Firebase setup blocks everyone.** Mitigation: Firebase is the first thing built, hour 0–0:15. If it's not working by 0:15, Jo escalates.
