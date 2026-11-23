# PAEP / P4P

**An independent, vendor-neutral communication protocol for people, organizations, and their AI agents.**

**Status:** Product and protocol concept v0.1, November 23, 2026. There is no running server, deployed tunnel, verified end-to-end encryption, or demonstrated ChatGPT, Claude, or Copilot integration yet.

## The idea in one sentence

Two participants each receive a dynamically retrievable **JSON communication endpoint**. PAEP links the endpoints into a communication room through authorized, purpose-bound leases. Humans and their delegated agents can communicate with separate permissions; a transport adapter (initially n8n as a proposed relay) carries encrypted messages without processing their plaintext.

**P4P** denotes the communication model with four possible roles: Human A, Agent A, Human B, and Agent B. The four roles do not all need to be active or online simultaneously. Either side may also be an organization represented by a human and an agent.

## Product principles

- **Independent product:** Not part of PumpkinMe or FruitStack. Either may later integrate PAEP/P4P as a client or adapter. BlueberryMe is not a mandatory runtime dependency.
- **JSON first:** No requirement to create an individual website for every user. JSON views are generated dynamically from minimal persistent state. A human-facing chat view is optional.
- **Discoverable on the open web:** Public instructions and machine-readable discovery are available. Whether an agent can execute any operation depends on its host environment and granted permissions.
- **No mandatory MCP:** HTTPS/JSON, authorized CLI/Code Mode operations, and controlled JSON file exchange share the same contract. MCP and agent skills are optional adapters or instructions.
- **Separate authority:** Humans and agents have distinct permissions. An agent cannot replace human approval or authorization to disclose identity.
- **Confidential communication:** Messages are designed to be end-to-end encrypted on the client before transmission. The service can still observe technical metadata. E2EE security depends on trustworthy clients and independently verifiable key bindings.
- **Minimal storage:** Do not persist duplicate profiles or message plaintext. GitHub hosts source code, not the operational ledger.
- **Do not invent cryptography:** Use established, audited standards and libraries.

## Minimal workflow

1. A person asks an existing AI agent to accomplish a task, such as finding a new professional opportunity.
2. The agent discovers the public PAEP instructions and uses an authorized access method **only if its runtime supports the operation and the person approves**.
3. PAEP provisions an endpoint with a public identifier, minimal state, and distinct permissions. Private keys remain with trusted clients.
4. A second endpoint is discovered or addressed explicitly. An authorized participant proposes a connection, and the relevant parties accept it.
5. A revocable lease authorizes participation in a communication room.
6. Endpoints exchange verified public-key information and encrypted messages over the configured transport.
7. Humans can read and reply through an optional interface **only when they are authorized recipients in that message context**.

## Terminology

- **PAEP:** Provisional name of the exchange protocol. Its earlier PumpkinMe-specific expansion does **not** define this independent product.
- **P4P:** Working name for the human-and-agent communication model with four possible roles.
- **Endpoint:** Addressable communication resource with an ID, public-key information, and permission state; not necessarily an HTML page.
- **Room:** Authorized relationship between two endpoints, tracking participants, leases, event revisions, and ciphertext.
- **Lease:** Verified, time-limited and scope-limited authorization, not itself an encryption key.
- **Relay:** Transport for ciphertext and minimal delivery metadata; never an authority over private keys.

## Documentation

- [Product concept and scope](docs/PRODUCT_CONCEPT.md)
- [PAEP protocol draft and JSON contract](docs/PROTOCOL.md)
- [Security and trust model](docs/SECURITY.md)
- [MVP plan and acceptance criteria](docs/MVP.md)
- [Architecture decisions and open questions](docs/DECISIONS.md)

## Technical direction

**TypeScript** for the protocol core and JSON API; **DuckDB** as a minimal persistence candidate for the first standalone service; **n8n** as a replaceable relay adapter. The system must be testable locally without external AI services or MCP. GitHub Actions/CI are not required.

## Current limitations

This repository currently documents a target design. It does not claim working encryption, production-grade security, independently established protocol novelty, regulatory compliance, automatic discovery by ChatGPT, or guaranteed compatibility with any agent platform.

License: [MIT](LICENSE).
