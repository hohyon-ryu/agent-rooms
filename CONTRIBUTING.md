# Contributing to Agent Rooms

Agent Rooms is currently a design-stage open-source project. Contributions that sharpen the model, expose failure cases, or reduce implementation ambiguity are welcome.

## Before contributing

1. Read the README and current design principles.
2. Search existing issues before opening a new one.
3. Describe the organizational scenario, not only the technical feature.
4. State the security and privacy implications.
5. Prefer the smallest change that preserves runtime and channel neutrality.

## Useful contribution types

- domain model critiques;
- threat models and authorization scenarios;
- agent-runtime contract proposals;
- Telegram, Slack, Teams, and other channel edge cases;
- Knowledge provenance and correction semantics;
- WorkItem reliability and recovery scenarios;
- JSON Schema drafts;
- executable conformance tests;
- reference adapters.

## Design rules

- Rooms are organizational contexts, not aliases for chat IDs.
- Authorization is deterministic and occurs outside LLM prompts.
- Delegation cannot expand permissions.
- Conversations do not automatically become approved Knowledge.
- Cross-boundary transfers leave receipts.
- Runtimes are replaceable and must not own organizational state.
- Avoid adding UI or orchestration breadth before the core contract is proven.

## Pull requests

A good pull request includes:

- the problem and affected organizational scenario;
- the proposed invariant or behavior;
- security and migration considerations;
- tests or concrete acceptance examples;
- explicit non-goals.

For substantial changes, begin with an issue or short design note.

## Conduct

Be direct, kind, and specific. Critique ideas and assumptions rather than people. Organizational AI touches power, privacy, and accountability; disagreement should be documented rather than hidden.
