# PAEP / P4P — Initial Implementation and Acceptance Criteria

**Status:** Planned prototype. No implementation is claimed.

## 1. Goal of the first working proof

**Two independently launched clients connect two dynamic JSON endpoints without MCP, PumpkinMe, or a third-party LLM API.** A human and a delegated agent on each side can hold different rights. A message is encrypted locally, transported through the relay channel, decrypted only by its authorized recipient, and further delivery under a revoked lease is prevented.

This is deliberately smaller than a complete agent network. Real AI environments will be integrated only afterward.

## 2. Technical direction

- **TypeScript:** Lightweight typed API and shared protocol/client module. No separate Flask or Python service.
- **DuckDB:** Minimal local state for endpoints, key references, permissions, leases, rooms, ciphertext, and delivery cursors. No separate graph database or external account as an operational prerequisite.
- **n8n:** Proposed replaceable transport adapter for ciphertext. A local delivery path must allow the same contract to be tested. Unprotected webhooks may never bypass permission checks.
- **Browser/CLI:** Simple, verifiable clients first. The human interface and agent adapters use the same core.
- **Cryptography:** Use an established, audited implementation; finalize and test the exact algorithms, suite, and key storage before implementation.
- **Operations:** Locally reproducible commands and tests; **no GitHub Actions/CI**. No mandatory paid infrastructure.

## 3. Implementation in verifiable steps

1. **Freeze the contract:** JSON Schema 0.1, error format, versioning, state transitions, public discovery data, private operations, and complete negative schema cases.
2. **Core model:** Endpoint, principal, agent delegation, key binding, room, lease, and atomic revision in DuckDB; distinct resource IDs and permission scopes.
3. **Local end-to-end test:** Two clients create their own keys, verify bindings, encrypt, transport opaque ciphertext, and decrypt only at the authorized recipient.
4. **Message operations:** Deduplicated messages, polling and cursors, acknowledgements, expiration, restart recovery, and a durable outbox.
5. **Relay adapter:** n8n carries only authenticated, authorized ciphertext and metadata. Server-side checks remain mandatory, and the relay never transforms plaintext.
6. **Four-role proof:** Distinct human and agent identities, delegated rights, Human→Agent and Agent→Human replies; messages intended only for humans remain inaccessible to agents.
7. **Multi-discovery and interoperability:** Demonstrate discovery by public web entry, known-domain well-known JSON, CLI/Code Mode, and direct invitation. Describe optional Agent Skills and A2A agent-card adapters using the same canonical metadata. No MCP required; no global registry. Test that discovery never authorizes participation.
8. **External agent trial:** Use only capabilities actually available in an authorized ChatGPT Work, Claude Code, or Copilot environment. Record capabilities **per host** rather than assuming them.

## 4. Mandatory acceptance scenarios

| ID | Test | Expected result |
|---|---|---|
| A01 | Retrieve a private JSON view without authentication | Denied, no private information exposed |
| A02 | Create two independent endpoints | Distinct GUIDs and independent principal/key bindings |
| A03 | Propose a room with approval from only one side | No active communication channel |
| A04 | Both sides approve | Room-scoped, time-limited lease |
| A05 | Client A encrypts for B | Relay/server cannot reconstruct plaintext if the client and key bindings are authentic |
| A06 | An unauthorized role attempts decryption | Cannot access the message key |
| A07 | Agent tries to substitute for human approval | Authorization fails |
| A08 | Replay the same message or revision | Idempotent result, no duplicate delivery |
| A09 | Lease expires or is revoked | No new authorized deliveries; previously obtained copies are not falsely claimed to be erased |
| A10 | Client is offline and later resumes | Message appears once through cursor/outbox |
| A11 | n8n is unavailable | Controlled retry/error; state remains consistent |
| A12 | Attacker substitutes a registered public key | Verification path detects or blocks substitution |
| A13 | Generated JSON state changes | Only revision/delta changes; no unnecessary profile copy |
| A14 | Use API/CLI without MCP | Identical authorization behavior |
| A15 | Discover through web, known-domain JSON, CLI, and invitation | All supported routes resolve compatible PAEP metadata; no secret leaked |
| A16 | Attempt private access using only a discovery reference or invitation | Access denied without authenticated, scoped authorization |
| A17 | Publish a malicious agent card, skill document, or redirected discovery endpoint | No unauthorized operation, key substitution, internal-network fetch or permission escalation |
| A18 | Infrastructure principal creates its own signed administrative notification | Allowed with proper scope, correctly attributed as service-originated |
| A19 | Infrastructure attempts to read or decrypt third-party message ciphertext | Cannot decrypt; no private key or privileged plaintext access |
| A20 | Forward encrypted participant JSON and create a dynamic endpoint view | Routing metadata readable; payload remains opaque and ciphertext round-trips unchanged |
| A21 | Run MVP without blockchain or hash-ledger services | Complete local communication proof without those dependencies |

**Acceptance boundary:** A successful local demo does not establish production security. Browser-client trust and independent key verification must be solved before public E2EE guarantees are made.

## 5. Outside the first milestone

No job board, matching engine, paid enterprise functionality, second database, real-time delivery guarantee, ChatGPT push notification, broad web crawling, automatic sign-up on external platforms, large-scale group encryption, or CI pipeline.

## 6. Observable deliverables

A locally reproducible results log must demonstrate two endpoints, two independently authorized human/agent role pairs, unambiguous room and lease revisions, delivery order, successful decryption only by the intended recipient, and all required negative cases. No example private keys or real personal data belong in the repository.

## 7. Release scope clarification (2026-11-23)

**Multi-discovery, single protocol** is a firm architectural decision. All supported discovery routes point to the same capabilities and access rules; no special privilege is granted to an A2A adapter, skill, CLI or direct invitation. Initial local proof should support at least the web entry, known-domain JSON, CLI, and invitation cases. Additional provider-specific integrations can be added after validating host capabilities.

The infrastructure may generate its own explicitly attributed messages and forward participant ciphertext but may **not** read participant plaintext. Encryption begins at the sender's trusted client, not at the PAEP TypeScript server. The first milestone must run with **no blockchain and no hash-chain ledger**.
