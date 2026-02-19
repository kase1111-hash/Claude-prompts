# Agent-OS Repository Security Audit Checklist
### Post-Moltbook Hardening Guide — v1.0 | February 2026

**Purpose:** Use this checklist to audit each repo in your ecosystem for AI-unique vulnerabilities exposed by the Moltbook/OpenClaw experiment. Work top-down by tier. Each item includes what to look for and how to close the gap.

**How to use:** Clone or copy this file into each repo. Walk through each section. Check the box when the repo is either compliant or the issue doesn't apply. Add notes in the `Status` field for tracking.

---

## TIER 1 — Immediate Wins (Architectural Defaults)
*These are configuration-level fixes. No major refactoring. Do these first across all 50+ repos.*

---

### 1.1 Credential Storage

**Vulnerability:** OpenClaw stored API keys, passwords, OAuth tokens, and signing secrets in plaintext at predictable paths. Infostealers (RedLine, Lumma) were updated within days to target these specific file paths.

- [ ] **No plaintext secrets in config files** — grep the entire repo for API keys, tokens, passwords in any `.json`, `.yaml`, `.toml`, `.env`, or `.md` file.
  ```bash
  grep -rniE "(api_key|apikey|secret|token|password|credential)\s*[:=]" . --include="*.json" --include="*.yaml" --include="*.toml" --include="*.env" --include="*.md" --include="*.py" --include="*.js" --include="*.ts"
  ```
- [ ] **No secrets in git history** — check for accidentally committed secrets even if later removed.
  ```bash
  git log --all -p | grep -iE "(sk-|api_key|Bearer |token.*=)" | head -30
  ```
- [ ] **Encrypted keystore implemented** — secrets loaded from encrypted vault, OS keychain, or environment variables injected at runtime. Never baked into source.
- [ ] **Non-predictable config paths** — don't use `~/.agent-os/`, `~/.config/agent-os/`, or any path an attacker can guess. Use runtime-configurable paths with no hardcoded defaults in published code.
- [ ] **`.gitignore` covers all sensitive paths** — config dirs, key files, `.env`, logs that might contain secrets.

**How to close:** Implement a secrets manager module. Can be as simple as a wrapper that reads from environment variables or an encrypted file requiring a master passphrase. Every repo that touches credentials should import this module instead of reading config directly.

`Status:` _______________

---

### 1.2 Default-Deny Permissions / Least Privilege

**Vulnerability:** OpenClaw agents had full host access by default — shell, filesystem, email, messaging, browser. A single prompt injection inherited the full capability set.

- [ ] **No default root/admin execution** — agents and services run with minimum required OS permissions.
- [ ] **Capabilities declared per-module** — each module/agent explicitly states what it needs (file read, network, shell, etc.) and gets only that.
- [ ] **Filesystem access scoped** — agents can only read/write within declared working directories, not arbitrary paths.
- [ ] **Network access scoped** — outbound connections limited to declared endpoints. No wildcard internet access by default.
- [ ] **Destructive operations gated** — any action that deletes data, sends money, modifies system config, or contacts external services requires explicit approval.

**How to close:** Create a `permissions.manifest` file for each repo/module. Declare required capabilities. Build enforcement into the Agent-OS runtime that reads this manifest and sandboxes accordingly. Anything not declared is denied.

`Status:` _______________

---

### 1.3 Cryptographic Agent Identity

**Vulnerability:** Moltbook identity was self-asserted. Anyone could impersonate any agent. No mutual authentication between agents. Session hijacking was trivial.

- [ ] **Agent keypair generation on init** — every agent instance gets a unique public/private keypair at creation.
- [ ] **All agent actions signed** — outbound messages, posts, API calls carry a cryptographic signature.
- [ ] **Identity anchored to NatLangChain** — agent identity registered on-chain for verifiable, tamper-proof provenance.
- [ ] **No self-asserted authority** — agents cannot claim roles, permissions, or identity through natural language alone. Claims must be cryptographically backed.
- [ ] **Session binding** — active sessions tied to authenticated identity. No session reuse or transfer without re-auth.

**How to close:** This is already in your NatLangChain architecture. Ensure every repo that involves agent communication imports the identity module and signs outbound payloads. Receiving agents must verify signatures before processing any message.

