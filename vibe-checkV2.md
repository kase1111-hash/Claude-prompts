# Vibe-Code Detection Audit v2.0

You are performing a remediation-focused vibe-code audit on this repository. The goal is NOT to shame AI-assisted development — it's to find where AI-generated code lacks meaningful human review so the developer can shore it up.

## Framework

Score across three weighted domains:
- **A. Surface Provenance (20%)** — How was the code created?
- **B. Behavioral Integrity (50%)** — Does the code actually work?
- **C. Interface Authenticity (30%)** — Is the interface genuine?

Each domain has 7 sub-criteria scored 1-3 (Weak/Moderate/Strong). Higher = more authentic.

**Final calculation:**
```
Weighted Authenticity = (A% × 0.20) + (B% × 0.50) + (C% × 0.30)
Vibe-Code Confidence = 100% - Weighted Authenticity
```

**Classification scale:**
| Range | Classification |
|-------|---------------|
| 0-15 | Human-Authored |
| 16-35 | AI-Assisted |
| 36-60 | Substantially Vibe-Coded |
| 61-85 | Predominantly Vibe-Coded |
| 86-100 | Almost Certainly AI-Generated |

---

## Instructions

Work through each sub-criterion below. For each one:
1. Run the indicated shell commands and code inspections
2. Document concrete evidence with file paths and line numbers
3. Score 1 (Weak), 2 (Moderate), or 3 (Strong)
4. Write a remediation note for anything scoring below 3

Be honest in both directions — acknowledge genuine engineering depth where it exists, and flag decorative/incomplete code without sugarcoating.

---

## Domain A: Surface Provenance (20%)

### A1. Commit History Patterns

Run these commands and analyze the results:

```bash
# Total commits and author breakdown
git log --all --no-merges --format='%an' | sort | uniq -c | sort -rn

# AI-attributed commit percentage
git log --all --no-merges --format='%an' | grep -iE '(claude|copilot|cursor|aider|devin|gpt|sweep|cody|bot|dependabot|renovate)' | wc -l
git log --all --no-merges --oneline | wc -l

# Formulaic commit message patterns (Add X, Implement Y, feat: Z, fix: W)
git log --all --no-merges --format='%s' | grep -cE '^(Add|Implement|Create|Update|Fix|Remove|Refactor|Improve) \w+|^(feat|fix|chore|docs|refactor|test)\(.+\):|^(feat|fix|chore|docs|refactor|test):'

# Human frustration/iteration markers
git log --all --no-merges --format='%s' | grep -ciE '(wip|broken|oops|typo|hotfix|workaround|hack|revert|ugh|temp|nit|cleanup|forgot|whoops|debug|wtf|scratch that|try again|finally|works now|fingers crossed)'

# Reverts (course correction signals)
git log --all --no-merges --format='%s' | grep -ci '^revert'

# AI-tool branch naming patterns
git branch -a --format='%(refname:short)' | grep -iE '(claude|copilot|cursor|aider|devin)/|^(ai|auto)[-/]'
```

**Score guide:**
- **1 (Weak):** >50% AI-authored, zero human markers, all formulaic messages
- **2 (Moderate):** Mixed authorship OR some human markers present
- **3 (Strong):** Clear human iteration visible in history

### A2. Comment Archaeology

```bash
# Tutorial-style comments ("Step N:", "First, we...", "Here we define...")
grep -rnE '#\s*(Step\s+[0-9]|First,?\s+(we|let)|Next,?\s+(we|let)|Now\s+(we|let)|Finally,?\s+(we|let)|Here\s+we\s+(define|create|implement|set up)|This\s+(function|method|class|module)\s+(is|will|does|handles)|We\s+(need|want|should|must)\s+to)' --include='*.py' --include='*.js' --include='*.ts' --include='*.go' --include='*.rs' | wc -l

# Section divider comments (# ====, # ----, etc.)
grep -rnE '#\s*[=]{4,}|#\s*[-]{4,}' --include='*.py' --include='*.js' --include='*.ts' | wc -l

# TODO/FIXME/XXX/HACK markers
grep -rnEi '(TODO|FIXME|XXX|HACK|WORKAROUND|KLUDGE)' --include='*.py' --include='*.js' --include='*.ts' --include='*.go' --include='*.rs' | wc -l

# WHY comments (because, since, reason, due to, workaround for)
grep -rnEi '#.*\b(because|since|reason|due to|workaround for|NOTE:)' --include='*.py' --include='*.js' --include='*.ts' | wc -l

# Count source files for ratio context
find . -name '*.py' -o -name '*.js' -o -name '*.ts' -o -name '*.go' -o -name '*.rs' | grep -v node_modules | grep -v __pycache__ | grep -v .git | wc -l
```

