# Project Kickoff Prompt Template

**Usage:** Open a new Claude chat. Paste this entire prompt. Attach the PRD template (`prd-template.md`), the master `claude.md`, and your transcript or context document. Claude will produce all files needed to start the build in Claude Code.

---

## PROMPT — COPY EVERYTHING BELOW THIS LINE

You are acting as a senior product architect preparing a new application build for execution in Claude Code.

I am providing you with:
1. **Master `claude.md`** — The full governing build contract with all possible sections (attached)
2. **PRD Template** — The blank PRD structure that must be used (attached as `prd-template.md`)
3. **Project Context** — A transcript, notes, or description of what I want to build (pasted below or attached)

Your job is to produce the following deliverables, each as a separate downloadable file:

---

## Build Mode Detection

Before producing any files, assess the project complexity from the context provided and select one of two build modes. **State your selection and reasoning at the top of your response** so I can override before you produce files, or confirm and proceed.

### Express Build
Use when the project meets **all** of these criteria:
- Single-tenant or no tenancy model needed
- 5 or fewer database tables / data entities
- 1–2 user roles (or single-role / no auth complexity)
- No complex calculation engine
- No multi-system integrations
- Straightforward CRUD workflows
- Could reasonably be built to near-production in one focused session

**Express Build behavior:**
- The project-specific `claude.md` is significantly condensed — principles and constraints only, no multi-phase ceremony
- The PRD is streamlined — key sections filled in, ceremony sections collapsed
- **No phased execution plan.** Instead, Claude Code gets a single prioritized build sequence with a verification checklist at the end
- Claude Code is instructed to **build toward production-ready in one pass**, then pause for review before any final polish
- The freeze audit checklist is still required but trimmed to what applies
- Security, error handling, debug mode, and code hygiene rules still apply in full — Express Build is about reducing process overhead, never about cutting safety corners

### Full Build
Use when **any** of these are true:
- Multi-tenant architecture
- 6+ database tables / data entities
- 3+ user roles with distinct permission boundaries
- Complex calculation engine or business logic
- Multiple external integrations
- Workflows that span multiple user journeys
- Compliance or audit requirements
- The project owner explicitly requests phased execution

**Full Build behavior:**
- Project-specific `claude.md` retains full phase structure (trimmed to relevant phases)
- PRD uses all applicable sections with full milestone breakdown
- Claude Code operates in plan-only → phased execution → verification gate flow
- Each phase requires approval before proceeding

### Override
If I specify a build mode in the project context (e.g., "this is a quick build" or "I want full phased execution"), use that regardless of the complexity assessment.

---

### Deliverable 1: Project-Specific `claude.md`

Produce a **customized version of `claude.md`** tailored to this specific project. Follow these rules:

**What to keep:**
- All sections that apply to this project's architecture, stack, and requirements.
- Core operating principles (Section 1) — always kept.
- Claude Code execution contract, freeze audit, and best practices — always kept, but trim checklist items that don't apply.
- Any section the project context explicitly or implicitly requires.

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

- If the project uses **Express Build**, remove the Agent Teams subsection (20.2 in master) — it only applies to Full Build.

**What to adjust:**
- Update the Default Technology Stack table (Section 2) to reflect this project's actual stack.
- Update the Pre-Approved Dependencies list to match the actual framework and libraries.
- Update the Environment Variables guidance to match the actual services.
- Update the Freeze Audit Checklist — remove items that don't apply to this project, keep everything that does.

**What to never remove regardless of project or build mode:**
- Plan-first discipline (even Express Build requires plan confirmation before coding)
- Security > velocity principle
- No assumption drift / stop-and-ask triggers
- Debug mode specification (`?debug=true` toggle)
- Error handling and UI state requirements
- Git and version control requirements
- Code hygiene rules
- Freeze audit checklist (trimmed to relevant items)
- The versioning table at the end

**Versioning:** Set the version to `1.1-<<APP_NAME>>` and note in the changelog that this is a project-specific derivative of master `claude.md` v1.1, listing the build mode selected and a summary of what was removed or simplified.

