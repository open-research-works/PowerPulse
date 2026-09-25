---
name: social-roleplay-mvp
description: Implement the NOT YOUR BUSINESS Social Roleplay MVP safely and milestone-by-milestone. Use for Pool, Matching, Game Engine, private roles, WhatsApp room routing, and later browser provisioning.
---

# Social Roleplay MVP workflow

Use GitHub issue #1 as the active task specification. Do not replace it with a parallel plan.

## Phase A — Discovery / plan
- Read `AGENTS.md`.
- Inspect the current source and production behavior.
- Read issue #1 completely.
- Read `docs/GAME_ENGINE_BUILD_GUIDE.md` completely and treat it as the binding engine/design build guide.
- Produce a milestone plan before implementation.
- Identify unresolved product or data-model decisions.
- Do not create WhatsApp groups in this phase.

## Phase B — Data model + security tests
Define the smallest schema for:
- player/session
- pool entry
- room
- match assignment
- game template
- game run
- private role/payload

Constraints:
- standard room size 3–4
- hard max 5
- 18+
- no phone number field
- private-role access enforced server-side

Write authorization tests before or alongside implementation.

## Phase C — Pool + matching
Implement:
- join pool
- waiting state
- hard-filter eligibility
- soft scoring
- group formation
- room capacity
- repeat avoidance
- deterministic test fixtures

Validation must include simulated users and prove:
- no duplicate assignments
- no room >5
- no hard-rule violation
- reasonable waiting-time handling

## Phase D — Game engine
Create original templates only.

MVP template families:
1. Narrative Roleplay — 3–4
2. Secrets / asymmetric information — 4–5
3. Mission / light social deduction — 3–5
4. REAL / DEEP progression — 3–5

Template selection must consider:
- player count
- compatible modes
- intensity
- complexity
- recently played templates

Randomness may only be a tie-breaker.

## Phase E — WhatsApp room routing
Before browser provisioning, support preconfigured room records and invite links.

Flow:
`Pool -> Match -> Room assignment -> Join screen -> Confirm joined -> Ready -> Game run`

Show the WhatsApp invite link only to assigned players.

Before join, show a clear notice that normal WhatsApp groups may expose a member's phone number/profile information to other members.

## Phase F — Browser provisioning
This phase is gated by explicit user approval.

Prepare first:
- exact number of rooms
- exact names
- city/region
- capacity (3–5)
- intended settings
- where invite links will be stored

Then, with WhatsApp Web logged in:
- create only approved rooms
- configure intended settings
- obtain invite link
- store/configure link
- read back and verify each room
- stop on unexpected UI, warnings or rate limits

Do not:
- scrape phone numbers
- mass-add contacts
- create 6+ player game sessions
- create a new WhatsApp group for every match in MVP

## Phase G — Verification
Before reporting completion:
- lint/typecheck/build/tests pass
- current character flow smoke-tested
- Alter + Stadt still work
- cross-user negative authorization tests pass
- complete simulated flow passes
- production changes are read back after deployment
- remaining risks and deferred work are documented
