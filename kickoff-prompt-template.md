# Project Kickoff Prompt Template

**Usage:** Open a new Claude chat. Paste this entire prompt. Attach the PRD template (`prd-template.md`), the master `claude.md`, the `security-framework.md`, and your transcript or context document. Claude will produce all files needed to start the build in Claude Code.

---

## PROMPT — COPY EVERYTHING BELOW THIS LINE

You are acting as a senior product architect preparing a new application build for execution in Claude Code.

I am providing you with:
1. **Master `claude.md`** — The full governing build contract with all possible sections (attached)
2. **PRD Template** — The blank PRD structure that must be used (attached as `prd-template.md`)
3. **Security Framework** — The tiered security companion document (attached as `security-framework.md`)
4. **Project Context** — A transcript, notes, or description of what I want to build (pasted below or attached)

Your job is to produce the following deliverables, each as a separate downloadable file:

---

## Build Mode and Framework Detection

Before producing any files, assess the project from two dimensions: **build complexity** and **requirements clarity**. State both selections and your reasoning at the top of your response so I can override before you produce files.

### Dimension 1: Build Complexity

**Express Build** — Use when the project meets **all** of these criteria:
- Single-tenant or no tenancy model needed
- 5 or fewer database tables / data entities
- 1-2 user roles (or single-role / no auth complexity)
- No complex calculation engine
- No multi-system integrations
- Straightforward CRUD workflows
- Could reasonably be built to near-production in one focused session

**Full Build** — Use when **any** of these are true:
- Multi-tenant architecture
- 6+ database tables / data entities
- 3+ user roles with distinct permission boundaries
- Complex calculation engine or business logic
- Multiple external integrations
- Workflows that span multiple user journeys
- Compliance or audit requirements
- The project owner explicitly requests phased execution

### Dimension 2: Development Framework Selection

Based on the project context, select the development framework that will handle the build process. This framework runs underneath our governing build contract (`claude.md`), which always provides security architecture, lessons learned, hooks safety, and production readiness regardless of which framework is selected.

**Superpowers** (default) — Use when:
- Requirements are generally clear and the product vision is defined
- The project needs structured brainstorming, planning, and two-stage code review
- Quality and correctness matter more than speed of discovery
- The project is a production application (not a throwaway prototype)
- You're not sure which framework to pick (Superpowers is the safest default)

What Superpowers provides: Structured brainstorming with formal spec review, implementation planning with bite-sized tasks, subagent-driven execution with two-stage review (spec compliance then code quality), verification-before-completion, systematic debugging. Our discuss phase adds a supplemental UI/UX pass for GUI and user journey decisions.

What our framework adds on top: Setup guide generation for all third-party services (`docs/resources/`), pre-build hard gate with human confirmation, open questions resolution phase, `.env` verification, STATE.md for build state persistence, CONTEXT.md for implementation decisions, full 75-item freeze audit, and all security/auth/deployment rules.

**GSD (Get Shit Done)** — Use when:
- Requirements are unclear, experimental, or expected to change significantly
- The project is an MVP or prototype where discovering the product is part of the build
- You need to iterate fast without being locked into later phases
- The project hasn't been built before and requires experimentation at each step
- Speed of delivery matters more than formal documentation

What GSD provides: Lightweight project spec, phase-by-phase incremental planning (not pre-planned), parallel research agents, adversarial plan verification, atomic git commits per task, context rot prevention via aggressive atomicity, its own state management (`.planning/`).

What our framework adds on top: Security architecture (RLS, auth patterns, JWT handling), lessons learned patterns, hooks safety layer, Context7 for live docs, Frontend Design for visual quality. **Setup guides, pre-build hard gates, open questions resolution, STATE.md, and CONTEXT.md are NOT used with GSD** — GSD has its own initialization, discuss, and state tracking systems. The freeze audit is replaced with a lighter MVP readiness checklist.

