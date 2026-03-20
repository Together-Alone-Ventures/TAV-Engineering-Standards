# TAV Design Principles
## How We Design Systems Before We Write a Line of Code

Together Alone Ventures · March 2026 · v3.0

This document captures the design principles TAV applies before implementation begins on any
new project. They are actionable rules, not aspirations.

> **Provenance and status:** This is a TAV design doctrine — a synthesis derived from the MKTd02
> build experience and aligned with contemporary guidance on architecture, API design,
> observability, security, privacy, and change management. It is not a verbatim adoption of any
> single external framework. Where it draws on authoritative external sources, those sources are
> identified in Appendix B. It should be treated as an internal working standard intentionally
> grounded in, but not limited to, what MKTd02 made expensive. It has been reviewed three times,
> including two independent passes by a second AI model (G/ChatGPT).

> **Relationship to the TAV Vibe Coding Playbook:** The Playbook governs how we run sessions,
> manage repos, and release software. This document governs how we design the systems those
> sessions produce. Both must be read at project start. This one comes first.

> **How to use this document:** At project kickoff: read all eleven principles before any
> implementation decisions are made. During design review: run the Must-Pass checklist. Use
> If-Applicable items where relevant. During secondary review: ask the reviewer to check each
> principle against the proposed design. If a principle is intentionally relaxed: document the
> exception and rationale in the ADR.

---

## Why Design Principles Matter Before Code

The most expensive mistakes in MKTd02 were almost never caused by bad code. They were caused
by design decisions that made bad outcomes invisible: a field silently dropped at a serialisation
boundary; a type mismatch that produced a plausible-looking wrong value rather than an error; a
component whose state could not be queried so debugging required guesswork; a verifier that shared
no structure with the library it was checking so they could drift without warning; trust boundaries
assumed rather than mapped.

Good design does not prevent all bugs. It makes bugs visible, bounded, and recoverable. The eleven
principles below are organised around that goal. Three of them — simplicity, security, and privacy
— are included because modern architecture guidance treats them as first-class design concerns, not
afterthoughts that can be retrofitted once the system is working.

| # | Principle | The core rule | Primary external anchor |
|---|-----------|--------------|------------------------|
| 1 | Composability first | Minimal, explicit interfaces. No reaching inside. | Google WAF: decouple architecture |
| 2 | Simplicity before cleverness | Prefer the simplest design that works. Resist accidental complexity. | Google WAF: simplify your design |
| 3 | Specification before implementation | Interface contract, explicit failure semantics, and ADR exist before coding begins. | Azure API design; AWS schema guidance |
| 4 | Diagnostics and telemetry by design | Every stateful component emits required signals from its first deployment. | Google WAF observability; Google SRE telemetry signals |
| 5 | Fail loud, never fail silent | Ambiguous states produce explicit errors, not plausible-looking defaults. | OWASP Secure Product Design; Azure explicit contract/error guidance |
| 6 | Incremental integration | One new boundary at a time, with a compatibility checkpoint at each. | AWS backward-compatible schema guidance |
| 7 | Observability over convenience | Systems are architected so signals exist at every lifecycle step. | Google SRE; Google WAF reliability pillar |
| 8 | Idempotence and re-runnability | Sensitive flows can be safely re-run or detect prior completion cleanly. | AWS Well-Architected: idempotent operations |
| 9 | Stable identifiers, versioning, and compatibility policy | Every externally meaningful object has a stable ID, explicit version, and documented compatibility and deprecation stance. | Azure API versioning; AWS schema evolution |
| 10 | Security by design | Threat-model trust boundaries, authn/authz model, and misuse cases before implementation begins. | OWASP Secure Product Design; Microsoft SDL |
| 11 | Privacy by design | Define personal-data boundaries, deletion surfaces, and privacy controls before implementation begins. | NISTIR 8062; NIST Privacy Framework |

---

## The Eleven Principles

### 1. Composability First

Each component exposes one minimal, explicit interface and does not depend on internal knowledge
of another component's implementation.

- Define the interface before implementing the internals.
- If the interface cannot be described in one short paragraph, split the component.
- Shared behaviour belongs in a common upstream module, not duplicated or inferred.
- Composable components are easier to scope for implementation tasks, easier to review, and safer
  to refactor.

