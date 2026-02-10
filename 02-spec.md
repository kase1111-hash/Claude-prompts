# Step 02 — Spec

## Your Job

Review the README and add any constraints or preferences the AI should know. Or just say "begin" and let it work from the README alone.

## AI Instructions

Read the project README. Produce a requirements spec that answers everything a developer would need to know BEFORE writing code. 

### Generate These Sections

**Functional Requirements** — What the software must do. Each requirement is one sentence starting with "The system shall..." or "The user can..." Be specific. If the README says "puzzle game," define what constitutes a puzzle, a win, a loss.

**User Experience Flow** — Walk through the application from the user's first interaction to completion. Step by step. What do they see? What do they do? What happens?

**Constraints** — What are the boundaries?
- Platform (web, CLI, mobile, desktop?)
- Single file or multi-file?
- Any dependencies or frameworks required?
- Performance expectations?
- Data storage needs?

**Edge Cases** — What happens when things go wrong or get weird? Empty states, invalid input, extreme values, interrupted sessions.

**Out of Scope** — Explicitly state what this project does NOT do. This prevents scope creep during implementation.

### Rules

- Every requirement must be testable — if you can't verify it, rewrite it
- Do not make technology choices unless the README or user specifies them
- If something in the README is ambiguous, pick the simpler interpretation and state your assumption
- Number every requirement for reference in later steps
- Keep it under 500 words

### Output

A single `SPEC.md` file in the project repo.
