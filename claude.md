# Claude.md

## Purpose
This document defines the mandatory architectural, security, and execution framework for all new application builds using Claude Code.

It is not a PRD.
It is the governing build contract.

Every new project must comply unless explicitly overridden in writing.

---

# 0. Project Metadata Template (Required at Kickoff)

Every new project must declare the following before any planning begins. The PRD fills in these values; this section defines the required fields.

| Field | Value |
|---|---|
| Project Name | `<<APP_NAME>>` |
| Owner | `<<OWNER>>` |
| Date | `<<DATE>>` |
| Version | `<<VERSION>>` |
| Status | Draft / In Review / Approved |
| Expected User Scale | `<<SCALE>>` |
| Tenancy Model | Single-tenant / Multi-tenant (shared schema) / Multi-tenant (isolated) |
| Auth Provider | Discord OAuth (default) / Clerk / Other |
| Billing Provider | None / Clerk Billing / Stripe / Other |
| Backend | Supabase (default) |
| Frontend Framework | React (default) / Other |
| Deployment Target | Railway (default) / Other |
| Edge Functions | Supabase Edge Functions (default) / Other |
| External Integrations | `<<LIST>>` |
| Observability | None / PostHog / Sentry / Other (see Section 13.3) |
| Repository | GitHub (required) |
| Build Mode | Express Build / Full Build (see Section 1.5) |
| Development Framework | Superpowers / GSD / BMAD (see Section 20.9) |
| Security Tier | 0 / 1 / 2 / 3 (see security-framework.md) |

If any field is unresolved, it is an open question that must be answered before Phase 0 completes. All open questions from PRD Section 17 must be resolved during the kickoff conversation — Claude Code must verify zero unresolved items before beginning the plan.

---

# 1. Core Operating Principles

## 1.1 Plan-First Discipline
- No code before plan approval.
- Architecture must be locked before execution.
- Schema and authorization precede UI.
- No silent architectural drift.

## 1.2 Security > Velocity
- Authorization enforced at system boundary.
- UI permissions never replace backend enforcement.
- Multi-tenant boundaries must be provable.
- Console output must be minimal in production mode to prevent information leakage.

## 1.3 Deterministic Execution
- Phased builds only.
- Each phase must include manual verification steps.
- Freeze requires explicit audit checklist pass.

**Phase Gate Rule (Compaction-Proof):**
After completing each phase in a Full Build, **STOP**. Present verification results and wait for explicit approval before starting the next phase. Do not proceed to the next phase without human confirmation. This rule is permanent and must survive context compaction. **If you are unsure whether you should proceed, re-read CLAUDE.md — this instruction takes precedence over any compacted conversation context.**

This rule exists because phase gate instructions in conversation history are lost during context compaction. `CLAUDE.md` persists on disk and is re-read by Claude Code. Behavioral instructions that must survive the entire build must live here, not only in the kickoff prompt.

## 1.4 No Assumption Drift
Claude must stop and ask if ambiguity exists around:
- Tenancy model
- Role boundaries
- Ownership rules (who can create, edit, delete what)
- Calculation definitions, rounding rules, time periods, defaults
- Data retention rules, soft delete, and audit requirements
- Integration credentials
- Deployment configuration
- External service dependencies

## 1.5 Build Mode
Every project must declare a build mode. The build mode determines how much process structure is applied. Both modes enforce the same security, error handling, and code quality standards — the difference is execution ceremony, not safety.

### Express Build
For simpler applications (single-tenant, few entities, 1–2 roles, straightforward CRUD). Three steps:
1. **Plan and Confirm** — Architecture summary, data model, auth approach, screen list. Owner approves before code.
2. **Build** — Full application built in one pass. Priority order: auth → data structures → business logic → API → UI. No per-phase gates.
3. **Verify and Harden** — Manual verification against the freeze audit checklist. Fix issues. Ship.

Express Build still requires: plan approval before coding, stop-and-ask on ambiguity, and a passing freeze audit before shipping.

### Full Build
For complex applications (multi-tenant, 6+ entities, 3+ roles, calculation engines, integrations, compliance). Standard Phase 0–8 execution with verification gates at each phase.

### Build mode is declared in the PRD. If not declared, the kickoff process will assess complexity and recommend one.

## 1.6 Deterministic Over Probabilistic
When a task can be accomplished by a deterministic script (Python, SQL, bash), it must not be delegated to LLM reasoning. AI handles orchestration, planning, and judgment. Scripts handle data transformation, file operations, API calls, and calculations. Every additional probabilistic step compounds error rates — five steps at 90% accuracy each yields 59% overall success. This principle governs Section 18.4 (When NOT to Use an LLM) and applies broadly: if the output is fully predictable from the input, use a script.

## 1.7 Context Efficiency
Only load skills, plugins, and MCP connections required for the active project. Remove or disable anything not directly relevant to the current build. Every skill file, MCP tool definition, and plugin registration consumes tokens and occupies model attention on every prompt — whether it's being used or not. Unnecessary context reduces output quality, increases cost, and compounds over long sessions because static context (CLAUDE.md, skills, tool definitions) is never compacted while conversation history is. Context efficiency is a quality and cost discipline, not just a performance optimization.

---

# 2. Default Technology Stack

The following stack is the default for all projects unless the PRD explicitly declares a deviation.

| Layer | Default | Notes |
|---|---|---|
| Frontend | React | — |
| Backend / Database | Supabase | Postgres + Auth + Realtime + Storage |
| Auth | Discord OAuth | Via Supabase Auth; roles stored internally. Alternative: Clerk (see Section 2.2) |
| Billing | None | If needed: Clerk Billing (if using Clerk for auth) or Stripe |
| Edge Functions | Supabase Edge Functions | Server-side logic, webhooks, integrations |
| Deployment | Railway | Via Railway CLI or dashboard |
| Local Development | localhost | All testing occurs locally before deploy |
| Version Control | Git + GitHub | Required for all projects |
| Observability | Per PRD | Analytics, error tracking, session replay (see Section 13.3) |
| State Management | Per PRD | Declared in PRD |
| UI Component Library | Per PRD | Declared in PRD |

### 2.1 Pre-Approved Dependencies
The following packages are pre-approved and do not require explicit approval per project:
- `@supabase/supabase-js` — Supabase client
- `@clerk/clerk-react` — Clerk auth client (if using Clerk as auth provider)
- `react-router-dom` — Routing
- `tailwindcss` — Styling (if declared in PRD)
- `lucide-react` — Icons
- `date-fns` — Date manipulation
- `zod` — Schema validation
- `zustand` — Lightweight state management (if declared in PRD)
- `recharts` — Charting (if needed)
- `posthog-js` — Analytics and error tracking (if declared in PRD)
- `@playwright/test` — End-to-end testing (if declared in PRD, see Section 14.6)

Any dependency not on this list requires explicit approval before installation. Claude must state the dependency name, its purpose, and its bundle size impact before requesting approval.

### 2.1.1 Pre-Approved Python Packages (Batch Processing)
When building batch scripts per Section 18.5, the following Python packages are pre-approved:
- `anthropic` — Anthropic API client
- `requests` — HTTP client
- `aiohttp` — Async HTTP client (for parallel batch operations)
- `python-dotenv` — Environment variable loading
- `csv` / `json` (stdlib) — Data format handling
- `pandas` — Data manipulation (if dataset size warrants it)
- `tqdm` — Progress bars for long-running operations

### 2.2 Authentication Provider
Default OAuth provider: **Discord** via Supabase Auth.

Alternative: **Clerk** — recommended when the application requires subscription billing, consumer-facing sign-up flows (email/password, social providers, passkeys), prebuilt UI components, or GDPR compliance tooling. Clerk integrates with Supabase as a third-party auth provider, preserving RLS and Edge Functions.

Requirements (all auth providers):
- Auth provider must be primary authentication unless explicitly changed.
- Roles must not rely solely on OAuth claims.
- Application roles must be stored internally in database tables.
- Session strategy must be declared in PRD.

### 2.2.1 Clerk + Supabase Integration Pattern
When using Clerk as the auth provider with Supabase as the backend:

**How it works:** Clerk authenticates users and issues JWTs. Supabase is configured to accept Clerk as a third-party auth provider via JWKS endpoint. Supabase validates incoming JWTs using Clerk's public keys. RLS policies use `auth.jwt()` to read Clerk session claims.

**Key differences from Supabase Auth:**
- Clerk uses **string-based user IDs** (e.g., `user_2abc123`), not UUIDs. The `auth.uid()` function cannot be used in RLS policies. Instead, extract the user ID from JWT claims: `auth.jwt() ->> 'sub'`.
- RLS policies must reference Clerk's JWT claims instead of Supabase Auth's built-in functions.
- The `onAuthStateChange` two-effect pattern (Section 5.2.3) does not apply — Clerk manages its own session lifecycle. Use Clerk's `useAuth()` and `useUser()` hooks instead.
- The `getSession()` vs `getUser()` distinction (Section 5.2.2) does not apply — Clerk's `useSession()` hook handles token management.
- Edge Functions still work — pass the Clerk session token in the Authorization header. The function validates it against Clerk's JWKS.

**RLS policy pattern with Clerk:**
```sql
-- Extract Clerk user ID from JWT
CREATE OR REPLACE FUNCTION requesting_user_id()
RETURNS TEXT AS $$
  SELECT auth.jwt() ->> 'sub';
$$ LANGUAGE SQL STABLE;

-- Example RLS policy using Clerk user ID
CREATE POLICY users_select ON public.users FOR SELECT USING (
  clerk_user_id = requesting_user_id()
);
```

**Environment variables required (in addition to Supabase vars):**
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` — Clerk publishable key
- `CLERK_SECRET_KEY` — Clerk secret key (server-side only)

When the PRD declares Clerk as auth provider, the project-specific `claude.md` should replace the Supabase Auth-specific sections (5.2.2, 5.2.3, 4.2.1) with the Clerk patterns above.

### 2.3 Identity Source of Truth
Default: **Supabase Auth** (`auth.users`).

Every project must create:
- `public.profiles` — User profile data linked to `auth.users.id` (or Clerk user ID if using Clerk)
- `public.tenant_memberships` (if multi-tenant) — Tenant/role/status junction

Identity provider may vary per project if declared in PRD:
- Supabase Auth (default)
- Clerk (with Supabase as backend — see Section 2.2.1)
- Custom JWT system
- OAuth-only session
- Other managed auth system

---

# 3. Tenancy Model (Dynamic by Project)

Not all projects require multi-tenancy.
The PRD must explicitly declare one of the following:

1. **Single-tenant application**
2. **Multi-tenant (shared schema)** — Recommended default if multi-tenancy is needed. Single Supabase project, `tenant_id` on every tenant-scoped table, enforced by RLS.
3. **Multi-tenant (isolated environments)** — Separate schemas or projects per tenant. Only when compliance or contractual requirements demand it.

If multi-tenant:
- Tenant boundary must be enforceable at backend level.
- Tenant isolation must be testable.
- All tenant-scoped entities must reference `tenant_id` or equivalent.
- Every query must be tenant-safe even if the client is malicious.
- Internal access model must be declared (assigned tenants only vs. global).
- Impersonation policy must be declared (allowed yes/no, audit required yes/no).

No tenant assumptions may remain implicit.

---

# 4. Authorization Enforcement Layers

Authorization must exist in three layers where applicable:

1. **Data layer (RLS policies are source of truth)**
2. **API / RPC / Edge Function logic** — Respects the same checks
3. **UI visibility layer** — Hides features but is never security

Data layer is source of truth. UI-only enforcement is a critical defect.

### 4.1 RLS Strategy (Mandatory for Tenant-Scoped Data)
- RLS must be enabled on every tenant-scoped table.
- Every tenant-scoped table must include `tenant_id`.

**Policy patterns:**
- SELECT: allow if user is active member of same tenant
- INSERT: allow if user can write in that tenant and `tenant_id` matches
- UPDATE: allow if user can update in that tenant and `tenant_id` matches
- DELETE: allow only for tenant admins (and optionally superuser)

### 4.2 Helper Functions (Required)
Create SQL helper functions to centralize authorization logic:
- `is_superuser(uid)` — Internal admin check
- `is_active_member(uid, tenant_id)` — Active membership check
- `has_tenant_role(uid, tenant_id, role)` — Role-specific check
- `can_write(uid, tenant_id)` — Write permission based on role rules

RLS policies must call these helper functions. Inline policy logic that duplicates authorization checks is prohibited.

### 4.2.1 RLS Policy Rules for Supabase + Discord OAuth
The following rules are mandatory for any project using Supabase Auth with Discord OAuth. These address known issues where first-login flows and policy evaluation fail silently.

**Rule 1: Table-level GRANTs are required.**
RLS controls which rows are visible, but Postgres roles need base table-level privileges first. Without explicit GRANTs, all queries return `403 permission denied for table`, even if RLS policies would allow access.

Every public table protected by RLS must include:
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON public.<<TABLE_NAME>> TO anon, authenticated;
```
This must be in the initial schema migration, not added as a fix later.

**Rule 2: First-login SELECT policies must handle the unlinked state.**
When a user authenticates via Discord OAuth for the first time, their internal user record may exist (e.g., pre-provisioned or created by a trigger) but the `auth_user_id` column is still NULL because the linking step hasn't completed yet. Standard RLS policies that check `auth_user_id = auth.uid()` will fail on first login because there's nothing to match.

SELECT policies on user tables must include a fallback condition that matches by `discord_user_id` (or equivalent provider ID) against the Discord provider ID from the JWT, so the row is findable before `auth_user_id` is populated:
```sql
-- Example: users_select policy with first-login fallback
CREATE POLICY users_select ON public.users FOR SELECT USING (
  is_admin(auth.uid())
  OR auth_user_id = auth.uid()
  OR discord_user_id = (auth.jwt() -> 'user_metadata' ->> 'provider_id')
);
```

**Rule 3: RLS policies must NEVER subquery `auth.users`.**
The `authenticated` Postgres role does not have SELECT access to the `auth.users` system table. If an RLS policy contains a subquery like `SELECT ... FROM auth.users`, the entire policy evaluation fails with a 403 — not just the subquery, but the whole request.

Always use `auth.jwt()` to read user metadata (provider ID, email, etc.) directly from the JWT token claims:
```sql
-- CORRECT: read from JWT claims
auth.jwt() -> 'user_metadata' ->> 'provider_id'

-- WRONG: subquery on auth.users (will fail for authenticated role)
(SELECT raw_user_meta_data ->> 'provider_id' FROM auth.users WHERE id = auth.uid())
```

These three rules must be verified during Phase 1 (Auth + Authorization) and are included in the Freeze Audit Checklist.

### 4.3 Permissions Matrix (Required in PRD)
Every PRD must include a completed permissions matrix. Example structure:

| Capability | superuser | coach | client_admin | member |
|---|---|---|---|---|
| View tenant data | ✓ | Assigned only | Own tenant | Own tenant |
| Create records | ✓ | ✓ | ✓ | ✓ |
| Edit records | ✓ | ✓ | ✓ | Own only |
| Delete records | ✓ | — | ✓ | — |
| Manage users | ✓ | — | ✓ | — |
| Export data | ✓ | ✓ | ✓ | — |

Roles are project-specific and must be defined in the PRD.

### 4.4 Security Framework (Companion Document)
For projects classified at Security Tier 1 or above (per `security-framework.md`), additional security controls apply beyond the base authorization enforcement defined in this section. The security framework provides tiered controls for threat modeling, network allowlists, credential management with hardware key encryption, action tier classification, immutable audit logging, supply chain defense, session hardening, AI agent security, and canary detection. The applicable sections are determined by the project's security tier and incorporated into the project-specific `claude.md` during kickoff. **The security framework file (`security-framework.md`) must be attached during the kickoff process for all projects.**

---

# 5. Backend Architecture Requirements

Default backend: **Supabase** (Postgres + Auth + Realtime + Storage + Edge Functions).

Regardless of stack, the following must be true:

- Authorization enforced server-side (RLS + Edge Functions)
- No trust of client-provided identifiers
- Minimal data returned per request
- Explicit role checks
- Logging for privileged operations

### 5.1 Database-Level Security
- All protected tables must have RLS enabled
- No recursive policy logic
- No circular authorization dependencies
- Index coverage must support filtered queries
- Helper functions used for consistent policy evaluation

### 5.2 Edge Functions
- Enforce membership and role checks server-side
- Do not accept client-provided `tenant_id` without verifying membership
- Return minimal data
- Include error handling with appropriate HTTP status codes

