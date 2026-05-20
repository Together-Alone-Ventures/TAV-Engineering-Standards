# TAV Gate-Doc Pattern

Status: Playbook pattern.
Authority: TAV Engineering Standards pattern for docs-only gate artifacts that cite or adopt it.
Scope: required structure, content rules, hard exclusions, acceptance criteria, and process invariants for gate documents that decide whether or how a future implementation slice may open. Promoted from the TinyPressZD slice chain S7-44 through S7-49. Does not govern implementation slices or non-gating documents.

## When this pattern applies

A gate doc is a docs-only artifact whose purpose is to decide one or more of:

* whether a future implementation slice should open;
* which candidate approach a future slice should take;
* what dependencies must close before that slice runs;
* whether to reaffirm or depart from a prior gate's recommendation.

A gate doc does not authorize implementation, does not commit code, and does not commit host-side
or upstream protocol rulings.

## Required sections

### Authority header

Form: `Authority: advisory implementation planning only; source rulings remain in [list]`. The list contains only docs verified to exist at commit time; existence is checked via `ls` for each cited path. Ruling weight is deferred to source-ruling docs and prior ratified gates. The gate doc itself provides advisory analysis and sequencing posture.

### Baseline

State current repo facts relevant to the question being gated, including current commit baseline or referenced prior gate. File references use inline code, for example `src/lib.rs`, not markdown links with absolute or line-anchored local filesystem paths.

### Candidate approaches

Candidates are ordered ascending by runtime or scope surface area: smallest surface first, largest last. Each candidate is one paragraph following the structure: what / what it exposes / advantage / risk. No candidate is pre-designated as leading; the recommendation is the output of the comparison, not its input.

### Orthogonal modifiers

A behavior that modifies candidate choices, such as an idempotency posture or already-resolved lookup posture, is recorded in its own section explicitly labeled `Orthogonal modifier` or equivalent. The section identifies which peer candidates it could combine with. Modifiers are not mixed into the candidate list.

### Required analysis points

Each analysis point is bounded to one short paragraph. For every point, the gate must either answer from current repo state, prior gate rulings, or source-ruling docs, or explicitly flag the point as dependent on an unresolved upstream artifact, such as a host API spec, authorization model, or open polish item. The gate identifies what would have to land first and does not invent host-side or upstream-spec answers.

### Per-candidate dependency mapping

Include one short subsection per candidate identifying which open questions each candidate depends on, such as authorization model, idempotency rule, atomicity rule, receipt-runtime decision, canonicalization or PII clarification, host API stability, production provenance, or module_hash posture. A candidate that depends on unresolved upstream artifacts is flagged accordingly, not silently treated as available.

### Recommendation

If a prior gate's posture exists on the same question, the recommendation explicitly states whether it reaffirms or departs from that posture. A departure must name what changed in the analysis and what safety-valve conditions apply. The recommendation explicitly disclaims implementation authorization.

### Hard exclusions for the next slice

The gate enumerates what its recommended next slice must not introduce. The standing exclusion list below applies; domain-specific exclusions may be added for the particular slice family.

### Not-yet-authorized list

The gate enumerates runtime postures and surfaces uncovered by its candidate analysis that remain unauthorized. This list is more specific than the standing exclusions: it reflects what the particular candidate set surfaces.

### Follow-on process requirement

The gate states that any implementation slice that follows must have its own drafted prompt, bounded scope, full hard-exclusion list, C adversarial review, and explicit G ratification before any Codex execution. The gate recommends only sequencing and dependency closure; it does not authorize implementation by itself.

## Standing hard exclusions in any gate doc

* No implementation code.
* No `.did`, frontend, public-method, or canister-API changes unless the gate is explicitly about designing a future interface gate.
* No CLI/test harness or backend/internal diagnostic implementation.
* No observation-surface expansion or full receipt payload output where applicable to the slice family.
* No host-protocol edits, such as MKTd03 edits from a TinyPressZD-side gate.
* No commitment of any host-side ruling that has not been validated against the consuming protocol's actual host API contract; host-side questions are surfaced as dependencies, not asserted.
* No data-model pre-design: the gate identifies what facts or state would need to be captured but does not specify field sets, record shapes, or storage forms.
* No absolute filesystem paths or path-with-line-anchor markdown links that bake in local-machine paths.
* No production authorization, provenance, or module_hash rulings beyond identifying what remains unresolved.

## Standing acceptance criteria

* Docs-only scope; exactly one deliverable file per gate slice unless C/G explicitly amend.
* Every cited source-ruling doc is verified to exist before commit.
* Specific content claims about a source-ruling doc are grep-verified before commit.
* C adversarial review before commit.
* G ratification before any Codex editing prompt and before commit.
* Codex pauses for explicit approval before any `git add` or `git commit`.
* Pre-commit grep for absolute filesystem paths and stale slice labels returns empty.

## Standing process invariants

* Default sequence: C draft or review checklist -> G review -> approved prompt -> Codex execution -> C adversarial review -> G ratification -> Stef commit.
* No Codex prompt is issued without G approval.
* Use per-step commits with clean messages.
* Avoid `!` in commit messages because of WSL bash history expansion.
* Wrap `git commit -m` arguments in single quotes.
* Keep the slice to one deliverable file unless the slice scope is explicitly broadened by G.

## Promotion provenance

This pattern was promoted from the TinyPressZD gate chain:

* S7-44 - local demo CVDR observation surface.
* S7-45 - developer CVDR observation path gating.
* S7-47 - developer CVDR observation surface gate.
* S7-48 - `delete_profile` to MKTd03 integration gate.
* S7-49 - `delete_profile` authorization and sequencing gate.

Those slices repeatedly surfaced the same mechanics: authority-header verification, candidate ordering by surface area, equal-footing comparison, depth caps, must-surface-or-flag framing, per-candidate dependency mapping, host-input ruling guard, reaffirm-or-depart framing, orthogonal-modifier treatment, no data-model pre-design, no absolute local paths, and pause-before-commit discipline.

Subsequent gate slices that cite this pattern should reference it rather than re-deriving the structure and acceptance criteria.

Intentionally excluded from P-1:

* implementation-slice pattern;
* full promotion of other queued Playbook lessons;
* detailed Codex prompt format;
* TinyPressZD-specific PII, CVDR, MKTd03, or profile-store rules;
* continuity-file discipline;
* toolchain/version-pin doctrine;
* formal C verdict format.
