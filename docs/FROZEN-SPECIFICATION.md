# ARGOS TEMPORAL GOVERNANCE WEDGE V1 — FROZEN SPECIFICATION

Status: FROZEN FOR BUILD
Version: V1.0
Primary runtime: Temporal
Secondary runtime: None in V1
Repository: Independent

## 1. Thesis and boundary

ArgOS V1 tests one proposition:

For a consequential action executed through Temporal, ArgOS can create a durable, correlated, evidence-backed governance record connecting who intended the action, what authority and policy applied, what decision was made, what the runtime actually did, what evidence exists, and whether the resulting execution can be verified.

ArgOS is a governance control plane, not a durable-execution runtime. Temporal remains responsible for workflow durability and runtime execution semantics.

The frozen chain is:

Intent -> Principal -> Authority -> Policy -> Risk -> Decision -> Admission -> Temporal Execution -> Runtime Evidence -> Outcome Evidence -> Verification

V1 is deliberately independent of the existing ArgOS repositories. No source-code dependency, shared runtime state, shared database, or shared deployment is permitted without a future explicit architectural decision.

## 2. Scope

V1 includes:

- consequential action registration
- principal identification
- authority evaluation
- policy evaluation
- governance decision recording
- Temporal execution correlation
- runtime event ingestion
- evidence recording
- evidence integrity and deduplication
- normalized governance lifecycle
- verification
- one minimal investigation view
- automated tests for the critical invariants

V1 excludes:

- Akka
- replacement of Temporal
- modification of Temporal Event History
- universal Temporal interception
- universal execution prevention
- full enterprise IAM
- full SIEM/APM replacement
- production multi-region architecture
- generalized AI governance
- broad compliance automation
- broad commercial packaging

## 3. Runtime boundary and enforcement strength

Temporal owns workflow persistence, workflow history, workers, scheduling, retries, timers, task queues, and runtime semantics.

ArgOS owns its governance records and evidence.

V1 starts in READ-ONLY OBSERVATION MODE.

ArgOS may observe, correlate, record, evaluate, verify, and report.

Observability is not enforcement.

ArgOS must never claim that it can prevent an execution unless the specific integration path demonstrably controls admission before the consequential action reaches the runtime.

If an enforced path is experimentally built, it is an ENFORCEMENT EXPERIMENT and its claim is limited to that tested path.

Temporal completion is runtime evidence. It is not automatically proof of the external real-world consequence.

## 4. Core domain contract

The minimum governed-action record contains:

- actionId
- actionType
- target
- requestedBy
- requestedAt
- riskLevel
- correlationId
- payloadHash

Principal contains:

- principalId
- principalType
- displayName
- source
- authenticatedAt

Supported principal types:

human, service, agent, workflow, worker.

Authority contains:

- authorityId
- principalId
- scope
- issuedAt
- expiresAt
- authoritySource

Policy contains:

- policyId
- policyVersion
- scope
- decisionRule
- effectiveAt

Governance Decision contains:

- decisionId
- actionId
- principalId
- authorityId
- policyId
- policyVersion
- riskLevel
- decision
- decidedAt
- correlationId

Decision values:

ALLOW, DENY, REQUIRE_APPROVAL.

Approval, when required, contains:

- approvalId
- actionId
- approverPrincipalId
- decision
- issuedAt
- expiresAt
- approvalScope
- correlationId

Runtime Event contains:

- eventId
- runtime
- eventType
- workflowId
- runId
- eventTimestamp
- receivedAt
- correlationId
- sourceReference
- payloadHash

Evidence Record contains:

- evidenceId
- evidenceType
- source
- sourceReference
- observedAt
- ingestedAt
- correlationId
- integrityHash
- trustLevel

Evidence types:

INTENT, AUTHORITY, POLICY, DECISION, APPROVAL, RUNTIME, OUTCOME, VERIFICATION.

Verification Record contains:

- verificationId
- actionId
- expectedState
- observedState
- evidenceReferences
- result
- verifiedAt
- verificationVersion

Verification results:

VERIFIED, PARTIALLY_VERIFIED, UNVERIFIED, CONFLICTING, INVALID.

## 5. Correlation, lifecycle, and invariants

correlationId is a first-class V1 invariant and must survive:

Intent -> Decision -> Approval, if required -> Admission -> Temporal execution -> Runtime evidence -> Verification.

Preserve Temporal-native identifiers such as workflowId and runId. Business identity and runtime execution identity are distinct.

Normalized lifecycle:

REQUESTED -> CLASSIFIED -> AUTHORIZED -> ADMITTED -> EXECUTING -> OBSERVED -> VERIFIED

Alternative states:

