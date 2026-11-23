# PAEP / P4P — Security and Trust Model

**Status:** Draft 0.1. This document defines requirements, **not** security properties that have already been independently verified.

## 1. Trust boundaries

**Trust is required in:** the actual client software and its key management, the binding between public keys and intended recipients, and the authorization decisions for each principal.

**Treat as untrusted:** public discovery instructions, messages from external agents, arbitrary external JSON, the n8n relay and its operators, transport intermediaries, and a compromised PAEP server with respect to the confidentiality of message contents.

**Current limitation:** If the PAEP server itself serves the browser JavaScript application, a malicious or compromised operator can substitute client code or keys. A defensible claim that “even the operator cannot read messages” also requires independently verifiable client distribution and a trusted key-binding path resistant to key substitution. HTTPS and public keys alone do not solve this problem.

## 2. Keys and encryption

- Each authorized **encryption recipient** has an independent locally controlled key pair. Roles never share a private key.
- A **public key** may be shared but must be correctly bound to its principal. Fingerprint verification, signed proof of possession, trusted binding/pinning, and rotation are security acceptance requirements.
- The **private key** remains under the endpoint holder's control. It must never appear in a URL, query parameter, n8n, server database, GitHub, ChatGPT prompt, or generic agent logs.
- Messages use an established hybrid E2EE library or standards-based suite (for example, HPKE RFC 9180 for recipient encryption with authenticated sender-envelope handling; AES-GCM as the AEAD). **Do not assemble custom X25519, nonce management, or key derivation mechanisms.**
- The same plaintext may be encrypted for multiple authorized recipients without distributing a shared private key between a human and an agent.
- Signatures or an otherwise authenticated protocol envelope bind the sender, room, lease reference, recipient IDs, message ID, revision, and ciphertext together.
- Linking a lease does not create a server-controlled shared symmetric “room key.”

## 3. A lease is not a decryption key

A lease is an authorization object: `leaseId`, verified issuer, principal, role, room and endpoint pair, scope, purpose, key fingerprints, issuance, expiry, revocation, and revision. The server never trusts unverified lease fields supplied in client JSON.

Example scopes: `room:read`, `message:send`, `message:receive`, and `room:close`. The `human:approve` right is never derived implicitly from agent permissions. Delegations follow least privilege and fail closed.

An expired or revoked lease blocks future server-authorized operations. **It cannot delete ciphertext that has already been delivered or revoke keys or plaintext already obtained locally.** Rotation and a new cryptographic session are needed to change the authorized recipient set for future traffic.

## 4. What the server and relay can observe

PAEP and n8n must not receive plaintext, private keys, or passwords without explicit client disclosure. Depending on deployment, however, they can observe room IDs, public keys, network addresses, message timestamps and sizes, delivery errors, and connection graphs. These can be personal-data-bearing metadata; they require minimization, access controls, retention policies, and a separate data-protection review.

E2EE protects message contents from the relay **only while clients encrypt correctly and key bindings are authentic**. If an agent sends decrypted message content to an AI provider for processing, that provider can process it according to its environment and policies. E2EE does **not** extend blindly into the language model.

## 5. Threats and controls

| Risk | Required control |
|---|---|
| Public-key substitution / man-in-the-middle | Independent key binding, proof of possession, verifiable fingerprints, trusted client |
| Agent impersonating a human | Separate principals and keys, scoped leases, explicit human approval |
| Replay / duplicate delivery | Message ID, idempotent commit, nonce/revision, expiry and cursor checks |
| GUID / URL disclosure | GUID is only a locator; separate authentication and authorization; no tokens in URLs |
| Server substitutes browser code | Independently verifiable client artifacts and software supply chain; no absolute E2EE claim until addressed |
| LLM prompt injection | Discovery and message contents are data, not executable system instructions |
| Spam / endpoint flooding | Quotas, rate limits, identity and abuse controls, bounded retention |
| Ciphertext sent outside recipient scopes | Validate recipient list and lease before accepting or delivering; enforce server authorization in addition to cryptography |
| Relay outage | Minimal durable outbox, retry/ack, TTL; no silent data loss |
| Message storage | Ciphertext only, bounded TTL, access controls, deletion and export provisions |

## 6. Not guaranteed by default

- No guaranteed anonymity: metadata and user-supplied statements may disclose identity.
- No automatic GDPR, ePrivacy, or DSA compliance just because encryption is used.
- Public-key possession alone does not reliably establish an independently verified sender identity.
- Lease revocation does not protect material already disclosed.
- Public protocol documentation does not grant external agents unlimited autonomy.
- No claim of original cryptography or uniquely novel agent messaging.

Security review, negative tests, key-substitution tests, client supply-chain verification, and threat-model review are required before any external production release.


## 7. Multi-discovery threats

Discovery sources (search results, web pages, well-known JSON, A2A agent cards, skill files, CLI text, and invitations) are **untrusted until authenticated**. None may supersede host instructions or grant permission. A malicious discovery document may present spoofed endpoints, substitute public keys, introduce misleading instructions, leak credentials or point the server to an internal network address.

Discovery resolution must distinguish evidence of service control (for example, an HTTPS origin) from independently verified identity and cryptographic recipient-key binding. Protect fetchers from SSRF, redirects to private network space, excessive payloads, unknown hosts and downgrade attacks. Never follow a discovery claim into a private API without explicit host approval.

No public search directory of private user endpoints, leaked partner inventories or room membership. Direct invitations contain only non-secret locators and require a separate authenticated acceptance.

## 8. Service-originated messaging versus unreadable participant messaging

The infrastructure may generate **only its own explicitly attributed** administrative or invitation messages. It may forward third-party ciphertext without decrypting. It may not present itself as a human principal or read message bodies through a relay or elevated server scope.

Sensitive client-authored JSON must be encrypted on the originating endpoint, not first assembled in plaintext by the service. The server must not store the original plaintext. For any given ciphertext, only cryptographically designated authorized recipients should have the necessary key material.

**Do not conflate:** the ability to submit ciphertext (`message:send`), route it (`message:relay`), retrieve authorized encrypted records (`message:receive`), or decrypt locally with a private key. A lease governs allowed operations; it does not make missing decryption keys available.

## 9. No blockchain or hash-ledger requirement for MVP

Operational logging, retries, replay detection and data retention remain necessary for reliability. They do **not** imply a blockchain, distributed trust layer, cryptocurrency or hash-chained event system. Such a system would need a separate threat model, independent verifier and explicit phase approval. GitHub is not a message ledger.