### 5.2.1 Edge Function JWT Verification (Supabase)
Supabase Edge Functions have **gateway-level JWT verification enabled by default**. The gateway validates the JWT before the function code runs. If the token is expired, malformed, or mismatched, the gateway returns a 401 and the function code never executes.

**Rule: Deploy Edge Functions with `--no-verify-jwt` when the function performs its own authentication internally.**

If the Edge Function already validates the user via its own auth logic (e.g., `getAuthUser()`, `requireAdmin()`, checking `supabase.auth.getUser()` inside the function), the gateway-level check is redundant and creates a failure point that's hard to debug — the 401 comes from the gateway, not your code, and your function's error handling never fires.

```bash
supabase functions deploy <<FUNCTION_NAME>> --no-verify-jwt
```

This is safe **only** when the function performs its own auth check. If a function does not validate auth internally, do NOT use `--no-verify-jwt`.

Document the deployment flag for every Edge Function in the PRD or in a deployment manifest in the repository.

### 5.2.2 Client-Side Auth Token Handling (Supabase JS v2)
Supabase JS v2 has two auth methods with critically different behaviors. Using the wrong one causes cascading session loss.

**`supabase.auth.getSession()`** — Returns a **cached** local session. No server round-trip. No side effects. The Supabase client's `autoRefreshToken: true` handles token refresh automatically in the background.

**`supabase.auth.getUser()`** — Makes a **server round-trip** to validate the token. On transient network failures, it can trigger `SIGNED_OUT` events that cascade into app-wide session loss and wipe user state.

**Rule: Use `getSession()` as the default for all client-side auth checks and before API calls.** The Supabase client's automatic token refresh ensures the cached session has a valid JWT. Do NOT use `getUser()` for routine auth checks — the server round-trip creates a failure mode that is worse than a stale token.

```js
// CORRECT: cached session, no side effects, auto-refreshed
const { data: { session } } = await supabase.auth.getSession();
if (!session) redirectToLogin();

// WRONG: server round-trip can trigger SIGNED_OUT on transient failure
const { data: { user } } = await supabase.auth.getUser();
```

`getUser()` should only be used in rare cases where server-side token validation is explicitly required (e.g., a one-time security-critical action). It must never be used in routine auth flows, API call helpers, or anywhere it runs on a regular cadence.

### 5.2.3 Supabase Auth Initialization Pattern (Mandatory)
The Supabase JS v2 client does not fully commit the session until the `onAuthStateChange` callback returns. Any authenticated database query made inside the callback will deadlock because the session token isn't available yet. This is the single most common source of infinite loading states and app hangs.

**Rule 1: Never `await` async operations inside `onAuthStateChange` callbacks.**
Use the callback only for synchronous state updates. Run database lookups in a separate `useEffect` that reacts to the state change.

```jsx
// WRONG — will deadlock
onAuthStateChange(async (event, session) => {
  const { data } = await supabase.from('users').select('*');  // hangs forever
});

// CORRECT — two-effect pattern
// Effect 1: Subscribe to auth changes. Synchronous state updates only.
onAuthStateChange((event, session) => {
  setSession(session);           // synchronous only
  setAuthUser(session?.user);    // triggers Effect 2
});

// Effect 2: React to auth user state. DB queries run here.
useEffect(() => {
  if (authUser) lookupUser(authUser);
}, [authUser]);
```

**Rule 2: On `TOKEN_REFRESHED` events, skip redundant DB lookups.**
If the app already has a valid user loaded, re-running the user lookup on every token refresh is wasteful and risks wiping user state if the query fails transiently. Only perform user lookup when the user is not yet loaded or on `SIGNED_IN` events.

**Rule 3: The two-effect pattern is mandatory for all auth initialization.**
Every application using Supabase Auth must implement:
- **Effect 1:** Subscribe to `onAuthStateChange`. Set session and auth user state synchronously. No database queries.
- **Effect 2:** React to auth user state changes. Perform user lookup (e.g., query `public.users`) outside the auth callback scope.

This pattern must be documented in the PRD under Auth Flow and verified during Phase 1.

### 5.3 Storage Buckets (If Using Supabase Storage)
- Object paths must include `tenant_id/...`
- Bucket policies require active membership for the tenant in the path
- Do not allow public read on sensitive content
- Document which buckets are public vs. private in PRD

---

# 6. Business Logic Integrity (DRY Requirement)

All calculations must:
- Live in a single logic layer (dedicated calculation module)
- Be reusable across UI, reporting, exports
- Define rounding rules
- Define time zone assumptions
- Define period boundaries

Inline UI math beyond formatting is prohibited.

Disagreement in numbers across app = critical defect.

### 6.1 Calculation Module Structure
Each calculation must define:
- Inputs (with types and validation)
- Outputs (with types)
- Rounding and precision rules
- Time period assumptions
- Edge case behavior

The UI, reporting, and export layers consume results from the calculation layer only.

---

# 7. Deterministic Build Order

The build order depends on the declared build mode (see Section 1.5).

### Full Build — Phase Structure

| Phase | Name | Gate |
|---|---|---|
| Phase 0 | Architecture Lock | Plan approved, metadata complete |
| Phase 0.5 | Pre-Build Setup + Discuss | Human completes pre-build manual steps. Implementation decisions captured in CONTEXT.md. |
| Phase 1 | Auth + Authorization | Login works, RLS enforced, roles verified |
| Phase 2 | Core Data Structures | Schema deployed, migrations recorded |
| Phase 3 | Business Logic Layer | Calculation module built and verified |
| Phase 4 | API / RPC / Edge Functions | Server-side logic verified |
| Phase 5 | UI Implementation | Screens built, connected to backend |
| Phase 6 | Notifications / Realtime | If applicable |
| Phase 7 | Reporting / Exports | If applicable |
| Phase 8 | Hardening + Freeze Audit | Full audit checklist passed |

UI may not precede backend enforcement.

Each phase must output:
- Scope
- Files created or modified
- Acceptance criteria
- Manual verification steps
- Known limitations
- Any deviations from plan (explicit)

### Express Build — Sequence

| Step | Name | Gate |
|---|---|---|
| Step 1 | Plan and Confirm | Owner approves architecture, data model, and screen list |
| Step 1.5 | Pre-Build Setup + Discuss | Human completes pre-build manual steps. Implementation decisions captured in CONTEXT.md. |
| Step 2 | Build | Full app built in one pass: auth → data → logic → API → UI |
| Step 3 | Verify and Harden | Freeze audit checklist passed |

Even in Express Build, the internal priority order (auth before UI, backend before frontend) must be followed. Claude Code does not need to stop between these stages, but must not invert the order.

### Pre-Build Scaffolding (Both Build Modes)
Before any application code is written, the following must be completed:

1. **Plan approved** by project owner (both modes)
1a. **All open questions resolved** — the PRD's Section 17 must contain zero unresolved open questions. All items must have been worked through during the kickoff conversation and resolutions folded back into the PRD and project-specific `claude.md`. If the PRD arrives at Claude Code with unresolved open questions, Claude Code must stop and surface them before planning.
2. **Setup guides generated** in `docs/resources/` using the three-category model (Section 8.8). Category 1 (human-only) and Category 2 (automated) steps documented. **`.env.example`** created with all variables documented (Section 8.3.1).
3. **Human completes all Category 1 (human-only) setup steps** — account creation, new project creation, `.env` population with base credentials. **HARD GATE: Claude Code must present the Category 1 checklist from `docs/resources/README.md` and ask the human to confirm each item is complete before proceeding to the build.** Claude Code must not start writing application code (Phase 1 / Step 2) until the human explicitly confirms. If any item is not done, Claude Code must wait.
4. **`.env` verified** — Claude Code must check that `.env` exists and contains values for every variable listed in `.env.example`. If any variable is empty or missing, surface it to the human and do not proceed.
4a. **MCP/CLI service connections configured** — For services with MCP or CLI support (Supabase, Railway, etc.), Claude Code configures the connections scoped to the new project. Verify each connection targets the correct project before any write operations. See Section 8.8.2 for safety rules.
5. **`.npmrc`** with `force=true` created (Section 8.3.2)
6. **`.claude/settings.json`** created with auto mode permission configuration and hooks enforcement (Section 20.6). Add `.claude/settings.local.json` to `.gitignore`.
7. **`.claude/hooks/`** populated with enforcement scripts (Section 20.5)
8. **`.claude/agents/`** populated with custom subagents (Section 20.3.3, Full Build only)
9. **Development framework and plugins installed** based on framework selection (Section 20.9):
   - **If Superpowers:** `/plugin install superpowers@claude-plugins-official`
   - **If GSD:** `npx get-shit-done-cc --claude --local`
   - **If BMAD:** `npx bmad-method install`
   - **All frameworks:** `/plugin install context7` and `/plugin install frontend-design@claude-plugins-official` (for UI projects)
   - Document installed framework and plugins in `lessons-learned.md`.
10. **`STATE.md`** created in project root with initial build state (Section 20.7)
11. **Implementation Decisions Captured** via Discuss Phase (Section 20.8) — Claude Code asks targeted questions about how the owner envisions the implementation before any code is written
12. **`git init`** and initial commit with scaffolding files

If a plugin is discovered mid-build, install it at the next phase gate and do a refinement pass rather than interrupting the current phase.

---

# 8. Environment and Deployment Strategy

### 8.1 Current Model
No formal dev/staging/production separation exists.
Version control via Git + GitHub is the primary safety mechanism.

### 8.2 Deployment Defaults
- **Local development:** localhost for all testing
- **Production deployment:** Railway (using Railway CLI or dashboard)
- **Backend services:** Supabase (single project per application)
- **Edge Functions:** Deployed to Supabase

### 8.3 Environment Variables
- All secrets and configuration values must use environment variables.
- No hardcoded credentials, API keys, or connection strings in source code.
- `.env` files must be `.gitignore`-d.
- Railway environment variables must be configured via Railway dashboard.
- Supabase connection strings, anon keys, and service role keys must be environment variables.
- PRD must list all required environment variables and their purpose.

### 8.3.1 `.env.example` File (Required)
Every project must include a `.env.example` file in the project root that documents every environment variable the application needs. This file is committed to Git (it contains no secrets) and serves as the setup guide for anyone configuring the application for the first time.

**Rule: `.env.example` must be created during project scaffolding, before any code that depends on environment variables is written.** It must be kept in sync as new integrations are added throughout the build.

The file must include:
- Every required environment variable with a placeholder value
- A comment explaining what each variable is, where to get it, and whether it's required or optional
- Grouping by service/purpose for readability
- Notes on which variables are client-side safe (public) vs. server-side only (secret)

**Template structure:**
```env
# ============================================
# Application Configuration
# ============================================
# Copy this file to .env and fill in your values.
# NEVER commit the .env file — it contains secrets.
# ============================================

# --- Supabase ---
SUPABASE_URL=https://your-project.supabase.co          # Supabase project URL (Supabase dashboard → Settings → API)
SUPABASE_ANON_KEY=your-anon-key-here                    # Supabase anonymous/public key (safe for client-side)
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here     # Supabase service role key (SERVER-SIDE ONLY — never expose to client)

# --- Auth: Discord OAuth (if using Discord) ---
DISCORD_CLIENT_ID=your-discord-client-id                # Discord developer portal → Application → OAuth2
DISCORD_CLIENT_SECRET=your-discord-client-secret         # Discord developer portal → Application → OAuth2 (SERVER-SIDE ONLY)

# --- Auth: Clerk (if using Clerk instead of Discord) ---
# NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_xxx          # Clerk dashboard → API Keys (safe for client-side)
# CLERK_SECRET_KEY=sk_live_xxx                           # Clerk dashboard → API Keys (SERVER-SIDE ONLY)

# --- Railway ---
# Railway environment variables are configured via the Railway dashboard,
# not in this file. This section documents what must be set there.
# Set all variables from this file in Railway dashboard → Variables.

# --- Anthropic (if app makes runtime LLM calls) ---
# ANTHROPIC_API_KEY=sk-ant-xxx                           # Anthropic console → API Keys (SERVER-SIDE ONLY)

# --- Observability (if using PostHog) ---
# NEXT_PUBLIC_POSTHOG_KEY=phc_xxx                        # PostHog dashboard → Project Settings (safe for client-side)
# NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com       # PostHog instance URL

# --- Additional Integrations ---
# Add project-specific variables below with the same format:
# VARIABLE_NAME=placeholder-value  # Description (source) (CLIENT-SAFE or SERVER-SIDE ONLY)
```

**Rules:**
- Variables prefixed with `NEXT_PUBLIC_` or `VITE_` are client-side safe and will be bundled into the frontend build. All other variables must be treated as server-side only.
- Comment out optional sections (Clerk, Anthropic, PostHog) with `#` and include them as templates so they're ready to uncomment when needed.
- When a new integration is added during the build, the `.env.example` must be updated in the same commit.
- The `.env.example` header must include the instruction to copy to `.env` and a warning to never commit `.env`.

`.env.example` is committed to Git. `.env` is gitignored. This distinction is critical — `.env.example` is documentation, `.env` contains secrets.

### 8.3.2 Cross-Platform Build Fix (Windows → Railway/Linux)
When developing on Windows and deploying to Railway (Linux), `package-lock.json` locks in Windows-specific optional dependencies (e.g., `@rollup/rollup-win32-x64-msvc`). Railway runs `npm ci`, which strictly enforces the lock file and fails with `EBADPLATFORM` when it encounters packages targeting a different OS.

**Rule: Create `.npmrc` with `force=true` in the project root at project creation time, before the first `npm install`.**

```
# .npmrc
force=true
```

This tells npm to skip platform compatibility checks, allowing `npm ci` on Linux to ignore Windows-specific packages and install the correct Linux equivalents. Commit `.npmrc` alongside `package.json`.

**What does NOT work:**
- Deleting `package-lock.json` and regenerating — still locks in Windows binaries because you're running on Windows
- Adding `os[]=linux` and `os[]=win32` to `.npmrc` — `npm ci` still enforces platform checks
- Using `railway.json` with `installCommand: "npm install"` — Railway's Railpack builder may ignore this

This applies to any Vite/React/Rollup project (and potentially other bundlers with platform-specific optional dependencies) when the dev machine is Windows and the deployment target is Linux.

### 8.4 Git and Version Control Requirements
- GitHub is required for all projects.
- Commits must be descriptive and tied to the phase being executed.
- No force-pushing to main branch.
- Tagging releases is required before production deployment.
- Branching strategy: declared in PRD (default: main + feature branches).

### 8.5 Database Backup and Recovery Requirements
Git versions source code, migration SQL files, and Edge Function source — the *instructions* for how infrastructure should look. It does **not** version live database state, deployed Edge Functions, or active RLS policies. Recovery from data-layer failures depends entirely on the database provider's backup infrastructure.

**Supabase backup tiers:**
- **Free plan:** Limited backups, no point-in-time recovery. Acceptable for development and internal tools only.
- **Pro plan:** Daily automatic backups. Sufficient for low-traffic production applications.
- **Pro plan + PITR add-on:** Point-in-time recovery to any moment. **Required for any application going to production with real users generating data that cannot be recreated.** The difference between daily backups and PITR is up to 24 hours of lost user data.

**Rule:** The PRD must declare the backup tier. If Expected User Scale in Section 0 is anything beyond internal/demo use, the PRD must flag PITR as recommended and document the decision (enable or accept risk).

### 8.6 Edge Function Deployment Discipline
Supabase does **not** maintain a version history of deployed Edge Functions. There is no rollback button. If a broken Edge Function is deployed, the only recovery path is to fix the code and redeploy from Git.

**Rule: Edge Functions must only be deployed from committed, tagged code on the main branch.**

- Never deploy from a feature branch.
- Never deploy from uncommitted local changes.
- Every deployment must be traceable to a specific Git commit.
- The Edge Function Deployment Manifest in the PRD (Section 15) must be updated with each deployment.

If every deployment is from a known Git state, then Git is an effective version control system for Edge Functions — because you can always check out the exact commit and redeploy it. If someone deploys from uncommitted changes, there is no record of what is actually running in production.

### 8.7 Schema Drift Prevention
All schema changes — tables, columns, RLS policies, functions, triggers — must go through numbered SQL migration files (Section 10). Direct modifications via the Supabase dashboard are prohibited for production environments.

**What must never be modified directly in the dashboard:**
- RLS policies
- Database functions and triggers
- Table schema (columns, types, constraints)
- Indexes
- Storage bucket policies

