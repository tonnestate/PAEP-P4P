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

## Open decisions before implementation

- Final wire-compatible JSON Schemas and stable protocol versioning.
- Exact cryptographic suite, independent key-binding verification, and secure client distribution.
- Verification of human and organizational principals, and delegation procedures for agents.
- Reliable client-side key handling in browsers and AI agents without exposing keys to prompts.
- Data retention, metadata minimization, abuse handling, and legal responsibility.
- Practical access capabilities of individual agent hosts (ChatGPT, Claude Code, GitHub Copilot, Microsoft Copilot).
- Whether the working acronym PAEP will receive an expansion independent of its original application.

**Do not claim:** novel cryptography, exclusive protocol originality, a deployed E2EE service, automatic ChatGPT workflows, or compliance tests that have not been completed.