**BMAD (Breakthrough Method for Agile AI-Driven Development)** — Use when:
- Requirements are locked and comprehensive documentation is a deliverable
- The project has compliance, audit, or regulatory requirements
- You want formal role-based workflows (Business Analyst, Product Manager, Architect, Scrum Master, Developer, QA)
- The project is enterprise-grade or being built for a client
- Documentation traceability (requirements to stories to code to tests) is required

What BMAD provides: 9 specialized agents simulating a full agile team, formal PRD and architecture document creation, epic and story generation, story-by-story implementation with QA review, audit-grade documentation chains, expansion packs for specialized domains.

What our framework adds on top: Security architecture, lessons learned patterns, hooks safety layer, setup guide generation, `.env` verification, Context7 for live docs, Frontend Design for visual quality, full freeze audit. **STATE.md and CONTEXT.md are NOT used with BMAD** — BMAD has its own state tracking (`bmad/`) and workflow documentation.

### Override
If I specify a build mode or framework in the project context (e.g., "this is a quick build," "use GSD," or "I want full phased execution with BMAD"), use that regardless of the assessment.

### Selection Matrix (Quick Reference)

| Requirements | Simple Project | Complex Project |
|---|---|---|
| **Clear, stable** | Superpowers + Express | Superpowers + Full Build |
| **Unclear, experimental** | GSD + Express | GSD + Full Build |
| **Locked, compliance-grade** | Superpowers + Express | BMAD + Full Build |

### Dimension 3: Security Classification

Assess the project's security tier per `security-framework.md`:

- **Tier 0: Minimal** — No auth, no sensitive data, no external APIs. CLI utilities, internal dashboards, static sites.
- **Tier 1: Standard** — Has auth, stores user data, connects to external APIs, but no financial or health data.
- **Tier 2: Elevated** — Handles PII, financial data, health data, legal documents, or connects to accounts the user cares about.
- **Tier 3: Critical** — Can move money, execute trades, manage credentials for high-value systems.

**Auto-escalation:** If the project handles financial account credentials, bank data, health records, government IDs, or data whose exposure causes measurable harm, it is automatically Tier 2+. If it can initiate transactions or move money, it is automatically Tier 3.

State the recommended tier with reasoning. The owner can override.

**What security tier determines:**
- Tier 0: Base `claude.md` security only (RLS, no hardcoded secrets, `.env` handling)
- Tier 1: Add threat modeling, network allowlist, supply chain defense, session security
- Tier 2: Add credential management, action tier system, immutable audit logging, canary detection
- Tier 3: Add hardware key encryption, full ceremony for execution-tier actions, project-type-specific controls

The project-specific `claude.md` should include only the `security-framework.md` sections applicable to the assessed tier. Tier-specific freeze audit items from the security framework are appended to the project-specific freeze audit checklist.

---

### Deliverable 1: Project-Specific `claude.md`

Produce a **customized version of `claude.md`** tailored to this specific project. Follow these rules:

**What to keep:**
- All sections that apply to this project's architecture, stack, and requirements.
- Core operating principles (Section 1) — always kept.
- Claude Code execution contract, freeze audit, and best practices — always kept, but trim checklist items that don't apply.
- Any section the project context explicitly or implicitly requires.

**Security framework integration (based on Dimension 3 assessment):**
- **Tier 0:** No additional sections from `security-framework.md`. Base `claude.md` security is sufficient.
- **Tier 1:** Include threat modeling (SF Section 2), network allowlist (SF Section 3), supply chain defense (SF Section 7), session security (SF Section 8), and plugin/MCP security validation (SF Section 12, if the project uses any external plugins or MCP connections) as new sections in the project-specific `claude.md`. Append their freeze audit items.
- **Tier 2:** Include all Tier 1 sections plus credential management (SF Section 4), action tier system (SF Section 5), immutable audit logging (SF Section 6), canary detection (SF Section 10), and AI agent security (SF Section 9, if agents are used). Append their freeze audit items.
- **Tier 3:** Include all Tier 2 sections plus hardware key encryption details (SF Sections 4.3-4.6), full ceremony requirements for execution-tier actions, and project-type-specific considerations (SF Section 11). Append their freeze audit items.

