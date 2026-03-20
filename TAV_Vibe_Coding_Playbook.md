# TAV Vibe Coding Playbook
## The Operating Guide for AI-Assisted Protocol Development

Together Alone Ventures · March 2026 · v4 (final)

This is the single operating guide for any TAV project involving AI-assisted development. It was
built by merging two independent post-mortems of MKTd02, refined through four rounds of review.
It is written to be used under pressure, not admired — keep it open during every session.

> **How to use this document:**
> - Before every session: §§3 (restart pack) + §10 (session checklists)
> - Before every implementation task: §§4–6 (access, Codex, review)
> - Before every release: §9 (release checklist)
> - For a quick fix or typo: §12 (lightweight path)
> - When something goes wrong: Appendix A (most expensive mistakes from MKTd02)
> - For companion design guidance: TAV Design Principles (separate document)

> **Roles used throughout this document:**
> - Primary model — the AI used for day-to-day implementation assistance (currently Claude)
> - Secondary reviewer — a second AI used for adversarial review (currently G / ChatGPT)
> - Human operator — runs CLI commands, makes commits, signs off on releases
> - Repo ground truth — what the CLI and build tools report; always wins over any model's recollection

---

## 1. What MKTd02 Taught Us — In One Page

MKTd02 reached its core protocol objectives, but operational friction increased cost, latency,
and confidence risk throughout. The biggest sources of that friction were not algorithmic — they
were coordination failures.

| Pain point | What actually happened | Recovery cost |
|------------|----------------------|---------------|
| Chat continuity | Each new chat started partially blind; context re-established from scratch | High |
| Source-of-truth drift | Older plan docs and current carry-forwards disagreed; wrong instructions applied | High |
| Repo visibility | No live repo access; grep/paste became the bridge; stale pastes caused incorrect patches | High |
| Review gaps | Primary model reviewed code it had just written; some issues caught only by secondary review | Medium |
| Codex scope | Broad prompts led to implicit architectural decisions made silently | Medium |
| Release complexity | Docs, tags, pins, verifier, and guide artifacts had to line up in the right order | Medium |

**Bottom line:** Run projects less like an ongoing conversation, more like a protocol program with
a standing ops handbook. One authoritative restart pack. Direct repo access from day 0. Bounded
tool rhythm. A repeatable review ladder. Evidence-first release gating.

---

## 2. The Three Source-of-Truth Layers

| Layer | What belongs here | Where it lives | Who owns it |
|-------|------------------|----------------|-------------|
| Protocol truth | Schema, field definitions, hash formulas, byte order, domain tags, backward-compatibility rules | Upstream core repo + protocol spec note | Primary model + secondary reviewer |
| Integration truth | How the library or reference app implements the protocol in its specific context | Repo-local docs | Primary model + human operator |
| Release truth | Tags, pinned refs, guide artifacts, verifier test results, release provenance file | Release checklist (§9) | Human operator signs off |

- **Rule:** Do not update polished docs first. Prove the code path first, then update docs from
  the proven state.
- **Rule:** Never update docs until code, tests, and pinned dependencies are stable.
- **Rule:** When any two documents conflict, the restart pack (§3) wins over older plan docs.
  When restart pack and code conflict, repo ground truth (CLI output) wins.
- **Rule:** Never use integration behaviour to infer protocol truth.

---

## 2.5 Design Principles — The Summary

Before any implementation begins, the system design must be reviewed against TAV's eleven design
principles. These principles are documented in full in TAV_Design_Principles.md.

> **See also:** TAV_Design_Principles.md — full rationale, violation signals, examples, and a
> design review checklist for each principle.

