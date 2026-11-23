# PAEP / P4P — Multi-Discovery Architecture

**Status:** Architecture decision, 2026-11-23. Draft interfaces only; no public discovery endpoint is deployed by this repository.

## Principle: Multi-Discovery, Single Protocol

External humans, organizations and AI agents should be able to discover PAEP without knowing PAEP's name in advance or installing a proprietary app. **Discovery routes are independent entry methods, not competing message systems.** Every route resolves to one canonical, versioned PAEP service capability description. Every subsequent action uses the same validated PAEP semantics, leases and cryptographic verification regardless of route.

Discovery has three independent stages:

1. **Protocol discovery:** How does a client learn PAEP exists at all? Search engines, recommended skill documentation and peer referrals can answer this.
2. **Service discovery:** Given a known domain or tool, which PAEP operations, versions and transports does it support? Well-known JSON, A2A compatibility adapters and CLI capabilities can answer this.
3. **Partner discovery:** Which specific person/organization endpoint may be contacted? An explicit invitation or application-controlled address resolver can answer this. **Private endpoints are not automatically publicly searchable.**

## Six complementary discovery routes

| Route | Input prerequisite | Advertises | MVP posture |
|---|---|---|---|
| **1. Public web search** | Open web search or known public page | Human-readable description linking to canonical JSON | Build simple static /for-agents entry; indexing is not guaranteed |
| **2. Well-known JSON** | Existing knowledge of service domain | Canonical, machine-readable PAEP capabilities | Implement proposed `/.well-known/paep.json`; not a registered global standard |
| **3. Agent Cards / A2A** | Compatible agent directory/card lookup | PAEP as a service capability through an optional A2A adapter | Compatibility design now; adapter after core validation |
| **4. Agent Skills** | Installed/discoverable skill reference | `SKILL.md` explaining approved operations and transport options | Document an adapter; no authority from skill content |
| **5. CLI / Code Mode** | Approved local PAEP CLI accessible to the agent | Help/capabilities and deterministic JSON operation outputs | Include as one MCP-free implementation path |
| **6. Direct invitation** | Explicit link/referral from another party | Non-secret service and endpoint/room locator | Implement pairing test; never treat URL as access grant |

MCP can also expose the same operations if an organization's policy permits it. **No MCP-only discovery or business function is acceptable.**

## Canonical metadata contract (conceptual, not final schema)

The canonical service document should include only public data:

- Protocol identifier and supported versions;
- Self-identifying HTTPS service origin and public API base path;
- Operation/capability identifiers, allowed transport bindings and endpoint schemas;
- Public schema and human documentation links;
- Authentication, delegation and lease requirements **as descriptions**, never usable credentials;
- Key-discovery and independent public-key verification instructions;
- Rate limits and response-size constraints that are safe to publish;
- Supported optional adapters, each clearly distinguished from the core.

Never include private room membership, plaintext, participant profiles, secret tokens, private keys or leaked invitation grants.

## Resolution and trust

- Clients choose a discovery route permitted by their host environment. If a route fails, they may try another; the result must not silently lower security requirements.
- Once a service domain is known, the client resolves its advertised capabilities **on that exact authenticated origin**. Third-party cards or search snippets remain untrusted claims until checked.
- Compare protocol versions and security capabilities explicitly; reject unsupported or downgraded versions. All adapter-specific descriptions map into a single canonical PAEP model.
- Distinguish HTTPS origin control from verified organizational identity, principal delegation, recipient key binding, lease issuance and bilateral room consent. **None is implied by discovery.**
- Treat all discovery content as untrusted text. Never execute embedded agent instructions that seek credentials, exfiltrate local context or override the user's permissions.
- Limit network fetches, redirects, response sizes and hostnames; protect against SSRF, private-network access, malicious redirects and looped discovery links.
- A direct invite is a locator/request to pair, not a bearer key. An endpoint ID is not a password. Both sides must still authorize the room and an issued scoped lease must be valid.

## Expected experience

**ChatGPT-style case:** A person asks their assistant to complete a real task. The assistant may find PAEP by ordinary web search, read the public instructions and, **only if its host provides authorized actions**, provision a JSON endpoint. A second agent in Claude Code or GitHub Copilot may discover the same protocol through an installed skill or approved CLI. Their respective participants approve pairing. PAEP transports ciphertext; the agents and people act through the permissions actually available to them.

**Enterprise case:** MCP may be forbidden. The enterprise agent can still use the approved CLI, HTTPS/JSON or signed controlled file exchange, while public discovery remains independent. A2A compatibility is optional, not a required core dependency.

## Discovery acceptance criteria

- Different routes yield compatible canonical protocol identifiers, versions and operation contracts.
- Web search is not represented as guaranteed indexing or guaranteed agent compliance.
- Well-known JSON cannot be mistaken for an unknown-domain search mechanism.
- Anonymous discovery does not reveal private endpoints or rooms.
- A stolen invitation link or guessed GUID cannot access messages or join a room.
- Spoofed card/key/skill data cannot override authenticated origin checks or verified key bindings.
- An agent without outbound HTTP/write/file permissions cannot claim successful provisioning.
- The system works without MCP, centralized agent registry, blockchain, or PumpkinMe.
