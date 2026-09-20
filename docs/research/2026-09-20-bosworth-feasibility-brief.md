# Bosworth: Feasibility Research Briefing

**Date:** September 20, 2026  
**Status:** Recommendation: proceed with a narrowly scoped proof of concept  
**Decision horizon:** 6–10 week pilot

## Executive decision

Bosworth is technically feasible and addresses a real enterprise-agent problem: reusable AI capabilities are increasingly fragmented across Agent Skills, MCP servers, agents, and ordinary APIs. A user or host agent should not need to know which individual skill or tool implements a desired outcome.

Bosworth should be built as a governed, team-oriented capability control plane: it lets an organization package these underlying assets as discoverable teams, describe the work each team is competent and authorized to perform, and provide a stable, auditable invocation contract.

The recommendation is to proceed, with an important product boundary: do **not** position the initial product merely as “an MCP server in front of other MCP servers.” MCP federation, proxying, API adaptation, and gateway policy are established patterns. Bosworth’s differentiated value is the semantic and operating layer above those mechanisms: outcome-oriented discovery, team contracts, authorization-aware routing, lifecycle management, evaluation, provenance, and organizational ownership.

## Problem statement

AI capability discovery has become a coordination problem. A capable organization may have:

- Agent Skills containing instructions, workflow knowledge, scripts, templates, and references.
- MCP servers exposing tools, resources, and prompts.
- Specialized agents that can carry out multi-step work.
- REST and gRPC APIs that still need wrappers, policy, and discoverability.
- Internal documentation and institutional knowledge distributed among teams.

Raw capability enumeration does not scale well. A host exposed to hundreds of tools sees overlapping names, unclear scope, inconsistent schemas, uneven security posture, and too much implementation detail. A human user similarly faces the burden of finding the right skill, checking whether it applies, learning its inputs, and understanding its permissions.

Bosworth changes the discovery unit from an implementation primitive to a responsible operational role. Instead of asking a person or agent to find `eligibility_check`, `retrieve_case_documents`, and `payer_policy_search`, it offers an `Eligibility Operations` team that can explain its scope, permissions, expected outputs, limitations, and approval requirements.

## Product thesis

> Bosworth turns distributed skills, agents, MCP services, and APIs into governed, outcome-oriented teams that people and AI agents can discover, evaluate, and invoke through a stable contract.

A Bosworth team is a versioned, owned, policy-bound composition of capabilities for a bounded class of work. A team may expose one or more workflows, but it is not merely a friendly label for an unconstrained agent prompt. It must have a declarative contract.

### Team contract

Each team should define:

- The problems and intents it accepts.
- Inputs, outputs, and artifacts using typed schemas.
- Example requests and explicit refusal conditions.
- The skills, MCP tools, agents, and APIs it may use.
- Data classifications and access requirements.
- Approval rules for sensitive or irreversible actions.
- Expected latency, cost, availability, and fallback behavior.
- Owner, support path, version, and deprecation policy.
- Evaluation suite, quality threshold, and release evidence.

This contract converts a loose set of tools into a managed organizational capability.

## Conceptual architecture

```text
Human or host agent
        |
        | MCP: discovery, planning, invocation, status
        v
Bosworth MCP server / control plane
        |
        +-- Team catalog and semantic index
        +-- Policy and authorization decision point
        +-- Team manifest and version registry
        +-- Planner/router
        +-- Evaluation, audit, and telemetry service
        |
        +---- Team: Eligibility Operations
        |       +-- Skills: investigate-denial, payer-policy-research
        |       +-- MCP: CRM, ticketing, document retrieval
        |       +-- APIs: eligibility gateway, claims data
        |       +-- Guardrails: PHI scope, approval threshold, redaction
        |
        +---- Team: Analytics Engineering
        |       +-- Skills: dbt incident triage, metric definition review
        |       +-- MCP: GitHub, warehouse, observability
        |       +-- APIs: CI/CD, catalog
        |
        +---- Team: Research Operations
                +-- Skills: evidence synthesis, source assessment
                +-- MCP: search, document corpus
                +-- APIs: citation store
```

Bosworth should present a thin, stable MCP façade to client applications. It should not expose every transitive upstream tool in a client’s default context. Clients should first discover a team and its declared contract; implementation detail should be progressively disclosed only when it is relevant or explicitly requested.

### Recommended MCP surface