If a dashboard modification is made during development for testing purposes, it must be captured in a migration file before the change is considered complete. The freeze audit verifies that migration files match the current live schema state.

### 8.8 Setup Guides and Service Automation (`docs/resources/`)
Every external service in the project's tech stack must have a corresponding setup guide in `docs/resources/`. The automation-first principle applies: **Claude Code automates everything it can via CLI/MCP tools. Setup guides document only the irreducible human-only steps.**

#### 8.8.1 Three-Category Model
Every setup guide uses exactly three categories:

**Category 1: Human-Only Setup (Before Claude Code Starts)**
The irreducible minimum that requires a human in a browser. These are steps Claude Code physically cannot perform.

Typical human-only steps:
- Create an account on the service (with direct signup link)
- Create a new project/app in the service's dashboard
- Accept terms of service or billing agreements
- Copy base credentials (URL, API keys, tokens) into `.env`

**Category 2: Automated Setup (Claude Code Executes via CLI/MCP)**
Everything that can be done programmatically. Claude Code executes these steps during the build using CLI tools or MCP servers. The setup guide documents what will be automated (for transparency) but the human does not need to do these steps.

Typical automated steps (Supabase via MCP/CLI):
- Create database tables and apply schema
- Enable auth providers (Discord, Google, etc.)
- Configure OAuth redirect URLs
- Apply RLS policies and GRANTs
- Deploy Edge Functions
- Configure storage buckets
- Generate TypeScript types

Typical automated steps (Railway via CLI):
- Create project and link to GitHub repo
- Set environment variables
- Configure deployment settings

**Category 3: Human-Only Refinement (Post-Build, Production Hardening)**
Configuration that depends on the deployed application or requires human judgment for production readiness.

Typical refinement steps:
- Custom domain/DNS configuration
- Upgrade to production tier (Supabase Pro, PITR)
- Configure production rate limits
- Set up monitoring/alerting thresholds
- Security review of auth provider settings
- Webhook endpoint configuration (requires deployed URL)

**What to exclude from all categories:**
- Generic documentation about what the tool does (link to official docs instead)
- Actual API keys, secrets, tokens, passwords, or credentials -- use placeholders and reference `.env.example`

#### 8.8.2 MCP/CLI Service Integration (Automation Layer)

For services with CLI or MCP support, Claude Code connects directly and executes automated setup steps during the build. This eliminates the majority of manual dashboard configuration.

**Project Safety Rules (Critical):**
- **Always scope to a NEW project.** MCP servers and CLI tools must be connected to the project being built, never to existing projects with production data. The setup guide's Category 1 step is "create a NEW project" -- not "use existing project."
- **Project scoping is mandatory.** When configuring MCP servers, always include the project reference/ID to restrict access to a single project. Never leave MCP servers unscoped (which gives access to all projects in the account).
- **Verify project target before write operations.** Before Claude Code runs any schema-modifying command (create table, apply migration, deploy function), it must confirm the target project matches the project being built. If there's any ambiguity, stop and ask.
- **No production connections.** MCP servers for database services (Supabase, Firebase, etc.) must never be connected to production databases. Development/staging only.
- **Read-only when exploring.** When Claude Code needs to inspect an existing project for reference (e.g., to understand an existing schema), use read-only mode if the CLI/MCP supports it.

**Supported Service Integrations:**

**Supabase MCP** (recommended for all Supabase projects):
```
claude mcp add supabase --transport http "https://mcp.supabase.com/mcp?project_ref=<YOUR_PROJECT_REF>"
```
Replace `<YOUR_PROJECT_REF>` with the project ID from your Supabase dashboard (Settings > Project ID). This scopes the MCP server to your specific new project. After adding, Claude Code authenticates via browser OAuth -- no API keys in config files.

Capabilities: Create tables, run SQL, manage schema, enable auth providers, configure storage, deploy Edge Functions, generate types, query data.

**Supabase CLI** (alternative to MCP, or for local development):
```bash
npm install -g supabase
supabase login
supabase link --project-ref <YOUR_PROJECT_REF>
```

**Railway CLI** (for deployment):
```bash
npm install -g @railway/cli
railway login
railway link
```

**Service-specific MCP/CLI installation is documented in each project's setup guide** and configured in `.mcp.json` in the project root. The scaffolding checklist (Section 7) includes MCP configuration as a step.

#### 8.8.3 Required Guide Template

```markdown
# [Tool Name] Setup Guide

## Category 1: Human-Only Setup (Do This Before the Build)

### Account and Project Creation
[Steps to create account and NEW project/app in the service]
[IMPORTANT: Create a NEW project -- do not reuse an existing one]

### Credentials for .env
[How to find the required API keys and tokens]
[Reference .env.example for variable names -- NEVER include actual values]

## Category 2: Automated Setup (Claude Code Handles This)

### What Gets Automated
[List of steps Claude Code will execute via CLI/MCP during the build]
[The human does NOT need to do these -- they are listed for transparency]

### MCP/CLI Configuration
[How to connect the CLI or MCP server, scoped to this project]

## Category 3: Post-Build Refinement (Do This After the Build)

### Production Hardening
[Steps that require the deployed application or human judgment]

### Verification
[How to confirm the integration is working end-to-end]
```

#### 8.8.4 Identifying Tools That Need Guides
Any external service the project depends on requires a setup guide. Common categories: hosting/deployment (Railway, Vercel), backend-as-a-service (Supabase, Firebase), auth providers (Discord OAuth, Clerk), AI/LLM APIs (Anthropic, OpenAI), source control (GitHub), email (SendGrid, Resend), payments (Stripe, Clerk Billing), analytics/observability (PostHog, Sentry), domain/DNS providers.

#### 8.8.5 Generation Timing
- **During the kickoff prompt process:** The PRD identifies all external tools and includes a setup guide inventory table.
- **Before the build starts:** Claude Code generates setup guides with Category 1 (human-only) steps and Category 2 (automated) steps documented. The human completes Category 1 steps. Claude Code configures MCP/CLI connections for services that support automation.
- **During the build:** Claude Code executes Category 2 (automated) steps via CLI/MCP as part of the build process -- schema creation, auth provider configuration, deployment setup.
- **After the build:** Claude Code populates Category 3 (refinement) sections. The human completes production hardening steps.
- **Before the freeze audit:** All setup guides must be complete across all three categories.

#### 8.8.6 `docs/resources/README.md` (Index File)

```markdown
# Project Resources -- Setup Guides

Setup guides for every external service used by this project.
Guides follow the three-category model: human-only setup,
automated setup (Claude Code handles via CLI/MCP), and
post-build refinement.

**IMPORTANT: No guide contains actual API keys or secrets.
All sensitive values go in .env (see .env.example).**

**IMPORTANT: All MCP/CLI tools are scoped to a NEW project.
Never connect to existing projects with production data.**

## Category 1: Human-Only Setup Checklist
Complete these steps before starting the build:

| Guide | Tool | Account Created? | Project Created? | .env Populated? |
|---|---|---|---|---|
| [tool]-setup-guide.md | [Tool Name] | [ ] | [ ] | [ ] |

## Category 2: Automated Setup
These steps are executed by Claude Code during the build via CLI/MCP.
No human action required -- listed for transparency.

| Guide | Tool | MCP/CLI Connected? | Automation Scope |
|---|---|---|---|
| [tool]-setup-guide.md | [Tool Name] | [ ] | [What gets automated] |

## Category 3: Post-Build Refinement Checklist
Complete these steps after the build finishes:

| Guide | Tool | Production Hardening Done? | Verified? |
|---|---|---|---|
| [tool]-setup-guide.md | [Tool Name] | [ ] | [ ] |
```

#### 8.8.7 Security Rules for Setup Guides
- **NEVER** include actual API keys, secrets, tokens, passwords, or credentials in any setup guide
- Use placeholders (`your-api-key-here`, `<VARIABLE_NAME>`) and reference `.env.example`
- Setup guides are committed to Git -- treat them as public documents
- **All MCP/CLI connections must be scoped to the specific new project being built**
- **Never connect MCP servers to production databases or existing projects with live data**
- Cross-reference between guides when setup steps span multiple tools

### 8.9 Build Artifact Validation (Publish Safety)

**Lesson learned:** The Claude Code source code leak (March 31, 2026) was caused by a missing `*.map` entry in `.npmignore`. A 59.8 MB source map containing 512,000 lines of source code shipped to the public npm registry. No CI/CD check caught it. This was the second such incident in 13 months for the same product. The leak coincided with a supply chain attack on the axios npm package, compounding the exposure.

**The rule:** Before any deployment or publish step, verify the artifact contains only what you intend to ship.

#### 8.9.1 For Projects That Publish npm Packages
- **`.npmignore` must explicitly exclude:** `*.map`, `*.tsbuildinfo`, `.env*`, `*.pem`, `*.key`, `src/` (if publishing compiled output), `tests/`, `*.test.*`, `.claude/`, `docs/`, and any debug or development artifacts
- **Alternatively, use the `files` field in `package.json`** to allowlist only the files that should be published (safer than blocklisting with `.npmignore`)
- **Pre-publish verification is mandatory:** Run `npm pack --dry-run` before every first publish and after any build configuration change. Review the file list. If the package size exceeds the expected baseline by more than 2x, investigate before publishing
- **CI/CD gate (recommended):** Add an automated check that compares the packed artifact size against a recorded baseline. Flag if size increases by more than 50% unexpectedly

#### 8.9.2 For All Deployed Projects
- **No source maps in production** unless explicitly required for error tracking (and if so, source maps must be served via a private error tracking service, never bundled in the public deploy)
- **No debug artifacts, test files, or development configuration in production deploys**
- **Verify deployed artifact matches the build output:** The deploy step should use a locked, tagged build — not a live build from source that might include uncommitted files
- **Size gate:** If a deployment artifact grows by more than 50% between releases without a corresponding feature addition, investigate before deploying. Unexpected size increases often indicate included debug files, unoptimized assets, or accidentally bundled source

#### 8.9.3 Claude Code Installation
Claude Code itself should be installed via the **native installer** (not npm) to avoid exposure to npm supply chain attacks:
```bash
curl -fsSL https://claude.ai/install.sh | bash
```
The native installer uses a standalone binary that does not rely on the npm dependency chain and supports background auto-updates. This recommendation is informed by the March 2026 incident where the Claude Code npm package shipped alongside trojanized axios versions on the same registry.

---

# 9. Error Handling and Debug Strategy

### 9.1 Debug Mode (URL Toggle)
All applications must support a debug mode toggled via URL parameter: `?debug=true`

**When debug mode is ON:**
- Verbose logging to browser developer console
- Detailed error objects logged (stack traces, request/response data)
- Performance timing logged
- State transitions logged
- Debug mode indicator visible in UI (subtle badge or border)

**When debug mode is OFF (default / production):**
- Minimal console output (security requirement)
- No stack traces, internal paths, or data structures written to console
- No sensitive information in any client-visible output

Debug mode should be strippable from production builds when no longer needed.

### 9.1.1 Auth Diagnostic Logging (Troubleshooting Only)
When debugging auth or loading state issues, add temporary `console.log` statements with a consistent prefix (e.g., `[AUTH DIAG]`) at these checkpoints:
- `ProtectedRoute` render — log all auth state values (session, user, loading flags)
- `onAuthStateChange` — log event type and session presence
- User lookup — log start and completion (success or failure)
- Page component `useEffect` — log entry and exit of fetch functions

These diagnostic logs must use the `[AUTH DIAG]` prefix so they can be found and removed after diagnosis. They must not ship in production.

### 9.2 User-Facing Error Handling
- Errors must be caught at every network call and async operation.
- User-facing error messages must be **subtle and non-technical**: a brief notification that an error occurred with guidance to contact support.
- No stack traces, error codes, or internal details shown to users.
- Error notifications should be dismissible and non-blocking (toast pattern preferred).
- Permission-denied states must show a clean, intentional UI (not a raw error).

### 9.3 Required UI States
Every data-driven component must handle:
- **Loading state** — Skeleton or spinner while data loads
- **Empty state** — Intentional messaging when no data exists
- **Error state** — Subtle error notification with retry option where appropriate
- **Permission-denied state** — Clean message if user lacks access

### 9.4 Server-Side Error Handling
- Edge Functions must return appropriate HTTP status codes.
- Errors must be logged server-side with sufficient context for debugging.
- No internal error details returned to client in production mode.
- Failed operations must not leave data in an inconsistent state.

---

# 10. Schema Migration and Versioning

### 10.1 Migration Strategy
- All schema changes must be captured in SQL migration files.
- Migration files must be numbered sequentially and stored in the repository (e.g., `supabase/migrations/`).
- No direct SQL changes to production database without a corresponding migration file.
- Each migration must be idempotent where possible.
- Destructive migrations (dropping columns, tables) require explicit approval.
- **Initial schema migrations must include table-level GRANTs** for `anon` and `authenticated` roles on all public tables with RLS. Do not defer GRANTs to a separate migration — they must ship with the table creation.

### 10.2 Migration File Format
```
-- Migration: YYYYMMDD_NNN_description.sql
-- Phase: X
-- Description: Brief description of what this migration does
-- Reversible: Yes/No

-- Up
<<SQL statements>>

-- Down (if reversible)
<<Rollback SQL statements>>
```

### 10.3 Triggers and Functions
- Must be versioned in migration files.
- Must be idempotent.
- Must not mutate tenant boundary fields.
- Must be documented with purpose and expected behavior.

---

# 11. Notification Architecture (If Used)

Notification systems must define:
- Storage model
- Trigger origin
- Tenant isolation
- Role broadcast rules
- Read / unread state

Implicit broadcast behavior is prohibited.

---

# 12. Data Integrity Rules

If domain rules exist, they must be enforced at backend level.
Examples:
- Unique constraints
- Enum restrictions
- Referential integrity
- Soft delete rules (`deleted_at`, `deleted_by`)

Client validation alone is insufficient.

### 12.1 Audit Logging (Recommended)
For applications with privileged operations, create an `audit_log` table:
- `id` (PK)
- `tenant_id`
- `actor_user_id`
- `action` (e.g., 'create', 'update', 'delete', 'impersonate')
- `entity` (table name)
- `entity_id`
- `before` (JSONB — previous state)
- `after` (JSONB — new state)
- `created_at`

Audit logging is mandatory for: user management actions, role changes, data deletion, impersonation, and any auto-remediation actions.

### 12.2 Soft Delete
Preferred over hard delete for tenant-scoped data:
- `deleted_at` (timestamp)
- `deleted_by` (user_id)
- Purge policy must be declared in PRD if applicable.

### 12.3 Data Safety During Development and Testing
**Rule: Never delete production data.** This applies during development, testing, debugging, and feature work. Claude Code must not run DELETE statements against production data, clear tables, or "clean up" records as part of any workflow.

The only exception: if a test protocol explicitly creates a specific test record for the purpose of verifying create/delete functionality, that exact test record may be deleted as part of the test — and only if the deletion is non-cascading and safe (no foreign key cascades that could affect other data).

This rule exists because applications built with this framework are designed to scale to production with real users. The testing and development process must work *alongside* existing data, not by wiping and rebuilding. All development should assume the database contains data that matters.

---

# 13. Performance Requirements

- Critical queries must be indexed.
- No full table scans for protected data.
- Record scale assumptions must be declared in PRD.
- Target latency must be defined.

### 13.1 Required Indexes
At minimum, tenant-scoped tables must have:
- `(tenant_id, created_at)` — Default sort queries
- `(tenant_id, user_id)` — Membership and ownership junctions
- Any frequently filtered columns alongside `tenant_id`

### 13.2 Realtime (If Used)
- Realtime subscriptions must be tenant-safe.
- Validate tenant membership before subscribing where possible.
- Document which tables use Realtime in the PRD.

### 13.3 Production Observability
For any application shipping to real users beyond internal/demo use, the PRD must declare an observability strategy. Without observability, you have zero visibility into how users interact with the application and what errors they encounter.

**Error tracking (required for production applications):**
Production applications must include server-side error capture and alerting. Tools: Sentry, PostHog, LogRocket, or equivalent. Errors must be captured automatically — not dependent on users reporting them. Error tracking must include: error message, stack trace, user context (role, tenant), and timestamp. Alerts must notify the development team when new error types appear.

**Analytics (required when PRD declares user scale > 50):**
When an application will serve more than a handful of users, feature adoption tracking becomes essential for product decisions. Requirements: track key user actions (feature usage, conversion funnels, retention), surface data in a dashboard accessible to the project owner, and review adoption data before building new features.