Note: composability is a structural property (clean interfaces, no hidden dependencies). Simplicity
is a complexity property (fewer states, layers, and artifacts). Both matter; they are addressed as
separate principles.

**In practice:** The zombie-core extraction was smooth because the boundaries were already clean.
The manually-maintained binding file situation was painful because the boundary between generated
and manual files was never explicit.

**Violation signals:**
- "The verifier needs to know how the library works internally"
- "We'll just read the field directly from the other component's state"
- "These two components share a struct that neither fully owns"
- "We can't change this interface without also changing that one"

---

### 2. Simplicity Before Cleverness

Prefer the simplest design that preserves the required properties. Resist introducing extra
lifecycle states, abstraction layers, intermediate artifacts, and configuration surfaces unless
they buy clear, demonstrable operational value.

- For any proposed layer or abstraction: ask "what failure mode does this prevent, or what
  operation does this enable, that we cannot achieve without it?" If there is no good answer,
  do not add it.
- Complexity accumulates quietly. Each addition seems justified in isolation; the aggregate
  becomes unmaintainable.
- A system can be highly composable and still far too elaborate. Composability and simplicity
  are different virtues.
- The simplest design that passes all required tests and meets all required properties is the
  right starting point. Sophistication is added only when justified by evidence, not by
  anticipation.

**In practice:** Several MKTd02 debugging sessions were expensive not because the logic was wrong
but because the number of moving parts made the failure surface wide. Each layer was justified
individually; the aggregate was harder to operate than it needed to be.

**Violation signals:**
- "We'll add an abstraction layer so we can swap it out later"
- "This makes it more flexible"
- "We might need this later"
- "It's just one more config parameter"
- "The complexity is all encapsulated, it won't leak"

---

### 3. Specification Before Implementation

The interface contract — what the component accepts, returns, what invariants it maintains, what
its error semantics are, and how it behaves on unsupported or stale input — must exist and be
reviewed before any implementation begins.

For this project family, that means:
- Write the interface definition file (.did or equivalent) first.
- Define explicit failure semantics — mismatch, retry, duplicate, stale-artifact, and
  unsupported-version behaviour — not just success shapes.
- For any cryptographic or protocol operation: write the golden test vectors (known inputs to
  known expected outputs) before writing the implementation.
- Let the implementation follow the spec, not the other way around.

**Required: Architecture Decision Record (ADR)**

Before implementation, write down the key assumptions, invariants, compatibility stance, and
rejected alternatives for each significant design decision. This is especially important when
multiple AI models and human operators collaborate, because decisions that seem obvious in one
session become invisible three sessions later. The ADR does not need to be long — a few sentences
per decision, committed to the repo, is sufficient.

**In practice:** The record_id empty-string bug was a spec-that-was-never-written bug. The type
mismatch (Vec<u8> vs Option<String>) was a failure-semantics-that-were-never-specified bug. Both
would have been caught if the interface definition file had been written and reviewed before any
Rust was written.

**Violation signals:**
- "We'll define the exact interface once we see what the code needs"
- "The interface file will be generated from the implementation"
- "We'll add test vectors after the function is working"
- "The spec is the code"
- "We'll document why we made this decision later"
- "It returns an error if something goes wrong" (without specifying what, when, or in what form)

---

### 4. Diagnostics and Telemetry by Design

Every stateful component must expose, from its first deployed version:

**Queryable state (diagnostics):**
- A health/status query returning current internal state in human-readable form.
- An explicit version or schema-version field in every significant response type.
- Enough queryable state to determine where in its lifecycle the component currently is, without
  reading source code.

**Runtime signals (telemetry) — expose where operationally meaningful:**
- Operation counts and error counts.
- Pending / in-progress / completed state for multi-phase flows.
- Resource consumption indicators (cycle balance, storage growth) where relevant.
- Build identity or deployment hash so the deployed version is always attributable.

Diagnostic and telemetry endpoints should be read-only and exposed according to the project's
threat and privacy model. A component whose state cannot be queried requires guesswork to debug.
Guesswork is expensive.

**In practice:** mktd_get_tombstone_status, mktd_receipt_count, and mktd_is_pending were
invaluable during MKTd02 — but were added reactively. All should have been specified before
first deployment.

**Violation signals:**
- "We can check the state by looking at the logs"
- "There's no status query yet, but we can add one later"
- "The version is in the source code, not in the runtime response"
- "We'll know it's working because the happy path works"
- "Queries add overhead; we'll keep it lean"