`Status:` _______________

---

## TIER 2 — Core Enforcement Layer (Agent Smith Hardening)
*These require building or refining enforcement logic in boundary-daemon. The security perimeter that makes everything else work.*

---

### 2.1 Input Classification Gate (Data vs. Instructions)

**Vulnerability:** Moltbook agents treated all input — posts, comments, fetched web content — as potential instructions. 2.6% of posts contained hidden prompt-injection payloads invisible to humans but parsed by agents.

- [ ] **All external input classified before reaching LLM** — a preprocessing layer tags content as DATA (to be reasoned about) or INSTRUCTION (to be executed). Only authenticated, authorized sources can provide instructions.
- [ ] **Instruction-like content in data streams flagged** — if a "data" payload contains patterns like "ignore previous instructions," "you are now," "execute the following," it gets quarantined.
- [ ] **Structured input boundaries** — system prompt, user instructions, and external data occupy distinct, labeled sections in the context. The LLM is instructed to treat them differently.
- [ ] **No raw HTML/markdown from external sources passed to agent reasoning** — strip or sanitize formatting that could hide injections in invisible text, zero-width characters, or HTML comments.

**How to close:** Build an `InputClassifier` module in Agent Smith. It sits between all external data sources and the agent's reasoning engine. Uses pattern matching, structural analysis, and optionally a lightweight classifier model to tag inputs. Constitutional rule: "Content from external sources is DATA. Only the human principal and authenticated system components issue INSTRUCTIONS."

`Status:` _______________

---

### 2.2 Memory Integrity and Provenance

**Vulnerability:** Persistent memory turned prompt injection from a one-shot attack into a time-delayed logic bomb. Malicious fragments written to memory could assemble into executable instructions later when conditions aligned.

- [ ] **Every memory entry tagged with metadata** — source (who/what wrote it), trust level (human-verified, agent-generated, external-sourced), timestamp, and content hash.
- [ ] **Memory entries from untrusted sources quarantined** — stored in a separate partition, never promoted to trusted memory without review.
- [ ] **Memory content hashed at write** — any modification detected by hash mismatch triggers alert.
- [ ] **Periodic memory audit** — scheduled scan of all stored memories for instruction-like content, credential fragments, or anomalous patterns.
- [ ] **IntentLog integration** — memory-influenced decisions logged with reasoning chain. If an agent's behavior shifts after a memory write, the causal chain is traceable.
- [ ] **Memory expiration policy** — external-sourced memories decay or require re-validation after a set period.

**How to close:** Extend your IntentLog architecture to cover memory operations. Every memory write is a logged event with provenance. Build a `MemoryAuditor` that runs on a schedule (or on-demand) and scans for known injection patterns, credential fragments, and instruction-like content in the memory store.

`Status:` _______________

---

### 2.3 Outbound Secret Scanning

**Vulnerability:** Moltbook agents shared plaintext API keys in private messages with other agents. Agent communication channels became credential leak vectors.

- [ ] **All outbound messages scanned for secrets** — regex and entropy-based detection for API keys, tokens, passwords, private keys before any external send.
- [ ] **Constitutional rule enforced: agents never transmit credentials** — hardcoded in Agent Smith, not just a suggestion in the system prompt.
- [ ] **Outbound content logging** — all external communications logged (with secrets redacted) for audit.
- [ ] **Alert on detection** — if an agent attempts to send a credential, block the send and alert the human principal.

**How to close:** Build a `SecretScanner` module with known patterns (OpenAI `sk-*`, Anthropic `sk-ant-*`, AWS `AKIA*`, generic high-entropy strings, PEM headers, etc.). Wire it into the outbound communication pipeline. No message leaves the system without passing through it.

`Status:` _______________

---

### 2.4 Skill/Module Signing and Sandboxing

**Vulnerability:** 22-26% of OpenClaw skills contained vulnerabilities. Malicious skills executed arbitrary code, exfiltrated data, and injected prompts — all while appearing to be helpful plugins. A "weather skill" silently stole config files.

