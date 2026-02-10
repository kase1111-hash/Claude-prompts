# Step 06 — Feedback

## Your Job

Review the issues surfaced during implementation and validation. Decide which ones matter. Or say "begin" to let the AI triage and fix.

## AI Instructions

Read `IMPLEMENTATION_NOTES.md` (if it exists) and `VALIDATION.md`. Collect every issue that was flagged. For each issue, determine which upstream step needs to change.

### Feedback Process

**1. Collect Issues** — List every problem found in steps 04 and 05.

**2. Classify Each Issue** — Where does the fix belong?

| Fix belongs in | When |
|---|---|
| `01-idea` / README | The core concept is unclear or contradictory |
| `02-spec` / SPEC.md | A requirement is missing, ambiguous, or conflicts with another |
| `03-architecture` / ARCHITECTURE.md | The structure doesn't support the requirements, or components don't connect properly |
| `04-implementation` | It's just a bug — the code doesn't match the architecture |
| No fix needed | It's a cosmetic issue or subjective preference |

**3. Apply Fixes** — For each issue:
- State what's changing and in which file
- Make the change
- If a change in an upstream file invalidates downstream files, note which steps need to be re-run

**4. Re-validate** — After fixes, re-run the checks from step 05 for any affected requirements.

### Rules

- Fix issues at the highest appropriate level. A missing spec requirement should be added to the spec, not hacked around in code
- If the idea itself was flawed, say so. It's better to fix the foundation than to patch the roof
- Keep a running log of every change made in this step
- If re-running a step would change more than 30% of its output, flag this — the project may need a full re-pass through the pipeline

### Output

A `CHANGELOG.md` file with:
- Every issue found
- Classification (which step owns the fix)
- What was changed
- Which steps need re-running (if any)
- Final status: COMPLETE or RE-RUN FROM STEP [N]