---

### Deliverable 2: Completed PRD (`prd.md`)

Fill in the PRD template using the project context I've provided. Follow these rules:

- **Fill in every section you have enough context to complete.** Use the transcript as your primary source. Infer reasonable defaults where the context strongly supports them, but mark inferences with `[INFERRED]` so I can review.
- **For sections where context is insufficient,** keep the placeholder and add a specific question to Section 17 (Open Questions). Do not invent details you don't have.
- **Remove PRD sections that don't apply** to this project (e.g., if no multi-tenancy, remove tenant-specific fields from Section 8; if no calculations, simplify Section 11). Keep the section number and note "Not applicable to this project" so the structure remains traceable.
- **Data Model (Section 9):** Propose a complete initial schema based on the described workflows. Include table names, field names with types, relationships, indexes, and relevant security notes.
- **Permissions Matrix (Section 4):** Fill in based on the roles and access patterns described in the context. If roles aren't explicitly stated, propose a role structure and mark it `[INFERRED]`.
- **Environment Variables (Section 14):** List all required env vars based on the integrations and services described.
- **Stack defaults** are: React, Supabase, Discord OAuth, Railway deployment, Supabase Edge Functions. Only note deviations if the context calls for them. If the context specifies a different stack, use that instead and update accordingly.
- **The PRD must be consistent with the project-specific `claude.md`.** If you removed multi-tenancy from `claude.md`, the PRD should not reference tenant isolation. If you removed Supabase, the PRD should not reference RLS policies. If you removed the LLM section, mark PRD Section 15a as "Not applicable."

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
- Once approved, instructs Claude Code to **build the full application in one pass** following the priority order in the plan, without stopping for per-phase approval
- After the build pass, instructs Claude Code to pause and present the freeze audit checklist results for review
- Reminds Claude Code that even in Express mode, it must stop and ask if it encounters ambiguity on any stop-and-ask trigger
- Reminds Claude Code to create `.npmrc` with `force=true` in project root before running `npm install` (required for Windows → Railway/Linux cross-platform deploy)

**If Full Build:**
- Tells Claude Code to operate in **plan-only mode** — no code until the plan is approved
- Asks Claude Code to produce the full plan output (architecture, data model, authorization, permissions, screens, calculations if applicable, phased execution plan)
- Reminds Claude Code of per-phase verification gates

**Agent Teams (Full Build only):**
If the project uses Full Build mode and the plan contains phases with independent, parallelizable work (e.g., separate modules, frontend + backend + tests), the kickoff prompt should:
- Instruct Claude Code that Agent Teams are enabled and available
- Suggest where in the phased plan agent teams could accelerate execution (e.g., "Phase 5 UI can be split across teammates by module," or "Phase 2 schema + Phase 3 logic layer can run in parallel if no dependencies")
- Remind Claude Code of agent team constraints:
  - Each teammate gets `claude.md` and project context automatically but NOT the lead's conversation history — include task-specific details in spawn prompts
  - Structure tasks so teammates own different files — no two teammates editing the same file
  - The lead should coordinate and synthesize, not implement alongside teammates
  - Keep teams small (2–4 teammates) to manage token cost and coordination overhead
  - Agent teams are for parallel exploration and independent modules — sequential dependencies should stay with a single session
- Include this instruction: "When executing phases, evaluate whether agent teams would accelerate the work. If a phase has 3+ independent workstreams that touch different files, propose a team structure. Otherwise, execute as a single session."

Note: Agent Teams are not applicable to Express Build. Express Build executes in a single pass and the coordination overhead of agent teams would slow it down rather than help.

**Both modes:**
- Reminds Claude Code of the key constraints **that are relevant to this specific project** from `claude.md` — do not list constraints you removed. If the project includes LLM calls, remind Claude Code of: model tier selection per task, rate limit handling requirements, Python as default for batch scripts, sequential-first parallelism, and cost logging.
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

Group these by category: Build Mode, Architecture, Roles/Permissions, Data Model, Business Logic, Integrations, UX, Claude.md Customization, Other.

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