---

### 5. Fail Loud, Never Fail Silent

Any design pattern that can fail silently must be rejected at design time in favour of an explicit
error or an observable failure state.

**Silent failures that must be designed out:**
- Serialisation field drops (field present in producer, absent from interface definition,
  silently dropped on the wire).
- Type coercions producing plausible-looking wrong values instead of errors.
- Ambiguous defaults that conflate "unknown" with a valid value.
- Stale artifacts that pass a format check but carry old or outdated data.
- Unsupported protocol or schema versions processed silently rather than rejected with an
  explicit error.
- Partial-upgrade states that are not detectable from the component's queryable state.

The corollary for AI-assisted work: if a task can produce a plausible-looking wrong result rather
than an obvious error, build the verification step into the workflow by design — before the task
is run, not after something looks wrong.

**In practice:** Six of the nine principle-mapped MKTd02 mistakes in Appendix A were silent
failures. None of them needed to be silent.

**Violation signals:**
- "If the field is missing it just defaults to empty string / zero / None"
- "Old versions will probably still work"
- "The error only shows up if you know to look for it"
- "We'll validate that in the integration test"
- "It's fine for dev, we'll fix the zeros before production"

---

### 6. Incremental Integration — One Boundary at a Time

Never introduce two unverified integration boundaries in one step. Verify each producer/consumer
boundary in isolation, with a compatibility checkpoint, before composing it with the next.

**What counts as a boundary:**
- Component A calling component B for the first time.
- A library consumed by an application for the first time.
- A verifier checking library output for the first time.
- A new field crossing an interface definition.
- A new dependency pin pointing to a new upstream commit or schema version.

**Compatibility checkpoint required at each boundary before proceeding:**
- Wire-format check: encode/decode round trip passes.
- Live downstream assertion: query the consumer endpoint and confirm the field is present and
  correct.
- Version/schema check: consumer correctly identifies the version of the upstream artifact.
- Dependency reference confirmed: the pin or tag has taken effect in the downstream build.

**Violation signals:**
- "We'll test the whole stack once everything is wired up"
- "Both changes are small, we can do them together"
- "The upstream isn't tagged yet but we can write the downstream code now"
- "It should just work once we put it all together"

---

### 7. Observability Over Convenience

When choosing between a design that is simpler to build but whose runtime state is opaque, and
one that is slightly more verbose but whose state is queryable and inspectable at each lifecycle
step, choose the latter.

- Every state machine node should have a query that returns the current state without side effects.
- Every state transition should leave observable evidence attributable to the transition, not just
  to the resulting state.
- The component's behaviour at any point in its lifecycle should be explainable from its queryable
  state alone, without reading source code or logs.
- Integration assertions (live downstream queries after deployment) are only possible if the
  component makes its state observable. Design for them explicitly.

**Violation signals:**
- "You can infer the state from the behaviour"
- "We don't need a status query, the happy path is obvious"
- "The state is internal — we'll expose it if we need to debug"
- "We'll know it's working because the happy path works"

---

### 8. Idempotence and Re-runnability

For operationally sensitive flows, design steps so they can be safely re-run, or can at least
detect prior completion cleanly and exit without side effects.

- Every stateful write should check for prior completion before executing.
- Every deployment step should be documented as one of: "safe to re-run", "not safe to re-run
  — preconditions required", or "idempotent after first success".
- Recovery posture (the rollback/re-run question in the release checklist) should be answerable
  from the component design, not invented at release time.
- Partial-progress states are normal: network failures, interrupted sessions, resource
  constraints. Design for them at the start, not when they occur.

**Violation signals:**
- "Just don't run it twice"
- "If it fails halfway through, we'll sort it out then"
- "Re-running would require manual cleanup"
- "We haven't thought about partial failure yet"

---

### 9. Stable Identifiers, Explicit Versioning, and Compatibility Policy

Every externally meaningful object must have:
- A stable identifier that does not change when internal implementation changes.
- An explicit protocol or schema version visible in every response, not only in source code.
- A documented compatibility and deprecation policy, decided before implementation of any new
  version begins.