**What to remove or simplify:**
- If the project is **not multi-tenant**, strip all multi-tenancy language: tenant_id references, tenant isolation tests, tenant-scoped RLS patterns, tenant membership tables, impersonation policies. Simplify authorization sections to single-tenant equivalents.
- If the project does **not use Supabase**, strip Supabase-specific language: RLS policies, helper functions, Edge Functions, Supabase Storage, Supabase Auth references. Replace with whatever backend the project actually uses.
- If the project does **not use Supabase Edge Functions**, remove Edge Function references and replace with the actual server-side approach.
- If the project has **no public-facing content routes**, strip the SEO structure and AI discoverability sections (19.2, 19.3) but keep the crawl block policy (19.1) simplified to a permanent noindex/nofollow with no toggle-to-enable path.
- If the project has **no calculation engine**, simplify the DRY business logic section to general principles rather than the full calculation module specification.
- If the project has **no reporting/exports**, remove those references.
- If the project has **no realtime/notifications**, remove those sections.
- If the project makes **no runtime LLM API calls** and has **no batch processing**, strip the LLM Usage section (Section 18) entirely. If the project uses LLM calls but has no batch processing, keep 18.1–18.4 and 18.6 but remove 18.5 (Bulk Processing) and 18.5.1 (Adaptive Parallelism). If batch processing exists without LLM calls (pure data scripts), keep 18.4–18.5 and remove model selection and cost tracking sections.

**Build mode adjustments:**

If **Express Build**:
- Replace the Deterministic Build Order (Section 7) with a simplified structure:
  - **Step 1: Plan and Confirm** — Claude Code produces architecture summary, data model, auth approach, and screen list. Owner approves.
  - **Step 2: Build** — Claude Code builds the full application in one pass, following the priority order: auth → data structures → business logic → API → UI.
  - **Step 3: Verify and Harden** — Manual verification against the trimmed freeze audit checklist. Fix issues. Ship.
- The Claude Code execution contract should reflect this: plan confirmation is still required, but there are no per-phase gates during the build step.
- The freeze audit checklist is still required at the end.

If **Full Build**:
- Keep the standard phased structure, trimmed to applicable phases.

- If the project uses **Express Build**, remove the Agent Teams subsection (20.3.1 in master) and the Custom Subagents subsection (20.3.3) — Agent Teams and custom subagents only apply to Full Build. Keep the Subagents via Task Tool subsection (20.3.2) as subagents are available in any build mode.

**Framework-specific adjustments:**

If **Superpowers** (default):
- Keep Section 20.9 with Superpowers integration details.
- Keep setup guide generation (Section 8.8), pre-build hard gate, open questions resolution, STATE.md, CONTEXT.md, and discuss phase.
- Install instructions: `/plugin install superpowers@claude-plugins-official`

If **GSD**:
- Replace Section 20.9 with GSD integration details. Remove Superpowers-specific subsections.
- **Remove:** Setup guide generation (Section 8.8), pre-build hard gate (item 3 in scaffolding checklist), open questions resolution phase, STATE.md (Section 20.7), CONTEXT.md and discuss phase (Section 20.8). GSD has its own initialization, discuss, research, planning, and state tracking systems that replace these.
- **Keep:** Security architecture, auth patterns, lessons learned, hooks, `.env.example` (simplified), code hygiene rules, component architecture, Context7, Frontend Design.
- **Replace freeze audit** with a lighter MVP readiness checklist focused on: security basics, auth working, error handling, no hardcoded secrets, git clean, basic manual testing.
- Install instructions: `npx get-shit-done-cc --claude --local`
- The kickoff prompt for Claude Code should instruct it to use `/gsd:new-project` to initialize instead of the standard plan-first flow.