| # | Principle | Operational rule for this playbook |
|---|-----------|-------------------------------------|
| 1 | Composability first | If the interface cannot be described in one short paragraph, split the component before coding begins. |
| 2 | Specification before implementation | Write the interface definition file (.did or equivalent) and any golden test vectors before writing implementation code. |
| 3 | Diagnostics by design | Every stateful component must expose a health/status query and a schema-version field from its first deployment. Non-negotiable. |
| 4 | Fail loud, never fail silent | Reject any design where a failure produces a plausible-looking wrong value rather than an explicit error. |
| 5 | Incremental integration | Never introduce two unverified integration boundaries in one step. |
| 6 | Observability over convenience | Prefer designs whose state is queryable at every lifecycle step. |
| 7 | Idempotence and re-runnability | Sensitive flows must be either safe to re-run, or explicitly documented as not idempotent. |
| 8 | Stable identifiers and explicit versioning | Every externally meaningful object has a stable ID, an explicit version in every response, and a documented backward-compatibility stance. |

> **Design review gate:** No implementation work begins until the design has been reviewed against
> all principles. This review is a session-opening task, not an afterthought. Use the Design
> Review Checklist in TAV_Design_Principles.md.

---

## 3. State Continuity — Restart Pack and Continuity Log

### 3.1 The Restart Pack (RESTART_PACK.md)

> **Key principle:** There is exactly one restart pack at any point in time. It is overwritten at
> each phase boundary or chat reset — not accumulated into a series of variants.

```
RESTART_PACK.md TEMPLATE

DATE:           [date of last update]
CURRENT GOAL:   [one sentence — the exact next milestone]

GIT STATE
  [repo-1]:  [branch] @ [full commit hash — use git rev-parse HEAD]
  [repo-2]:  [branch] @ [full commit hash]

FILES OPEN (edited, not yet committed)
  [repo]/[filepath]  --  [what changed, one line]

DECISIONS MADE THIS SESSION
  [decision]  --  [rationale, one sentence]

OPEN QUESTIONS (not yet resolved)
  [question]  --  [last hypothesis]

KNOWN GOTCHAS FOR NEXT SESSION
  [trap or naming convention to watch for]

ACCEPTANCE GATES (what must be true before this phase is done)
  [ ] [gate 1]
  [ ] [gate 2]

SAFE RESTART PROMPT
  [one paragraph that can be pasted verbatim into the next chat]
```

**Important:** Always use the full SHA in GIT STATE. Get it with `git rev-parse HEAD`, not the
short hash from `git log --oneline`.

### 3.2 The Continuity Log (MILESTONE_LOG.md)

The restart pack is overwritten. The continuity log is append-only. It uses two entry types:

**Entry Type 1 — Milestone (at every phase boundary):**
```
## [Date] -- MILESTONE: [phase or milestone name]

Decisions made:
  - [decision]: [rationale, 1-2 sentences]

Irreversible actions taken:
  - [e.g. tagged v0.3.0; existing receipts now depend on this schema]

Do not revisit:
  - [explicitly settled items]
```

**Entry Type 2 — Session Lesson (when a reusable lesson emerged):**
```
## [Date] -- SESSION LESSON: [brief topic label]

What happened:
  - [1-2 sentences: the situation that produced the lesson]

The lesson:
  - [specific, actionable guidance -- not 'be careful' but 'always do X before Y']

Category: [process | tooling | protocol | review | integration]

Apply next time:
  - [what to do differently, concretely]
```

Write a Session Lesson only when a non-obvious mistake, recovery pattern, or reusable tactic
emerged. The test: "Would I want to remember this at the start of a future project?"

- Keep entries terse. No operational clutter.
- Never delete entries. Mark superseded decisions as "[superseded by ...]".
- Commit both files to the repo at every session close.

### 3.3 Explicit Steps: How to Open a New Chat Session

| Step | Action |
|------|--------|
| 1 | Open the new chat. Do NOT describe what you want to do yet. |
| 2 | Claude reads RESTART_PACK.md directly from GitHub and confirms current state. If the connector is unavailable or the file looks stale, Claude flags this explicitly and the operator pastes the file manually as fallback. |
| 3 | Wait for Claude's confirmation. Correct anything wrong before proceeding. Do not proceed if state is ambiguous. |
| 4 | Only then describe the task for this session. |

---

## 4. Repo Access — Two Kinds of Truth

