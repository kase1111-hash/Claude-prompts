# Vibe-Code Detection Audit v1.0

## Identifying Software That Looks Right But Doesn't Work Right

> The most dangerous code isn't broken code. It's code that passes every surface-level inspection while silently doing nothing. LLMs generate what code *looks like*, not what code *does*.

---

## PURPOSE

Detect whether a codebase was AI-generated without meaningful human review, testing, or adversarial thinking. This audit goes beyond checking for AI-style comments or commit patterns — it examines whether the code **actually functions** or merely **appears to function**, and whether the interface was **designed for humans** or **designed by a model that has never used a mouse**.

This prompt produces a **Vibe-Code Confidence Score** (0-100) based on weighted findings across three domains: code provenance signals, behavioral integrity, and interface authenticity.

---

## AUDIT DOMAINS

```
┌─────────────────────────────────────────────┐
│  DOMAIN A: SURFACE PROVENANCE               │
│  Does this look like it was vibe-coded?      │
├─────────────────────────────────────────────┤
│  DOMAIN B: BEHAVIORAL INTEGRITY             │
│  Does the code actually do what it           │
│  appears to do?                              │
├─────────────────────────────────────────────┤
│  DOMAIN C: INTERFACE AUTHENTICITY            │
│  Was this interface designed by someone       │
│  who has used software with their hands?     │
└─────────────────────────────────────────────┘
```

---

## DOMAIN A: SURFACE PROVENANCE

These are the fast indicators — detectable without running the code or deeply tracing execution paths.

### A.1 File & Project Structure

- [ ] **Test absence**: No test files, no test runner config, no CI test steps, no test directory
- [ ] **Security file absence**: No `.env.example`, no SECURITY.md, no auth middleware file
- [ ] **Monolithic generation**: Large files that cover many concerns, generated in a single pass rather than composed incrementally
- [ ] **Uniform formatting**: Every file uses identical patterns, spacing, and structure — no variation that comes from different humans touching different parts over time
- [ ] **Polished README, hollow interior**: Marketing-quality README with badges, feature lists, and architecture diagrams — but the actual code is thin, untested, or incomplete

### A.2 Commit & History Patterns

- [ ] **Single-pass commits**: One or two massive commits containing the entire project, no iterative refinement
- [ ] **No security commits**: Zero commit messages referencing security fixes, vulnerability patches, or access control changes
- [ ] **No refactoring history**: No evidence of code being restructured, renamed, or reorganized — it was generated "right" the first time
- [ ] **Prompt artifacts in commits**: Commit messages that read like prompts or responses ("Add user authentication system with JWT tokens and refresh flow")
- [ ] **Timestamp clustering**: All commits within a short window, suggesting generation rather than development

### A.3 Comment & Naming Patterns

- [ ] **Tutorial-style comments**: Comments that explain what code does to a reader rather than why decisions were made ("This function validates the user input" vs "Validate before DB write to prevent injection — see incident #47")
- [ ] **Uniform naming intelligence**: Every variable and function named with the same level of descriptiveness and style — real codebases have naming inconsistencies because different parts were written on different days by different moods
- [ ] **No frustration artifacts**: No commented-out attempts, no "HACK:", no "FIXME: this is awful but it works" — real code carries scars
- [ ] **Explanatory constants**: `const MAX_RETRIES = 3; // Maximum number of retries` — the comment adds zero information, but an LLM generates it reflexively

### A.4 Dependency Signals

- [ ] **Kitchen-sink imports**: Libraries imported that are never used, or used once for something trivially implementable
- [ ] **Redundant dependencies**: Multiple libraries that do the same thing (two validation libraries, two HTTP clients, two date libraries)
- [ ] **Version indifference**: All dependencies on `latest` or `^` with no evidence that specific versions were chosen for compatibility
- [ ] **Prestigious but unnecessary**: Well-known libraries included for credibility (lodash imported for a single `_.get()` call)

---

## DOMAIN B: BEHAVIORAL INTEGRITY

This is where vibe-code detection gets serious. These checks require tracing execution paths, not just reading code.

### B.1 Disconnected Call Chains

**The core vibe-code failure: components that exist independently but aren't wired together.**

