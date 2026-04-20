# MKTd03 Residual Trust Note — Standards Companion

**Status:** Non-normative companion note. Not TAV doctrine.
**Date:** 2026-04-20
**Companion to:** MKTd03 Residual Trust Statement (RST) Evaluation Lens v1 (MKTd03 repo, `docs/analysis/MKTd03_rst_evaluation_lens_v1.md`)
**Governs:** Rhetorical and comparative claim framing about MKTd03 in operator-facing, partner-facing, and external communications.

## 1. Purpose

This note holds the rhetorical and comparative claim framing about MKTd03 that cannot live inside MKTd03's normative specification artefacts. The separation exists because normative protocol artefacts — ADRs, interface files, companion rules, verifier documents — must be strictly technical. They define what MKTd03 proves. Rhetorical and comparative framing, by contrast, addresses how MKTd03 is *characterised* in non-technical registers: pitch materials, partner conversations, investor communications, press, standards-body contributions, marketing copy.

Both registers are legitimate. They are not interchangeable. This note keeps them separate by holding the rhetorical/comparative register here, non-normative, outside the MKTd03 spec.

## 2. Scope

This note applies whenever MKTd03 is characterised in language that:
- makes comparative claims against other protocols, prior family members, or categories of deletion-evidence mechanism,
- frames MKTd03 in commercial, regulatory-compliance, or industry-positioning terms,
- restates technical properties in a register intended to persuade rather than specify,
- attributes broader trust or completeness properties to MKTd03 than its baseline evidentiary scope supports.

This note does **not** apply to the MKTd03 normative artefacts themselves. Normative artefacts remain governed by their own ADR and companion-rule authority.

## 3. Non-scope

This note does not:
- certify any claim as true or false,
- authorise or forbid specific commercial or marketing decisions,
- define TAV doctrine (Playbook and Design Principles remain the doctrine authorities),
- act as authority over MKTd03's normative spec (the ADR chain and interface artefacts remain authoritative for protocol truth),
- substitute for legal, regulatory, or compliance review of any external communication.

## 4. Relation to the MKTd03 RST Evaluation Lens

The MKTd03 Residual Trust Statement (RST) Evaluation Lens v1 in the MKTd03 repo is a non-normative analytical artefact that identifies useful residual-trust categories and analytical questions for MKTd03 claims.

This Standards companion note operates in a different register — operator-facing rather than engineer-facing. It translates the same boundary discipline into rules for external claim framing. The two documents are intended to remain consistent, but neither is doctrine and neither is authoritative for the other.

## 5. Rules for rhetorical and comparative claim framing about MKTd03

### 5.1 Pair comparative claims with residual-trust statements
Any comparative claim about MKTd03 — against another protocol, against prior family members, against a category of mechanism, or against an unspecified baseline — must either:
- state explicitly which residual-trust categories remain after the comparison, or
- be narrowed to a purely technical-framing statement that does not imply broader trust relocation.

### 5.2 Do not collapse technical framing into commercial framing
A claim stated in technical framing — for example, that MKTd03 baseline verification is archival-first and does not depend on live fetch as a mandatory first step — must not be restated in commercial framing in a way that drops the qualifiers ("archival-first," "baseline," "mandatory first step," "by default"). Dropping the qualifiers produces a stronger-sounding claim that no longer corresponds to what MKTd03 proves.

### 5.3 Name the baseline boundary when making completeness claims
Any claim that uses completeness language — "complete deletion," "full erasure," "total compliance" — must either:
- explicitly restate MKTd03's baseline non-claim posture (per ADR-03 "Explicit non-claims") within the same communication, or
- be removed and replaced with scope-bounded language that matches what baseline evidence supports.

### 5.4 Do not import rhetorical framing into normative artefacts
Rhetorical and comparative language that appears in this note, in external communications, or in operator-facing materials must not be back-ported into MKTd03's normative spec artefacts — ADRs, protocol refresh, interface files, companion rules, verifier documents, security/privacy note, diagnostics note, versioning note. The separation is a hard boundary and runs in both directions.

### 5.5 Check review discipline before external publication
Any document characterising MKTd03 for external audiences — including pitch decks, investor material, partner briefs, standards-body contributions, press material, and website copy — should be reviewed against rules 5.1 through 5.4 before publication. This review is operator responsibility, not protocol responsibility.

## 6. What this note is not

- It is **not** TAV doctrine. Doctrine lives in `TAV_Vibe_Coding_Playbook.md` and `TAV_Design_Principles.md`.
- It is **not** project-management authority. MKTd03's Playbook session discipline remains unchanged.
- It is **not** a permission slip for any specific commercial claim. It is a set of rules for constructing such claims responsibly.
- It is **not** a general-purpose TAV rhetorical-framing policy. It is scoped to MKTd03. Future TAV protocol-family products may warrant similar notes, or the rules may later be abstracted into doctrine — that decision is not made here.
- It is **not** a substitute for external legal, regulatory, or compliance review.

## 7. Registration

This note is registered in `CHANGELOG.md` on the date above.