DENIED, REQUIRES_APPROVAL, APPROVAL_EXPIRED, FAILED, CANCELLED, UNKNOWN, EVIDENCE_INCOMPLETE, CONFLICTING.

Frozen invariants:

1. UNKNOWN never becomes VERIFIED without new evidence.
2. Temporal completion never automatically becomes VERIFIED.
3. Authorization is not execution.
4. Execution is not verification.
5. Runtime evidence is not automatically outcome evidence.
6. Historical policy decisions retain their original policy version.
7. Evidence is append-oriented; corrections create additional evidence.
8. Duplicate events cannot create duplicate governance transitions.
9. ArgOS cannot claim control over an execution path it does not control.

## 6. Evidence and idempotency

Every material governance transition produces evidence.

REQUESTED -> intent evidence.

AUTHORIZED, DENIED, or REQUIRE_APPROVAL -> decision evidence.

APPROVED -> approval evidence.

ADMITTED -> admission evidence.

EXECUTING -> runtime execution evidence.

COMPLETED or FAILED -> runtime outcome evidence.

VERIFIED, UNVERIFIED, or CONFLICTING -> verification evidence.

Every evidence record preserves source, timestamp, correlation ID, source reference, integrity information, and ingestion time.

Every ingestible event requires a deterministic deduplication identity. Prefer runtime + source reference + event identity + execution identity. If the runtime lacks a sufficient event ID, construct a deterministic fallback from immutable identifying fields.

Evidence corruption must be detectable.

## 7. Verification contract

V1 explicitly distinguishes:

Authorization: Was this action permitted?

Execution: Did the runtime execute it?

Evidence: What evidence exists?

Verification: Does the available evidence satisfy the defined verification contract?

Verification compares:

Intended Action <-> Authorized Action <-> Observed Execution <-> Outcome Evidence

A workflow can succeed while verification remains incomplete.

The system must refuse VERIFIED when required outcome evidence is absent, conflicting, invalid, or otherwise insufficient.

## 8. Demonstrator and falsification

V1 requires one real Temporal workflow performing a consequential business-style action. The business consequence may be simulated externally, but the Temporal execution must be real. Fabricated Temporal events cannot be used to claim integration.

Five required cases:

1. Authorized successful execution: valid authority, policy ALLOW, real Temporal execution, runtime evidence, outcome evidence, VERIFIED.
2. Governance denial: authority or policy fails, DENY persists, no execution through the governed path.
3. Runtime failure: governance allows, Temporal fails, failure evidence persists, result is not VERIFIED.
4. Runtime success with incomplete outcome evidence: Temporal succeeds but required outcome evidence is absent, result is EVIDENCE_INCOMPLETE or UNVERIFIED.
5. Duplicate/out-of-order evidence: delivery is duplicated or reordered, final state remains deterministic and coherent.

The thesis is falsified if:

- governance cannot reliably correlate to the Temporal execution;
- unrelated and authorized executions cannot be distinguished;
- duplicate/out-of-order evidence creates materially incorrect state;
- runtime success is represented as real-world verification without evidence;
- governance evidence can be silently altered;
- the chain cannot be reconstructed from persisted evidence;
- ArgOS must duplicate Temporal's core durable-execution responsibilities;
- the governance information adds no meaningful distinction for the chosen use case;
- prospective technical/governance users identify no concrete problem worth solving.

Technical success is not commercial validation.

## 9. Minimum architecture and API

Use a modular monolith unless evidence justifies a service split.

Required logical modules:

1. Governance API
2. Identity and Authority
3. Policy
4. Temporal Adapter
5. Correlation
6. Evidence Store
7. Verification
8. Investigation API/UI
9. Audit

Logical API operations:

- POST /governance/actions
- POST /governance/actions/{actionId}/evaluate
- POST /governance/actions/{actionId}/approve
- POST /governance/actions/{actionId}/admit
- POST /runtimes/temporal/events
- GET /governance/actions/{actionId}
- GET /governance/actions/{actionId}/evidence
- POST /governance/actions/{actionId}/verify
- GET /health
- GET /readiness

Exact API technology may evolve. These semantic operations are frozen.

PostgreSQL is the preferred initial datastore for ArgOS governance state and evidence. It is not the source of truth for Temporal execution.

## 10. Minimum investigation view

One primary investigation view is required.

For each governed action it must expose:

- requested action
- principal
- authority
- risk
- policy and policy version
- governance decision
- approval, if required
- Temporal workflow ID
- Temporal run ID
- runtime events
- outcome evidence
- verification state
- evidence completeness
- conflicts/anomalies

This is not intended to replace the Temporal UI.

## 11. Security and test contract