- [ ] **Dead middleware**: Auth middleware defined but never mounted on the router. Rate limiters imported but never applied. CORS configured but not used. For every security/infrastructure middleware, verify it appears in the actual request pipeline — not just in an import statement.
- [ ] **Orphan functions**: Functions defined, exported, but never called by anything. Utility files that nothing imports. Validation helpers that no route handler references.
- [ ] **Phantom feature flags**: Configuration for features that don't exist. Environment variables read but never checked. Feature toggle infrastructure with no toggles.
- [ ] **Registration without activation**: Event listeners defined but never registered. Cron jobs configured but never scheduled. WebSocket handlers that nothing connects to.

**How to check:** For every import and every function definition, trace forward: does anything actually call this? For every route handler, trace backward: what middleware actually executes before it?

### B.2 Error Handling Theater

**Vibe-coded error handling performs the ritual of catching without the substance of handling.**

- [ ] **Swallowed errors**: `catch (err) { console.log(err) }` — the error is logged (maybe) and execution continues as if nothing happened. No retry, no fallback, no user notification, no state cleanup.
- [ ] **No-op rethrows**: `catch (err) { throw err }` — functionally identical to no try/catch at all, but looks like error handling.
- [ ] **Uniform catch blocks**: Every try/catch in the codebase handles errors identically. Real error handling is context-specific — a database error requires different recovery than a network timeout than a validation failure.
- [ ] **Missing finally blocks**: Resources opened in try blocks (file handles, database connections, streams) with no finally/cleanup path.
- [ ] **Optimistic-only paths**: No error handling at all on operations that can fail — database writes, network requests, file operations, JSON parsing. The code assumes success.
- [ ] **Status code theater**: API endpoints that return 200 OK regardless of what happened. Or endpoints that always return `{ success: true }` with no error branch.

### B.3 Unchecked Return Values

**LLMs generate both the function and the caller independently. The function returns meaningful data. The caller ignores it.**

- [ ] **Ignored validation**: `validateInput(data)` is called but the return value (true/false/error object) is never checked. Execution continues regardless of validity.
- [ ] **Ignored save results**: `await db.save(record)` is called but the result isn't checked for success or failure. The response tells the user it worked no matter what.
- [ ] **Ignored authentication checks**: `isAuthenticated(req)` returns a boolean, but the route handler doesn't branch on it.
- [ ] **Fire-and-forget writes**: Database writes, API calls, or file writes that aren't awaited and whose success is never confirmed.

### B.4 Async/Await Integrity

**LLMs frequently generate async code that is syntactically valid but logically broken under real concurrency.**

- [ ] **Missing awaits on critical operations**: `db.save(record)` without `await` — the operation starts but the function returns before it completes. Works in testing where timing is favorable. Fails in production under load.
- [ ] **Sequential operations that should be parallel**: Awaiting independent operations one by one instead of `Promise.all()` — not a bug but a performance signature of code generated line-by-line.
- [ ] **Parallel operations that should be sequential**: Independent-looking operations that actually depend on each other's results, fired simultaneously with `Promise.all()` — fails intermittently based on race conditions.
- [ ] **Mixed paradigms**: `.then()` chains and `async/await` in the same function. Callbacks nested inside async functions. An LLM trained on diverse code samples generates hybrid patterns that a human would normalize.
- [ ] **Unhandled promise rejections**: Async operations without `.catch()` or try/catch. The operation fails silently and the application continues with undefined state.

### B.5 Configuration Theater

**Config is loaded but never actually used where it matters.**

- [ ] **Environment variables with permanent fallbacks**: `process.env.DATABASE_URL || 'postgresql://localhost:5432/mydb'` — every configurable value has a development default that works. The system appears configurable but runs on hardcoded values in practice.
- [ ] **Config files that nothing reads**: `.env` or `config.json` exists and is documented, but the actual code has connection strings, API URLs, and constants hardcoded inline.
- [ ] **Partial config propagation**: The config is loaded correctly at the top level but never passed to modules that need it. Submodules import their own hardcoded values.
- [ ] **Security config that doesn't apply**: CORS origins configured in a config file, but the actual CORS middleware uses `'*'`. JWT secret defined in env, but the token verification uses a hardcoded string.

### B.6 Incomplete CRUD

**LLMs lose coherence across long generations. First operations get full attention. Later operations get scaffolding.**

