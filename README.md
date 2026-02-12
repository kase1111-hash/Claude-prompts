# Idea Pipeline

A step-by-step natural language pipeline for building software with AI.

## How It Works

1. State your idea in one sentence
2. Point your AI at each step in order: *"begin on [link to step]"*
3. Each step transforms the output of the previous step into something more concrete

## The Hierarchy

```
  ▲ Least words, most compressed
  │
  │  01 — IDEA           → One sentence. What are you building?
  │  02 — SPEC           → Real-world requirements. What must it do?
  │  03 — ARCHITECTURE   → Code outline. How is it structured?
  │  04 — IMPLEMENTATION → File-by-file build instructions
  │  05 — VALIDATION     → How do you know it works?
  │  06 — FEEDBACK       → What changed? Propagate upward.
  │
  ▼ Most words, most specific
```

## Usage

```
You: "Create a game where you change lightbulbs"
AI:  [generates README from 01-Idea.md]

You: "Begin on 02-spec.md"
AI:  [expands idea into requirements]

You: "Begin on 03-architecture.md"
AI:  [designs code structure from spec]

...and so on
```

## Steps

| Step | File | Input | Output |
|------|------|-------|--------|
| 1 | [01-Idea.md](01-Idea.md) | Your one sentence | Project README |
| 2 | [02-spec.md](02-spec.md) | README | Requirements document |
| 3 | [03-architecture.md](03-architecture.md) | Requirements | Code outline / file map |
| 4 | [04-implementation.md](04-implementation.md) | Code outline | Built files |
| 5 | [05-validation.md](05-validation.md) | Built files | Test results / fixes |
| 6 | [06-feedback.md](06-feedback.md) | Issues found | Updates to upstream steps |

## Evaluation Frameworks

Use these after building to assess quality and alignment:

| Framework | File | Purpose |
|-----------|------|---------|
| Project Analyzer | [Concept-Execution-Evaluation.md](Concept-Execution-Evaluation.md) | Quick concept/execution/scope assessment with actionable verdicts |
| Quality Evaluation | [Quality-Evaluation.md](Quality-Evaluation.md) | Comprehensive purpose-fidelity and production-readiness audit |

## Philosophy

Every layer decompresses the one above it. The idea is maximally compressed. Each step adds specificity and removes ambiguity. When a lower layer reveals a problem, the feedback loop carries that signal back up to the layer where the fix belongs.
