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
| Auth Provider | Discord OAuth (default) / Other |
| Backend | Supabase (default) |
| Frontend Framework | React (default) / Other |
| Deployment Target | Railway (default) / Other |
| Edge Functions | Supabase Edge Functions (default) / Other |
| External Integrations | `<<LIST>>` |
| Repository | GitHub (required) |
| Build Mode | Express Build / Full Build (see Section 1.5) |

If any field is unresolved, it is an open question that must be answered before Phase 0 completes.

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

## 1.6 Deterministic Over Probabilistic
When a task can be accomplished by a deterministic script (Python, SQL, bash), it must not be delegated to LLM reasoning. AI handles orchestration, planning, and judgment. Scripts handle data transformation, file operations, API calls, and calculations. Every additional probabilistic step compounds error rates — five steps at 90% accuracy each yields 59% overall success. This principle governs Section 18.4 (When NOT to Use an LLM) and applies broadly: if the output is fully predictable from the input, use a script.

---

# 2. Default Technology Stack

The following stack is the default for all projects unless the PRD explicitly declares a deviation.

| Layer | Default | Notes |
|---|---|---|
| Frontend | React | — |
| Backend / Database | Supabase | Postgres + Auth + Realtime + Storage |
| Auth | Discord OAuth | Via Supabase Auth; roles stored internally |
| Edge Functions | Supabase Edge Functions | Server-side logic, webhooks, integrations |
| Deployment | Railway | Via Railway CLI or dashboard |
| Local Development | localhost | All testing occurs locally before deploy |
| Version Control | Git + GitHub | Required for all projects |
| State Management | Per PRD | Declared in PRD |
| UI Component Library | Per PRD | Declared in PRD |

### 2.1 Pre-Approved Dependencies
The following packages are pre-approved and do not require explicit approval per project:
- `@supabase/supabase-js` — Supabase client
- `react-router-dom` — Routing
- `tailwindcss` — Styling (if declared in PRD)
- `lucide-react` — Icons
- `date-fns` — Date manipulation
- `zod` — Schema validation
- `zustand` — Lightweight state management (if declared in PRD)
- `recharts` — Charting (if needed)

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

### 2.2 OAuth Standard
Default OAuth provider: **Discord**

Requirements:
- Discord OAuth must be primary authentication unless explicitly changed.
- Roles must not rely solely on OAuth claims.
- Application roles must be stored internally in database tables.
- Session strategy must be declared in PRD.

### 2.3 Identity Source of Truth
Default: **Supabase Auth** (`auth.users`).

Every project must create:
- `public.profiles` — User profile data linked to `auth.users.id`
- `public.tenant_memberships` (if multi-tenant) — Tenant/role/status junction

Identity provider may vary per project if declared in PRD:
- Supabase Auth (default)
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
| Step 2 | Build | Full app built in one pass: auth → data → logic → API → UI |
| Step 3 | Verify and Harden | Freeze audit checklist passed |

Even in Express Build, the internal priority order (auth before UI, backend before frontend) must be followed. Claude Code does not need to stop between these stages, but must not invert the order.

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

### 8.3.1 Cross-Platform Build Fix (Windows → Railway/Linux)
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

### 14.4 Automated Testing (When Ready)
When adopted, automated tests are required for:
- Calculation engine (unit tests for each formula, edge cases, regressions)
- Authorization boundary tests
- API response validation

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

### 18.1 Default Crawl Policy (Strict No-Index)
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

### 18.1.1 Debug Route Protection (Permanent)
Debug mode URLs (any URL containing `?debug=true` or the `debug` parameter) must **never** be indexable, even after the site-wide crawl policy is lifted. This is enforced at three layers:

1. **robots.txt** — `Disallow: /*?debug` must persist in all versions of the robots file
2. **Meta tag** — Any page rendered in debug mode must inject `<meta name="robots" content="noindex, nofollow">`
3. **Response header** — Debug mode requests must return `X-Robots-Tag: noindex, nofollow`
4. **Sitemap** — Debug URLs must never appear in `sitemap.xml`
5. **Canonical tags** — Pages in debug mode must set `<link rel="canonical">` to the non-debug version of the URL (stripping the debug parameter)

This protection is **permanent and non-overridable** — it is not subject to the site-wide indexing toggle.

### 18.2 SEO Structure (Build Ready, Ship Disabled)
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

### 18.3 Generative AI Discoverability
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

### 18.4 Implementation Notes
- SEO meta tags and structured data should be managed via a shared utility or layout component (DRY — not scattered across individual pages).
- If the application is a pure internal tool with no public-facing pages, Sections 18.2 and 18.3 may be scoped to marketing or landing pages only, as declared in the PRD.
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

### 20.3 Agent Teams (Full Build Only)
Claude Code Agent Teams allow multiple Claude Code instances to work in parallel, coordinated by a lead session. Agent Teams are an experimental feature and must be explicitly enabled.

**When to use Agent Teams:**
- Full Build mode only (not Express Build)
- Phases with 3+ independent workstreams that touch different files
- Parallel module development (e.g., frontend + backend + tests)
- Multi-perspective code review or debugging