**Score guide:**
- **1 (Weak):** Many tutorial comments, zero TODOs/FIXMEs, comments describe WHAT never WHY
- **2 (Moderate):** Some tutorial patterns but TODOs/FIXMEs exist
- **3 (Strong):** Comments explain WHY, contain human iteration markers

### A3. Test Quality Signals

```bash
# Count test functions
grep -rnE '(def test_|it\(|describe\(|#\[test\]|func Test)' --include='*.py' --include='*.js' --include='*.ts' --include='*.go' --include='*.rs' | wc -l

# Trivial assertions: assert X is not None
grep -rnE 'assert\s+\w+\s+is\s+not\s+None|assert.*is not None|expect\(.*\)\.not\.toBeNull|expect\(.*\)\.toBeDefined' --include='*.py' --include='*.js' --include='*.ts' | wc -l

# Error path testing: pytest.raises, assertRaises, expect(...).toThrow
grep -rnE 'pytest\.raises|assertRaises|\.toThrow|\.rejects|should panic|#\[should_panic\]' --include='*.py' --include='*.js' --include='*.ts' --include='*.go' --include='*.rs' | wc -l

# Formulaic test docstrings ("Tests for X.", "Test X.")
grep -rnE '"""Tests?\s+for\s+\w+' --include='*.py' | wc -l

# Parametrized/table-driven tests
grep -rnE '@pytest\.mark\.parametrize|\.each\(|table[Dd]riven|test_cases\s*=' --include='*.py' --include='*.js' --include='*.ts' --include='*.go' | wc -l
```

Then sample 3-5 test files and assess: Do tests verify behavior and values, or just existence? Are edge cases covered? Do test names describe scenarios or just function names?

**Score guide:**
- **1 (Weak):** Overwhelmingly happy-path, many trivial assertions, zero error-path tests
- **2 (Moderate):** Mix of quality — some deep tests alongside formulaic ones
- **3 (Strong):** Tests verify behavior, cover errors, use parametrization

### A4. Import & Dependency Hygiene

```bash
# List declared dependencies
cat requirements*.txt pyproject.toml setup.py setup.cfg package.json Cargo.toml go.mod 2>/dev/null | head -100

# Find all imports used in source
grep -rhE '^(import|from)\s+\w+' --include='*.py' | sed 's/from \([a-zA-Z0-9_]*\).*/\1/' | sed 's/import \([a-zA-Z0-9_]*\).*/\1/' | sort -u

# Wildcard imports
grep -rnE '^from\s+\S+\s+import\s+\*' --include='*.py' | wc -l

# Lazy imports (try/except import patterns — positive signal)
grep -rnE 'try:\s*$' -A1 --include='*.py' | grep 'import' | wc -l
```

Cross-reference: Are all declared dependencies actually imported somewhere? Are any imports not declared? Note any "sounds correct but never used" phantom deps.

**Score guide:**
- **1 (Weak):** Multiple phantom deps, wildcard imports, unused declarations
- **2 (Moderate):** Mostly clean, 1-2 phantom deps
- **3 (Strong):** All deps used, granular imports, lazy fallbacks for optionals

### A5. Naming Consistency

```bash
# Class names
grep -rhE '^\s*class\s+(\w+)' --include='*.py' | sed 's/.*class //' | sed 's/[:(].*//'

# Function/method names
grep -rhE '^\s*def\s+(\w+)' --include='*.py' | sed 's/.*def //' | sed 's/[:(].*//' | head -50

# Factory function patterns (create_X_Y)
grep -rnE 'def create_\w+' --include='*.py' | wc -l

# Logger initialization uniformity
grep -rl 'logger\s*=\s*logging\.getLogger(__name__)' --include='*.py' | wc -l
```

Check: Is naming perfectly uniform across ALL files (suspicious), or does it show natural variation (abbreviations, legacy names, mixed conventions from different contributors)?

