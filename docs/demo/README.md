# FIR Lab — False-Interrupt Rate Demo

Interactive simulation environment for exploring false-interrupt rates under decay-gated delegation. Part of the Orbital Trust Fabric research preview.

> **This is a simulation.** No production telemetry, detection logic, or calibrated parameters are exposed in this repository. All traces, decay coefficients, and outcome distributions are illustrative.

## What this shows

When agentic systems delegate authority across multiple hops, the operator's confidence that the downstream action still reflects original human intent should not remain constant. FIR Lab visualizes the tradeoff space:

- Tighter gates → fewer compromised actions slip through (higher TIR), more legitimate actions are blocked (higher FIR).
- Looser gates → the inverse.
- The interesting question is not "what is the FIR" but "what is the FIR *for a given workload at a given safety posture.*"

The demo lets you move three parameters (λ, α, β) and three thresholds (T1/T2/T3) across five synthetic workload profiles, and observe how the false-interrupt rate, true-interrupt rate, and mean chain depth respond.

## What this does NOT show

- The actual decay function used in production deployments.
- The drift estimator, behavioral signal pipeline, or any component of the trust scoring system.
- Real adversarial traces, real operator telemetry, or real workload distributions.
- Calibrated parameter values, gate thresholds, or operating points used in any deployment.

The numbers are designed to move in plausible directions when you adjust controls. They are not predictive of any real system's performance.

## Running locally

```bash
npm install
npm run dev
```

Opens on `http://localhost:5173`. No environment variables, no backend, no network calls.

## Stack

TypeScript · React · Vite · Tailwind. Charting via Recharts. No telemetry, no analytics, no third-party state.

## Repository scope

When published as its own repository, that repo contains only the demo frontend. The trust-decay protocol specification, SDK surface, and integration documentation are mirrored in this repository’s [documentation index](../../README.md).

## Use restrictions

This demo is published as a research preview for evaluation and discussion. It is not a product, not a reference implementation, and not suitable for any production or operational use. Do not deploy, do not adapt for live decisioning, do not represent it as functional security infrastructure.

## License

Source available under a research-preview license. See `LICENSE`. Commercial use, redistribution, and derivative works require written permission.

## Contact

For technical inquiries, partnership questions, or controlled-disclosure requests, contact the maintainers via the address listed on the project landing page. Please do not file public issues describing suspected vulnerabilities, attack vectors, or operational assumptions about related systems.