**When NOT to use Agent Teams:**
- Express Build (single-pass execution, coordination overhead not justified)
- Sequential tasks with dependencies between steps
- Work where multiple teammates would edit the same files
- Simple phases with a single workstream

**Constraints:**
- Each teammate loads `claude.md`, PRD, and project context automatically but does NOT inherit the lead's conversation history
- Spawn prompts must include task-specific details — do not rely on prior conversation context
- Structure tasks so each teammate owns distinct files — no overlapping file edits
- The lead coordinates and synthesizes; it should not implement alongside teammates
- Keep teams small (2–4 teammates) to manage token cost and coordination overhead
- Agent teams consume significantly more tokens — approximately proportional to the number of active teammates
- One team per session; no nested teams; `/resume` does not restore teammates

**Planning requirement:**
During the plan phase, Claude Code should identify which phases (if any) would benefit from Agent Teams and propose a team structure. The project owner must approve the team structure before execution.

---

# 21. Freeze Audit Checklist

Before production readiness, Claude must confirm:

- [ ] Architecture matches approved plan
- [ ] Authorization boundaries enforced at data layer
- [ ] Table-level GRANTs applied to all public tables with RLS (anon + authenticated roles)
- [ ] First-login RLS policies handle unlinked auth_user_id state (Discord OAuth fallback)
- [ ] No RLS policies contain subqueries on auth.users (use auth.jwt() instead)
- [ ] Edge Functions deployed with correct JWT verification flag (--no-verify-jwt only when function handles auth internally)
- [ ] Client-side auth uses getSession() as default — getUser() not used in routine auth flows (per Section 5.2.2)
- [ ] Auth initialization uses two-effect pattern — no async operations inside onAuthStateChange (per Section 5.2.3)
- [ ] TOKEN_REFRESHED events do not trigger redundant DB lookups
- [ ] React context provider values are referentially stable (useMemo on all context objects)
- [ ] No useCallback dependency arrays include React context objects (toast, api, etc.)
- [ ] No useEffect that makes API calls has unstable dependencies that could cause render loops
- [ ] No auth diagnostic logging prefixes remain in production code
- [ ] Tenant isolation proven (if applicable)
- [ ] No recursive authorization logic
- [ ] No unused schema artifacts
- [ ] No commented-out production code
- [ ] No orphaned triggers or functions
- [ ] No duplicated business logic
- [ ] Manual isolation tests passed
- [ ] Manual role boundary tests passed
- [ ] Privileged actions logged
- [ ] Debug mode toggles correctly and does not leak data when off
- [ ] Error handling covers all network calls and async operations
- [ ] Environment variables documented and no hardcoded secrets
- [ ] Migration files match current schema state
- [ ] Git repository is clean with descriptive commit history
- [ ] `lessons-learned.md` reviewed — any items that should be folded into master `claude.md` flagged to project owner
- [ ] `.npmrc` with `force=true` exists in project root (required for Windows → Railway cross-platform deploy)
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
Before creating a new utility, helper, component, or script, Claude must check whether one already exists in the project. Search the codebase for similar functionality. If a suitable implementation exists, use or extend it. Do not build parallel implementations of the same logic. This applies to auth helpers, API wrappers, UI components, data transformation utilities, and any shared code. Combined with the DRY requirement in Section 6, this prevents the codebase from accumulating redundant implementations that diverge over time.

## 22.5 Documentation Integrity
Do not modify `CLAUDE.md`, `prd.md`, or any workflow/documentation file without explicit approval from the project owner. These files are governing artifacts, not living notes. If a documentation update is warranted by a discovery during the build, propose the change and wait for approval. The only exception is appending to `lessons-learned.md` in the project root, which is designed for in-build documentation of failures and fixes (see Section 20.2).

## 22.6 Stop and Ask Triggers
Claude must pause and request clarification when encountering ambiguity around:
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
| Version | 1.1 |
| Status | Active |
| Owner | <<OWNER>> |
| Last Updated | 2025-02-22 |
| Changes from v1.0 | Added: Project Metadata Template, Default Stack, Pre-Approved Dependencies, RLS Helper Functions, Permissions Matrix requirement, Environment/Deployment Strategy, Error Handling/Debug Mode, Schema Migration Strategy, Testing Strategy, Auto-Remediation framework, Rollback/Recovery, Code Hygiene Rules, React Render Stability Rules, Nightly Security Audit, LLM Usage/Cost Efficiency/Processing Strategy, SEO/Crawl Policy/AI Discoverability, Build Mode (Express/Full), Agent Teams guidance, Supabase+Discord OAuth RLS rules, Edge Function JWT verification rules, Client-side getUser() vs getSession() context rules, Supabase Auth two-effect initialization pattern, Auth diagnostic logging strategy, Deterministic Over Probabilistic principle, Mid-Build Error Recovery Protocol with lessons-learned.md, Reuse Over Recreation rule, Documentation Integrity rule, Railway cross-platform build fix, expanded Freeze Audit Checklist |

This document governs all Claude-built applications unless superseded by a higher-version Claude.md.