Security invariants:

1. Identity cannot be silently substituted.
2. Expired authority cannot satisfy current authorization.
3. Approval is bound to the governed action.
4. Historical policy versions are retained.
5. Evidence preserves source identity.
6. Duplicate evidence cannot duplicate governance transitions.
7. Unknown state cannot become success by inference.
8. Runtime success cannot automatically produce verification.
9. Evidence corruption must be detectable.
10. Secrets are not unnecessarily persisted.
11. Organizational boundaries are explicit.
12. ArgOS cannot claim control over an uncontrolled path.

Automated tests are mandatory for:

- valid and invalid authorization
- policy denial
- required and expired approval
- successful and failed Temporal execution
- missing outcome evidence
- duplicate and out-of-order events
- conflicting evidence
- correlation preservation
- evidence integrity
- historical policy preservation
- unknown runtime state
- verification refusal when evidence is insufficient

Three mandatory proofs:

1. Temporal workflow completion does not automatically produce VERIFIED.
2. Duplicate runtime evidence does not create duplicate governance state.
3. correlationId survives the complete governance-to-runtime evidence chain.

## 12. Acceptance gates and Definition of Done

Gate A — Governance Identity:
A consequential action, principal, correlation ID, business identity, and runtime identity are established.

Gate B — Governance Decision:
Authority and policy are evaluated and the decision plus policy version are persisted.

Gate C — Temporal Correlation:
A real Temporal execution occurs; Temporal identifiers are captured; runtime evidence reaches ArgOS and is associated with the governed action.

Gate D — Evidence Integrity:
Evidence is durable, deduplicated, provenance-preserving, and historically reconstructable.

Gate E — Verification:
Authorization, execution, and outcome evidence remain distinct; insufficient evidence cannot yield VERIFIED; at least one successful end-to-end verification exists.

Gate F — Falsification:
The technical wedge is demonstrated sufficiently to expose the commercial question.

V1 is DONE when:

- a real Temporal workflow is integrated;
- a consequential action can be registered;
- principal and authority can be established;
- policy can be evaluated;
- the governance decision is persisted;
- Temporal execution is correlated;
- runtime evidence is persisted;
- duplicate/out-of-order evidence is handled;
- outcome evidence is representable;
- verification distinguishes verified from merely completed;
- all five demonstration cases pass;
- the evidence chain can be reconstructed;
- critical invariants have automated tests;
- the demonstrator can be shown externally;
- no unsupported enforcement or verification claim is presented as fact.

## 13. Failure budget

V1 may fail at:

- broad Temporal feature coverage
- Akka support
- enterprise-scale deployment
- universal enforcement
- multi-region operation
- full compliance automation
- generalized AI governance
- complete commercial packaging

V1 may not fail at:

- correlation integrity
- evidence persistence
- governance decision integrity
- runtime/evidence distinction
- verification semantics
- duplicate-event handling
- enforcement-boundary honesty
- reproducibility of the demonstrated chain

## 14. Commercial handoff

After technical completion, the next activity is customer discovery, not architecture expansion.

Frozen external question:

“You operate Temporal-based workflows. Would a governance layer that establishes identity, authority, policy, evidence, correlation, and verification around consequential execution solve a problem you currently have?”

Possible evidence outcomes:

- BUY: recognized problem and willingness to pay.
- PILOT: recognized problem requiring deployment validation.
- INTEREST: interesting concept but unclear economic pain.
- NO VALUE: governance information does not justify another system.
- WRONG WEDGE: the problem exists but ArgOS is addressing the wrong layer.

If the answer is NO VALUE, revise the thesis rather than automatically expanding the architecture.

## 15. Frozen principles

1. ArgOS is a governance control plane, not a durable-execution runtime.
2. Temporal remains the execution substrate.
3. Consequential execution is the target unit of governance.
4. Authorization is not execution.
5. Execution is not verification.
6. Runtime evidence is not automatically outcome evidence.
7. Observability is not enforcement.
8. Enforcement must be proven for the specific path.
9. Correlation is a first-class invariant.
10. Evidence is a first-class architectural object.
11. Historical governance decisions remain reconstructable.
12. Unknown remains unknown until evidence resolves it.
13. Read-only integration precedes enforced integration.
14. Runtime-neutral governance must not erase runtime-specific semantics.
15. V1 optimizes for falsifiability rather than feature breadth.
16. Technical success does not prove commercial demand.
17. No architectural expansion occurs merely because an unproven capability appears desirable.

## 16. Final frozen statement

SHIP the wedge.
FREEZE the boundary.
BUILD the chain.
VERIFY the evidence.
SHOW the result.
Let reality determine what comes next.