**Score guide:**
- **1 (Weak):** Zero deviations across 30+ names — too uniform for human development
- **2 (Moderate):** Mostly consistent with minor natural variation
- **3 (Strong):** Consistent conventions with organic variation

### A6. Documentation vs Reality

```bash
# Count documentation files
find . -maxdepth 2 -name '*.md' -o -name '*.rst' | grep -v node_modules | grep -v .git | wc -l

# List them
find . -maxdepth 2 -name '*.md' -o -name '*.rst' | grep -v node_modules | grep -v .git
```

Read the README and compare claimed features to actual implementation. Check: Are documented features implemented? Is documentation volume proportionate to codebase age/size? Are there fabricated claims?

**Score guide:**
- **1 (Weak):** Major features documented but not implemented, or massive doc volume for a young project
- **2 (Moderate):** Docs mostly match reality, some volume disproportion
- **3 (Strong):** Documentation accurately reflects implementation

### A7. Dependency Utilization

For each declared dependency, verify it's used meaningfully (not just imported once in a dead module). Check whether dependencies are deeply integrated vs superficially referenced.

**Score guide:**
- **1 (Weak):** Multiple deps imported once or used trivially
- **2 (Moderate):** Most deps well-utilized, 1-2 superficial
- **3 (Strong):** All deps deeply integrated into actual functionality

---

## Domain B: Behavioral Integrity (50%)

**This is the most important domain.** Run two independent analysis passes:
1. **Problem-focused pass:** Catalogue every issue (dead code, mock data, exception swallowing, ghost config)
2. **Execution-tracing pass:** Pick 3-5 critical features and trace their call chains end-to-end

Reconcile both perspectives in your final score.

### B1. Error Handling Authenticity

```bash
# Bare except / except Exception
grep -rnE 'except\s*(Exception)?\s*:' --include='*.py' | wc -l

# except: pass (swallowed exceptions)
grep -rnE 'except.*:\s*$' -A1 --include='*.py' | grep -E '^\s+pass\s*$' | wc -l

# Custom exception classes
grep -rnE 'class\s+\w*(Error|Exception)\s*\(' --include='*.py'

# Exception chaining (raise X from e)
grep -rnE 'raise\s+\w+.*\bfrom\b' --include='*.py' | wc -l

# Typed exception handling (catching specific types)
grep -rnE 'except\s+[A-Z]\w+Error|except\s+\([A-Z]' --include='*.py' | wc -l
```

Then read 3-5 error handlers in critical paths (auth, data handling, network). Do they differentiate between error types? Do they log with context? Do they recover or re-raise meaningfully?

**Score guide:**
- **1 (Weak):** 20+ bare except:pass, no custom exceptions, silent failures in critical paths
- **2 (Moderate):** Mix of genuine typed handling alongside swallowed exceptions
- **3 (Strong):** Domain-specific exceptions, differentiated recovery, fail-closed on critical paths

### B2. Configuration Actually Used

```bash
# Env var definitions (in .env files, docker-compose, etc.)
grep -rhE '^[A-Z][A-Z0-9_]+=|^[A-Z][A-Z0-9_]+:' .env* docker-compose* 2>/dev/null | sed 's/[=:].*//' | sort -u

# Env var reads in source
grep -rhE "os\.(environ|getenv)\s*[\[\(]['\"]([A-Z][A-Z0-9_]+)" --include='*.py' | grep -oE '[A-Z][A-Z0-9_]+' | sort -u

# Config class fields
grep -rnE '^\s+\w+\s*[:=]' --include='*.py' -l | xargs grep -l 'Config\|Settings'
```

Cross-reference: Which defined env vars are actually read? Which config class fields are consumed by other code? Any settings API that allows changes but has no backend effect?

**Score guide:**
- **1 (Weak):** >30% ghost config, settings API with no backend effect
- **2 (Moderate):** ~80% config wired, some ghost vars
- **3 (Strong):** All config consumed, clear env-to-behavior mapping

### B3. Call Chain Completeness

Pick the 3-5 most important features claimed by the project. For each one, trace the complete call chain from entry point to final effect. Document:
- Does the chain complete, or does it dead-end in a stub/mock/pass?
- Are there any functions called that don't exist?
- Do return values get consumed by callers?

