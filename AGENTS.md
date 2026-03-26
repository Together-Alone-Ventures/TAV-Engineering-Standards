This repository contains durable cross-project engineering doctrine.

Scope
- Keep this repo tool-agnostic and cross-project where possible.
- Standards here should generalize cleanly across projects.
- Do not leak project-local operational detail into standing doctrine.

Working discipline
- Prefer additive or corrective updates over unnecessary structural rewrites.
- Distinguish durable doctrine from tool-specific setup notes.
- If a rule only makes sense for one repo, it probably does not belong here.
- If a rule is tool-specific, state the durable doctrine here and leave tool mechanics to setup docs or repo-local guidance.

Current prep context
- MKTd03 prep may propose standards deltas from recent lessons.
- Any AI-tool-related uplift should be phrased at tool-agnostic level, for example:
  - establish repo-level AI instruction context before coding begins
  - verify instruction loading before implementation starts

Non-goals
- No TinyPress-only or MKTd03-only commands, paths, or deployment procedures in main doctrine.
- No version-coupling of standards text to one AI tool unless explicitly justified and bounded.