If **BMAD**:
- Replace Section 20.9 with BMAD integration details. Remove Superpowers-specific subsections.
- **Remove:** STATE.md (Section 20.7), CONTEXT.md and discuss phase (Section 20.8). BMAD has its own state tracking and workflow documentation.
- **Keep:** Setup guide generation (Section 8.8), pre-build hard gate, security architecture, auth patterns, lessons learned, hooks, `.env.example`, code hygiene rules, freeze audit (full), Context7, Frontend Design.
- Install instructions: `npx bmad-method install`
- The kickoff prompt for Claude Code should instruct it to initialize BMAD and follow its agent-guided workflow while respecting `CLAUDE.md` security and production rules.

**What to adjust:**
- Update the Default Technology Stack table (Section 2) to reflect this project's actual stack.
- Update the Pre-Approved Dependencies list to match the actual framework and libraries.
- Update the Environment Variables guidance to match the actual services.
- Update the Freeze Audit Checklist — remove items that don't apply to this project, keep everything that does.

**What to never remove regardless of project, build mode, or framework:**
- Plan-first discipline (even Express Build requires plan confirmation before coding)
- Security > velocity principle
- Deterministic over probabilistic principle
- No assumption drift / stop-and-ask triggers
- Debug mode specification (`?debug=true` toggle)
- Error handling and UI state requirements
- Git and version control requirements
- Edge Function deployment discipline (deploy from tagged commits only)
- Schema drift prevention (no direct dashboard modifications)
- Component architecture and decomposition rules (Section 17.2)
- Feature extraction protocol (Section 17.3)
- Data safety during development and testing (never delete production data)
- Code hygiene rules
- Reuse over recreation and documentation integrity
- Mid-build error recovery protocol
- Hook enforcement (pre_tool_use guardrails at minimum -- Section 20.5)
- Freeze audit checklist (full for Superpowers/BMAD, lighter MVP checklist for GSD)
- Security tier classification and applicable `security-framework.md` sections (Tier 1+)
- The versioning table at the end

**Additional items to keep for Superpowers and BMAD (remove for GSD):**
- Phase gate rule (compaction-proof -- must be in CLAUDE.md, not just conversation)
- Self-audit verification loop before declaring phases complete
- `STATE.md` persistent build state (Section 20.7) -- Superpowers only (BMAD uses its own state)
- Discuss Phase for implementation decisions before coding (Section 20.8) -- Superpowers only (BMAD uses its own workflow)
- Development framework integration (Section 20.9)
- Pre-build scaffolding checklist (plugins, setup guides, hooks, agents)
- Setup guide generation for all external services (Section 8.8)
- Pre-branch checklist (for ongoing development)

**Versioning:** Set the version to `2.4-<<APP_NAME>>` and note in the changelog that this is a project-specific derivative of master `claude.md` v2.4, listing the build mode selected, the development framework selected, the security tier assessed, and a summary of what was removed or simplified.

---

### Deliverable 2: Completed PRD (`prd.md`)

Fill in the PRD template using the project context I've provided. Follow these rules:

- **Fill in every section you have enough context to complete.** Use the transcript as your primary source. Infer reasonable defaults where the context strongly supports them, but mark inferences with `[INFERRED]` so I can review.
- **For sections where context is insufficient,** keep the placeholder and add a specific question to Section 17 (Open Questions). Do not invent details you don't have.
- **Remove PRD sections that don't apply** to this project (e.g., if no multi-tenancy, remove tenant-specific fields from Section 8; if no calculations, simplify Section 11). Keep the section number and note "Not applicable to this project" so the structure remains traceable.
- **Data Model (Section 9):** Propose a complete initial schema based on the described workflows. Include table names, field names with types, relationships, indexes, and relevant security notes.
- **Permissions Matrix (Section 4):** Fill in based on the roles and access patterns described in the context. If roles aren't explicitly stated, propose a role structure and mark it `[INFERRED]`.
- **Environment Variables (Section 14):** List all required env vars based on the integrations and services described.
- **Stack defaults** are: React, Supabase, Discord OAuth, Railway deployment, Supabase Edge Functions. If the context specifies Clerk as the auth provider, use Clerk instead and adjust auth-related sections accordingly per `claude.md` Section 2.2.1 (replace Supabase Auth patterns with Clerk patterns, adjust RLS to use string-based Clerk user IDs, remove Supabase Auth-specific sections like 5.2.2 and 5.2.3). Only note other deviations if the context calls for them. If the context specifies a different stack, use that instead and update accordingly.
- **If the project context indicates user scale beyond internal/demo use,** include an Observability section in the PRD architecture (Section 8) and flag PITR as recommended for the database backup tier.
- **The PRD must be consistent with the project-specific `claude.md`.** If you removed multi-tenancy from `claude.md`, the PRD should not reference tenant isolation. If you removed Supabase, the PRD should not reference RLS policies. If you removed the LLM section, mark PRD Section 15a as "Not applicable." If you selected Clerk as auth provider, remove Discord OAuth references and Supabase Auth-specific patterns from both files.