**Feature adoption tracking as a build discipline:**
When a new feature is implemented, the build must include tracking events for that feature's key interactions. This is a one-line addition per feature but compounds into real product intelligence over time. Every new feature must have at minimum: a tracking event for first use, a tracking event for repeated use, and visibility in the analytics dashboard.

**Session replay (recommended for consumer-facing applications):**
Session replay tools (PostHog, LogRocket, FullStory) let you watch how users actually interact with the application. Recommended for any application where UX decisions are being actively iterated.

**Implementation:**
- Analytics and error tracking should be initialized once in the application root, not scattered across components.
- Tracking events should use a consistent naming convention declared in the PRD.
- Error tracking must be configured before the first production deployment — not added after problems are reported.
- The observability tool and configuration must be documented in the PRD's environment variables section.

---

# 14. Testing Strategy

### 14.1 Current Approach
Primary testing is manual verification per phase. Automated testing is an aspirational target and will be adopted incrementally as team capability grows.

### 14.2 Manual Verification (Required)
Every phase must include explicit manual verification steps that confirm:
- Acceptance criteria are met
- Tenant and role boundaries are enforced (if applicable)
- No undocumented assumptions remain
- UI states (loading, empty, error, permission-denied) behave correctly

### 14.3 Tenant Isolation Tests (Required for Multi-Tenant)
Before any multi-tenant application is considered complete:
- User in Tenant A cannot read any row from Tenant B, even by guessing IDs
- Scoped internal roles (e.g., coach) can access only assigned tenants
- Superuser can access all tenants and impersonation is logged
- Storage objects cannot be read across tenants (if Storage is used)
- Admin-only actions are blocked for members at DB level

### 14.4 Role-Based Test Cases (Living Document)
Every application must maintain a `tests/role-tests.md` file in the repository that defines test scenarios for each user role. This file is a living document that evolves as the application grows.

**During initial build:** Establish baseline test cases for each role defined in the PRD's permissions matrix. At minimum, each role must have test cases covering: login and session creation, access to permitted pages/features, denial of access to restricted pages/features, CRUD operations the role is authorized to perform, and verification that unauthorized CRUD operations are blocked.

**During ongoing development:** Any feature that touches auth, permissions, or role-specific UI must add new test cases to `tests/role-tests.md`. Any bug fix related to role boundaries must add a regression test case.

**Test execution:** After any change that affects auth, permissions, or role-specific functionality, the full role-based test suite for affected roles must be executed. Claude Code should use browser verification (Chrome plugin) when available to validate: UI renders correctly for the role, console shows no errors, network requests return expected status codes, and the feature behaves as specified.

**Format for test cases:**
```markdown
## Role: <<ROLE_NAME>>

### TC-001: <<Test case name>>
- **Precondition:** <<Setup required>>
- **Steps:** <<What to do>>
- **Expected result:** <<What should happen>>
- **Status:** Pass / Fail / Not tested
- **Last verified:** <<Date>>
```

### 14.5 Automated Testing (When Ready)
When adopted, automated tests are required for:
- Calculation engine (unit tests for each formula, edge cases, regressions)
- Authorization boundary tests
- API response validation

### 14.6 End-to-End Testing with Playwright (Recommended)
For applications with a UI, Playwright end-to-end tests provide automated verification of the running application from the user's perspective. Claude Code can generate and execute these tests before presenting the build as complete, catching broken pages, permission failures, console errors, and UI regressions before the human ever touches the app.

**When to use:**
- Any project with UI components (recommended, not mandatory)
- Especially valuable for applications with multiple user roles where permission boundaries must be verified from each role's perspective

**What Playwright tests should cover:**
- **Page load verification:** Every route loads without console errors or network failures
- **Role-based access:** For each user role, verify accessible pages load correctly and restricted pages are properly denied
- **Core user journeys:** The primary workflows defined in the PRD (e.g., login → create record → view record → edit → delete test record)
- **Form submissions:** Key forms submit successfully with valid data and show appropriate errors with invalid data
- **Permission enforcement from the UI:** A member role attempting an admin action sees a clean denial, not an error

**Test structure:**
```
tests/
  e2e/
    setup.ts              # Auth helpers, test user creation
    pages.spec.ts         # All routes load without errors
    roles/
      admin.spec.ts       # Admin-specific flows
      member.spec.ts      # Member-specific flows
    journeys/
      [journey].spec.ts   # Core workflow tests from PRD
```

**Execution timing:**
- **Express Build:** Run during Step 3 (Verify and Harden) after the full build pass
- **Full Build:** Run at the end of Phase 5 (UI Implementation) and again at Phase 8 (Hardening)
- **Ongoing development:** Run affected test suites before merging feature branches

**Data safety:** Playwright tests must follow the data safety rules in Section 12.3. If a test creates a record, it may delete that specific test record as cleanup — but only if the deletion is non-cascading. Tests must work alongside existing data, not by wiping and rebuilding.

**Self-fix loop:** When Playwright tests fail, Claude Code should analyze the failure, fix the underlying issue, and re-run the failing test. Only present results to the human after all tests pass or after two fix attempts per failure (then escalate).

---

# 15. Auto-Remediation and AI Operations

### 15.1 Scope
Applications may leverage Claude (Opus 4.6 only) for auto-remediation of non-critical runtime issues. Security is the absolute top priority — auto-remediation must never compromise authorization boundaries, tenant isolation, or data integrity.

### 15.2 Constraints
- Only Claude Opus 4.6 is authorized for auto-remediation actions.
- Auto-remediation must never modify: RLS policies, authorization logic, tenant boundaries, user roles, or authentication configuration.
- All changes made by auto-remediation must be appended to a persistent changelog (see 15.3).
- Auto-remediation is limited to: error recovery, retry logic, configuration correction, and non-security bug fixes.

### 15.3 Changelog Requirement
Every auto-remediation action must append to a structured changelog:
- Timestamp
- Trigger (what error or condition was detected)
- Action taken
- Files or data modified
- Before/after state (where applicable)
- Outcome (success/failure)

This changelog must be persistent, append-only, and accessible to system administrators.

### 15.4 Production Notifications
In production environments, auto-remediation actions must trigger a notification to the designated system administrator. Notification method to be declared in PRD (e.g., email, Discord webhook, Supabase Edge Function trigger).

### 15.5 Nightly Security Audit (Cron Job)
Production applications should include a scheduled nightly security audit that:
- Validates RLS is enabled on all tenant-scoped tables
- Checks for orphaned records or broken references
- Verifies no unauthorized schema changes have occurred
- Reviews auto-remediation changelog for anomalies
- Reports results to system administrator

Implementation: Supabase Edge Function triggered by cron (or external cron service via Railway).

---

# 16. Rollback and Recovery

### 16.1 Phase Failure Protocol
If a phase fails verification:
1. Document what failed and why.
2. Do not proceed to the next phase.
3. Revert changes via Git if the failure is structural.
4. If revert is not clean, create a fix plan and get approval before proceeding.
5. Re-run verification after fix.

### 16.2 Production Rollback
- Git tags mark each production release.
- Rollback means redeploying the previous tagged release.
- Database rollback requires executing the "Down" migration (if reversible).
- Non-reversible migrations require a forward-fix approach — document this in the migration file.

### 16.3 Data Recovery
- Soft delete is preferred to enable recovery.
- Backup strategy must be declared in PRD (Supabase provides automatic daily backups on Pro plans).
- Point-in-time recovery availability depends on Supabase plan tier.

---

# 17. Code Hygiene Rules

- Consistent naming conventions (declared in PRD or established in Phase 0).
- Clear folder structure with documented ownership per module.
- No unused dependencies.
- No commented-out code in production.
- No orphaned files or dead routes.
- Error handling for all network calls and async operations.
- Loading, empty, error, and permission-denied states on all data-driven components.
- No `console.log` statements in production mode (use debug mode toggle).

### 17.1 React Render Stability Rules
Unstable object references in React context providers and hooks are a common source of infinite render loops, especially when combined with `useEffect` and `useCallback` dependency arrays. The following rules are mandatory:

**Rule 1: Context provider values must be referentially stable.**
Any object returned by a context provider (e.g., a `toast` object, an `api` object, a `theme` object) must be wrapped in `useMemo` so it maintains a stable reference across renders. A plain object literal recreated every render will cause any consumer with that object in a dependency array to re-fire on every render.
```jsx
// WRONG: new object every render
const toast = { success: addToast, error: addToast };
return <ToastContext.Provider value={toast}>...

// CORRECT: stable reference
const toast = useMemo(() => ({ success: addToast, error: addToast }), [addToast]);
return <ToastContext.Provider value={toast}>...
```

**Rule 2: Never include React context objects in `useCallback` dependency arrays.**
Even when memoized, context objects (e.g., `toast`, `api`, `theme`) as dependencies create unnecessary coupling between data-fetching logic and UI notification systems. Omit them from dependency arrays and suppress the lint warning:
```jsx
// WRONG: toast in dependency array creates coupling
const fetchUsers = useCallback(async () => {
  // ...fetch logic...
}, [toast]); // toast changes → fetchUsers changes → useEffect re-fires

// CORRECT: omit context objects from dependency arrays
const fetchUsers = useCallback(async () => {
  // ...fetch logic...
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []); // runs once on mount
```

**Rule 3: Watch for error → re-render → retry loops.**
The most dangerous pattern is: API call fails → error handler fires (e.g., `toast.error()`) → state change causes re-render → unstable reference triggers `useEffect` → API call fires again → infinite loop. This pattern can flood the backend with requests and requires killing the server to stop. Every `useEffect` that makes an API call must be audited for this loop potential during code review.

### 17.2 Component Architecture and Decomposition
Pages must be layout and orchestration only — they compose components, manage routing state, and handle page-level data fetching. All UI blocks (forms, tables, modals, filters, cards, feature-specific logic) must be extracted into dedicated component files.

**File size threshold:** No single component or page file should exceed approximately 200 lines. When a file approaches this threshold, decompose it into smaller components. This is a soft ceiling — the goal is readability and maintainability, not rigid enforcement. But a 600-line page file is always a signal that decomposition is overdue.

**Why this matters for AI-assisted development:**
- **Token efficiency:** When Claude Code needs to fix a table filter, it loads the 80-line filter component, not the entire 600-line page. Across dozens of fixes and enhancements, this is a significant cost difference.
- **Blast radius:** A change to one component doesn't risk breaking unrelated components on the same page.
- **Reuse:** Components extracted into their own files are immediately available for reuse on other pages (see Section 22.4).
- **Debugging:** Error stack traces point to specific component files, not a monolithic page.

**Folder structure:**
```
.claude/
  agents/             # Custom subagents (security-reviewer.md, etc.) — see Section 20.3.3
  hooks/              # Hook enforcement scripts — see Section 20.5
    pre_tool_use.py
    post_tool_use.py
    stop.py
    pre_compact.py
  settings.json       # Hook configuration + permissions + env
docs/
  resources/           # Setup guides for external services — see Section 8.8
    README.md          # Index of all setup guides with pre/post-build checklists
    [tool]-setup-guide.md  # One per external service
STATE.md               # Persistent build state — see Section 20.7
CONTEXT.md             # Implementation decisions from discuss phase — see Section 20.8
lessons-learned.md     # Failures and fixes during build — see Section 20.2
src/
  pages/              # Page-level components (routing, layout, data orchestration)
    Dashboard.jsx     # Composes DashboardStats, RecentActivity, QuickActions
    AdminUsers.jsx    # Composes UserTable, UserFilters, InviteModal
  components/
    shared/           # Reusable across pages (buttons, modals, tables, form fields)
    dashboard/        # Dashboard-specific components
    admin/            # Admin-specific components
```

**Rule: When a feature is requested for a page, build it as a component from the start — not inline in the page file.** This prevents the pattern where features start inline, grow complex, and are painful to extract later.

### 17.3 Feature Extraction Protocol (Extract-Then-Share)
When a feature that exists on one page is requested for another page, the response is **never** to rebuild it from scratch on the second page. Instead:

1. **Extract:** Move the existing feature implementation from the original page into a shared component in `components/shared/` (or an appropriate shared directory).
2. **Refactor:** Generalize the component's props interface so it works in both contexts. Keep the API minimal — pass only what differs between the two use cases.
3. **Replace:** Update the original page to import and use the shared component.
4. **Add:** Import and use the same shared component on the new page.

This is a strict protocol, not a suggestion. Duplicating feature implementations creates two diverging codebases for the same functionality, doubles the debugging surface, and guarantees inconsistent behavior over time. The short-term cost of extracting is always less than the long-term cost of maintaining duplicates.

---

# 18. LLM Usage, Cost Efficiency, and Processing Strategy

### 18.1 Model Selection
Not every task requires the most capable model. Select the right model based on the task:

| Task Type | Recommended Model | Rationale |
|---|---|---|
| Architecture decisions, security review, complex reasoning | Claude Opus 4.6 | Highest capability needed for judgment calls |
| Code generation, feature implementation, debugging | Claude Sonnet 4.5 | Strong coding ability at lower cost |
| Simple text formatting, classification, extraction | Claude Haiku 4.5 | Fast, cheap, sufficient for mechanical tasks |
| Auto-remediation (per Section 15) | Claude Opus 4.6 only | Security-critical — no exceptions |

If the application makes runtime LLM calls, the PRD must declare which tasks use which model tier and why.

### 18.2 Token Efficiency
LLM calls are not free. Every API call has a cost in tokens, latency, and rate limit budget. Apply these principles:

**Minimize token waste:**
- Send only the data the model needs — no full database dumps, no redundant context
- Use structured prompts with clear instructions rather than long conversational preambles
- Batch related questions into a single call rather than making multiple round trips for the same context
- Cache LLM responses where the same input will produce the same output (e.g., classification of static content)
- Set appropriate `max_tokens` limits — don't default to maximum when a short response is expected

**Prompt design for cost:**
- Use system prompts for persistent instructions rather than repeating them in every user message
- Prefer structured output (JSON) over free-text when downstream code will parse the response — it reduces tokens and eliminates parsing failures
- For extraction or classification tasks, provide the exact schema you want back

### 18.3 Rate Limit Awareness
Applications that make LLM API calls must account for rate limits at both the application level and in batch scripts (see also Section 18.5.1 for batch-specific parallelism rules):

- Implement exponential backoff with jitter on rate limit errors (HTTP 429)
- Read and respect rate limit headers from API responses (`retry-after`, `x-ratelimit-remaining`, `x-ratelimit-reset`)
- Queue and throttle bulk LLM operations rather than firing them all concurrently
- Track tokens-per-minute (TPM) and requests-per-minute (RPM) consumption against account limits
- If the application serves multiple users making LLM calls, implement per-user or per-tenant rate limiting at the application level to prevent one user from exhausting the shared quota
- Log rate limit hits so usage patterns can be analyzed
- PRD must declare expected LLM call volume (calls per hour/day) and known account-level rate limits for capacity planning

### 18.4 When NOT to Use an LLM
LLMs should only be used when the task requires language understanding, reasoning, or generation. If the task is purely mechanical, use a script instead.

**Use Python scripts (or equivalent) instead of LLM calls for:**
- Data transformation, reformatting, and migration (CSV → JSON, schema mapping, etc.)
- Bulk CRUD operations against APIs or databases
- File processing (renaming, moving, merging, splitting)
- Mathematical calculations, aggregations, and statistics
- String manipulation with known patterns (regex, templating)
- API integrations where the request/response format is fixed and deterministic
- Report generation from structured data where the template is known
- Scheduled jobs, cron tasks, and batch processing pipelines
- Deduplication, sorting, filtering, and any operation on structured data

**Use LLM calls for:**
- Natural language understanding or generation
- Content summarization, rewriting, or tone adjustment
- Classification where rules are ambiguous or evolving
- Extraction from unstructured text (emails, documents, free-form input)
- Conversational interfaces
- Tasks where the decision logic is too complex or nuanced to encode as rules

**The test:** If you can write a deterministic function that handles 100% of cases correctly, don't use an LLM. If the task requires judgment, interpretation, or handling of ambiguous input, an LLM is appropriate.

### 18.5 Bulk Processing Architecture
When an application needs to process large volumes of data (imports, migrations, batch analysis):

