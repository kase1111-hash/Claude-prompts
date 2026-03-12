# Dependency Audit & Reduction

## Goal

Analyze all project dependencies, identify which are truly necessary, and intelligently reduce them where possible — without breaking functionality. The priority is **working, efficient code** with the smallest practical dependency footprint.

## AI Instructions

Read the project's dependency manifests (e.g. `package.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`, or equivalent) and the full codebase. Perform the following analysis and actions.

### Phase 1 — Inventory

**1. Map Every Dependency**

For each dependency (direct and dev/build):
- Name and current version
- What it provides (one-line purpose)
- Where it is used in the codebase (file:line references)
- How heavily it is used (single call, moderate, pervasive)
- Whether it is a direct or transitive dependency

**2. Identify the Dependency Graph**

- Total direct dependency count
- Total transitive dependency count (if determinable)
- Heaviest sub-trees (dependencies that pull in the most transitive deps)
- Any circular or redundant dependency chains

### Phase 2 — Analysis

**3. Classify Each Dependency**

Assign each dependency one of these labels:

| Label | Meaning |
|---|---|
| **ESSENTIAL** | Core to the project's function; no reasonable alternative to using it |
| **JUSTIFIED** | Provides significant value relative to its cost; keeping it is sensible |
| **REPLACEABLE** | Could be swapped for a lighter or built-in alternative without major effort |
| **REMOVABLE** | Used minimally or not at all; can be dropped with trivial code changes |
| **REDUNDANT** | Overlaps with another dependency or built-in functionality already present |
| **DEAD** | Imported/listed but never actually used in the codebase |

**4. Evaluate Each Non-Essential Dependency**

For every dependency classified as REPLACEABLE, REMOVABLE, REDUNDANT, or DEAD:
- What would replacing or removing it require?
- Estimated lines of code to write as a replacement (if applicable)
- Risk level of removal: NONE | LOW | MEDIUM | HIGH
- Does a native/stdlib alternative exist?
- Does another already-included dependency cover the same functionality?

**5. Check Dependency Health**

For all dependencies:
- Last published update (stale if >2 years without release)
- Known CVEs or security advisories
- Maintenance status (active, maintenance-mode, abandoned)
- License compatibility with the project

### Phase 3 — Action

**6. Remove Dead Dependencies**

Delete any dependency that is listed but never imported or used. This is free improvement — do it immediately.

**7. Consolidate Redundant Dependencies**

Where two or more dependencies serve overlapping purposes, consolidate to one. Pick the one that:
- Is already more widely used in the codebase
- Has better maintenance and community health
- Pulls in fewer transitive dependencies

**8. Replace Where the Trade-Off Is Clear**

For REPLACEABLE dependencies, implement the replacement **only when**:
- The replacement code is straightforward (not a re-implementation of a complex library)
- The result is equal or better in correctness and performance
- The maintenance burden of the inline code is lower than the dependency cost

Do **not** replace dependencies where:
- The library handles genuine complexity (crypto, parsing, protocol compliance)
- The replacement would be fragile or bug-prone
- The library is well-maintained and lightweight already

**9. Pin and Clean Up**

- Ensure all kept dependencies have version constraints (no floating/unpinned versions)
- Remove lockfile entries for removed dependencies
- Regenerate the lockfile if applicable

### Rules

- **Never sacrifice correctness for a smaller dependency list.** A working app with 30 deps beats a broken app with 10.
- **Test after every removal or replacement.** Run the full test suite (or manually verify if no tests exist) before moving to the next change.
- **Make changes incrementally.** One dependency at a time. Commit between changes so each removal is independently revertible.
- **Preserve public API and behavior.** No user-facing change should result from this process.
- **Do not introduce new dependencies** to replace old ones unless the new one is strictly smaller in scope and footprint.
- **Be honest about trade-offs.** If a dependency is heavy but genuinely needed, say so and move on.

### Output

A `DEPENDENCY-AUDIT.md` file containing:

#### Summary
- Total dependencies before and after
- Total transitive dependencies before and after (if measurable)
- Number removed, consolidated, or replaced

#### Dependency Table
Full inventory with classification labels and action taken.

#### Changes Made
For each dependency removed, consolidated, or replaced:
- What was changed and why
- Files modified
- Replacement code written (if any, with brief explanation)
- Test results after the change

#### Kept With Reservations
Dependencies that are heavier than ideal but justified. Note why and any future considerations.

#### Health Warnings
Dependencies with security issues, abandonment risk, or license concerns that warrant attention even if not removed now.

#### Final Status
- LEANER — Dependencies were reduced with no functional impact
- ALREADY-MINIMAL — No meaningful reductions possible; dependency set is healthy
- NEEDS-WORK — Reductions identified but some require manual intervention or decisions
