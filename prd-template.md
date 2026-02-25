# PRD Template — Claude.md Companion

**Usage:** Fill in this template for each new application build. Attach alongside `claude.md` at project kickoff. Sections marked `<<PLACEHOLDER>>` must be completed before planning begins. Sections marked *(If Applicable)* can be removed if not relevant.

---

## 0) Document Control

| Field | Value |
|---|---|
| Product | `<<APP_NAME>>` |
| Owner | `<<OWNER>>` |
| Date | `<<DATE>>` |
| Version | `<<VERSION>>` |
| Status | Draft / In Review / Approved |
| Build Mode | Express Build / Full Build *(see claude.md Section 1.5)* |

### 0.1) Working Canvas Notes
- This is a living PRD. Sections are modular.
- Document assumptions explicitly. Do not hide uncertainty.
- If scope changes, update the Milestones and Acceptance Criteria.
- Transcript or source context used to generate this PRD should be referenced here.

---

## 1) Product Purpose and Positioning

### 1.1 What this product is
- Core job-to-be-done: `<<DESCRIPTION>>`
- Outcome users get: `<<DESCRIPTION>>`
- Minimal scope boundary: `<<DESCRIPTION>>`

### 1.2 What this product is not
- `<<Adjacent tool or workflow you are NOT replacing>>`
- `<<Workflow you are intentionally NOT supporting>>`

### 1.3 Why this matters
- What pain exists today: `<<DESCRIPTION>>`
- Why current tools or process are insufficient: `<<DESCRIPTION>>`
- Why now: `<<DESCRIPTION>>`

---

## 2) Goals, Non-Goals, and Success Metrics

### Goals
- G1: `<<GOAL>>`
- G2: `<<GOAL>>`
- G3: `<<GOAL>>`

### Non-Goals
- NG1: `<<NON_GOAL>>`
- NG2: `<<NON_GOAL>>`

### Success Metrics (Measurable)
- Adoption: `<<METRIC>>`
- Time saved: `<<METRIC>>`
- Data completeness: `<<METRIC>>`
- Accuracy: `<<METRIC>>`
- Performance: `<<METRIC>>`

---

## 3) Users, Personas, and Roles

### Personas
- **Persona 1:** `<<NAME_OR_TYPE>>`
  - Responsibilities:
  - Frequency of use:
  - Key friction points:

- **Persona 2:** `<<NAME_OR_TYPE>>`
  - Responsibilities:
  - Frequency of use:
  - Key friction points:

### Role Definitions
Define all roles used in this application. These must align with the Permissions Matrix in Section 4.

- `superuser` — `<<DESCRIPTION>>`
- `coach` or internal operator *(optional)* — `<<DESCRIPTION>>`
- `client_admin` — `<<DESCRIPTION>>`
- `member` — `<<DESCRIPTION>>`

---

## 4) Permissions Matrix

Complete this matrix for every role defined in Section 3. This is a required artifact per `claude.md` Section 4.3.

| Capability | superuser | coach | client_admin | member |
|---|---|---|---|---|
| View tenant data | | | | |
| Create records | | | | |
| Edit records | | | | |
| Delete records | | | | |
| Manage goals/settings | | | | |
| Manage users | | | | |
| Export data | | | | |
| Impersonation (support) | | | | |

### Discussion Prompts (Resolve Before Build Begins)
- Should internal roles be read-only by default?
- Are there any limited edit permissions needed during live support?
- Which actions require audit logging?

---

## 5) High-Level User Journeys

Write these as narrative flows. Keep them short and testable.

### 5.1 Authentication and Access
- Auth method: Discord OAuth via Supabase Auth *(default — change if needed)*
- Tenant access grant rules: `<<RULES>>`
- Access revocation rules: `<<RULES>>`

### 5.1.1 Auth Initialization Pattern (Required for Supabase Auth)
Per `claude.md` Section 5.2.3, the application must use a two-effect pattern for auth initialization:
- **Effect 1:** Subscribe to `onAuthStateChange`. Set session and auth user state synchronously only. No database queries inside the callback.
- **Effect 2:** React to auth user state changes. Perform user lookup (query `public.users`) outside the auth callback scope.
- On `TOKEN_REFRESHED` events, skip redundant DB lookups if user is already loaded.