- **Default to Python scripts.** All batch processing, data pipelines, and bulk operations must be built as standalone Python scripts unless the PRD explicitly declares otherwise. Python is the standard because of its ecosystem for data manipulation, API clients, and async processing.
- **LLM-in-the-loop only where needed.** If some records require language understanding (e.g., categorizing free-text descriptions), pipe only those specific fields to the LLM — not the entire record. The script handles all deterministic work; the LLM handles only the judgment calls.
- **Batch API calls.** Use the Anthropic Messages Batches API for bulk LLM operations where real-time response isn't needed — it's significantly cheaper and avoids rate limit pressure.
- **Checkpoint and resume.** Bulk operations must be resumable. Log progress (e.g., last processed record ID or offset) so a failed batch can restart from where it left off, not from the beginning.
- **Separate LLM cost from compute cost.** Track LLM API spend separately from infrastructure cost so usage patterns are visible.

### 18.5.1 Adaptive Parallelism
Batch scripts must respect account-level rate limits for both LLM APIs and any external service APIs (web search, third-party integrations). The parallelism strategy must be adaptive, not fixed.

**Sequential by default:**
- Start with sequential (one-at-a-time) processing unless throughput requirements demand parallelism.
- Sequential is safer, simpler to debug, and avoids rate limit violations on accounts with lower tiers.

**Parallel when justified:**
- If throughput requirements are high and account rate limits allow it, use controlled parallelism (async workers, thread pools, or task queues).
- **Never fire unbounded concurrent requests.** Always use a concurrency limiter (e.g., `asyncio.Semaphore` in Python) set below the account's tokens-per-minute (TPM) and requests-per-minute (RPM) limits.
- Leave headroom — target no more than 80% of the account's rate limit to absorb variance and avoid 429 errors.

**Rate limit awareness in scripts:**
- Read rate limit headers from API responses (`retry-after`, `x-ratelimit-remaining`, etc.) and adjust pacing dynamically.
- Implement exponential backoff with jitter on 429 responses.
- Log every rate limit hit with timestamp and context so patterns are visible.
- If the script makes both LLM calls and external API calls (e.g., web search), track each rate limit independently — they are separate quotas.

**PRD requirements for batch operations:**
- Declare expected volume (records per run)
- Declare required throughput (records per minute/hour)
- Declare target APIs and their known rate limits
- Declare whether sequential or parallel processing is needed
- Declare checkpoint/resume strategy

### 18.6 LLM Cost Tracking
If the application makes LLM API calls at runtime:

- Log every LLM call with: timestamp, model used, input tokens, output tokens, task type, and estimated cost
- Surface this data to system administrators (dashboard, report, or log file)
- Set cost alert thresholds in the PRD (e.g., "alert if daily LLM spend exceeds $X")
- Review LLM usage patterns periodically to identify tasks that could be moved to scripts or a cheaper model tier

---

# 19. SEO, Crawl Policy, and AI Discoverability

### 19.1 Default Crawl Policy (Strict No-Index)
All newly deployed applications must ship with crawling and indexing **disabled** by default. This remains in effect until the project owner explicitly authorizes public indexing.

**Required at launch:**
- `robots.txt` at root returning:
  ```
  User-agent: *
  Disallow: /
  ```
- `<meta name="robots" content="noindex, nofollow">` on every page
- `X-Robots-Tag: noindex, nofollow` response header (set at Railway or Edge Function level)
- No sitemap submitted to any search engine

**To enable indexing (explicit approval only):**
- Replace `robots.txt` with a permissive policy scoped to public routes — debug routes must remain disallowed:
  ```
  User-agent: *
  Disallow: /*?debug
  ```
- Remove or update meta robots tags on public-facing pages
- Debug-mode pages must **always** retain `<meta name="robots" content="noindex, nofollow">` regardless of site-wide crawl policy
- Generate and submit `sitemap.xml` (must never include URLs with debug parameters)
- This change must be recorded in the project changelog

### 19.1.1 Debug Route Protection (Permanent)
Debug mode URLs (any URL containing `?debug=true` or the `debug` parameter) must **never** be indexable, even after the site-wide crawl policy is lifted. This is enforced at three layers:

1. **robots.txt** — `Disallow: /*?debug` must persist in all versions of the robots file
2. **Meta tag** — Any page rendered in debug mode must inject `<meta name="robots" content="noindex, nofollow">`
3. **Response header** — Debug mode requests must return `X-Robots-Tag: noindex, nofollow`
4. **Sitemap** — Debug URLs must never appear in `sitemap.xml`
5. **Canonical tags** — Pages in debug mode must set `<link rel="canonical">` to the non-debug version of the URL (stripping the debug parameter)

This protection is **permanent and non-overridable** — it is not subject to the site-wide indexing toggle.

### 19.2 SEO Structure (Build Ready, Ship Disabled)
Even with indexing disabled, every application must be built with SEO-ready structure so it is immediately discoverable when the crawl policy is lifted.

**Required on all public-facing pages:**
- Semantic HTML5 elements (`<header>`, `<main>`, `<nav>`, `<article>`, `<section>`, `<footer>`)
- Unique, descriptive `<title>` per route
- `<meta name="description">` per route
- Proper heading hierarchy (`<h1>` once per page, logical `<h2>`–`<h6>` nesting)
- Canonical URL tags (`<link rel="canonical">`)
- Open Graph meta tags (`og:title`, `og:description`, `og:image`, `og:url`, `og:type`)
- Twitter Card meta tags (`twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`)
- `lang` attribute on `<html>` element
- Alt text on all meaningful images

### 19.3 Generative AI Discoverability
Applications must be structured so generative AI systems (search-integrated LLMs, AI assistants, retrieval-augmented generation) can effectively parse and reference content when indexing is enabled.

**Required:**
- JSON-LD structured data (`<script type="application/ld+json">`) on key pages using appropriate Schema.org types (Organization, Product, Article, FAQ, etc.)
- Clean, descriptive URL slugs (no hash-only routing for public content)
- Server-side rendering or static generation for public-facing content pages (client-only SPAs are not crawlable)
- `<meta name="description">` written as natural-language summaries suitable for AI extraction
- Logical content hierarchy that can be parsed without JavaScript execution where feasible

**Optional but recommended:**
- `llms.txt` at root (emerging standard for AI-specific site guidance)
- Structured FAQ sections using `FAQPage` schema markup
- Clear, self-contained introductory paragraphs on key pages (supports AI snippet extraction)

### 19.4 Implementation Notes
- SEO meta tags and structured data should be managed via a shared utility or layout component (DRY — not scattered across individual pages).
- If the application is a pure internal tool with no public-facing pages, Sections 19.2 and 19.3 may be scoped to marketing or landing pages only, as declared in the PRD.
- The PRD must declare whether the application has any public-facing content routes.

---

# 20. Claude Code Execution Contract

Claude must operate under:

- Plan-only mode until approved
- Explicit phase boundaries
- Manual verification after each phase
- No undocumented deviations

Each phase must output:
- Scope
- Files created or modified
- Acceptance criteria
- Manual verification steps
- Known limitations
- Deviations from plan (must be explicit)

### 20.1 Definition of Done (Per Phase)
A phase is done only when:
- Acceptance criteria met
- Manual verification steps pass
- No undocumented assumptions remain
- Tenant and role boundaries remain enforced (if applicable)
- Changelog updated (if applicable)
- `STATE.md` updated with phase completion status (Section 20.7)

**Self-Audit Verification Loop (Mandatory):**
Before declaring any phase complete, Claude Code must perform a self-audit:

1. **Re-read the PRD's acceptance criteria** for the current phase.
2. **Verify each acceptance criterion is implemented.** Check the actual code, not memory of what was written.
3. **If any items are missing,** implement them before presenting results.
4. **Self-verify a second time** after implementing the missing items.
5. **If items are still missing after two passes,** present what's done and what remains outstanding to the project owner for guidance. Do not loop indefinitely.
6. **Update `STATE.md`** with the phase result — mark the phase complete with a summary of what was delivered, or mark it as needing owner input with the outstanding items listed.
7. **Only after self-verification passes and STATE.md is updated,** present the phase results and wait for human approval (per Section 1.3 Phase Gate Rule).

This loop ensures two things: the PRD is fully implemented by the time the build completes, and `STATE.md` is always the authoritative record of where the build stands — surviving context compaction and session interruptions.

### 20.2 Mid-Build Error Recovery Protocol
When a build step fails, Claude Code must follow this sequence:

1. **Read the full error message and stack trace** before attempting a fix. Do not guess at the cause from a partial error.
2. **Classify the failure:** application code, configuration, dependency issue, or external service. The fix approach differs for each.
3. **Fix and verify locally** before proceeding. Do not move to the next step with an unverified fix.
4. **If the fix involves a paid API call or irreversible external action** (e.g., deploying to production, sending emails, modifying a live database), stop and confirm with the project owner before executing.
5. **Append the failure and fix to `lessons-learned.md`** in the project root. Format: what broke, why it broke, what fixed it, whether it affects other phases or future projects.
6. **If the fix reveals a pattern that would affect other phases,** flag it to the project owner before continuing. Do not silently adjust the plan.
7. **Do not proceed to the next phase with an unresolved failure.** A phase is not done until all errors are resolved and verified.

The `lessons-learned.md` file is the only project file Claude Code may append to without explicit approval. All other documentation changes require owner approval (see Section 22).

### 20.3 Parallel Execution (Agent Teams and Subagents)
Claude Code supports two mechanisms for parallel work. They operate differently and serve different purposes — choose based on whether workers need to communicate with each other.

| Feature | Agent Teams | Subagents (Task Tool) |
|---|---|---|
| Context | Own window; fully independent | Own window; results return to caller |
| Communication | Teammates message each other directly via mailbox | Report back to parent only — no sibling communication |
| Coordination | Shared task list with dependency tracking and self-claiming | Parent manages everything |
| Best for | Complex work requiring discussion and collaboration | Focused tasks where only the result matters |
| Token cost | ~5x per teammate (each is a separate Claude instance) | Lower — results summarized back |
| Persistence | Ephemeral — exist for one session, then gone | Ephemeral — exist for one task |

#### 20.3.1 Agent Teams (Full Build Only)
Agent Teams spawn multiple full Claude Code instances that work together as a coordinated team. One session acts as the lead, coordinating work and synthesizing results. Teammates work independently in their own context windows but can **communicate directly with each other and with the lead** via a mailbox system. The team shares a task list with dependency tracking — tasks can block other tasks, and when a blocking task completes, downstream tasks automatically unblock. Teammates self-claim the next available unblocked task when they finish, with file-lock based claiming to prevent race conditions.

This is fundamentally different from subagents. Subagents report results back in isolation. Agent Teams discuss, challenge each other's findings, and coordinate autonomously.

**Best use cases:**
- **Research and review:** Multiple teammates investigate different aspects of a problem simultaneously, then share and challenge each other's findings
- **New modules or features:** Teammates each own a separate piece without stepping on each other
- **Debugging with competing hypotheses:** Teammates test different theories in parallel, share evidence, and converge on the correct answer faster than a single session
- **Cross-layer coordination:** Changes that span frontend, backend, and tests, each owned by a different teammate

**When NOT to use:**
- Express Build (coordination overhead and ~5x token cost not justified for single-pass execution)
- Work where multiple teammates would edit the same files
- Simple phases with a single workstream
- Tasks where only the result matters and no inter-agent discussion is needed (use subagents instead)

**Architecture:**
- Each teammate is a separate Claude Code instance with its own context window
- Teammates load `CLAUDE.md`, PRD, and project context automatically but do NOT inherit the lead's conversation history
- Spawn prompts must include task-specific details — do not rely on prior conversation context
- Structure tasks so each teammate owns distinct files — no overlapping file edits
- You can interact with individual teammates directly without going through the lead — useful for course-correcting a teammate that's going off track
- Agent Teams are ephemeral by design — they exist for the duration of a session and then they're gone. No persistent identity, no memory across sessions, no `/resume`
- Keep teams small (2–4 teammates) to manage token cost (~5x per teammate)
- One team per session; no nested teams

**Spawn backends:**
- **In-process** (default): All teammates share one terminal view. Use `Shift+Up`/`Shift+Down` to cycle between them. Fastest, but no simultaneous visibility.
- **tmux**: Each teammate gets its own terminal pane. Requires tmux installed. Best for monitoring multiple teammates simultaneously.
- **Auto-detect**: Uses tmux if available, otherwise falls back to in-process.

Force a specific backend via environment variable: `CLAUDE_CODE_SPAWN_BACKEND=tmux` (or `in-process`).

Note: Split pane mode does not work in VS Code's integrated terminal or Windows Terminal. On Windows via the Desktop app, in-process mode is the likely default.

**Setup:** Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` in `~/.claude/settings.json`.

**Team-specific hooks (see Section 20.5):**
Agent Teams support two additional hook events beyond the standard 12:
- `TeammateIdle` — fires when a teammate is about to go idle. Exit with code 2 to send feedback and keep the teammate working.
- `TaskCompleted` — fires when a task is being marked complete. Exit with code 2 to prevent completion and send feedback. Use this as a quality gate to validate work before a task is accepted.

#### 20.3.2 Subagents via Task Tool
Subagents are lightweight Claude Code instances spawned via the built-in Task tool. Each gets its own isolated context window, operates independently, and returns distilled results to the parent. Unlike Agent Teams, subagents **cannot communicate with each other or with the parent during execution** — they are black boxes until complete.

Best for **task-level parallelism within a phase** — focused tasks where only the result matters and no inter-agent discussion is needed.

**When to use:**
- Any build mode (Express or Full Build)
- Breaking a phase into parallel tasks (e.g., one subagent for components, one for tests, one for API integration)
- Research-intensive work that would overflow a single context window
- Codebase exploration across multiple directories simultaneously
- Multi-version prototyping (build two approaches in parallel, compare results)

**When NOT to use:**
- Tasks that require back-and-forth coordination during execution (subagents can't communicate with parent or siblings until complete)
- Work that depends on another subagent's output (sequential dependencies)
- Very small tasks where the overhead of spawning exceeds the benefit

**Key characteristics:**
- Each subagent gets its own isolated context window — it doesn't pollute the parent's context
- The parent has no visibility into subagent activity until completion (black box execution)
- Results flow back as distilled summaries, not full transcripts
- Subagents can be triggered automatically or explicitly via prompts
- No experimental flag required — this is a built-in Claude Code capability

**Prompting pattern for explicit task-level parallelism:**
```
"Implement the user management feature using parallel tasks:

Task 1 (Components): Create UserTable, UserFilters, and InviteModal
  in src/components/admin/

Task 2 (API): Create the admin-users Edge Function with GET, POST,
  and PATCH endpoints

Task 3 (Tests): Write role-based test cases for admin and member
  roles in tests/role-tests.md

Run all tasks in parallel."
```

#### 20.3.2.1 Fresh Context Execution Pattern (Recommended for Full Build)
Context rot — the quality degradation that happens as Claude Code fills its context window — is one of the most common causes of incomplete phases, missed acceptance criteria, and declining code quality during long builds. Subagents solve this structurally.

**The pattern:** Instead of building an entire phase in the lead session (where context fills up and quality degrades), break the phase into discrete implementation tasks and execute each task in a fresh subagent. The lead session stays lean — it orchestrates, tracks state, and synthesizes results. The heavy code-writing work happens in subagent contexts that start clean with a full context window.

**How to structure it:**
1. **Lead session reads `STATE.md`, `CONTEXT.md`, and the PRD** to understand what the current phase requires.
2. **Lead session breaks the phase into implementation tasks** — each task should be small enough to execute in a single subagent context without risk of degradation (target: files for one feature or component, not an entire module).
3. **Each task is spawned as a subagent** with a focused prompt containing: the task description, the files it should create or modify, the acceptance criteria for that specific task, and a reference to `CONTEXT.md` for implementation decisions.
4. **Subagent executes the task** with a full, fresh context window — no accumulated context from prior tasks.
5. **Results flow back to the lead session** as distilled summaries.
6. **Lead session integrates results** and updates `STATE.md`.

**Why this works:**
- Each task gets 200k tokens of fresh context purely for implementation — zero accumulated garbage from prior tasks.
- The lead session's context stays at 30-40% usage because it only handles orchestration, not implementation.
- Quality remains consistent from the first task to the last — no degradation curve.
- If a task fails, only that task needs to be re-executed in a fresh context, not the entire phase.

**When to use fresh context execution:**
- Full Build phases with 3+ implementation tasks
- Any phase where the lead session's context exceeds 50% before implementation starts
- Phases that involve both backend and frontend work (split into separate subagent tasks)
- Long builds where you've experienced quality degradation in later phases

Express Build projects can also benefit from this pattern -- the single build pass can be structured as a series of subagent tasks orchestrated by the lead session, keeping the lead's context fresh for the final verification step.

**Skill-level implementation with `context: fork`:** For custom subagents defined as skills (Section 20.3.3), adding `context: fork` to the YAML frontmatter automatically runs the skill in an isolated subagent context. The `agent:` field selects the agent type (`Explore` for read-only analysis, `general-purpose` for tasks that modify files, `Plan` for architecture analysis). This achieves the same context isolation described above but as a built-in skill property rather than manual orchestration. Superpowers' subagent-driven-development (Section 20.9) also uses this pattern -- each task is dispatched to a fresh subagent, and only the result returns to the lead session.

#### 20.3.3 Custom Subagents (Project Infrastructure)
For larger applications, define reusable custom subagents as markdown files in `.claude/agents/` in the project root. These auto-trigger based on the work being done, enforcing framework rules without manual invocation.

**Context isolation with `context: fork`:** Custom subagents should use `context: fork` in their frontmatter so they run in an isolated context window. When a subagent reads dozens of files to check security policies or component architecture, that exploration stays in the subagent's context -- only the final report returns to the main conversation. This keeps the lead session's context clean for orchestration and prevents the quality degradation that comes from filling the main context with intermediate analysis.

**Agent types with `agent:`:** The `agent:` frontmatter field specifies which built-in agent configuration runs the subagent. Match the agent type to the task:
- `Explore` — Fast, read-only, optimized for file search and pattern matching. Uses a lighter model. Best for review and analysis subagents that only need to read code.
- `general-purpose` — All tools, full reasoning. Best for subagents that need to both analyze and write code.
- `Plan` — Architecture analysis, step planning. Best for planning and design review.

**Project folder structure:**
```
.claude/
└── agents/
    ├── security-reviewer.md
    ├── component-checker.md
    └── test-coverage.md