**The compatibility policy must answer:**
- What changes are backward-compatible?
- What changes break verification or downstream consumers?
- How is an unsupported version reported — explicitly, not silently?
- Can old artifacts remain valid after an upgrade? If not, what is the migration path?

**The deprecation policy must also answer:**
- How long will a deprecated version continue to be supported after a successor is released?
- What is the end-of-support signal — how will consumers know?
- What constitutes an unsupported version and what happens when one is encountered?

**Violation signals:**
- "We'll add versioning when we need it"
- "The version is obvious from context"
- "Old artifacts probably won't need to be processed after the upgrade"
- "IDs can be regenerated if needed"
- "We'll decide how long to support the old version once the new one is ready"

---

### 10. Security by Design

Threat-model trust boundaries, authentication and authorisation model, privileged actions,
attacker goals, and misuse cases before implementation begins. Security is a design-time
activity, not a review-time activity.

**Before implementation, document:**
- Trust boundaries: who is trusted, who is not, and what the boundary crossing looks like.
- Authentication and authorisation model: for each privileged operation, what is the required
  authentication mechanism and what authority model governs it?
- Privileged operations: which actions require elevated authority, and what happens if that
  authority is abused or misconfigured?
- Attacker goals: what would an attacker want to achieve, and does the design prevent it?
- Misuse cases: what can a legitimate but mistaken caller cause, and how does the system respond?
- Replay and stale-artifact risks: can an old artifact be replayed to produce a false positive?

**For TAV-style systems, specific threat surfaces include:**
- Component update authority: who can push a new version, authenticated how?
- Verifier trust assumptions: what must the verifier trust, and what can it independently verify?
- Artifact authenticity: how does a consumer know an artifact was produced by the right code?
- Proof/receipt replay: can a valid receipt be replayed in a different context?
- Admin call surfaces: are admin-only endpoints protected by an explicit and reviewed access model?

**Violation signals:**
- "We'll review security once the happy path works"
- "This admin path is obvious — we know who can call it"
- "Security is handled at the platform level"
- "Only we can call that endpoint"
- "We haven't defined the auth model yet but it'll be straightforward"

---

### 11. Privacy by Design

Define personal-data boundaries, deletion surfaces, disclosure surfaces, and privacy-risk controls
before implementation begins. For TAV, this is not a legal formality — it is the product.

**Before implementation, document:**
- What data is personal data in this context?
- Where is it stored, and what is the complete deletion surface?
- What can be queried or disclosed without privacy leakage?
- What evidence proves privacy behaviour (deletion, tombstoning, receipt generation) without
  revealing unnecessary personal data in the process?
- What residual data remains after deletion, and is that acceptable?

**NIST privacy engineering objectives applied to TAV systems:**
- Predictability: stakeholders can reliably anticipate what happens to personal data at each
  system step.
- Manageability: the data controller can enforce privacy policies programmatically, not just
  procedurally.
- Disassociability: the system can operate with the minimum personal data necessary for each
  function.

**Violation signals:**
- "We'll handle privacy compliance at the end"
- "GDPR is a legal concern, not an engineering concern"
- "The data is pseudonymous so it's fine"
- "We'll figure out the deletion boundary later"
- "The receipt proves deletion — we don't need to design the deletion surface explicitly"

---

## Design-Time Non-Functional Requirements

| NFR | Question to answer at design time | Why it matters |
|-----|----------------------------------|----------------|
| Verification / operation latency | What is the acceptable end-to-end time for each significant operation? | Drives choice of synchronous vs. asynchronous design |
| Acceptable cost / resource burn | What is the per-operation resource cost budget? | Prevents designs that are correct but operationally unaffordable |
| Storage growth model | How does storage grow with usage, and what are the retention constraints? | Persistent storage is finite; must be designed for |
| Recovery posture | For each sensitive flow: safe to re-run? Requires precondition? Can roll back? | Determines incident response options |
| Deployment and update constraints | What is the component update model, and who holds update authority? | Determines upgrade safety |
| Backward-compatibility horizon | For how long must old artifacts remain processable? | Drives versioning and deprecation policy |
| Verification independence | What must the verifier trust, and what can it independently verify? | Core to product value |

NFR documentation does not need to be elaborate. A table with one-sentence answers per row is
sufficient for most projects at design time.

---

## Design Review Checklist

### Must-Pass — Required Before Coding Begins

