# Step 03 — Architecture

## Your Job

Review the spec. Override any technology choices or structural decisions you disagree with. Or just say "begin."

## AI Instructions

Read `SPEC.md`. Design the code structure that satisfies every numbered requirement. This is the blueprint — no actual code yet.

### Generate These Sections

**Technology Decisions** — State every choice and why. Language, framework, libraries, data format. Each decision should reference which spec requirements drove it.

**File Map** — List every file that will be created. For each file:
- Filename and path
- Purpose (one sentence)
- Which spec requirements it addresses (by number)

**Data Model** — What data structures exist? What are their fields? How do they relate to each other? Use plain language, not code.

**Component Breakdown** — For each major piece of functionality:
- Name
- What it does
- What it receives (inputs)
- What it produces (outputs)  
- Which other components it talks to

**Flow Diagram** — Describe the execution flow in numbered steps. "1. User opens app → 2. System loads level data → 3. Renderer draws room..." etc. This should map directly to the User Experience Flow from the spec.

**State Management** — What state exists? Where does it live? What triggers state changes? This prevents the most common class of bugs in AI-generated code.

### Rules

- Every spec requirement must appear in at least one file's responsibility list — if a requirement is orphaned, the architecture is incomplete
- Prefer fewer files over more files unless complexity demands separation
- Flag any spec requirements that conflict with each other
- If a requirement is expensive or complex to implement, say so and suggest a simpler alternative
- Keep it under 800 words

### Output

A single `ARCHITECTURE.md` file in the project repo.