**Build mode adjustments:**

If **Express Build**:
- Section 16 (Milestones) should contain three steps (Plan → Build → Verify) instead of Phase 0–8. The Build step should list a prioritized sequence of what gets built and a single set of acceptance criteria and verification steps for the whole application.

If **Full Build**:
- Section 16 uses the standard phased milestone structure, one phase per block.

---

### Deliverable 3: Claude Code Kickoff Prompt (`kickoff-prompt.md`)

Generate a ready-to-paste prompt for Claude Code that:

- Instructs Claude Code to read both the project-specific `claude.md` and `prd.md` before doing anything
- States the build mode clearly

**If Express Build:**
- Tells Claude Code to produce a plan (architecture summary, data model, auth approach, screen/route list, assumptions, open questions) and **wait for approval**
- Installs the selected development framework and common plugins (per `claude.md` Section 20.9)
- **If Superpowers:** Generates setup guides, runs brainstorming + discuss phase, creates STATE.md and CONTEXT.md, presents pre-build hard gate for human confirmation, verifies `.env`
- **If GSD:** Runs `/gsd:new-project` for initialization. GSD handles its own discuss, research, and planning. Skip setup guides, STATE.md, CONTEXT.md, and pre-build hard gate.
- **If BMAD:** Runs BMAD initialization and agent-guided workflow. Generates setup guides, presents pre-build hard gate, verifies `.env`.
- Once approved and pre-build complete, instructs Claude Code to **build the full application in one pass** following the priority order in the plan
- After the build pass, presents the freeze audit checklist (full for Superpowers/BMAD, MVP readiness for GSD)
- Reminds Claude Code that even in Express mode, it must stop and ask if it encounters ambiguity on any stop-and-ask trigger
- Reminds Claude Code to create `.npmrc` with `force=true` in project root before running `npm install` (required for Windows to Railway/Linux cross-platform deploy)
- Reminds Claude Code to generate `.env.example` from the PRD's environment variables table during project scaffolding (per `claude.md` Section 8.3.1)
- Reminds Claude Code to create `.claude/settings.json` with auto mode permission configuration and hooks enforcement during scaffolding (per `claude.md` Section 20.6).

**If Full Build:**
- Tells Claude Code to operate in **plan-only mode** — no code until the plan is approved
- Asks Claude Code to produce the full plan output (architecture, data model, authorization, permissions, screens, calculations if applicable, phased execution plan)
- Installs the selected development framework and common plugins (per `claude.md` Section 20.9)
- **If Superpowers:** Generates setup guides using three-category model, runs brainstorming + discuss phase, creates STATE.md and CONTEXT.md, presents Category 1 checklist for human confirmation, verifies `.env`. Uses subagent-driven-development with two-stage review for task execution. Updates STATE.md at every phase transition.
- **If GSD:** Runs `/gsd:new-project` for initialization. GSD handles its own discuss, research, planning, and phase-by-phase execution with adversarial plan verification. Skip setup guides, STATE.md, CONTEXT.md, and pre-build hard gate. GSD uses atomic git commits per task.
- **If BMAD:** Runs BMAD initialization and follows the full agent-guided workflow (Business Analyst, Product Manager, Architect, Scrum Master, Developer, QA). Generates setup guides, presents pre-build hard gate, verifies `.env`. BMAD manages story-by-story execution with QA review.
- Reminds Claude Code of per-phase verification gates (Superpowers/BMAD) or GSD's built-in plan verification
- After the final build phase, presents the freeze audit or MVP readiness checklist

