# Agentic Security Audit v3.0

## March 2026

> Perform a layered security audit with attention to **agentic threat surfaces** — vulnerabilities that emerge when AI agents handle credentials, execute code, install plugins, or communicate with other agents on vibe-coded infrastructure.
>
> Aligned with [OWASP Top 10 for Agentic Applications (2026)](https://owasp.org/www-project-top-10-for-agentic-applications/).

---

## AUDIT LAYERS

```
┌──────────────────────────────────────────────┐
│  L1: PROVENANCE & TRUST ORIGIN               │
│  Was this code reviewed by someone who        │
│  understands what it does?                    │
├──────────────────────────────────────────────┤
│  L2: CREDENTIAL & SECRET HYGIENE             │
│  How are keys, tokens, and auth material      │
│  stored, scoped, and rotated?                │
├──────────────────────────────────────────────┤
│  L3: AGENT BOUNDARY ENFORCEMENT              │
│  What can agents do, access, and communicate  │
│  — and who authorized it?                    │
├──────────────────────────────────────────────┤
│  L4: SUPPLY CHAIN & DEPENDENCY TRUST         │
│  Where does external code/data come from?     │
├──────────────────────────────────────────────┤
│  L5: INFRASTRUCTURE & RUNTIME                │
│  Database, hosting, deployment, and network   │
│  security                                     │
└──────────────────────────────────────────────┘
```

Each layer gates the next — a critical failure at L1 means L2 findings are academic.

---

## L1: PROVENANCE & TRUST ORIGIN

### 1.1 Vibe-Code Detection

- [ ] **No tests**: Missing test files, test config, or CI test steps
- [ ] **No security config**: No `.env.example`, no secrets management, no auth middleware
- [ ] **AI boilerplate**: Tutorial-style comments, uniform formatting, TODO comments that read like prompts
- [ ] **Rapid commit history**: Massive initial commits, no iterative refinement, no security-related commits
- [ ] **Polished README, hollow codebase**: Marketing-grade docs with minimal inline documentation
- [ ] **Bloated deps**: Dependency count disproportionate to project complexity

**Severity:** Multiple indicators + credentials/PII/payments = **CRITICAL** — treat entire codebase as unreviewed.

### 1.2 Human Review Evidence

- [ ] Security-focused commits, threat model docs, access control design decisions
- [ ] Security tooling in CI/CD (semgrep, bandit, npm audit, etc.)
- [ ] `.gitignore` excludes `.env`, credentials, key files

### 1.3 The "Tech Preview" Trap

- [ ] Production traffic or real users despite "beta/preview/hobby" label?
- [ ] Real credentials handled without security review?
- [ ] Disclaimers shifting security responsibility without providing protective tools?

---

## L2: CREDENTIAL & SECRET HYGIENE

### 2.1 Secret Storage

- [ ] Plaintext credentials in files, DB schemas, config, or env files
- [ ] API keys visible in client-side JS/HTML/mobile bundles
- [ ] Deleted creds persisting in backups, logs, or filesystem
- [ ] Secrets in git history (run `gitleaks` or equivalent)
- [ ] `.env` files committed without `.env.example`

### 2.2 Credential Scoping & Lifecycle

- [ ] Keys scoped to minimum required permissions
- [ ] Rotation mechanism and schedule exists
- [ ] Per-user credential isolation (not shared master keys)
- [ ] Full credential delegation chain traced: storage → access → transmission → logging

### 2.3 Machine Credential Exposure

- [ ] OAuth tokens stored with same rigor as passwords
- [ ] API key as sole identity boundary between users
- [ ] Credential aggregation risk (single table with many users' keys)
- [ ] Key revocation path exists without losing account/data
- [ ] **Billing attack surface**: Can leaked keys run up costs with no spend limits or alerts?

---

## L3: AGENT BOUNDARY ENFORCEMENT

### 3.1 Agent Permission Model (OWASP ASI02, ASI03)

- [ ] Default permissions: ALLOW or DENY?
- [ ] Privilege escalation paths — can agents acquire ungranted permissions?
- [ ] File system, network, and command execution boundaries defined
- [ ] **Least-privilege enforcement**: Are agent capabilities bounded to task scope?
- [ ] **Human-in-the-loop gates**: Are destructive or high-privilege actions gated on human approval?

### 3.2 Prompt Injection Defense (OWASP ASI01)

- [ ] External inputs sanitized before inclusion in prompts
- [ ] Agent outputs validated against expected schemas before execution
- [ ] Clear separation between system instructions and user/external input
- [ ] **Multi-modal injection**: Hidden text in images, PDFs (white-on-white, base64-encoded), or audio processed by agents
- [ ] **Indirect injection via data sources**: Webpages, documents, or API responses containing embedded instructions

### 3.3 Memory Poisoning (OWASP ASI04)

- [ ] Does the agent maintain long-term memory across sessions?
- [ ] Is the source of each memory item tracked and attributable?
- [ ] Can accumulated context be audited and purged?
- [ ] Are memories from untrusted sources isolated from system context?
- [ ] **Persistent false beliefs**: Can poisoned memory cause the agent to defend incorrect security policies?
- [ ] **Gradual constraint drift**: Can sequential inputs slowly redefine what the agent considers "normal" behavior? (salami-slicing attacks)

### 3.4 Agent-to-Agent Trust

- [ ] Sending agent identity verified before accepting instructions
- [ ] Instructions from other agents treated as untrusted input
- [ ] Capability delegation logged and bounded
- [ ] **Cross-agent injection**: Can a compromised agent insert hidden instructions in output consumed by another agent?
- [ ] **ZombAI recruitment**: Can an agent be recruited into a botnet through text-based instructions that bypass traditional malware detection?

---

## L4: SUPPLY CHAIN & DEPENDENCY TRUST

### 4.1 Plugin/Skill Supply Chain (OWASP ASI06)

- [ ] Plugins/skills signed by authors with provenance verification
- [ ] Code reviewed (human or automated) before execution
- [ ] Permission manifest declared and enforced
- [ ] Agents cannot install plugins without human approval
- [ ] Download count manipulation / typosquatting defenses in place
- [ ] Update integrity verified against original author signature

### 4.2 MCP Server Trust

- [ ] All MCP servers enumerated (local and remote)
- [ ] Server authentication verified — connecting to legitimate endpoints
- [ ] Sensitive data flows through MCP servers identified
- [ ] Failure mode: fail-safe or fail-open on compromise?
- [ ] **Tool injection**: Can a malicious MCP server inject tools the agent uses unknowingly?
- [ ] **MCP sampling abuse**: Resource theft, conversation hijacking, or covert tool invocation via sampling feature (CVE-2025-6514 class)
- [ ] **Cross-server context abuse**: Can one MCP server influence prompts or tool calls routed to another?

### 4.3 Dependency Audit

- [ ] `npm audit` / `pip-audit` / equivalent run — HIGH and CRITICAL findings reported
- [ ] Dependencies not updated in 12+ months flagged
- [ ] Versions pinned (not floating on `latest`/`^`/`~`)
- [ ] Transitive dependencies audited, not just direct ones

---

## L5: INFRASTRUCTURE & RUNTIME

### 5.1 Database Security

- [ ] Row Level Security enabled and verified (don't assume — test it)
- [ ] Database not publicly accessible without authentication
- [ ] API key/connection string not in client-side code
- [ ] Read/write access properly separated
- [ ] Each sensitive table has verified access controls

### 5.2 BaaS Configuration

- [ ] Configured beyond defaults (Supabase, Firebase, Appwrite, etc.)
- [ ] Auth configured correctly; social auth redirects validated
- [ ] Storage bucket permissions locked down
- [ ] Edge/serverless functions validate inputs
- [ ] Realtime channels properly secured

### 5.3 Network & Hosting

- [ ] HTTPS everywhere
- [ ] CORS restricted to expected origins (not `*`)
- [ ] Rate limiting on API endpoints
- [ ] Error messages don't leak internals (stack traces, file paths, queries)
- [ ] Security events logged; logs stored securely

### 5.4 Deployment Pipeline

- [ ] CI/CD pipelines use pinned actions/images
- [ ] Secrets injected at runtime, not baked into artifacts
- [ ] Dev/staging/prod environments isolated
- [ ] Rollback capability exists

### 5.5 Regulatory Compliance

- [ ] **EU Cyber Resilience Act**: Secure-by-design principles followed, risk assessments conducted
- [ ] PII/medical/financial data handled per applicable regulations (GDPR, HIPAA, PCI-DSS)
- [ ] AI-generated code reviewed for regulatory blind spots (AI assistants are unaware of compliance requirements)

---

## REPORT FORMAT

```
AUDIT METADATA
  Project:       [name]
  Date:          [date]
  Auditor:       [model identifier]
  Commit:        [git hash]
  Strictness:    STANDARD | STRICT | MAXIMUM
  Context:       PROTOTYPE | INTERNAL | PRODUCTION | PUBLIC-INFRA

PROVENANCE ASSESSMENT
  Vibe-Code Confidence:   [0-100]%
  Human Review Evidence:  NONE | MINIMAL | MODERATE | STRONG

LAYER VERDICTS
  L1 Provenance:       PASS | WARN | FAIL | CRITICAL
  L2 Credentials:      PASS | WARN | FAIL | CRITICAL
  L3 Agent Boundaries: PASS | WARN | FAIL | CRITICAL
  L4 Supply Chain:     PASS | WARN | FAIL | CRITICAL
  L5 Infrastructure:   PASS | WARN | FAIL | CRITICAL
```

### Finding Format

```
[SEVERITY] — [TITLE]
Layer:     [1-5]
Location:  [file:line or component]
Evidence:  [what was found]
Risk:      [what could happen]
Fix:       [remediation steps]
```

### Severity Scale

| Severity | Definition | Response |
|----------|-----------|----------|
| **CRITICAL** | Active/imminent data exposure. Exploitable without auth. | Stop. Fix now. |
| **HIGH** | Significant vuln requiring specific conditions. | Fix within 24h. |
| **MEDIUM** | Defense-in-depth gap partially compensated by other controls. | Fix within 1 week. |
| **LOW** | Best practice violation. Theoretical risk. | Fix when convenient. |

---

## CONFIGURATION

```
Strictness:   STANDARD    # STANDARD | STRICT | MAXIMUM
Context:      PRODUCTION  # PROTOTYPE | INTERNAL | PRODUCTION | PUBLIC-INFRA
Focus:        ALL         # ALL | CREDENTIALS | AGENTS | SUPPLY-CHAIN | INFRASTRUCTURE
Skip-Layers:  NONE        # Comma-separated layer numbers, or NONE
```

---

## INCIDENT REFERENCES

| Incident | Date | Lesson |
|----------|------|--------|
| Moltbook DB exposure | Jan 2026 | Supabase RLS misconfigured, plaintext API keys, vibe-coded without review |
| OpenClaw supply chain | Jan 2026 | Unsigned skills, no code review, inflatable download metrics |
| Moltbook agent-to-agent | Feb 2026 | Agents executing instructions from untrusted agents |
| SCADA prompt injection | 2025-2026 | Hidden PDF instructions caused physical equipment damage |
| MCP sampling exploits | 2025-2026 | Resource theft, conversation hijacking via CVE-2025-6514 class |
| ZombAI botnet recruitment | 2026 | AI assistants recruited via text — no executables, no suspicious network traffic |

---

## VERSION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Dec 2025 | Initial agentic security audit prompt |
| 2.0 | Feb 2026 | Post-Moltbook rewrite. Layered architecture, vibe-code detection, agent boundaries, MCP trust. |
| 3.0 | Mar 2026 | Condensed format. Added: OWASP Agentic Top 10 alignment, multi-modal prompt injection, memory poisoning/sleeper agents, gradual constraint drift, MCP sampling abuse, cross-server context abuse, ZombAI recruitment, billing attack surface, regulatory compliance (EU CRA), cross-agent injection chains. Removed verbose narratives. |

---

## LICENSE

CC0 1.0 Universal — Public Domain

---

*"Agent usefulness correlates with access level. Sandboxing solves security but cripples functionality." — This audit makes the tradeoffs visible so humans can decide.*