This is required because the Supabase JS v2 client does not fully commit the session until the callback returns. Async operations inside the callback will deadlock.

### 5.2 Day-to-Day Workflow (Primary Persona)
1. `<<STEP>>`
2. `<<STEP>>`
3. `<<STEP>>`

### 5.3 Weekly Meeting / Cadence Workflow *(If Applicable)*
1. `<<STEP>>`

### 5.4 Planning / Review Workflow *(If Applicable)*
1. `<<STEP>>`

---

## 6) UX and Design System *(If Applicable)*

Required when you want consistent UI across multiple internal tools or when brand standards exist.

### 6.1 Brand and Layout Standards
- Header behavior: `<<DESCRIPTION>>`
- Layout grid: `<<DESCRIPTION>>`
- Card patterns: `<<DESCRIPTION>>`
- Button standards: `<<DESCRIPTION>>`
- Dark mode: Yes / No / Optional

### 6.2 Accessibility
- Target: WCAG 2.1 AA
- Keyboard navigation requirements: `<<DESCRIPTION>>`
- Touch target requirements: `<<DESCRIPTION>>`

### 6.3 Motion and Micro-Interactions
- Respect `prefers-reduced-motion`
- Keep transitions subtle

---

## 7) Core Modules

List modules as independent blocks so you can scope MVP vs. later phases.

### Module 1: `<<MODULE_NAME>>`
- Primary objects: `<<LIST>>`
- What it tracks: `<<DESCRIPTION>>`
- Key principles: `<<DESCRIPTION>>`
- Out of scope: `<<DESCRIPTION>>`

### Module 2: `<<MODULE_NAME>>`
- Primary objects: `<<LIST>>`
- What it tracks: `<<DESCRIPTION>>`
- Key principles: `<<DESCRIPTION>>`
- Out of scope: `<<DESCRIPTION>>`

*(Repeat per module.)*

---

## 8) Tenancy Model and System Architecture

This section must be completed before planning begins. Values here override `claude.md` defaults only where explicitly stated.

### Tenancy
- Selected tenancy pattern: `<<Single-tenant / Multi-tenant shared schema / Multi-tenant isolated>>`
- Tenant isolation method: `<<RLS + tenant_id / Separate schemas / Separate projects / N/A>>`
- Internal access model: `<<Assigned tenants only / Global / N/A>>`
- Impersonation allowed: Yes / No
- Audit requirement for impersonation: Yes / No

### Frontend
- Framework: React *(default)*
- State management: `<<LIBRARY_OR_APPROACH>>`
- Routing: react-router-dom *(default)*
- UI component system: `<<LIBRARY>>`

### Backend
- Database: Supabase *(default)*
- Realtime: Yes / No
- Edge Functions: Yes / No
- Background jobs: Yes / No

### Auth
- Login method: Discord OAuth via Supabase Auth *(default)*
- Session strategy: `<<DESCRIPTION>>`

### Authorization
- Source of truth: RLS policies *(default)*
- Enforcement layers: Data (RLS) → API/Edge Functions → UI *(default)*

### Deployment
- Local development: localhost *(default)*
- Production: Railway *(default)*
- Edge Functions: Supabase *(default)*
- Cross-platform build: `.npmrc` with `force=true` required in project root when developing on Windows and deploying to Railway/Linux (per `claude.md` Section 8.3.1)

### Agent Teams *(Full Build Only)*
- Use Agent Teams for parallel execution: Yes / No / TBD during planning
- If Yes, which phases are candidates for parallel work? `<<LIST>>`
- Estimated team size per phase: `<<2–4 TEAMMATES>>`

### Public-Facing Content
- Does this application have public-facing content routes? Yes / No
- If Yes, which routes? `<<LIST>>`
- *(Required for SEO/crawl policy scoping per claude.md Section 19)*

---

## 9) Data Model (Mandatory)

Provide a block for each entity. Add entities as needed.