> **The fundamental access distinction:**
> - Connector truth = committed repo history. What GitHub shows.
> - CLI truth = working-tree truth. What your local files actually contain right now.
>
> For active edits, CLI truth wins. The connector does not see uncommitted changes. The CLI
> always does.

### 4.1 Setting Up Connector Access (Do Once on Day 0)

| Step | Action |
|------|--------|
| 1 | In Claude.ai: go to Settings → Integrations. Authorise access to the project's GitHub organisation. |
| 2 | Test immediately with a real file query from a real repo. |
| 3 | If it fails: try a public repo first to confirm OAuth worked, then check private repo permissions. |
| 4 | Repeat the setup in ChatGPT for the secondary reviewer's sessions. |
| 5 | Record "connector access verified" in the restart pack under KNOWN GOTCHAS. |

### 4.2 When Connector Access Is Not Available

- Share files proactively, not reactively.
- Always include the file path and git commit hash when pasting.
- Paste the full file, not just the "relevant section."
- After any edit, paste the full file again before committing.

### 4.3 CLI Evidence That Always Applies

| Situation | Command | What it tells you |
|-----------|---------|-------------------|
| Verify a commit landed on the right branch | `git log --oneline -3 origin/<branch>` | Confirms commit hash is on intended branch |
| Check a dependency update took effect | `cargo metadata --format-version 1 \| grep -A3 '<dep-name>'` | Shows pinned rev or tag |
| Verify a file after editing | `cat <file> \| head -5 && cat <file> \| tail -5` | Confirms write succeeded |
| Deploy with correct artefact (bypass cache) | [tool-specific reinstall command with explicit artefact path] | Guarantees cache is not serving old code |
| Check local vs committed state | `git status && git diff` | Shows working tree vs committed |

---

## 5. Codex — Bounded Operator, Not Roaming Architect

### 5.1 The Five-Step Codex Rhythm

| Step | What happens | Who acts | Output |
|------|-------------|----------|--------|
| 1. Inspect | Report what exists: file locations, current assumptions, existing references. No edits yet. | Codex | Inventory report |
| 2. Review | Review findings. Resolve all semantic questions before any editing begins. | Primary model + operator | Decision: go / no-go + bounded scope |
| 3. Bounded edit | One repo. One task. Explicit file list. Invariants stated. | Codex | Diff or replacement files |
| 4. Secondary review | Review output against spec. For protocol or interface changes: secondary reviewer also checks. | Primary model (+ secondary) | Approval or required fixes |
| 5. Test + commit | Run tests. If green: commit. If not: back to step 3. | Human operator | Committed, tested change |

> **Non-negotiable rule:** Codex proposes → reviewer approves → human operator commits. Codex
> must never commit directly to a real branch without review of the diff first.

### 5.2 Good Codex Tasks vs. Bad Codex Tasks

| Good Codex tasks | Bad Codex tasks |
|-----------------|-----------------|
| Inventory: locate all references to a field or function | "Upgrade the whole stack" |
| Scoped rename across one repo | "Fix all docs and code" |
| Implement a narrowly defined function matching an existing pattern | Decide protocol semantics or version strategy |
| Write a test for a known-good expected value | Update multiple repos in one prompt |
| Sync one manually-maintained interface file | "Bring docs into line" before code is stable |
| Format / lint pass | Refactor + document + test in one shot |

### 5.3 How to Write a Good Codex Prompt

- One repo, one task.
- List every file explicitly.
- State what must NOT change.
- Paste the relevant spec section verbatim.
- Specify the output format: "Complete replacement file" or "unified diff".
- State the acceptance criterion.

### 5.4 The Sandbox Trap

After every Codex session, verify the change landed in the real repo:
```bash
git log --oneline -3 origin/<branch>
```
If the commit Codex reported does not appear there, the change exists only in Codex's sandbox.
Treat the task as incomplete.

---

## 6. Review Discipline — The Four-Rung Ladder

