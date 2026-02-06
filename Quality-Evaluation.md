COMPREHENSIVE SOFTWARE PURPOSE & QUALITY EVALUATION

(Idea-Centric, Drift-Sensitive, Production-Grade)

FOUNDATIONAL DOCTRINE (READ FIRST)

This evaluation treats software as a vehicle for ideas, not merely behavior.

Core axioms:

Purpose precedes polish. A well-built implementation of the wrong idea is a failure.

The idea must survive the code. If the implementation were deleted and rewritten from the spec, the same concept should emerge.

Legibility is load-bearing. Code that obscures intent fails, even if correct.

Drift is debt. Any divergence from intent that is undocumented is a liability.

The README is the deed. It must stake the conceptual claim clearly enough to establish attribution and priority.

All subsequent criteria exist to serve these principles.

EVALUATION SCOPE & PARAMETERS

Evaluator must explicitly set:

Strictness: LENIENT | STANDARD | STRICT

Context: PROTOTYPE | INTERNAL-TOOL | PRODUCTION | LIBRARY-FOR-OTHERS

Purpose Context: IDEA-STAKE | ADOPTION-SEEKING | REVENUE-GENERATING | ECOSYSTEM-COMPONENT

Focus Areas (optional): e.g. concept-clarity-critical, security-critical, performance-sensitive

Sections marked [CORE] are mandatory.
Sections marked [CONTEXTUAL] may be weighted or omitted with justification.

I. PURPOSE AUDIT [CORE]

Intent Alignment
Does the implementation match the documented purpose exactly?

Identify:

Features present in code but absent from spec (scope creep)

Features specified but missing in code (incomplete claims)

Has the project drifted from its original vision? Identify divergence points.

Are architectural decisions traceable to stated requirements?

Conceptual Legibility
Can a competent reader grasp the core idea within 5 minutes?

Is the novel concept clearly expressed in structure, naming, and boundaries?

Would an LLM indexing this repo extract the correct conceptual primitives?

Is the “why” as explicit as the “what”?

Does the README lead with the idea, not the implementation?

Specification Fidelity
Line-by-line: does behavior match documented behavior?

Are all documented constraints actually enforced in code?

Where implementation diverges, is the divergence explicitly documented with rationale?

Do identifiers (functions, classes, modules) consistently reflect spec terminology?

Doctrine of Intent Compliance
Is there a clear provenance chain: human vision → spec → implementation?

Is authorship of the idea defensible based on repo artifacts?

Are timestamps, versioning, and history sufficient to establish priority?

Is human judgment visibly distinct from AI-generated implementation?

Ecosystem Position
How does this repo relate to adjacent projects?

Are shared concepts implemented consistently across the portfolio?

Does this repo occupy a clear, non-overlapping conceptual territory?

Are declared dependencies accurate and justified?

II. STRUCTURAL ANALYSIS [CORE]

Map architecture and file organization

Identify entry points and execution flow

Document module/component relationships

Assess separation of concerns

Evaluate coupling and cohesion

Does structure reflect the conceptual model described in documentation?

III. IMPLEMENTATION QUALITY [CORE]
Code Quality

Readability: naming, formatting, cognitive load

DRY violations and duplication

Dead code and unused artifacts

Magic numbers / strings

Code smells (long methods, god objects, primitive obsession, etc.)

Pattern consistency

Naming alignment with specification language

Functionality & Correctness

Does the code do exactly what it claims—no more, no less?

Logic errors, boundary conditions, off-by-one mistakes

Edge case handling (nulls, empties, limits)

State management correctness

Concurrency or race conditions (if applicable)

API contract compliance

IV. RESILIENCE & RISK [CONTEXTUAL]
Error Handling

Exception coverage and intent

Graceful degradation

Retry / circuit breaker logic where appropriate

Logging adequacy (debuggable without leaking sensitive data)

User-facing error clarity

Security

Input validation and sanitization

Injection risks (SQL, XSS, CSRF, command, path traversal)

AuthN / AuthZ correctness

Secrets management

Dependency CVEs

Sensitive data handling

Cryptographic practices (if applicable)

Performance

Algorithmic complexity of critical paths

Inefficient access patterns (e.g., N+1)

Memory growth or leaks

Redundant computation

Caching opportunities

Resource cleanup

V. DEPENDENCY & DELIVERY HEALTH [CONTEXTUAL]
Dependencies

Count: appropriate vs bloated

Currency and maintenance status

License compatibility

Vendored vs external strategy

Testing

Test existence and estimated coverage

Test quality (spec-level vs implementation-level)

Organization and clarity

Testability of code (DI, mockability)

Missing test categories (unit, integration, e2e)

Documentation

README completeness and clarity

Unique value proposition stated immediately

Usage examples

API docs (if applicable)

Inline comments explain why, not what

Architecture or decision records

Spec-to-code traceability

Build & Deployment

Build correctness

Environment configuration

CI/CD presence and quality

Containerization / IaC (if applicable)

VI. MAINTAINABILITY PROJECTION [CORE]

Onboarding difficulty

Technical debt indicators

Extensibility for likely futures

Refactoring risk zones

Bus factor risks

Can the idea survive a full rewrite?

OUTPUT FORMAT (MANDATORY)
EXECUTIVE SUMMARY

Overall Assessment: PRODUCTION-READY | NEEDS-WORK | SIGNIFICANT-CONCERNS | NOT-RECOMMENDED

Purpose Fidelity: ALIGNED | MINOR-DRIFT | SIGNIFICANT-DRIFT | DISCONNECTED

Confidence Level: HIGH | MEDIUM | LOW

One-paragraph summary leading with purpose alignment.

SCORES (1–10)

Each dimension with brief justification:

Purpose Fidelity (subscores)

Implementation Quality

Resilience & Risk

Delivery Health

Maintainability

Overall

FINDINGS

Purpose Drift Findings (spec vs code, with file:line)

Conceptual Clarity Findings

Critical Findings (must fix)

High-Priority Findings

Moderate Findings

Observations (non-blocking)

POSITIVE HIGHLIGHTS

What the code does well

Idea expression strengths

RECOMMENDED ACTIONS

Immediate (Purpose)

Immediate (Quality)

Short-term

Long-term

QUESTIONS FOR AUTHORS

Clarifications required to close evaluation gaps.