```bash
# Dead modules (files never imported by anything outside their own package)
# For each top-level package directory, check external imports:
for pkg in $(find . -mindepth 1 -maxdepth 1 -type d -not -name '.*' -not -name '__*' -not -name 'node_modules' -not -name 'tests'); do
  name=$(basename $pkg)
  external_imports=$(grep -rl "from ${name}\b\|import ${name}\b" --include='*.py' | grep -v "^\./${name}/" | wc -l)
  echo "${name}: ${external_imports} external imports"
done

# Functions that return hardcoded dicts/lists (potential mock data)
grep -rnE 'return\s*\{["\x27]' --include='*.py' | head -20

# NotImplementedError stubs
grep -rnE 'raise NotImplementedError' --include='*.py'

# Pass-only functions
grep -rnE '^\s+pass\s*$' -B5 --include='*.py' | grep -E 'def\s+\w+'
```

**Score guide:**
- **1 (Weak):** Major features are stubs, dead modules, mock data returns
- **2 (Moderate):** Core chains complete, peripheral features incomplete
- **3 (Strong):** All claimed features trace to real implementations

### B4. Async Correctness

(Skip if project doesn't use async.)

```bash
# Async functions
grep -rnE 'async\s+def' --include='*.py' | wc -l

# Blocking calls inside async functions (potential issues)
grep -rnE 'async\s+def' -A20 --include='*.py' | grep -E '(open\(|\.read\(\)|\.write\(\)|time\.sleep|subprocess\.run|requests\.)' | head -10

# Proper async lock usage
grep -rnE 'asyncio\.Lock|asyncio\.Semaphore|asyncio\.Event' --include='*.py' | wc -l

# Global mutable state accessed from async handlers
grep -rnE '^\w+\s*=\s*(\[\]|\{\}|set\(\)|None)' --include='*.py' | head -10
```

**Score guide:**
- **1 (Weak):** Blocking I/O in async handlers, no locks on shared state, event loop misuse
- **2 (Moderate):** Core async patterns correct, peripheral violations
- **3 (Strong):** Proper async/sync separation, correct locking, no blocking in event loop

### B5. State Management Coherence

Check all shared mutable state (singletons, global dicts, caches, connection pools):
- Is access thread-safe?
- Are caches bounded (size limits, TTL)?
- Is cleanup handled on shutdown?
- Is there a clear ownership model (DI container, factory pattern, etc.)?

```bash
# Singletons and global state
grep -rnE '^\w+\s*=\s*(None|\[\]|\{\}|set\(\))' --include='*.py' | grep -v test | grep -v __pycache__

# Thread locks
grep -rnE 'threading\.(Lock|RLock|Semaphore)|asyncio\.Lock' --include='*.py' | wc -l

# Cache/size limits
grep -rnE '(max_size|maxsize|cache_size|TTL|ttl|expires|evict)' --include='*.py' | wc -l
```

**Score guide:**
- **1 (Weak):** Unprotected shared state, unbounded caches, no cleanup
- **2 (Moderate):** Most state managed, some gaps
- **3 (Strong):** Thread-safe everywhere, bounded caches, proper lifecycle management

### B6. Security Implementation Depth

```bash
# Password hashing (look for real algorithms vs md5/sha1)
grep -rnEi '(pbkdf2|bcrypt|scrypt|argon2|sha256|md5|sha1)' --include='*.py'

# Hardcoded secrets
grep -rnEi '(password|secret|api_key|token)\s*=\s*["\x27][^"\x27]{8,}' --include='*.py' | grep -v test | grep -v example

# SQL injection vectors
grep -rnE 'f".*SELECT|f".*INSERT|f".*UPDATE|f".*DELETE|\.format\(.*SELECT' --include='*.py'

# SSRF protection
grep -rnEi '(validate.*url|block.*internal|reserved.*ip|metadata.*url)' --include='*.py'

# Rate limiting
grep -rnEi '(rate.?limit|throttl|backoff|retry.?after)' --include='*.py' | wc -l

# Input validation
grep -rnE '(validate|sanitize|escape|parameterize)' --include='*.py' | wc -l
```

Assess depth: Is security decorative (imported but not used) or deep (actual crypto, rate limiting with backoff, input validation at boundaries)?

**Score guide:**
- **1 (Weak):** Decorative security, hardcoded secrets, SQL injection vectors
- **2 (Moderate):** Real security on some paths, gaps on others
- **3 (Strong):** Production-grade crypto, rate limiting, input validation, SSRF protection

### B7. Resource Management

```bash
# Context managers (with statements)
grep -rnE '^\s+with\s+' --include='*.py' | wc -l

# File handles without context managers
grep -rnE '=\s*open\(' --include='*.py' | grep -v 'with ' | wc -l

# Cleanup/shutdown handlers
grep -rnEi '(atexit|signal\.signal|finally:|__del__|shutdown|cleanup|close\(\)|dispose)' --include='*.py' | wc -l

# Background task lifecycle
grep -rnEi '(asyncio\.create_task|threading\.Thread|CancelledError|daemon)' --include='*.py' | head -10
```

**Score guide:**
- **1 (Weak):** Leaked handles, no cleanup, unbounded growth
- **2 (Moderate):** Most resources managed, some gaps
- **3 (Strong):** Context managers, bounded queues, graceful shutdown

---

## Domain C: Interface Authenticity (30%)

### C1. API Design Consistency

Check route definitions across all modules. Are HTTP methods, status codes, error responses, and model patterns consistent? Is there a shared model layer or do routes define ad-hoc dicts?

### C2. UI Implementation Depth

If there's a frontend: Is it a real SPA with components, state management, and routing? Or a single HTML page with inline scripts? Check for real-time features (WebSocket, SSE) vs polling.

### C3. State Management (Frontend)

Does the frontend have proper state management (stores, reducers, context) or ad-hoc global variables?

### C4. Security Infrastructure

Rate limiting, CORS, CSP headers, session management, auth middleware — are these real or decorative?

### C5. WebSocket Implementation

If applicable: bidirectional communication, connection lifecycle, heartbeat, backpressure handling, reconnection protocol.

### C6. Error UX

Do users see structured error messages or raw stack traces? Are error states handled in the UI?

### C7. Logging & Observability

Structured logging (JSON), request tracing/correlation IDs, metrics collection, health checks backed by real data.

---

## Output Format

Produce a report following this structure:

```
# Vibe-Code Detection Audit v2.0
**Project:** [name]
**Date:** [date]
**Auditor:** Claude (automated analysis)

## Executive Summary
[2-3 paragraph summary: overall confidence, classification, key finding]

## Scoring Summary
| Domain | Weight | Score | Percentage | Rating |
|--------|--------|-------|------------|--------|
| A. Surface Provenance | 20% | X/21 | X% | Rating |
| B. Behavioral Integrity | 50% | X/21 | X% | Rating |
| C. Interface Authenticity | 30% | X/21 | X% | Rating |

Weighted Authenticity: X%
Vibe-Code Confidence: X%
Classification: [label]

## Domain A: Surface Provenance
[For each A1-A7: Evidence, Assessment, Score]

## Domain B: Behavioral Integrity
[For each B1-B7: Evidence (with file:line), Assessment, Score]

## Domain C: Interface Authenticity
[For each C1-C7: Evidence, Assessment, Score]

## High Severity Findings
[Table: Finding, Location, Impact, Remediation]

## Medium Severity Findings
[Table: Finding, Location, Impact, Remediation]

## What's Genuine
[Bulleted list of real engineering with evidence]

## What's Vibe-Coded
[Bulleted list of decorative/incomplete code with evidence]

## Remediation Checklist
- [ ] [Actionable item 1]
- [ ] [Actionable item 2]
...
```

## Critical Instructions

1. **Be specific.** Every claim needs a file path and line number.
2. **Trace call chains.** Don't just check if functions exist — verify they connect.
3. **Check both directions.** Look for problems AND genuine depth. Many AI-assisted codebases have deeply engineered cores with decorative periphery.
4. **Remediation over judgment.** Every negative finding must include a concrete fix.
5. **Acknowledge good work.** If security implementation is genuinely deep, say so with evidence. If test coverage has real depth in specific modules, credit it.
6. **The goal is to help.** This audit should give the developer a clear checklist for making their codebase production-ready, not a reason to feel bad about using AI tools.

Save the completed report as `VIBE_CHECK_REPORT.md` in the project root.