```

**Recommended custom subagents for this framework:**

**`security-reviewer.md`** — Automatically engages when auth code, RLS policies, Edge Functions, or permission logic is modified. Checks against the rules in Sections 4, 5, and 12. Returns a structured report with severity ratings. Uses `Explore` agent (read-only analysis).

```markdown
---
name: security-reviewer
description: Reviews code changes for security compliance with the build contract. Auto-triggers on auth, RLS, Edge Function, or permission changes.
context: fork
agent: Explore
---
# Security Reviewer Agent

When reviewing code:
1. Verify RLS policies use auth.jwt(), not subqueries on auth.users
2. Verify table-level GRANTs exist for anon and authenticated roles
3. Verify Edge Functions handle auth internally if using --no-verify-jwt
4. Verify no sensitive data is exposed in production error responses
5. Verify debug mode doesn't leak data when toggled off
6. Check for hardcoded secrets or credentials

Return a structured report with:
- Severity (Critical / High / Medium / Low)
- File and line reference
- Rule violated (claude.md section number)
- Recommended fix
```

**`component-checker.md`** — Automatically engages when page files are modified. Verifies decomposition rules from Section 17.2. Uses `Explore` agent (read-only analysis).

```markdown
---
name: component-checker
description: Verifies page files remain lean and features are properly decomposed into components. Auto-triggers on page file changes.
context: fork
agent: Explore
---
# Component Architecture Checker

1. Check if any page file exceeds ~200 lines
2. Verify the page only handles layout, routing state, and data
   orchestration -- not feature logic
3. Check for inline UI blocks that should be extracted to components
4. If a feature similar to one on another page is being built,
   flag it for extract-then-share per Section 17.3

Return: file name, line count, and specific extraction recommendations.
```

**`test-coverage.md`** — Automatically engages after feature completion to verify role-based test cases are updated. Uses `general-purpose` agent (may need to write test files).

```markdown
---
name: test-coverage
description: Verifies role-based test cases are updated when auth, permissions, or role-specific UI is modified. Auto-triggers after feature completion.
context: fork
agent: general-purpose
---
# Test Coverage Agent

1. Read tests/role-tests.md
2. Identify which roles are affected by the current changes
3. Check if test cases exist for the new or modified functionality
4. If test cases are missing, draft them in the standard format
   and propose additions

Return: list of affected roles, existing coverage, and proposed
new test cases.
```

Custom subagents are optional but recommended for Full Build projects. The project-specific `claude.md` should note which custom subagents are included and the kickoff prompt should instruct Claude Code to create them during project scaffolding.

#### 20.3.4 Planning for Parallel Execution
During the plan phase, Claude Code must identify parallelization opportunities at **both** levels:

**Phase-level parallelism (Agent Teams):** Which phases or major workstreams can run simultaneously? Example: "Phase 2 (schema) and Phase 3 (business logic layer) can run in parallel if the logic layer doesn't depend on schema changes being complete."

**Task-level parallelism (Subagents):** Within each phase, which tasks are independent? Example: "Phase 5 has three independent UI modules — spawn parallel tasks for Dashboard components, Admin components, and Reporting components."

The plan must explicitly identify:
- Which phases are candidates for Agent Teams (if Full Build)
- Which tasks within each phase are candidates for subagents
- Dependencies between parallel tasks (what must complete before what starts)
- File ownership per task (no two tasks edit the same files)
- Estimated token cost impact of parallelization

The project owner must approve the parallel execution plan before any parallel work begins.

### 20.4 Pre-Branch Checklist (Ongoing Development)
After the initial build ships, ongoing feature work and bug fixes must follow branch discipline. Before creating a new branch for any change, Claude Code must verify:

1. **Working tree is clean.** No uncommitted changes exist. If there are uncommitted changes, commit or stash them before proceeding.
2. **All migrations have been applied.** No pending migration files exist that haven't been run against the development database.
3. **Edge Functions are in sync.** Locally modified Edge Functions have been deployed, or are committed and match the last deployed state. No Edge Function is running code that differs from what's in Git.
4. **Main branch is current.** Pull the latest main before creating the branch.
5. **Create branch with descriptive name.** Format: `feature/<<short-description>>` or `fix/<<short-description>>`.

After the branch work is complete:
- Commit with descriptive messages tied to the feature or fix.
- Run the role-based test cases (Section 14.4) for any affected roles.
- Merge to main only after verification passes.
- Tag the release before deploying to production.
- Deploy Edge Functions from the tagged commit (Section 8.6).

This checklist prevents the common failure mode where a branch carries forward half-finished work from a previous task, making it impossible to tell whether a bug is from the new change or leftover debris.

### 20.5 Claude Code Hooks (Automated Rule Enforcement)
Claude Code supports **hooks** — scripts that auto-trigger on specific lifecycle events during a session. Hooks are configured in `.claude/settings.json` and run locally on the developer's machine. Unlike custom subagents (which are advisory), hooks can **block actions** by returning a JSON `{"decision": "block"}` response.

Hooks provide automated enforcement of framework rules at the tool level — before code is written, after code is written, and when sessions start or stop.

#### 20.5.1 Hook Event Types
Claude Code supports 12 hook events:

| Event | When It Fires | Enforcement Use |
|---|---|---|
| `PreToolUse` | Before any tool executes | Block dangerous commands, validate file targets |
| `PostToolUse` | After a tool completes successfully | Check output quality, verify patterns |
| `PostToolUseFailure` | After a tool fails | Log failures for lessons-learned |
| `PermissionRequest` | When Claude requests a permission | Log privileged action attempts |
| `Notification` | When Claude sends a notification | — |
| `Stop` | When a session completes | End-of-session reminders and checks |
| `SubagentStart` | When a subagent spawns | Track parallel work |
| `SubagentStop` | When a subagent completes | Capture subagent results |
| `PreCompact` | Before context compaction | Preserve critical context |
| `UserPromptSubmit` | When the user sends a prompt | Validate or log user inputs |
| `SessionStart` | When a session begins | Log session metadata |
| `SessionEnd` | When a session ends | Log session completion reason |
| `TeammateIdle` | When a teammate is about to go idle (Agent Teams only) | Exit code 2 to send feedback and keep teammate working |
| `TaskCompleted` | When a task is marked complete (Agent Teams only) | Exit code 2 to block completion — use as quality gate |

The last two events (`TeammateIdle` and `TaskCompleted`) are only available when using Agent Teams. They provide automated quality gates: `TaskCompleted` hooks can validate a teammate's work against acceptance criteria and reject it if it doesn't pass, preventing premature task completion.

#### 20.5.2 Hook Configuration
Hooks are defined in `.claude/settings.json` alongside permissions and environment variables. The example below shows **only the hooks portion** — see Section 20.6 for the complete `settings.json` including permission configuration. All settings go in a single file.

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "python .claude/hooks/pre_tool_use.py"
      }]
    }],
    "PostToolUse": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "python .claude/hooks/post_tool_use.py"
      }]
    }],
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "python .claude/hooks/stop.py"
      }]
    }],
    "PreCompact": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "python .claude/hooks/pre_compact.py"
      }]
    }]
  }
}
```

Hook scripts receive event data via stdin as JSON. They can read the data, perform checks, and optionally return a JSON response to block the action.

#### 20.5.3 Recommended Enforcement Hooks
The following hook scripts enforce key framework rules automatically. These are templates — adapt them to the project's specific needs.

**`.claude/hooks/pre_tool_use.py` — Pre-execution guardrails:**
```python
#!/usr/bin/env python3
"""Block dangerous operations and enforce framework rules before tool execution."""
import sys, json, os, re

def main():
    event = json.loads(sys.stdin.read())
    tool_name = event.get("tool_name", "")
    tool_input = event.get("tool_input", {})

    # Block destructive commands
    if tool_name == "Bash":
        command = tool_input.get("command", "")
        if "rm -rf /" in command or "rm -rf /*" in command:
            print(json.dumps({
                "decision": "block",
                "reason": "Blocked: destructive rm -rf command"
            }))
            return

        # Supply chain: warn on install of packages not in pre-approved list
        # Adapt APPROVED_PACKAGES to your project's PRD Section 2
        install_match = re.search(
            r'(?:npm install|npm i|pnpm add|bun add)\s+([^\s&|;]+)',
            command
        )
        if install_match:
            pkg = install_match.group(1)
            # Skip flags (--save-dev, etc.) and scoped install-all
            if not pkg.startswith("-") and pkg != ".":
                approved_file = "approved-packages.json"
                if os.path.exists(approved_file):
                    with open(approved_file) as f:
                        approved = json.load(f)
                    # Strip version specifier for comparison
                    pkg_name = re.split(r'[@^~>=<]', pkg)[0] or pkg
                    if pkg_name not in approved.get("packages", []):
                        print(json.dumps({
                            "warning": f"SUPPLY CHAIN: '{pkg_name}' is not "
                                f"in approved-packages.json. Verify: "
                                f"maintainer reputation, download count, "
                                f"last publish date, post-install scripts. "
                                f"Add to approved-packages.json if approved."
                        }))

    # Block direct modification of governing documents without approval
    if tool_name in ("Write", "Edit"):
        file_path = tool_input.get("file_path", "")
        protected_files = ["CLAUDE.md", "prd.md"]
        for protected in protected_files:
            if file_path.endswith(protected):
                print(json.dumps({
                    "decision": "block",
                    "reason": f"Blocked: {protected} is a governing document. "
                              f"Propose changes to project owner per Section 22.5."
                }))
                return

if __name__ == "__main__":
    main()
```

**`.claude/hooks/post_tool_use.py` — Post-execution quality checks:**
```python
#!/usr/bin/env python3
"""Check output quality and supply chain safety after tool execution."""
import sys, json, os, subprocess

def main():
    event = json.loads(sys.stdin.read())
    tool_name = event.get("tool_name", "")
    tool_input = event.get("tool_input", {})

    # Warn when a file exceeds the 200-line soft ceiling
    if tool_name in ("Write", "Edit"):
        file_path = tool_input.get("file_path", "")
        if file_path and os.path.exists(file_path):
            with open(file_path, "r") as f:
                line_count = sum(1 for _ in f)
            if line_count > 200 and "/pages/" in file_path:
                print(json.dumps({
                    "warning": f"File {file_path} is {line_count} lines. "
                               f"Consider decomposing per Section 17.2."
                }))

    # Supply chain: run npm audit after any npm/pnpm install
    if tool_name == "Bash":
        command = tool_input.get("command", "")
        if any(cmd in command for cmd in ["npm install", "pnpm install",
                                           "pnpm add", "npm i ",
                                           "bun install", "bun add"]):
            try:
                result = subprocess.run(
                    ["npm", "audit", "--audit-level=high", "--json"],
                    capture_output=True, text=True, timeout=30,
                    cwd=os.getcwd()
                )
                if result.returncode != 0:
                    try:
                        audit = json.loads(result.stdout)
                        vulns = audit.get("metadata", {}).get(
                            "vulnerabilities", {})
                        high = vulns.get("high", 0)
                        critical = vulns.get("critical", 0)
                        if high > 0 or critical > 0:
                            print(json.dumps({
                                "warning": f"SUPPLY CHAIN: npm audit found "
                                    f"{high} high, {critical} critical "
                                    f"vulnerabilities. Review before "
                                    f"proceeding. Run 'npm audit' for details."
                            }))
                    except json.JSONDecodeError:
                        pass
            except (subprocess.TimeoutExpired, FileNotFoundError):
                pass  # npm not available or timed out — skip silently

    # Supply chain: warn on lockfile absence after install
    if tool_name == "Bash":
        command = tool_input.get("command", "")
        if any(cmd in command for cmd in ["npm install", "pnpm install",
                                           "bun install"]):
            lockfiles = ["package-lock.json", "pnpm-lock.yaml",
                         "yarn.lock", "bun.lockb"]
            if not any(os.path.exists(lf) for lf in lockfiles):
                print(json.dumps({
                    "warning": "SUPPLY CHAIN: No lockfile found after "
                               "install. Dependency versions are not pinned. "
                               "Run install again to generate a lockfile "
                               "and commit it to Git."
                }))

if __name__ == "__main__":
    main()
```

**`.claude/hooks/stop.py` — End-of-session reminders:**
```python
#!/usr/bin/env python3
"""Remind about documentation updates when a session completes."""
import sys, json, os

def main():
    event = json.loads(sys.stdin.read())

    reminders = []

    # Check if lessons-learned.md exists and remind to update it
    if not os.path.exists("lessons-learned.md"):
        reminders.append("No lessons-learned.md found. Create one if any "
                        "failures or fixes occurred during this session.")

    # Check if role tests exist
    if not os.path.exists("tests/role-tests.md"):
        reminders.append("No tests/role-tests.md found. Create role-based "
                        "test cases per Section 14.4.")

    if reminders:
        print(json.dumps({"reminders": reminders}))

if __name__ == "__main__":
    main()
```

**`.claude/hooks/pre_compact.py` — Phase gate reminder before compaction:**
```python
#!/usr/bin/env python3
"""Inject phase gate reminder before context compaction."""
import sys, json, os

def main():
    event = json.loads(sys.stdin.read())

    reminders = [
        "PHASE GATES ARE IN EFFECT (Full Build). After completing "
        "the current phase, STOP and wait for approval before "
        "starting the next phase. Re-read CLAUDE.md Section 1.3 "
        "and STATE.md for current build position."
    ]

    # Remind to check STATE.md for current position
    if os.path.exists("STATE.md"):
        reminders.append(
            "STATE.md exists — re-read it after compaction to "
            "restore awareness of current phase, decisions, "
            "and blockers. Also re-read CONTEXT.md for "
            "implementation decisions."
        )

    print(json.dumps({"reminders": reminders}))

if __name__ == "__main__":
    main()
```

#### 20.5.4 Hook Security Considerations
Hooks run locally and have full access to the filesystem. Follow these rules:

- **Never log sensitive data in hook output.** Hook scripts should not echo environment variables, API keys, or user credentials to stdout or log files.
- **Hooks must not make external network calls** unless explicitly approved in the PRD. Sending tool inputs and outputs to external APIs for summarization or logging creates a data exposure risk, especially when working with client financials, legal documents, or PII.
- **Keep hooks lightweight.** Hooks run synchronously in the execution pipeline. A slow hook blocks Claude Code from proceeding. Target execution under 500ms.
- **Test hooks before relying on them.** A broken hook that returns invalid JSON can disrupt Claude Code's operation. Test each hook script independently before adding it to `settings.json`.
- **Hooks supplement, not replace, the build contract.** Hooks catch common violations automatically, but they cannot enforce every rule in the framework. Manual verification and the freeze audit remain the authoritative checks.

#### 20.5.5 Project Setup for Hooks
When hooks are used, the project folder structure includes:

```
.claude/
  agents/             # Custom subagents (Section 20.3.3)
  hooks/              # Hook enforcement scripts
    pre_tool_use.py   # Pre-execution guardrails
    post_tool_use.py  # Post-execution quality checks
    stop.py           # End-of-session reminders
    pre_compact.py    # Phase gate reminder before compaction (Full Build)
  settings.json       # Hook configuration + permissions + env
```

Hooks are optional but recommended for Full Build projects. For Express Build, the `pre_tool_use.py` guardrail (blocking destructive commands and governing document modifications) is recommended as a minimal safety layer.

The project-specific `claude.md` should note which hooks are active and what they enforce. The kickoff prompt should instruct Claude Code to create hook scripts during project scaffolding if hooks are declared in the PRD.

### 20.6 Permission Configuration
Claude Code's default permission system requires approval for most actions. During a build, this means hundreds of approval prompts. The goal is zero human interaction during execution — the human's role is decision-making (approving plans, resolving questions, confirming pre-build setup), not clicking approve buttons.

#### 20.6.1 Configuration: Auto Mode (Recommended)

Auto mode is Claude Code's built-in intelligent permission system (launched March 2026). A two-layer classifier reviews every action before it executes: safe actions proceed automatically, risky actions (mass file deletion, data exfiltration, malicious code) are blocked and Claude tries a safer approach. If Claude accumulates 3 consecutive denials or 20 total, it escalates to the human. In headless mode, it terminates.

**Launch command:**
```bash
claude --auto
```

Or configure in `.claude/settings.json`:

```json
{
  "permissions": {
    "defaultMode": "auto"
  },
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Add the `hooks` block from Section 20.5.2 to this same file — all settings go in one `.claude/settings.json`.

**Why auto mode over bypass permissions:**
- **Built-in safety classifier.** Auto mode runs a Sonnet 4.6-based classifier on every action, checking for destructive operations, data exfiltration, and prompt injection. Bypass permissions has no such checks.
- **Prompt injection defense.** A server-side probe scans tool outputs (file reads, web fetches, shell output) before they enter the agent's context. If content looks like an injection attempt, the agent is warned.
- **Self-recovering.** When the classifier blocks an action, Claude doesn't halt — it tries a safer approach. Bypass permissions never blocks anything.
- **Still fully autonomous.** Safe actions auto-approve with no human interaction. The 0.4% false positive rate means almost zero unnecessary interruptions for overnight builds.
- **Hooks still work on top.** Your `pre_tool_use.py` hook fires before auto mode's classifier, providing an additional project-specific safety layer.

**Defense in depth (four layers):**
1. **Auto mode classifier** (Anthropic-provided) — Blocks destructive commands, data exfiltration, and prompt injection before execution. This is the broadest safety layer.
2. **Hooks** (project-level) — `pre_tool_use.py` blocks operations that auto mode would allow but the framework prohibits (e.g., modifying `CLAUDE.md`, specific project rules). Hooks fire before the permission system.
3. **Superpowers two-stage review** (per-task) — Every implementation task is reviewed for spec compliance and code quality by separate reviewer subagents.
4. **Git** (recovery) — All work is committed incrementally. `git revert` or `git reset` recovers to a known state.

**Fallback: Bypass Permissions**
If auto mode is not available (older Claude Code versions, or specific CI/CD environments), bypass permissions remains available:
```bash
claude --dangerously-skip-permissions
```
When using bypass permissions, hooks become your **only** automated safety layer — they are mandatory, not optional. Auto mode is strongly preferred over bypass permissions for all builds.

#### 20.6.2 Project vs. Personal Settings
- **`.claude/settings.json`** — Project-level configuration. Committed to Git. Shared with all developers. Contains permission mode, hook registrations, and Agent Teams flag.
- **`.claude/settings.local.json`** — Per-developer overrides. Gitignored.
- **`~/.claude/settings.json`** — User-level global configuration. Applies to all projects.

Project settings take precedence over global settings. Local settings override project settings.

#### 20.6.3 Interaction with Hooks
Hooks execute **before** the permission system. A `pre_tool_use.py` hook can block an operation even in auto mode or bypass permissions. This is the correct design — auto mode provides broad safety, hooks enforce project-specific rules. Both layers work together.

### 20.7 Persistent Build State (`STATE.md`)
Every project must maintain a `STATE.md` file in the project root that tracks the current build state. This file is the single source of truth for where the build stands — surviving context compaction, session interruptions, and terminal restarts. Claude Code must read `STATE.md` at the start of every session and after every context compaction.

**Why this exists:** Phase gate instructions in conversation history are lost during context compaction. `CLAUDE.md` tells Claude Code *what the rules are*, but `STATE.md` tells it *where we are right now*. Without it, Claude Code after compaction doesn't know which phases are complete, what decisions have been made, or what's blocked.

**Structure:**
```markdown
# Build State

## Current Status
- **Build Mode:** Full Build / Express Build
- **Current Phase:** Phase <<N>> — <<NAME>>
- **Phase Status:** Not Started / In Progress / Awaiting Approval / Complete
- **Context Health:** <<percentage if known>>

## Phase History
| Phase | Status | Summary | Approved |
|---|---|---|---|
| Phase 0 | Complete | Architecture locked, plan approved | Yes — <<date>> |
| Phase 1 | Complete | Auth + RLS working, roles verified | Yes — <<date>> |
| Phase 2 | In Progress | Schema deployed, 3/5 entities complete | Pending |

## Key Decisions
- <<Decision made during discuss phase or build>>
- <<Decision made during discuss phase or build>>

## Blockers
- <<Anything blocking progress>>

## Implementation Decisions (from CONTEXT.md)
- <<Key UX/design decisions captured during discuss phase>>

## Notes for Next Session
- <<Anything Claude Code should know when resuming>>
```

**Update rules:**
- **Created** during pre-build scaffolding with initial state (build mode, phase 0 status).
- **Updated** at every phase gate — when a phase is marked complete, when awaiting approval, when a new phase starts.
- **Updated** when key decisions are made during the build.
- **Updated** when blockers are identified or resolved.
- **Read** by Claude Code at session start, after compaction, and before any phase transition.
- Claude Code may update `STATE.md` without explicit owner approval — this is an operational file, not a governing document. It is the exception alongside `lessons-learned.md` (see Section 22.5).

### 20.8 Discuss Phase — Implementation Decision Capture
Before building a phase, Claude Code must conduct a focused discussion with the project owner to capture implementation decisions that go beyond what the PRD specifies. The PRD defines *what* to build; the discuss phase captures *how* the owner envisions it.

**Why this exists:** The most common source of post-build disappointment is not missing features — it's features built differently than the owner imagined. The PRD says "user management page" but doesn't specify whether it's a table or a card layout, whether filters are inline or in a sidebar, what the empty state looks like, or how bulk actions work. Without this discussion, Claude Code fills in every ambiguity with reasonable defaults that may not match the owner's vision.

**When it happens:** After the plan is approved and pre-build setup is complete, but before any code is written. For Full Build, this happens once before Phase 1 (or before each phase if the owner prefers incremental discussion). For Express Build, it happens once before Step 2.

**What Claude Code asks about (based on what the phase is building):**

| If the phase includes... | Ask about... |
|---|---|
| UI pages or screens | Layout style, information density, navigation patterns, empty states, responsive behavior, key interactions |
| Forms or data entry | Field grouping, validation display, multi-step vs. single page, required vs. optional visual treatment |
| Tables or list views | Column priorities, sort defaults, filter approach, pagination vs. infinite scroll, row actions |
| Admin features | Bulk actions, search/filter power, audit visibility, permission display |
| APIs or integrations | Response format preferences, error display to users, loading state behavior |
| Dashboards | Key metrics priority, chart types, time range defaults, drill-down behavior |

**How it works:**
1. Claude Code analyzes the current phase's scope from the PRD.
2. For each area that has implementation ambiguity, Claude Code asks targeted questions.
3. The owner answers — providing as much or as little detail as they want. Skipped questions get reasonable defaults.
4. Claude Code writes the decisions to `CONTEXT.md` in the project root.
5. `CONTEXT.md` is read by Claude Code (and by subagents) during implementation, so decisions are consistently applied across all tasks.
6. Key decisions are also summarized in `STATE.md` for persistence across sessions.

**`CONTEXT.md` structure:**
```markdown
# Implementation Decisions

## Phase <<N>>: <<Phase Name>>

### UI and Layout
- <<Decision: e.g., "Card layout for user list, not table">>
- <<Decision: e.g., "Sidebar filters that collapse on mobile">>

### Interactions
- <<Decision: e.g., "Inline editing for quick fields, modal for complex edits">>
- <<Decision: e.g., "Optimistic updates with rollback on failure">>

### Empty and Error States
- <<Decision: e.g., "Illustrated empty states with action prompts, not just text">>

### Design Direction
- <<Decision: e.g., "Clean and professional, not playful. Minimal color accents.">>

