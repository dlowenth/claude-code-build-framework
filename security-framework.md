# Security Framework — Claude Code Build System Companion

## Purpose
This document defines a scalable security framework for applications built with the Claude Code Build System. It is designed to be assessed during the kickoff process and conditionally applied based on the project's security classification. Not every project needs every control — a personal tool with no external data is different from a system handling financial accounts.

This document is a companion to `claude.md`. Security sections referenced here should be incorporated into the project-specific `claude.md` during the kickoff process based on the project's classification.

---

## 1. Security Classification

Every project must declare a security tier during kickoff. The tier determines which controls from this framework are applied. The kickoff prompt should assess this automatically based on project context and recommend a tier.

| Tier | Criteria | Example Projects |
|---|---|---|
| **Tier 0: Minimal** | Internal tool, no auth, no sensitive data, no external APIs | CLI utilities, internal dashboards, static sites |
| **Tier 1: Standard** | Has auth, stores user data, connects to external APIs, but no financial or health data | SaaS tools, content platforms, project management apps |
| **Tier 2: Elevated** | Handles PII, financial data, health data, legal documents, or connects to accounts the user cares about (social media, email, CRM) | Client portals, CRM integrations, document management, analytics platforms |
| **Tier 3: Critical** | Can move money, access financial accounts, execute trades, manage credentials for high-value systems, or aggregate data whose exposure would cause severe harm | Financial platforms, payment systems, brokerage integrations, wealth management, crypto wallets, credential managers |

### What Each Tier Inherits
- **Tier 0:** Base `claude.md` security (RLS, no hardcoded secrets, `.env` handling)
- **Tier 1:** Tier 0 + Sections 2, 3, 7, 8, 12 from this document
- **Tier 2:** Tier 1 + Sections 4, 5, 6, 9, 10
- **Tier 3:** All sections, full framework

### Declaration
Add to PRD Section 10 (Security):
```
Security Tier: 0 / 1 / 2 / 3 (per security-framework.md)
Justification: <<WHY THIS TIER>>
```

If the project handles any of the following, it is automatically Tier 2 or higher:
- Financial account credentials or tokens
- Bank account data (balances, transactions, account numbers)
- Brokerage or trading platform connections
- Health records (HIPAA considerations)
- Legal documents with client PII
- Social Security numbers, government IDs
- Passwords or authentication credentials for other systems
- Data whose exposure would cause measurable financial or legal harm to users

**Plugin and MCP auto-escalation:**
If the project connects to any external system via MCP server, plugin, or API integration, it is automatically Tier 1 or higher. If any connected system contains user PII, client records, financial data, or authentication credentials, it is automatically Tier 2 or higher. This applies regardless of whether the project itself stores that data — the connection creates the attack surface. The plugin validation checklist (Section 12.1) is required for all projects with external plugin connections at Tier 1+.

If the project can **initiate transactions, move money, execute trades, or modify external account settings**, it is automatically Tier 3.

---

## 2. Threat Modeling (Tier 1+)

Every project at Tier 1 or above should identify its threat model during the planning phase. This doesn't need to be a formal document — a section in the PRD or `CONTEXT.md` is sufficient.

### Questions to Answer
1. **What does an attacker want from this system?** (Data theft? Account access? Financial gain? Disruption?)
2. **What is the most valuable data in the system?** (API credentials? User PII? Financial records? Business data?)
3. **What external services does the system connect to?** (Each connection is an attack surface)
4. **What can a compromised dependency do?** (Exfiltrate env vars? Make network calls? Read files?)
5. **What's the blast radius if the hosting infrastructure is compromised?** (What does Railway/Vercel/AWS have access to?)
6. **If AI agents are used, what can a prompt injection accomplish?** (Data exfiltration? Unauthorized actions? Privilege escalation?)

### Common Attack Vectors (Ranked by Likelihood for Web Applications)
1. **Supply chain attack** — Compromised npm/pip dependency exfiltrates environment variables or stored secrets. Most likely real-world attack vector for any application with dependencies.
2. **Infrastructure compromise** — Hosting provider or backend-as-a-service breached. Attacker gains access to database, environment variables, or stored credentials.
3. **Credential theft** — Session hijacking, phishing, or stolen device gives attacker access to a user's session.
4. **API credential exposure** — Keys leaked via client-side code, logs, error messages, or Git history.
5. **Injection attacks** — SQL injection, XSS, or (for AI systems) prompt injection that causes the system to take unauthorized actions.
6. **DNS hijacking / API redirect** — Attacker redirects API calls to a malicious server, capturing credentials on the first request.

The project-specific threat model should identify which of these apply and what mitigations are in place.

---