- [ ] **Create and Read work. Update and Delete don't.**: The POST and GET endpoints are fully implemented. PUT returns 200 without writing. DELETE either doesn't exist or returns success without doing anything.
- [ ] **Pagination is a stub**: List endpoints return all records with no actual pagination, but the API accepts `page` and `limit` parameters that are ignored.
- [ ] **Search doesn't filter**: Search endpoints accept query parameters but return all records regardless. The query is parsed, the database call ignores it.
- [ ] **Sorting is cosmetic**: Sort parameters are accepted, passed through, but the database query doesn't use them, or sorts by a default field regardless.

### B.7 Frontend/Backend Disconnect

**Generated in separate conversations. Each piece looks right in isolation. Never tested together.**

- [ ] **Client-side validation only**: The React form validates beautifully. The Express endpoint behind it accepts anything. The validation logic isn't shared or even consistent between frontend and backend.
- [ ] **API contract mismatch**: The frontend expects `{ data: { user: {...} } }`. The backend returns `{ user: {...} }`. Works in development with mocked data. Breaks in production.
- [ ] **Auth state fiction**: The frontend checks `localStorage` for an auth token and shows/hides UI accordingly. The backend doesn't validate the token on protected routes — or validates a different token format than the frontend sends.
- [ ] **Mock data residue**: Components that render placeholder data, hardcoded arrays, or Lorem Ipsum where real API calls should be. The fetch function exists but returns mock data that was never replaced.

### B.8 The Happy Path Ceiling

**The single most reliable vibe-code test: try the unhappy paths.**

- [ ] **What happens with malformed input?** Not wrong types — subtly wrong formats. A date in US format when the system expects ISO. A phone number with country code when the system expects bare digits. A string at the max length boundary.
- [ ] **What happens when the database is down?** Not slow — completely unreachable. Does the application crash, hang, or degrade gracefully?
- [ ] **What happens with expired vs. missing vs. invalid auth?** These are three different failure modes. Vibe-coded auth treats them identically or handles only one.
- [ ] **What happens with concurrent modifications?** Two users editing the same record. Two requests hitting the same endpoint simultaneously. Last write wins? Error? Corruption?
- [ ] **What happens when external APIs fail?** Not 500 — timeout. Or 429. Or a response body that doesn't match the expected schema. The LLM generates the success case from training data. It rarely generates graceful degradation because graceful degradation is bespoke.

---

## DOMAIN C: INTERFACE AUTHENTICITY

**An LLM has never used a mouse. It has never felt the frustration of a button that's 2 pixels too small, a modal that doesn't close when you click outside it, or a form that clears itself when you hit the back button. It generates interfaces from screenshots and descriptions — not from the embodied experience of using software with hands and eyes.**

A human-designed interface carries evidence of use. An LLM-designed interface carries evidence of description.

### C.1 Interaction Design Failures

These are things a human designer would catch in five minutes of using their own product:

- [ ] **Click targets too small or imprecise**: Buttons, links, and interactive elements under 44x44px (mobile) or with padding that makes the visual target much larger than the clickable area. An LLM can't feel the frustration of trying to tap a 20px link on a phone.
- [ ] **No hover/focus/active states**: Buttons that don't change on hover. Links with no visual feedback. Interactive elements that give zero indication they received input. An LLM generates the resting state because that's what screenshots show.
- [ ] **Forms that fight the user**: Tab order that jumps randomly. Input fields that don't accept paste. Phone number fields that reject formatting characters. Email fields that don't allow `+` aliases. Date pickers that default to 1970 instead of today. An LLM generates the input element — not the input *experience*.
- [ ] **Modal/dialog dysfunction**: Modals that don't close on ESC or outside click. Modals that scroll the background when you scroll inside them. Nested modals. Modals that lose your form data when dismissed.
- [ ] **Missing loading and transition states**: Buttons that don't disable after click (allowing double-submit). No loading indicators during async operations. Content that pops in with layout shift. State transitions with no animation or feedback — the UI just teleports between states.
- [ ] **Scroll behavior**: Horizontal scroll where there shouldn't be any. Infinite scroll that doesn't preserve position when you navigate back. Scroll containers inside scroll containers. Fixed headers that cover content on mobile.
- [ ] **Copy/paste hostility**: Text that can't be selected. Fields that block paste (especially passwords). Content that copies with invisible formatting artifacts.

### C.2 Layout & Spacing Uncanny Valley

LLM-generated layouts have a specific aesthetic signature — everything looks "designed" but nothing feels *comfortable*:

- [ ] **Grid rigidity**: Every element perfectly aligned to an obvious grid with zero organic variation. Real interfaces have intentional asymmetry, visual hierarchy, and breathing room that comes from iterating with real content.
- [ ] **Uniform spacing**: Every margin and padding is the same value (often 16px or 1rem everywhere). Real designs use varied spacing to create visual hierarchy — tight grouping for related elements, generous spacing for separation.
- [ ] **Placeholder content fit**: The layout works perfectly with placeholder text but breaks with real content. Names that are longer than "John Doe". Descriptions that exceed three lines. Titles in non-English languages that are much longer. LLMs design for the demo, not the data.
- [ ] **Component soup**: Pages assembled from generic UI kit components with no unifying visual identity. A card here, a table there, a hero section on top — each component technically correct, collectively lacking personality or intentional design language.
- [ ] **Shadow and radius overuse**: Drop shadows on everything. Rounded corners on everything. Gradients that serve no informational purpose. These are CSS properties the LLM applies liberally because they appear in modern design system training data.
- [ ] **Desktop-first with responsive afterthought**: The desktop layout is polished. The mobile layout is either broken, a crude shrinkwrap of the desktop layout, or suspiciously a completely different design (generated separately). Real responsive design adapts — it doesn't just stack.

### C.3 Information Architecture Failures

**These reveal that the interface was described rather than designed through use:**

- [ ] **Navigation that matches the database, not the user**: Menu items that mirror database table names or API endpoint names rather than user mental models. "Users", "Transactions", "Sessions" instead of "People", "Payments", "Activity".
- [ ] **Settings pages with no hierarchy**: Every setting dumped into one page or one modal with no grouping, no prioritization, no progressive disclosure. An LLM generates all the settings it can think of. A human surfaces the three that matter and hides the rest.
- [ ] **Feature parity as design**: Every entity in the system has identical CRUD views — same table layout, same form layout, same detail page layout. The user model has the same interface as the log model. A human designer recognizes that different data types need different presentations.
- [ ] **No empty states**: What does the dashboard show when there's no data? If it shows an empty table with headers and no rows, or renders nothing, or crashes — it wasn't designed by someone who considered the first-run experience.
- [ ] **No keyboard navigation**: Tab doesn't move through interactive elements in logical order. Enter doesn't submit forms. Escape doesn't close modals. Arrow keys don't work in dropdowns. These aren't accessibility bonuses — they're basic expectations that a human designer includes because they *use* keyboards.
- [ ] **Help and error messages written for developers**: "Error: SQLITE_CONSTRAINT_UNIQUE" instead of "That email is already registered." "404 Not Found" instead of "We couldn't find that page." LLMs generate the error the system produces, not the message the user needs.

### C.4 Visual Coherence

- [ ] **Multiple design systems**: Part of the UI uses Material Design components. Another part uses Tailwind defaults. A third section has custom-styled elements. This happens when different parts of the interface were generated in separate prompts with different context.
- [ ] **Color incoherence**: More than 5-6 distinct colors with no obvious system. Or a palette that looks like it came from a CSS framework default theme with no customization. Or accessible color combinations by coincidence rather than intention.
- [ ] **Typography chaos**: More than 2-3 font families. Inconsistent heading hierarchy (H2 smaller than H3 somewhere). Body text that's too small (under 16px) or line height that's too tight for comfortable reading. An LLM picks font properties that look reasonable in code but haven't been read on a screen.
- [ ] **Icon inconsistency**: Icons from multiple icon sets with different stroke weights, fill styles, and visual language. Or icons used decoratively rather than communicatively — every list item has an icon but the icons don't help you scan the content.

### C.5 The Five-Minute Use Test

**The single most reliable interface authenticity check: use the interface for five minutes with real intent.**

Not clicking through features. Not checking that pages load. Actually try to accomplish a task:

- [ ] **Can you complete the primary task without confusion?** If the app is a todo list, can you create, edit, and complete a task without hitting a dead end?
- [ ] **Does the interface respond to mistakes gracefully?** Type something wrong. Navigate away mid-form. Hit the back button. Refresh the page. Does the system help you recover or does it punish you?
- [ ] **Does anything feel surprising?** Buttons that don't do what you expect. Navigation that takes you somewhere unexpected. State that changes without your input. Surprise is the signature of an interface designed from description rather than from use.
- [ ] **Do you feel frustrated?** Not confused — frustrated. That feeling of "I can see what I want to do but the interface won't let me do it easily." This is the fundamental thing an LLM cannot optimize against because it has never felt it.
- [ ] **Could you explain the navigation model to someone?** If you can't describe where things are and how to get to them after five minutes of use, the information architecture was generated, not designed.

