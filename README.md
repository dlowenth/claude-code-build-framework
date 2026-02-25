# AI-Assisted Application Build System
## A Framework for Building Production-Quality Software with Claude Code

---

## What Is This?

This is a three-file system that turns a rough idea — a voice transcript, a set of notes, a conversation — into a structured, production-ready application built by Claude Code. It enforces security, consistency, and best practices across every build, while capturing lessons learned so the same mistakes never happen twice.

Think of it as a build contract, a project spec, and a launch checklist, all wired together so an AI coding agent can execute with discipline instead of guessing.

---

## The Problem It Solves

When you use an AI coding agent to build an application, the default experience is fast but chaotic. The agent makes assumptions, skips security steps, builds the UI before the backend is secure, introduces dependencies you didn't ask for, and produces code that works in a demo but breaks in production.

This system fixes that by:

- **Forcing a plan before any code is written.** The agent must produce an architecture plan and get approval before writing a single line.
- **Encoding hard-won lessons into the build contract.** Every bug you fight gets documented so the next project starts with that knowledge baked in.
- **Scaling the process to project complexity.** Simple tools get built in one shot. Complex multi-tenant platforms get phased execution with verification gates.
- **Keeping security as the top priority.** Authorization, data isolation, error handling, and deployment safety are enforced structurally, not left to the agent's judgment.

---

## The Three Files

### 1. `claude.md` — The Build Contract

This is the master document that governs how every application gets built. It's not a project spec — it's the rules of engagement. It covers:

- **Core principles:** Plan before code. Security over speed. No silent changes. Deterministic scripts over probabilistic AI for any task with predictable output — because every additional AI reasoning step compounds error rates.
- **Default technology stack:** React, Supabase, Discord OAuth, Railway deployment. Your stack defaults are declared once here; individual projects only need to note deviations.
- **Authentication and authorization:** Detailed rules for RLS policies, helper functions, permission matrices, and — critically — a library of specific Supabase + Discord OAuth patterns that prevent common first-login failures, session deadlocks, and policy evaluation errors.
- **Build order:** Either a full 8-phase build with verification gates at each step, or a 3-step express build for simpler projects. The agent can't build UI before the backend is secured.
- **Mid-build error recovery:** A defined protocol for what Claude Code does when something breaks — read the full error, classify it, fix and verify, check before running paid API calls, and document the failure in a `lessons-learned.md` file so you have a record to fold back into the master build contract after each project.
- **Error handling and debug mode:** A URL-toggled debug mode (`?debug=true`) that writes verbose logs to the console for troubleshooting but ships silent in production. Users see subtle error notifications, never stack traces.
- **Deployment:** Environment variables, Git requirements, Railway configuration, and a cross-platform build fix for Windows-to-Linux deployment.
- **LLM usage:** When to use an AI model vs. a Python script, which model tier to use for which task, rate limit handling, batch processing architecture, and cost tracking.
- **SEO and crawl policy:** Applications ship with indexing disabled by default. SEO structure is built in so it's ready when you flip the switch. Debug routes are permanently excluded from indexing.
- **Agent Teams:** Rules for when and how to use Claude Code's multi-agent feature for parallel execution on complex builds.
- **A freeze audit checklist:** A comprehensive list of items that must pass before any application ships.

The build contract is a living document. Every time a build uncovers a new failure pattern — a Supabase auth quirk, a React render loop, a Railway deployment issue — the fix gets added to the contract so every future project starts with that knowledge.

### 2. `prd-template.md` — The Project Spec Template

This is a blank PRD (Product Requirements Document) structure that gets filled in for each new project. It's designed to pair with the build contract and covers:

- **Product purpose and positioning:** What you're building, what you're not, and why it matters.
- **Users, roles, and permissions:** Personas, role definitions, and a required permissions matrix.
- **User journeys:** Narrative flows for key workflows.
- **Architecture:** Tenancy model, frontend/backend stack, auth method, deployment targets.
- **Data model:** Entity definitions with fields, types, relationships, indexes, and security notes.
- **Business logic:** Calculation definitions with DRY requirements — one source of truth for all math.
- **LLM usage:** If the app makes AI calls at runtime, a task inventory with model tiers, expected volume, rate limits, and cost controls.
- **Milestones:** Either a 3-step express plan or a full phased execution plan with acceptance criteria per phase.
- **Edge Function deployment manifest:** Documents the JWT verification flag for each function.
- **Auth initialization pattern:** The mandatory two-effect pattern for Supabase auth.

The template includes all the security acceptance tests and implementation notes that reference back to the build contract, so nothing falls through the cracks.

### 3. `kickoff-prompt-template.md` — The Automation Prompt

This is the prompt you paste into a new Claude chat to generate project files from raw context. You provide a transcript or notes about what you want to build, and it produces four deliverables:

1. **Project-specific `claude.md`** — A trimmed version of the master build contract with only the sections relevant to this project. A simple single-tenant app with no Supabase won't carry 24 sections of multi-tenant RLS strategy.
2. **Completed `prd.md`** — The PRD template filled in from your transcript, with inferences marked for your review.
3. **`kickoff-prompt.md`** — A ready-to-paste prompt for Claude Code that instructs it to read the build contract and PRD, then produce a plan for approval.
4. **`open-questions.md`** — Every gap, inference, and decision for you to review before the build starts.

The prompt also detects project complexity and recommends either Express Build (simple apps, one-shot execution) or Full Build (complex apps, phased with verification gates).

---

## How the Workflow Works