### Entity: `<<TABLE_NAME>>`
- Purpose: `<<DESCRIPTION>>`
- Fields: `<<LIST WITH TYPES>>`
- Relationships: `<<FK REFERENCES>>`
- Indexes: `<<LIST>>`
- RLS notes: `<<POLICY APPROACH>>`

### Entity: `<<TABLE_NAME>>`
- Purpose: `<<DESCRIPTION>>`
- Fields: `<<LIST WITH TYPES>>`
- Relationships: `<<FK REFERENCES>>`
- Indexes: `<<LIST>>`
- RLS notes: `<<POLICY APPROACH>>`

*(Repeat per entity.)*

---

## 10) Security, Compliance, and Privacy

### 10.1 Security Controls
- Encryption in transit and at rest: Supabase default *(confirm)*
- RLS enabled tables: `<<LIST ALL TENANT-SCOPED TABLES>>`
- Audit logging requirements: `<<DESCRIPTION>>`
- Soft delete and purge requirements: `<<DESCRIPTION>>`
- Tenant export requirements: `<<DESCRIPTION>>`

### 10.2 Compliance Targets *(If Applicable)*
- `<<COMPLIANCE_1>>`
- `<<COMPLIANCE_2>>`

### 10.3 Security Acceptance Tests
- A user in Tenant A cannot access Tenant B data by ID guessing
- Role boundaries are enforced at DB level
- Storage access (if used) is tenant-safe
- Debug mode does not expose sensitive data
- Production console output is minimal
- Table-level GRANTs are applied to all public tables with RLS
- First-login flow works end-to-end (Discord OAuth user can authenticate and access their record before auth_user_id is linked)
- No RLS policies subquery auth.users (all use auth.jwt() for user metadata)
- Edge Functions deployed with correct JWT verification flags per `claude.md` Section 5.2.1
- Client-side auth uses getSession() as default — getUser() not used in routine auth flows per `claude.md` Section 5.2.2
- Auth initialization uses two-effect pattern (no async in onAuthStateChange) per `claude.md` Section 5.2.3

---

## 11) Business Logic and Calculations (DRY Required)

Per `claude.md` Section 6: all calculations must be defined once and reused everywhere.

### Rules
- Calculations live in a dedicated calculation layer.
- UI, reporting, exports consume results from this layer only.
- No inline math inside UI components beyond formatting.
- Any disagreement in numbers across the app is a critical defect.

### Calculation: `<<CALCULATION_NAME>>`
- Inputs: `<<LIST WITH TYPES>>`
- Outputs: `<<LIST WITH TYPES>>`
- Rounding and precision: `<<RULES>>`
- Time period assumptions: `<<RULES>>`
- Validation rules: `<<RULES>>`

*(Repeat per calculation.)*

### Tests Required (When Automated Testing Is Adopted)
- Unit tests for each formula
- Edge case tests
- Regression tests for prior bugs

---

## 12) Reporting and Exports *(If Applicable)*
- Standard reports: `<<LIST>>`
- Filters: `<<LIST>>`
- Export formats: `<<LIST>>`
- Permissions: `<<WHICH ROLES CAN ACCESS>>`

---

## 13) Non-Functional Requirements
- Mobile-first UX requirements: `<<DESCRIPTION>>`
- Performance targets (page load, interaction latency): `<<TARGETS>>`
- Scalability assumptions (records per tenant): `<<ESTIMATES>>`
- Reliability targets: `<<DESCRIPTION>>`
- LLM cost targets (if applicable): `<<DAILY/MONTHLY SPEND THRESHOLD>>`

---

## 14) Environment Variables (Required)

List all environment variables the application requires. No hardcoded secrets in source code.

| Variable | Purpose | Source |
|---|---|---|
| `SUPABASE_URL` | Supabase project URL | Supabase dashboard |
| `SUPABASE_ANON_KEY` | Supabase anonymous key | Supabase dashboard |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key (server-side only) | Supabase dashboard |
| `DISCORD_CLIENT_ID` | Discord OAuth client ID | Discord developer portal |
| `DISCORD_CLIENT_SECRET` | Discord OAuth client secret | Discord developer portal |
| `ANTHROPIC_API_KEY` | Anthropic API key (if app makes runtime LLM calls) | Anthropic console |
| `<<VARIABLE>>` | `<<PURPOSE>>` | `<<SOURCE>>` |