| Tool | Purpose |
|---|---|
| `search_teams` | Finds candidate teams using an outcome-oriented query, role, domain, data classification, and permission context. |
| `get_team_profile` | Returns scope, examples, owners, interfaces, required approvals, cost and latency expectations, and version. |
| `plan_engagement` | Returns a proposed plan: recommended team(s), dependencies, intended actions, risk, and approvals. |
| `run_team` | Executes a bounded workflow against a validated structured input contract. |
| `get_run_status` | Returns progress, pending approvals, artifacts, and trace identifier. |
| `get_run_artifacts` | Returns structured results, evidence, decisions, and provenance. |
| `report_feedback` | Captures success, correction, failure, escalation, and evaluation labels. |

Where a client genuinely needs standard raw MCP enumeration, Bosworth can optionally expose a team as a virtual MCP server. That should be an advanced integration path rather than the default interaction model.

## Team manifest illustration

```yaml
team_id: healthcare-eligibility-ops
version: 1.2.0
display_name: Healthcare Eligibility Operations
owner:
  group: revenue-cycle-automation
problem_statement: >
  Investigates coverage and eligibility discrepancies, produces
  evidence-backed explanations, and drafts next actions.
intents:
  - eligibility denial analysis
  - coverage verification
  - payer policy lookup
inputs:
  schema_ref: schemas/case-intake.json
outputs:
  schema_ref: schemas/eligibility-resolution.json
capabilities:
  skills:
    - investigate-eligibility-denial@2.1.0
    - payer-policy-research@1.4.0
  tools:
    - member_lookup
    - eligibility_check
    - retrieve_case_documents
  agents:
    - policy-research-agent
policies:
  data_classification: phi
  allowed_roles:
    - revenue_cycle_analyst
  human_approval:
    required_for:
      - external_submission
      - case_status_update
quality:
  evaluation_suite: evals/eligibility-regression.yaml
  minimum_pass_rate: 0.93
```

## Prior art

### Model Context Protocol

MCP supplies the baseline interoperability layer: a host/client can discover and invoke server-provided tools, resources, and prompts through a standard protocol. Its specification also discusses Skills over MCP, making the protocol direction compatible with richer workflow-like capabilities rather than only isolated functions.

**Implication for Bosworth:** use MCP as the external interaction contract, not as the complete product model. Bosworth provides organizational semantics above MCP primitives.

### Agent Skills

Agent Skills packages reusable capability as a directory containing a `SKILL.md` file, metadata, instructions, and optional scripts, assets, and references. The format’s progressive-disclosure model is particularly relevant: concise metadata supports discovery; detailed instructions and resources load only after the capability becomes relevant.

**Implication for Bosworth:** a team should reference versioned skills rather than duplicate their procedural content in a gateway. Skills remain portable authoring and execution assets; Bosworth supplies catalog, policy, composition, lifecycle, and evaluation.

### MCP gateways and federation

IBM ContextForge provides open-source registry and proxy capabilities that federate MCP, A2A, REST, and gRPC services. It supports virtual MCP servers, authentication, rate limiting, and OpenTelemetry-oriented observability. Envoy AI Gateway provides MCP backend routing, multiplexing, and policy-mediated filtering.

**Implication for Bosworth:** do not recreate generic protocol proxying as a first principle. Adopt or build on a gateway for upstream federation and invest engineering effort in the team-level control plane.

### Agent-to-agent delegation

A2A-capable gateways establish that specialist agents can coexist with direct tool calls.

**Implication for Bosworth:** a team may internally use an agent, but must expose a bounded contract, typed outputs, policy bindings, and observable state. Teams should not become opaque delegation boxes.

## Feasibility assessment

| Dimension | Assessment | Rationale |
|---|---|---|
| Technical feasibility | High | MCP provides standardized discovery and invocation; Agent Skills provides portable procedural packages; gateway software can aggregate MCP/API/A2A backends. |
| MVP feasibility | High | A useful pilot needs cataloging, search, profile inspection, deterministic workflow execution, and tracing—not open-ended multi-agent autonomy. |
| Integration complexity | Medium | Identity propagation, MCP-to-MCP forwarding, API adapters, asynchronous work, retries, and transport differences require deliberate adapter design. |
| Semantic discovery | Medium | Mapping an intent to a team is tractable, but needs explainable ranking, an evaluation corpus, and user override; embedding search alone is insufficient. |
| Security and governance | Medium–high risk | The gateway/control plane is a high-value trust boundary, particularly for sensitive data and side-effecting systems. |
| Differentiation | Medium if proxy-only; high if control-plane-first | Generic aggregation exists. Outcome-oriented contracts, governance, evaluation, and organization-wide lifecycle management are the differentiating layer. |
| Organizational adoption | Medium | Team owners need a low-friction contribution path and clear incentives to maintain manifests, skills, tests, and versions. |
| Enterprise value | Promising | Best fit is an organization with tool sprawl, fragmented expertise, sensitive systems, and a need for auditability and reuse. |

