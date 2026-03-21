# Claude Code Build Framework
## Build Production-Quality Software with AI — From Idea to Deployed Application

A governance framework that transforms rough ideas into structured, production-ready applications built by [Claude Code](https://code.claude.com). It enforces security, quality, and best practices at every step while integrating best-in-class open source tools to handle the development workflow.

---

## Why This Exists

AI coding agents are fast but undisciplined. Left to their defaults, they make silent assumptions, skip security steps, build UI before the backend is secure, and produce code that works in a demo but breaks in production. Worse, they forget what they were doing halfway through a long build as their context window fills up.

This framework solves that by separating **what gets built** (your product spec) from **how it gets built** (the build contract) and **who does what** (the human decides, the AI executes). The human's only job during execution is decision-making — approving plans, resolving questions, confirming setup. The AI handles everything else autonomously, including overnight builds with no human present.

---

## How It's Different from Other Approaches

Most AI coding workflows fall into one of two camps: **vibe coding** (just tell the AI what to build and hope for the best) or **process frameworks** (structured workflows for how the AI should brainstorm, plan, and execute). This framework is neither — it's a **governing build contract** that works alongside process frameworks.

### What We Integrate (and Why)

This framework stands on the shoulders of two excellent open source tools:

**[Superpowers](https://github.com/obra/superpowers)** by Jesse Vincent (100k+ stars) — The best development process framework for AI coding agents. Superpowers handles *how* code gets built: structured brainstorming that produces formal design specs, implementation planning with bite-sized tasks, subagent-driven execution where every task is reviewed twice (once for spec compliance, once for code quality), and verification-before-completion that requires real output evidence before claiming work is done. It's battle-tested across thousands of developers.

**[Context7](https://github.com/upstash/context7)** by Upstash — A documentation MCP server that gives Claude Code access to real-time, version-specific library documentation. Instead of hallucinating outdated Supabase methods or React patterns from stale training data, Claude Code pulls current docs from source repositories. This prevents an entire class of bugs that come from AI coding with outdated knowledge.

**[Frontend Design](https://github.com/anthropics/claude-code/tree/main/plugins/frontend-design)** by Anthropic (277k+ installs) — The official skill for breaking Claude Code out of generic "AI slop" aesthetics. It auto-activates during UI work and pushes toward distinctive typography, purposeful color palettes, and atmospheric depth. Before writing any frontend code, Claude Code thinks through purpose, tone, constraints, and what makes this interface memorable instead of forgettable.

### What This Framework Adds on Top

Superpowers and Context7 are excellent at *development process* and *knowledge access*. They don't cover:

- **Security architecture** — RLS policies, auth patterns, JWT handling, data isolation, permission enforcement at the data layer. This framework encodes dozens of specific, tested patterns for Supabase + Discord OAuth (and Clerk) that prevent common auth failures.
- **Production readiness** — A 69-item freeze audit checklist, setup guide generation for every third-party service, deployment discipline (tagged commits, schema drift prevention, PITR backup), observability requirements, and environment variable management.
- **Stack-specific lessons learned** — Every bug fought across real builds gets encoded as a mandatory rule with the exact code pattern to follow. The `getSession()` vs `getUser()` deadlock. The two-effect auth initialization pattern. The `.npmrc` fix for Windows-to-Railway deployment. These are the lessons that save you hours of debugging.
- **Hard gates that prevent skipping steps** — Open questions must be resolved before handoff. Pre-build setup must be confirmed complete before coding starts. Claude Code verifies `.env` against `.env.example`. These gates exist because without them, critical steps get forgotten (we learned this the hard way).
- **Build state persistence** — `STATE.md` tracks which phases are complete, what decisions were made, and what's blocked. This survives context compaction and session restarts, so Claude Code always knows where the build stands.

**The integration model:** Superpowers handles development process. Context7 handles documentation access. Frontend Design handles visual quality. This framework handles architecture, security, production readiness, and the governing rules. When they conflict, the build contract wins — Superpowers explicitly defers to `CLAUDE.md`.

---

## The Three Files

| File | Purpose | When to Edit |
|---|---|---|
| `claude.md` | Master build contract — the rules for every project | After each build, when lessons learned are folded back in |
| `prd-template.md` | Blank PRD (Product Requirements Document) template | When you want to add new standard sections |
| `kickoff-prompt-template.md` | Prompt that generates project files from raw context | When you change the deliverable structure or workflow |

---

## Step-by-Step: Your First Build

### Prerequisites

- **Claude Pro, Max, Team, or Enterprise subscription** (for Claude Code access)
- **Claude Code installed** — [Install instructions](https://code.claude.com)
- **Git + GitHub** for version control
- **Node.js** for React projects
- Accounts for your stack (Supabase, Railway, Discord, etc. — the framework will generate setup guides for each)

### Step 1: Download the Framework Files

Clone this repository or download the three core files:
```bash
git clone https://github.com/dlowenth/claude-code-build-framework.git
```

You need `claude.md`, `prd-template.md`, and `kickoff-prompt-template.md`.

### Step 2: Describe What You Want to Build

Record a voice note, write out your idea, paste a conversation transcript, or just write a paragraph. Any format works. This is your raw project context. The more detail you provide, the fewer gaps Claude needs to infer.

### Step 3: Generate Your Project Files

Open a **new Claude chat** (claude.ai or the Claude app — not Claude Code yet). Paste the entire contents of `kickoff-prompt-template.md`. Attach the master `claude.md` and `prd-template.md` as files. Paste your project context at the bottom where indicated. Send it.

Claude will assess the complexity of your project, select a build mode (Express or Full), and produce **four deliverable files**:

1. **Project-specific `claude.md`** — The master build contract trimmed to only the sections relevant to your project
2. **Completed `prd.md`** — The PRD template filled in from your context, with inferences marked
3. **`kickoff-prompt.md`** — A ready-to-paste prompt for Claude Code
4. **`open-questions.md`** — Every gap, inference, and decision for your review

### Step 4: Resolve Open Questions (Mandatory Gate)

**Do not skip this step.** Claude will walk you through every open question interactively — architecture decisions, auth choices, data model questions, UX decisions. For each item, confirm, override, or provide the missing answer. Once all questions are resolved, Claude updates all four files and re-delivers them with zero unresolved items.

This gate exists because unresolved questions become silent assumptions in Claude Code that are expensive to fix later.

### Step 5: Set Up Your Project and Install Plugins

Create a new project folder. Place the project-specific `CLAUDE.md` (uppercase) and `prd.md` in the root. Initialize Git:

```bash
mkdir my-project && cd my-project
git init
```

Copy your `CLAUDE.md` and `prd.md` into the project root.

Start Claude Code with bypass permissions (so it can work autonomously):
```bash
claude --dangerously-skip-permissions
```

Install the required plugins:
```
/plugin install superpowers@claude-plugins-official
/plugin install context7
/plugin install frontend-design@claude-plugins-official
```

Restart Claude Code after installing plugins.

### Step 6: Start the Build

Paste the contents of your `kickoff-prompt.md` as the first message. Claude Code will:

1. **Read** `CLAUDE.md` and `prd.md`
2. **Run Superpowers brainstorming** — general design exploration, formal spec document, reviewer validation
3. **Run the discuss phase** — targeted UI/UX questions about layout, interactions, user journeys, empty states
4. **Generate setup guides** in `docs/resources/` for every third-party service
5. **Present the Pre-Build checklist** and ask you to confirm each item is complete
6. **Verify `.env`** against `.env.example` — won't proceed if variables are missing
7. **Produce an architecture plan** and wait for your approval

### Step 7: Approve and Execute

Once you approve the plan:

- **Express Build:** Claude Code builds the full application in one pass using Superpowers' subagent-driven execution. Every task gets a fresh context window and two-stage review. You review at the end.
- **Full Build:** Claude Code executes phase by phase. Each phase produces deliverables, gets reviewed by spec and code quality subagents, and stops for your approval before the next phase begins.

During execution, Claude Code works autonomously — no approval prompts, no interruptions. Hooks block dangerous operations. Superpowers' reviewers catch quality issues. You can walk away or let it run overnight.

### Step 8: Review and Ship

Claude Code presents the freeze audit checklist results. Review any flagged items, fix remaining issues, and deploy. After the build, review `lessons-learned.md` and fold any systemic patterns back into your master `claude.md` for the next project.

---

## How the Build Contract Learns

The master `claude.md` is a living document. Every time a build uncovers a new failure pattern — a Supabase auth quirk, a React render loop, a Railway deployment issue — the fix gets added as a mandatory rule with the exact code pattern to follow. The next project starts with that knowledge baked in.

During each build, Claude Code writes failures and fixes to a `lessons-learned.md` file. After shipping, you review that file and fold systemic lessons back into the master build contract. The system gets smarter with every project.

---

## What's Inside `claude.md`

The build contract covers 24 sections including: core operating principles (plan-first, security over velocity, deterministic over probabilistic), default technology stack, tenancy model, authorization enforcement (RLS + helper functions), backend architecture, build order (Express 3-step or Full 8-phase), environment and deployment strategy, setup guide generation, error handling and debug mode, schema migration, data integrity, performance and observability, testing strategy (including Playwright E2E), code hygiene rules (React render stability, component architecture, feature extraction), LLM usage and cost efficiency, SEO and crawl policy, Claude Code execution contract (self-audit loop, parallel execution, hooks, permissions, STATE.md, discuss phase, Superpowers integration), and a 69-item freeze audit checklist.

---

## Express Build vs. Full Build

**Express Build** (3 steps) — For simpler applications: single-tenant tools, internal dashboards, straightforward CRUD apps. One plan approval, one build pass, one freeze audit.

**Full Build** (8 phases) — For complex applications: multi-tenant SaaS, multiple user roles, complex business logic, external integrations. Phased execution with verification gates, subagent-driven task execution, and phase-by-phase approval.

The kickoff prompt detects complexity and recommends a mode. You can always override.

---

## Adapting for Your Stack

The framework ships with defaults for React + Supabase + Discord OAuth + Railway, but it's designed to be adapted. Replace stack-specific sections with equivalents for your tools while keeping the structural principles: authorization at the data layer, plan before code, lessons learned encoded as rules.

---

## Credits and Acknowledgments

- **[Superpowers](https://github.com/obra/superpowers)** by Jesse Vincent / Prime Radiant — Development workflow skills for AI coding agents. This framework integrates Superpowers as a required plugin for brainstorming, planning, execution, and code review.
- **[Context7](https://github.com/upstash/context7)** by Upstash — Real-time library documentation MCP server. Recommended plugin for preventing outdated API hallucinations.
- **[Frontend Design](https://github.com/anthropics/claude-code/tree/main/plugins/frontend-design)** by Anthropic — Official skill for producing distinctive, production-grade frontend interfaces. Required for projects with user-facing screens.
- **[Claude Code](https://code.claude.com)** by Anthropic — The AI coding agent this framework governs.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

This framework was developed through iterative real-world application builds. You are free to use, adapt, and modify these files for your own projects. If you find it useful, consider contributing your own lessons learned back to the community.