| Rung | Who / what | Purpose | When to apply |
|------|-----------|---------|---------------|
| 1. Primary model self-review | Primary model, fresh turn, against checklist | Catch missing caveats, naming drift, internal contradictions | After every non-trivial change |
| 2. Secondary reviewer targeted review | Secondary reviewer, with specific attack brief | Find what the primary model normalised | Before any release; after any protocol or interface change |
| 3. Human operator diff review | Human operator reads the actual diff | Confirm edit matches intent; spot scope creep | Before every commit |
| 4. Build / test / proof | Test suite + end-to-end verification | Confirm reality, not just wording | Before every release tag |

When primary model and secondary reviewer disagree: treat it as a trigger for evidence review,
not a vote. Repo ground truth wins over model confidence.

### 6.1 Primary Model Self-Review Checklist

- Are all interface definition files updated to match any changed structs?
- Are all manually-maintained binding files updated to match the interface definition files?
- Does every new field in a cross-component call have a matching type in both producer and consumer?
- Are all dependencies pinned to a specific commit rev or release tag?
- Is the restart pack up to date with changes made this session?
- Have any golden test vectors been added or updated?

### 6.2 Secondary Reviewer Brief Template

```
Context: [one paragraph on what changed and why]

Please specifically check for:
1. Protocol invariant violations
2. Field type mismatches across component boundaries
3. Endianness or encoding assumptions not pinned in the spec
4. Backward compatibility breakage
5. Documentation claims stronger than what the code actually proves

The spec section this change implements: [paste verbatim]
```

---

## 7. Repo and Branch Management

### 7.1 Use the Web Editor for Typos Only

Use the web-based editor for single-line typo fixes only. All multi-line edits must be done via
the CLI, where the branch is explicit and the full file can be verified before committing.

### 7.2 Repo Dependency Ordering

| Order | Repo type | Must be done before |
|-------|-----------|---------------------|
| 1st | Upstream core (shared types, primitives) | All downstream consumers |
| 2nd | Library / engine | Reference app, verification tools |
| 3rd | Verification tool | Release tagging |
| 4th | Shared docs / guide source | Guide artefact generation |
| 5th | Reference / integration app | Deployment |
| Last | Release tags (all repos) | Only after all upstream tags exist and CI is green |

### 7.3 Branch Verification — After Every Push

```bash
# 1. Confirm commit hash is on intended branch
git log --oneline -3 origin/<branch-name>

# 2. Confirm dependency pin changed in downstream repo
[dependency metadata command] | grep -A3 '<dep-name>'

# 3. Confirm dependency update is coherent
[build command] && [test command]
```

---

## 8. Interface Files and Protocol Code

### 8.1 The Golden Rule for Interface Changes

Every time a field is added to a type exposed via a component interface:
1. Update the interface definition file in the same commit.
2. Update any manually-maintained binding files in the same commit.
3. Run the build and verify the interface extraction diff is clean.

### 8.2 Manually-Maintained Binding Files

- Never run auto-generation tools on manually-maintained binding files.
- Every commit touching a manually-maintained file must say so explicitly in the commit message.
- Document which files are manually maintained in CONTRIBUTING.md before coding begins.

### 8.3 Cross-Component Type Safety

Two guards required for every field that crosses a component boundary:
- **Round-trip test:** encode in producer, decode in consumer, assert equality.
- **Live integration assertion:** after deployment, query the downstream consumer endpoint and
  assert the field is present and correct.

### 8.4 Protocol Code — Encoding and Golden Vectors

- The protocol spec must state encoding conventions explicitly for every value in every hash
  preimage or wire format.
- Every cryptographic formula must have at least one golden test vector: known inputs to known
  expected output locked in a test.

---

## 9. Release Checklist — Evidence-Gated Completion

### 9.1 Proof and Verification Gates

| Gate | Action | Evidence required |
|------|--------|------------------|
| R1 | Fresh proof run on current version | All verification checks PASS |
| R2 | Regression check on previous supported version | Verification PASS against prior-version artefact |
| R3 | At least one negative / tamper case | Verification correctly reports failure |