### Overall assessment

The project is credible enough to begin a controlled proof of concept. It should be evaluated as a product and operating-model experiment, not only as a protocol integration exercise.

## Security and governance

Bosworth centralizes useful control, but it also centralizes trust risk. Tool metadata and outputs should be treated as untrusted unless they originate from a reviewed, trusted source. A compromised or malicious upstream server can attempt indirect prompt injection, tool poisoning, credential misuse, or unsafe steering of downstream agents.

### Minimum safeguards

- Allowlist upstream MCP servers and APIs; do not permit arbitrary end-user server registration in a production tenant.
- Maintain reviewed, versioned manifests for teams and all transitive dependencies.
- Pin versions and detect changes to tool metadata, schemas, and instructions.
- Enforce authorization at the gateway and backend, never solely through agent instructions.
- Use scoped, short-lived, per-capability credentials; avoid broad shared service tokens.
- Separate discovery/research contexts from systems with PHI, production data, financial authority, or external write capability.
- Apply explicit human approval for irreversible, externally visible, sensitive, or high-impact actions.
- Validate structured inputs and outputs; do not treat prose as a sufficient execution contract.
- Record a complete audit trail: requester identity, policy decision, team and skill versions, tool calls, inputs, outputs, approvals, and trace identifier.
- Include declared capabilities and transitive permissions in team review and release gates.

### Identity model

Access must be distinct for:

1. Discovering that a team exists.
2. Viewing its detailed profile and dependencies.
3. Executing a read-only workflow.
4. Performing side-effecting actions.
5. Approving high-impact actions.

Bosworth must propagate the initiating user or workload identity and its scopes through every downstream call. The platform must not become a confused-deputy path that allows a user to exercise the broader permissions of a team service account.

## Product and implementation principles

### Start deterministic

Do not begin with an autonomous planner that freely chains teams. Start with explicit, versioned workflow graphs; declared dependencies; bounded retries; typed handoffs; and human approval nodes. Add adaptive planning only after observed data demonstrates that the contracts, evaluations, and guardrails are reliable.

### Prefer progressive disclosure

The client should receive a compact team card first: purpose, key examples, authorization fit, expected output, owner, and risk. It should retrieve instructions, schemas, detailed tool descriptions, and references only after selecting a team or planning an engagement.

### Treat evaluation as a release artifact

Every team should own a small regression suite. A team release should be blocked if it does not meet its declared quality threshold or produces unexpected policy outcomes. For side-effecting workflows, evaluation must include approval-path behavior, not merely answer quality.

### Make routing explainable and reversible

A recommendation should say why it was made: intent match, domain fit, data classification, requester authorization, quality score, cost, and expected latency. Users and host agents should be able to select another eligible team or request escalation.

## MVP recommendation

### Pilot scope

Build a narrow pilot around one operational domain with clear inputs, substantial value, known tool fragmentation, and measurable outcomes. Two appropriate initial candidates are:

- Healthcare eligibility investigation and payer-policy research: high value and strong relevance to governed, evidence-backed workflow execution, but requires careful PHI controls.
- Analytics-engineering incident triage: lower-risk access profile and a good proving ground for GitHub, warehouse, dbt, CI/CD, and observability integrations.

Start with three to five teams in one domain, not a universal catalog.

### MVP capabilities

1. Git-backed team manifests with owner, version, input/output schemas, policies, and evaluation references.
2. Agent Skills referenced as portable workflow assets.
3. Two or three reviewed MCP/API integrations per team.
4. `search_teams`, `get_team_profile`, `plan_engagement`, `run_team`, and `get_run_status`.
5. Deterministic execution plans and bounded workflow states.
6. OpenTelemetry-compatible traces and append-only audit events from the beginning.
7. Human approval gates for all material side effects.
8. An evaluation corpus based on realistic tasks, including intentional out-of-scope and unsafe requests.

