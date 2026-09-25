# AGENTS.md — NOT YOUR BUSINESS

## Purpose
This repository powers the live NOT YOUR BUSINESS experience. Work conservatively: preserve the existing production flow while building the Social Roleplay MVP in verifiable milestones.

## Source of truth
- Current implementation task: GitHub issue #1, `Codex: Social Roleplay Pool + WhatsApp Game Rooms MVP`.
- Engine construction guide: `docs/GAME_ENGINE_BUILD_GUIDE.md` (binding for engine primitives, state machine, template schema, game families, tests and build order).
- Concept/master planning note: `03_PLAN__Social_Roleplay__Matching_WhatsApp_Gruppen_Game_Engine__2026-09-25.md` in the project Drive.
- Do not invent a competing plan or silently broaden scope.

## Hard product constraints
- 18+ only.
- Standard group size: 3–4.
- Hard maximum: 5 players per session.
- 1:1 is a separate optional mode, not the default.
- No large-group / 6+ mode in the MVP.
- WhatsApp is the conversation space; the website/app is the game master.
- Do not store phone numbers in the app backend for the MVP.
- Do not scrape phone numbers or mass-add WhatsApp members.
- Deep Talk is always optional and skippable.
- Do not expose one player's private role or private payload to another player.

## Execution workflow
1. Inspect repository state, existing live behavior and tests before editing.
2. Use plan mode for multi-milestone work.
3. Work one milestone at a time.
4. After every milestone run the relevant lint/typecheck/tests/build.
5. If validation fails, stop and repair before proceeding.
6. Keep diffs scoped to the active milestone.
7. Record material architecture decisions and known limitations.
8. Do not claim DONE without readback/evidence for acceptance criteria.

## Production safety
- Existing character flow must keep working.
- Alter + Stadt must remain in the flow.
- Do not reintroduce roleplay scenes into production before Pool + Room assignment exist.
- Do not deploy speculative UX to production just to demonstrate progress.

## Matching constraints
- Hard filters first; soft scoring second.
- Randomness may only break near-ties; it is not the main matching method.
- Never produce a room with more than 5 assigned players.
- Test duplicate assignment, room capacity, repeat avoidance and waiting-time fairness.

## Authorization / privacy
Apply cross-user negative testing:
- Player A cannot read Player B's private role.
- Player A cannot manipulate Player B's assignment.
- A room invite link is available only to an assigned session/player.
- Server-side authorization must hold even if UI checks are bypassed.

## WhatsApp browser provisioning — approval gate
Browser automation that creates or changes WhatsApp groups is an external side effect.
Before doing it:
1. finish Pool + Matching + Game Engine tests,
2. produce a provisioning plan and exact intended room count/names,
3. verify browser access and logged-in WhatsApp Web,
4. obtain explicit user approval for the provisioning checkpoint.

After approval:
- create only the approved rooms,
- verify each room name/settings/invite link by readback,
- stop on warnings, rate limits, unexpected UI or ambiguous state,
- never improvise mass actions.

## Copyright/content
External games may inform abstract mechanics only. Do not copy proprietary cards, role text, scenarios, rule text or branded content. Write original templates.