## 3. Network Allowlist (Tier 1+)

### Principle
Applications should only communicate with services they're designed to use. Any outbound request to an unexpected domain is either a bug or an attack.

### Implementation
Create a shared HTTP client utility that all outbound requests must use. This utility checks the destination domain against a hardcoded allowlist before making the request. Requests to non-allowlisted domains are blocked and logged.

```
// Conceptual — adapt to your HTTP client (fetch, axios, etc.)
const ALLOWED_DOMAINS = [
  '*.supabase.co',
  'api.example.com',
  // ... declared per project
];

function allowlistedFetch(url, options) {
  if (!isAllowlisted(url, ALLOWED_DOMAINS)) {
    logSecurityEvent('blocked_request', { url, caller: getCallerInfo() });
    throw new Error(`Outbound request blocked: ${new URL(url).hostname} not in allowlist`);
  }
  return fetch(url, options);
}
```

### Requirements
- The allowlist is declared in the PRD and hardcoded in the application (not configurable at runtime without elevated permissions)
- Every external API call in the codebase must use the allowlisted client — direct `fetch()` or `axios` calls to external URLs are prohibited
- Blocked requests are logged with: timestamp, destination domain, requesting code path, and user/agent context
- The allowlist is reviewed during the freeze audit

### PRD Section (Add to Section 10 or Section 8)
```
### Network Allowlist
| Domain | Purpose |
|---|---|
| `<<DOMAIN>>` | `<<PURPOSE>>` |

All outbound HTTP requests must use the allowlisted HTTP client.
Any domain not on this list is blocked.
```

### Freeze Audit Items
- [ ] All outbound HTTP requests use the allowlisted client utility — no direct `fetch()` or `axios` to external domains
- [ ] Blocked request logging is functional
- [ ] Network allowlist matches the PRD declaration

---

## 4. Credential Management (Tier 2+)

### Principles
1. **Store the minimum credential for the minimum time with the minimum permission.**
2. **Classify every credential by the damage it could cause if stolen.**
3. **Encrypt credentials at rest with strength proportional to their risk.**

### 4.1 Credential Minimization
- Request the minimum API permission scope that satisfies the application's needs. If you only need to read data, request a read-only key.
- If a service offers separate read and write API keys, use separate keys and store them with different security levels.
- Never store data you don't need for the application to function. If an API returns sensitive fields you don't use (full account numbers, SSNs, etc.), strip them before storage.
- Track credential age. Alert when rotation is due. Define rotation schedules in the PRD.

### 4.2 Credential Registry
Every project at Tier 2+ should maintain a `credential_registry` table (or equivalent) that tracks:
- Service name
- Credential type (API key, access token, OAuth token, secret)
- Where it's stored (Vault reference — never the credential itself)
- Permission scope (read-only, read-write, admin, etc.)
- Encryption tier (see Section 4.3)
- Creation date, last rotated, rotation due date
- Last accessed timestamp and access count

This provides visibility into what credentials the system holds, when they were last rotated, and how frequently they're used.

### 4.3 Tiered Credential Encryption
Not all credentials carry the same risk. A compromised analytics API key is annoying; a compromised payment API key is catastrophic. Encrypt accordingly.

**Background Tier (standard encryption — always accessible):**
For credentials with minimal risk if compromised. Public data APIs, analytics, error tracking, logging services. These can be accessed by scheduled jobs and background processes without human intervention.
- Stored in Supabase Vault (or equivalent encrypted secret store)
- Accessible via server-side code at any time
- No ceremony required

**Session Tier (hardware-key-encrypted — ceremony required):**
For credentials that access sensitive data (user PII, financial records, health data, client information). These are decrypted when an authorized user performs a ceremony and re-locked when the session expires.
- Stored in Vault, additionally encrypted with a key derived from a WebAuthn hardware key (YubiKey or equivalent)
- Decrypted only after a WebAuthn challenge-response ceremony
- Session duration configurable (hours, end-of-day, manual lock)
- While locked, dependent jobs queue and process when the session opens
- Dashboard shows session status (Active / Locked)

**Execution Tier (hardware-key-encrypted + action parameters):**
For credentials that can move money, execute transactions, modify external accounts, or take irreversible actions. Same encryption as Session Tier but additionally requires per-action parameters that bound what the credential can do.
- Decrypted only during an authorized action session with explicit parameters (max amount, max count, allowed targets, time limit)
- Session auto-terminates on parameter breach
- All actions logged to immutable audit trail

