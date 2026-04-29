# magen-orbital-trust

**Orbital Trust Fabric** — research-preview documentation for decay-gated delegation in agentic systems.

This repository is the public documentation index. It holds the protocol specification (RFC), the SDK integration guide, and the FIR Lab demo overview. Live code for the demo and SDK lives in separate repositories when published; the copies here track the same content for a single entry point.

## What this repository does not disclose

Nothing here states **where** a trust control plane or conforming clients might run (for example cloud regions, enterprise networks, orbital or aerospace systems, industrial sites, or remotely accessed field or IoT fleets). **Deployment topology, transport bindings, link-layer or wide-area communication design, cryptographic wire formats, and operational parameters** belong in private integration material and are intentionally absent from this index. The text is **not** a map of production systems, physical infrastructure, or adversary models against any particular environment.

## Documentation

| Document | Description |
| -------- | ----------- |
| [FIR Lab — False-Interrupt Rate Demo](docs/demo/README.md) | Interactive simulation overview: parameters, scope, and how to run the demo locally. |
| [orbital-sdk](docs/sdk/README.md) | Client SDK for control-plane integration: `Envelope`, `Hop`, `Gate`, `Challenge`, and mock mode. |
| [RFC-001 — Decay-Gated Delegation](docs/protocol/RFC-001-decay-gated-delegation.md) | Interoperability surface for delegation chains: envelope structure, gate contract, verdict semantics. |

## License and use

Each document states its own license and use restrictions. This is a research preview, not production security infrastructure.