---

## 15) Implementation Notes for Claude Code

- Tech stack: React + Supabase + Discord OAuth *(defaults — list deviations only)*
- Centralized permission helper required (per `claude.md` Section 4.2)
- UI permissions never replace backend RLS
- Calculations isolated into shared logic modules
- Debug mode via `?debug=true` URL parameter (per `claude.md` Section 9.1)
- SEO/structured data via shared utility component (per `claude.md` Section 19)
- LLM usage governed by `claude.md` Section 18 (if applicable)
- Client-side auth must use `getSession()` as default — no `getUser()` in routine flows (per `claude.md` Section 5.2.2)
- Auth initialization must use two-effect pattern (per `claude.md` Section 5.2.3)
- All React context provider values must be referentially stable with `useMemo` (per `claude.md` Section 17.1)
- No React context objects in `useCallback` dependency arrays (per `claude.md` Section 17.1)
- Additional build notes: `<<NOTES>>`

### Edge Function Deployment Manifest *(If Using Supabase Edge Functions)*
Document the JWT verification flag for each Edge Function. Per `claude.md` Section 5.2.1, use `--no-verify-jwt` only when the function handles auth internally.

| Function Name | Handles Auth Internally | Deploy Flag |
|---|---|---|
| `<<FUNCTION_NAME>>` | Yes / No | `--no-verify-jwt` / default |
| `<<FUNCTION_NAME>>` | Yes / No | `--no-verify-jwt` / default |

---

## 15a) LLM Usage and Batch Processing *(If Applicable)*

Complete this section if the application makes runtime LLM API calls or includes batch processing scripts. Per `claude.md` Section 18, this section governs model selection, cost, and rate limit planning. If the application does not use LLM calls at runtime, mark this section "Not applicable" and move on.

### Does this application make runtime LLM API calls?
Yes / No

### LLM Task Inventory
List every task that will use an LLM call, with the model tier assigned per `claude.md` Section 18.1.

| Task | Description | Model Tier | Frequency |
|---|---|---|---|
| `<<TASK>>` | `<<DESCRIPTION>>` | Opus / Sonnet / Haiku | `<<PER_USER_ACTION / BATCH / CRON>>` |
| `<<TASK>>` | `<<DESCRIPTION>>` | Opus / Sonnet / Haiku | `<<PER_USER_ACTION / BATCH / CRON>>` |

### Expected LLM Call Volume
- Estimated calls per hour (normal usage): `<<ESTIMATE>>`
- Estimated calls per hour (peak usage): `<<ESTIMATE>>`
- Estimated daily token consumption: `<<ESTIMATE>>`
- Known account-level rate limits (TPM / RPM): `<<LIMITS>>`

### Batch Processing *(If Applicable)*
Per `claude.md` Section 18.5, batch operations default to Python scripts.

- Expected volume per batch run: `<<RECORDS>>`
- Required throughput: `<<RECORDS PER MINUTE/HOUR>>`
- Target APIs and known rate limits: `<<LIST>>`
- Processing mode: Sequential *(default)* / Parallel *(justify)*
- Checkpoint/resume strategy: `<<DESCRIPTION>>`

### Cost Controls
- Daily LLM spend alert threshold: `$<<AMOUNT>>`
- Monthly LLM budget cap: `$<<AMOUNT>>`
- Cost logging: Required per `claude.md` Section 18.6

---

## 16) Milestones and Execution Plan (Mandatory)

Build mode determines the structure of this section. Use the appropriate template below.

### --- EXPRESS BUILD ---

Use this structure if Build Mode is **Express Build**.

#### Step 1: Plan and Confirm
- Architecture summary: `<<DESCRIPTION>>`
- Data model: `<<ENTITY LIST>>`
- Auth approach: `<<DESCRIPTION>>`
- Screen / route list: `<<LIST>>`
- **Gate:** Owner approves plan before any code is written.

#### Step 2: Build
Claude Code builds the full application in one pass, following this priority order:

1. Auth and session management
2. Data structures and schema
3. Business logic / core functionality
4. API or server-side logic (if applicable)
5. UI screens and routing
6. Error handling, loading states, empty states
7. Debug mode toggle

**Acceptance criteria for the complete build:**
- `<<CRITERION>>`
- `<<CRITERION>>`
- `<<CRITERION>>`

#### Step 3: Verify and Harden
**Manual verification:**
- `<<VERIFICATION_STEP>>`
- `<<VERIFICATION_STEP>>`
- `<<VERIFICATION_STEP>>`

**Freeze audit:** Run the trimmed checklist from the project-specific `claude.md`. Fix any failures. Ship.

---

### --- FULL BUILD ---

Use this structure if Build Mode is **Full Build**. Each phase must include files affected, acceptance criteria, and manual verification steps. Phase order follows `claude.md` Section 7.

#### Phase 0: Architecture Lock
- Scope: `<<DESCRIPTION>>`
- Files affected: `<<LIST>>`
- Acceptance criteria: `<<LIST>>`
- Manual verification steps: `<<LIST>>`

#### Phase 1: Auth + Tenancy + RBAC
- Scope: `<<DESCRIPTION>>`
- Acceptance criteria: `<<LIST>>`
- Manual verification steps: `<<LIST>>`

#### Phase 2: Core Data Structures
- Scope: `<<DESCRIPTION>>`
- Acceptance criteria: `<<LIST>>`
- Manual verification steps: `<<LIST>>`

#### Phase 3: Business Logic Layer
- Scope: `<<DESCRIPTION>>`
- Acceptance criteria: `<<LIST>>`
- Manual verification steps: `<<LIST>>`

#### Phase 4: API / RPC / Edge Functions
- Scope: `<<DESCRIPTION>>`
- Acceptance criteria: `<<LIST>>`
- Manual verification steps: `<<LIST>>`

#### Phase 5: UI Implementation
- Scope: `<<DESCRIPTION>>`
- Acceptance criteria: `<<LIST>>`
- Manual verification steps: `<<LIST>>`

#### Phase 6: Notifications / Realtime *(If Applicable)*
- Scope: `<<DESCRIPTION>>`
- Acceptance criteria: `<<LIST>>`
- Manual verification steps: `<<LIST>>`

#### Phase 7: Reporting / Exports *(If Applicable)*
- Scope: `<<DESCRIPTION>>`
- Acceptance criteria: `<<LIST>>`
- Manual verification steps: `<<LIST>>`

#### Phase 8: Hardening + Freeze Audit
- Scope: Full freeze audit per `claude.md` Section 21
- Acceptance criteria: All checklist items pass
- Manual verification steps: Execute full checklist

---

## 17) Assumptions, Non-Assumptions, and Open Questions

### Explicit Assumptions
- A1: `<<ASSUMPTION>>`
- A2: `<<ASSUMPTION>>`

### Explicit Non-Assumptions
- NA1: `<<NON_ASSUMPTION>>`
- NA2: `<<NON_ASSUMPTION>>`

### Open Questions (Must Be Resolved Before Build Begins)
1. `<<QUESTION>>`
2. `<<QUESTION>>`

---

## 18) Acceptance Criteria (Top-Level)
- AC1: `<<CRITERION>>`
- AC2: `<<CRITERION>>`
- AC3: `<<CRITERION>>`

---

## 19) Change Log
- v0.1 — Initial PRD created
- v0.2 — `<<CHANGE>>`

---

## Kickoff Instructions (Send to Claude Code)

1. Attach project-specific `claude.md`
2. Attach this completed PRD
3. Send the following instruction:

**If Express Build:**
> **Produce the plan (architecture summary, data model, auth approach, screen/route list, assumptions, open questions). Do not write code. Once I approve, build the full application in one pass following the priority order, then present the freeze audit checklist for review.**

**If Full Build:**
> **Produce the Plan output only. Do not write code. Include assumptions and open questions explicitly. Break execution into phases with acceptance criteria and manual verification steps.**

Claude must confirm receipt of both `claude.md` and this PRD before beginning the plan.