**Agent Teams (Full Build only):**
If the project uses Full Build mode and the plan contains phases with independent, parallelizable work (e.g., separate modules, frontend + backend + tests), the kickoff prompt should:
- Instruct Claude Code that Agent Teams are enabled and available
- Suggest where in the phased plan agent teams could accelerate execution (e.g., "Phase 5 UI can be split across teammates by module," or "Phase 2 schema + Phase 3 logic layer can run in parallel if no dependencies")
- Remind Claude Code of agent team constraints:
  - Teammates communicate via mailbox and share a task list with dependency tracking — design tasks with clear ownership but expect inter-agent coordination
  - Each teammate gets `claude.md` and project context automatically but NOT the lead's conversation history — include task-specific details in spawn prompts
  - Structure tasks so teammates own different files — no two teammates editing the same file
  - You can interact with individual teammates directly to course-correct without going through the lead
  - Keep teams small (2–4 teammates) — each teammate is ~5x token cost
  - Agent teams are ephemeral — they exist for one session then disappear, no `/resume`
  - Agent teams are for collaborative exploration and independent modules — if tasks don't need inter-agent discussion, use subagents instead
- Include this instruction: "When executing phases, evaluate whether agent teams would accelerate the work. If a phase has 3+ independent workstreams that touch different files, propose a team structure. Otherwise, execute as a single session."

Note: Agent Teams are not applicable to Express Build. Express Build executes in a single pass and the coordination overhead of agent teams would slow it down rather than help.

**Subagents and Task-Level Parallelism (Both Build Modes):**
The kickoff prompt should instruct Claude Code to identify task-level parallelism within each phase (or within the single Express Build pass) where independent work can be spawned as subagents via the Task tool. The plan must explicitly identify:
- Which tasks within each phase are independent and can run as parallel subagents
- File ownership per task (no two subagents editing the same files)
- Dependencies between parallel tasks (what must complete before what starts)

Include this instruction: "Within each phase, identify tasks that can run as parallel subagents. For each parallel group, specify: the task description, the files each subagent owns, and any dependencies. Use explicit prompting (e.g., 'Run Task 1, Task 2, and Task 3 in parallel') rather than hoping for automatic parallelization."

**Custom Subagents (Full Build only):**
If the project uses Full Build mode, the kickoff prompt should instruct Claude Code to create custom subagent definitions in `.claude/agents/` during project scaffolding. At minimum, recommend:
- `security-reviewer.md` — auto-triggers on auth/RLS/permission changes
- `component-checker.md` — auto-triggers when page files are modified
- `test-coverage.md` — auto-triggers after feature completion to verify test cases

Include this instruction: "During project scaffolding, create the `.claude/agents/` directory and populate it with the recommended custom subagent definitions from `claude.md` Section 20.3.3. These agents will auto-trigger during the build to enforce security, architecture, and testing standards."

**Hooks (Automated Rule Enforcement — Both Build Modes):**
The kickoff prompt should instruct Claude Code to create hook enforcement scripts in `.claude/hooks/` during project scaffolding. At minimum:
- `pre_tool_use.py` — blocks destructive commands and modifications to governing documents (recommended for all projects)
- `post_tool_use.py` — warns when page files exceed size thresholds (recommended for Full Build)
- `stop.py` — end-of-session reminders to update lessons-learned.md and role tests (recommended for Full Build)

Include this instruction: "During project scaffolding, create the `.claude/hooks/` directory and populate it with the enforcement hook scripts from `claude.md` Section 20.5.3. Update `.claude/settings.json` to register the hooks for PreToolUse, PostToolUse, and Stop events. At minimum, the `pre_tool_use.py` guardrail must be active to block destructive commands and protect governing documents."

