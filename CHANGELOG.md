# TAV Engineering Standards — Changelog

## 2026-04-20 — v1.1 — MKTd03 Residual Trust Note added

- Added `MKTd03_Residual_Trust_Note.md` as a project-scoped, non-normative companion note for operator-facing rhetorical and comparative framing about MKTd03.
- Keeps rhetorical/comparative framing separate from MKTd03 normative protocol artefacts.
- Companion to the MKTd03 RST Evaluation Lens v1 in the MKTd03 repo; the two documents are intended to remain consistent, but neither is doctrine and neither is authoritative for the other.
- This addition is non-doctrine. `TAV_Vibe_Coding_Playbook.md` and `TAV_Design_Principles.md` remain unchanged.

## 2026-03-20 — v1.0 — Initial markdown conversion

- Migrated TAV Vibe Coding Playbook v4 from docx to markdown (TAV_Vibe_Coding_Playbook.md)
- Migrated TAV Design Principles v3 from docx to markdown (TAV_Design_Principles.md)
- Both documents are authoritative replacements for the previous docx versions
- Repo created under Together-Alone-Ventures/TAV-Engineering-Standards

### Pending Playbook updates (from MKTd03 Day 0 session lessons)
These lessons are recorded in MKTd03/MILESTONE_LOG.md and should be incorporated
into the Playbook in the next revision:

1. When downloading files to copy into WSL, delete previous versions from
   Downloads folder first. Verify filename has no (1) or (2) suffix before running cp.

2. At session start, Claude reads RESTART_PACK.md directly from GitHub and confirms
   state back to operator. Manual paste is the fallback only if connector is down or
   file looks stale.

3. Playbook update process (new -- not in v4):
   - During a session: record lessons in MILESTONE_LOG.md SESSION LESSON entries only
   - At project close: review all SESSION LESSON entries, decide which warrant a
     Playbook edit, do that as a deliberate post-project activity
   - G must do a secondary review of any proposed Playbook changes before they are
     committed
   - Record all changes in this CHANGELOG with rationale
