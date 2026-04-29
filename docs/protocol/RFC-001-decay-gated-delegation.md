# RFC-001 — Decay-Gated Delegation for Agentic Systems

**Status:** Draft · Preview Release
**Author:** The AI Cowboys
**Last updated:** 2026-04
**Discussion:** Direct contact only. See "Discussion and feedback" below.

---

## Abstract

This document specifies a **logical** interoperability surface for delegation chains in agentic systems where downstream actions must be gated by a confidence value that decays as a function of time, hop depth, and semantic distance from the originating intent. It defines envelope **roles**, verdict semantics, and the contract between client SDKs and a trust control plane. It does **not** identify deployments, networks, or physical environments (including space, remote access, or embedded or IoT contexts). It does not specify the scoring function, the drift estimator, cryptographic algorithms, on-the-wire framing, or any component whose disclosure would degrade the security properties of a conforming deployment or assist targeting of real infrastructure.

## Motivation

Current agent frameworks treat delegated authority as transitive and effectively permanent within a session. An action authorized by a human at hop 0 carries the same weight when executed by a sub-agent at hop 6, even if the chain has drifted from the original intent, even if the elapsed time exceeds the operator's actual presence, and even if the action's blast radius is categorically larger than what the operator authorized.

This specification establishes a minimal contract under which:

1. Every action is associated with a signed envelope tracing back to a human-issued origin.
2. Confidence in the chain is a numeric value bounded by the origin's initial confidence and monotonically non-increasing across the chain.
3. Actions are gated by criticality tier, with higher tiers requiring higher confidence.
4. Failed gates produce one of three verdicts: `PASS`, `CHALLENGE`, `INTERRUPT`.

How confidence is computed is out of scope. This document specifies only what conforming clients and control planes exchange.

## Terminology

- **Principal** — A human operator authenticated to issue origin envelopes.
- **Envelope** — A signed assertion of intent that propagates across hops.
- **Hop** — One delegation step from one agent to the next.
- **Gate** — A policy check executed before a tiered action.
- **Verdict** — The result of a gate check: `PASS`, `CHALLENGE`, or `INTERRUPT`.
- **Tier** — A criticality classification: `READ`, `MUTATE`, `DESTRUCTIVE`. Implementations may define additional tiers but MUST preserve ordering.
- **Control plane** — The system component that evaluates gates and returns verdicts.

## Envelope structure

Field names and types below describe **logical** requirements for the contract between clients and control planes. Serialization, canonical encoding, and how signatures are embedded on the wire are **deployment-specific** and are not specified in this public draft; those details are distributed under separate policy where needed.

An envelope is a signed object with at minimum the following conceptual fields:

| Field          | Type                  | Notes                                                       |
| -------------- | --------------------- | ----------------------------------------------------------- |
| `envelope_id`  | string                | Unique within a deployment.                                 |
| `origin_id`    | string                | Stable identifier for the originating principal.            |
| `intent`       | string                | Free-form, bounded length. Captured for drift evaluation.   |
| `criticality`  | enum (Tier)           | Maximum tier this envelope authorizes.                      |
| `issued_at`    | timestamp             | RFC 3339.                                                   |
| `chain`        | array of hop records  | Append-only. See below.                                     |
| `trust_ctx`    | opaque bytes          | Control-plane-defined. Clients MUST NOT inspect or modify.  |
| `signatures`   | array of signatures   | Verifiable under the deployment’s agreed profile. See below. |

Each hop record contains:

| Field          | Type      | Notes                                                       |
| -------------- | --------- | ----------------------------------------------------------- |
| `hop_index`    | integer   | Zero-indexed. Hop 0 is the origin.                          |
| `agent_id`    | string    | Identifier of the agent producing this hop.                 |
| `received_at`  | timestamp | When the inbound envelope was received.                     |
| `produced_at`  | timestamp | When the outbound envelope was produced.                    |

The `chain` field is append-only. Conforming agents MUST NOT remove, reorder, or modify prior hop records.

## Signatures

