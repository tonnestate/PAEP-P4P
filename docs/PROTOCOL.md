# PAEP — Protocol Draft for P4P

**Status:** Draft 0.1, 2026-11-23. **Not implemented.** The paths and message types below are an initial contract for review, not registered Internet standards or deployed URLs.

## 1. Layers

1. **Discovery:** Publicly describes supported versions, operations, and transport bindings.
2. **Resource model:** Endpoint, principal, room, lease, key binding, message, and change cursor.
3. **Authority:** Verifies identity and delegation, roles, scopes, expiration, revocation, and version preconditions.
4. **Messaging:** Message envelope and opaque ciphertext; confidential payloads are processed only by authorized clients.
5. **Transport:** Initially HTTPS/JSON, with identical operation semantics for CLI, controlled file import, and optional MCP.
6. **Presentation:** Optional human chat displaying the same state, with no separate protocol logic.

A GUID identifies a resource and **never acts as authentication on its own**.

## 2. Entities

| Entity | Minimum meaning |
|---|---|
| `endpointId` | Random opaque endpoint identifier; public visibility never includes private state |
| `principalId` | Authenticated human, organizational representative, or delegated agent |
| `role` | `human`, `organization_human`, or `agent`; actual permission scopes are checked separately |
| `roomId` | Room linking exactly two sides/endpoints (potentially with multiple authorized principals) |
| `leaseId` | Time-limited authorization binding principal, room, purpose, permissions, and revision |
| `keyId` | Fingerprint or reference of a verified public recipient key; no private-key storage |
| `messageId` | Random message ID supporting deduplication |
| `cursor` | Monotonic, room-scoped change position; never a globally readable activity feed |

Private keys remain with the client. Publishing a key identifier **does not by itself establish** the key holder's identity.

## 3. Platform-independent operations

| Operation | Purpose | Security rule |
|---|---|---|
| `DISCOVER` | Read publicly available protocol information | No private state |
| `ENDPOINT_CREATE` | Provision an endpoint for an authenticated principal | Rate-limited; no implicit agent access |
| `KEY_REGISTER` | Register a public key and proof of possession | Verify signed possession proof and identity binding |
| `DELEGATE` | Authorize an agent | Only legitimate human or organizational representative |
| `ROOM_PROPOSE` | Propose a connection to another endpoint | Requires an authorized principal |
| `ROOM_ACCEPT` | Confirm pairing and issue scoped lease | Requires both parties' approvals and applicable policies |
| `MESSAGE_SEND` | Submit ciphertext for explicitly specified recipients | Lease, sender authenticity, and replay protection |
| `CHANGES_READ` | Retrieve authorized changes after a cursor | Read permission, pagination, and quotas |
| `MESSAGE_ACK` | Acknowledge delivery or retrieval | Does not imply that a human read the message |
| `LEASE_REVOKE` | Terminate future access | Authorized principal or authority only |
| `KEY_ROTATE` | Establish a new verified key binding | No silent key substitution |

The generic core has **no `IDENTITY_REVEAL` operation**. Applications may implement separate human approval procedures; agent delegation must never imply the ability to reveal identity.

## 4. Multi-path discovery (one canonical contract)

PAEP uses **multiple independent discovery routes**, each mapped to one canonical, versioned description of the service's protocol capabilities. No individual route is a hard dependency:

1. **Ordinary web search:** A publicly indexable human-readable explanation (proposed `GET /for-agents`) links to machine-readable metadata. Search appearance and agent adherence cannot be guaranteed.
2. **Known-domain well-known JSON:** Proposed `GET /.well-known/paep.json`; retrieves metadata from an **already known domain**. This name is not a registered Internet well-known URI standard.
3. **Agent cards / A2A:** An optional compatibility adapter advertises PAEP capability through an A2A-compatible agent card; do not claim that A2A semantics are inherently PAEP semantics.
4. **Agent Skills:** Optional installable or referenced `SKILL.md` documents available operations and permitted transport methods. A skill is guidance, not executable authority.
5. **CLI / Code Mode:** Local approved commands and `--help` / machine-readable capabilities expose the same operations for Claude Code, GitHub Copilot, and other tool-enabled agents without MCP.
6. **Direct invitations:** A deliberate invitation points to an endpoint/room reference and discovery metadata; it must still require independently verified authentication, valid pairing authority and consent before joining.

The authoritative discovery document lists supported protocol versions, endpoint URLs, transport bindings, optional adapter capabilities, public key verification methods, schema/documentation URLs, authentication requirements, and limits. It must **not** expose private profiles, messages, secrets, private keys, participant lists or room occupancy.

Distinguish discovering **the protocol**, finding **a compatible service**, and locating **an intended partner**. A well-known URL solves only known-domain service discovery. Public discovery never proves identity, authenticates a sender, grants a lease, or authorizes access. See [DISCOVERY.md](DISCOVERY.md) for resolver precedence, trust separation, and conformance tests.

An agent needs actual host-authorized HTTP, browser, file or CLI capabilities; public instructions cannot force behavior or create background access.

## 5. Provisional HTTPS bindings

| Method / path | Operation |
|---|---|
| `POST /v1/endpoints` | ENDPOINT_CREATE |
| `GET /v1/endpoints/{endpointId}` | Authorized dynamic JSON view |
| `POST /v1/endpoints/{endpointId}/keys` | KEY_REGISTER / KEY_ROTATE |
| `POST /v1/endpoints/{endpointId}/delegations` | DELEGATE |
| `POST /v1/rooms` | ROOM_PROPOSE |
| `POST /v1/rooms/{roomId}/accept` | ROOM_ACCEPT |
| `GET /v1/rooms/{roomId}/changes?after={cursor}` | CHANGES_READ |
| `POST /v1/rooms/{roomId}/messages` | MESSAGE_SEND |
| `POST /v1/rooms/{roomId}/messages/{messageId}/ack` | MESSAGE_ACK |
| `POST /v1/leases/{leaseId}/revoke` | LEASE_REVOKE |