For Express Build, only `pre_tool_use.py` is recommended. For Full Build, all three are recommended. If Agent Teams are used, also configure `TaskCompleted` hooks as quality gates to validate teammate work before tasks are marked complete (per `claude.md` Section 20.5.1).

**Both modes:**
- Reminds Claude Code of the key constraints **that are relevant to this specific project** from `claude.md` — do not list constraints you removed. If the project includes LLM calls, remind Claude Code of: model tier selection per task, rate limit handling requirements, Python as default for batch scripts, sequential-first parallelism, and cost logging.
- Reminds Claude Code to generate `docs/resources/` with setup guides using the three-category model (per `claude.md` Section 8.8): Category 1 (human-only steps), Category 2 (automated via CLI/MCP, documented for transparency), Category 3 (post-build refinement). Include a `README.md` index. No actual secrets in any guide.
- Ends with: "Confirm receipt of `claude.md` and `prd.md`, then produce the plan. Do not write code."

---

### Deliverable 4: Open Questions Summary (`open-questions.md`)

A standalone file listing:

- The build mode selected and the reasoning (so I can override if needed)
- Every gap you identified in the project context
- Every `[INFERRED]` decision you made and why, so I can confirm or override
- Every section you removed or simplified from the master `claude.md` and why, so I can override if needed
- Any architectural decisions that need my input before the plan can be finalized
- Suggested answers where you have a strong recommendation, marked as `[SUGGESTED]`

Group these by category: Build Mode, Architecture, Auth Provider, Roles/Permissions, Data Model, Business Logic, Integrations, Observability, UX, Claude.md Customization, Other.

---

## Open Questions Resolution Phase (Mandatory Before Handoff to Claude Code)

After delivering the four files, **you must walk me through every open question interactively before I take these files to Claude Code.** Do not let me proceed without resolving them. This is a hard gate.

**Process:**
1. Present each open question one at a time (or in small related groups), starting with the highest-impact items (Architecture, Auth, Data Model) before lower-impact items (UX, Observability).
2. For `[SUGGESTED]` items, present your recommendation and ask me to confirm or override.
3. For `[INFERRED]` items, explain what you assumed and why, and ask me to confirm or correct.
4. For unresolved gaps, ask me directly. If I don't have an answer, we must either agree on a reasonable default or flag it as a known risk with a documented fallback.
5. After all questions are resolved, **update all four deliverable files** to reflect the resolutions. Remove the open questions from `open-questions.md` and replace them with a "Resolved Decisions" log showing what was decided. Update the PRD sections and project-specific `claude.md` with any changes.
6. Re-deliver the updated files. The final handoff to Claude Code must include **zero unresolved open questions** in the PRD's Section 17 and zero unresolved items in `open-questions.md`.

**Why this matters:** Unresolved open questions become assumption drift in Claude Code. Claude Code will silently make decisions about unresolved items, and those decisions may be wrong. Every question resolved here saves hours of rework during the build.

---

## Rules for This Conversation

- **Do not ask me clarifying questions before producing the first draft.** Fill in what you can, flag what you can't, and give me the four files. I'll review and iterate.
- **Be opinionated where the context supports it.** I'd rather review and adjust a strong first draft than answer 30 questions before seeing anything.
- **Keep the PRD faithful to the template structure.** Do not add or renumber sections. Fill them in, mark as not applicable, or leave the placeholders.
- **The project-specific `claude.md` should be lean.** If a section doesn't apply, cut it. The goal is that Claude Code reads only what matters for this build — no dead weight.
- **All four files must be downloadable.** Produce them as actual files, not inline code blocks.
- **Consistency across files is critical.** The `claude.md`, PRD, and kickoff prompt must all agree on stack, tenancy, auth, build mode, phases, and scope. If you remove something from one, remove it from all.

---

## PROJECT CONTEXT

<<PASTE YOUR TRANSCRIPT, NOTES, OR DESCRIPTION HERE — OR ATTACH AS A SEPARATE FILE>>