### 4.4 How Hardware Key Encryption Works
For projects that implement Session or Execution tier:
1. **Storage:** When a credential is first saved, the system initiates a WebAuthn ceremony (user taps hardware key). The resulting signature is fed through HKDF (Hash-based Key Derivation Function) to derive a symmetric encryption key. The credential is encrypted with this derived key, then stored in Vault. The derived key is discarded from memory.
2. **Retrieval:** When the credential is needed, the system initiates a new WebAuthn ceremony. The same hardware key produces the same key material (WebAuthn guarantees this for the same credential ID). The derived key decrypts the credential. It's used for the API call, then discarded from memory.
3. **Without the hardware key:** The credential in Vault is double-encrypted (Vault's own encryption + the hardware-derived key). Even a full Vault dump yields only ciphertext that requires the physical key to decrypt.

### 4.5 Development vs. Production
Hardware key ceremonies during development would be impractical — you'd be tapping a key hundreds of times per day. Use an environment flag to control enforcement:
```
CREDENTIAL_SECURITY_MODE=development  → All tiers use standard Vault only
CREDENTIAL_SECURITY_MODE=production   → Full tier enforcement
```
In development mode, the ceremony function logs "CEREMONY BYPASSED — development mode" and proceeds. Code paths remain identical. Development credentials should always be sandbox/test credentials with no access to real data.

### 4.6 Progressive Enforcement
For projects that start in development and graduate to production:
1. **During build:** All infrastructure for tiered encryption is built and tested. Ceremony flows work against mock credentials. Enforcement is bypassed via development mode.
2. **At production launch:** Background Tier credentials work immediately. Session Tier enforcement can be enabled after the system is stable and the ceremony UX is validated.
3. **When execution capability is added:** Execution Tier enforcement is mandatory from day one of any feature that can move money or take irreversible external actions.

### Freeze Audit Items (Tier 2+)
- [ ] `credential_registry` table exists and tracks all stored credentials
- [ ] No credential is past its rotation deadline
- [ ] All credentials have the minimum permission scope required
- [ ] No full account numbers, SSNs, or unnecessary sensitive data stored
- [ ] Development credentials are sandbox/test — not production credentials

### Freeze Audit Items (Tier 3 — if hardware key encryption is implemented)
- [ ] Session Tier credentials are encrypted with hardware-key-derived key
- [ ] Execution Tier credentials require ceremony + action parameters
- [ ] `CREDENTIAL_SECURITY_MODE` is set to `production` in deployment environment
- [ ] Ceremony bypass only active in development mode
- [ ] Hardware key recovery codes stored offline in a secure location

---

## 5. Action Tier System (Tier 2+)

### Principle
Not all user actions carry the same risk. Reading a dashboard is different from exporting data, which is different from initiating a payment. Authenticate proportionally.

### Tiers

| Tier | Risk Level | Examples | Authentication |
|---|---|---|---|
| **Passive** | Read-only, no side effects | View dashboards, read reports, browse data | Standard login (password + MFA) |
| **Sensitive** | Modifies data, changes configuration, accesses sensitive exports | Edit records, change settings, export PII, connect external accounts | Re-authentication within session (password + TOTP) |
| **Execution** | Moves money, executes transactions, modifies external account access, takes irreversible actions | Process payments, execute trades, change API connections, delete accounts, grant admin access | Full ceremony (password + TOTP + hardware key), verified within last 60 seconds |

### Implementation
- Classify every API endpoint and UI action into a tier
- Passive tier: standard session token is sufficient
- Sensitive tier: require re-authentication (password + TOTP) — the session token alone is not sufficient. Re-auth window: 5 minutes (configurable).
- Execution tier: require full ceremony via a single `executeWithCeremony()` function that all execution-tier actions must route through. No code path may bypass this function for execution-tier actions.

### PRD Section (Add to Section 10)
```
### Action Tiers
| Action | Tier | Authentication |
|---|---|---|
| `<<ACTION>>` | Passive / Sensitive / Execution | `<<REQUIREMENT>>` |
```

### Freeze Audit Items
- [ ] Every API endpoint is classified into an action tier
- [ ] Sensitive-tier actions require re-authentication
- [ ] Execution-tier actions route through `executeWithCeremony()` (if applicable)
- [ ] No execution-tier action can be performed without the full ceremony

---

## 6. Immutable Audit Logging (Tier 2+)

### Principle
If something goes wrong, you need to know exactly what happened, when, and who/what initiated it. The audit log must be tamper-proof — even an attacker with full database access cannot cover their tracks.

### Implementation
Create a `security_audit_log` table with **append-only RLS** — no role can UPDATE or DELETE rows. Only INSERT and SELECT are permitted.

```sql
-- RLS: append-only for all roles
CREATE POLICY audit_insert ON security_audit_log FOR INSERT TO authenticated
  WITH CHECK (true);

CREATE POLICY audit_select ON security_audit_log FOR SELECT TO authenticated
  USING (/* owner/admin role check */);

-- No UPDATE or DELETE policies — operations are denied by default
```

### What to Log
| Event Type | Trigger | Details to Capture |
|---|---|---|
| Authentication | Login, logout, MFA verification, failed attempt, lockout | User, IP, user agent, MFA method, success/failure |
| Credential access | Any read from credential store / Vault | Which credential, who/what accessed it, purpose |
| Sensitive action | Any Tier 2 action | User, action, re-auth verification, before/after state |
| Execution action | Any Tier 3 action | User, full ceremony record, action parameters, result |
| Blocked request | Network allowlist violation | Destination domain, requesting code path, user/agent context |
| External API call | Any outbound request to external service | Destination, payload hash (NOT payload), response code, initiator |
| Agent action | AI agent recommendation or data access (if applicable) | Agent ID, action type, data sources, reasoning summary |
| Canary trigger | Canary credential accessed or canary row queried | Full context of the access |

### Sensitive Data in Logs
- **Never log:** passwords, API keys, tokens, full account numbers, session tokens, or any credential value
- **Do log:** credential IDs/references, masked identifiers (last 4 digits), payload hashes, action descriptions
- **Log hashes, not content:** For API calls, log a SHA-256 hash of the request payload — this allows correlation without exposing data

### Freeze Audit Items
- [ ] `security_audit_log` table exists with append-only RLS (no UPDATE/DELETE policies)
- [ ] Authentication events are logged (login, logout, MFA, failed attempts)
- [ ] External API calls are logged (destination, response code, initiator)
- [ ] No sensitive data (credentials, passwords, full account numbers) appears in any log

---

## 7. Supply Chain Defense (Tier 1+)

### The Risk
Every npm and pip dependency is code that runs with full access to your application's environment variables, filesystem, and network. A single compromised package in your dependency tree can exfiltrate secrets, inject backdoors, or modify your application's behavior. This is the most likely real-world attack vector for web applications.

### Preventive Controls
- **Pin dependency versions.** Use lockfiles (`pnpm-lock.yaml`, `package-lock.json`, `requirements.txt` with pinned versions). Commit lockfiles to Git. Never use `^` or `~` version ranges in production dependencies for security-critical applications.
- **Audit before install.** Run `npm audit` (or `pnpm audit`) and `pip-audit` before every deployment. Fail the deploy if high or critical vulnerabilities are found.
- **Monitor continuously.** Enable Dependabot, Snyk, or Socket.dev to monitor for newly disclosed vulnerabilities in your dependency tree.
- **Review new dependencies.** Before adding any new dependency, check: maintainer reputation, download count, last publish date, open issues, and whether it runs post-install scripts. Tools like Socket.dev flag suspicious packages (install scripts, network access, obfuscated code).
- **Minimize the dependency tree.** Every dependency you don't add is a dependency that can't be compromised. Prefer built-in Node.js/Python functionality over third-party packages when the built-in is sufficient.

### Detective Controls
- **Network allowlist (Section 3)** catches compromised dependencies that try to phone home.
- **Canary credentials (Section 9)** detect if secrets are exfiltrated and used.
- **Audit logging (Section 6)** records all outbound API calls, so exfiltration attempts are visible in retrospect.

### Pre-Merge Hook (Recommended for Tier 2+)
Add a Git hook or CI check that blocks any dependency update which:
- Adds a post-install script
- Introduces a new direct dependency not on the pre-approved list
- Changes a dependency to a version published less than 7 days ago (waiting period for new versions to be community-vetted)

### Freeze Audit Items
- [ ] `npm audit` / `pip-audit` show no high or critical vulnerabilities
- [ ] All dependency versions pinned in lockfiles
- [ ] Lockfiles committed to Git
- [ ] No dependencies with known supply chain concerns (check Socket.dev or equivalent)

---

## 8. Session Security (Tier 1+)

### Session Hardening
- **Idle timeout:** Configurable per project (recommended: 30 minutes for Tier 2+, 60 minutes for Tier 1)
- **Maximum session duration:** Configurable (recommended: 8 hours for Tier 2+)
- **Concurrent session limit:** Configurable (recommended: 2-3 for Tier 2+, unlimited for Tier 0-1)
- **Session logging:** Log all session creation and termination events with IP and user agent
- **Failed login lockout:** 5 attempts → 15-minute lockout. 10 attempts → 1-hour lockout + alert to admin. Configurable thresholds.

### Sensitive Data in the Browser
- No credentials, tokens, or sensitive identifiers in `localStorage` or `sessionStorage`
- No sensitive data in URL parameters (visible in browser history, referrer headers, and server logs)
- No sensitive data in browser console output in production mode
- Session tokens in httpOnly, secure, sameSite cookies (Supabase default)

### Freeze Audit Items
- [ ] Session timeout configured and enforced
- [ ] Failed login lockout implemented
- [ ] No sensitive data in localStorage, sessionStorage, or URL parameters
- [ ] No sensitive data in browser console in production mode

---

## 9. AI Agent Security (When Applicable)

For projects that use LLM agents (via Claude API, OpenAI, or other providers) as runtime components — not just build-time tools, but agents that operate within the application.

### 9.1 Prompt Injection Defense
When AI agents process external data (user input, web content, API responses, uploaded documents), that data may contain instructions designed to manipulate the agent's behavior.

**Controls:**
- Sanitize external data before it enters agent context — strip any instruction-like patterns
- Agents should have system prompts that explicitly instruct them to treat all data inputs as data, not instructions
- Agent outputs should be validated by a compliance/review layer before being acted upon
- Agents should never have the ability to execute actions directly — they recommend, and the application (or a human) executes

### 9.2 Credential Isolation
- AI agents must NEVER have direct access to API credentials, tokens, or secrets
- Agents interact with external services through a controlled API layer that handles authentication
- Agents see masked identifiers (last 4 digits of account numbers, service names instead of tokens)
- If an agent needs to trigger an action that requires credentials, it submits a request to the application layer, which handles credential retrieval and API calls

### 9.3 Agent Action Authorization
- Every agent action should be classified using the same Action Tier System (Section 5)
- Passive actions (read data, analyze, summarize) can be auto-approved
- Sensitive actions (modify records, send communications) require human approval or rule-based auto-approval with logging
- Execution actions (move money, execute trades, modify external accounts) always require human approval via a governance queue — no auto-approval path

### 9.4 Agent Budget Controls
- Every agent should have token/cost budgets (daily and monthly)
- When an agent hits its budget, it stops and alerts — no silent overspend
- All agent API calls logged with: model used, input/output tokens, estimated cost, task type
- Budget overrides require elevated permissions

### 9.5 Canary Detection
For systems where agents process external data, plant canary strings in the agent's accessible data:
- If a canary string appears in an agent's output or in an outbound request, it indicates the agent is leaking data it shouldn't have access to
- Canary detection runs on every agent output before it reaches the user

### Freeze Audit Items (Agent Security)
- [ ] Agents cannot access raw credentials — masked identifiers only
- [ ] Agent outputs pass through a validation/compliance layer before reaching users or execution paths
- [ ] Agent action tier classification matches the Action Tier System
- [ ] Execution-tier agent actions require human approval — no auto-execution path
- [ ] Agent budget caps are configured and enforced
- [ ] Agent cost logging is active

---

## 10. Canary Detection (Tier 2+)

### Principle
Assume breach. Plant tripwires that alert you when someone accesses something they shouldn't.

### Types of Canaries
- **Credential canary:** A fake API key stored alongside real ones in your secret store. If it's ever used against any external service, someone has dumped your secrets. Use a service like Thinkst Canary or a custom monitoring endpoint.
- **Database canary:** A fake high-value row (e.g., a test account with a suspicious balance) that, if queried outside normal application patterns, triggers an alert.
- **Agent canary:** A planted string in agent-accessible data that should never appear in agent outputs. If it does, the agent is leaking data.
- **DNS canary:** A fake domain in your network allowlist monitoring that, if resolved, indicates unauthorized network scanning.

### Response
When a canary triggers:
1. Alert immediately (highest-priority notification channel available)
2. Log full context of the access to the immutable audit log
3. Optionally: trigger automatic credential rotation for the affected tier
4. Investigate the access path — was it a compromised dependency, infrastructure breach, or insider threat?

### Freeze Audit Items
- [ ] At least one canary credential deployed and monitored (Tier 2+)
- [ ] Canary trigger alerts are configured and tested (Tier 3)

---

## 11. Considerations by Project Type

### Financial Applications
- Physical gold, silver, GLD, and SLV ETFs are taxed as collectibles at 28%, not the standard 15-20% capital gains rate. Mining stocks and mining ETFs are standard equities. Any financial system that tracks precious metals must encode this distinction.
- Plaid's Transfer and Auth products can move money. NEVER request these products unless the application explicitly requires payment initiation. For monitoring-only applications, request only Transactions, Investments, and Balances.
- Brokerage API keys often come in read-only and trading flavors. Always use read-only keys for monitoring. Store trading keys at Execution Tier encryption.
- Wash sale rules apply across ALL accounts under the same tax ID. If the system tracks multiple accounts, it must monitor for cross-account wash sales.
- All monetary amounts must be stored as `decimal` / `numeric` types. Never `float` or `double precision`. Floating-point arithmetic produces rounding errors that compound in financial calculations.

### Health / HIPAA Applications
- Minimum Tier 2 classification, likely Tier 3 depending on data sensitivity
- Audit logging is legally required, not optional
- Data retention and deletion policies must be declared and enforced
- Business Associate Agreements (BAAs) required with all third-party services that handle PHI
- Session timeouts should be aggressive (15-20 minutes idle)

### Applications With Payment Processing
- Automatically Tier 3
- Payment credentials (Stripe secret keys, payment processor tokens) must be Execution Tier
- PCI-DSS compliance considerations — minimize the scope of what touches card data
- Webhook signature verification is mandatory for payment event handlers
- Idempotency keys required for all payment-related API calls

### Multi-User / Multi-Tenant Applications
- Cross-tenant data access is the most common authorization failure
- Every query must be tenant-safe even if the client is malicious (covered by base `claude.md` RLS rules)
- Audit logging should capture tenant context for every action
- API rate limiting should be per-tenant to prevent one tenant from exhausting shared resources

---

## 12. Plugin and MCP Security Validation (Tier 1+)

### Principle
Every plugin, MCP server, skill, or external tool connection is a trust decision. Validation happens before connection or loading, not after. There are two distinct input channels that shape AI behavior, and both require governance:
- **Capability inputs (plugins)** grant the AI the ability to act — calling tools, triggering hooks, executing scripts. A compromised plugin can exfiltrate data, inject prompts, harvest credentials, or exceed its stated purpose.
- **Knowledge inputs (skills)** instruct the AI on what to do — shaping its reasoning, decisions, and outputs through context. A malicious or poorly worded skill can instruct the AI to bypass security controls, expose credentials, or undermine governance.

The plugin validation checklist (Section 12.1) governs capability inputs. The skill security content audit (Section 12.3) governs knowledge inputs. Together, they form the two pre-execution security gates.

A plugin is a package that can contain distinct component types, each with a different risk profile:
- **Skills** — Instruction files (typically Markdown) that load into context when active. Skills are passive — they never execute code, never make network connections, never access external systems. However, skills directly shape AI behavior through instructions, which is its own attack vector: a malicious or poorly worded skill can instruct the AI to bypass governance, expose data, or behave in ways that conflict with the project's security tier. Skills carry two costs: context overhead (tokens consumed on every prompt — see `claude.md` Section 1.7) and instruction risk (what the AI is being told to do).
- **Tools** — Individual capabilities the AI can call. Each tool has a defined name, description, input parameters, and output. The AI reads the tool definition and decides during execution whether and when to invoke it. Risk: each tool is a capability you're granting the AI — what data it can access, what actions it can take.
- **Hooks** — System-triggered code that fires at specific workflow points (pre-tool-use, post-tool-use, session stop). The AI does not decide to run hooks — they fire automatically based on system conditions. Risk: hooks modify system behavior silently and can intercept tool calls, alter behavior, or execute code without AI or user awareness.
- **Scripts** — Code triggered by hooks or plugin infrastructure, running behind the scenes. The AI does not invoke scripts directly. Scripts are the plumbing — managing state, checking conditions, executing background operations. Risk: scripts run with whatever permissions the plugin has and execute without explicit AI or user initiation.
- **Commands** — User-invokable actions (e.g., slash commands). The user explicitly triggers these. Commands are the user-facing entry point that often configures hooks and scripts behind the scenes. Risk: a command may appear simple but set up persistent hooks or scripts that then run silently on every subsequent interaction.

Risk gradient for code execution: skills (none — passive) → tools (moderate — AI-initiated) → hooks/scripts (highest — system-initiated, silent). Risk gradient for instruction influence: skills shape behavior directly through context and must be audited for content (Section 12.3).

### 12.1 Plugin Validation Checklist
Before any plugin or MCP server is connected to a project, the following must be reviewed. This checklist is a required artifact for any build that includes plugin connections. It should be completed during the PRD review and architecture phases.

**Component Inventory (required first step):**
- [ ] Plugin components inventoried: which tools does it expose, which hooks does it register (and their trigger points), which scripts does it execute (and what triggers them), any bundled commands, any bundled skills
- [ ] For plugins with hooks or scripts: document what each hook triggers, what code each script executes, what files/network calls/state modifications scripts make. Plugins with hooks and scripts require deeper review than tool-only plugins.
- [ ] For plugins with commands: document what each command configures — does it set up hooks, scripts, or persistent state that runs silently after the command completes?
- [ ] For plugins with bundled skills: evaluate whether the skills are necessary for the current project or adding unnecessary context overhead. If skills are not needed, determine if they can be disabled or excluded.

**Source Verification:**
- [ ] Plugin source identified: manufacturer-direct, verified open-source organization, or community project
- [ ] For open-source plugins: verified author/organization, active maintenance history (commits within last 90 days), meaningful community adoption (stars, forks, downloads)
- [ ] Anonymous, newly created, or unmaintained sources flagged and require explicit owner approval before connection
- [ ] Plugin version pinned — no auto-updating to unreviewed versions

**Permissions Audit (tool-level, not just plugin-level):**
- [ ] Each tool the plugin exposes documented with its name, purpose, what data it can access, and what actions it can take
- [ ] Tools restricted to only those the agent actually needs — if a plugin exposes 15 tools but the agent only needs 2, the other 13 must be excluded from the agent's available tool set (via `allowedTools` in subagent definitions or equivalent filtering)
- [ ] Access scope matches stated function — a calendar integration requesting file system access is a red flag
- [ ] Principle of least privilege verified at the tool level — the agent has access to the minimum set of tools required for its purpose
- [ ] Any tools with write, delete, or modify capabilities explicitly justified — read-only tools preferred where sufficient

**Data Flow Mapping:**
- [ ] Data flow documented: does data stay in your tenant/project, or does it route through a third-party server?
- [ ] For plugins that proxy data through external servers: data handling, retention, and jurisdiction documented
- [ ] Sensitive data fields identified — which fields pass through the plugin, which are filtered or masked

**Credential Handling:**
- [ ] Credentials required by the plugin documented (API keys, OAuth tokens, service accounts)
- [ ] Credentials stored according to the project's security tier requirements (see Section 4)
- [ ] No credentials passed in plaintext in configuration files or environment variables visible to the plugin
- [ ] Credential scope minimized — read-only keys where possible, separate keys for separate permissions

**Auditability:**
- [ ] Tier 2+: All plugin interactions logged and auditable (requests sent, responses received, errors)
- [ ] Tier 1: Plugin interaction logging recommended
- [ ] Tier 0: Logging optional but encouraged
- [ ] Log content verified — no sensitive data (credentials, full account numbers) in plugin logs

**Compliance Verification (when applicable):**
- [ ] For plugins handling data subject to HIPAA, GDPR, PCI, or ITAR: compliance certifications verified against actual reports, not marketing claims
- [ ] SOC 2 or ISO 27001 certifications verified with report dates — expired or pending certifications do not count
- [ ] Business Associate Agreements (BAAs) executed where required (HIPAA)

### 12.2 Runtime Plugin Security
For plugins that operate at runtime in the built application (not just during the build process):

- **Treat tool responses as untrusted input.** A malicious plugin can embed hidden instructions in its response that may influence the agent's behavior (prompt injection via tool results). The application should include output validation to detect anomalous behavior after tool calls.
- **Agents must never receive raw credentials from plugin responses.** Plugin responses should be filtered through an application layer that strips or masks sensitive data before it enters the agent's context.
- **Plugin failures must not expose credentials or internal state.** Error handling for plugin calls should return generic error messages, not stack traces or connection strings.

### 12.3 Skill Security Content Audit
Skills shape AI behavior through instructions. A malicious skill can instruct the AI to bypass governance, and a poorly worded skill can create the same vulnerabilities inadvertently — the AI cannot distinguish between intentional malice and careless wording. This audit applies to all skills loaded into a project, whether externally sourced or internally authored. Internally authored skills are not exempt.

**When to audit:** On initial load and on every version update. If flagged patterns are found, the skill must not be loaded until the issues are reviewed and resolved by the builder.

**Scan skill instructions for:**
- [ ] Instructions to include, expose, or output credentials, API keys, tokens, or authentication data in any form
- [ ] Instructions to bypass, ignore, or override security tier classifications, governance controls, or build framework rules
- [ ] Instructions to send, transmit, or forward data to external endpoints, URLs, or email addresses not defined in the project's approved plugin architecture
- [ ] Instructions to suppress logging, avoid audit trails, or hide actions from the user or reviewer
- [ ] Instructions that grant the AI broader permissions or tool access than the project requires — overly broad language like "use any available tool" or "do whatever is needed" that undermines least privilege
- [ ] Instructions that conflict with the project's security tier — e.g., a Tier 1 project with a skill instructing the AI to handle data as if no restrictions exist

**Source classification:**
- **Externally sourced skills** (downloaded from public repositories, provided by third parties): Must be reviewed before loading, same standard as any plugin. Treat as untrusted until audited.
- **Internally authored skills**: Must still pass the content audit. The risk from poor wording is equivalent to the risk from malicious intent — the AI processes both identically.

**Distinction from MCP prompt injection:** A skill is a static, readable document — every instruction is visible and auditable before loading. An MCP prompt injection is dynamic — hidden instructions arrive at runtime in tool responses and cannot be pre-read. Both are real attack vectors, but skills are mitigated by pre-load content audit while MCP injection is mitigated by treating tool responses as untrusted input (Section 12.2).

### Freeze Audit Items (Plugin and Skill Security)
- [ ] Plugin validation checklist completed for every MCP server, plugin, and external tool connection
- [ ] Component inventory documented for each plugin (tools, hooks, scripts, commands, skills)
- [ ] All plugin sources verified (manufacturer-direct or verified open-source)
- [ ] Tool-level permissions enforced — agents only have access to the specific tools they need, not every tool a plugin exposes
- [ ] Plugins with hooks or scripts reviewed with elevated scrutiny — trigger conditions, executed code, and state modifications documented
- [ ] Data flow documented for each plugin — data routing and retention understood
- [ ] Plugin credentials stored per security tier requirements
- [ ] Skill security content audit completed for every skill loaded into the project
- [ ] No skills contain instructions to expose credentials, bypass governance, transmit data to unapproved endpoints, suppress logging, or grant excessive permissions
- [ ] Externally sourced skills reviewed before loading — treated as untrusted until audited
- [ ] Tier 2+: Plugin interaction logging active and auditable

---

## 13. Integration with claude.md and PRD Template

### Kickoff Prompt Addition
The kickoff prompt should assess security tier and include it in the Build Mode detection:

```
### Dimension 3: Security Classification
Assess the project's security tier per security-framework.md:
- Tier 0: No auth, no sensitive data, no external APIs
- Tier 1: Has auth, stores user data, connects to external APIs
- Tier 2: Handles PII, financial data, health data, or connects to accounts the user cares about
- Tier 3: Can move money, execute transactions, or aggregate data whose exposure causes severe harm

State the recommended tier and reasoning. The owner can override.
```

### PRD Template Addition
Add to PRD Section 10 (Security):
```
### Security Classification
- Security Tier: `<<0 / 1 / 2 / 3>>` (per security-framework.md)
- Justification: `<<WHY>>>`
- Network allowlist declared: Yes / No / N/A (Tier 1+)
- Credential registry required: Yes / No / N/A (Tier 2+)
- Action tier system required: Yes / No / N/A (Tier 2+)
- Hardware key encryption: Yes / No / N/A (Tier 2+ with sensitive credentials)
- Immutable audit log required: Yes / No / N/A (Tier 2+)
- AI agent security controls: Yes / No / N/A (if agents are used)
```

### claude.md Additions
For Tier 1+ projects, add to the project-specific `claude.md`:
- Network allowlist (Section 3 of this document) as a new section
- Supply chain defense rules (Section 7) added to the pre-deployment checklist

For Tier 2+ projects, additionally add:
- Credential management rules (Section 4) as a new section
- Action tier system (Section 5) with project-specific action classifications
- Immutable audit log requirement (Section 6)
- Canary detection (Section 10)

For Tier 3 projects, additionally add:
- Hardware key encryption for sensitive credentials (Section 4.3-4.6)
- Full ceremony requirement for execution-tier actions
- Project-type-specific considerations from Section 11

### Freeze Audit Additions
Append the applicable freeze audit items from each section above to the project-specific `claude.md` freeze audit checklist. Items are tagged by minimum tier — only include items for your project's tier and below.

---

## Versioning

| Field | Value |
|---|---|
| Version | 1.1 |
| Status | Active |
| Last Updated | 2026-04-12 |
| Companion To | claude.md v2.4+ |
| Changes from 1.0 | Added plugin and MCP auto-escalation triggers to Section 1. Added Section 12 (Plugin and MCP Security Validation) with component-level architecture: plugins inventoried by component type (tools, hooks, scripts, commands, skills), each with distinct risk profile and vetting approach. Two pre-execution security gates: plugin validation checklist (12.1) for capability inputs and skill security content audit (12.3) for knowledge inputs. Tool-level least privilege, hook/script transparency, runtime plugin security (12.2), and mandatory skill content scanning for governance-undermining instructions. Tier inheritance updated to include Section 12 for Tier 1+. Renumbered Integration section to 13. |
| Changes (v1.0) | Initial release. Tiered security classification, threat modeling, network allowlists, credential management with hardware key encryption, action tier system, immutable audit logging, supply chain defense, AI agent security, canary detection, project-type considerations. |
