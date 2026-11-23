# PAEP / P4P — Standalone Product Concept

**Version:** Concept v0.1, 2026-11-23  
**Scope:** An independent communication service. PumpkinMe is a potential first application, not an architectural dependency.

## 1. Problem and solution

People and companies use different AI assistants. A shared agent and messaging space must not require a specific AI platform, MCP, or an installed native app.

PAEP defines a **dynamic, structured JSON communication endpoint** and the operations available on it. P4P extends direct communication between participants to authorized representatives: two human or organizational sides and their respective agents. The service **brokers endpoints and permissions**; it does not host the AI agents.

This simplifies the earlier idea of individual websites: **A website is an optional view of an endpoint, not the endpoint itself.** The communication room is established from two endpoint references, approved leases, cryptographically verifiable recipient keys, and a message transport.

## 2. Two sides, four possible roles

| Side A | Side B | Principle |
|---|---|---|
| Human A / Organization A | Human B / Organization B | Human decision-making and accountability |
| Agent A (delegated) | Agent B (delegated) | Limited authority on behalf of each side |

Four roles **do not require four simultaneous network connections**. A human may participate without an agent; agents may exchange messages asynchronously. “Organization” means a verified organizational identity and an identifiable responsible human decision-maker, not an unverified claim of company affiliation.

Example: A personal agent discovers an employment opportunity, a company agent asks about availability, a human replies directly, and the two people decide whether more information may be released. The PAEP core has **no job-specific semantics**: other services can use the same communication rooms for project work, customer conversations, or collaboration.

## 3. Product core

1. **Discovery:** A publicly accessible landing page, discoverable by standard search engines, points to a machine-readable protocol description. Untrusted web content does not grant authorization.
2. **Provisioning:** An eligible request creates an endpoint with a random opaque ID. The JSON state is generated dynamically from minimal persistent records.
3. **Delegation:** Each human or legitimate organizational representative grants an agent explicitly limited read/write rights.
4. **Pairing:** The parties propose a connection between their endpoints and confirm its authorized purpose. Knowledge of a GUID alone never grants room access.
5. **Lease:** The broker checks the room, participants, permissions, duration, revocation, and resource revision.
6. **Key exchange:** Each eligible principal has a bound, verifiable public key. Private decryption keys never leave the trusted client.
7. **Messaging:** An authorized sender encrypts for permitted recipients; the relay delivers ciphertext and minimal delivery information.
8. **Human view:** An optional browser chat supports reading, replying, revoking, and approving permissions through the same JSON contract.

## 4. Deliberately excluded from storage

The generic PAEP core does **not** need a central skill graph, résumé, company registry, full chat plaintext, or proprietary agent context. Such data belongs to the application layer when needed, or remains with the respective participant.

Minimal persistent state consists of endpoint IDs, public keys and their verified bindings, roles and authorizations, lease states, room relationships, message ciphertext with minimal delivery data, sequential change positions, and bounded technical evidence. JSON documents are assembled on request rather than copied in full after every change.

A domain-specific application may reference optional, approved JSON fields. An endpoint ID never makes profile contents public by itself.

## 5. Vendor-independent interfaces

- **Web + JSON:** Indexable entry point, public discovery document, authenticated JSON operations.
- **CLI / Code Mode:** Equivalent operations for authorized Claude Code, GitHub Copilot, and other local agents.
- **File exchange:** Verified import/export format for isolated enterprise environments, with the same signing, revision, and replay protections.
- **Agent skills:** Explanatory, optionally installed instructions. Installation alone conveys no authority.
- **MCP:** Strictly optional adapter where enterprise policy permits it.
- **Human interface:** Optional presentation of the same protocol state, not a second chat backend.

Microsoft Copilot/Copilot Studio and GitHub Copilot are distinct integration targets. Specific capabilities and enterprise policies must be tested for each environment.

## 6. Product positioning

PAEP/P4P is first a **communication and authorization system**, not a job board, general-purpose agent host, or social network. It may later be embedded in other products or offered as an independent service.

Its technical building blocks are not new: A2A, ANP, DIDComm, Matrix, MLS, and established public-key cryptography are important prior art. A possible differentiator is the combined **automatic provisioning of accessible JSON endpoints, four explicitly separated roles, human-controlled leases, and vendor-independent access without MCP**. Novelty, protectability, and market potential have **not** been demonstrated.

## 7. Non-goals of the first release

- No proprietary LLM host or automatic execution of remote agents.
- No custom cryptographic algorithm or unverified security guarantees.
- No requirement to process applicant data in the communication core.
- No mandatory sign-up with any AI provider.
- No assumption that external agents remain online continuously.
- No full enterprise platform, billing, or large-scale group chat in v0.1.
- No copying PumpkinMe modules; integration only through generic interfaces.

## 8. Product success criterion

Two independently launched clients, including at least one agent adapter and one human interface, can provision their own endpoints, demonstrate separate permissions, create a room with mutual authorization, exchange authenticated end-to-end encrypted messages, retrieve messages after an offline interval, and prevent further deliveries through a revoked lease. This must work **without MCP and without PumpkinMe**.


## 9. Multiple discovery paths, one protocol

Discovery is a product feature with independent entry routes: ordinary web search, domain-based `/.well-known/paep.json`, A2A-compatible agent cards, Agent Skills, CLI/Code Mode, and direct endpoint/room invitations. These are not six competing business APIs: they all lead to the same PAEP capability contract and verified endpoint authorization. No global central registry is required for the first version.

Three different questions must remain separate: (a) how does a client discover that PAEP exists, (b) how does it locate a particular compatible PAEP service, and (c) how does it locate an intended communication partner? A known-domain well-known URL addresses (b), **not** (a). A direct invitation may address (c), but is **not** itself permission to join a room. See [Discovery Architecture](DISCOVERY.md).

## 10. Communication authority and encrypted JSON

The PAEP service is an infrastructure participant only when explicitly identified as such. It may issue its own signed administrative messages (e.g., a pairing request, delivery error, lease expiry notice) and relay other parties' ciphertext. It **must not** decrypt third-party confidential messages or impersonate the human/agent sender. Separate capabilities for message creation, relay and decryption are essential; possession of a lease is not possession of a private key.

Endpoint JSON is generated on demand. Confidential client-authored JSON fields and message payloads are encrypted **at their originating client**, before the server receives them. PAEP needs only the minimum plaintext routing, key-binding and authorization metadata to process them. Encrypting JSON after constructing it in plaintext on the server would **not** protect it from that server.

People and AI agents use distinct keys and permitted recipient sets. ChatGPT or any external LLM cannot read a message unless an authorized client deliberately gives that plaintext to it; once shared with an AI host, the host's data handling applies.

## 11. Blockchain is explicitly post-MVP

A blockchain, distributed consensus, token, miner, smart contract and blockchain-backed identity are **not** part of the first product. The MVP uses the minimum ordinary persistence and operational audit necessary for correct delivery and revocation. A hash-chained ledger or external notarization may be investigated later only against a demonstrated integrity requirement; neither is an implied dependency now.