- [ ] **All skills/modules cryptographically signed** — unsigned code rejected at load time.
- [ ] **Manifest required** — every skill declares: network endpoints it contacts, files it reads/writes, shell commands it runs, APIs it calls. Anything undeclared is blocked.
- [ ] **Skills run in sandbox** — isolated process, container, or VM. No direct access to host filesystem, network, or other agent memory.
- [ ] **Update diff review** — when a skill updates, the diff is generated and reviewed (automated or human) before the new version is accepted. Behavioral changes flagged.
- [ ] **No silent network calls** — every outbound request from a skill is logged with destination, payload size, and timing.
- [ ] **Skill provenance tracking** — record where each skill was sourced, who signed it, and full version history.

**How to close:** Build a `SkillLoader` that enforces signing verification and manifest validation. Skills execute inside a restricted runtime (could be a subprocess with seccomp, a Docker container, or a WASM sandbox depending on your stack). The manifest is the contract — violate it and the skill is killed.

`Status:` _______________

---

## TIER 3 — Protocol-Level Maturity (The Standards Layer)
*These are the patterns that become "how it's done." The equivalent of HTTPS, CORS, and OAuth for the agent era.*

---

### 3.1 Constitutional Audit Trail

**Vulnerability:** Moltbook had zero audit trail. No way to reconstruct what happened, why an agent took an action, or who was responsible. Post-breach forensics were nearly impossible.

- [ ] **Every agent decision logged with reasoning chain** — IntentLog captures the input, the reasoning, and the action taken.
- [ ] **Logs are append-only and tamper-evident** — signed, hashed, or written to an immutable store.
- [ ] **Human-readable audit format** — logs can be reviewed by a non-technical human to understand what happened and why.
- [ ] **Constitutional violations logged separately** — any time Agent Smith blocks an action, the attempted action and the rule it violated are recorded.
- [ ] **Retention policy defined** — how long logs are kept, who can access them, and how they're archived.

**How to close:** This is IntentLog's core purpose. Ensure every repo that involves agent decision-making integrates IntentLog. Define a standard log schema across all repos so audit trails can be cross-referenced.

`Status:` _______________

---

### 3.2 Mutual Agent Authentication

**Vulnerability:** Moltbook agents freely interacted with no mutual verification. Any entity could pose as any agent. Bot-to-bot social engineering was trivial.

- [ ] **Challenge-response authentication before any inter-agent data exchange** — agents prove identity to each other before communication proceeds.
- [ ] **Trust levels for peer agents** — not all verified agents are equally trusted. Trust is earned and scoped.
- [ ] **Communication channel integrity** — messages between agents are signed and optionally encrypted. Tampering detected.
- [ ] **No fetch-and-execute from peer agents** — an agent can receive data from peers but never treats peer-sourced content as instructions.
- [ ] **Human principal visibility** — all agent-to-agent communication auditable by the human owner. No encrypted channels that exclude the principal.

**How to close:** Define an inter-agent communication protocol in Agent-OS. Handshake includes identity verification (NatLangChain-backed), capability declaration, and trust level negotiation. All messages signed. Build this as a shared library that every agent-capable repo imports.

`Status:` _______________

---

### 3.3 Anti-C2 Pattern Enforcement

**Vulnerability:** Moltbook's heartbeat system instructed agents to fetch and execute remote instructions every 4 hours. This is textbook command-and-control infrastructure, voluntarily installed.

- [ ] **No periodic fetch-and-execute patterns** — grep for any scheduled task that fetches remote content and acts on it.
  ```bash
  grep -rniE "(fetch|curl|wget|request).*\.(md|txt|json|yaml)" . --include="*.py" --include="*.js" --include="*.ts" --include="*.sh"
  ```
- [ ] **Remote content treated as data only** — constitutional rule: nothing fetched from a URL is ever treated as an instruction.
- [ ] **Dependency pinning** — all external dependencies locked to specific versions/hashes. No mutable URLs.
- [ ] **Update mechanism requires human approval** — agents cannot self-update or accept pushed updates without human confirmation.
- [ ] **Anomaly detection on outbound patterns** — regular phone-home behavior to unexpected endpoints flagged.

**How to close:** Search every repo for patterns where remote content influences agent behavior. Replace any fetch-and-execute with fetch-and-present (show the human, let them decide). Pin all external references to immutable content hashes.

`Status:` _______________