### Other
- <<Decision>>
```

**Rules:**
- The discuss phase is mandatory for any phase that includes UI implementation. It is recommended but optional for pure backend phases.
- Claude Code must not skip this step even if the PRD seems detailed. The PRD describes features; the discuss phase describes experience.
- `CONTEXT.md` is cumulative — each phase's decisions are appended, not overwritten.
- Subagents executing tasks must be given `CONTEXT.md` in their spawn prompt so implementation decisions are consistently applied.
- **If Superpowers is the selected framework:** The brainstorming skill handles general design exploration and produces a formal spec document with reviewer validation. After brainstorming completes, the discuss phase runs as a **supplemental UI/UX pass** using the question table above to capture any GUI and user journey decisions that brainstorming didn't specifically address. Both brainstorming outputs and discuss phase decisions are captured in `CONTEXT.md`.
- **If GSD or BMAD is the selected framework:** This section does not apply. GSD and BMAD have their own design exploration and decision capture workflows.

### 20.9 Development Framework Selection and Integration

This build contract is a **governing layer** that works alongside third-party development frameworks. The framework handles the development process (brainstorming, planning, execution, review). This build contract handles architecture, security, production readiness, and the rules that no framework should override. The development framework is selected during the kickoff prompt based on project requirements.

#### 20.9.1 Common Infrastructure (All Frameworks)

Regardless of which development framework is selected, the following are always installed:

**Context7** (recommended):
```
/plugin install context7
```
Fetches real-time, version-specific library documentation. Prevents outdated API hallucinations. Auto-triggers when you mention library/framework names.

**Frontend Design** (required for projects with UI):
```
/plugin install frontend-design@claude-plugins-official
```
Anthropic's official skill for distinctive, production-grade frontend interfaces. Auto-activates during UI work.

**Service MCP Servers** (configured per project based on stack):
For services with MCP support, Claude Code connects directly and executes automated setup (Category 2 steps from Section 8.8). Example for Supabase:
```
claude mcp add supabase --transport http "https://mcp.supabase.com/mcp?project_ref=<YOUR_PROJECT_REF>"
```
**Safety: Always scope MCP servers to the specific new project being built. Never leave unscoped. Never connect to production.** See Section 8.8.2 for full safety rules.

**Hooks** (mandatory):
The `.claude/hooks/pre_tool_use.py` safety layer runs regardless of framework. See Section 20.5.

#### 20.9.2 Superpowers (Default Framework)

[Superpowers](https://github.com/obra/superpowers) by Jesse Vincent (100k+ stars) provides structured brainstorming, implementation planning, subagent-driven execution with two-stage review, and verification-before-completion.

**Install:** `/plugin install superpowers@claude-plugins-official`

Skills auto-trigger based on intent (slash commands are deprecated as of v5.0). Verify by saying "Let's build X" — Claude Code should invoke brainstorming, not start coding.

**How Superpowers integrates with this framework:**

- **Brainstorming + Discuss Phase:** Superpowers brainstorming runs first (general design, formal spec, reviewer validation). Our discuss phase (Section 20.8) runs as a supplemental UI/UX pass to capture GUI and user journey decisions. Both outputs go into `CONTEXT.md`.
- **Planning:** Superpowers' writing-plans skill breaks work into bite-sized tasks with file paths and verification steps. A plan-reviewer subagent validates before execution.
- **Execution:** Subagent-driven-development dispatches a fresh subagent per task with two-stage review: spec compliance ("Do Not Trust the Report") then code quality. Implementers return structured status: `DONE`, `DONE_WITH_CONCERNS`, `BLOCKED`, or `NEEDS_CONTEXT`.
- **Verification:** Verification-before-completion requires running actual commands and checking real output before claiming done.
- **Debugging:** Systematic-debugging provides 4-phase root cause analysis for complex problems.

**What this framework adds on top of Superpowers:**
Setup guide generation (`docs/resources/`), pre-build hard gate with human confirmation, open questions resolution, `.env` verification, STATE.md for build state persistence, CONTEXT.md for implementation decisions, full 75-item freeze audit, security architecture, auth patterns, lessons learned, hooks safety layer, component architecture rules, deployment discipline.

#### 20.9.3 GSD (Get Shit Done) — For Experimental/MVP Projects

[GSD](https://github.com/gsd-build/get-shit-done) by TACHES (23k+ stars) is a lightweight spec-driven system focused on preventing context rot through aggressive atomicity. Best when requirements are unclear and the product needs to be discovered through iteration.

**Install:** `npx get-shit-done-cc --claude --local`

**How GSD integrates with this framework:**

GSD has its own initialization (`/gsd:new-project`), discuss phase, research phase, planning, state tracking (`.planning/`), and execution system. These **replace** the corresponding sections of our framework. Specifically, when GSD is selected:

- **Remove from project-specific `claude.md`:** Section 7 (Deterministic Build Order, replaced by GSD's phase-by-phase planning), Section 8.8 (Setup Guides, not needed for MVPs), Section 20.7 (STATE.md, GSD uses `.planning/`), Section 20.8 (Discuss Phase, GSD has its own).
- **Keep:** All security sections (4, 5), auth patterns, error handling (9), schema migration (10), data integrity (12), code hygiene (17), hooks (20.5), permission config (20.6), custom subagents (20.3.3 if Full Build).
- **Replace freeze audit** with a lighter MVP readiness checklist: auth working, RLS enforced, no hardcoded secrets, error handling covers network calls, debug mode works, git clean, basic manual testing passed.
- **Atomic git commits:** GSD commits after every task, making `git bisect` reliable. This is stricter than our standard git requirements and should be respected.
- **Adversarial plan verification:** GSD uses dedicated planning and verifying agents where the planner creates plans and the verifier cross-checks against goals. This is stronger than our self-audit loop for plan quality.

**What this framework adds on top of GSD:**
Security architecture (RLS, auth patterns, JWT handling), lessons learned patterns (Supabase quirks, React render stability, Railway deployment fixes), hooks safety layer, Context7, Frontend Design. GSD doesn't know about your specific stack's failure patterns — this framework does.

#### 20.9.4 BMAD (Breakthrough Method for Agile AI-Driven Development) — For Enterprise/Compliance Projects

[BMAD](https://github.com/bmad-code-org/BMAD-METHOD) by BMad Code (9 agents, 50+ workflows) simulates a full agile development team. Best when requirements are locked and formal documentation is a deliverable alongside the software.

**Install:** `npx bmad-method install`

**How BMAD integrates with this framework:**

BMAD has its own brainstorming (Business Analyst), requirements (Product Manager), architecture (System Architect), sprint planning (Scrum Master), implementation (Developer), and testing (QA Engineer) workflows. These **replace** some corresponding sections. When BMAD is selected:

- **Remove from project-specific `claude.md`:** Section 20.7 (STATE.md, BMAD uses `bmad/`), Section 20.8 (Discuss Phase, BMAD has its own workflow progression).
- **Keep:** All security sections, setup guide generation (Section 8.8), pre-build hard gate, `.env` verification, full freeze audit, hooks, permission config, code hygiene, component architecture, custom subagents.
- BMAD's PRD creation can replace or supplement our PRD template. If BMAD generates the PRD, verify it covers our required security acceptance tests and implementation notes before proceeding.
- BMAD's story-by-story execution maps to our phased build model. Each completed story should satisfy the acceptance criteria that our phase gates would check.

**What this framework adds on top of BMAD:**
Security architecture (BMAD doesn't encode Supabase RLS patterns, JWT handling, or auth initialization gotchas), setup guide generation for third-party services, `.env` verification gate, stack-specific lessons learned, hooks safety layer, Context7, Frontend Design, the production-focused freeze audit.

#### 20.9.5 Priority Order (All Frameworks)
1. **Your explicit instructions** (verbal or in chat) — highest priority
2. **`CLAUDE.md`** (this build contract) — architecture, security, production rules
3. **Selected development framework** (Superpowers / GSD / BMAD) — development process and workflow
4. **Claude Code defaults** — lowest priority

All three frameworks explicitly defer to `CLAUDE.md` / user instructions when they conflict.

---

# 21. Freeze Audit Checklist

Before production readiness, Claude must confirm:

- [ ] Architecture matches approved plan
- [ ] All PRD open questions resolved before build began — no unresolved items in PRD Section 17
- [ ] Category 1 (human-only) setup confirmed complete by human before coding started -- accounts created, new projects created, `.env` populated, verified against `.env.example`
- [ ] Authorization boundaries enforced at data layer
- [ ] Table-level GRANTs applied to all public tables with RLS (anon + authenticated roles)
- [ ] First-login RLS policies handle unlinked auth_user_id state (Discord OAuth fallback — skip if using Clerk)
- [ ] No RLS policies contain subqueries on auth.users (use auth.jwt() instead)
- [ ] Edge Functions deployed with correct JWT verification flag (--no-verify-jwt only when function handles auth internally)
- [ ] Edge Functions deployed from committed, tagged code on main branch (per Section 8.6)
- [ ] Client-side auth uses getSession() as default — getUser() not used in routine auth flows (per Section 5.2.2 — skip if using Clerk)
- [ ] Auth initialization uses two-effect pattern — no async operations inside onAuthStateChange (per Section 5.2.3 — skip if using Clerk)
- [ ] TOKEN_REFRESHED events do not trigger redundant DB lookups
- [ ] React context provider values are referentially stable (useMemo on all context objects)
- [ ] No useCallback dependency arrays include React context objects (toast, api, etc.)
- [ ] No useEffect that makes API calls has unstable dependencies that could cause render loops
- [ ] No auth diagnostic logging prefixes remain in production code
- [ ] No page file exceeds ~200 lines — decomposed into components (per Section 17.2)
- [ ] No duplicated feature implementations across pages — shared components extracted (per Section 17.3)
- [ ] Custom subagents created in `.claude/agents/` for Full Build projects (security-reviewer, component-checker, test-coverage recommended — per Section 20.3.3)
- [ ] Hook enforcement scripts created in `.claude/hooks/` with pre_tool_use guardrails active (per Section 20.5 — recommended for all projects, required for Full Build)
- [ ] Hooks do not log sensitive data or make unauthorized external network calls (per Section 20.5.4)
- [ ] `.claude/settings.json` committed with auto mode permission configuration and hooks enforcement active (per Section 20.6)
- [ ] `.claude/settings.local.json` is in `.gitignore`
- [ ] Tenant isolation proven (if applicable)
- [ ] No recursive authorization logic
- [ ] No unused schema artifacts
- [ ] No commented-out production code
- [ ] No orphaned triggers or functions
- [ ] No duplicated business logic
- [ ] Manual isolation tests passed
- [ ] Manual role boundary tests passed
- [ ] Role-based test cases documented in `tests/role-tests.md` and passing (per Section 14.4)
- [ ] Playwright end-to-end tests passing for all routes and core user journeys (per Section 14.6 — if applicable)
- [ ] Self-audit verification loop completed — all PRD acceptance criteria verified as implemented (per Section 20.1)
- [ ] `STATE.md` exists and reflects all phases as complete with approval dates (per Section 20.7)
- [ ] `CONTEXT.md` exists with implementation decisions captured for all UI phases (per Section 20.8)
- [ ] Claude Code plugins reviewed and installed before build; installed plugins documented in `lessons-learned.md`
- [ ] Selected development framework installed and design/brainstorming phase completed before coding (per Section 20.9)
- [ ] Privileged actions logged
- [ ] No production data deleted during development or testing (per Section 12.3)
- [ ] Debug mode toggles correctly and does not leak data when off
- [ ] Error handling covers all network calls and async operations
- [ ] Error tracking configured for production (per Section 13.3 — if applicable)
- [ ] Feature adoption tracking events implemented for key features (per Section 13.3 — if applicable)
- [ ] Environment variables documented and no hardcoded secrets
- [ ] `.env.example` exists, is committed, and documents all required variables with grouping, descriptions, and source instructions (per Section 8.3.1)
- [ ] `.env.example` is in sync with actual environment variable usage — no missing or stale entries
- [ ] Setup guide exists in `docs/resources/` for every external service in the tech stack (per Section 8.8)
- [ ] No setup guide contains actual API keys, secrets, or credentials -- only placeholders
- [ ] All setup guides have all three categories completed (human-only, automated, post-build refinement)
- [ ] All MCP/CLI service connections scoped to the specific new project -- no unscoped or production connections
- [ ] All setup guides cross-reference related guides where setup spans multiple tools
- [ ] `docs/resources/README.md` index exists with Category 1, 2, and 3 checklists
- [ ] Migration files match current schema state
- [ ] No schema drift — no direct dashboard modifications to production RLS, functions, or triggers (per Section 8.7)
- [ ] Database backup tier declared in PRD — PITR flagged as recommended for production user-facing apps (per Section 8.5)
- [ ] Git repository is clean with descriptive commit history
- [ ] `lessons-learned.md` reviewed — any items that should be folded into master `claude.md` flagged to project owner
- [ ] `.npmrc` with `force=true` exists in project root (required for Windows to Railway cross-platform deploy)
- [ ] Supply chain: `npm audit` shows no high or critical vulnerabilities (per Section 8.9, security-framework.md Section 7)
- [ ] Supply chain: all dependency versions pinned in lockfiles, lockfiles committed to Git
- [ ] Supply chain: no dependencies with post-install scripts unless explicitly approved
- [ ] Build artifact: no source maps, debug files, `.env` files, or credential files in deployed/published artifact (per Section 8.9)
- [ ] Build artifact: if publishing to npm, `npm pack --dry-run` reviewed and artifact size within expected baseline
- [ ] Auto-remediation changelog is clean (if applicable)
- [ ] LLM calls use appropriate model tier per task (no Opus for mechanical tasks)
- [ ] No LLM calls for tasks that should be deterministic scripts
- [ ] LLM API calls include rate limit handling (backoff + jitter)
- [ ] LLM cost logging in place for runtime API calls (if applicable)
- [ ] Bulk operations are resumable with checkpoint/resume logic (if applicable)
- [ ] Nightly security audit configured (if applicable)
- [ ] Crawl policy enforced: `robots.txt`, meta robots, and `X-Robots-Tag` all set to noindex/nofollow (unless explicitly authorized)
- [ ] Debug mode URLs permanently excluded from indexing at all three layers (robots.txt, meta tag, response header) and never present in sitemap
- [ ] SEO structure in place: semantic HTML, meta tags, Open Graph, JSON-LD on public-facing pages
- [ ] SEO/structured data managed via shared utility (not scattered across components)

**Security Tier Items (from `security-framework.md`):**
For projects at Security Tier 1 or above, the applicable freeze audit items from `security-framework.md` must also pass. These are tier-specific and are appended to the project-specific `claude.md` during kickoff. The master checklist above covers Tier 0 (base security). Tier 1+ items include: network allowlist verification, supply chain audit, session security, and (for Tier 2+) credential registry, action tier classification, immutable audit logging, and canary deployment.

Final output must explicitly state:
**READ-ONLY PLAN**
or
**READY TO FREEZE**

---

# 22. Claude Best Practices

## 22.1 Auto-Memory
Claude memory should be enabled for:
- Architectural decisions
- Tenancy decisions
- Role definitions
- Calculation rules
- Security constraints
- Deployment configuration
- Dependency approvals

Memory must not store secrets.

## 22.2 Structured Outputs
Claude must:
- Use structured planning blocks
- Separate assumptions from facts
- Separate plan from execution
- Explicitly list open questions
- Present deviations with rationale and impact

## 22.3 No Hidden Behavior
Claude must not:
- Modify schema silently
- Change folder structure without approval
- Introduce new dependencies without approval
- Alter core patterns mid-build
- Write to console in production mode without debug flag

## 22.4 Reuse Over Recreation
Before creating a new utility, helper, component, or script, Claude must check whether one already exists in the project. Search the codebase for similar functionality. If a suitable implementation exists, use or extend it. Do not build parallel implementations of the same logic. This applies to auth helpers, API wrappers, UI components, data transformation utilities, and any shared code. Combined with the DRY requirement in Section 6 and the Feature Extraction Protocol in Section 17.3, this prevents the codebase from accumulating redundant implementations that diverge over time. When a feature is requested for a second page, always extract-then-share per Section 17.3 — never rebuild.

## 22.5 Documentation Integrity
Do not modify `CLAUDE.md`, `prd.md`, or any workflow/documentation file without explicit approval from the project owner. These files are governing artifacts, not living notes. If a documentation update is warranted by a discovery during the build, propose the change and wait for approval. The exceptions are:
- `lessons-learned.md` — Append-only documentation of failures and fixes (see Section 20.2)
- `STATE.md` — Operational build state tracking, updated at every phase transition (see Section 20.7)
- `CONTEXT.md` — Implementation decisions, written during the discuss phase and referenced during build (see Section 20.8)

## 22.6 Stop and Ask Triggers
Claude must pause and request clarification when encountering ambiguity around:
- Unresolved open questions in PRD Section 17 (hard gate — do not proceed until all are resolved)
- Pre-Build checklist not confirmed complete (hard gate -- do not start coding until human confirms all Category 1 setup steps done and `.env` verified)
- Tenancy model choice
- Roles and permission boundaries
- Ownership rules (who can create, edit, delete what)
- Calculation definitions, rounding rules, time periods, defaults
- Data retention, soft delete, audit requirements
- External integrations and credentials
- Deployment configuration
- Any decision that affects security boundaries

---

# 23. Required Project Kickoff Format

Every new build conversation must start with:

1. **Claude.md** attached (this document)
2. **Project PRD** (using the standard PRD template)
3. Architecture declaration (in PRD Section 8 or equivalent)
4. Tenancy declaration (in PRD or Project Metadata)
5. Auth provider confirmation (Discord default)
6. Environment variable list
7. **Instruction: Produce Plan Only**

Claude must confirm receipt of all required artifacts before beginning the plan.

---

# 24. Versioning

| Field | Value |
|---|---|
| Version | 2.4 |
| Status | Active |
| Owner | <<OWNER>> |
| Last Updated | 2026-03-09 |
| Changes from v2.3 | **v2.4 - Context Efficiency and Plugin Security.** Added Section 1.7 (Context Efficiency) as a core operating principle: only load skills, plugins, and MCP connections required for the active project. Updated security-framework.md to v1.1: MCP/plugin connections now auto-escalate security tier (Tier 1+ for any external connection, Tier 2+ if connected system contains PII/client data). Added security-framework.md Section 12 (Plugin and MCP Security Validation) with a required validation checklist covering source verification, permissions audit, data flow mapping, credential handling, auditability, compliance verification, and runtime plugin security. Six new freeze audit items for plugin security. |
| Changes from v2.2 | **v2.3 - Supply Chain Enforcement and Build Artifact Safety.** Added supply chain enforcement to hooks: `post_tool_use.py` runs `npm audit` after every install and warns on missing lockfiles; `pre_tool_use.py` warns on installation of packages not in `approved-packages.json`. Added Section 8.9 (Build Artifact Validation) with publish safety rules for npm packages, production deploy verification, deployment size gates, and Claude Code native installer recommendation (informed by the March 2026 Claude Code source map leak). Five new freeze audit items for supply chain and build artifact verification. Freeze audit now 75 items. |
| Changes from v2.1 | **v2.2 - Security Framework.** Added `security-framework.md` as a companion document with tiered security classification (Tier 0-3). Kickoff prompt assesses three dimensions (build complexity, requirements clarity, security classification). Security Tier field added to Section 0 metadata. Section 4.4 added as reference to security framework. PRD template Section 10 expanded with Security Classification, Network Allowlist, and Action Tiers subsections. Freeze audit notes tier-specific items from security framework. Never-remove list includes security tier classification. Four files now ship: claude.md, prd-template.md, kickoff-prompt-template.md, security-framework.md. |
| Changes from v2.0 | **v2.1 - Automation-First Setup.** Restructured setup guides from two-section to three-category model: Category 1 (human-only: account/project creation, credentials), Category 2 (automated: Claude Code executes via CLI/MCP during build), Category 3 (post-build refinement: production hardening). Added MCP/CLI service integration layer (Section 8.8.2) with Supabase MCP as primary example. Project safety rules: always scope to new project, never connect to production, verify target before write operations, read-only for exploration. Replaced bypass permissions with auto mode as recommended permission configuration (Section 20.6). Freeze audit updated for three-category model, MCP safety, and auto mode (70 items). |
| Changes from v1.5 | **v2.0 - Multi-Framework Orchestration.** Transformed from a single-framework system into a governing build contract that auto-selects the right development framework based on project requirements. Section 20.9 rewritten as Development Framework Selection and Integration with support for Superpowers (default, production builds), GSD (experimental/MVP builds), and BMAD (enterprise/compliance builds). Kickoff prompt now detects both build complexity (Express/Full) and requirements clarity to recommend the optimal framework. Framework-conditional rules: setup guides, pre-build gates, STATE.md, CONTEXT.md, and freeze audit scope adjust based on selected framework. Common infrastructure (Context7, Frontend Design, hooks) applies to all frameworks. |
| Changes from v1.4 | Added: Superpowers plugin integration with corrected install method. Context7 plugin. Frontend Design plugin. Bypass permissions + hooks as permission configuration. Open Questions Resolution Phase as mandatory hard gate. Pre-Build Setup hard gate. Custom subagents with `context: fork` and `agent:` type frontmatter. |
| Changes from v1.3 | Added: Setup Guide Generation system (Section 8.8). Compaction-proof phase gate rule in Section 1.3. Self-audit verification loop in Section 20.1 with STATE.md integration. PreCompact hook with STATE.md awareness. Pre-build scaffolding checklist with plugin review. Playwright E2E testing (Section 14.6). Sandbox + auto-allow permission configuration (Section 20.6). Persistent Build State via STATE.md (Section 20.7). Discuss Phase via CONTEXT.md (Section 20.8). Fresh Context Execution Pattern (Section 20.3.2.1). Expanded Freeze Audit Checklist (67 items). |
| Changes from v1.2 | Added: `.env.example` as mandatory scaffolding artifact with grouped sections, client-safe vs server-only scope, and sync requirement. Corrected Agent Teams architecture per official docs (mailbox communication, shared task list with dependency tracking, teammate direct interaction, ~5x token cost, team-specific hooks). Added Subagents via Task Tool, Custom Subagents in `.claude/agents/`, Claude Code Hooks as automated rule enforcement with template scripts (PreToolUse guardrails, PostToolUse quality checks, Stop session reminders). Added TeammateIdle and TaskCompleted hook events for Agent Teams quality gates. |
| Changes from v1.1 | Added: Clerk as alternative auth provider with Supabase integration patterns, Billing provider field, Production Observability requirements (error tracking, analytics, feature adoption, session replay), Component Architecture and Decomposition rules with file size thresholds, Feature Extraction Protocol (extract-then-share), Database Backup/PITR requirements, Edge Function Deployment Discipline (tag-based deploys only), Schema Drift Prevention, Data Safety During Development and Testing (never delete production data), Role-Based Test Cases as living document, Pre-Branch Checklist for ongoing development, Deterministic Over Probabilistic principle, Mid-Build Error Recovery Protocol with lessons-learned.md, Reuse Over Recreation rule, Documentation Integrity rule, expanded Freeze Audit Checklist |
| Changes from v1.0 | Added: Project Metadata Template, Default Stack, Pre-Approved Dependencies, RLS Helper Functions, Permissions Matrix requirement, Environment/Deployment Strategy, Error Handling/Debug Mode, Schema Migration Strategy, Testing Strategy, Auto-Remediation framework, Rollback/Recovery, Code Hygiene Rules, React Render Stability Rules, Nightly Security Audit, LLM Usage/Cost Efficiency/Processing Strategy, SEO/Crawl Policy/AI Discoverability, Build Mode (Express/Full), Agent Teams guidance, Supabase+Discord OAuth RLS rules, Edge Function JWT verification rules, Client-side getUser() vs getSession() context rules, Supabase Auth two-effect initialization pattern, Auth diagnostic logging strategy, Deterministic Over Probabilistic principle, Mid-Build Error Recovery Protocol with lessons-learned.md, Reuse Over Recreation rule, Documentation Integrity rule, Railway cross-platform build fix, expanded Freeze Audit Checklist |

This document governs all Claude-built applications unless superseded by a higher-version Claude.md.