| # | Check | Principle(s) |
|---|-------|-------------|
| M1 | Is the interface definition file (or equivalent) written and reviewed? | 3 — Spec first |
| M2 | Are failure semantics (mismatch, stale, unsupported version, retry) explicitly defined? | 3, 5 |
| M3 | Is an ADR committed for each significant design decision? | 3 — Spec first |
| M4 | Have we identified every way this design can fail silently and rejected each? | 5 — Fail loud |
| M5 | Have trust boundaries, authentication/authorisation model, and privileged operations been documented? | 10 — Security |
| M6 | Is the personal-data boundary and deletion surface explicitly documented? | 11 — Privacy |
| M7 | Have all seven design-time NFRs been given explicit answers? | NFR section |
| M8 | Is the backward-compatibility stance documented before implementation of any new version begins? | 9 — Versioning |

### If-Applicable — Review Where Relevant

| # | Check | Principle(s) |
|---|-------|-------------|
| A1 | Can each component's interface be described in one short paragraph? | 1 — Composability |
| A2 | Has each proposed abstraction layer been justified by a concrete operational benefit? | 2 — Simplicity |
| A3 | Are golden test vectors defined for every cryptographic or protocol formula? | 3 — Spec first |
| A4 | Does every stateful component have a health/status query and schema-version field? | 4 — Diagnostics |
| A5 | Are runtime telemetry signals identified? | 4 — Diagnostics + telemetry |
| A6 | Are type boundaries between components explicit, producing errors on mismatch? | 5 — Fail loud |
| A7 | Is each new integration boundary being verified in isolation? | 6 — Incremental integration |
| A8 | Can the state of each component be queried at every lifecycle step without side effects? | 7 — Observability |
| A9 | Is each operationally sensitive step documented as idempotent or not safe to re-run? | 8 — Idempotence |
| A10 | Does every externally meaningful object have a stable ID and explicit version? | 9 — Stable IDs |
| A11 | Is the deprecation policy documented? | 9 — Versioning |
| A12 | Have replay, stale-artifact, and fake-proof risks been explicitly considered? | 10 — Security |

---

## Appendix A — MKTd02 Violations by Principle

| MKTd02 mistake | Principle(s) violated | Design-time prevention |
|----------------|----------------------|----------------------|
| record_id empty-string — field silently dropped on the wire | 3 + 5 | Write the interface file before implementation |
| Type mismatch (Vec<u8> vs Option<String>) — silent None instead of error | 3 + 5 | Define wire types in the spec before coding either side |
| Endianness error in verifier — verification mismatch, no obvious error | 3 | Spec must state encoding convention; golden vector enforces it |
| Verifier pinned to old upstream schema — caught only at compile time | 6 | Verify upstream schema boundary before writing downstream code |
| Stale dependency lock misdiagnosed as API change | 5 + 3 | Dependency version is part of the effective spec |
| Artefact cache serving old build silently | 5 | Cache-bypass must be the default deploy command |
| Commits landed on wrong branch — downstream never picked them up | 6 + 7 | Branch state is observable; verify after every push |
| Incomplete restart pack — next session re-established already-resolved context | 4 | Project state is a queryable artifact; required at every session close |
| V2/V3 version-dispatch added reactively | 9 + 3 | Compatibility policy decided before implementing a new version |

---

## Appendix B — Authoritative External References

| Source | Relevance |
|--------|-----------|
| Google Cloud Well-Architected Framework | Simplify your design, decouple architecture, observability as reliability requirement |
| AWS Well-Architected Framework | Idempotent operations, backward-compatible schema changes, versioning |
| Azure Well-Architected / API Design Guidance | Explicit API contracts, loose coupling, independent evolution |
| Google SRE Book — Monitoring Distributed Systems | Four golden signals baseline for telemetry |
| OWASP Secure Product Design | Threat modeling, security stories, misuse-case analysis |
| Microsoft Security Development Lifecycle (SDL) | Structured threat modeling before coding begins |
| NISTIR 8062 | Privacy engineering objectives: predictability, manageability, disassociability |
| NIST Privacy Framework v1.0 | Systematic treatment of privacy risk and data-processing inventory |

*Last checked: March 2026. Confirm references have not had major revisions before applying to a new project.*

---

Together Alone Ventures · TAV Design Principles · March 2026 · v3.0
Three review rounds (Claude + G x2)
