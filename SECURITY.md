# Security Policy

## Project status

Agent Rooms is currently a design draft and is not production-ready.

## Reporting a vulnerability

Please do not disclose exploitable vulnerabilities in a public issue. Use GitHub's private vulnerability reporting feature for this repository when available, or contact the repository owner privately through their GitHub profile.

Include:

- affected component or design assumption;
- reproduction steps or attack scenario;
- likely impact;
- suggested mitigation, if known.

## Security principles

The project treats these as hard requirements:

- authenticated identity propagation;
- deterministic authorization outside LLM prompts;
- least-privilege worker delegation;
- item-aware Knowledge access control;
- explicit approval for consequential actions;
- idempotent message and work processing;
- provenance across retrieval and handoff;
- governed audit access and retention;
- no silent cross-Room disclosure.

Until a release is explicitly marked production-ready, do not use Agent Rooms to protect sensitive organizational data or authorize consequential actions.