### Example experience

A user asks: “Why was this member’s eligibility rejected, and what should our next action be?”

Bosworth should:

1. Recommend `Eligibility Operations`, explaining that it matches coverage-verification and payer-rule analysis and is authorized for the relevant data class.
2. Reveal the planned actions: read the case record and eligibility response, consult the approved policy corpus, and draft—but not submit—a recommended next action.
3. Request any required missing fields or approvals.
4. Run the declared workflow.
5. Return a structured result containing the finding, confidence, evidence, missing information, recommended action, approval requirement, and complete provenance.

This is the intended advantage over raw tool discovery: users select a governed outcome capability rather than manually assembling an implementation sequence.

## Success measures

| Metric | Why it matters |
|---|---|
| Top-1 and top-3 team-selection accuracy | Tests whether discovery identifies the appropriate responsible team. |
| Team completion rate | Measures production of an acceptable structured output for in-scope work. |
| Human override rate | Indicates whether discovery and execution plans earn trust. |
| Escalation and refusal quality | Tests whether scope boundaries are honest and safe. |
| Policy denial quality | Measures whether authorization blocks inappropriate work without blocking legitimate workflows. |
| Median and p95 completion time | Quantifies the latency cost of orchestration and indirection. |
| Cost per resolved case | Tests whether composition is economically preferable to direct use of a specialist capability. |
| Trace completeness | Confirms whether consequential runs can be reconstructed end-to-end. |
| Evaluation regression rate | Identifies whether new versions degrade task, safety, or policy performance. |

## Proposed 6–10 week plan

| Phase | Duration | Deliverables |
|---|---:|---|
| 1. Scope and contracts | Weeks 1–2 | Pilot-domain selection; team schema; policy taxonomy; initial task corpus; threat model; three to five team manifests. |
| 2. Control-plane skeleton | Weeks 2–4 | Catalog and registry; MCP discovery surface; identity propagation; gateway integration; audit event schema; basic UI or CLI. |
| 3. Workflow integration | Weeks 4–6 | Versioned skills; vetted upstream MCP/API adapters; deterministic execution engine; structured results; approval gates. |
| 4. Evaluation and hardening | Weeks 6–8 | Regression suite; routing evaluation; security tests; load/latency baseline; observability dashboards; operator feedback loop. |
| 5. Pilot decision | Weeks 8–10 | Measured results; risk review; cost model; adoption feedback; decision to extend, narrow, or stop. |

## Build versus adopt

Adopt or build on an established gateway for:

- MCP transport and upstream federation.
- REST/gRPC/API adaptation.
- Connection management and basic routing.
- Authentication integration, rate limiting, and baseline observability.

Build Bosworth’s differentiated layer:

- Team contract and manifest schema.
- Semantic discovery, ranking, and explanation.
- Team catalog and lifecycle management.
- Policy bindings at team and capability granularity.
- Execution planning and approval-aware orchestration.
- Evaluation registry, release gates, and feedback loop.
- Outcome-level telemetry, provenance, and operational reporting.

## Go/no-go criteria

Advance beyond the pilot only if Bosworth demonstrates both of the following:

1. Users or host agents discover and select the right responsible team more reliably than they can discover and compose raw skills and tools.
2. Team-level packaging produces measurable gains in governance, repeatability, provenance, and safe reuse without unacceptable latency, cost, or maintenance burden.

If the system only aggregates endpoints, it duplicates a growing MCP gateway category. If it gives organizations a reliable way to define, publish, discover, govern, evaluate, and improve outcome-oriented AI teams, it addresses a durable coordination layer in enterprise AI.

## Sources

- [Model Context Protocol Specification](https://modelcontextprotocol.io/specification/draft)
- [MCP Tools Specification](https://modelcontextprotocol.io/specification/draft/server/tools)
- [Agent Skills Specification](https://agentskills.io/specification)
- [Anthropic Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [IBM ContextForge](https://ibm.github.io/mcp-context-forge/)
- [Envoy AI Gateway: MCP Gateway](https://aigateway.envoyproxy.io/docs/next/capabilities/mcp/)
- [OWASP: MCP Tool Poisoning](https://owasp.org/www-community/attacks/MCP_Tool_Poisoning)
- [OWASP MCP Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html)