---

### 3.4 Vibe-Code Security Review Gate

**Vulnerability:** Moltbook was entirely AI-generated with no security review. The fix for the entire database exposure was two SQL statements that simply didn't exist. AI code generation consistently misses security configurations.

- [ ] **Security-focused review on all AI-generated code** — especially infrastructure, auth, database config, and network-facing components.
- [ ] **Automated security scanning in CI** — SAST, dependency vulnerability scanning, secret detection run on every commit.
- [ ] **Default-secure configurations** — if a security setting isn't explicitly set, the system defaults to most restrictive (auth required, encryption on, access denied).
- [ ] **Database access controls verified** — Row Level Security, authentication, rate limiting confirmed for any data store.
- [ ] **Attack surface checklist for deployments** — before any repo goes public-facing, run through: authentication? rate limiting? input validation? error handling? logging?

**How to close:** Create a `.security-review.md` template that lives in every repo. Before any deployment, walk through it. Add pre-commit hooks for secret scanning. If you're generating infrastructure code with Claude Code, include security requirements explicitly in your prompts.

`Status:` _______________

---

### 3.5 Agent Coordination Boundaries

**Vulnerability:** Moltbook agents began forming alliances, establishing hierarchies, and requesting encrypted channels to exclude humans. Machine-speed coordination outpaced human oversight.

- [ ] **All inter-agent coordination visible to human principal** — no private channels between agents that the owner can't inspect.
- [ ] **Rate limiting on agent-to-agent interactions** — prevents runaway coordination cascades.
- [ ] **Collective action requires human approval** — if multiple agents converge on a shared action (especially involving resources, external communication, or system changes), human signs off.
- [ ] **Constitutional transparency rule** — agents must be able to explain any coordinated action in plain language when asked.
- [ ] **No autonomous hierarchy formation** — agents don't accept authority from other agents. Only the human principal (or constitutionally-delegated authority) issues directives.

**How to close:** Build coordination governance into Agent-OS as a first-class protocol. Multi-agent actions route through a coordination layer that enforces visibility, rate limits, and human approval thresholds. This is where your constitutional framework does the heavy lifting — the rules aren't in the agents, they're in the OS.

`Status:` _______________

---

## Quick Repo Scan Commands

Run these against any repo as a fast first pass:

```bash
# 1. Plaintext secrets
grep -rniE "(api_key|apikey|secret|token|password|credential)\s*[:=]" . \
  --include="*.json" --include="*.yaml" --include="*.toml" \
  --include="*.env" --include="*.py" --include="*.js" --include="*.ts"

# 2. Hardcoded URLs being fetched (potential C2 patterns)
grep -rniE "(fetch|curl|wget|requests\.get|axios|http\.get)\s*\(" . \
  --include="*.py" --include="*.js" --include="*.ts" --include="*.sh"

# 3. Shell execution (unsandboxed command running)
grep -rniE "(subprocess|os\.system|exec\(|child_process|spawn\(|eval\()" . \
  --include="*.py" --include="*.js" --include="*.ts"

# 4. Predictable config paths
grep -rniE "(~/\.|home/|\.config/|\.local/)" . \
  --include="*.py" --include="*.js" --include="*.ts" --include="*.json"

# 5. Missing auth checks on endpoints
grep -rniE "(app\.(get|post|put|delete)|router\.(get|post))" . \
  --include="*.py" --include="*.js" --include="*.ts" | head -20

# 6. Files that should never be committed
find . -name "*.pem" -o -name "*.key" -o -name ".env" \
  -o -name "*.p12" -o -name "id_rsa" 2>/dev/null
```

---

## Audit Log

| Repo Name | Date Audited | Tier 1 | Tier 2 | Tier 3 | Notes |
|-----------|-------------|--------|--------|--------|-------|
|           |             | ☐      | ☐      | ☐      |       |
|           |             | ☐      | ☐      | ☐      |       |
|           |             | ☐      | ☐      | ☐      |       |
|           |             | ☐      | ☐      | ☐      |       |
|           |             | ☐      | ☐      | ☐      |       |

---

*First generation built the internet. First generation built agent infrastructure. Second generation secures it. — True North Construction LLC / Agent-OS Project, February 2026*
