# Architecture Decisions — PAEP / P4P

**Date:** 2026-11-23. These decisions move the concept **into an independent product**. The original PumpkinMe-specific PAEP design remains historical application context, not a technical dependency.

| Area | Decision |
|---|---|
| Product independence | PAEP/P4P is a standalone repository and product; PumpkinMe is an optional client |
| Protocol | PAEP is a vendor- and transport-independent, JSON-centered exchange contract |
| Communication model | P4P comprises two sides and up to four separately authorized human/agent roles |
| Representation | Dynamic JSON endpoints; websites are optional human interfaces |
| Lifecycle | Two endpoints, mutual authorization to pair, lease, encrypted channel |
| Identity / authorization | Public/private key cryptography as a building block; distinct roles and key ownership; a lease controls operations but is not an AES key |
| Privacy | Client-side E2EE is the target; the relay receives no plaintext; metadata is not automatically anonymous |
| Transport | n8n is a proposed replaceable ciphertext relay, not a key manager |
| Technology | TypeScript, minimal JSON contract, DuckDB for compact persistent state |
| Integrations | Web/HTTPS, CLI/Code Mode, JSON file exchange; skills and MCP are optional; no MCP requirement |
| GitHub | Hosts code and specifications, not an operational message or profile ledger |
| Domain neutrality | No mandatory jobs, profile, or skill model in the generic core |
| Development | Start with a reproducible local prototype; no GitHub Actions/CI |
| Discovery | Multiple independent entry routes: web search, well-known JSON, A2A agent cards, Skills, CLI/Code Mode, direct invitation; one canonical protocol |
| Discovery trust | Discovery is not identity, key verification, authentication, consent, or a lease; no mandatory global directory |
| Messages and relay | Service may send its own attributed messages and forward ciphertext, but cannot read participants' encrypted traffic |
| Encrypted JSON | Originating trusted clients encrypt confidential payloads before transmitting to the TypeScript server |
| Blockchain | Deferred possible extension only; no blockchain, consensus or hash-chained ledger in MVP |
| Donors | Reuse established open-source techniques and adapters with license review, never treat donors as mandatory vendor runtime dependencies |

## Open decisions before implementation

- Final wire-compatible JSON Schemas and stable protocol versioning.
- Exact cryptographic suite, independent key-binding verification, and secure client distribution.
- Verification of human and organizational principals, and delegation procedures for agents.
- Reliable client-side key handling in browsers and AI agents without exposing keys to prompts.
- Data retention, metadata minimization, abuse handling, and legal responsibility.
- Practical access capabilities of individual agent hosts (ChatGPT, Claude Code, GitHub Copilot, Microsoft Copilot).
- Whether the working acronym PAEP will receive an expansion independent of its original application.

**Do not claim:** novel cryptography, exclusive protocol originality, a deployed E2EE service, automatic ChatGPT workflows, or compliance tests that have not been completed.

## Decision boundaries updated on 2026-11-23

The **multi-discovery / single-protocol** decision replaces any earlier reading that a public webpage or MCP adapter is the sole entry point. Discovery must accommodate unknown-domain web search, known-domain discovery, agent-card compatibility, skill instructions, permitted CLI commands and verified direct invitations.

The **P4P message boundary** makes the infrastructure a send-capable, relay-capable but non-reading operator for third-party confidential communications. Its own administrative messages are a distinct authenticated sender type.

**Blockchain and even a hash-chained audit ledger are removed from the MVP.** Conventional minimal delivery/revision/audit metadata remains, with optional verifiable logging or blockchain integration only in a later separately approved phase. The product remains independent of PumpkinMe and BlueberryMe.

The current donor shortlist is a research input, **not an implementation or dependency lock**. See [DONORS.md](DONORS.md).
