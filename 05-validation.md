# Step 05 — Validation

## Your Job

Say "begin" and let the AI verify its own work. Add any specific things you want tested.

## AI Instructions

Read `SPEC.md` and the implemented code. Systematically verify that every numbered requirement is met.

### Validation Process

**1. Requirement Walkthrough** — Go through every numbered spec requirement. For each one:
- State the requirement
- State which file(s) address it
- Confirm it works or flag the gap

**2. Flow Test** — Walk through the User Experience Flow from the spec. Trace the actual code path step by step. Does the code do what the flow describes? Note any divergence.

**3. Edge Case Check** — Review every edge case listed in the spec. Is each one handled? How?

**4. Integration Check** — Do the components actually connect the way the architecture said they would? Check every interface between components.

**5. Run It** — If possible, execute the code. Report what happens. If errors occur, fix them and document what was wrong.

### Rules

- Do not skip requirements — every single one gets checked
- If a requirement is partially met, explain what's missing
- Fix any bugs found during validation immediately
- If a fix changes the architecture, log it for the feedback step
- Be honest. "This doesn't work" is more useful than "this mostly works"

### Output

A `VALIDATION.md` file with:
- Pass/fail status for each requirement
- List of bugs found and fixed
- List of issues that need feedback to upstream steps
- Final status: READY or NEEDS WORK