Every private operation requires **verified authentication and authorization bound to the transport**. Public URLs and logged JSON payloads must not carry usable secrets, regardless of how opaque their resource IDs look.

## 6. Example: Dynamic authenticated endpoint view

~~~json
{
  "protocol": "paep",
  "version": "0.1-draft",
  "endpointId": "ep-example-a",
  "revision": 2,
  "keys": [
    {"keyId": "key-human-a", "purpose": "encrypt", "verified": true},
    {"keyId": "key-agent-a", "purpose": "encrypt", "verified": true}
  ],
  "rooms": [
    {"roomId": "room-example", "state": "active", "cursor": "12"}
  ]
}
~~~

This is a **synthetic schema example**, not the final JSON Schema file. A real response would include only key and room details permitted for the authenticated principal. Public keys are delivered separately as verifiable cryptographic data. A `verified` boolean alone does not replace independent key authentication.

## 7. Message envelope (synthetic example)

~~~json
{
  "protocol": "paep",
  "version": "0.1-draft",
  "type": "MESSAGE_SEND",
  "messageId": "7ae5eb52-4729-4cf1-9da6-d804a7960dbc",
  "roomId": "room-example",
  "sender": "agent-a",
  "recipientKeyIds": ["key-human-b"],
  "leaseId": "lease-example",
  "expectedRevision": 7,
  "crypto": {
    "format": "hpke-aead-envelope-draft",
    "ciphertext": "<base64url-encrypted-payload>",
    "keyEncapsulations": "<recipient-specific-key-material>",
    "senderProof": "<signed-authenticated-envelope>"
  }
}
~~~

Values in angle brackets are placeholders, **not working cryptography**. The final cipher suite, signature coverage, canonical serialization, nonce rules, and wire-compatible JSON Schema must be specified before implementation and validated with test vectors. The `leaseId` is a reference, not proof of access.

**Critical:** The recipient list defines which roles may receive plaintext. Humans and agents have **separate keys**. Agents may access human-private messages only if explicitly authorized as recipients or processing parties.

## 8. State machines (target design)

- Endpoint: `CREATED → ACTIVE → SUSPENDED / CLOSED`.
- Room: `PROPOSED → ACCEPTED → ACTIVE → REVOKED / CLOSED`. `MESSAGE_SEND` is prohibited before `ACTIVE`.
- Lease: `ISSUED → ACTIVE → EXPIRED / REVOKED`; a revoked lease cannot be reactivated.
- Message: `RECEIVED → QUEUED → DELIVERED/EXPIRED`. A delivery acknowledgement **does not mean** a human has read or approved a message.

Every state-changing request requires idempotent handling, an expected revision, and verified role/scope authority. Unknown fields and unsupported versions are rejected.

## 9. Synchronization and errors

- Monotonic room-scoped cursors and incremental changes; no complete profile or chat replication on every read.
- Message IDs for deduplication, TTL, and bounded retries; safe replay of requests after disconnection.
- `401` missing/invalid authentication; `403` insufficient authority; `404` resource not visible; `409` revision conflict; `410` expired/deleted resource; `429` rate limit.
- The same protocol-level error object applies to CLI and file transports independently of HTTP status codes.
- Signed or offline-imported messages must pass the same authorization and replay checks as live HTTPS messages.

## 10. Protocol boundaries

PAEP does not treat general LLM prompts as executable system instructions. Received messages and discovery documents are **untrusted content**. The protocol does not itself execute agent actions, create ChatGPT tasks, send push notifications, or schedule background sessions without explicit support from the host environment.

The first version binds **two endpoints** per room. Later multi-party or multi-device designs require separately reviewed group key management.


## 11. Separation of original messages, relay operations, and decryption

The PAEP service may originate its **own** authenticated administrative envelopes with explicit service identity and `message:send` rights. It may relay opaque ciphertext with `message:relay` rights subject to valid routing authorization. Neither action entitles it to read confidential messages sent by other participants.

`message:receive` and `room:read` govern server-mediated delivery and access to the encrypted record. **Decryption is a client-side cryptographic ability**, not a scope that grants the server a missing private key. A server-side `message:decrypt` permission must not be invented to circumvent recipient encryption. A private message must be encrypted to each intended authorized recipient principal's verified key, whether human or agent; delegation alone does not implicitly add the agent to the recipient set.

The authorization service may be backed by BlueberryMe when available, but the standalone PAEP contract requires no specific external authority service.

## 12. Encrypted JSON resource model

The client creates sensitive JSON and encrypts it **before** sending it to the PAEP server. The server may dynamically assemble permitted endpoint views around opaque encrypted fields and stable resource references. Plaintext server-side metadata is limited to the information needed for protocol/version negotiation, routing, permissions, signature verification and minimal delivery operation.

The server must not claim that data it generates in plaintext and only then encrypts is hidden from the server. Keys are authenticated and checked separately; no private key is provided in the JSON envelope, URL, relay or AI prompt.

## 13. First-version exclusions

No operational blockchain, distributed consensus, token or mandatory hash-chain audit ledger. No compulsory MCP. No public global directory of private endpoints. No background action triggered merely by discovery. All of these remain outside the protocol's first implementation, with any later extension subject to a separate decision.