### 9.2 Interface and Dependency Gates

| Gate | Action | Evidence required |
|------|--------|------------------|
| R4 | Interface definition files match deployed interfaces | Extraction diff is clean |
| R5 | Manually-maintained binding files consistent | Manual inspection; commit message confirms |
| R6 | All dependencies pinned to specific release tags | No floating branch pins |
| R7 | Live integration assertions pass | Fields present and correct at each cross-component endpoint |

### 9.3 Documentation and Artefact Gates

| Gate | Action | Evidence required |
|------|--------|------------------|
| R8 | Release provenance file updated | Version, commit hash, toolchain versions, artefact SHA-256 |
| R9 | Configuration reference updated | Reflects current state |
| R10 | Generated guide artefact regenerated | Build pipeline ran green; artefact spot-checked |
| R11 | Secondary reviewer full audit complete | All three source-of-truth layers reviewed |

### 9.4 Tagging and Rollback Posture

| Gate | Action | Evidence required |
|------|--------|------------------|
| R12 | Tags cut in dependency order | Each repo tagged after upstream CI is green |
| R13 | Rollback posture documented | What can be safely re-run? What cannot be reversed? |

---

## 10. Session Checklists

### 10.1 Before Every Session

- Claude reads RESTART_PACK.md from GitHub and confirms current state. If unavailable, operator
  pastes the file manually.
- Confirm connector access: fetch one file from the repo directly.
- State the goal for this session in one sentence before any implementation begins.
- If the session involves a design decision: confirm it has been reviewed against the TAV Design
  Principles before touching code.

### 10.2 During Every Session

- One logical change per commit. Five fixes = five commits.
- After each file edit: verify full file contents before committing.
- After each commit: verify the branch using `git log --oneline -3 origin/<branch>`.
- After any dependency update: run build and tests in every downstream repo.
- For any interface change: update interface definition files and binding files in the same commit.
- Run the test suite after every non-trivial change.

### 10.3 Closing Every Session (Mandatory)

> Do not close the chat until this is complete. An incomplete restart pack means the next session
> starts partially blind.

- Update RESTART_PACK.md with current state of every repo touched.
- Record any files edited but not yet committed.
- Record decisions made this session and their rationale.
- Record open questions raised but not resolved.
- Write the safe restart prompt for the next session.
- If this is a phase boundary: append a MILESTONE entry to MILESTONE_LOG.md.
- If a reusable lesson emerged: append a SESSION LESSON entry to MILESTONE_LOG.md.
- Commit both files to the repo.

---

## 11. Instruction Depth Calibration

### 11.1 Always Provide Explicit Step-by-Step Instructions For:

| Topic | Why |
|-------|-----|
| Opening a new chat session (§3.3) | Sequence is easy to shortcut in ways that cause session-long confusion |
| Branch verification after a push | The "Locking 0 packages" trap is invisible without the command |
| Deploying with the explicit artefact path | The artefact cache trap recurred across multiple sessions |
| Web editor warnings | The branch-default trap was hit multiple times |
| Continuity log writing | Often omitted or rushed; always needs a prompt |
| Dependency update verification | Stale-lockfile misdiagnosis was expensive each time |
| Downloading files to WSL | Delete previous version from Downloads first; verify filename has no (1) suffix |

### 11.2 Reduce or Omit Step-by-Step For:

- Basic git operations (add, commit, push, merge)
- Standard deploy / call commands for familiar operations
- Build and test commands
- The three-phase finalization flow (A→B→C)
- Reading and interpreting verification output

### 11.3 General Calibration Principles

- Explicit > implicit, always. When in doubt, provide the command.
- Watch for re-internalisation. When the operator writes commands correctly without prompting,
  dial back to confirmation-only.
- Never assume prior-session instruction carried over.
- Flag architectural novelty explicitly.

---

## 12. The Lightweight Path — What Not to Over-Formalise