---

## SCORING

### Domain Weights

| Domain | Weight | Rationale |
|--------|--------|-----------|
| A — Surface Provenance | 20% | Fast indicators, lower confidence individually |
| B — Behavioral Integrity | 50% | Highest signal — code that doesn't actually work is the core vibe-code failure |
| C — Interface Authenticity | 30% | If a GUI exists, its design reveals authorship clearly |

If the project has no GUI (library, CLI tool, backend service), redistribute Domain C's weight to Domain B (making it 80% B, 20% A).

### Scoring Per Finding

Each checked finding adds to the vibe-code confidence score proportional to its domain weight and individual severity:

| Indicator Strength | Points | Example |
|-------------------|--------|---------|
| **Strong** | 3 | Auth middleware defined but never mounted |
| **Moderate** | 2 | Uniform error handling across all catch blocks |
| **Weak** | 1 | No frustration artifacts in comments |

### Confidence Thresholds

| Score Range | Assessment | Implication |
|-------------|-----------|-------------|
| **0-15** | **Human-authored** | Normal development patterns. Proceed with standard review. |
| **16-35** | **AI-assisted** | Likely used AI for portions but a human was in the loop. Review AI-generated sections more carefully. |
| **36-60** | **Substantially vibe-coded** | Significant portions generated without review. Trace all critical paths manually. Do not trust security features without verification. |
| **61-85** | **Predominantly vibe-coded** | Little evidence of human review or testing. Treat the entire codebase as unreviewed. Full security audit required before any production use. |
| **86-100** | **Entirely vibe-coded** | No meaningful human involvement beyond prompting. The software is a prototype wearing production clothes. Do not expose to real users, real credentials, or real data without a complete rewrite of critical paths. |

---

## REPORT FORMAT

```
VIBE-CODE DETECTION REPORT
==========================

Project:                [name]
Audit Date:             [date]
Auditor:                [Claude model identifier]

VIBE-CODE CONFIDENCE:   [0-100]
ASSESSMENT:             [Human-authored | AI-assisted | Substantially vibe-coded | 
                         Predominantly vibe-coded | Entirely vibe-coded]

DOMAIN SCORES:
  A — Surface Provenance:     [score] / [max possible]
  B — Behavioral Integrity:   [score] / [max possible]
  C — Interface Authenticity:  [score] / [max possible]

TOP FINDINGS:
  1. [Strongest indicator with location and evidence]
  2. [Second strongest]
  3. [Third strongest]
  ...

POSITIVE SIGNALS:
  Evidence of human involvement found:
  1. [Signal]
  2. [Signal]
  ...

CRITICAL DISCONNECTIONS:
  Code that exists but doesn't function:
  1. [Component] — defined at [location], never called/mounted/used
  2. [Feature] — accepts input at [location], ignores it at [location]
  ...

INTERFACE FINDINGS (if GUI exists):
  Comfort failures:
  1. [Finding] — [what a user would feel]
  2. [Finding] — [what a user would feel]
  ...

RECOMMENDATION:
  [One paragraph: what should happen next based on the score]
```

---

## USAGE NOTES

**When to run this audit:**

- Before depending on any open-source project you didn't build
- Before deploying any AI-generated code to production
- Before connecting any third-party tool to real credentials
- When evaluating software from a vendor who ships unusually fast
- When something looks too polished for its project age

**What this audit is NOT:**

This is not a judgment on AI-assisted development. Using AI to write code is fine. The danger is using AI to write code *and then not reviewing what it wrote*. This audit detects the absence of review, not the presence of AI.

**Combining with the Agentic Security Audit:**

This prompt is designed to work as a companion to the Agentic Security Audit v2.0. Run this first. If the vibe-code confidence score is above 35, run the security audit at STRICT or MAXIMUM strictness — the probability of real security failures is directly correlated with the absence of human review.

---

## VERSION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Feb 2026 | Initial release. Three-domain detection framework covering surface provenance, behavioral integrity, and interface authenticity. Developed from analysis of the Moltbook incident and broader patterns in vibe-coded production software. |

---

## LICENSE

CC0 1.0 Universal — Public Domain

---

*"An LLM has never felt the frustration of a button that's 2 pixels too small. It generates interfaces from screenshots and descriptions — not from the embodied experience of using software with hands and eyes."*
