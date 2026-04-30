# CLAUDE.md — Christian (Game Flow & State Machine)

> Hackathon mode. Ship fast. 2-hour build.

## Your Role

You own the state machine and the main game screen. You wire everyone else's work together. You merge last.

## Files You Own

Only edit these. Do not touch anything else.

- `/lib/gameFlow.js`
- `/app/game/[code]/page.jsx`
- `/components/PhaseRouter.jsx`

## Branch

`feat/game-flow`

## What You're Building

1. State machine: `lobby → scene → round1 → twist → round2 → vote → reveal`
2. `advancePhase(roomCode)` — host triggers, updates Firebase, all players move forward together
3. Main game screen that listens to room state and renders the correct phase
4. Host controls (Next Round button, only host sees it)
5. Per-phase timer (2 minutes per round, visible to all players)
6. PhaseRouter that imports Lebert's phase components and renders the right one based on `room.currentPhase`

## What You Depend On

- Joffre's `subscribeToRoom` and `updateRoom` from `/lib/room.js`
- Lebert's phase components from `/components/phases/*` — you import them, you don't edit them
- Jo's API route at `POST /api/generate` — you call it at phase transitions

## The Contracts You Consume

**Phase component prop shape (from Lebert):**
```javascript
{
  player: { id, name, role, secretMission },  // current player only
  room: { ...full room object... },
  onAction: (actionType, payload) => void
}
```

**API route (from Jo):**
```javascript
POST /api/generate
Body: { type: "sceneAndRoles" | "challenge" | "twist" | "finalResult", payload: {...} }
```

Call this at phase transitions:
- Entering `scene` → `type: "sceneAndRoles"`, write result to `room.gameData.scene` and `room.players[*].role`
- Entering `round1` and `round2` → `type: "challenge"`, write to `room.gameData.challenges`
- Entering `twist` → `type: "twist"`, write to `room.gameData.twist`
- Entering `reveal` → `type: "finalResult"`, write to `room.gameData.finalResult`

## Hour-by-Hour

- **0:00–0:15** — Wait for Joffre's scaffold to land on main. Pull it.
- **0:15–1:00** — Build the state machine and PhaseRouter using mock data and mock phase components (just `<div>{phase name}</div>` until Lebert's land).
- **1:00–1:30** — Integration. Pull main, wire in Joffre's room subscription, Lebert's real phase components, Jo's API route.
- **1:30** — Push. Test on real phones.

## Hackathon Rules

- Mock data is fine until integration
- Inline handlers are fine, ship it
- No tests
- One try/catch per API call is enough
- Don't optimize re-renders, the build is 2 hours not 2 weeks

## Hard Boundaries

- Do NOT edit Joffre's `/lib/room.js`. If you need a function that doesn't exist, ask him to add it.
- Do NOT edit Lebert's phase components. If a prop is wrong, ask him.
- Do NOT edit Jo's `/lib/prompts.js` or `/app/api/generate/route.js`. If the response shape is wrong, ask Jo.
- Do NOT edit `package.json`. Ask Jo.

## When You're Stuck

15 minutes max. Then post in the team channel.

## Before You PR

- [ ] Phase transitions work with mock room data
- [ ] PhaseRouter renders the right component for each phase
- [ ] Host clicking Next Phase updates Firebase and advances all connected clients
- [ ] AI calls happen at the right transitions and write to the right keys
- [ ] PR description: "Game flow + state machine + PhaseRouter. Wires room data to phase components. AI calls fire at phase entry."
