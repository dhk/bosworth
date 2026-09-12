# Bosworth

**A personal capability and continuity layer for AI agents.**

Bosworth is intended to let a person move between Claude, ChatGPT, coding
agents, devices, and underlying systems without repeatedly reconstructing
context or losing track of work products. Existing AI applications remain the
conversational interface; Bosworth supplies durable continuity and controlled
access to capabilities behind them.

The core proposition is:

> **Start anywhere. Bosworth knows where you left off.**

Bosworth is not primarily a chat client or an MCP aggregator. MCP and APIs are
the adaptor layer. The durable product is the continuity layer.

## Status

Bosworth is in design and feasibility testing. No service runtime or supported
client integration exists yet.

- [x] Product hypothesis and Phase 1 boundaries defined
- [x] Minimal Session Event Registry v0.1 specified
- [ ] Registry service implemented
- [ ] A client producer emits session lifecycle events
- [ ] “Hello Bosworth” acceptance test passes end to end
- [ ] Capability Registry contract specified
- [ ] Artifact Registry contract specified
- [ ] Cross-session context assembly implemented

See [issue #1](https://github.com/dhk/bosworth/issues/1) for the first
executable milestone.

## Phase 1

Phase 1 is a service/runtime exposing six related primitives:

| Primitive | Question it answers |
|---|---|
| Capability Registry | What can my agents access and do? |
| Session Store | What happened during a particular interaction? |
| Journal | What significant things happened over time? |
| Memory | What should the system currently remember or believe? |
| Artifact Registry | What exists, which instance is canonical, and where does it live? |
| Context Assembler | What does this interaction need to know right now? |

The first implementation slice is intentionally narrower than Phase 1:
instrument session existence and human interaction before attempting memory,
semantic extraction, or context injection.

## Architecture direction

```text
Claude / ChatGPT / CLI / IDE
             |
          Bosworth
             |
   continuity + capabilities
             |
       MCP / APIs / tools
             |
       user's systems
```

Existing AI clients remain the interface initially. A bespoke iPhone client is
explicitly deferred until the service proves that current clients cannot
provide the required experience.

Secure capability access must eventually make authentication, permissions,
risk level, approval requirements, and availability inspectable. MCP is the
preferred adaptor protocol, but Bosworth must not depend on MCP being the only
integration mechanism.

## First experiment

[Session Event Registry v0.1](docs/design/session-event-registry-v0.1.md)
defines a metadata-only, append-only event stream for:

- `SESSION_STARTED`
- `INTERACTION_OCCURRED`
- `SESSION_HEARTBEAT`
- `SESSION_ENDED`

Success means another process can establish that a session existed and that a
human interaction occurred within it. It does **not** mean Bosworth can yet
remember, summarize, or continue that session.

## Privacy boundary

The v0.1 registry records mechanics, not conversation content. The “Hello
Bosworth” acceptance test must prove that lifecycle events are observable while
the event database contains neither the human message nor the assistant reply.

Future transcript, memory, and context systems require separate data contracts
and privacy policies.

## Project continuity

Read [HANDOFF.md](HANDOFF.md) before starting work. It is the living statement
of current state, the next task, and known watch points. Standing design
decisions belong under `docs/design/`.

## License

[MIT](template/LICENSE)
