# Handoff

**Last updated:** 2026-09-12  
**Active branch:** `docs/session-event-registry-v0.1`  
**Tracking issue:** [#1 — Establish the Session Event Registry v0.1 experiment](https://github.com/dhk/bosworth/issues/1)

Read this file and
[`docs/design/session-event-registry-v0.1.md`](docs/design/session-event-registry-v0.1.md)
before beginning work.

This file is the living statement of current state. Update it at the end of
each working session. Permanent rationale belongs under `docs/design/`.

## Current state

Bosworth is a product hypothesis with one bounded technical design. The
repository does not yet contain an implemented service, database, client
producer, automated test, or supported integration.

The canonical v0.1 design specifies:

- an append-only Session Event Registry;
- four event types: start, interaction, heartbeat, and end;
- SQLite as the initial store;
- metadata-only recording with no conversation content;
- derived `active` and `stale` state;
- idempotency through producer-generated event IDs;
- explicit handling of missing, duplicate, late, and partially observed events.

Phase 1 remains a broader continuity runtime: Session Store, Journal, Memory,
Artifact Registry, Context Assembler, Capability Registry, and MCP/API
integration. Those components are planned, not implemented.

A bespoke iPhone client remains deferred. Existing AI clients should remain the
interface until evidence shows that this is insufficient.

## What's next

Implement the smallest local vertical slice against **Claude Code CLI only**:

1. Create a SQLite-backed registry with `POST /events`,
   `GET /sessions/{session_id}/events`, and `GET /sessions/active`.
2. Create a deliberately simple producer or test harness that emits the four
   lifecycle events for one session.
3. Run the “Hello Bosworth” / “Hello Angel” acceptance test.
4. Assert that the ordered session history is correct and that neither message
   string exists in the database.
5. Record which observations came from real Claude Code hooks and which were
   simulated by the harness.

Do not begin memory, semantic extraction, the Artifact Registry, or additional
client surfaces in this slice.

## Completion criterion

A reproducible command starts the registry, runs one instrumented or simulated
Claude Code session, retrieves the expected lifecycle, and verifies that
conversation content was not stored. The README must then be updated to
distinguish exactly what passed from what remains simulated or unverified.

## Known issues / watch points

1. **Instrumentation is unproven** — the design intentionally does not claim
   that Claude Code, Desktop, web, or iPhone exposes the required hooks.
2. **Session identity may be observer-local** — a generated identifier must not
   be presented as a native product conversation ID.
3. **An absent end signal is uncertainty** — derive `stale`; never invent
   `SESSION_ENDED`.
4. **Delivery guarantees vary by surface** — claim at-least-once delivery only
   where durable local queuing is actually available.
5. **Repository settings are not yet verified** — the inherited template
   documents recommended branch protection, but that does not prove the setting
   is enabled.

## Open questions

- Which Claude Code mechanism can observe session start, interaction, and end?
- Is a stable native session identifier exposed?
- Can the producer distinguish completed interactions from streaming activity?
- Can a hook emit externally without capturing message content?
- What timeout should turn an apparently active session into `stale`?
- Which security and privacy constraints apply to local observation?
- How should logical conversation identity be preserved if a conversation moves
  between clients or devices?