### Step 1: Describe What You Want to Build
Record a voice note, write out your idea, paste a transcript — any format. This is your raw project context.

### Step 2: Generate Project Files
Open a new Claude chat. Paste the kickoff prompt template. Attach the master `claude.md` and the `prd-template.md`. Paste your project context at the bottom. Send it.

Claude assesses complexity, selects a build mode, and produces four files: a project-specific build contract, a completed PRD, a Claude Code kickoff prompt, and an open questions summary.

### Step 3: Review and Iterate
Read the open questions file first. Confirm or override inferences. Answer gaps. Verify the build mode selection and architecture decisions. Ask Claude to revise until the PRD and build contract are solid.

### Step 4: Start the Build
Open your project folder. Place the project-specific `CLAUDE.md` (uppercase) and `prd.md` in the root. Initialize Git. Start Claude Code (terminal or desktop app). Paste the kickoff prompt.

Claude Code reads both files, produces an architecture plan, and waits for your approval before writing any code.

### Step 5: Execute
For Express Build: approve the plan, Claude Code builds the full app in one pass, then presents the freeze audit checklist for review.

For Full Build: approve the plan, then execute phase by phase. Each phase produces deliverables, acceptance criteria, and verification steps. You approve each phase before the next begins. Agent Teams can parallelize independent workstreams.

### Step 6: Ship
The freeze audit checklist must pass before production deployment. Every item on the checklist has been earned through real build experience.

---

## What Makes This Different

### It Learns From Every Build
The build contract isn't static. Every time a build uncovers a new class of bug — an auth deadlock, a render loop, a deployment platform quirk — the fix gets encoded as a mandatory rule with the exact code pattern to follow. The next project starts with that knowledge built in.

During each build, Claude Code writes failures and fixes to a `lessons-learned.md` file in the project root. After the build ships, you review that file and fold any systemic lessons back into the master build contract. The system gets smarter with every project — not just for you, but for everyone using the same build contract.

### It Scales to Complexity
A simple internal tool doesn't carry the overhead of a multi-tenant SaaS platform. The system detects complexity and strips irrelevant sections so the AI agent only reads what matters for this specific build.

### It Prevents, Not Just Fixes
Most of the rules in the build contract are preventive. The `.npmrc` file gets created before the first `npm install`, not after a failed Railway deploy. The auth initialization pattern is required from Phase 1, not discovered after a deadlocked login screen. The debug mode is designed into the architecture, not bolted on during troubleshooting.

### Security Is Structural
Authorization isn't a suggestion — it's enforced at the data layer, verified at each phase, and audited before ship. The build contract encodes specific, tested patterns for RLS policies, JWT handling, session management, and error exposure.

---

## Prerequisites

- **Claude subscription** (Pro, Max, Team, or Enterprise) for Claude Code access
- **Git + GitHub** for version control
- **Node.js** for React projects
- **Supabase account** (if using Supabase as backend)
- **Railway account** (if using Railway for deployment)
- **Discord application** (if using Discord OAuth)

### Claude Code Setup (One-Time)

Create or update `~/.claude/settings.json` (Mac/Linux) or `%USERPROFILE%\.claude\settings.json` (Windows):

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "permissions": {
    "allow": [
      "Read",
      "Write",
      "Edit",
      "Bash(npm *)",
      "Bash(node *)",
      "Bash(git *)",
      "Bash(npx *)",
      "Bash(python *)",
      "Bash(pip *)",
      "Bash(cd *)",
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(mkdir *)",
      "Bash(cp *)",
      "Bash(mv *)"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Bash(sudo *)",
      "Bash(chmod 777 *)",
      "Read(.env)"
    ]
  }
}
```

---

## Adapting This for Your Own Stack

The system ships with defaults for React + Supabase + Discord OAuth + Railway, but it's designed to be adapted:

- **Different backend?** Replace the Supabase-specific sections (RLS, Edge Functions, auth patterns) with equivalents for your stack. Keep the structural principles (authorization at data layer, no client trust, migration files).
- **Different auth provider?** Replace Discord OAuth references. The auth initialization pattern (two-effect, no async in callbacks) may still apply depending on your auth library.
- **Different deployment target?** Replace Railway references. The cross-platform `.npmrc` fix is Railway-specific; your platform may have its own quirks to document.
- **Different frontend?** Replace React-specific sections (render stability rules, context patterns). The principles (stable references, no render loops) apply to any reactive framework.

The key value isn't the specific technology choices — it's the system of encoding lessons learned into a build contract that prevents repeat failures and gives an AI coding agent the structure it needs to build with discipline.

---

## Quick Reference

| File | Purpose | When to Edit |
|---|---|---|
| `claude.md` (master) | Governing build contract for all projects | When a build uncovers a new lesson learned |
| `prd-template.md` | Blank PRD structure | When you want to add new standard sections |
| `kickoff-prompt-template.md` | Automation prompt for generating project files | When you change the deliverable structure |

| Deliverable | Produced By | Used In |
|---|---|---|
| Project-specific `claude.md` | Kickoff prompt → Claude | Claude Code (as `CLAUDE.md` in project root) |
| Completed `prd.md` | Kickoff prompt → Claude | Claude Code (in project root) |
| `kickoff-prompt.md` | Kickoff prompt → Claude | Pasted into Claude Code as first message |
| `open-questions.md` | Kickoff prompt → Claude | Your review before build starts |

---

## License and Attribution

This framework was developed through iterative real-world application builds, with each lesson learned encoded back into the system. You are free to use, adapt, and modify these files for your own projects.

If you find it useful, consider contributing your own lessons learned back to the community.
