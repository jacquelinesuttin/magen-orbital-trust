# orbital-sdk

Client SDK for integrating with an Orbital Trust Fabric control plane. Provides typed primitives for issuing signed envelopes, propagating delegation context across hops, and gating actions by criticality tier.

> **Preview release.** Interface surface is stable enough to prototype against. The reference control plane is not publicly available. This SDK ships with a local mock for development only. This document does not describe deployment environments, transport, or how clients reach a control plane across any particular network (including wide-area, embedded, or space-linked paths).

## Install

```bash
npm install orbital-sdk
# or
pip install orbital-sdk
```

## Concepts

The SDK exposes four primitives. Implementation details of the underlying trust model — scoring, decay, drift estimation, behavioral signal collection — are intentionally not part of the public surface.

### `Envelope`
A signed statement of intent issued by an authenticated principal at hop 0. Envelopes carry a criticality declaration, an origin signature, and an opaque trust context.

### `Hop`
A delegation step. Each hop accepts an inbound envelope, produces an outbound envelope for the next agent, and contributes telemetry to the trust context. Hops cannot elevate criticality.

### `Gate`
A policy check executed before a tiered action. Returns `PASS`, `CHALLENGE`, or `INTERRUPT`. Callers must respect the verdict; the SDK does not execute the action itself.

### `Challenge`
A request bubbled back to the originating principal for fresh authorization. Resolution returns a refreshed envelope with updated trust context.

## Minimal example (TypeScript)

```ts
import { Orbital, Tier } from "orbital-sdk";

const orbital = new Orbital({ mode: "mock" }); // local mock; no network

// Hop 0 — origin
const envelope = await orbital.issue({
  intent: "example:intent-alpha",
  criticality: Tier.MUTATE,
});

// Hop N — downstream agent prepares a higher-tier action
const verdict = await orbital.gate(envelope, {
  action: "example-action-beta",
  criticality: Tier.DESTRUCTIVE,
});

switch (verdict.outcome) {
  case "PASS":      /* proceed */ break;
  case "CHALLENGE": /* await orbital.challenge(envelope) */ break;
  case "INTERRUPT": /* halt and log */ break;
}
```

## What is not in this SDK

- The decay function. Not exposed, not configurable from the client.
- Drift estimation. Computed server-side; clients receive only the verdict.
- Behavioral signal collection. Handled by separate agents under controlled distribution.
- Threshold values. Set per-deployment by the control plane; clients cannot read or modify them.
- Any algorithm, model, or heuristic that constitutes detection capability.

If you find yourself wanting to override or inspect any of the above, this SDK is not the right integration surface for your use case.

## Mock mode

`mode: "mock"` returns deterministic synthetic verdicts based on declared criticality and a local pseudo-random seed. It is suitable for unit tests, integration scaffolding, and UI development. It is not suitable for any form of evaluation, benchmarking, or security analysis. Verdicts produced in mock mode have no relationship to verdicts a real control plane would produce.

## Versioning and stability

The public interface follows semver. The mock backend, fixture data, and any internal helpers are not part of the public surface and may change without notice. Do not depend on mock-mode output values.

## Use restrictions

Preview release for integration prototyping. Not licensed for production deployment, security-critical decisioning, or any use where the verdict materially affects safety, compliance, or financial outcomes. Controlled-distribution builds with operational backends are available under separate agreement.

## Reporting issues

Functional bugs, type errors, and documentation gaps: open a public issue.

Anything touching trust-model behavior, suspected bypasses, or assumptions about a real control plane: do not file publicly. Contact the maintainers directly via the address on the project landing page.

## License

Source available. See `LICENSE`. Includes restrictions on derivative works and commercial redistribution.
