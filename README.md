# Agent Rooms

> An open control plane for human–agent collaboration in the team rooms you already use.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Status: Design Draft](https://img.shields.io/badge/status-design%20draft-orange.svg)](#project-status)

Agent Rooms connects existing communication spaces—Telegram groups, Slack channels, Microsoft Teams, Discord, email, and the web—to governed AI agents, shared knowledge, background work, approvals, artifacts, and audit trails.

It is **not another chat app**, **not an agent org-chart simulator**, and **not a replacement for agent runtimes**. It is the organizational layer that remains stable while channels, models, and agent frameworks change.

> **The Room is durable. The agents are replaceable.**

## Project status

Agent Rooms is currently a **public design draft**. This repository defines the problem, principles, proposed domain model, runtime boundary, and first MVP. It does not yet contain a production-ready implementation.

We are looking for collaborators working on:

- organizational AI and team agents;
- identity, authorization, and audit;
- knowledge provenance and correction;
- agent-runtime interoperability;
- Telegram, Slack, Teams, and other communication adapters;
- reliable background execution.

## Why Agent Rooms?

Most agent systems begin with the agent:

- Which agent should the user call?
- How do agents delegate to each other?
- What does the agent org chart look like?
- Which framework owns the workflow?

Organizations begin somewhere else:

- Who is in this room?
- What can they access?
- What work was requested?
- Which sources support the answer?
- Who approved the decision?
- Where is the result?
- What crossed an organizational boundary?

Agent Rooms makes the **organizational room** the primary unit of collaboration, authority, knowledge, and accountability.

Humans select the organizational context. The visible Team Agent selects the technical execution path.

## The basic model

```text
Telegram / Slack / Teams / Web
              │
              ▼
┌─────────────────────────────────┐
│              Room               │
│                                 │
│  People · Visible Team Agent    │
│  Knowledge · Work · Decisions   │
│  Artifacts · Membership         │
│  Permissions · Approvals        │
└────────────────┬────────────────┘
                 │ governed invocation
                 ▼
┌─────────────────────────────────┐
│       Background Runtimes       │
│                                 │
│  Research · Data · Code         │
│  Documents · Browser · Search   │
└─────────────────────────────────┘
```

The same Room may eventually have multiple communication surfaces. A Telegram group is therefore a **Channel Binding**, not the Room itself.

```text
Organization
└── Room
    ├── Memberships and grants
    ├── Knowledge scope
    ├── Work and decisions
    ├── Approvals and artifacts
    ├── Visible Team Agent
    └── Channel Bindings
        ├── Telegram group
        ├── Slack channel
        └── Web room
```

## Design principles

### 1. Rooms before agents

The Room is the durable organizational context. Agent identities and runtimes can be added, replaced, or removed without migrating the organization's knowledge, authority, and work history.

### 2. One coherent human interface

A Room starts with one visible Team Agent. That agent may invoke specialist workers, but humans should not have to select among a collection of technical agents.

### 3. Existing channels first

Teams should be able to work from the communication tools they already use. A new dashboard may help administrators, but it should not become mandatory for ordinary collaboration.

### 4. The Board is a ledger, not the workplace

Conversation remains the primary collaboration surface. A durable work ledger records ownership, status, deadlines, approvals, artifacts, retries, recovery, and cost.

### 5. Knowledge is promoted, not harvested

A conversation is not automatically organizational knowledge.

```text
Conversation
→ Knowledge or decision candidate
→ Source and conflict review
→ Authorized approval
→ Versioned organizational knowledge
```

### 6. Policy is deterministic

Identity, permissions, budgets, invocation eligibility, and approval requirements must be enforced by software. They must not depend on an LLM deciding to follow a prompt.

### 7. Every boundary crossing leaves a receipt

Cross-room transfers, external sends, restricted retrieval, and consequential actions must be explicit, authorized, and auditable.

### 8. Agent runtimes are interchangeable

Agent Rooms standardizes the organizational envelope and observable execution lifecycle—not the internal reasoning architecture of every agent.

## Core domain objects

| Object | Purpose |
|---|---|
| `Organization` | Administrative and policy boundary |
| `Person` | Stable human identity independent of channel accounts |
| `AgentIdentity` | Stable identity of a visible or background agent |
| `Room` | Organizational context for membership, policy, knowledge, and work |
| `ChannelBinding` | Maps a Telegram group, Slack channel, or other surface to a Room |
| `Membership` | Connects a person or agent to a Room with roles and grants |
| `RoomAgent` | Declares the visible Team Agent and permitted worker relationships |
| `KnowledgeRecord` | Versioned, sourced, classified organizational knowledge |
| `KnowledgeProposal` | Proposed addition, correction, or supersession awaiting review |
| `WorkItem` | Durable unit of requested work and its execution state |
| `Decision` | Authorized conclusion with scope, evidence, and decision maker |
| `Artifact` | Registered output such as a document, dataset, report, or code change |
| `Approval` | Human authorization for a governed transition or action |
| `TransferReceipt` | Record of an authorized cross-room or external handoff |
| `AuditEvent` | Append-only record of significant actions and state transitions |

A personal agent profile or an agent organizational chart is intentionally **not** a foundational object.

## Example Room configuration

```yaml
room:
  id: npi-investment
  organization_id: npi
  display_name: NPI Investment

  bindings:
    - id: telegram-npi-investment
      type: telegram
      external_id: "-100123456789"
      invocation_policy:
        agent_mention: true
        reply_to_agent: true
        human_mentions: ignore
        unsolicited_response: critical_only

  memberships:
    - subject: person:jehyun-park
      roles: [partner]
    - subject: person:jaehun-wi
      roles: [member]
    - subject: person:hohyon-ryu
      roles: [member]

  visible_agent:
    id: agent:npi-investment
    runtime: hermes:npi-investment

  workers:
    - agent:research
    - agent:financial-analysis
    - agent:document

  knowledge_scopes:
    - npi-common
    - npi-investment
    - assigned-portfolio

  approvals:
    external_send: human
    investment_decision: role:partner
    payment: role:authorized-person
```

This format is illustrative. The specification will define a validated schema before implementation.

## Conversation behavior

### Human-only conversation

```text
@Jae-hyun Please check Company A's latest materials.
```

The agent remains silent because the message addresses another human.

### Agent invocation

```text
@NPIAgent Compare Company A's previous investment review with the new revenue data.
```

The visible Team Agent immediately acknowledges the request:

```text
Review started.

Owner: NPI Investment Agent
Support: Financial Analysis Worker
Sources: Prior review · Current revenue data
Estimated completion: 20 minutes
Result destination: This room
External sharing: None
```

The runtime may invoke workers without exposing that technical routing decision to the requester.

### Result

```text
Company A review completed.

Changes
1. Revenue growth: 18% → 9%
2. Customer concentration increased
3. Cash runway: 14 months → 8 months

Recommendation
Conduct an additional founder interview before deciding whether to proceed.

Decision requested
Should we schedule the interview?

Sources
- 2026-05 investment review memo
- 2026-07 revenue report
```

## Invocation policy

Message eligibility must be determined **before** an LLM is invoked.

A channel adapter extracts platform-specific facts:

```yaml
message_context:
  sender: telegram-user:12345
  room_binding: telegram-npi-investment
  mentions:
    - person:jehyun-park
  replies_to: null
  text: "Please review this."
```

The kernel resolves stable identities and applies Room policy:

```yaml
eligibility:
  invoke_agent: false
  reason: human_only_mention
```

This prevents unnecessary model calls and unwanted intervention in human conversation.

## Authenticated invocation envelope

Every runtime invocation should carry authenticated organizational context:

```yaml
invocation:
  id: inv_01J...
  idempotency_key: telegram:-100123456789:message:456

  actor:
    person_id: person:jehyun-park
    membership_id: membership:npi-investment:jehyun-park

  context:
    organization_id: npi
    room_id: npi-investment
    channel_binding_id: telegram-npi-investment

  agent:
    visible_agent_id: agent:npi-investment
    runtime_id: hermes:npi-investment

  grants:
    - knowledge.read:npi-common
    - knowledge.read:npi-investment
    - work.create:npi-investment

  request:
    text: "Compare the previous review with the current revenue data."

  delivery:
    destination: originating-room
    external_send: prohibited
```

Identity and grants must come from trusted system state—not prompt text supplied by a person or an agent.

## Knowledge lifecycle

A `KnowledgeRecord` has an explicit lifecycle:

```text
proposed → approved → superseded
         ↘ rejected
approved → expired
approved → deleted
```

A record should include:

- source and source location;
- author or proposing agent;
- approver;
- version and effective date;
- security classification;
- Room and organizational scope;
- status and supersession relationship;
- correction history;
- index synchronization state.

### Corrections

A correction does not silently overwrite history. It creates a `KnowledgeProposal` that:

1. identifies the challenged record;
2. preserves both old and proposed versions;
3. provides evidence;
4. names the required approver;
5. supersedes the old record only after approval;
6. invalidates or rebuilds derived search indexes.

## Work lifecycle

A background task becomes a durable `WorkItem`:

```text
proposed
→ accepted
→ queued
→ running
→ awaiting_approval
→ completed
```

Alternative terminal states:

```text
cancelled · failed · expired · rejected
```

A WorkItem distinguishes:

- requester;
- accountable owner;
- executing agent runtime;
- approving person or role;
- originating Room;
- result destination;
- visible and restricted artifacts.

Retries must be idempotent. Worker crashes and gateway restarts must not lose accepted work or create duplicate results.

## Cross-Room transfers

A Room Agent may request work from another Room only through an authorized transfer.

```text
NPI Investment Room
        │
        │ TransferReceipt
        ▼
NPI Legal & Operations Room
```

A receipt records:

- requester and sending Room;
- destination Room;
- disclosed text, context, and artifacts;
- authorization decision;
- destination acceptance;
- resulting WorkItem;
- delivery and failure state;
- replay protection.

Both Rooms receive a human-readable receipt. No private agent should secretly move organizational information between Rooms.

## Agent-runtime interface

A runtime adapter exposes capabilities and an observable lifecycle:

```python
from typing import Protocol

class AgentRuntime(Protocol):
    def capabilities(self) -> "AgentCapabilities": ...
    def execute(self, request: "InvocationEnvelope") -> "RunHandle": ...
    def status(self, run_id: str) -> "RunStatus": ...
    def cancel(self, run_id: str) -> "CancelResult": ...
    def collect_artifacts(self, run_id: str) -> list["Artifact"]: ...
    def usage(self, run_id: str) -> "UsageRecord": ...
```

Possible adapters include:

- Hermes Agent;
- Claude Agent SDK;
- OpenAI Agents SDK;
- LangGraph;
- Google ADK;
- custom HTTP agents;
- deterministic scripts and workers.

The adapter executes. The kernel authorizes and records.

## Kernel and adapters

### Kernel responsibilities

- stable identities and memberships;
- Room and Channel Binding state;
- grants and policy decisions;
- invocation eligibility;
- Knowledge lifecycle and provenance;
- WorkItem state machine;
- approvals, decisions, and artifacts;
- transfer protocol;
- idempotency and recovery;
- append-only audit events;
- transactional event/outbox delivery.

### Adapter responsibilities

- parsing and publishing platform messages;
- translating mentions, replies, files, and threads;
- invoking, cancelling, and inspecting agent runtimes;
- integrating search and storage backends;
- integrating identity providers and secret managers;
- reporting usage and collecting artifacts.

Adapters must not become alternate policy engines.

## Security model

Agent Rooms assumes that agent output can be incorrect and external content can be hostile.

Key requirements:

- deterministic authorization before retrieval and action;
- item-level access control for restricted knowledge;
- least-privilege grants for workers;
- no privilege expansion through delegation;
- explicit approval for consequential actions;
- content and metadata separation in audit storage;
- configurable retention, redaction, export, and deletion;
- secrets referenced by handles rather than embedded in prompts;
- replay protection and idempotency for channel updates;
- provenance preserved across retrieval, summarization, and handoff.

“Full audit” does not mean every person can read every payload. Audit access is itself governed.

For private vulnerability reports, see [SECURITY.md](SECURITY.md).

## Minimal administration surface

The long-term admin experience can remain small:

### Rooms

- connected channels;
- people and memberships;
- visible Team Agent;
- knowledge scope;
- invocation policy.

### Knowledge

- approved organizational knowledge;
- sources and versions;
- security classifications;
- corrections and pending approvals.

### Work

- active and blocked WorkItems;
- human and agent ownership;
- deadlines, retries, and results;
- registered artifacts.

### Admin

- roles and grants;
- runtime health;
- usage and cost;
- audit and retention controls.

## NPI Knowledge Room MVP

The first deployment should contain exactly one Room:

```text
NPI Knowledge Room
├── NPI employees
├── NPI Knowledge Agent
├── Existing NPI Knowledge Broker
├── Company documents, policies, and prior decisions
└── Questions, corrections, and knowledge proposals
```

### First capabilities

1. Connect one existing Telegram group.
2. Respond only to `@NPIAgent` or replies to the agent.
3. Remain silent for messages that mention only humans.
4. Search approved shared knowledge.
5. Include sources in every factual answer.
6. Filter retrieval by membership and classification.
7. Register corrections as proposals.
8. Run long tasks as durable background WorkItems.
9. Return results to the originating Room.
10. Audit every accepted request, state transition, and result.

### Explicitly deferred

- general-purpose Board UI;
- multiple visible agents in one Room;
- personal agents;
- arbitrary cross-Room routing;
- autonomous agent negotiation;
- visual agent org charts;
- generic workflow builders;
- organization-wide semantic search;
- broad runtime support before the contract is proven.

## MVP acceptance tests

The MVP is not complete until it handles these cases:

1. An unauthorized person requests restricted material.
2. An answer would combine authorized and unauthorized sources.
3. Telegram delivers the same update twice.
4. A worker crashes during execution.
5. The gateway restarts before publishing a result.
6. A person mentions another person but not the agent.
7. A correction conflicts with approved knowledge.
8. Membership is revoked while background work is running.
9. An agent attempts an external send without approval.
10. A result is too large for the channel and must become an Artifact.

## Success measures

For the first four weeks, measure:

- weekly active users;
- questions per employee;
- reduction in repeated questions;
- percentage of factual answers with sources;
- incorrect-answer and correction rates;
- approved Knowledge proposals;
- median and tail response time;
- unnecessary agent interventions;
- usage relative to personal agents;
- unauthorized retrieval attempts correctly denied;
- duplicate messages and duplicate WorkItems prevented.

## Non-goals

Agent Rooms is not intended to:

- define how an agent reasons internally;
- require a particular model or framework;
- expose every specialist agent to humans;
- ingest every conversation into permanent memory;
- replace Telegram, Slack, Teams, or email;
- let LLM prompts substitute for authorization;
- score employees through agent telemetry;
- make consequential decisions without accountable human approval.

## Proposed repository structure

```text
agent-rooms/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── SECURITY.md
├── specs/
│   ├── room.schema.json
│   ├── invocation.schema.json
│   ├── work-item.schema.json
│   └── events.md
├── kernel/
├── adapters/
│   ├── channels/
│   │   └── telegram/
│   └── runtimes/
│       ├── hermes/
│       └── http/
├── examples/
│   └── npi-knowledge-room/
└── tests/
    ├── authorization/
    ├── invocation/
    └── recovery/
```

This is a target structure, not a claim about the current repository contents.

## Roadmap

### Phase 0 — Specification

- settle vocabulary and invariants;
- define JSON Schemas;
- define invocation and runtime contracts;
- model Knowledge and WorkItem state machines;
- publish threat model and acceptance scenarios.

### Phase 1 — Single Room reference implementation

- PostgreSQL-backed kernel;
- Telegram adapter;
- Hermes runtime adapter;
- mention/reply invocation policy;
- approved Knowledge retrieval with citations;
- durable background WorkItems;
- append-only audit events.

### Phase 2 — Interoperability proof

- generic HTTP runtime adapter;
- second channel adapter;
- capability negotiation;
- conformance test suite;
- export and migration tools.

### Phase 3 — Governed organizational workflows

- cross-Room transfer receipts;
- richer approval policies;
- enterprise identity integrations;
- retention and audit administration;
- deployment and isolation reference patterns.

## Open-source strategy

Agent Rooms should remain useful without a hosted service. The protocol, schemas, kernel, reference adapters, and conformance tests should be open.

A sustainable ecosystem may grow around:

- managed hosting;
- enterprise identity and compliance integrations;
- premium operational tooling;
- supported channel and runtime adapters;
- migration and deployment services;
- governance templates and evaluation suites.

The moat is not hidden-agent orchestration. It is trustworthy organizational state and compatibility across changing agent runtimes.

## Questions we want help answering

- Which invariants must every Room implementation preserve?
- How small can the runtime contract remain without becoming a lowest-common-denominator trap?
- Should Knowledge authorization be ACL-, policy-, or capability-based?
- Which audit events need payload retention, and which need metadata only?
- How should active work react to membership or grant revocation?
- What delivery guarantees should channel adapters provide?
- How should organizations export all Room state and move between implementations?
- When does a specialist agent deserve a visible social identity?

## Contributing

This project is at the stage where careful disagreement is especially valuable.

Good first contributions include:

- concrete use cases and failure scenarios;
- domain-model critiques;
- authorization and privacy threat analysis;
- runtime-interface proposals;
- channel behavior edge cases;
- JSON Schema drafts;
- conformance and recovery tests.

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

## License

Licensed under the [Apache License 2.0](LICENSE).

## One-sentence design

> Agent Rooms turns existing team chats into governed workplaces where people collaborate with one visible Team Agent backed by shared knowledge, durable work, approvals, artifacts, and interchangeable agent runtimes.