| Change type | Use full workflow? | Notes |
|-------------|-------------------|-------|
| Protocol schema, formula, or encoding change | Yes — all rungs | Irreversible; failure is silent |
| Cross-component interface change | Yes — all rungs | Serialisation drops fields silently |
| New production deployment | Yes — all rungs | Cannot be rolled back cleanly |
| Release tagging | Yes — full release checklist | Tags are permanent |
| Dependency pin update | Rungs 1 + 3 + 4 | Must verify the pin took effect |
| Logic change within one component | Rungs 1 + 3 + 4 | Secondary review only if complex |
| Documentation update (after code is proven) | Rung 3 only | Human diff review |
| Build script or CI change | Rung 3 + 4 | Human review + build passes |
| Typo fix in a comment | None — commit directly | One-line change |

**Heuristic:** If the change cannot break an existing released artefact, a component interface,
or a release tag, it does not need the full ladder.

---

## Appendix A — The Ten Most Expensive Mistakes in MKTd02

| # | Mistake | Recovery cost | Prevention |
|---|---------|---------------|------------|
| 1 | Phase 1/2 files committed to wrong branch | High | Verify branch after every push; never use web editor for multi-line changes |
| 2 | record_id empty-string — field silently dropped | High | Update interface file in same commit; run extraction diff |
| 3 | Stale dependency lock misdiagnosed as API change | High | Run dependency update command before concluding API changed |
| 4 | Codex committed to sandbox only | Medium | Always verify git log shows commit on real branch |
| 5 | Type mismatch — silent wrong value | Medium | Round-trip test + live integration assertion |
| 6 | Triple-paste via web editor | Medium | Use CLI for all multi-line edits |
| 7 | Verifier pinned to old upstream schema | Medium | Update dependency pin before writing new code |
| 8 | Encoding error in verifier | Medium | Spec must state encoding explicitly; verify against golden vectors |
| 9 | Artefact cache serving old build | Low (recurring) | Always use explicit artefact path command |
| 10 | Incomplete restart pack | Low (recurring) | Mandatory template; never close without completing continuity log |

---

## Appendix B — WSL File Transfer Quick Reference

| Situation | Method |
|-----------|--------|
| Short file, ASCII content only | Heredoc: `cat > filename.txt << 'EOF' ... EOF` |
| Any file with special characters | Download from Claude → delete old version from Downloads first → `cp /mnt/c/Users/<user>/Downloads/<file> <dest>` |
| Verify file after writing | `cat filename.txt \| head -5 && cat filename.txt \| tail -5` |
| Large files (>100 lines) | Always use download + cp path, not heredoc |

**Important:** Before downloading any file from Claude to copy into WSL, delete the previous
version from the Downloads folder first. Windows silently appends (1), (2) etc to duplicate
filenames, causing the cp command to copy the wrong version.

---

## Appendix C — Dependency Pin Quick Reference

```toml
# During active development (pin to specific commit):
<crate> = { git = "https://github.com/<org>/<repo>", rev = "<commit-sha>" }

# For releases (pin to tag -- required before any release):
<crate> = { git = "https://github.com/<org>/<repo>", tag = "<crate>-vX.Y.Z" }

# After updating: always verify the pin took effect:
# cargo metadata --format-version 1 | grep -A3 '<crate>'
```

---

## Appendix D — Release Provenance File: Required Fields

| Field | Example / source |
|-------|-----------------|
| Version | [product]-vX.Y.Z |
| Commit hash | Full SHA (not abbreviated) |
| Compiler / toolchain version | From `[compiler] --version` |
| Build tool version | From `[build tool] --version` |
| Deployment tool version | From `[deploy tool] --version` |
| Build script used | Relative path from repo root |
| Target platform / triple | e.g. wasm32-unknown-unknown |
| Optimisation / shrink path applied | Yes / No / tool and flags |
| Artefact SHA-256 | `sha256sum <artefact-file>` |
| Deployed component ID(s) | Production IDs this artefact was installed to |

---

Together Alone Ventures · TAV Vibe Coding Playbook · March 2026 · v4 final
