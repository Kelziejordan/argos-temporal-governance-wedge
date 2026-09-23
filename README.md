# ArgOS Temporal Governance Wedge

Independent experimental repository for ARGOS TEMPORAL GOVERNANCE WEDGE V1.

This repository is intentionally separate from the existing ArgOS repositories.

## Frozen purpose

Test whether ArgOS can establish a durable, correlated, evidence-backed governance chain around a consequential action executed through Temporal:

Intent -> Identity -> Authority -> Policy -> Decision -> Execution -> Evidence -> Verification

Temporal remains the durable-execution/runtime substrate. This project does not replace Temporal.

## Repository boundary

- No source-code dependency on the existing ArgOS runtime.
- No shared database or runtime state.
- No shared deployment.
- No automatic import of existing ArgOS components.
- Existing ArgOS work may be referenced as architectural background, but this repository owns its own implementation and evidence.
- Any future integration with ArgOS requires an explicit architectural decision.

## V1 mode

READ-ONLY observation and governance evidence first.

Observability is not enforcement. Enforcement claims require a demonstrably controlled execution path.

## Success condition

A real Temporal execution must be correlated to a governance decision and reconstructed from persisted evidence, while distinguishing authorization, execution, outcome evidence, and verification.

## Public repository policy

This repository contains architecture, implementation, tests, and experiment evidence for the wedge. Do not commit secrets, credentials, customer data, private Temporal endpoints, proprietary customer information, or sensitive evidence.

See docs/FROZEN-SPECIFICATION.md.
