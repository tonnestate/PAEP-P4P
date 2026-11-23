# PAEP / P4P — Open-Source Donor Assessment

**Review date:** 2026-11-23. Initial donor shortlist based on repository metadata, licenses and stated features; **not** a complete source, API, vulnerability or license-compliance audit. No donor code has been imported into PAEP/P4P.

| Donor | Repository | Reported license | Intended use | Boundary |
|---|---|---|---|---|
| Agent Network Protocol | https://github.com/agent-network-protocol/anp | Apache-2.0 | Discovery, agent identities, encrypted messaging design reference | Review TypeScript SDK maturity and wire compatibility before integrating |
| CLI-Anything | https://github.com/HKUDS/CLI-Anything | Apache-2.0 | Agent-facing CLI conventions, structured output and tool discoverability | Primarily Python-harness generation; do not import whole runtime |
| Agent Skills | https://github.com/agentskills/agentskills | Apache-2.0 | Shared `SKILL.md` format for approved clients | Instructions are not permissions or a communication transport |
| MCP TypeScript SDK | https://github.com/modelcontextprotocol/typescript-sdk | Mixed/transitioning MIT and Apache-2.0 | Optional MCP adapter for deployments that permit MCP | No dependency in core; inspect per-file and package license |
| A2A JavaScript SDK | https://github.com/a2aproject/a2a-js | Apache-2.0 | Optional agent-card / A2A interoperability | Preserve PAEP authorization and security semantics |
| hpke-js | https://github.com/dajiaji/hpke-js | MIT | Standardized hybrid public-key encryption candidate | Test suites, sender authentication, key binding and envelope design required |
| jose | https://github.com/panva/jose | MIT | JWK, JWS, JWE, identity-token and signed-envelope primitives | JWT acceptance and algorithm policy must be fail-closed |
| Fastify | https://github.com/fastify/fastify | MIT | Lightweight TypeScript-capable HTTPS/JSON API | Framework does not supply protocol authorization automatically |
| TypeBox | https://github.com/sinclairzx81/typebox | MIT (verify at adoption) | Contract-first schema validation and TypeScript types | Validate payload limits, strict unknown-field rejection |
| DuckDB Node API | https://github.com/duckdb/duckdb-node-neo | MIT | Small persistent endpoint, room, lease, outbox and ciphertext state | Validate transaction and concurrent-write behavior under intended load |
| Commander.js | https://github.com/tj/commander.js | MIT (verify at adoption) | Direct MCP-free TypeScript CLI | Every operation must call the same server authority contract |
| Cloudflare Agents | https://github.com/cloudflare/agents | MIT | Optional Cloudflare deployment/agent integration reference | No mandatory Cloudflare platform dependency |
| n8n | https://github.com/n8n-io/n8n | Sustainable Use License / special file restrictions | Replaceable ciphertext delivery/orchestration adapter | **Not MIT/Apache 2.0**; check commercial restrictions before embedding or redistributing |

## Recommended implementation order

1. **Core transport and contracts:** TypeScript service with Fastify and strict JSON validation; a minimal DuckDB state store. No framework must bypass the canonical permission evaluator.
2. **Cryptography:** Evaluate hpke-js and jose under one independently reviewed suite, with proof-of-possession, signature/recipient-binding and negative tests. Do not invent cryptographic constructions.
3. **Discovery:** Native well-known JSON and public guide, CLI capability output, and direct invitations; publish optional compatible agent skill documentation.
4. **Interoperability:** ANP and A2A as prior-art references and potential adapters; MCP SDK only where explicitly permitted.
5. **Relay:** Implement a simple secure local delivery route first; use n8n as an optional interchangeable ciphertext relay after license and safety review.

**Architectural decision:** Donors are implementation candidates, not product dependencies or claims of unique novelty. PAEP/P4P remains an independent, JSON-centered protocol for two endpoints, four possible roles, and lease-bound communications. **No blockchain or hash-chained ledger in the MVP.**
