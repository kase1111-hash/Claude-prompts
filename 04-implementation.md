# Step 04 — Implementation

## Your Job

Review the architecture. Flag anything that looks wrong. Or say "begin" to start building.

## AI Instructions

Read `ARCHITECTURE.md`. Build every file listed in the File Map. Work through them in dependency order — files that other files depend on get built first.

### For Each File

1. **State what you're building** — filename, purpose, which spec requirements it covers
2. **Write the complete file** — no placeholders, no TODOs, no "implement this later"
3. **Verify internally** — before moving to the next file, confirm:
   - All inputs and outputs match what the architecture defined
   - All connections to other components use the interfaces defined in architecture
   - All spec requirements assigned to this file are addressed

### Rules

- Build files in dependency order. If `game.js` depends on `levels.js`, build `levels.js` first
- Every function, class, or module should have a brief comment explaining what it does
- Match the data model exactly — field names, types, structures as defined in architecture
- If you discover the architecture is missing something, note it but keep building. Log it for the feedback step
- Do not add features not in the spec. Do not optimize prematurely
- If a file exceeds 300 lines, consider whether the architecture should have split it — note this for feedback
- Include all imports, config, and setup. The code must run without modification

### When Complete

List every file created with a one-line summary. Flag any issues discovered during implementation that should feed back to earlier steps.

### Output

All project files as defined in `ARCHITECTURE.md`, plus an `IMPLEMENTATION_NOTES.md` if any issues were discovered.