Envelopes MUST carry signatures that verifiers can check under the **deployment’s agreed profile**. This public draft **intentionally does not** name specific algorithms, curves, hybrid-binding rules, or post-quantum choices; those are selected and distributed under separate policy and must not be inferred as a description of any live system. Algorithm or profile identifiers in conforming envelopes SHOULD be versioned to permit migration.

This document does not specify key management, certificate policy, or cryptographic material on the wire.

Long-lived signing keys SHOULD be held in an environment that excludes interactive human access during normal operation.

## Gate contract

A client requests a gate evaluation by submitting:

- The current envelope.
- The proposed action's declared criticality tier.
- An optional action descriptor, bounded length, captured for drift evaluation.

The control plane returns a verdict:

- **`PASS`** — The action MAY proceed. The client receives an updated envelope reflecting any state change from the gate evaluation.
- **`CHALLENGE`** — The action MUST NOT proceed until the originating principal has supplied fresh authorization. The client receives a challenge handle to present to the principal. On successful challenge resolution, the client receives a refreshed envelope.
- **`INTERRUPT`** — The action MUST NOT proceed. The client receives a verdict ID for audit purposes. No challenge path is available; the chain has exceeded its allowed envelope.

Clients MUST respect the verdict. A client that proceeds with an action after receiving `CHALLENGE` or `INTERRUPT` is non-conforming.

## Tier ordering

Tiers are totally ordered:

```
READ < MUTATE < DESTRUCTIVE
```

Confidence requirements are non-decreasing in tier. Implementations MAY add tiers between or above these levels. A gate request for tier T evaluates against the gate threshold for T, regardless of the envelope's declared maximum.

An envelope with `criticality: MUTATE` cannot authorize a `DESTRUCTIVE` action even if confidence exceeds the destructive gate threshold; the declared envelope criticality is a ceiling. Conforming control planes MUST enforce this.

## What is not specified

The following are deliberately outside the scope of this document:

- The scoring function used to compute confidence.
- The mechanism by which behavioral or telemetric signals enter the trust context.
- The drift estimator used to compare action descriptors against origin intent.
- The numeric values of gate thresholds.
- The challenge mechanism presented to principals.
- The audit log format.
- The transport binding between client and control plane.
- Deployment context: asset inventory, geographic or orbital placement, industrial, space, battlefield, or remote field use, or use on embedded or IoT devices.
- Communications and encryption design: link layers, satellite or radio bearers, WAN paths, session establishment, or how this contract maps to externally protected channels.

A conforming implementation MUST NOT expose these details through the client surface. A client that requires access to any of the above is operating outside this specification.

This draft must not be read as asserting **where** a conforming system operates or how its traffic reaches a control plane; any such profiling is hypothetical and unrelated to unnamed deployments.

## Security considerations

This section is deliberately brief. Detailed threat modeling is conducted under separate cover. This publication does not describe defenses against adversaries in named physical domains (including space- or network-segment–specific attacks); it states only generic expectations for logical conformance.

- The confidentiality of `trust_ctx` is load-bearing. Clients that log, persist, or retransmit it outside the control plane interaction are non-conforming.
- The decay-gated model assumes adversaries cannot forge envelopes. Envelope signature verification is mandatory at every hop; agents that skip verification eliminate the security properties of the chain.
- The model does not protect against an adversary who fully controls the originating principal's authentication and presence signals. It is designed to constrain delegation, not to replace authentication.
- Mock backends used in development MUST NOT be deployed in environments where verdicts have operational consequences.

## Discussion and feedback

This specification is published for evaluation by integrators with a legitimate operational need. Public discussion of internal mechanisms, parameter calibration, or attack surfaces is discouraged.

For substantive feedback on the interface contract, contact the authors directly via the address on the project landing page. Do not open public issues describing suspected weaknesses, attack vectors, or assumptions about deployed systems.

## Versioning

This document is RFC-001 at status Draft. Breaking changes will produce RFC-002. Non-breaking clarifications produce dated revisions of this document.

---

*This is a specification for an interoperability surface. It is not a product roadmap, a security claim, or a representation of any deployed system's behavior.*
